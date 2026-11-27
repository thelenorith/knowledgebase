# Dispatcher fixes go live only when the host checkout is current

Scheduled tasks (BoardTick, closer) run scripts from the local checkout of
`claude-orchestrator` at `C:\Users\thele\source\_claude`, not from GitHub. A
merged PR does nothing until that checkout is fast-forwarded. In 2026-11 three
merged fixes were silently inactive because nothing pulled (work card #74).

- `board-tick.ps1` now fetches and `merge --ff-only origin/main` at the start of
  every pass (`checkout-pull-lib.ps1`); outcomes go to `state\board\checkout-pull.log`.
- It never resets or stashes, so untracked files such as `spikes/` survive; a
  non-colliding untracked file does not block the pull.
- Skips (non-main branch, dirty tracked file, diverged main, colliding untracked
  file) raise one user action on card #74; a fetch failure only after 6 passes.
- The running `board-tick.ps1` is already loaded, so its own changes apply from
  the next pass; children started in the same pass use the new code.
- Gotcha (PS 5.1): `Get-Content -Raw` of an empty file yields `$null`, and
  `[string]` of that can still be `$null`; use `"$(...)"` before `.Trim()`.

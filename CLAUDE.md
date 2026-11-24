@AGENTS.md

# Claude Code — Additional Rules

## Terminal & Raspberry Pi
- Run long jobs (parameter sweeps, simulations, builds over ~2 min) inside **tmux** on the Pi so they survive SSH drops:
  `tmux new -s <job>` → run → detach `Ctrl-b d` → reattach `tmux attach -t <job>`.
- Log long runs to a file (`... 2>&1 | tee telemetry/raw/<job>-$(date +%F).log`) and commit only a summary to `telemetry/`.
- Be mindful of Pi resources: check `free -h` and `vcgencmd measure_temp` before heavy runs.

## Before committing
- Build and run the tests (or the project's smoke check) **before every commit**. Do not commit red builds.
- Use conventional commit prefixes: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`.
- Reference the plan file in the commit body when executing a plan.

## After every completed task
- Update `STATE.md` (pull first) and, if a plan was executed, set its status to `done` with results.

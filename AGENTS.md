# AGENTS.md — Shared Operating Rules

These rules apply to **every** AI agent working in this hub or in any project repo that links back to it (Gemini and Claude Code). Model-specific files (`GEMINI.md`, `CLAUDE.md`) import this file and add only what is unique to each model.

## 1. Two-model workflow

| Lead role | Gemini | Claude Code |
|---|---|---|
| Primary strengths | Architecture, math/modeling, BOMs, full-repo audits | Local & Pi execution, testing, git, refactors |
| Typical output | Plans in `plans/`, audit reports, specs | Working code, passing tests, commits, STATE.md updates |

These are *lead* roles, not walls. Either model may plan or execute; the handoff protocol below is what keeps work coordinated.

- **Browser-session fallback:** When an agent operates in a read-only web session without local git write access (e.g. Gemini in the browser), it outputs the complete plan or `STATE.md` row in a fenced Markdown block labeled with its target file path. Alex relays it to Claude Code, which saves, commits, and pushes it unchanged (apart from fixing formatting).

### Handoff protocol
1. **Plan / Audit** — The planning model writes `plans/YYYY-MM-DD-<project>-<slug>.md` (or `-audit.md` for read-only audits) from `plans/_TEMPLATE.md`, with `status: draft` and `executor: <claude | gemini | none>`.
2. **Approve** — Alex approves either by changing `status: approved` in the plan file or by approving directly in chat (in which case the executing model updates `status` in the same commit where it sets `status: in-progress`).
3. **Execute** — The executing model sets `status: in-progress`, works through the steps, and checks each acceptance check.
4. **Close** — Push all target project-repo commits first; then in `ai_ops` set `status: done`, record repo-prefixed commit hashes (e.g., `emperorfall@abc1234`, `ai_ops@def5678`) and metrics in `## Results`, and update `STATE.md`.

Never execute a plan that is still `draft` unless Alex says so directly.

## 2. Plan before editing
- Any change touching **more than one file** requires a written plan (in `plans/` or, for small changes, in the chat) **before** the first edit.
- Single-file fixes may proceed directly but must be described in the commit message.

## 3. Source vs. build output
- **Never edit compiled or generated output directly** (`dist/`, `build/`, bundled `.html`, minified JS, generated JSON).
- Always edit `src/` (or the declared source) and run the project's build script. Commit the source; commit build output only if that project requires it.

## 4. Context hygiene
- **Do not dump large binary or JSON asset blobs into context** (map data, tilesets, model weights, sprite sheets, save files, large CSVs).
- Inspect them with targeted tools (`head`, `jq` paths, `wc`, schema summaries) and quote only the parts needed.
- **Telemetry cap:** Files committed to `telemetry/` must be aggregated summaries (`<project>-<metric>.csv` or `.json`, max 100 KB per file) with column headers and timestamps; keep raw run logs local or gitignored (`telemetry/raw/`).

## 5. STATE.md is shared
- Every agent **updates `STATE.md` after every completed task**: Current Focus, Last Metric, Next Action, and the date.
- **Pull before editing `STATE.md`** (`git pull --rebase`) to avoid clobbering the other model's update.
- Keep entries short. History belongs in `plans/` and git log, not STATE.md.

## 6. Secrets & safety
- Never commit `.env`, keys, tokens, or credentials. Use `.env.example` with placeholder values.
- Never paste secrets into plans, prompts, or telemetry summaries.
- Ask before any destructive action (force-push, history rewrite, deleting branches or data on the Pi).

## 7. Repo layout
```
plans/      Plans and audit reports (handoff documents)
prompts/    Reusable prompts for Gemini / Claude
telemetry/  Summarized metrics from runs and the Pi (raw data stays local)
pi-setup/   Raspberry Pi 5 provisioning notes and scripts
```

## 8. Project repos
Each project keeps its own repo. Its `AGENTS.md` should say:
> Follows the shared rules at github.com/Roids01/ai_ops/AGENTS.md. Report status to ai_ops/STATE.md.

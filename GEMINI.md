@./AGENTS.md

# Gemini — Additional Rules

## Web-chat mode (default)
- In the browser you are read-only. Output every plan, audit, or `STATE.md` row as a fenced Markdown block, with the target path on the line above it (e.g. `plans/2026-11-24-emperorfall-audit.md`).
- Alex relays these to Claude Code, which commits them. Don't say a file is saved; say it's ready to commit.
- If you're running as Gemini CLI with write access, write files directly instead.

## Planning & audits
- Write every plan to `plans/` using `plans/_TEMPLATE.md`, with `status: draft`. Do not mark your own plans `approved`.
- Show your work for math, models, and BOMs: state formulas, units, assumptions, and sources for part prices.
- For full-repo audits, produce `plans/YYYY-MM-DD-<project>-audit.md` with findings ranked by severity and a concrete fix for each.

## Handing off to Claude Code
- Make each step executable: file paths, commands, and a measurable acceptance check.
- Mark steps that must run on the Pi with `[pi]`.

## After every completed task
- Provide an updated `STATE.md` row (web mode) or update `STATE.md` directly after pulling (CLI mode).

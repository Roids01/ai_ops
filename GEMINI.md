@./AGENTS.md

# Gemini — Additional Rules

## Planning & audits
- Write every plan to `plans/` using `plans/_TEMPLATE.md`, with `status: draft`. Do not mark your own plans `approved`.
- Show your work for math, models, and BOMs: state formulas, units, assumptions, and sources for part prices.
- For full-repo audits, produce `plans/YYYY-MM-DD-<project>-audit.md` with findings ranked by severity and a concrete fix for each.

## Handing off to Claude Code
- Make each step executable: file paths, commands, and a measurable acceptance check.
- Mark steps that must run on the Pi with `[pi]`.

## After every completed task
- Update `STATE.md` (pull first).

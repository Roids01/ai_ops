# ai_ops — Mission Control Hub

A shared coordination repo for two AI agents and a Raspberry Pi 5.

```
            ┌──────────────┐   plans/ (draft → approved)   ┌───────────────┐
            │    Gemini    │ ─────────────────────────────▶ │  Claude Code  │
            │ architecture │                                │ execution,    │
            │ math · BOMs  │ ◀───────────────────────────── │ tests · git   │
            │ audits       │     STATE.md · telemetry/      └──────┬────────┘
            └──────────────┘                                       │ ssh + tmux
                         ▲                                         ▼
                         │                                 ┌───────────────┐
                         └──────── Alex approves plans ─── │ Raspberry Pi 5│
                                                           └───────────────┘
```

## How it works
1. **Gemini** writes a plan in `plans/` (`status: draft`).
2. **Alex** approves it.
3. **Claude Code** runs it locally or on the Pi, tests, commits, and updates `STATE.md`.
4. Results flow back as metrics in `telemetry/` for Gemini's next audit.

## Files
| File | Purpose |
|---|---|
| `AGENTS.md` | Shared rules for both models (the source of truth) |
| `CLAUDE.md` | Claude Code entry point — imports AGENTS.md |
| `GEMINI.md` | Gemini CLI entry point — imports AGENTS.md |
| `STATE.md` | One-line status per project |
| `plans/` | Plans, audits, handoffs (`_TEMPLATE.md` inside) |
| `prompts/` | Reusable prompts for each model |
| `telemetry/` | Summarized run metrics |
| `pi-setup/` | Pi 5 provisioning notes and scripts |

# Gemini Prompt 02 — Pi 5 BOM & Provisioning Plan

```
Following AGENTS.md and GEMINI.md in Roids01/ai_ops, write a plan at
plans/2026-11-24-pi5-homelab-bom-and-provisioning.md using plans/_TEMPLATE.md (status: draft, executor: claude).

Context: I'm picking up a Raspberry Pi 5 tomorrow. It will be the always-on execution box for this hub: running Claude Code over SSH, long simulation sweeps (e.g. plasmodium-atlas) in tmux, and data collectors that feed telemetry/. It replaces an earlier Pi 4 dashboard idea (Telegram bot + multi-source collectors).

Include:
1. BOM table: item, why it's needed, recommended spec, approximate price, required vs. optional. Cover RAM tier, official 27W USB-C PSU, active cooler, storage (microSD vs. NVMe via M.2 HAT — recommend one and justify), case, and anything else I'd regret skipping. Show the total for a minimal build and a recommended build.
2. Provisioning steps Claude Code can run, each with a command and an acceptance check. Mark on-device steps [pi]. Cover: OS image choice, headless SSH with key auth, hostname, updates, firewall, tmux, git + GitHub SSH key, Node.js (for Claude Code), Python venv, cloning ai_ops, and a thermal/throttle check under load.
3. Thermal and power assumptions with numbers (idle/load watts, throttle temperature).
4. Risks and open questions for me.

Write scripts as steps in the plan only; Claude Code will turn them into files in pi-setup/.
```

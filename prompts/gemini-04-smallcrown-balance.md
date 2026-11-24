# Gemini Prompt 04 — Small Crown Balance Model

Upload `prompts/smallcrown-balance-brief.md` (and optionally `telemetry/emperorfall-balance-baseline.json`) with this prompt.

```
Following AGENTS.md and GEMINI.md, read the uploaded Small Crown balance brief. Claude Code ran 240 headless bot games and found three problems: warlord boss nights (15 and 20) cause 76% of losses, late-game gold piles up unspent, and towers are the only defense worth building.

Output one fenced Markdown block for plans/2026-11-24-emperorfall-balance-pass-1.md using plans/_TEMPLATE.md (status: draft, executor: claude).

Design goals (target win rates with 20 games per cell):
- easy: decent strategies (balanced/turtle/economy) win 90–100%
- normal: 60–80%, and no single strategy more than 15 points ahead of another
- hard: 30–50%, and the economy path must be viable (≥ 25%)
- random bot stays at 0% on every difficulty
- losses should spread across nights 8–20, not cluster on 15 and 20

Include:
1. Math, with formulas shown: model expected raid HP per night, h(d) × composition × n(d), against expected tower DPS for a typical build, and show why nights 15 and 20 spike. Propose a smoother warlord curve (HP, escort, or a telegraph/warning mechanic) with the new formula.
2. A gold sink that turns late-game gold into defense without trivializing it. Give exact costs and effects (e.g. mercenaries, tower specialization, keep upgrades, emergency repairs) and use existing systems where possible.
3. Changes that make catapults, archery ranges and barracks competitive with towers (upkeep, cost, or output), with the math for gold-per-damage of each.
4. Difficulty multipliers retuned toward the targets.
5. Each change as an exact edit Claude Code can make: file (src/game/*.js), the constant or formula, old value → new value. Limit pass 1 to at most 8 changes so the results stay attributable.
6. Acceptance checks Claude Code verifies with `npm run balance -- --games 20` (it re-runs the same 240-game grid and compares against the baseline).

Do not propose changes to rendering, UI text, or files outside src/game/.
```

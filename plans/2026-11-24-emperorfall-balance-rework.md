---
project: emperorfall
title: Economy, Defense Viability, and Boss-Night Balance Rework
author: gemini
executor: claude
status: draft   # draft → approved → in-progress → done
created: 2026-11-24
---

## Goal
Rebalance Small Crown (`EmperorFall`) economy, unit/defense costs, and Warlord boss nights so that late-game gold converts into meaningful defense, Catapults and Barracks/Archery Ranges compete with Tower spam, and all three non-random bot archetypes achieve viable, well-separated win rates across Easy, Normal, and Hard.

## Context
- Repo: `Roids01/EmperorFall` @ `ef89bcf`; build command: `node build.mjs` (never edit compiled `index.html` or `game.js` directly).
- Baseline telemetry: `telemetry/emperorfall-balance-baseline.json` (240 games via `tools/bot.js`, 20 games per cell).
- Baseline win rates (Easy / Normal / Hard):
  - `balanced` (holy): `75% / 55% / 30%`
  - `turtle` (warlord): `90% / 75% / 60%`
  - `economy` (merchant): `85% / 55% / 15%`
  - `random`: `0% / 0% / 0%`
- Core mathematical diagnosis from `smallcrown-balance-brief.md`:
  1. **Boss-night cliff (55 of 72 non-random losses = 76% on nights 15 & 20):**
     - Normal wave counts: `n(5)=11`, `n(10)=21`, `n(15)=30`, `n(20)=38`.
     - Base HP `h(d) = 2 + floor(0.75d)`: `h(5)=5`, `h(10)=9`, `h(15)=13`, `h(20)=17`.
     - Warlord HP (`5.2h`): Normal `26.0 → 46.8 → 67.6 → 88.4`; Hard (`×1.15`) `29.9 → 53.8 → 77.7 → 101.7`.
     - Hard difficulty compounds `wave 1.22 × hp 1.15 = 1.403×` total raid EHP while starting at `0.85×` resources.
     - Speed climbs `1.15 → 1.70`, reducing tower time-on-target by ~32% by night 20.
  2. **Superlinear Tower dominance vs. double-taxed alternatives:**
     - Tower DPS (`L / max(0.36, 0.75 - 0.15(L-1))`): `L1 = 1.33 DPS`, `L2 = 3.33 DPS`, `L3 = 6.67 DPS` (`0` gold upkeep, `0` food).
     - Catapult single-target DPS (`3 / 2.4s`): `1.25 DPS` (`+1 gold/day` upkeep → 20 gold total from day 4); must hit 5.3+ enemies/shot to match one L3 tower.
     - Barracks/Archery Range: `1 gold/day` upkeep + `1 food/soldier` creates a permanent dual-resource drag.
  3. **Unspent late-game gold (median end gold: balanced 157, turtle 192, economy 232):**
     - One-time upgrades exhaust by mid-game; tower upgrades are stone-gated; market buy price compounds `+12%/buy` (`1.12^10 = 3.11×`, `1.12^20 = 9.65×`), preventing Merchant/economy builds from converting gold into wood/stone or combat power.

### Proposed Formula Changes (Math Shown)

| System | Current formula / value | Proposed value | Mathematical rationale |
|---|---|---|---|
| **Warlord HP** (`enemyStats`) | `5.2 × h(d)` (Normal d15: 67.6, d20: 88.4) | `4.0 × h(d)` (Normal d15: 52.0, d20: 68.0) | Cuts ~23% off boss HP spike without trivializing d5 (`20.0`) or d10 (`36.0`) |
| **Boss-night escort size** (`planWave`) | `1.00 × n(d)` on `d % 5 === 0` | `0.82 × n(d)` on `d = 10, 15, 20` | Boss nights test burst/frontline holding instead of stacking a 68+ HP boss onto a full 38-unit swarm |
| **Enemy speed cap** (`enemyStats`) | `min(1.70, 0.95 + 0.04d)` | `min(1.50, 0.95 + 0.03d)` | Preserves late-game tower/catapult firing windows (`d20`: `1.50` vs `1.70`, +13% time-in-range) |
| **Difficulty curves** (`DIFF`) | Easy `0.75w / 0.85hp` (`0.638×` EHP); Hard `1.22w / 1.15hp / 0.85res` (`1.403×` EHP) | Easy `0.70w / 0.80hp / 1.35res` (`0.560×` EHP); Hard `1.15w / 1.12hp / 0.90res` (`1.288×` EHP) | Widens Easy forgiving floor (~90–95% win rate) and removes Hard's excessive early-res + 1.40× double squeeze |
| **Tower attack rate** (`weapon`) | `max(0.36, 0.75 - 0.15(L-1))` → `1.33 / 3.33 / 6.67 DPS` | `max(0.45, 0.78 - 0.11(L-1))` → `1.28 / 2.99 / 5.36 DPS` | Keeps L2/L3 upgrades strong (2.3× / 4.2×) while cutting runaway L3 dominance by ~20% |
| **Catapult** (`weapon` / `econ`) | Dmg `3`, rate `2.4s` (`1.25 DPS`), upkeep `1g/day` | Dmg `4`, rate `2.0s` (`2.00 DPS` + splash), cost `{wood 6, stone 4, gold 6}`, upkeep `0` | Eliminates upkeep penalty; hitting 3 enemies yields `6.0 DPS`, rivaling L3 towers against swarms |
| **Barracks & Archery Range** (`econ`) | Upkeep `1g/day` + `1 food/soldier` | Upkeep `0g/day` (keep `1 food/soldier`) | Removes double-tax so food-rich/balanced builds can field real frontlines to stall Warlords |
| **Market buy price escalation** (`econ`) | `base × 1.12^k` (unbounded; `9.65×` at `k=20`) | `base × min(2.5, 1 + 0.08k)` (or reset/decay 1 step per day) | Keeps gold-to-stone/wood conversion viable late game without making markets infinitely cheap |
| **Late-game Gold Sink** (`02-rules-economy.js`, `03-sim-folk-combat.js`) | None after 13 one-time upgrades | Add repeatable **War Chest / Mercenary Levy** (e.g., `18g` + `6g` per prior use: spawns 2 veteran guards for the upcoming night, OR `25g + 10g/tier` repeatable **Guild Munitions** giving `+10%` defense dmg, capped at 5 tiers = 225g max) | Gives Merchant/economy builds a direct, wood/stone-free way to spend 150–230 late-game gold |

## Steps
1. [ ] **Inspect exact constants and functions in `EmperorFall` before editing** — files: `src/game/02-rules-economy.js`, `src/game/03-sim-folk-combat.js`, `src/game/12-worldmap-economy.js`, `src/game/15-paths-victory.js`, `tools/bot.js` — command: `git status && node tools/bot.js --help || head -n 60 tools/bot.js`
   - *Acceptance check:* Exact variable names for `planWave`, `enemyStats`, `DIFF`, `weapon`, `econ`, market buy scaling, and bot build heuristics are confirmed against this plan (flag any mismatch before editing).
2. [ ] **Rebalance boss nights, speed cap, and difficulty multipliers** — files: `src/game/02-rules-economy.js` — command: Apply `Warlord = 4.0h`, `0.82×` escort scaling on nights `10, 15, 20`, speed `min(1.50, 0.95 + 0.03d)`, and updated `DIFF` table; verify build with `node build.mjs`.
   - *Acceptance check:* `node build.mjs` exits `0` with zero syntax/bundle errors; spot-check printed wave tables for nights 5, 10, 15, 20 on Easy/Normal/Hard match proposed math.
3. [ ] **Rebalance Tower scaling, Catapults, Barracks/Archery Range, and Market escalation** — files: `src/game/02-rules-economy.js`, `src/game/03-sim-folk-combat.js` — command: Update tower fire rate (`max(0.45, 0.78 - 0.11*(L-1))`), catapult stats (`dmg 4`, `rate 2.0s`, `cost {wood 6, stone 4, gold 6}`, `upkeep 0`), remove `1g/day` gold upkeep on barracks/archery range (keep `1 food` per soldier), and cap market price escalation at `min(2.5, 1 + 0.08 * buys)`; run `node build.mjs`.
   - *Acceptance check:* `node build.mjs` exits `0`; unit/building cost tooltips and `src/i18n-es.js` strings remain consistent if any static cost text appears in UI.
4. [ ] **Implement repeatable late-game Gold Sink** — files: `src/game/02-rules-economy.js`, `src/game/03-sim-folk-combat.js`, `src/game/13-dawn-hooks-ui.js`, `src/i18n-es.js` — command: Add one repeatable gold-only sink (recommended: repeatable **Guild Munitions** `+10%` defense damage for `25 + 10*tier` gold up to 5 tiers, or **Mercenary Levy** for `18 + 6*uses` gold spawning guards for the night) and wire its UI trigger + Spanish localization key; run `node build.mjs`.
   - *Acceptance check:* `node build.mjs` exits `0`; purchasing the sink deducts gold cleanly, applies combat effect, persists across save/load (`09-sound-save-ui.js`), and renders cleanly in both EN and ES.
5. [ ] **Update `tools/bot.js` to exercise new levers and re-run 240-game matrix** — files: `tools/bot.js`, `ai_ops/telemetry/emperorfall-balance-v2.json` — command: Update bot logic so `economy` uses the softened market + repeatable gold sink and `balanced`/`turtle` can evaluate Catapults and Barracks/Ranges without hard-coded avoidance; run the full 240-game matrix (20 games × 4 bots × 3 difficulties) and write a compact summary (`< 100 KB`) to `ai_ops/telemetry/emperorfall-balance-v2.json`.
   - *Acceptance check:* All 240 games complete with zero uncaught exceptions, summary JSON is `< 100 KB`, and before/after tables are recorded in `## Results`.
6. [ ] **Iterative tuning pass (max 2 loops) if acceptance targets miss** — files: `src/game/02-rules-economy.js`, `src/game/03-sim-folk-combat.js`, `ai_ops/telemetry/emperorfall-balance-v2.json` — command: Adjust only the specific constant driving any missed target (e.g., Warlord HP or sink cost), rebuild (`node build.mjs`), and re-run the 240-game matrix.
   - *Acceptance check:* All quantitative targets in `## Acceptance checks` pass simultaneously.

## Acceptance checks
- [ ] `node build.mjs` succeeds cleanly in `Roids01/EmperorFall` with zero console/runtime errors across 240 bot runs.
- [ ] **Win-rate bands (20 games per cell):**
  - Easy: `balanced`, `turtle`, and `economy` all in **85%–100%**; `random` stays **0%–5%**.
  - Normal: `balanced`, `turtle`, and `economy` all in **60%–85%** (no single strategy >20 percentage points above another).
  - Hard: `balanced`, `turtle`, and `economy` all in **35%–65%** (specifically lifting `economy` from 15% to ≥35%).
- [ ] **Loss distribution:** Night 15 + Night 20 combined share of non-random losses drops from **76%** to **≤55%** (more even pressure across nights 8–20).
- [ ] **Gold sink effectiveness:** Median unspent end-of-game gold across `balanced`, `turtle`, and `economy` drops below **80 gold** (down from 157 / 192 / 232).
- [ ] **Defense diversity:** Average Catapults + Barracks + Archery Ranges built per game rises to **≥3.0** combined for `balanced` (up from <1 catapult / 1–2 barracks), while Tower spam remains viable without being the sole optimal building.
- [ ] Summary telemetry file committed to `ai_ops/telemetry/emperorfall-balance-v2.json` is `< 100 KB` with timestamps and headers.

## Risks / open questions
- **Gold sink design preference (Question for Alex):** Which repeatable gold-only late-game sink fits *Small Crown*'s theme better?
  - **Option A — Repeatable Guild Munitions** (`25g → 35g → 45g → 55g → 65g`, +10% global defense damage per tier, max 5 tiers = 225g): minimal UI/entity overhead, directly helps Merchant builds kill d15/d20 Warlords.
  - **Option B — Mercenary Levy** (`18g + 6g/use`, hires temporary veteran soldiers for the upcoming night): more active/tactical, physically blocks Warlords and sappers, slightly more work in `03-sim-folk-combat.js`.
  - **Option C — Both** (Mercenary Levy for emergency night-15/20 blocking + Munitions for passive scaling).
- **Bot AI bias caveat:** If `tools/bot.js` currently hard-codes "build `2 + d/1.6` towers" and barely considers Catapults/Barracks/Market buys, buffing those buildings won't show up in telemetry until Step 5 updates the bot's decision weights. Keep track of which delta came from *game rules* vs. *smarter bot* by running one 240-game pass with the *old* bot after Step 3 before updating `tools/bot.js`.
- **Human vs. bot ceiling:** Because bots ignore hero abilities, trade routes, the court (`court_en.ink`), and the world map, human players will outperform bots by ~15–25%. Target Hard bot win rates around ~40–55% (not 70%+) so Hard still feels genuinely tense for human players.
- **Keep HP vs. Warlord damage:** Warlord `dmg 3` against `10 HP` Keep kills in 4 hits; if reducing Warlord HP to `4.0h` still leaves sudden keep-snipe losses on d15/d20, consider reducing Warlord keep damage from `3` to `2` during Step 6.

## Results (filled in by executor)
- Commits:
- Metrics:
- Notes:
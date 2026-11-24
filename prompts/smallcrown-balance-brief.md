# Small Crown (EmperorFall): Balance Brief for Gemini

_Prepared by Claude Code, 2026-11-24, from repo Roids01/EmperorFall @ `ef89bcf`._
_Baseline data: `telemetry/emperorfall-balance-baseline.json` (240 bot games)._

## The game in one paragraph
A 15×15 tile kingdom around a central keep (10 HP). By day you place buildings and buy upgrades. Each night a raid walks in from 1 to 4 edge spawns. Survive 20 nights to win. If the keep falls, you lose. Gold comes from taxes, markets, kills and path buildings. Wood, stone and food come from producer buildings that need villagers.

## Key rules (from source)

**Raid size per night** (`planWave`, src/game/02-rules-economy.js):
```
n(d) = round( (d ≤ 10 ? 3 + 2(d−1) : 21 + 1.7(d−10)) × DIFF.wave )
mix: runners 25% from d3, archers 18% from d4, brutes 15% from d6 (+1 from d10),
     rams 1+floor((d−8)/4) from d8, shamans from d7, sappers from d10,
     WARLORD boss every 5th night (d = 5, 10, 15, 20)
```
**Enemy HP** (`enemyStats`): `h(d) = 2 + floor(0.75d)`, then × type factor × DIFF.hp
runner 0.55h · archer 0.75h · raider 1h · brute 2.6h · **warlord 5.2h** (speed 0.62, dmg 3, pays 8 gold)
Enemy speed `s(d) = min(1.7, 0.95 + 0.04d)`.

**Difficulty** (`DIFF`): easy wave 0.75 / hp 0.85 / start-res 1.35 · normal 1/1/1 · hard 1.22 / 1.15 / 0.85

**Defense** (`weapon`):
- keep: range 1.8, rate 1.3s, dmg 1 + mods
- tower lvl L: range 2.4 + 0.35(L−1), rate max(0.36, 0.75 − 0.15(L−1)), dmg L + mods; upgrades cost {stone 4, gold 5} → {stone 6, gold 8}
- catapult: range 3.6, rate 2.4s, dmg 3, splash; cost {wood 6, stone 4, gold 4}, upkeep 1 gold/day
- barracks / range: field soldiers, upkeep 1 gold/day each, soldiers eat 1 food

**Building costs** (selection): farm {wood 3} · house {wood 4} · lumber {gold 3} · mine {wood 4, gold 2} · palisade {wood 2} · tower {wood 4, gold 3} · barracks {wood 6, gold 4} (day 2) · market {wood 6, stone 4} (day 3) · archery range {wood 6, gold 5} (day 3) · catapult (day 4) · storehouse {wood 8, stone 4}

**Economy** (`econ`): farm 1×harvest output · lumber 4 wood · mine 3 stone · taxes ≈ 1 gold per working villager × tax/standard multipliers · market +3 gold · storage cap 80 + 60/storehouse (gold is uncapped) · market buy price rises 12% per purchase.

**Upgrades:** 13 one-time upgrades, 6–18 gold each (Irrigation, Iron tools, Granary, Tax collectors, Fletching, Masonry, Barbed spikes, Counterweights, Drill yard, Muster roll, Warhorse, Great horn, Shipwrights).

**Paths** (chosen mid-game): merchant (counting houses → gold), warlord (war camps → army), holy (temples).

## Bot strategies used in the baseline (tools/bot.js)
- **balanced:** holy path, towers ≈ 1 + d/2.5, palisades from day 4, a few catapults and barracks
- **turtle:** warlord path, towers ≈ 2 + d/1.6, palisades from day 2, upgrades defense first
- **economy:** merchant path, few towers early, economy upgrades first
- **random:** places random buildings (a floor for comparison)

## Baseline results: win rate (20 games per cell)

| | balanced | turtle | economy | random |
|---|---|---|---|---|
| easy | 75% | **90%** | 85% | 0% |
| normal | 55% | **75%** | 55% | 0% |
| hard | 30% | **60%** | **15%** | 0% |

## What the data says

1. **Boss nights decide the game.** Of 72 losses by the non-random bots, **55 (76%) happened on night 15 or 20**, the warlord nights. Only 4 happened on night 10. The night-15 and night-20 spike is much sharper than the steady growth in raid size.
2. **Gold piles up and goes unspent.** Median gold at game end: balanced 157, turtle 192, economy 232. The economy bot loses on hard while sitting on 200+ gold, so there's nothing useful to turn late-game gold into.
3. **Towers are the only defense that pays.** All bots stack tower upgrades (turtle ~17 per game). Catapults and archery ranges are rarely built (under 1 per game on average) and barracks only 1–2, probably because of daily upkeep and food costs.
4. **The difficulty spread is narrow at the top and wide at the bottom.** Easy still loses 10–25% of games for decent bots, while the economy path collapses on hard (15%).
5. **The random bot never wins** and dies around night 10–14. Good: building at random isn't rewarded.

**Caveat:** these are bot results. The bots don't use hero abilities well, and they ignore trade routes, the court and the world map. Treat these numbers as relative signals, not absolute ones.

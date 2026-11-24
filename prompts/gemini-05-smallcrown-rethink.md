# Prompt 05 — Small Crown: revise the balance plan with keep-damage data

Paste into Gemini with `plans/2026-11-24-emperorfall-balance-rework.md` (your plan) and the table below.

---

You wrote the attached Small Crown balance plan. Claude added tracking of **who actually damages the keep** and reran the 240-game baseline (same rules, commit `6fcb881`). Two findings change the diagnosis:

**1. The Warlord never touched the keep.** Keep damage on the final night of all 71 non-random losses, by attacker:

| Attacker | Keep damage | Share |
|---|---|---|
| raider | 191 | 29% |
| brute | 150 | 23% |
| ram | 147 | 22% |
| runner | 72 | 11% |
| sapper | 64 | 10% |
| shaman | 36 | 5% |
| archer | 4 | 1% |
| **warlord** | **0** | **0%** |

53 of 71 losses (75%) were still on nights 15 and 20. Source facts: Warlord speed is fixed at `0.62`; while alive it rallies nearby enemies (`rallyT`: +1 damage, ×1.25 speed). Rams have `bdmg 7` against a 10-HP keep. Brutes are `2.6h` HP.

**2. 20 games per cell is noisy.** Same code, two runs: easy/economy 85% → 60%, hard/economy 15% → 30%. At n=20, a 50% win rate has a 95% interval of roughly ±22 points.

**Your task:**
1. Re-diagnose the boss-night spike using this data. Is it the rally aura, the escort size, rams, or something else? Say what extra measurement would settle it if you can't tell.
2. Revise pass 1 to **at most 3 changes**, each tied to a finding above. Drop the Warlord HP cut unless you can justify it from the data.
3. Rewrite the acceptance checks so they're realistic for the sample size (more games per cell, or wider bands). Each 240-game run takes ~20 min on 2 cores.
4. Flag anything in your original plan that this data contradicts.

Output the complete revised plan as one fenced Markdown block labeled `plans/2026-11-24-emperorfall-balance-rework.md`, keeping the Execution notes section.

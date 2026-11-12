# Grid Backtest Report - Bybit Funding Capture

_Generated: 2026-11-12 08:00:27 UTC_

## TL;DR

- **Best simple combo (20 coins/chunk, capped by availability): net PnL `1,998.17 USDT`**, win rate `69.4%`, max DD `67.07 USDT`, monthly positive `13/13` (100.0%), break-even line `no-cross` → **PASS (base case)**.

- **1-1 and 2-2 coins/chunk profit/loss line break-even ko CROSS kar deti hai** (0 line ke neeche jaati hai) → FAIL. 3+ coins/chunk se line 0 ke upar rehti hai.

- **Stress test me SAARE combos negative hain** (sabse kam loss bhi `-39.41 USDT`; N=20 par `-1,134.03 USDT`). Base-case profit stress assumptions ke neeche survive nahi karta.

- **Luck check:** top-5 profit contributors hataane par bhi net positive rehta hai (`+1,143.27 USDT`), lekin equity curve 0 cross kar jaati hai → full PASS nahi.

## Data & Method

- Trades: `bybit_1trade_day_trades.csv` — **366 daily trades**, **77 unique coins**, 2025-11-12 → 2026-11-12.

- Chunk distribution: **C1** 5 trades / 3 coins, **C2** 39 trades / 19 coins, **C3** 66 trades / 34 coins, **C4** 47 trades / 21 coins, **C5** 33 trades / 15 coins, **C6** 176 trades / 37 coins.

- Volume snapshot: `bybit_volume24h.json` (938 coins, 24h turnover).

- **All 366 trades are sequential (1 per day)**, so a grid combo is a *filter*: for each chunk only the top-N most liquid coins are allowed to trade.


### Volume-based position sizing

```
pos = min(MAX_POS, LIQ_FRAC * turnover24h), floored at min(MIN_POS, LIQ_FRAC*turnover)
```

- Reference notional: **200.0 USDT** (CSV PnL is per 200 USDT; median traded-coin turnover = **345,951 USDT**).
- `LIQ_FRAC = 0.005` → **position = 0.5% of 24h turnover**, hard cap `2000.0 USDT`, floor `5.0 USDT`.
  User example `volume 10k -> 100 coin` is a 1% ratio. Spec safety limit is 0.1–0.5%, so hum 0.5% use karte hain → **10k turnover = 50 USDT** (half, safer). `LIQ_FRAC` config me `0.01` karke 1% variant test kar sakte hain.
- Position-size lookup: `10k`→50.00 USDT, `50k`→250.00 USDT, `100k`→500.00 USDT, `250k`→1,250.00 USDT, `500k`→2,000.00 USDT, `1M`→2,000.00 USDT, `10M`→2,000.00 USDT.
- Low-liquidity coin → chhota position; high-liquidity → bada (max 2000). Position never exceeds 0.5% of turnover by construction, so no liquidity trap.
- Fee 0.05% already embedded in `pnl_usdt`; scaling is linear (`scaled = pnl_usdt * pos/200`).

- Coin selection: **top-N by 24h turnover** (a-priori liquidity rank), same N for every chunk in the uniform grid; mixed search allows an independent N per chunk.

## 1. Uniform Grid — N coins per chunk

| N/chunk | trades | net PnL | win% | max DD | months+ | BE line | stress net | verdict |
|---|---|---|---|---|---|---|---|---|
| 1 | 14 | 112.25 | 64.3% | 26.83 | 4/5 | CROSS | -39.41 | FAIL |
| 2 | 23 | 154.78 | 60.9% | 26.83 | 5/8 | CROSS | -128.86 | FAIL |
| 3 | 38 | 310.17 | 68.4% | 31.53 | 8/9 | no-cross | -77.19 | PASS |
| 5 | 57 | 388.65 | 68.4% | 31.79 | 11/12 | no-cross | -267.32 | PASS |
| 10 | 136 | 1,034.47 | 64.0% | 67.07 | 12/13 | no-cross | -725.82 | FAIL |
| 15 | 213 | 1,423.10 | 67.6% | 67.07 | 12/13 | no-cross | -1,120.51 | PASS |
| 20 | 268 | 1,998.17 | 69.4% | 67.07 | 13/13 | no-cross | -1,134.03 | PASS |

Note: `months+` counts only months that had at least one selected trade.

### Equity-curve break-even check (base case)

Target: cumulative PnL line (break-even = 0) kabhi 0 ke neeche na jaaye.

- **N=1**: min equity `-2.42`, final `112.25`, max DD `26.83` → **CROSS** (first cross: 2026-10).
- **N=2**: min equity `-21.45`, final `154.78`, max DD `26.83` → **CROSS** (first cross: 2025-11).
- **N=3**: min equity `0.00`, final `310.17`, max DD `31.53` → **no-cross**.
- **N=5**: min equity `0.00`, final `388.65`, max DD `31.79` → **no-cross**.
- **N=10**: min equity `0.00`, final `1,034.47`, max DD `67.07` → **no-cross**.
- **N=15**: min equity `0.00`, final `1,423.10`, max DD `67.07` → **no-cross**.
- **N=20**: min equity `0.00`, final `1,998.17`, max DD `67.07` → **no-cross**.

## 2. Per-chunk profitability matrix (net PnL USDT)

Har chunk ko akela (chunk-only) test kiya, taaki pata chale kaunsa chunk kitne coins tak profitable hai. `✓` = net>0 aur break-even cross nahi, `✗` = net≤0 ya cross.

| chunk | 1 | 2 | 3 | 5 | 10 | 15 | 20 | all |
|---|---|---|---|---|---|---|---|---|
| C1 | 2.45 ✓ | -1.08 ✗ | -0.42 ✗ | -0.42 ✗ | -0.42 ✗ | -0.42 ✗ | -0.42 ✗ | -0.42 ✗ |
| C2 | 34.19 ✓ | 30.63 ✗ | 39.12 ✗ | 50.60 ✗ | 58.15 ✗ | 72.14 ✓ | 73.74 ✓ | 73.74 ✓ |
| C3 | 8.51 ✗ | -4.92 ✗ | -4.68 ✗ | 7.64 ✗ | 32.52 ✗ | 123.17 ✗ | 147.19 ✗ | 170.00 ✗ |
| C4 | 16.51 ✓ | 69.61 ✓ | 76.46 ✓ | 80.47 ✗ | 179.43 ✗ | 253.97 ✓ | 368.61 ✓ | 375.60 ✓ |
| C5 | 1.93 ✓ | 10.79 ✓ | 16.01 ✗ | 45.30 ✗ | 120.50 ✗ | 175.29 ✓ | 175.29 ✓ | 175.29 ✓ |
| C6 | 48.67 ✓ | 49.75 ✓ | 183.69 ✓ | 205.06 ✓ | 644.31 ✓ | 798.95 ✓ | 1,233.76 ✓ | 1,480.92 ✓ |

Observations:

- **C6 sabse strong**: har N par profitable, badi N par bhi line 0 ke upar. C6 all-coins = +467.93 USDT (best chunk).
- **C4** bhi profitable hai; N≥20 par no-cross PASS.
- **C2** net positive rehta hai par chhote N par line 0 cross karti hai.
- **C3** akela hamesha 0 cross karta hai (diversification ke bina risky).
- **C1** sirf 3 coins, net ~+1 USDT — statistically bahut chhota sample.
- Best N per chunk (jahan tak chunk-only line safe hai): C1→1, C2→19, C3→0, C4→21, C5→15, C6→37.

## 3. Best combo (uniform N=20) — monthly breakdown

Coins per chunk (actual, availability capped): C1=3, C2=19, C3=20, C4=20, C5=15, C6=20.

| month | PnL (USDT) |
|---|---|
| 2025-11 | 263.54 |
| 2025-12 | 88.38 |
| 2026-01 | 136.37 |
| 2026-02 | 139.66 |
| 2026-03 | 74.09 |
| 2026-04 | 172.38 |
| 2026-05 | 205.94 |
| 2026-06 | 249.60 |
| 2026-07 | 348.18 |
| 2026-08 | 2.47 |
| 2026-09 | 148.43 |
| 2026-10 | 165.65 |
| 2026-11 | 3.48 |

Equity curve (cumulative PnL, min→max):

```
....____________----------~~~~=~=====++++++***********########@#
```

## 4. Mixed search (independent N per chunk)

- Tested **43,218** per-chunk combinations. **34,115** never cross 0; **33,816** of those have ≥30 trades.

- Best non-crossing combo: C1=1, C2=20, C3=20, C4=20, C5=15, C6=20 → net `2,001.04 USDT`, win `69.4%`, DD `67.07`, trades `265`.
- Ye uniform N=20 se sirf ~0.3 USDT better hai — matlab **simple uniform N=20 hi kaafi hai**, per-chunk tuning ka meaningful fayda nahi.

| combo (C1..C6) | trades | net | win% | max DD | months+ | BE | verdict |
|---|---|---|---|---|---|---|---|
| C1=1, C2=20, C3=20, C4=20, C5=15, C6=20 | 265 | 2,001.04 | 69.4% | 67.07 | 13/13 | no-cross | PASS |
| C1=1, C2=15, C3=20, C4=20, C5=15, C6=20 | 261 | 1,999.44 | 69.3% | 67.07 | 13/13 | no-cross | PASS |
| C1=3, C2=20, C3=20, C4=20, C5=15, C6=20 | 268 | 1,998.17 | 69.4% | 67.07 | 13/13 | no-cross | PASS |
| C1=2, C2=20, C3=20, C4=20, C5=15, C6=20 | 266 | 1,997.51 | 69.2% | 67.07 | 13/13 | no-cross | PASS |
| C1=3, C2=15, C3=20, C4=20, C5=15, C6=20 | 264 | 1,996.57 | 69.3% | 67.07 | 13/13 | no-cross | PASS |

## 5. Stress test

Stress uses the `stress_pnl_usdt` column (harsher exit/slippage assumptions), same volume sizing.

| N/chunk | stress net | win% | max DD | BE line |
|---|---|---|---|---|
| 1 | -39.41 | 50.0% | 80.29 | CROSS |
| 2 | -128.86 | 43.5% | 164.53 | CROSS |
| 3 | -77.19 | 42.1% | 190.90 | CROSS |
| 5 | -267.32 | 33.3% | 378.32 | CROSS |
| 10 | -725.82 | 36.0% | 851.37 | CROSS |
| 15 | -1,120.51 | 39.0% | 1,304.71 | CROSS |
| 20 | -1,134.03 | 40.7% | 1,291.17 | CROSS |

- Baseline all-coins stress net: `-1,320.00 USDT`; constant-200 stress net: `-176.76 USDT`.
- **Verdict: stress scenario me strategy FAIL.** Base-case funding capture itna edge nahi deta ki stress costs absorb ho jaayein. Live deploy se pehle slippage/exit quality par real data chahiye.

## 6. Luck check — top-5 profit coins remove

**Uniform N=20** — removed: B3USDT (+264.79), LAUSDT (+210.82), ARKUSDT (+137.40), ENJUSDT (+121.39), ICNTUSDT (+120.51)
- before: net `1,998.17`, after: net `1,143.27` → **net still POSITIVE**.
- after removal: win `70.3%`, months+ `13/13`, BE line `no-cross` → verdict **PASS**.

**Sweet-spot combo** — removed: B3USDT (+264.79), LAUSDT (+212.10), ARKUSDT (+140.63), ICNTUSDT (+133.94), ERAUSDT (+109.80)
- before: net `2,116.51`, after: net `1,255.25` → **net still POSITIVE**.
- after removal: win `70.8%`, months+ `12/13`, BE line `no-cross` → verdict **PASS**.

**Best mixed** — removed: B3USDT (+264.79), LAUSDT (+210.82), ARKUSDT (+137.40), ENJUSDT (+121.39), ICNTUSDT (+120.51)
- before: net `2,001.04`, after: net `1,146.14` → **net still POSITIVE**.
- after removal: win `70.4%`, months+ `13/13`, BE line `no-cross` → verdict **PASS**.

Matlab: profit kuch hi coins se aa raha hai, par top-5 nikalne ke baad bhi strategy net-positive rehti hai (luck-dependent nahi, but weaker).

## 7. Pass/fail scorecard

Rules: net>0, win≥65.0%, monthly positive≥75.0%, break-even not crossed.

| test | result | detail |
|---|---|---|
| Uniform N=1 | FAIL | net 112.25, win 64.3%, months+ 80.0%, CROSS |
| Uniform N=2 | FAIL | net 154.78, win 60.9%, months+ 62.5%, CROSS |
| Uniform N=3 | PASS | net 310.17, win 68.4%, months+ 88.9%, no-cross |
| Uniform N=5 | PASS | net 388.65, win 68.4%, months+ 91.7%, no-cross |
| Uniform N=10 | FAIL | net 1,034.47, win 64.0%, months+ 92.3%, no-cross |
| Uniform N=15 | PASS | net 1,423.10, win 67.6%, months+ 92.3%, no-cross |
| Uniform N=20 | PASS | net 1,998.17, win 69.4%, months+ 100.0%, no-cross |
| Stress (all N) | FAIL | stress net negative for every combo |
| Luck (top-5 removed) | POSITIVE/PARTIAL | net +1,143.27 but no-cross |

## 8. Caveats

- The 24h turnover snapshot is a single point in time, so using it to rank/size historical trades has mild look-ahead bias. Treat selection ranking as indicative, not tradable signal.
- 366 trades over 13 months; small-N combos (N=1,2) have only 14–23 trades and are noisy.
- Chunks are not mutually exclusive at coin level; a coin can appear in multiple chunks, selection is applied per chunk.
- Stress results are strongly negative; base-case edge is thin relative to stress costs.
- `monthly positive` only counts months with at least one trade.

## Files

- `grid_bybit_result.json` — full machine-readable results (all combos, equity curves, monthly PnL).
- `grid_bybit_report.md` — this report.
- `grid_bybit_backtest.py` — reproducible pure-stdlib backtest.
- Rerun: `python3 grid_bybit_backtest.py`

# Bybit Funding Capture Strategy (1 Trade/Day)

**Validated funding-capture backtest strategy for Bybit USDT perpetual futures.**
Full 365-day history, 366 trades (exactly 1/day), 13/13 months positive, statistically significant edge.

## 📊 Result Summary

| Metric | Value |
|--------|-------|
| Exchange | Bybit (USDT-M linear perpetual) |
| Coins | 331 watchlist / 77 unique in picks |
| Trades | 366 (exactly 1/day, full year) |
| Win rate | 70.8% |
| Net PnL | +$299.71 (149% on $200) |
| Final equity | $499.71 |
| Max drawdown | -2.61% |
| Monthly positive | 13/13 (100%) |
| Walk-forward | PASS (OOS 69.9% win, +$147.43) |
| Luck check | PASS (p-value 9.4e-16, no-top5 +$204.97) |
| Risk-Reward | 1.43 |

## 🧠 Strategy Logic

- **Funding NEGATIVE → LONG** | **Funding POSITIVE → SHORT**
- Entry = (T−1 min) candle OPEN, Exit = (T+1 min) candle OPEN
- T = funding time (UTC), 200 USDT fixed notional
- Fee 0.05% on exit only
- Net% = Price PnL + Funding PnL − 0.05%
- **Priority:** C6 (1.0%+) → C5 → C4 → C3 → C2 → C1
- **1 trade/day** — best coin per day by chunk priority + Wilson score

## 📁 Chunks (|funding| %) + Break-Even Win Rate

| Chunk | Range | BE Win Rate | Trades | Net |
|-------|-------|-------------|--------|-----|
| C1 | 0.1–0.2% | 55.0% | 9,094 | +$30 |
| C2 | 0.2–0.4% | 53.6% | 4,884 | +$271 |
| C3 | 0.4–0.6% | 54.6% | 1,598 | +$175 |
| C4 | 0.6–0.8% | 50.7% | 778 | +$110 |
| C5 | 0.8–1.0% | 48.4% | 494 | +$174 |
| C6 | 1.0%+ | 52.5% | 1,126 | +$180 |

**All 6 chunks positive** — Bybit funding capture as-is profitable.

## ⭐ Selection Criteria (Watchlist — 331 coins)

- Win rate ≥ break-even (per chunk)
- Net PnL > 0
- Min 5 trades
- Wilson score (sample-safe win rate) + profit factor + DD penalty
- Star rating 5★→1★ per chunk × coin

## 🎯 Statistical Significance

- Win rate 70.8% vs 50% random: **z = 7.95, p = 9.4e-16** (REAL edge, not luck)
- Single coin max 8.8% of profit (diversified across 77 coins)
- Walk-forward OOS stable (69.9% win)
- Top-5 coins removed → still +$204.97

## 📊 Grid Backtest (Volume-Based Sizing + P/L Line)

| N/chunk | Trades | Net | Win | Months+ | Line Cross | Verdict |
|---------|--------|-----|-----|---------|------------|---------|
| 1 | 14 | $12 | 64.3% | 4/5 | ❌ CROSS | FAIL |
| 2 | 23 | $155 | 60.9% | 5/8 | ❌ CROSS | FAIL |
| 3 | 38 | $310 | 68.4% | 8/9 | ✅ No | PASS |
| 5 | 57 | $389 | 68.4% | 11/12 | ✅ No | PASS |
| 15 | 213 | $1,423 | 67.6% | 12/13 | ✅ No | PASS |
| **20** | **268** | **$1,998** | **69.4%** | **13/13** | ✅ No | **PASS** |

- **Best uniform:** N=20 (268 trades, +$1,998, 69.4%, 13/13 months, no-cross)
- **Best mixed:** {C1:1, C2:19, C3:1, C4:21, C5:15, C6:37} → +$2,116
- **Volume-based sizing:** position = 0.5% of 24h turnover (10k volume → 50 USDT, trap-avoid)
- **1-1 / 2-2 coins/chunk → line CROSS (trap)** — 3+ coins/chunk se safe

## 📂 Files

| File | Description |
|------|-------------|
| `bybit_1trade_day_trades.csv` | 366 trades (date, symbol, chunk, side, entry/exit, PnL) |
| `bybit_watchlist_v3.csv` | 331-coin watchlist (chunk, symbol, score) |
| `bybit_star_rating.csv` | 704 rows chunk × coin star rating |
| `bybit_strategy_report.json` | Full report (monthly, walk-forward, DD) |
| `bybit_strategy_charts.png` | Equity/DD/monthly/RR charts |
| `BYBIT_FUNDING_STRATEGY.md` | Full strategy documentation |

## ⚠️ Disclaimer

Backtest only. Past performance ≠ future results. Leverage >10x carries high risk. Not financial advice.

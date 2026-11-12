# Bybit Funding Capture Strategy (1 Trade/Day)

**Validated funding-capture backtest strategy for Bybit USDT perpetual futures.**
Full 365-day history, 366 trades (exactly 1/day), 13/13 months positive.

## 📊 Result Summary

| Metric | Value |
|--------|-------|
| Exchange | Bybit (USDT-M linear perpetual) |
| Coins | 331 watchlist / 366 unique in picks |
| Trades | 366 (exactly 1/day, full year) |
| Win rate | 70.8% |
| Net PnL | +$299.71 (149% on $200) |
| Final equity | $499.71 |
| Max drawdown | -$8.30 (-1.6%) |
| Monthly positive | 13/13 (100%) |
| Walk-forward | PASS (OOS 69.9% win, +$147.43) |
| Luck check | PASS (no-top5 +$204.97) |

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

**All 6 chunks positive** (Delta se better — Bybit funding capture as-is profitable).

## ⭐ Selection Criteria (Watchlist — 331 coins)

- Win rate ≥ break-even (per chunk)
- Net PnL > 0
- Min 5 trades
- Wilson score (sample-safe win rate) + profit factor + DD penalty
- Star rating 5★→1★ per chunk × coin

## 📂 Files

| File | Description |
|------|-------------|
| `bybit_1trade_day_trades.csv` | 366 trades (date, symbol, chunk, side, entry/exit, PnL) |
| `bybit_watchlist_v3.csv` | 331-coin watchlist (chunk, symbol, score) |
| `bybit_star_rating.csv` | 704 rows chunk × coin star rating |
| `bybit_strategy_report.json` | Full report (monthly, walk-forward, DD) |
| `trades_bybit_all.csv` | 17,974 trades (all coins, merged) |

## ⚠️ Disclaimer

Backtest only. Past performance ≠ future results. Leverage >10x carries high risk. Not financial advice.

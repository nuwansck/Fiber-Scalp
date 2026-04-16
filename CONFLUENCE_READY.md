# Fiber Scalp v1.3 — Technical Specification & Operations Wiki

**Bot:** Fiber Scalp v1.3  **Pair:** EUR/USD  **Exchange:** OANDA (demo)
**Platform:** Railway (Singapore region)  **Timeframe:** M5  **Cycle:** 5 min

---

## 1. Architecture

```
scheduler.py  (APScheduler — every 5 min)
      |
      ├── run_bot_cycle()
      |       ├── _guard_phase()     — 14 ordered pre-trade checks
      |       ├── _signal_phase()    — EMA + ORB + CPR scoring + position size
      |       └── _execution_phase() — margin check → spread check → place_order
      |
      ├── send_daily_report()   — 04:00 SGT Mon–Fri
      ├── send_weekly_report()  — Monday 08:15 SGT
      ├── send_weekly_export()  — Monday 08:20 SGT (trade_history.json)
      └── send_monthly_report() — First Monday 08:00 SGT
```

---

## 2. Signal Engine

**File:** `signals.py` → `SignalEngine.analyze(instrument="EUR_USD")`

| Component | Bull | Bear | Max |
|---|---|---|---|
| EMA9 fresh cross above EMA21 | +3 | — | |
| EMA9 fresh cross below EMA21 | — | +3 | |
| EMA9 aligned above EMA21 | +1 | — | |
| EMA9 aligned below EMA21 | — | +1 | |
| ORB break fresh (<60 min) | +2 | +2 | |
| ORB break aging (60–120 min) | +1 | +1 | |
| CPR bias aligned | +1 | +1 | |
| Exhaustion (>3× ATR, no ORB) | −1 | −1 | |
| **Maximum** | | | **6/6** |

---

## 3. Session Schedule

All times SGT (UTC+8):

| Session | Window | Threshold | Cap | Notes |
|---|---|---|---|---|
| Dead zone | 04:00–15:59 | No trading | — | Covers pre-Tokyo gap + full Tokyo window. Zero API calls. |
| London | 16:00–20:59 | ≥ 4/6 | 10 | PRIMARY — best EUR/USD session |
| US | 21:00–23:59 | ≥ 4/6 | 10 | Secondary |
| US cont | 00:00–03:59 | ≥ 4/6 | 10 | Secondary |

Dead zone extended to 04:00–15:59 SGT — covers the full Tokyo window.
Zero API calls during inactive hours. Bot wakes up at 16:00 for London open.

---

## 4. Position Sizing

pip_value_usd is **static $10.00** — EUR/USD is USD-quoted, always $10/pip per standard lot.

| Score | Position | Units | Per pip |
|---|---|---|---|
| 4 | $30 partial | ~10,000 | $1.00/pip |
| 5–6 | $60 full | ~20,000 | $2.00/pip |

SL: 18p · TP: 30p · RR: 1.67× · Break-even WR: 37.5%

At $2,000 demo account with 20:1 leverage (5% margin):
```
max_units = (2,000 × 0.6) / (1.08 × 0.05) = 22,222 ✅
```
20,000 units comfortably within margin limit.

---

## 5. Guard Stack (14 checks, ordered)

1. Market closed (Sat/Sun, Mon pre-08:00)
2. Dead zone early exit (04:00–07:59)
3. News block (high-impact event ±30 min)
4. News score penalty (medium event → −1)
5. Loss cooldown (consecutive losses → 30 min pause)
6. Friday cutoff (after 23:00 SGT Friday)
7. Session check (outside active windows)
8. Daily loss cap (8 losses → pause until 08:00 SGT)
9. Session cap (per-window trade limit)
10. Concurrent cap (1 per pair, 2 globally)
11. Margin guard (auto-scale units if insufficient)
12. Min trade units (reject if < 1,000 after margin guard)
13. Spread guard (skip if > 2p London / 3p US)
14. Hard dead zone execution block (final safety net)

---

## 6. H1 Trend Filter

Mode `soft` (current): labels each trade as H1-aligned or counter-trend.
Mode `strict`: blocks counter-trend entries entirely.
Switch to strict once weekly export confirms counter-trend WR is materially lower.

---

## 7. Key Files on Railway Volume (`/data`)

| File | Purpose |
|---|---|
| `settings.json` | Live config (synced from bundle on startup) |
| `trade_history.json` | All trade records |
| `runtime_state.json` | Last cycle status, balance |
| `ops_state_eurusd.json` | EUR/USD session state, caps |
| `score_cache_eurusd.json` | Signal dedup cache |
| `calendar_cache.json` | Forex Factory events |
| `rf_scalp.db` | SQLite cycle + signal log |

---

## 8. Version History

| Version | Date | Key changes |
|---|---|---|
| **v1.0** | **Apr 16 2026** | **Initial release — EUR/USD London M5 scalper. Based on Cable Scalp v1.4 / Ninja Scalp v1.2 architecture (all previous bot references removed). Tokyo disabled. $2/pip full position sizing.** |

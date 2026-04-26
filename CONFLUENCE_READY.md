# Fiber Scalp v1.6 — Technical Specification & Operations Wiki

**Bot:** Fiber Scalp v1.6  **Pair:** EUR/USD  **Exchange:** OANDA (demo)
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

| Component | Bull | Bear | Notes |
|---|---|---|---|
| EMA9 fresh cross above EMA21 | +3 | — | Requires ≥1 pip spread at cross (v1.6) |
| EMA9 fresh cross below EMA21 | — | +3 | Requires ≥1 pip spread at cross (v1.6) |
| EMA9 weak cross (< 1 pip spread) | +1 | +1 | Logged as "EMA Weak Cross Up/Down" |
| EMA9 aligned above EMA21 | +1 | — | No fresh cross this candle |
| EMA9 aligned below EMA21 | — | +1 | No fresh cross this candle |
| ORB break fresh (< 60 min) | +2 | +2 | |
| ORB break aging (60–120 min) | +1 | +1 | |
| ORB stale (> 120 min) | +0 | +0 | |
| CPR bias aligned | +1 | +1 | Price vs daily pivot |
| Exhaustion (> 3× ATR, no ORB) | −1 | −1 | |
| **Maximum** | | | **6/6** |

### EMA Minimum Spread (v1.6)

A crossover only earns the full +3 if `abs(EMA9 − EMA21) >= pip_size (0.0001)` at the candle close. Below this the cross is treated as aligned (+1) and labelled "Weak Cross" in the signal log and Telegram detail. This prevents flat-EMA noise entries from scoring identically to high-conviction crosses. The spread in pips is visible in every signal detail line.

### ORB Age Calculation

ORB age is measured from session open (London 16:00 SGT, US 21:00 SGT). For US continuation trades at 00:00–03:59 SGT the open reference is rolled back to the previous calendar day's 21:00 SGT, giving a correct age of 3–7 hours (stale, +0 pts). Prior to v1.6 the rollback only covered exactly 00:xx, so 01:00–03:59 trades incorrectly scored ORB as fresh (+2).

---

## 3. Session Schedule

All times SGT (UTC+8):

| Session | Window | Threshold | Cap | Notes |
|---|---|---|---|---|
| Dead zone | 04:00–15:59 | No trading | — | Zero API calls. Covers full pre-London period. |
| London | 16:00–20:59 | ≥ 4/6 | 10 | PRIMARY — best EUR/USD session |
| US | 21:00–23:59 | ≥ 4/6 | 10 | Secondary |
| US cont | 00:00–03:59 | ≥ 4/6 | 10 | Secondary — same ORB reference as US (21:00 SGT) |

Dead zone (04:00–15:59 SGT) covers the full pre-London period. Zero API calls while inactive.
Bot wakes at 16:00 SGT for the London open.

---

## 4. Position Sizing

pip_value_usd is **static $10.00** — EUR/USD is USD-quoted, always $10/pip per standard lot.

| Score | Position | Units | Per pip |
|---|---|---|---|
| 4 | $45 partial | ~15,000 | $1.50/pip |
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
2. Dead zone early exit (04:00–15:59 — no open trades)
3. News block (high-impact event ±30 min)
4. News score penalty (medium event → −1)
5. Loss cooldown (consecutive losses → 30 min pause)
6. Friday cutoff (after 23:00 SGT Friday)
7. Session check (outside active windows)
8. Daily loss cap (8 losses → pause until 08:00 SGT)
9. Session cap (per-window trade limit)
10. Concurrent cap (1 per pair, 1 globally)
11. Margin guard (auto-scale units if insufficient)
12. Min trade units (reject if < 1,000 after margin guard)
13. Spread guard (skip if > 2p London / 3p US)
14. Hard dead zone execution block (final safety net)

---

## 6. H1 Trend Filter

Mode `soft` (current): labels each trade as H1-aligned or counter-trend in the trade record and weekly report. No entries blocked.
Mode `strict`: blocks counter-trend entries entirely.

Weekly report automatically shows aligned vs counter-trend split with win rate comparison and a recommendation line. Switch to strict once the split confirms counter-trend WR is materially lower over ≥20 trades per group.

---

## 7. Key Files on Railway Volume (`/data`)

| File | Purpose |
|---|---|
| `settings.json` | Live config (synced from bundle on startup) |
| `trade_history.json` | All trade records |
| `runtime_state.json` | Last cycle status, balance |
| `ops_state_eurusd.json` | EUR/USD session state, caps |
| `score_cache_eurusd.json` | Signal dedup cache |
| `orb_cache.json` | Per-instrument per-session ORB high/low |
| `calendar_cache.json` | Forex Factory events |
| `rf_scalp.db` | SQLite cycle + signal log |

---

## 8. Known Behaviours

**Margin alert fires on every trade** — at $2,000 account size, the bot always requests slightly more units than the margin allows and auto-scales down. This is expected and harmless. All trades execute correctly at the scaled size.

**100% SELL bias during Apr 16–26** — EUR/USD trended from ~1.178 to ~1.165. All EMA crosses fired bearish. BUY path is code-complete but untested on live data during this period.

**ORB contribution currently rare** — most entries are EMA cross + CPR only (score 4). ORB breaks add points when price moves decisively through the opening range before the EMA cross, which is uncommon in EUR/USD's measured London open behaviour.

---

## 9. Version History

| Version | Date | Key changes |
|---|---|---|
| v1.0 | Apr 16 2026 | Initial release — EUR/USD London M5 scalper based on Cable Scalp v1.4 / Ninja Scalp v1.2 |
| v1.1 | Apr 16 2026 | Fix: Tokyo session open alert suppressed when threshold ≥99 |
| v1.2 | Apr 16 2026 | Dead zone extended from 04:00–07:59 to 04:00–15:59 SGT |
| v1.3 | Apr 16 2026 | Fix: max_total_open_trades corrected to 1 |
| v1.4 | Apr 16 2026 | H1 aligned/counter-trend split in weekly and monthly reports |
| v1.5 | Apr 16 2026 | Position sizing: partial $30→$45 ($1.50/pip), aligned with Cable Scalp v1.5 |
| **v1.6** | **Apr 26 2026** | **Bug: weekly report crash (KeyError 'wins' in _session_breakdown). Bug: ORB age wrong for US continuation 01–03 SGT. Bug: pip_size fallback was JPY default (0.01→0.0001). Improvement: EMA fresh cross requires ≥1 pip spread.** |

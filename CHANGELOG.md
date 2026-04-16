# Fiber Scalp — Changelog

---

## v1.0.0 — 2026-04-16

Initial release of **Fiber Scalp v1.2** — EUR/USD (Fiber) London-primary M5 scalping bot.
Built from Ninja Scalp v1.2 / Cable Scalp v1.4 architecture. All previous bot references removed.

### Instrument
EUR/USD only. "Fiber" is the standard informal trading nickname for EUR/USD.

### Strategy
EMA 9/21 crossover + Opening Range Breakout (ORB, time-decayed) + CPR daily pivot bias.
Score 0–6/6. Threshold: ≥4/6 for London and US sessions.

### Session schedule

| Window | SGT | Threshold |
|---|---|---|
| Dead zone | 04:00–07:59 | No trading |
| Tokyo | 08:00–15:59 | **Disabled** (threshold 99) |
| London | 16:00–20:59 | ≥ 4/6 ← PRIMARY |
| US | 21:00–23:59 | ≥ 4/6 ← SECONDARY |
| US cont | 00:00–03:59 | ≥ 4/6 ← SECONDARY |

Tokyo disabled — EUR/USD barely moves during Asian hours. London is where
EUR/USD makes its daily range. The London open ORB is the primary signal.

### Position sizing — $2.00/pip target

| Score | Position | Units | Per pip |
|---|---|---|---|
| 4 | $30 partial | ~10,000 | $1.00/pip |
| 5–6 | $60 full | ~20,000 | $2.00/pip |

EUR/USD pip_value_usd is static $10.00 (USD-quoted pair — no dynamic calculation needed).

### SL / TP

| Pair | SL | TP | RR | Break-even WR |
|---|---|---|---|---|
| EUR/USD | 18p | 30p | 1.67× | 37.5% |

### Spread limits
London: 2p · US: 3p · Tokyo: 2p
EUR/USD London spreads are typically 0.5–1p — spread guard rarely fires.

### Architecture
Full parity with Cable Scalp v1.4 and Ninja Scalp v1.2:
- Health server at `__main__` entry point
- Health handler always returns 200
- Reporting day total fix (pd_trades window)
- Session sentinel guards (99 handling)
- ORB sentinel guards
- No legacy sl_pct/rr_ratio checks
- All 14 guards active

### Key differences from Ninja Scalp v1.2
- Instrument: USD_JPY → EUR_USD
- pip_size: 0.01 → 0.0001
- pip_value_usd: dynamic → static $10.00
- SL: 20p → 18p · TP: 34p → 30p · RR: 1.70× → 1.67×
- BE trigger: 22p → 20p
- position_full_usd: $48 → $60
- Tokyo: disabled (threshold 99) ← EUR/USD specific
- US sessions: enabled (London primary, US secondary)

---

## v1.1.0 — 2026-04-16

### Fix — Session open card firing for disabled Tokyo session

**Problem:**
On startup at 15:23 SGT (inside Tokyo hours), the bot sent a
`EUR_USD Tokyo Window Open` Telegram card even though Tokyo is disabled
(`session_thresholds.Tokyo: 99`). Cosmetic but confusing.

**Fix:**
`_guard_phase()` now checks the session threshold before sending the
session open alert. If `threshold >= 99` the card is suppressed entirely.
Tokyo open card will never fire for Fiber Scalp while Tokyo is disabled.

**File changed:** `bot.py` — `_guard_phase()` session open block

---

## v1.2.0 — 2026-04-16

### Extended dead zone to cover Tokyo hours (04:00–15:59 SGT)

**Why:**
Tokyo was already disabled via `session_thresholds.Tokyo: 99` (score impossible
to reach). But the bot still ran a full cycle every 5 minutes during Tokyo hours
— fetching prices, calculating EMAs, querying OANDA — just to immediately
conclude no trade. Wasteful and produced misleading log lines
(`Session: Tokyo Window`).

**Fix:**
`dead_zone_end_hour` extended from `7` to `15` in settings.json.
The dead zone now covers 04:00–15:59 SGT — the full pre-London inactive period.
Bot exits immediately with zero API calls during these hours.
Wakes up at 16:00 SGT for the London open.

**Startup card:**
Tokyo line removed entirely — dead zone `04:00–15:59` communicates everything.
No redundant "Tokyo disabled" label needed.

**Settings changed:**
- `dead_zone_end_hour`: `7` → `15`

**Files changed:** `settings.json`, `settings.json.example`, `bot.py` (defaults),
`telegram_templates.py` (startup card), all docs.

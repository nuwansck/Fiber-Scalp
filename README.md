# Fiber Scalp v1.6 — EUR/USD M5 Scalping Bot

> **Deployed on Railway · OANDA API · Telegram Alerts**

Automated M5 scalping bot for **EUR/USD (Fiber)** on OANDA.
Strategy: EMA 9/21 crossover + Opening Range Breakout (ORB, time-decayed) + CPR pivot bias. Score 0–6/6.
London session primary — EUR/USD is most liquid and directional during London open.

---

## Strategy Overview

Every 5-minute cycle the signal engine scores three components:

| Component | Points | Condition |
|---|---|---|
| EMA crossover — fresh cross | +3 | EMA9 crosses EMA21 **with ≥1 pip spread at cross** |
| EMA crossover — weak cross | +1 | EMA9 crosses EMA21 but spread < 1 pip (treated as aligned) |
| EMA crossover — aligned | +1 | EMA9 already beyond EMA21, no fresh cross this candle |
| ORB breakout — fresh | +2 | Price beyond session open range, < 60 min old |
| ORB breakout — aging | +1 | ORB break, 60–120 min old |
| CPR bias | +1 | Price above/below daily pivot |
| Exhaustion penalty | −1 | > 3× ATR stretch without ORB |

**London ≥4/6 → trade. US ≥4/6 → trade. Tokyo disabled.**
Score 5–6 → full $60 (~20,000 units, $2.00/pip). Score 4 → partial $45 (~15,000 units, $1.50/pip).

> **EMA minimum spread rule (v1.6):** A fresh cross only earns +3 if `|EMA9 − EMA21| ≥ 1 pip (0.0001)` at the moment of crossing. Sub-pip crosses are logged as "EMA Weak Cross" and score +1, preventing noise entries on nearly-flat EMAs. The pip spread is shown in every signal detail line.

---

## Sessions (SGT = UTC+8)

```
✈️  04:00–15:59  Dead zone       No entries, zero API calls
                                (covers pre-London gap — Tokyo fully inactive)
🇬🇧 16:00–20:59  London          score ≥4/6  cap 10  ← PRIMARY
🗽 21:00–23:59  US              score ≥4/6  cap 10  ← SECONDARY
🗽 00:00–03:59  US cont         score ≥4/6  cap 10  ← SECONDARY
```

Day reset: 08:00 SGT. Global cap: 1 open trade.

---

## Risk & Position Sizing

| | Value |
|---|---|
| Full position | $60 (score 5–6) → ~20,000 units → **$2.00/pip** |
| Partial position | $45 (score 4) → ~15,000 units → **$1.50/pip** |
| EUR/USD SL | 18 pips |
| EUR/USD TP | 30 pips |
| RR | 1.67× |
| Break-even WR | 37.5% |
| Max losing trades/day | 8 |
| pip_value_usd | $10.00 (static — USD-quoted pair) |

**pip_value_usd is static** — EUR/USD is a USD-quoted pair, always $10.00/pip per standard lot.

---

## Railway Deployment

1. Push folder to GitHub
2. New Railway project → Singapore region → Deploy from GitHub
3. Add persistent volume at `/data`
4. Set environment variables (see below)
5. Deploy

### Environment Variables

| Variable | Required |
|---|---|
| `OANDA_API_KEY` | ✅ |
| `OANDA_ACCOUNT_ID` | ✅ |
| `OANDA_DEMO` | ✅ (`true` for practice) |
| `TELEGRAM_BOT_TOKEN` | ✅ |
| `TELEGRAM_CHAT_ID` | ✅ |

---

## File Structure

```
scheduler.py          — APScheduler entry point
bot.py                — Trade cycle: guard → signal → execute
signals.py            — EMA + ORB + CPR engine, static EUR/USD pip value
oanda_trader.py       — OANDA REST API wrapper
telegram_templates.py — All Telegram message cards
telegram_alert.py     — Telegram sender
reporting.py          — Daily / weekly / monthly reports
config_loader.py      — Settings loading + defaults
database.py           — SQLite cycle + signal logging
reconcile_state.py    — Startup trade reconciliation
news_filter.py        — Forex Factory news block/penalty
calendar_fetcher.py   — Calendar data fetcher
settings.json         — All configuration (source of truth)
version.py            — Version string
```

---

## Version History

| Version | Date | Summary |
|---|---|---|
| v1.0 | Apr 16 2026 | Initial release — EUR/USD London M5 scalper |
| v1.1 | Apr 16 2026 | Fix: Tokyo session open alert suppressed when disabled |
| v1.2 | Apr 16 2026 | Dead zone extended to 04:00–15:59 SGT (full Tokyo window) |
| v1.3 | Apr 16 2026 | Fix: max_total_open_trades corrected to 1 |
| v1.4 | Apr 16 2026 | H1 filter split added to weekly/monthly reports |
| v1.5 | Apr 16 2026 | Position sizing: partial $30 → $45 ($1.50/pip) |
| **v1.6** | **Apr 26 2026** | **Bug fixes: weekly report crash, ORB age (US cont window), pip fallback. EMA minimum spread filter (≥1 pip for fresh cross +3).** |

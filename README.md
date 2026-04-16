# Fiber Scalp v1.3 — EUR/USD M5 Scalping Bot

> **Deployed on Railway · OANDA API · Telegram Alerts**

Automated M5 scalping bot for **EUR/USD (Fiber)** on OANDA.
Strategy: EMA 9/21 crossover + Opening Range Breakout (ORB, time-decayed) + CPR pivot bias. Score 0–6/6.
London session primary — EUR/USD is most liquid and directional during London open.

---

## Strategy Overview

Every 5-minute cycle the signal engine scores three components:

| Component | Points | Condition |
|---|---|---|
| EMA crossover | +3 fresh / +1 aligned | EMA9 vs EMA21 on M5 |
| ORB breakout | +2 fresh (<60min) / +1 aging (60–120min) | Price beyond session open range |
| CPR bias | +1 | Price above/below daily pivot |
| Exhaustion penalty | −1 | >3× ATR stretch without ORB |

**London ≥4/6 → trade. US ≥4/6 → trade. Tokyo disabled.**
Score 5–6 → full $60. Score 4 → partial $30.

---

## Sessions (SGT = UTC+8)

```
✈️  04:00–15:59  Dead zone       No entries, zero API calls
                                (covers pre-Tokyo gap + full Tokyo window)
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
| Partial position | $30 (score 4) → ~10,000 units → **$1.00/pip** |
| EUR/USD SL | 18p |
| EUR/USD TP | 30p |
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

# Fiber Scalp v1.8 — EUR/USD M5 Scalping Bot

**Fiber Scalp v1.8** is the calmer-risk release. The strategy logic is unchanged, but live risk sizing is now more controlled.

## What changed in v1.8

- Score-based risk sizing added:
  - Score 4/6 → **$25 risk**
  - Score 5/6 → **$35 risk**
  - Score 6/6 → **$40 max risk**
- Hard unit cap added: `max_units = 20000`
- Existing EUR/USD structure retained: **SL 18 pips / TP 30 pips / RR 1.67**
- Telegram startup template updated to show the new risk ladder
- v1.7 Telegram noise reduction retained: WATCHING alerts below score 4 are suppressed

## Strategy

| Component | Value |
|---|---:|
| Pair | EUR/USD |
| Timeframe | M5 |
| Signal score threshold | 4/6 |
| SL | 18 pips |
| TP | 30 pips |
| Reward:risk | 1.67 |
| Telegram signal alerts | score ≥ 4 only |
| Trade cycle | every 5 minutes |

## Risk profile

| Score | Risk USD | Approx units with 18-pip SL | Use case |
|---:|---:|---:|---|
| 4/6 | $25 | ~13,888 units | Valid but normal setup |
| 5/6 | $35 | ~19,444 units | Stronger setup |
| 6/6 | $40 | capped by `max_units` | Best setup only |

`max_units = 20000` prevents the bot from requesting oversized positions and reduces margin-adjustment alerts.

## Key settings

```json
"score_risk_usd": {
  "4": 25,
  "5": 35,
  "6": 40
},
"max_units": 20000,
"pair_sl_tp": {
  "EUR_USD": {
    "sl_pips": 18,
    "tp_pips": 30,
    "pip_value_usd": 10.0,
    "be_trigger_pips": 20
  }
},
"telegram_min_score_alert": 4
```

## Deployment

1. Upload the project to Railway.
2. Keep `demo_mode = true` until enough demo trades are collected.
3. Confirm OANDA and Telegram environment variables are present.
4. Deploy and check the startup Telegram message.

## Version history

| Version | Date | Notes |
|---|---|---|
| **v1.8** | **May 02 2026** | **Calmer score-based risk sizing: $25/$35/$40, hard `max_units=20000`, updated docs and Telegram startup template.** |
| **v1.7** | **May 02 2026** | Telegram noise reduction: WATCHING alerts below score 4 suppressed by default. |
| v1.6 | Apr 2026 | Fixed-pip EUR/USD SL/TP and margin guard improvements. |

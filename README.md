# Fiber Scalp v2.0 — EUR/USD M5 Scalping Bot

**Fiber Scalp v2.0** adds the new **score-aware H1 filter** on top of the v1.9 controlled-risk model.

## What changed in v2.0

- Added score-aware H1 trade gating:
  - Score 4/6 + H1 aligned → **allow**
  - Score 4/6 + H1 neutral or opposite → **skip**
  - Score 5/6 or 6/6 + H1 aligned → **allow**
  - Score 5/6 or 6/6 + H1 neutral → **allow**
  - Score 5/6 or 6/6 + H1 opposite → **skip**
- Telegram signal and startup templates now show the H1 score-aware policy.
- Existing risk ladder retained: **$30 / $40 / $50** for scores **4 / 5 / 6**.
- Hard unit cap retained: `max_units = 20000`.
- EUR/USD structure retained: **SL 18 pips / TP 30 pips / RR 1.67**.
- Telegram noise reduction retained: WATCHING alerts below score 4 are suppressed.

## Strategy

| Component | Value |
|---|---:|
| Pair | EUR/USD |
| Timeframe | M5 |
| Signal score threshold | 4/6 |
| H1 filter mode | `score_aware` |
| SL | 18 pips |
| TP | 30 pips |
| Reward:risk | 1.67 |
| Telegram signal alerts | score ≥ 4 only |
| Trade cycle | every 5 minutes |

## H1 score-aware rules

| Score | H1 aligned | H1 neutral / unknown / flat | H1 opposite |
|---:|---|---|---|
| 4/6 | Trade allowed | Skipped | Skipped |
| 5/6 | Trade allowed | Trade allowed | Skipped |
| 6/6 | Trade allowed | Trade allowed | Skipped |

This makes score 4 stricter because it is the weakest tradeable score. Stronger score 5/6 setups can still trade during neutral H1, but the bot avoids clear counter-trend H1 trades.

## Risk profile

| Score | Risk USD | Approx units with 18-pip SL | Use case |
|---:|---:|---:|---|
| 4/6 | $30 | ~16,667 units | Valid setup, H1 alignment required |
| 5/6 | $40 | capped by `max_units` | Stronger setup, neutral H1 allowed |
| 6/6 | $50 | capped by `max_units` | Best setup only, neutral H1 allowed |

`max_units = 20000` prevents the bot from requesting oversized positions and reduces margin-adjustment alerts.

## Key settings

```json
"score_risk_usd": {
  "4": 30,
  "5": 40,
  "6": 50
},
"max_units": 20000,
"h1_filter_enabled": true,
"h1_filter_mode": "score_aware",
"h1_ema_period": 21,
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
5. Confirm the startup Telegram message shows `H1 filter: SCORE-AWARE`.

## Version history

| Version | Date | Notes |
|---|---|---|
| **v2.0** | **May 02 2026** | **Added score-aware H1 gating and updated Telegram/docs.** |
| v1.9 | May 02 2026 | Updated score-based risk sizing: $30/$40/$50, hard `max_units=20000`, updated docs and Telegram startup template. |
| v1.7 | May 02 2026 | Telegram noise reduction: WATCHING alerts below score 4 suppressed by default. |
| v1.6 | Apr 2026 | Fixed-pip EUR/USD SL/TP and margin guard improvements. |

# Fiber Scalp v1.8 — Settings Reference

## Core risk settings

| Setting | v1.8 value | Meaning |
|---|---:|---|
| `signal_threshold` | `4` | Minimum score required for execution |
| `score_risk_usd.4` | `25` | Risk amount for score 4/6 trades |
| `score_risk_usd.5` | `35` | Risk amount for score 5/6 trades |
| `score_risk_usd.6` | `40` | Risk amount for score 6/6 trades |
| `max_units` | `20000` | Hard cap on final requested units |
| `min_trade_units` | `1000` | Reject very small adjusted orders |
| `margin_safety_factor` | `0.6` | Uses only part of available margin |
| `auto_scale_on_margin_reject` | `true` | Auto-reduce units if broker margin rejects |

Legacy fields are still present for compatibility:

| Legacy setting | v1.8 fallback |
|---|---:|
| `position_partial_usd` | `25` |
| `position_full_usd` | `35` |

The bot prefers `score_risk_usd` when present.

## SL / TP

| Setting | Value | Meaning |
|---|---:|---|
| `pair_sl_tp.EUR_USD.sl_pips` | `18` | Stop loss distance |
| `pair_sl_tp.EUR_USD.tp_pips` | `30` | Take profit distance |
| `min_rr_ratio` | `1.6` | Minimum allowed reward:risk |
| `be_trigger_pips` | `20` | Break-even trigger reference; break-even is currently disabled by default |

## Telegram alerts

| Setting | Value | Meaning |
|---|---:|---|
| `telegram_min_score_alert` | `4` | Suppress score 0–3 WATCHING alerts |
| `telegram_show_margin` | `true` | Notify when margin guard adjusts units |

Telegram should send actionable items only: score 4+ signals, opened/closed trades, spread/news blocks, order failures, and scheduled reports.

## Recommended v1.8 JSON block

```json
{
  "signal_threshold": 4,
  "score_risk_usd": {
    "4": 25,
    "5": 35,
    "6": 40
  },
  "position_partial_usd": 25,
  "position_full_usd": 35,
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
}
```

## Why v1.8 changed risk sizing

Previous sizing could request around 25k–33k units, then rely on the margin guard to scale down. v1.8 makes the first requested size calmer and more realistic for a small demo account, while keeping the same SL/TP structure that produced a positive early profit factor.

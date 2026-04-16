# Fiber Scalp v1.1 — Settings Reference

All configuration lives in `settings.json` on the Railway volume (`/data`).

---

## Identity

| Key | Value |
|---|---|
| `bot_name` | `"Fiber Scalp v1.1"` |
| `demo_mode` | `true` |

---

## Pair

```json
"pairs": {
  "EUR_USD": {
    "enabled": true,
    "pip_size": 0.0001,
    "max_spread_pips": 2,
    "spread_limits": { "Tokyo": 2, "London": 2, "US": 3 },
    "max_concurrent_trades": 1
  }
}
```

---

## SL / TP

| Key | Value | Notes |
|---|---|---|
| `pair_sl_tp.EUR_USD.sl_pips` | `18` | Stop loss |
| `pair_sl_tp.EUR_USD.tp_pips` | `30` | Take profit |
| `pair_sl_tp.EUR_USD.pip_value_usd` | `10.0` | Static — USD-quoted pair |
| `pair_sl_tp.EUR_USD.be_trigger_pips` | `20` | Break-even trigger (if enabled) |
| `min_rr_ratio` | `1.6` | |
| `breakeven_enabled` | `false` | Enable once data confirms |

---

## Position Sizing

| Key | Value | Notes |
|---|---|---|
| `position_full_usd` | `60` | Score 5–6 → ~20,000 units → **$2.00/pip** |
| `position_partial_usd` | `30` | Score 4 → ~10,000 units → **$1.00/pip** |
| `min_trade_units` | `1000` | |

---

## Sessions (SGT = UTC+8)

| Key | Value | Notes |
|---|---|---|
| `tokyo_session_start_hour` | `8` | |
| `tokyo_session_end_hour` | `15` | |
| `london_session_start_hour` | `16` | 16:00 |
| `london_session_end_hour` | `20` | 20:59 |
| `us_session_start_hour` | `21` | 21:00 |
| `us_session_end_hour` | `23` | 23:59 |
| `us_session_early_end_hour` | `3` | US cont 00–03 |
| `dead_zone_start_hour` | `4` | |
| `dead_zone_end_hour` | `7` | |

```json
"session_thresholds": {
  "Tokyo":  99,
  "London": 4,
  "US":     4
}
```

**Tokyo threshold 99 = effectively disabled.**
To enable Tokyo: change to 5 or 6.

---

## Risk Controls

| Key | Default |
|---|---|
| `max_total_open_trades` | `2` |
| `max_losing_trades_day` | `8` |
| `max_trades_london` | `10` |
| `max_trades_us` | `10` |
| `loss_streak_cooldown_min` | `30` |
| `sl_reentry_gap_min` | `10` |

---

## H1 Filter

| Key | Default |
|---|---|
| `h1_filter_enabled` | `true` |
| `h1_filter_mode` | `"soft"` |

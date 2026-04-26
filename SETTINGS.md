# Fiber Scalp v1.6 — Settings Reference

All configuration lives in `settings.json` on the Railway volume (`/data`).

---

## Identity

| Key | Value |
|---|---|
| `bot_name` | `"Fiber Scalp v1.5"` |
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
| `pair_sl_tp.EUR_USD.pip_value_usd` | `10.0` | Static — USD-quoted pair. Do not remove — fallback uses dynamic calc. |
| `pair_sl_tp.EUR_USD.be_trigger_pips` | `20` | Break-even trigger (if enabled) |
| `min_rr_ratio` | `1.6` | |
| `breakeven_enabled` | `false` | Enable once data confirms |

---

## Signal Engine

| Key | Default | Notes |
|---|---|---|
| `signal_threshold` | `4` | Minimum score to trade |
| `ema_fast_period` | `9` | EMA fast period |
| `ema_slow_period` | `21` | EMA slow period |
| `orb_formation_minutes` | `15` | Minutes after session open before ORB locks |
| `orb_fresh_minutes` | `60` | ORB age below which break scores +2 |
| `orb_aging_minutes` | `120` | ORB age above which break scores +0 (stale) |
| `exhaustion_atr_mult` | `3.0` | ATR multiplier for exhaustion penalty |
| `atr_period` | `14` | ATR lookback |
| `m5_candle_count` | `40` | M5 candles fetched per cycle |

EMA minimum spread (v1.6): fresh cross (+3) requires abs(EMA9 - EMA21) >= pip_size at cross time. Sub-pip crosses score +1 (treated as aligned). This is derived from pip_size in settings, not a separate setting.

---

## Position Sizing

| Key | Value | Notes |
|---|---|---|
| `position_full_usd` | `60` | Score 5-6 → ~20,000 units → $2.00/pip |
| `position_partial_usd` | `45` | Score 4 → ~15,000 units → $1.50/pip |
| `min_trade_units` | `1000` | |

---

## Sessions (SGT = UTC+8)

| Key | Value | Notes |
|---|---|---|
| `london_session_start_hour` | `16` | 16:00 SGT |
| `london_session_end_hour` | `20` | 20:59 SGT |
| `us_session_start_hour` | `21` | 21:00 SGT |
| `us_session_end_hour` | `23` | 23:59 SGT |
| `us_session_early_end_hour` | `3` | US continuation 00:00-03:59 SGT |
| `dead_zone_start_hour` | `4` | Dead zone starts |
| `dead_zone_end_hour` | `15` | Dead zone ends — covers full pre-London period |
| `tokyo_session_start_hour` | `8` | Defined but Tokyo is disabled |
| `tokyo_session_end_hour` | `15` | Defined but Tokyo is disabled |

session_thresholds: Tokyo=99 (disabled), London=4, US=4.

Tokyo is disabled via session_thresholds.Tokyo: 99 and dead_zone_end_hour: 15. To enable Tokyo: set dead_zone_end_hour: 7 and session_thresholds.Tokyo: 5.

---

## Risk Controls

| Key | Default |
|---|---|
| `max_total_open_trades` | `1` |
| `max_losing_trades_day` | `8` |
| `max_trades_london` | `10` |
| `max_trades_us` | `10` |
| `max_losing_trades_session` | `4` |
| `loss_streak_cooldown_min` | `30` |
| `sl_reentry_gap_min` | `10` |

---

## H1 Filter

| Key | Default | Notes |
|---|---|---|
| `h1_filter_enabled` | `true` | Labels trades as aligned/counter-trend |
| `h1_filter_mode` | `"soft"` | soft = label only. strict = block counter-trend entries |
| `h1_ema_period` | `21` | H1 EMA period |

Weekly and monthly reports automatically show the aligned vs counter-trend win-rate split.

---

## News Filter

| Key | Default |
|---|---|
| `news_filter_enabled` | `true` |
| `news_block_before_min` | `30` |
| `news_block_after_min` | `30` |
| `news_lookahead_min` | `120` |
| `news_medium_penalty_score` | `-1` |

---

## Reporting Schedule

| Key | Default | Notes |
|---|---|---|
| `daily_report_hour_sgt` | `4` | 04:00 SGT — dead zone start, full day captured |
| `daily_report_minute_sgt` | `0` | |
| `weekly_report_hour_sgt` | `8` | Every Monday 08:15 SGT |
| `weekly_report_minute_sgt` | `15` | |
| `monthly_report_hour_sgt` | `8` | First Monday 08:00 SGT |
| `monthly_report_minute_sgt` | `0` | |

---

## Margin

| Key | Default | Notes |
|---|---|---|
| `margin_safety_factor` | `0.6` | Use 60% of free margin for sizing |
| `margin_retry_safety_factor` | `0.4` | Retry at 40% if first attempt rejected |
| `auto_scale_on_margin_reject` | `true` | Auto-scale down rather than skip |
| `telegram_show_margin` | `true` | Send alert when units are scaled |

---

## Database

| Key | Default |
|---|---|
| `db_retention_days` | `90` |
| `db_cleanup_hour_sgt` | `0` |
| `db_cleanup_minute_sgt` | `15` |
| `db_vacuum_weekly` | `true` |

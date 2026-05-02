# Fiber Scalp v1.8 — Technical Specification & Operations Wiki

**Bot:** Fiber Scalp v1.8  
**Pair:** EUR/USD  
**Broker:** OANDA  
**Mode:** Demo by default  
**Primary release goal:** calmer risk sizing without changing the core strategy.

## 1. Executive Summary

Fiber Scalp v1.8 keeps the existing M5 EMA + ORB + CPR signal engine and the EUR/USD 18-pip SL / 30-pip TP model. The main improvement is risk control: score 4, 5, and 6 signals now have separate risk amounts, and the bot has a hard `max_units` cap.

## 2. Architecture

```text
Railway Scheduler
  └── every 5 minutes
      └── bot.run_cycle()
          ├── load settings
          ├── fetch OANDA candles/prices
          ├── calculate EMA / ORB / CPR score
          ├── apply session, spread, news, open-trade and cooldown guards
          ├── calculate score-based risk
          ├── apply margin guard + max_units cap
          ├── place OANDA order when all checks pass
          └── send Telegram + persist DB state
```

## 3. Strategy Settings

| Item | Value |
|---|---:|
| Timeframe | M5 |
| Minimum execution score | 4/6 |
| Stop loss | 18 pips |
| Take profit | 30 pips |
| Reward:risk | 1.67 |
| Telegram watching alert minimum | 4/6 |

## 4. v1.8 Risk Model

| Score | Risk amount | Approx units with 18-pip SL | Notes |
|---:|---:|---:|---|
| 4/6 | $25 | ~13,888 | Normal valid setup |
| 5/6 | $35 | ~19,444 | Higher-quality setup |
| 6/6 | $40 | capped by `max_units` | Best setup only |

Hard cap:

```json
"max_units": 20000
```

This avoids repeated oversized requests and reduces dependency on margin auto-scaling.

## 5. Telegram Alert Policy

Telegram alerts are intentionally filtered:

| Event | Telegram? |
|---|---|
| Score 0–3 watching setup | No |
| Score 4+ signal | Yes |
| Trade opened | Yes |
| Trade closed | Yes |
| News block / spread block | Yes |
| Margin adjustment | Yes, if enabled |
| Startup / daily / weekly / monthly reports | Yes |

Startup template now displays:

```text
Risk: $25 (score 4) | $35 (score 5) | $40 (score 6)
```

## 6. Operational Notes

- Keep `demo_mode = true` until at least 50–100 closed trades are reviewed.
- Review score 4 vs score 5/6 performance separately.
- Do not increase risk until margin-adjustment alerts become rare and profit factor remains stable.
- Keep `telegram_min_score_alert = 4` to avoid low-score alert noise.

## 7. Version History

| Version | Date | Change |
|---|---|---|
| v1.8 | May 02 2026 | Added score-based risk sizing, `max_units=20000`, and updated Telegram/docs. |
| v1.7 | May 02 2026 | Suppressed Telegram WATCHING alerts below score 4. |
| v1.6 | Apr 2026 | Fixed-pip SL/TP and margin guard release. |

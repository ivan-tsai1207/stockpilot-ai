# API Contract Draft

## POST /api/analyze-stock

### Request

```json
{
  "symbol": "2408",
  "market": "TWSE",
  "timeframe": "short_term",
  "user_position": {
    "has_position": false,
    "cost": null,
    "shares": null
  },
  "preferences": {
    "avoid_chasing_high": true,
    "prefer_pullback_entry": true,
    "allow_batch_entry": true
  }
}
```

### Response

```json
{
  "symbol": "2408",
  "name": "南亞科",
  "generated_at": "2026-06-28T22:00:00+08:00",
  "data_freshness": {
    "price_updated_at": "",
    "chip_updated_at": "",
    "news_updated_at": ""
  },
  "market_context": {},
  "news_analysis": {},
  "fundamental_analysis": {},
  "technical_analysis": {},
  "chip_analysis": {},
  "strategy": {
    "decision": "watch",
    "entry_zone": [],
    "stop_loss": null,
    "take_profit": [],
    "position_plan": ""
  },
  "scenario_analysis": {
    "bullish": "",
    "neutral": "",
    "bearish": ""
  },
  "risk_notes": [],
  "one_sentence_conclusion": "可觀察但不急追"
}
```

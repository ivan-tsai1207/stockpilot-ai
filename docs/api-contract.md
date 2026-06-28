# API Contract Draft

## Design principles

- Every stock conversation is an independent session.
- Strategy settings are reusable and versioned.
- LLM outputs must be traceable to a session and a strategy profile.
- The API should support both synchronous analysis and persistent memory updates.

## POST /api/strategy-profiles

Create a user strategy profile.

### Request

```json
{
  "name": "短線拉回買",
  "risk_tolerance": "medium",
  "holding_period": "swing",
  "avoid_chasing_high": true,
  "prefer_pullback_entry": true,
  "allow_batch_entry": true,
  "max_position_size": 0.2,
  "stop_loss_rule": "跌破月線且量增時重新評估",
  "take_profit_rule": "分批停利，保留趨勢倉",
  "preferred_indicators": ["ma", "volume", "macd", "kd"],
  "strategy_prompt_overrides": "偏好拉回承接，不追高"
}
```

## POST /api/stock-sessions

Create an isolated stock analysis session.

### Request

```json
{
  "symbol": "2408",
  "market": "TWSE",
  "timeframe": "short_term",
  "session_type": "stock_research_session",
  "strategy_profile_id": "sp_001"
}
```

### Response

```json
{
  "session_id": "sas_2408_short_term_001",
  "symbol": "2408",
  "market": "TWSE",
  "timeframe": "short_term",
  "session_type": "stock_research_session",
  "strategy_profile_id": "sp_001",
  "summary_version": 1,
  "created_at": "2026-06-28T22:00:00+08:00"
}
```

## POST /api/stock-sessions/{session_id}/messages

Send a user message into a stock-specific session and trigger LLM analysis.

### Request

```json
{
  "message": "2408 南亞科現在適合布局嗎？請幫我看未來趨勢和當下風險。",
  "mode": "analysis",
  "refresh_external_data": true
}
```

### Response

```json
{
  "session_id": "sas_2408_short_term_001",
  "message_id": "msg_001",
  "analysis_snapshot_id": "snap_001",
  "assistant_response": {
    "current_status": "短線偏強，但不適合追價",
    "confirmed_facts": [],
    "inferences": [],
    "hypotheses": [],
    "support_levels": [],
    "resistance_levels": [],
    "risk_notes": [],
    "scenario_analysis": {
      "bullish": "",
      "neutral": "",
      "bearish": ""
    },
    "strategy": {
      "decision": "watch",
      "entry_zone": [],
      "stop_loss": null,
      "take_profit": [],
      "position_plan": ""
    },
    "one_sentence_conclusion": "可觀察但不急追"
  },
  "memory_updated": true,
  "data_freshness": {
    "price_updated_at": "",
    "chip_updated_at": "",
    "news_updated_at": ""
  }
}
```

## GET /api/stock-sessions/{session_id}/memory

Fetch the structured session memory used for future prompts.

### Response

```json
{
  "session_id": "sas_2408_short_term_001",
  "summary_text": "",
  "confirmed_facts": [],
  "inferences": [],
  "hypotheses": [],
  "risk_notes": [],
  "watch_points": [],
  "entry_plan": "",
  "updated_at": "2026-06-28T22:00:00+08:00"
}
```

## POST /api/analyze-stock

This endpoint supports direct one-shot analysis while still binding to an isolated session.

### Request

```json
{
  "symbol": "2408",
  "market": "TWSE",
  "timeframe": "short_term",
  "session_id": "sas_2408_short_term_001",
  "strategy_profile_id": "sp_001",
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
  "session_id": "sas_2408_short_term_001",
  "symbol": "2408",
  "name": "南亞科",
  "generated_at": "2026-06-28T22:00:00+08:00",
  "strategy_profile_id": "sp_001",
  "strategy_profile_version": 3,
  "prompt_template_version": 2,
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

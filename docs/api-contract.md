# API Contract Draft

## Design principles

- Every stock conversation is an independent session.
- Strategy settings are reusable and versioned.
- Analysis presets are reusable and independent from strategy profiles.
- Knowledge graph memory is the primary retrieval layer.
- LLM outputs must be traceable to a session and a strategy profile.
- The API should support both synchronous analysis and persistent memory updates.

## GET /api/markets/indices

Fetch Taiwan and US market index snapshots.

### Response

```json
{
  "tw_market": {
    "weighted_index": {},
    "otc_index": {},
    "turnover_value": 0,
    "advance_decline": {
      "up": 0,
      "down": 0,
      "flat": 0
    },
    "sector_performance": []
  },
  "us_market": {
    "dow_jones": {},
    "sp500": {},
    "nasdaq": {},
    "sox": {},
    "impact_to_tw": ""
  },
  "captured_at": "2026-06-28T22:00:00+08:00"
}
```

## GET /api/stocks/{symbol}/candles?interval=1m|5m|15m|1h|1d|1w|1mo

Fetch candlestick data for a stock.

### Response

```json
{
  "symbol": "2408",
  "market": "TWSE",
  "interval": "5m",
  "candles": [
    {
      "time": "2026-06-28T09:05:00+08:00",
      "open": 70.5,
      "high": 71.2,
      "low": 70.1,
      "close": 70.9,
      "volume": 18234,
      "last_price": 70.9
    }
  ]
}
```

## GET /api/stocks/{symbol}/indicators

Fetch categorized technical indicators.

### Response

```json
{
  "symbol": "2408",
  "interval": "1d",
  "trend": {},
  "momentum": {},
  "volume": {},
  "volatility": {},
  "trend_strength": {}
}
```

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

## POST /api/analysis-presets

Create a reusable analysis preset.

### Request

```json
{
  "name": "盤中快速判斷",
  "analysis_mode": "intraday",
  "objective": "快速判斷個股盤中是否可追、可等、可減碼",
  "required_data_fields": [
    "price",
    "intraday_candles",
    "volume",
    "market_indices",
    "technical_indicators",
    "recent_news"
  ],
  "default_output_template": {
    "sections": [
      "目前狀態",
      "支撐壓力",
      "風險",
      "Bull/Base/Bear",
      "一句話結論"
    ]
  },
  "default_strategy_profile_id": "sp_001"
}
```

## GET /api/analysis-presets

List reusable presets available to the user.

### Response

```json
{
  "items": [
    {
      "id": "ap_001",
      "name": "盤中快速判斷",
      "analysis_mode": "intraday",
      "is_system_preset": true,
      "version": 1
    }
  ]
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
  "analysis_preset_id": "ap_001",
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
  "analysis_preset_id": "ap_001",
  "strategy_profile_id": "sp_001",
  "summary_version": 1,
  "created_at": "2026-06-28T22:00:00+08:00"
}
```

## POST /api/stock-sessions/{session_id}/run-preset

Run the selected analysis preset without requiring freeform prompting every time.

### Request

```json
{
  "analysis_preset_id": "ap_001",
  "refresh_external_data": true,
  "custom_focus": "請特別注意爆量後續強度與大盤是否拖累"
}
```

### Response

```json
{
  "session_id": "sas_2408_short_term_001",
  "analysis_snapshot_id": "snap_002",
  "preset_run_id": "spr_002",
  "assistant_response": {},
  "knowledge_graph_updated": true
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

## GET /api/knowledge-graph/nodes

Fetch knowledge graph nodes scoped to the current user and optional stock session.

### Response

```json
{
  "items": [
    {
      "id": "kn_001",
      "node_type": "stock",
      "title": "2408 南亞科",
      "content": "記憶體題材，近期受報價與市場輪動影響",
      "tags": ["memory", "swing"]
    }
  ]
}
```

## GET /api/knowledge-graph/relations

Fetch graph relations between notes and entities.

### Response

```json
{
  "items": [
    {
      "id": "kr_001",
      "from_node_id": "kn_001",
      "to_node_id": "kn_002",
      "relation_type": "related_to",
      "weight": 0.86
    }
  ]
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
  "analysis_preset_id": "ap_001",
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
  "analysis_preset_id": "ap_001",
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

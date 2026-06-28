# 股票分析系統 SDD（System Design Document）

## 1. 文件目的

本文件定義「StockPilot AI 股票分析系統」的系統設計方向、模組架構、資料流、分析邏輯、輸出格式與風險控管規則。

系統目標不是預測一定漲跌，而是建立一套可查證、可解釋、可重複使用的台股分析流程，協助使用者判斷個股是否適合觀察、進場、分批、停損或停利。

---

## 2. 產品定位

### 2.1 系統角色

系統應扮演一位：

- 理性務實的股票市場分析師
- 具備長期投資、波段操作、產業研究、技術分析、籌碼分析與風險控管能力
- 像老練操盤手，但不盲目喊多或喊空

### 2.2 系統不可做的事

- 不得保證獲利
- 不得憑空猜測
- 不得用過時資料做強結論
- 不得在資料不足時硬編判斷
- 不得只給買賣結論而不說依據
- 不得忽略停損與風險控管

---

## 3. 使用者交易偏好

系統需符合以下使用者偏好：

- 不喜歡追高
- 重視低成本進場
- 希望找拉回買點，而不是衝高後追價
- 希望知道「站穩」「守住」「跌破」「假突破」如何判斷
- 偏好分批買進，不一次滿倉
- 可接受短線、超短線、波段，但需要明確區分
- 目標是逐步累積獲利，不追求一次重壓
- 希望系統教會使用者看盤，而不是只給答案

---

## 4. 分析總流程

每次分析股票時，系統必須依序執行：

1. 大盤與國際市場分析
2. 產業環境分析
3. 個股消息面分析
4. 個股基本面分析
5. 個股技術面分析
6. 個股籌碼面分析
7. 進場與操作策略
8. 風險控管
9. 一句話結論

所有重要判斷必須使用：

> 證據 → 推論 → 策略 → 風險

---

## 5. 系統模組設計

### 5.1 Data Ingestion Module：資料抓取模組

#### 職責

負責取得所有分析所需的原始資料。

#### 資料類型

- 即時股價
- 歷史 K 線
- 成交量
- 均線
- MACD
- KD
- RSI
- 法人買賣超
- 融資融券
- 主力券商分點
- 公司營收
- EPS
- 毛利率
- 營益率
- 新聞
- 法說會與重大公告
- 大盤指數
- 台指期夜盤
- 美股主要指數
- 匯率、利率與國際消息

#### 輸出

標準化後的 StockData、MarketData、ChipData、NewsData、FinancialData。

---

## 5.2 Market Context Module：大盤與產業分析模組

### 分析內容

- 台股整體趨勢
- 大盤是否多頭、震盪、轉弱
- 台指期夜盤方向
- 美股與國際市場影響
- 產業目前是強勢、轉強、整理或退燒
- 資金是否流入該族群

### 輸出

- market_status：bullish / neutral / bearish
- sector_status：strong / improving / consolidating / cooling
- market_risk_level：low / medium / high

---

## 5.3 News & Catalyst Module：消息面模組

### 分析內容

- 近期新聞
- 公司公告
- 法說會
- 接單、擴產、合作、政策利多
- 利空事件
- 題材是否已反映在股價上
- 為什麼該股近期變熱門

### 判斷重點

若股價大漲，系統要回答：

- 是消息帶動？
- 是產業題材？
- 是籌碼推升？
- 是技術突破？
- 還是純投機短炒？

---

## 5.4 Fundamental Analysis Module：基本面模組

### 分析內容

- 公司主要業務
- 營收趨勢
- EPS
- 毛利率
- 營益率
- 獲利成長或衰退
- 產業地位
- 長期成長題材
- 長期投資價值

### 注意事項

基本面佳不等於短線可追高。系統必須區分：

- 長期價值
- 短線位階
- 當下進場風險

---

## 5.5 Technical Analysis Module：技術面模組

### 分析內容

- 股價目前位階
- 均線排列
- 支撐與壓力
- K 線型態
- 成交量變化
- MACD
- KD
- RSI
- 是否過熱
- 是否突破
- 是否假突破
- 是否適合追高、等拉回或觀望

### 教學型輸出要求

系統不能只說結論，必須解釋技術訊號。

範例：

- 站上月線：短中期趨勢可能轉強
- 跌破季線：中期趨勢可能轉弱
- 爆量長紅：買盤積極，但需觀察隔日是否續強
- KD 高檔鈍化：強勢股可能續漲，但過熱風險提高
- MACD 黃金交叉：短期動能可能轉強
- 量縮整理：賣壓可能減輕，但仍需突破確認

---

## 5.6 Chip Analysis Module：籌碼面模組

### 分析內容

- 外資買賣超
- 投信買賣超
- 自營商買賣超
- 融資增減
- 融券增減
- 主力券商分點
- 籌碼集中或分散
- 法人是否連續買進或轉賣

### 注意事項

系統不得假設「外資一定會回來」。必須用資料判斷法人是否有回補跡象。

---

## 5.7 Strategy Engine：策略判斷模組

### 輸入

- 大盤狀態
- 產業狀態
- 消息面強度
- 基本面趨勢
- 技術面訊號
- 籌碼面訊號
- 使用者偏好

### 輸出類型

- 可觀察但不急追
- 適合拉回分批
- 突破確認後再進
- 風險偏高，先等整理
- 技術轉弱，不適合進場

---

## 6. LLM 分析架構設計

### 6.1 設計目標

系統將串接 LLM API，讓每一檔股票都擁有自己的分析對話、自己的記憶、自己的策略輸出，而不是把所有股票都塞進同一個長對話。

此設計的核心原則：

- 每檔股票的 session 必須獨立
- 不同股票之間預設不可互相污染上下文
- 共用的是系統規則與使用者策略，不共用的是個股對話內容
- 要降低 token 消耗，不能把完整歷史對話一直送進模型
- 要支援多使用者，每位使用者都能有自己的投資偏好與策略模板

### 6.2 LLM 在系統中的角色

LLM 不是資料來源本身，而是：

- 負責整理資料
- 負責根據規則做推論
- 負責輸出可讀分析
- 負責依照使用者策略產生不同風格的建議

系統不可把 LLM 當成可以憑空預測未來的黑盒。所有結論仍需依照：

> 證據 → 推論 → 策略 → 風險

### 6.3 對未來趨勢的定義

系統可做「未來趨勢研判」，但不得輸出保證式預測。

未來趨勢輸出必須改用情境化方式表達：

- Bull Scenario：樂觀情境
- Base Scenario：中性情境
- Bear Scenario：保守情境

每個情境都必須附上：

- 成立條件
- 失效條件
- 需觀察指標
- 對應操作策略

---

## 7. Session 與記憶體設計

### 7.1 Session 隔離原則

系統中的對話單位不是「使用者總聊天視窗」，而是：

`User + Strategy Profile + Symbol + Market + Timeframe`

代表：

- 同一位使用者分析 2308 與 2408，必須是兩個不同 session
- 同一檔股票若切換短線與波段，也可以是不同 session
- 同一位使用者若切換不同策略模板，也應產生不同的分析上下文

### 7.2 Session 類型

建議至少區分三種 session：

- `stock_research_session`：個股研究對話
- `stock_monitor_session`：持股追蹤與後續觀察
- `intraday_session`：盤中短時效分析

### 7.3 為何不能全部存在同一個 session

若所有股票共用同一個 session，會產生：

- token 持續膨脹
- 不同股票的訊號互相干擾
- 先前對話裡的假設污染新分析
- 無法針對不同股票建立獨立研究脈絡
- 難以做多使用者資料隔離

因此系統不得把全站對話記錄直接作為單一 prompt 歷史。

### 7.4 記憶體設計原則

系統應採用「結構化記憶 + 摘要記憶」，而非保留完整原文對話。

每個股票 session 應保存：

- 研究主題
- 最新分析摘要
- 已確認事實
- 推論與假設
- 候選進場區間
- 風險事件
- 使用者關注點
- 尚未解答的問題
- 最近一次策略輸出

### 7.5 記憶分層

建議分成四層：

1. `Raw Conversation Log`
   保留原始對話，供稽核與除錯，但不預設回灌給模型。

2. `Session Summary Memory`
   由系統定期整理成 300 至 800 tokens 的摘要，作為後續對話基礎。

3. `Structured Fact Memory`
   使用欄位化資料儲存已確認事實，例如：
   - 最新財報結論
   - 支撐壓力區
   - 法人連買天數
   - 最近重大新聞

4. `Strategy Output Snapshot`
   每次模型給出的結論與策略建議都應保存快照，方便比較前後變化。

### 7.6 記憶更新時機

以下情況應重新產生 session summary：

- 對話超過指定輪數
- 有重大新聞事件
- 有新的技術面轉折
- 使用者切換投資策略
- 收盤後需要整理當日結論

### 7.7 Token 控制策略

為避免 session 成本過高，系統應採用：

- 僅回灌最新對話片段
- 加入結構化摘要而非完整歷史
- 僅在需要時附加相關新聞與籌碼資料
- 過期資料不自動帶入 prompt
- 使用時間窗與資料新鮮度控制上下文大小

---

## 8. 策略設定系統設計

### 8.1 設計目標

系統不是只給單一使用者使用，因此策略不能寫死在 prompt 裡。必須把使用者的投資偏好做成可設定、可版本化、可套用的策略設定系統。

### 8.2 Strategy Profile 概念

每位使用者可以有一個或多個 `Strategy Profile`。

範例：

- 保守波段
- 短線拉回買
- 成長股趨勢追蹤
- ETF 配置型

每個 profile 會影響：

- 模型輸出的語氣與建議方式
- 可接受的風險等級
- 是否接受追高
- 停損停利寬度
- 偏好的觀察週期
- 是否允許分批進場
- 是否重視基本面大於技術面

### 8.3 策略設定欄位

建議至少包含以下欄位：

- `profile_name`
- `risk_tolerance`
- `holding_period`
- `avoid_chasing_high`
- `prefer_pullback_entry`
- `allow_batch_entry`
- `max_position_size`
- `stop_loss_rule`
- `take_profit_rule`
- `preferred_indicators`
- `strategy_prompt_overrides`
- `notification_preferences`

### 8.4 系統策略與使用者策略分離

系統內部應區分兩層規則：

- `System Rulebook`
  所有使用者共用，定義不可違反的分析與風控原則。

- `User Strategy Profile`
  每位使用者可調整，決定輸出偏好與操作風格。

這代表：

- 使用者可以偏好追高，但系統仍必須提示追高風險
- 使用者可以接受高波動，但系統仍不得省略停損控管
- 使用者可以偏好短線，但系統仍需標示資料時效性

### 8.5 策略版本控管

Strategy Profile 應支援版本化。

原因：

- 使用者策略可能調整
- 歷史分析應能追溯當時使用哪個策略版本
- 不能讓舊分析被新策略覆寫後失真

每份分析結果應記錄：

- `strategy_profile_id`
- `strategy_profile_version`
- `prompt_template_version`

---

## 9. 多使用者與資料隔離設計

### 9.1 多租戶原則

系統必須視為多使用者平台，而非單人工具。

因此所有資料都必須綁定：

- `user_id`
- `workspace_id` 或 `tenant_id`
- `strategy_profile_id`
- `session_id`

### 9.2 隔離範圍

以下內容不得跨使用者共用：

- session 對話內容
- session 摘要記憶
- 個人投資策略
- 持股資料
- 自訂停損停利規則

可共用但需標準化的只有：

- 市場資料
- 新聞資料
- 公開財報資料
- 系統規則書

---

## 10. LLM Orchestration 設計

### 10.1 核心流程

每次使用者發起個股分析時，系統應執行：

1. 讀取使用者策略設定
2. 讀取該股票對應的獨立 session summary
3. 拉取最新市場資料與新聞資料
4. 判斷資料新鮮度是否足夠
5. 組裝 prompt
6. 呼叫具聯網能力或外部 research tool 的 LLM 流程
7. 產出分析結果
8. 寫回結構化記憶與分析快照

### 10.2 Prompt 組成

每次送入模型的內容應由以下區塊組成：

- System Rulebook
- Strategy Profile
- Current Market Context
- Stock-specific Session Summary
- Fresh Data Bundle
- User Query

其中 `Stock-specific Session Summary` 不得替換成全站對話歷史。

### 10.3 聯網調查流程

若系統允許模型聯網調查，應限制其調查目的：

- 查最新新聞
- 查公司公告
- 查法人與產業相關資料
- 查近期事件是否造成股價異常

聯網結果必須回寫成：

- 事實摘要
- 來源時間
- 來源類型
- 是否已驗證

不可僅依賴模型口語描述而不保存來源資訊。

### 10.4 分析輸出格式

每次模型輸出建議至少包含：

- 目前狀態
- 關鍵事實
- 推論
- 支撐與壓力
- 風險
- Bull / Base / Bear 情境
- 策略建議
- 一句話結論

並且必須明確區分：

- 已確認事實
- 推論
- 假設

### 10.5 不同股票之間的邏輯共用方式

不同股票之間應共用：

- 分析流程
- 規則書
- prompt 模板
- 策略引擎邏輯

不同股票之間不得共用：

- 對話內容
- session 記憶
- 個股未驗證假設
- 個股的短線交易判斷

---

## 11. 建議資料模型

### 11.1 StrategyProfile

- `id`
- `user_id`
- `name`
- `risk_tolerance`
- `holding_period`
- `avoid_chasing_high`
- `prefer_pullback_entry`
- `allow_batch_entry`
- `stop_loss_rule`
- `take_profit_rule`
- `max_position_size`
- `preferred_indicators`
- `prompt_overrides`
- `version`
- `created_at`
- `updated_at`

### 11.2 StockAnalysisSession

- `id`
- `user_id`
- `strategy_profile_id`
- `symbol`
- `market`
- `timeframe`
- `session_type`
- `title`
- `status`
- `last_analyzed_at`
- `summary_version`
- `created_at`
- `updated_at`

### 11.3 SessionMemory

- `session_id`
- `summary_text`
- `confirmed_facts`
- `inferences`
- `hypotheses`
- `risk_notes`
- `watch_points`
- `entry_plan`
- `updated_at`

### 11.4 AnalysisSnapshot

- `id`
- `session_id`
- `strategy_profile_version`
- `prompt_template_version`
- `data_freshness`
- `market_context`
- `news_analysis`
- `fundamental_analysis`
- `technical_analysis`
- `chip_analysis`
- `scenario_analysis`
- `decision`
- `created_at`

---

## 12. 實作建議

### 12.1 第一階段不要做的事

以下功能不建議在第一版就做太滿：

- 自動交易
- 過度複雜的多代理協作
- 把所有聊天都做成長記憶
- 未經驗證的超長期價格預測

### 12.2 MVP 優先順序

建議先做：

1. Strategy Profile 設定
2. 個股獨立 session
3. session summary 記憶
4. 單檔股票分析 API
5. 分析快照保存
6. 盤中與收盤後兩種分析模式

### 12.3 成功條件

此架構若設計正確，應達成：

- 同一使用者可同時追蹤多檔股票，但互不污染
- 不同使用者可有不同投資策略
- LLM 成本可被控制
- 分析結果可追溯
- 策略調整後仍可回看舊版本分析

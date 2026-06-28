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

## 4.1 功能需求重構總覽

本系統的需求重構後，應以使用者可感知的產品能力來描述，而不是只以內部模組切分。

四個主域如下：

1. 市場與看盤資料
2. LLM 分析工作流
3. 策略與知識圖譜記憶
4. 投資追蹤與提醒

### 4.1.1 市場與看盤資料

需提供：

- 台股大盤與美股大盤作為一級資訊
- 個股多週期 K 線
- 每根 K 線完整 OHLCV 資料
- 分層技術指標
- 類股、題材與市場廣度資訊

### 4.1.2 LLM 分析工作流

需提供：

- 使用者主動建立個股分析 session
- 透過股票選擇 + 分析模組啟動分析
- 可保存與重用的 Analysis Preset
- 不同股票之間完全隔離的分析上下文

### 4.1.3 策略與知識圖譜記憶

需提供：

- 使用者可設定多組 Strategy Profile
- Analysis Preset 與 Strategy Profile 分離
- 記憶層以 Obsidian 風格內建知識圖譜為核心
- LLM 檢索優先使用圖譜節點、關聯與結構化 facts

### 4.1.4 投資追蹤與提醒

需提供：

- Watchlist 與 Portfolio 監控
- 條件提醒與重分析
- 純分析與監控，不納入自動下單或券商委託執行

---

## 5. 系統模組設計

### 5.1 Data Ingestion Module：資料抓取模組

#### 職責

負責取得所有分析所需的原始資料。

#### 資料類型

- 即時股價
- 歷史 K 線
- 分鐘級 K 線（1分、5分、15分、1小時）
- 成交量
- K 線 OHLCV 明細
- 趨勢類技術指標
- 動能類技術指標
- 成交量類技術指標
- 波動類技術指標
- 多空判斷類技術指標
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
- 台股市場廣度
- 類股表現
- 台指期夜盤
- 美股主要指數
- 美股指數對台股影響摘要
- 匯率、利率與國際消息

#### 輸出

標準化後的 StockData、MarketData、ChipData、NewsData、FinancialData。

---

## 5.2 Market Context Module：大盤與產業分析模組

### 分析內容

- 台股整體趨勢
- 加權指數
- 櫃買指數
- 成交值
- 漲跌家數
- 類股表現
- 大盤是否多頭、震盪、轉弱
- 台指期夜盤方向
- 美股與國際市場影響
- 道瓊工業指數
- S&P 500
- NASDAQ
- 費城半導體指數
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

- K 線週期切換：1分、5分、15分、1小時、日、週、月、年
- 每根 K 線明細：開盤價、最高價、最低價、收盤價、成交量、當前價
- 股價目前位階
- 均線排列
- 支撐與壓力
- K 線型態
- 成交量變化
- 趨勢類指標：MA、EMA、WMA、VWAP、Bollinger Bands、SAR
- 動能類指標：MACD、RSI、KD/Stochastic、CCI、ROC、Momentum
- 成交量類指標：Volume MA、OBV、MFI、Volume Ratio
- 波動類指標：ATR、Historical Volatility
- 多空判斷類指標：DMI/ADX
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

系統不應再以 session 摘要作為主記憶設計中心，而應改為「結構化記憶 + 知識圖譜記憶 + 最小必要對話片段」。

每個股票 session 應保存：

- 研究主題
- 已確認事實
- 推論與假設
- 候選進場區間
- 風險事件
- 使用者關注點
- 尚未解答的問題
- 最近一次策略輸出
- 對應圖譜節點與關聯

### 7.5 記憶分層

建議分成四層：

1. `Raw Conversation Log`
   保留原始對話，供稽核與除錯，但不預設回灌給模型。

2. `Knowledge Graph Memory`
   以 Obsidian 風格內建圖譜為核心，使用 note / entity / link / tag / backlink 來保存脈絡。

3. `Structured Fact Memory`
   使用欄位化資料儲存已確認事實，例如：
   - 最新財報結論
   - 支撐壓力區
   - 法人連買天數
   - 最近重大新聞

4. `Strategy Output Snapshot + Minimal Context Window`
   每次模型給出的結論與策略建議都應保存快照，方便比較前後變化。
   對話回灌時只附加最新必要片段，而非完整摘要歷史。

### 7.5.1 知識圖譜節點類型

至少需支援：

- `Stock Note`
- `Theme Note`
- `Strategy Note`
- `Event Note`
- `Risk Note`
- `User Insight Note`

### 7.5.2 知識圖譜關聯類型

至少需支援：

- `related_to`
- `supports`
- `contradicts`
- `triggered_by`
- `watch_for`
- `derived_from`

### 7.6 記憶更新時機

以下情況應更新圖譜節點、結構化 facts 或最小上下文：

- 對話超過指定輪數
- 有重大新聞事件
- 有新的技術面轉折
- 使用者切換投資策略
- 收盤後需要整理當日結論

### 7.7 Token 控制策略

為避免 session 成本過高，系統應採用：

- 僅回灌最新對話片段
- 優先取用圖譜節點與結構化 facts
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

### 8.6 Analysis Preset

除 Strategy Profile 外，系統還應支援可重用的 `Analysis Preset`。

Analysis Preset 用來定義：

- 本次要問 LLM 的目的
- 需要哪些資料欄位
- 預設分析輸出格式
- 預設套用哪一類策略

至少需內建以下 preset：

- `盤中快速判斷`
- `波段趨勢分析`
- `基本面研究`
- `技術面結構檢查`
- `風險檢查`
- `自訂模板`

Analysis Preset 可跨 session 重用，但 session 本身仍維持股票隔離。

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
2. 讀取該股票對應的獨立 session 記憶與知識圖譜關聯
3. 拉取最新市場資料與新聞資料
4. 判斷資料新鮮度是否足夠
5. 讀取 Analysis Preset
6. 組裝 prompt
7. 呼叫具聯網能力或外部 research tool 的 LLM 流程
8. 產出分析結果
9. 寫回知識圖譜、結構化記憶與分析快照

### 10.2 Prompt 組成

每次送入模型的內容應由以下區塊組成：

- System Rulebook
- Strategy Profile
- Analysis Preset
- Current Market Context
- Stock-specific Knowledge Graph Context
- Fresh Data Bundle
- User Query

其中 `Stock-specific Knowledge Graph Context` 不得替換成全站對話歷史。

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

### 11.1.1 AnalysisPreset

- `id`
- `user_id`
- `name`
- `analysis_mode`
- `objective`
- `required_data_fields`
- `default_output_template`
- `default_strategy_profile_id`
- `is_system_preset`
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
- `confirmed_facts`
- `inferences`
- `hypotheses`
- `risk_notes`
- `watch_points`
- `entry_plan`
- `updated_at`

### 11.3.1 KnowledgeNode

- `id`
- `user_id`
- `session_id`
- `node_type`
- `title`
- `content`
- `tags`
- `source_refs`
- `created_at`
- `updated_at`

### 11.3.2 KnowledgeRelation

- `id`
- `from_node_id`
- `to_node_id`
- `relation_type`
- `weight`
- `created_at`

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
2. Analysis Preset 設定
3. 個股獨立 session
4. 知識圖譜記憶
5. 單檔股票分析 API
6. 分析快照保存
7. 盤中與收盤後兩種分析模式

### 12.3 成功條件

此架構若設計正確，應達成：

- 同一使用者可同時追蹤多檔股票，但互不污染
- 不同使用者可有不同投資策略
- LLM 成本可被控制
- 分析結果可追溯
- 策略調整後仍可回看舊版本分析

---

## 13. 資料表 Schema 細化

### 13.1 User

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 使用者主鍵 |
| email | varchar | 登入帳號 |
| display_name | varchar | 顯示名稱 |
| status | varchar | active / suspended |
| default_strategy_profile_id | uuid | 預設策略 |
| created_at | timestamptz | 建立時間 |
| updated_at | timestamptz | 更新時間 |

### 13.2 StrategyProfile

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 策略主鍵 |
| user_id | uuid | 所屬使用者 |
| name | varchar | 策略名稱 |
| description | text | 策略說明 |
| risk_tolerance | varchar | low / medium / high |
| holding_period | varchar | intraday / short_term / swing / long_term |
| avoid_chasing_high | boolean | 是否避免追高 |
| prefer_pullback_entry | boolean | 是否偏好拉回買 |
| allow_batch_entry | boolean | 是否允許分批 |
| max_position_size | numeric | 單一標的最大資金占比 |
| stop_loss_rule | text | 停損規則 |
| take_profit_rule | text | 停利規則 |
| preferred_indicators | jsonb | 偏好指標 |
| prompt_overrides | text | 額外提示 |
| is_default | boolean | 是否預設策略 |
| version | integer | 策略版本 |
| archived_at | timestamptz | 封存時間 |
| created_at | timestamptz | 建立時間 |
| updated_at | timestamptz | 更新時間 |

### 13.2.1 AnalysisPreset

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | preset 主鍵 |
| user_id | uuid | 所屬使用者，可為空代表系統預設 |
| name | varchar | preset 名稱 |
| analysis_mode | varchar | intraday / swing / fundamental / technical / risk |
| objective | text | 分析目的 |
| required_data_fields | jsonb | 必要資料欄位 |
| default_output_template | jsonb | 預設輸出格式 |
| default_strategy_profile_id | uuid | 預設策略 |
| is_system_preset | boolean | 是否系統內建 |
| version | integer | 版本 |
| created_at | timestamptz | 建立時間 |
| updated_at | timestamptz | 更新時間 |

### 13.3 StockAnalysisSession

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | session 主鍵 |
| user_id | uuid | 所屬使用者 |
| strategy_profile_id | uuid | 使用中的策略 |
| symbol | varchar | 股票代號 |
| market | varchar | TWSE / OTC |
| timeframe | varchar | short_term / swing / long_term |
| session_type | varchar | research / monitor / intraday |
| analysis_preset_id | uuid | 使用中的分析模板 |
| title | varchar | session 標題 |
| status | varchar | active / archived |
| last_analyzed_at | timestamptz | 最後分析時間 |
| summary_version | integer | 摘要版本 |
| created_at | timestamptz | 建立時間 |
| updated_at | timestamptz | 更新時間 |

建議 unique key：

- `(user_id, strategy_profile_id, symbol, market, timeframe, session_type, status)`

### 13.4 SessionMessage

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 訊息主鍵 |
| session_id | uuid | 所屬 session |
| role | varchar | user / assistant / system |
| content | text | 訊息內容 |
| token_count | integer | 該訊息 token 粗估 |
| source_type | varchar | manual / generated |
| created_at | timestamptz | 建立時間 |

### 13.5 SessionMemory

| 欄位 | 型別 | 說明 |
|------|------|------|
| session_id | uuid | 對應 session |
| confirmed_facts | jsonb | 已確認事實 |
| inferences | jsonb | 推論 |
| hypotheses | jsonb | 假設 |
| risk_notes | jsonb | 風險重點 |
| watch_points | jsonb | 觀察點 |
| entry_plan | jsonb | 進場規劃 |
| latest_decision | varchar | 最新決策 |
| updated_at | timestamptz | 更新時間 |

### 13.5.1 KnowledgeNode

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 節點主鍵 |
| user_id | uuid | 所屬使用者 |
| session_id | uuid | 關聯 session |
| node_type | varchar | stock / theme / strategy / event / risk / insight |
| title | varchar | 節點標題 |
| content | text | 節點內容 |
| tags | jsonb | 標籤 |
| source_refs | jsonb | 來源參照 |
| created_at | timestamptz | 建立時間 |
| updated_at | timestamptz | 更新時間 |

### 13.5.2 KnowledgeRelation

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 關聯主鍵 |
| from_node_id | uuid | 起點節點 |
| to_node_id | uuid | 終點節點 |
| relation_type | varchar | related_to / supports / contradicts / triggered_by / watch_for / derived_from |
| weight | numeric | 關聯權重 |
| created_at | timestamptz | 建立時間 |

### 13.6 AnalysisSnapshot

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 快照主鍵 |
| session_id | uuid | 所屬 session |
| strategy_profile_version | integer | 策略版本 |
| prompt_template_version | integer | prompt 版本 |
| market_context | jsonb | 大盤與產業 |
| news_analysis | jsonb | 新聞分析 |
| fundamental_analysis | jsonb | 基本面 |
| technical_analysis | jsonb | 技術面 |
| chip_analysis | jsonb | 籌碼面 |
| scenario_analysis | jsonb | bull / base / bear |
| decision | varchar | watch / add / reduce / avoid |
| risk_score | numeric | 風險分數 |
| data_freshness | jsonb | 資料更新時間 |
| model_name | varchar | 使用模型 |
| created_at | timestamptz | 建立時間 |

### 13.7 ExternalResearchSource

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 來源主鍵 |
| snapshot_id | uuid | 對應快照 |
| source_type | varchar | news / filing / web / internal |
| title | varchar | 標題 |
| url | text | 來源連結 |
| published_at | timestamptz | 原始發布時間 |
| fetched_at | timestamptz | 系統抓取時間 |
| summary | text | 來源摘要 |
| is_verified | boolean | 是否通過驗證 |

### 13.8 MarketDataCache

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 快取主鍵 |
| symbol | varchar | 股票代號，可為空代表大盤 |
| market | varchar | 市場別 |
| data_type | varchar | price / chip / news / financial |
| payload | jsonb | 正規化資料 |
| effective_at | timestamptz | 資料生效時間 |
| expires_at | timestamptz | 快取失效時間 |
| created_at | timestamptz | 建立時間 |

### 13.8.1 MarketIndexSnapshot

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 指數快照主鍵 |
| market_type | varchar | tw / us |
| index_code | varchar | twii / tpex / dji / spx / ixic / sox |
| index_name | varchar | 指數名稱 |
| current_value | numeric | 當前點位 |
| change_value | numeric | 漲跌點數 |
| change_percent | numeric | 漲跌幅 |
| breadth_payload | jsonb | 漲跌家數、類股表現等 |
| captured_at | timestamptz | 擷取時間 |

### 13.9 PortfolioHolding

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | uuid | 持股主鍵 |
| user_id | uuid | 所屬使用者 |
| symbol | varchar | 股票代號 |
| market | varchar | 市場別 |
| shares | numeric | 股數 |
| average_cost | numeric | 持股成本 |
| stop_loss_price | numeric | 停損價 |
| take_profit_price | numeric | 停利價 |
| holding_period | varchar | 投資週期 |
| created_at | timestamptz | 建立時間 |
| updated_at | timestamptz | 更新時間 |

---

## 14. Backend Architecture 細化

### 14.1 核心服務拆分

建議至少拆成以下服務模組：

1. `api-gateway`
   提供前端 API、驗證使用者、做 rate limiting。

2. `session-service`
   建立與管理 stock session、讀寫 message、管理 session 狀態。

3. `strategy-service`
   管理使用者策略、版本控管、預設策略套用。

4. `market-data-service`
   抓取並正規化股價、分鐘線/日線 K 線、籌碼、財報、新聞與台美大盤資料。

5. `analysis-orchestrator`
   組裝 prompt、決定何時聯網、呼叫 LLM、寫回快照與記憶。

6. `knowledge-graph-service`
   負責知識圖譜節點、關聯、反向連結、結構化記憶與圖譜檢索。

7. `notification-service`
   根據風險事件與策略偏好發出提醒。

### 14.2 建議系統拓樸

- Frontend Web App
- API Gateway
- PostgreSQL
- Redis
- Object Storage
- Background Worker Queue
- Market Data Connectors
- LLM Provider Adapter

### 14.3 元件責任

- `PostgreSQL`
  存放交易策略、analysis preset、session、message、graph、memory、snapshot、持股與使用者資料。

- `Redis`
  存放短期 session cache、prompt 組裝中間結果、rate limit 與任務鎖。

- `Object Storage`
  存放大型原始 research payload、新聞全文快照、匯出的報告。

- `Background Worker Queue`
  執行摘要更新、收盤後重分析、批次監控與通知。

### 14.4 LLM Provider Adapter

為避免未來綁死單一供應商，系統應抽象出 `LLM Provider Adapter`。

Adapter 至少要統一：

- `generateAnalysis()`
- `retrieveKnowledgeContext()`
- `extractStructuredFacts()`
- `runWebResearch()`

這樣未來可切換不同模型，而不需要重寫 orchestration 主流程。

### 14.5 Prompt 組裝層

Prompt 不應散落在 controller 或前端，而應集中在 `prompt-template-layer`。

建議拆成：

- `system_rulebook_prompt`
- `strategy_profile_prompt`
- `analysis_preset_prompt`
- `knowledge_graph_prompt`
- `data_bundle_prompt`
- `user_query_prompt`

並對每一種 prompt 模板做版本編號。

### 14.6 非同步任務

下列工作建議非同步執行：

- 收盤後重跑持股分析
- 每日盤前市場摘要
- session 對話萃取成圖譜節點與關聯
- 新聞來源驗證
- 法人異常掃描
- 大量 watchlist 股票批次重分析

### 14.7 可觀測性

系統應記錄：

- 每次 LLM 呼叫耗時
- 每次 LLM 呼叫 token 使用量
- 每個 session 的最近分析結果
- 每個資料來源的更新時間
- 失敗的外部抓取與模型呼叫

建議至少有：

- request log
- model usage log
- background job log
- data freshness dashboard

---

## 15. 分析流程序列

### 15.1 使用者提問個股時

1. API Gateway 驗證使用者身份
2. Session Service 找到或建立對應 `stock session`
3. Strategy Service 載入策略設定與版本
4. Session Service 載入 Analysis Preset
5. Market Data Service 拉取最新資料與資料新鮮度
6. Knowledge Graph Service 取回該股票的圖譜上下文與結構化 facts
7. Analysis Orchestrator 組裝 prompt 並呼叫 LLM
8. 回傳分析結果給前端
9. Background Worker 非同步更新圖譜節點、關聯與 snapshot

### 15.2 收盤後批次分析時

1. 排程器撈出所有 active portfolio 與 watchlist
2. 依股票建立批次任務
3. 每個任務載入該使用者策略與股票 session
4. 若資料已過期則先刷新 market data
5. 產出新的 AnalysisSnapshot
6. 若命中停損、異常風險或題材轉弱則送出通知

### 15.3 Session 建立策略

系統建立 session 時的邏輯建議為：

- 有相同 `user + strategy + symbol + timeframe + session_type` 且 status = active，則重用
- 否則建立新 session
- 若策略版本切換幅度很大，可選擇建立新 session，避免舊脈絡混淆

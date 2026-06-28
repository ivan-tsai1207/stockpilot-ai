# StockPilot AI 功能需求重構

## 1. 目的

本文件將 StockPilot AI 從原本偏系統設計導向的描述，重構為產品功能需求導向的規格。

重構後的目標是：

- 讓產品功能更容易被討論、排序與實作
- 讓使用者可感知的能力與底層架構解耦
- 讓後續 PRD、IA、API、後端設計可以共用同一套功能地圖

本系統仍維持核心定位：

> 以分析與決策輔助為主，不納入自動下單與實際券商委託執行

---

## 2. 功能主域

系統功能重構為四個主域：

1. 市場與看盤資料
2. LLM 分析工作流
3. 策略與知識圖譜記憶
4. 投資追蹤與提醒

---

## 3. 市場與看盤資料

### 3.1 Market Data Capability

系統需整合：

- 台股即時股價
- 美股即時或可用即時報價
- 歷史 K 線
- 分鐘級 K 線
- 法人、融資融券、主力分點
- 財報與基本面資料
- 新聞、公告、法說會
- 大盤、期貨、國際市場與宏觀資料

### 3.2 Chart & Indicator Capability

K 線週期至少需支援：

- 1分
- 5分
- 15分
- 1小時
- 日
- 週
- 月
- 年

每根 K 線需可檢視：

- 開盤價
- 最高價
- 最低價
- 收盤價
- 成交量
- 當前價或最新價

技術指標需分層支援：

- 趨勢類：MA、EMA、WMA、VWAP、Bollinger Bands、SAR
- 動能類：MACD、RSI、KD/Stochastic、CCI、ROC、Momentum
- 成交量類：Volume MA、OBV、MFI、Volume Ratio
- 波動類：ATR、Historical Volatility
- 多空判斷類：DMI/ADX

### 3.3 Index & Macro Dashboard Capability

台股大盤至少包含：

- 加權指數
- 櫃買指數
- 成交值
- 漲跌家數
- 類股表現

美股大盤至少包含：

- 道瓊工業指數
- S&P 500
- NASDAQ
- 費城半導體
- 對台股影響摘要

---

## 4. LLM 分析工作流

### 4.1 Stock Session Workspace Capability

系統中的分析工作區，不是全站共享聊天室，而是以單一股票為核心的獨立 session。

每個 session 至少綁定：

- 使用者
- 股票代號
- 市場別
- 時間週期
- session 類型
- strategy profile
- analysis preset

支援的 session 類型：

- 個股研究
- 持股追蹤
- 盤中分析

### 4.2 Analysis Preset Capability

使用者不應每次都手打 prompt 說明要分析什麼。

系統需提供可保存與重用的 `Analysis Preset`，至少包含：

- 盤中快速判斷
- 波段趨勢分析
- 基本面研究
- 技術面結構檢查
- 風險檢查
- 自訂模板

每個 preset 至少定義：

- 問題目的
- 必要資料欄位
- 預設輸出格式
- 預設策略套用方式

### 4.3 LLM 分析輸出能力

每次分析輸出至少包含：

- 目前狀態
- 關鍵事實
- 推論
- 支撐與壓力
- 風險
- Bull / Base / Bear 情境
- 策略建議
- 一句話結論

系統必須明確區分：

- 已確認事實
- 推論
- 假設

---

## 5. 策略與知識圖譜記憶

### 5.1 Strategy Profile Capability

每位使用者需可建立一個或多個策略設定。

至少支援：

- 風險偏好
- 持有週期
- 是否接受追高
- 是否偏好拉回買
- 是否允許分批進場
- 停損規則
- 停利規則
- 偏好技術指標
- 通知偏好

每份策略需支援版本化。

### 5.2 Knowledge Graph Memory Capability

系統記憶層不應以 session 摘要為主，而應改用 Obsidian 風格內建知識圖譜。

圖譜核心元素：

- note
- entity
- link
- tag
- backlink

至少支援的節點類型：

- Stock Note
- Theme Note
- Strategy Note
- Event Note
- Risk Note
- User Insight Note

每次 session 的結果需被萃取成：

- 圖譜節點
- 節點關聯
- 結構化 facts
- 分析快照

LLM 後續檢索時應優先使用：

1. 知識圖譜關聯
2. 結構化 facts
3. 最新必要對話片段

而不是優先回灌整段 session 歷史。

---

## 6. 投資追蹤與提醒

### 6.1 Portfolio / Watchlist Monitoring Capability

系統需支援：

- Watchlist 建立與管理
- Portfolio 持股建立與管理
- 持股成本、股數、停損停利設定
- 條件監控與重分析

### 6.2 Alert Capability

至少支援以下提醒類型：

- 價格突破或跌破
- 成交量異常
- 技術指標轉折
- 法人異常
- 題材轉弱
- 重大新聞
- 停損或停利接近

### 6.3 邊界限制

本系統在此階段僅提供：

- 分析
- 監控
- 提醒
- 決策輔助

本系統在此階段不提供：

- 自動下單
- 實際委託執行
- 券商交易撮合

---

## 7. 驗收標準

功能需求重構完成後，應能明確回答：

1. 使用者如何在不手打 prompt 的情況下啟動分析
2. 系統如何同時支援多檔股票且互不污染
3. 系統如何用不同策略 profile 產生不同建議
4. 系統如何以知識圖譜而不是長 session 做記憶
5. 系統如何提供台股與美股大盤、分鐘 K 線與完整技術指標
6. 系統如何維持純分析與監控邊界，不延伸到自動下單

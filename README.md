# 2026-11-11-DIC3-Project-Refactor

### 一、 摘要層級對析（Abstract：A–F 架構）

**【A. Attention Getter — Motivation】**
氣象預報與空氣品質監測資料為民眾規劃日常活動的重要參考依據，中央氣象署與環境部亦已提供完善的開放資料服務。然而，不同使用者對於氣溫、濕度、紫外線及過敏原等環境因素的感受與關注焦點各異，客觀數據難以直接回應個體差異需求。
**【B. But — Challenge】**
既有政府與民間平台主要著重於客觀數據之查詢與單一氣象呈現，資訊分散於不同管道，使用者仍需手動切換平台並自行判讀降雨、防曬或口罩需求。既有 AI 氣象對話服務多僅停留在通用性預報，缺乏同時整合多源時空對齊數據、個人耐受度檔案與可追溯決策機制的系統。
**【C. Cure — Proposed Solution】**
本研究提出 **「個人化天氣助理」（Personalized Weather Assistant / AirGuard AI）** 領域 AI Agent 系統。旨在整合中央氣象署與環境部多源開放資料，針對使用者規劃的多站行程，提供符合地點、時段、活動型態及個人偏好的行前行動建議。
**【D. Development — Method】**
系統採用**混合式架構（Hybrid Architecture）**。首先將行程標準化為時空區段（`ItinerarySegment`）並進行資料時空對齊與品質檢查；接著由版本化的**領域規則決策引擎**依據降雨、紫外線、空品及冷熱舒適產生結構化判定；**大型語言模型（LLM）** 則嚴格限制於需求解析與自然語言解釋，不得改寫規則判定或編造數據。系統並堅持將健康防護與舒適偏好拆分處理，完整保留 Rule ID 與資料來源以供追溯。
**【E. Experiments — Evaluation】**
本研究規劃比較「僅顯示官方原始數據 (Baseline A)」、「固定規則與文字模板 (Baseline B)」及「整合規則引擎、個人偏好與 LLM (Proposed)」三種方法。在多站跨鄉鎮、高降雨、高 UVI、相同 AQI 但不同主導污染物及資料缺漏/過期等情境下測試，評估建議可重現性、重要提醒遺漏率、限制揭露率、實用性與 API 回應時間。
**【F. Findings — Expected Outcomes】**
預期本系統能顯著降低使用者跨平台查詢與自行判讀的負擔，並提升行前建議之個人相關性、可理解性、實用性及資料可追溯性。


---

### 二、 緒論與相關研究對析（Introduction & Related Work：G–O 架構）

在 **`A2Z SOP`（G–O）** 及 **`專題規範`（AI + Domain 各半）** 要求下的技術分析：

#### 1. Introduction (G–L 架構)
*   **G. General (現況)**：臺灣環境開放資料健全，民眾普遍有出門前確認氣象與 AQI 的習慣。
*   **H. However (瓶頸)**：不同使用者對高溫、濕度與過敏原的耐受度不同。單一通用預報無法回答「這項天氣數據對『我』代表什麼」。
*   **I. In literature (文獻探討背景)**：包含 LLM 對話式氣象對話、MCP 空氣品質介面、多代理人健康建議、個人化推薦 Memory 模組，以及熱舒適性與 PM2.5 流行病學文獻。
*   **J. Judgement (既有文獻評論與 SWOT)**：既有 AI 天氣服務欠缺針對戶外多站行程的「個人耐受度建模」，且純 LLM 生成常有幻覺或不可追溯的風險。
*   **K. Keypoint & E. Innovation (創新點)**：將資訊呈現延伸為「結合個人偏好與行動建議的決策服務」，且堅持**數據與判定由規則鎖定，LLM 僅做解釋**。
*   **L. List Organization (架構交代)**：依序為緒論、相關研究、三大核心問題、系統設計、評估機制。

#### 2. Related Work（1/2 AI + 1/2 Domain）與文獻摘要表
*   **AI 技術面 (1/2)**：
    1.  *LLM Weather Agent*（Weathernews 2025）：對話查詢天氣。
    2.  *MCP + LLM 空品對話*（Pan & Nipu 2025）：以 MCP 工具呼叫即時空品。
    3.  *Instructor–Worker 多代理人*（Gao et al. 2025）：依野火空品產生健康建議。
    4.  *Recommender Profile/Memory*（Peng et al. 2025, Chen 2025）：選擇性檢索個人紀錄。
    5.  *WXImpactBench*（Yu et al. 2025）：劇烈天氣影響評測。
*   **Domain Knowledge 領域知識面 (1/2)**：
    1.  *個體熱舒適 PCM-RL*（Abdulraheem et al. 2025）：動態熱舒適具個體差異。
    2.  *臺灣戶外 UTCI 趨勢*（Shih et al. 2025）：熱島效應對戶外舒適之影響。
    3.  *低濃度 PM2.5 健康風險*（Sun et al. 2024）：低於 WHO 標準仍有健康風險。
    4.  *UVI 與降雨機率心理認知*（Kaiser et al. 2025, Joslyn et al. 2009）：大眾常誤解降雨機率與 UVI。
    5.  *WMO 影響導向預報指引*（WMO 2015）：將氣象危害連結至行動建議。
*   **文獻歸納鏈 (Summary Table Chain)**：
    \\[\text{Existing Research} \rightarrow \text{Limitations} \rightarrow \text{Research Gap} \rightarrow \text{Our Project}\\]
    *   **Research Gap**：既有研究尚未建立同時整合多源氣象/空品資料、個人耐受度檔案，且兼具可追溯性與資料限制揭露的日常行前建議 Agent。

---

### 三、 專題三大核心問題對析（Deep Research：Q1–Q3）

依據 **`期末專題提案規範` 第三節** 的要求，該專題針對氣象與物聯網領域所提出了完整的解答：

#### Q1. 五個核心思維模式 (Mindsets)
1.  **以使用者的行動與影響為中心**：數據必須轉換為出門準備建議（如帶傘、擦防曬）。
2.  **相同環境條件，不代表相同的個人感受與需求**：客觀天氣不變，個人偏好改變建議門檻。
3.  **預報具有不確定性，建議應考慮判斷錯誤的代價**：導入預期代價公式 \\(p \times rL > C\\) 評估帶傘決策。
4.  **個人舒適偏好與健康防護應分開處理**：主觀「不覺得熱/不在意空品」不能取消官方健康防護警示。
5.  **建議必須可追溯，資料品質與生成文字需要分層驗證**：保留測站、時間戳記與規則 ID，LLM 不得補造缺失資料。

#### Q2. 三個重要專業爭論 (Controversies)
1.  **固定規則判斷 \\(\leftrightarrow\\) LLM 自主產生建議**：
    *   *專題決策*：採用**混合架構**（規則負責數值判定與防護門檻，LLM 負責自然語言解析與解釋）。
2.  **統一標準 \\(\leftrightarrow\\) 學習個人耐受度**：
    *   *專題決策*：先採用**手動設定偏好**，健康防護硬性依照官方指引，避免過度擬合偶發回饋。
3.  **簡單明確的行動建議 \\(\leftrightarrow\\) 保留機率、條件與不確定性**：
    *   *專題決策*：採**分層呈現**（第一層行動建議 \\(\rightarrow\\) 第二層簡短原因 \\(\rightarrow\\) 第三層展開原始數據與限制）。

#### Q3. 十個深度理解問題 (10 Questions)
涵蓋 Why（個人化機制）、What-if（無症狀與健康防護）、Trade-off（遇雨代價模型）、Failure Case（資料更新時間差與同 AQI 不同污染物）、Evidence（AI 親切度不等於有效性）、Comparison（三組 Baseline 公平比較）、Design Decision（冷啟動偏好設定）與 Real-world Scenario（跨地區時空行程對齊）。

---

### 四、 系統技術設計對析（Proposed Scheme：P–P 架構）

對接 **`A2Z Proposed Scheme (P.PA–P.PM)`** 與 **`專題規範技術三要素`**：

#### 1. 系統架構圖 (System Architecture)
系統共設計 **11 個編號模組**：
*   **① UI 介面** \\(\rightarrow\\) **② 行程解析** \\(\rightarrow\\) **③ 時空標準化 (`ItinerarySegment`)**
*   **④ 帳號與偏好管理 (`UserProfile`)**
*   **⑤ 資料取得 (氣象署 + 環境部)** \\(\rightarrow\\) **⑥ 時空對齊與品質檢查 (`EnvironmentalSnapshot`)**
*   **⑦ 規則決策引擎 (`SegmentDecision`)** \\(\rightarrow\\) **⑧ 行程建議彙整器 (`TripRecommendation`)**
*   **⑨ LLM 解釋與追問處理** \\(\rightarrow\\) **⑩ 結果與限制呈現** \\(\rightarrow\\) **⑪ 行後問卷與偏好更新候選 (`FeedbackRecord`)**

#### 2. 時空對齊策略與責任邊界
*   **當前/進行中行程**：使用最近測站即時 AQI。
*   **隔日行程**：使用空品區預報。
*   **超出預報範圍**：標示「尚無官方空品預報」，不盲目推測。
*   **LLM 責任邊界**：嚴格限制 LLM **不得改寫優先級、不得自行補造數值、不得將即時值誤寫為預報**。

#### 3. Flowchart 與 Pseudocode (具有行號之演算法)
提案中提供了完整的 `Algorithm 1: Personalized Weather Anchor Agent` Pseudocode：
```text
Algorithm 1: Personalized Weather Anchor Agent
Input : User U, Itinerary I, Explicit Preferences P
Output: Trip Recommendation TR
01: Validate user account U and retrieve profile P
02: Validate every itinerary segment in I
03: if a segment has no valid Taiwan location, time, or activity then
04:     return request for missing or invalid fields
05: end if
06: Normalize each segment into location, township, air-quality area, and time window
07: Initialize an empty list Decisions
08: for each normalized segment S in I do
09:     WeatherData <- retrieveCWAForecast(S.township, S.timeWindow)
10:     AirData <- selectAirQualityData(S.timeWindow)
11:     if S is current or in progress then
12:         AirData <- retrieveNearestMoENVStation(S.latitude, S.longitude)
13:     else if S matches an official next-day forecast then
14:         AirData <- retrieveMoENVForecast(S.airQualityArea, S.date)
15:     else
16:         AirData <- unavailable("No official air-quality forecast")
17:     end if
18:     Snapshot <- alignAndValidate(WeatherData, AirData, S)
19:     if Snapshot has missing, stale, or spatially unsuitable critical data then
20:         Decision <- createLimitedDecision(S, Snapshot.qualityFlags)
21:     else
22:         RainAlerts <- evaluateRainRules(Snapshot, S, P.rainTolerance)
23:         UVAlerts <- evaluateUVRules(Snapshot, S, P.sunProtectionPriority)
24:         AirAlerts <- evaluateAirQualityRules(Snapshot, S, P.airQualityAlertSensitivity)
25:         ThermalAlerts <- evaluateThermalComfortRules(Snapshot, S, P.thermalPreference)
26:         Decision <- combineAlerts(RainAlerts, UVAlerts, AirAlerts, ThermalAlerts)
27:         Decision <- attachEvidenceAndRuleIds(Decision, Snapshot, P)
28:     end if
29:     append Decision to Decisions
30: end for
31: TR <- aggregateItineraryDecisions(Decisions)
32: TR <- prioritizeCriticalProtectionBeforeComfort(TR)
33: persistDecisionRecord(U, I, TR)
34: TR.explanation <- generateExplanationFromStructuredRecord(TR)
35: return TR
```

---

### 五、 實驗評估與結論對析（Simulation & Conclusion：Q–Z 架構）

對接 **`A2Z (Q–Z)`** 與 **`專題規範評估與 Handover 標準`**：

*   **Q & R. Performance Metrics & Comparison (評估指標與比較組)**：
    *   *Baseline A*：原始氣象/AQI 數據。
    *   *Baseline B*：固定規則 + 文字模板。
    *   *Proposed*：規則引擎 + 個人偏好 + LLM 自然語言說明。
    *   *五大評估維度*：正確性（可重現性）、完整性（重要提醒遺漏率/限制揭露率）、個人化（滿意度與實用性）、可理解性（使用者能否說出原因）、系統性（API 成功率/回應時間）。
*   **W–Z. Conclusion & Zen (專題價值主張與貢獻)**：
    *   展現完整的 **Domain AI Agent** 範例，達成從「呈現數據」走向「輔助出門準備決策」的目標。
    *   保持強烈的**資料可追溯性**與**數據/模型責任劃分**，為生成式 AI 結合政府開放資料提供了標準示範。

### 六、核心發現與說服力對接摘要

1. **Important（極致重要性）— 迎戰 2026 超級聖嬰與 39.4°C 極端高溫與政策瓶頸**
   * **關鍵案例新聞**：**ESG 遠見** 與 **中央氣象署** 指出，在氣候變遷與超級聖嬰影響下，2026 年夏季台北內湖已測得 **39.4°C** 極端高溫。立法院與環境部質詢揭露目前臺灣調適治理面臨「跨部會指標紊亂」（勞動部/中研院 WBGT vs. 氣象署體感溫度）且現有抗高溫涼適地圖僅停留在靜態據點，欠缺與「個人行程與耐受度」的對接。
   * **國際權威數據**：**世界氣象組織（WMO）** 與 **Lancet Countdown 2025** 報告顯示，全球高溫相關死亡率較 1990 年代大幅增加 23%（每年約 54.6 萬人死亡），極力呼叫將氣象數據轉化為「行動導向防護（Actionable Warning）」。
   * **專題切入點**：AirGuard AI 直接解答了臺灣高溫與空品防護政策最缺乏的**最後一哩路（Last Mile）** — 將官方開放數據對齊個人化多站行程。

2. **Innovation（技術創新性）— 歐盟 ECMWF 2026 混合 AI 架構趨勢**
   * **最新研究標竿**：根據 **EuroHPC JU** 與 **WeatherBench 2** 2026 最新 AI 氣象與空品對比報告，歐盟與 ECMWF 正全面採用圖神經網路（GNN）與 AI 氣象預測（如 EPT-2、AIFS）將多污染物推算速度縮短至「數秒」級。
   * **專題切入點**：AirGuard AI 採用的「硬性規則引擎（判定門檻、0 幻覺）+ LLM（自然語言解釋）」**Hybrid AI Agent 混合式架構**，完全符合國際上對環境與醫療 AI 代理人的可追溯性（Traceability）規範，解決了純 LLM 容易產生「高溫與空品風險幻覺」的致命缺陷。

3. **Interesting（實用與趣味性）— 解決盲目帶傘與 UVI/AQI 認知誤區**
   * **心理學與決策模型**：結合 WMO 與心理學實驗數據（大眾常誤解降雨機率為降雨面積或時間比例），專題引進「遇雨代價模型 (\\(p \times rL > C\\))」，並將「主觀舒適偏好（怕熱/怕冷）」與「官方硬性健康防護（AQI 污染物、UV3+）」明確拆分，避免通知疲勞。

### 七、應用預期貢獻

1. **環境資料整合（減少跨平台查詢負擔）**
* **跨源資料統一處理**：成功整合**中央氣象署**（氣溫、濕度、降雨機率、紫外線）與**環境部**（即時 AQI、主要污染物、空品預報）的多源官方開放數據。
* **一站式對齊呈現**：建立時空對齊架構，解決使用者日常需切換多個政府/民間平台自行湊合與交叉比對氣象資訊的操作痛點。

2. **個人化偏好設定機制（對接個體差異需求）**
* **彈性耐受度模型**：設計**個人偏好管理模組**，紀錄使用者對高低溫（怕熱/怕冷）、防曬重視度及空品提醒敏感度等個人耐受度與關注焦點。
* **突破通用型預報**：針對相同的客觀環境條件，系統能結合個別使用者的體感與活動情境，動態調整提醒門檻與資訊呈現優先順序。

3. **行前行動建議產生機制（數據轉化為決策）**
* **數據導向行動建議**：建立結合環境條件與個人偏好的**領域規則決策引擎**，將抽象數值（如 AQI、UVI）轉換為具體、可執行的出門準備建議（如攜帶雨具、防曬遮陽、配戴口罩等）。
* **降低認知負擔**：結合預期代價決策架構與分層呈現設計（主要建議 \\(\rightarrow\\) 原因說明 \\(\rightarrow\\) 展開原始數據與限制），提升資訊在日常生活中的實用價值。

4. **系統實作與多維度評估（實證與 AI 可追溯性示範）**
* **完整系統與嚴謹實驗**：完成全系統實作，並規劃**三組對照實驗**（Baseline A 原始數據、Baseline B 固定模板、Proposed 規則引擎+偏好+LLM），於多站跨鄉鎮及極端情境下測試。
* **多維度量化驗證**：透過功能測試與行後問卷，客觀評估建議的可重現性、重要提醒遺漏率、資料限制揭露率、使用者可理解性與 API 效能。
* **確立 AI 責任劃分與可追溯性**：堅持「數值與防護門檻由規則引擎鎖定，LLM 僅做自然語言改寫與問答」，保留資料來源與 Rule ID，為生成式 AI 結合政府開放資料提供具高可靠度與可追溯性的系統範例。

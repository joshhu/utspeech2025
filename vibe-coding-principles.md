## Vibe Coding 最重要的原則（技術版）

> 本文件整理「Vibe Coding」在專業軟體工程脈絡下的可操作準則，面向團隊落地、可治理與可審計。

### 定義與邊界
- Vibe Coding 定義：以自然語言/對話驅動的產碼流程；人類以需求與回饋引導 LLM 產生、修補與重構程式，而不以逐行手寫為主（Karpathy, 2025）。
- 與「AI 協助寫碼」區別：若工程師完整審閱並理解每行程式再合併，屬於「AI 輔助」而非 vibe coding（Willison, 2025）。
- 風險假設：預設 LLM 產碼存在安全、相依性、可維護性風險；所有策略均以可驗證證據（測試、靜態分析、SBOM、審計紀錄）作為收斂依據。

### 需求→設計→計畫先於產碼（Plan-before-Code）
- 以 PRD/ADR（Architecture Decision Record）寫清：目標、約束、資料流、外部介面、成功準則與風險。
- 強制模型先輸出計畫與模組切分（steps／WBS／API 合約與資料結構），經人審再允許進入產碼階段。
- 產出與計畫自動同步（在 repo 內生成 `/docs`、`/design`、`/contracts` 目錄並版本控管）。

### 上下文工程（Context Engineering）作為核心工序
- 上下文組態層：將規格、合約、範例、風險列表、測試骨架作為顯式上下文輸入，避免隱含假設。
- 檔案/知識檢索：以 RAG/向量索引選入最小充分子集；避免無關檔案污染生成。
- 上下文壓縮：摘要、關鍵函式抽取、介面簽章提取與程式切塊（chunking with boundaries）以符合 context window。

### 以「可執行事實」收斂（Tests-as-Truth）
- 在生成前先產生可執行規格（測試樣例、合約測試、property-based tests、安全測試腳本）。
- 對每次 AI 變更執行：單元/整合/端到端/安全掃描（SAST/DAST）、授權/授信測試（authz/authn）。
- 將測試通過作為合併門檻；把測試失敗訊號回饋給模型迭代（fail → reason → patch）。

### 介面先行與邊界明確（Contracts-first）
- 所有跨模組用 IDL/Schema（OpenAPI/JSON Schema/Protobuf）先行固化；模型按合約生成 stub/impl。
- 以轉接器/Facade 隔離第三方 API 與不穩定相依性，降低重生程式（regeneration）對核心域的衝擊。

### 生成管線與環境可重現（Reproducibility）
- 使用鎖定檔與可重現建置（`poetry.lock`/`package-lock.json`/`uv.lock`、容器鏡像 digest）。
- 追蹤 SBOM（CycloneDX/SPDX），比對供應鏈風險；CI 強制 License/漏洞守門。
- 對每次 LLM 呼叫保存：提示、模型版本、上下文來源、產出雜湊，供審計與回溯。

### 觀測性與審計（Observability & Auditability）
- 針對生成與執行路徑埋設遙測：日誌、追蹤、度量；建立變更日誌（changelog）自動生成。
- 對提示/回應加上紅隊標註（prompt injection / data exfiltration）分類，持續微調防護規則。

### 人在迴圈內的風險治理（Human-in-the-Loop）
- 人工必審：資料處理/隱私、金流、權限、加密、錯誤處理、可用性 SLA 等敏感區塊。
- 對非關鍵路徑允許快速接受（accept-and-iterate）；對關鍵路徑採雙人審/變更委員會。

### 安全預設最小權限（Least Privilege by Default）
- 將執行代理（agents/tools）以零信任/沙箱隔離；對外呼叫白名單化，API key 以密鑰管理服務託管。
- 自動產碼強制過秘密檢測、危險函式黑名單、規則式 SAST（例如：禁止 `eval`、SQL 直串）。

### 模型/工具鏈分工與版本化
- 規劃/推理用高推理模型；批量重構/生成用效能型模型；明確切換點與輸入輸出契約。
- 對模型版本與提示模板版本化；根據工作負載建立評測基準（coverage、latency、bug intro rate）。

### MVP 原則與漸進交付（MVP-driven Development）
- 以最小可行產品思維驅動生成：每次迭代產出可執行、可驗證的最小功能集，避免過度工程化。
- 優先實現核心價值主張，延後次要功能；讓早期使用者盡快體驗與回饋。
- 建立假設驗證循環：功能假設 → 最小實現 → 使用者測試 → 資料收集 → 假設修正。

### 迭代節奏（Tight Loop）
規劃（Plan）→ 生成（Generate）→ 執行（Run）→ 觀測（Observe）→ 封裝回饋（Constrain/Context Update）→ 重複
- 每輪輸出 Diff + 通過測試證據；嚴禁「一口氣大改」超過可審查尺度。

### 專案規模化（Scale-out）
- 以功能旗標/實驗控制風險釋出；灰度上線與回滾策略標準化。
- 用任務圖（task graph）/DAG 驅動多代理協作，節點輸入輸出完全可追蹤。

### 既有工程制度整合
- 保留設計審查、程式碼風格、分支策略、釋出準則等既有制度。
- Vibe Coding 與傳統工具（IDE、專案管理、文件協作）並行使用；以治理與品質為主線。

---

#### 參考與延伸閱讀（部分）
- Karpathy, A. 「There's a new kind of coding I call 'vibe coding' ...」(Tweet, 2025-02)  
- Simon Willison, *Not all AI-assisted programming is vibe coding* (2025-03)  
- Google Cloud, *What is Vibe Coding?* (2025-??)  
- IBM Think, *Vibe coding topic page* (2025-04-08)  
- Financial Times, *'Vibe coding' is the new DIY* (2025-06)  
- Business Insider, *YC Demo Day: vibe coding startups* (2025-09)


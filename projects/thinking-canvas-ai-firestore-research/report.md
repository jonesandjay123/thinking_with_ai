# thinking-canvas：AI 操作 Firestore 架構研究報告

## 1. 問題定義

### 為什麼需要讓 AI 存取 Firestore
thinking-canvas 這類 Firebase + AI 應用，如果要讓 AI 不只是「回答文字」，而是真的能參與產品流程，通常就必須能讀寫 Firestore，例如：

- 讀取 canvas / node 狀態，理解目前思考圖內容
- 新增、修改、整理 node
- 根據使用者指令批次更新結構
- 寫入 AI 執行結果、摘要、建議、系統狀態
- 觸發後續流程（審核、通知、版本化、回滾）

也就是說，AI 一旦從「對話模型」升級成「可行動 agent」，資料庫存取就會變成核心能力。

### 目前常見的設計挑戰
真正困難的點不是「技術上能不能寫 Firestore」，而是：

- *權限邊界怎麼切*：AI 到底能改哪些資料？
- *誰代表誰執行*：AI 是以使用者身份、bot 身份，還是 system 身份寫入？
- *怎麼避免誤寫*：模型 hallucination、誤判、prompt injection 都可能造成錯誤寫入
- *怎麼追責與回溯*：要知道是誰、何時、透過哪個 agent、改了什麼
- *怎麼符合 production 設計*：不能只是「能跑」，而要可管控、可審計、可演進

對 thinking-canvas 而言，真正的工程決策題目是：

> 是否應讓 AI 直接拿 Firestore 寫權限，還是只讓 AI 呼叫一層受控 API / Function 來代為操作？

---

## 2. 主流架構方案比較

以下列出 5 種常見方案，從 Firebase 生態與一般 AI/backend 系統角度一起看。

### 方案 A：Client SDK + 一般使用者 Auth + Firestore Rules

#### 架構圖（文字）
使用者端 / Node script -> Firebase Client SDK -> Firebase Auth 登入 -> Firestore Security Rules -> Firestore

#### 流程
1. AI script 用 Firebase Client SDK 初始化。
2. 透過 Email/Password 或其他 Firebase Auth provider 登入。
3. 帶著該使用者 token 直接讀寫 Firestore。
4. Firestore Security Rules 根據 `request.auth.uid`、custom claims、文件內容判斷權限。

#### 優點
- 完全沿用現有 Firebase client model。
- Security Rules 仍然有效，權限模型一致。
- 適合「AI 其實就是某個使用者的延伸」的情境。
- 不需要額外 server 層就能先跑起來。

#### 缺點
- bot account 本質上仍是「模擬一般使用者」。
- 若權限要放大，很容易把 rules 寫得過寬。
- 憑證管理麻煩：要保護 email/password、refresh token、登入狀態。
- script 端很容易繞過前端原本的 UX/驗證流程，直接打資料層。
- 多 agent / 多能力時，rules 會變得難維護。
- 難以做細粒度操作審核，例如「允許改 node.title，但不允許改 ownership / billing / hidden metadata」。

#### 適用場景
- 早期原型
- 單一 bot、低風險資料
- AI 代表固定使用者做有限操作
- 寫入行為簡單且可容忍風險

---

### 方案 B：Client SDK + bot account + 白名單 UID / custom claims 強化

#### 架構圖（文字）
AI script -> Firebase Client SDK -> bot account 登入 -> Firestore Rules 檢查 bot UID / custom claims -> Firestore

#### 流程
1. 建立專用 bot 帳號。
2. 在 Firestore Rules 中針對 bot UID 或 custom claims 給予特定路徑權限。
3. AI script 直接以 bot 身份操作特定 collection / document。

#### 優點
- 比普通使用者登入更可控，至少身份是獨立的 bot principal。
- 可以在 rules 中做 path-based allowlist。
- 對既有 Firebase-only 架構改動小。
- 比讓 AI 混用真人帳號安全得多。

#### 缺點
- 仍然是「直接打資料庫」。
- Firestore Rules 擅長 access control，不擅長複雜 business logic orchestration。
- 一旦 bot UID 被濫用，攻擊面就是 rules 允許的全部資料範圍。
- 如果 bot 需要跨文件檢查、狀態機、審批流程、審計欄位自動補寫，rules 會很快不夠用。
- 無法自然表達「某些寫入必須二階段確認」這類 agent safety flow。

#### 適用場景
- 比原型略成熟，但仍想低成本保留 direct Firestore 寫入
- bot 只操作少數固定路徑
- 寫入模型很單純，例如只可新增 AI note、不可覆蓋核心內容

---

### 方案 C：Backend / Cloud Functions 作為唯一寫入入口（推薦主流）

#### 架構圖（文字）
AI agent -> HTTPS / callable function / backend API -> 驗證身份與授權 -> Admin SDK -> Firestore

#### 流程
1. AI 不直接寫 Firestore，而是呼叫受控 API（例如 Cloud Functions / Cloud Run API）。
2. API 驗證呼叫來源（使用者 token、service-to-service auth、API key + signed context 等）。
3. Backend 做 schema validation、permission check、business rule check。
4. 通過後才用 Firebase Admin SDK 寫 Firestore。
5. 寫入 audit log、版本紀錄、可回滾資訊。

#### 優點
- 最符合 production 常見做法。
- 可把「能不能改」與「怎麼改」集中在 backend 控制。
- 可以做細粒度 validation、workflow、rate limit、idempotency、人工審批。
- 容易記錄 audit trail。
- AI 工具面可以被收斂成有限幾個安全操作，而不是給整個 DB 寫權。
- 便於後續替換 Firestore、增加 queue、審核、multi-agent。

#### 缺點
- 系統複雜度提高。
- 需要維護 backend contract。
- 若 function 設計太粗糙，可能變成「繞過 rules 的超級入口」。

#### 適用場景
- production
- 任何涉及核心資料寫入
- 需要可審核、可回滾、可演進的 AI 寫入能力
- 多角色、多種寫入操作、跨文件一致性要求高

---

### 方案 D：Backend command layer / tool layer（Agent 只能呼叫高階工具）

#### 架構圖（文字）
LLM / Agent -> Tool call（例如 `createNode`, `updateNodePosition`, `linkNodes`, `proposeRewrite`） -> Backend command handler -> Admin SDK -> Firestore

#### 流程
1. 給 AI 的不是資料庫權限，而是一組明確工具。
2. 每個工具有固定 schema、白名單參數、業務意圖。
3. Backend command handler 檢查參數、上下文、使用者權限。
4. 只允許工具執行對應 mutation。

#### 優點
- 最符合目前 AI agent 實務設計。
- 工具是 capability boundary，不暴露底層 DB 細節給模型。
- 易於做人機協作，例如高風險操作先 `proposeChange` 再 `applyChange`。
- 易於接 LangChain / OpenAI tool calling / Gemini function calling。
- 能有效降低 prompt injection 直接變成任意 DB 寫入的風險。

#### 缺點
- 需要先設計 tool surface。
- 初期比「直接讓 AI 寫 DB」慢一些。
- 如果工具切得太細，維護成本會上升；切得太粗，又會失去控制力。

#### 適用場景
- AI deeply integrated product
- 想讓 agent 穩定做事，而不是擁有任意寫資料庫能力
- thinking-canvas 這種 node / canvas 操作明確的產品

---

### 方案 E：非同步任務 / Queue / 審核式寫入

#### 架構圖（文字）
AI agent -> 提交 mutation proposal / task -> Queue / Firestore staging collection -> worker / reviewer -> Admin SDK -> Firestore 正式資料

#### 流程
1. AI 先寫入 proposal 或 task，而不是直接改正式資料。
2. 系統或人類 reviewer 檢查內容。
3. 背後 worker 套用變更到正式資料。
4. 記錄前後版本與審批結果。

#### 優點
- 安全性最高。
- 適合高風險寫入或大範圍重構。
- rollback 與 diff 非常自然。
- 對 AI 誤寫容忍度最高。

#### 缺點
- 即時性較差。
- 使用體驗較重。
- 系統設計成本最高。

#### 適用場景
- 高價值資料
- 批次重構、內容重寫、刪除操作
- 需要人工審核或可逆執行的環境

---

## 3. Firebase 生態最佳實踐

### 官方推薦做法
從 Firebase / Google Cloud 官方文件看，幾個方向很明確：

1. *Client SDK 存取會經過 Security Rules*  
   這是給前端或終端使用者環境的基本模式。

2. *Server-side 應使用 Firebase Admin SDK*  
   Admin SDK 用於 privileged environment，適合 server、Cloud Functions、Cloud Run 等環境。

3. *生產環境優先使用 Application Default Credentials（ADC）與附加的 service account*  
   官方明確偏好在 Google Cloud 環境中使用 attached service account / ADC，而不是散落的 service account key JSON。

4. *Service account 應 single-purpose + least privilege*  
   Google Cloud 明確建議不要共用高權限預設 service account，不要讓單一 SA 橫跨多個應用。

5. *Cloud Functions / backend 寫入邏輯應具備 idempotency、可重試與本地 emulator 測試*  
   這對 AI agent 特別重要，因為模型與網路都可能造成重試或重複提交。

### 常見錯誤
- 把 rules 寫成 `allow read, write: if request.auth != null`
- 為了 bot 方便，直接開大範圍 collection 權限
- 在本機或 repo 中保存 service account key JSON
- 讓多個系統共用同一個高權限 service account
- 用 Admin SDK 直寫 production，但沒有額外的業務驗證與 audit log
- 把 AI 工具做成「任意 document path + 任意 payload 寫入」
- 只記錄最終資料，不記錄 mutation intent / actor / before-after diff

---

## 4. AI Agent 實務案例

### 真實世界通常怎麼做
從目前主流 agent framework 與實務模式觀察，*AI agent 很少被設計成直接擁有任意 DB 寫權限*。更常見的是：

- AI 呼叫 tool / function
- tool 背後再呼叫 backend API
- backend API 驗證後才寫 DB

也就是說，業界標準不是「讓 LLM 直接碰資料庫」，而是「讓 LLM 呼叫受限制的能力介面」。

### LangChain / OpenAI / Gemini 類似案例觀察
雖然框架本身通常不規定你一定怎麼存 DB，但它們的主流設計都偏向：

- 用 *tool schema* 限制可呼叫的操作
- 用 *structured arguments* 限制輸入格式
- 用 *human-in-the-loop / tracing / observability* 監控 agent 行為
- 把外部 side effects（寫資料、發信、下單、刪除）放在可控工具後面

這代表 AI agent 的安全邊界，不應建立在 prompt 上，而應建立在 *tooling boundary + backend enforcement* 上。

### 是否有「AI 寫資料」的標準設計
有，但不是單一標準，而是收斂成幾個共通原則：

1. *LLM 不直接持有資料庫 root 權限*  
2. *所有 write 透過有限工具或 API*  
3. *每個工具有明確 schema 與 allowed effect*  
4. *高風險寫入要有審核 / 二階段提交 / rollback*  
5. *每次 mutation 必須可追蹤 actor、source、reason、timestamp*  

所以如果把問題翻成一句話：

> AI agent 寫資料的標準設計，不是 direct DB access，而是 controlled action execution。

---

## 5. 安全性分析

### 各方案風險

#### A / B 類：Client SDK 直接寫 Firestore
主要風險：
- bot credential 洩漏
- rules 寫寬後造成大面積誤寫
- 難做複雜業務限制
- prompt injection 一旦影響 agent decision，就可能直接變成資料異動
- 難做精準審計與回滾

#### C 類：Function / API 寫入
主要風險：
- backend 本身成為高價值攻擊面
- 若 API 設計過寬，相當於把 Firestore root access 包成另一層薄皮
- 若缺少 actor context，寫入後仍然難追責

#### D 類：command/tool layer
主要風險：
- tool 設計不良時，仍可能過寬
- 過於複雜時，維護成本提高
- 若沒有 per-tool authz，工具仍可能被濫用

#### E 類：proposal / queue / review
主要風險：
- 流程延遲
- staging 與正式資料一致性要管理
- 若 worker 權限太大且缺少 guardrail，仍可能出事

### 如何降低風險

#### 1. Least privilege
- 每個 agent / workload 使用單一用途 service account
- 不共用 default service account
- 僅授權需要的 Firestore / Cloud Functions / Secret Manager 權限
- 若可行，將不同功能拆成不同 function / service account

#### 2. 不讓 AI 直接持有廣泛寫權
- 最好不要讓 AI 直接拿 Admin SDK 對整個 DB 任意寫入
- 更不要給「任意 path、任意 payload」這種能力

#### 3. Capability-based tool design
- 只暴露 `createNode`、`updateNodeContent`、`moveNode`、`createEdge` 這種明確能力
- 禁止模型自由指定 collection path
- 對每個工具做 schema validation（例如 Zod / TypeBox / JSON Schema）

#### 4. Audit log + app-level mutation log
建議至少同時保留兩種紀錄：
- *Cloud Audit Logs*：基礎 infra 層誰用了哪個 service account
- *App mutation log*：哪個 user / agent / session 因為什麼理由改了哪筆 node / canvas

app-level log 建議包含：
- actorType（user / agent / system）
- actorId
- actingForUserId
- operation
- target refs
- before / after snapshot 或 patch
- requestId / traceId
- model name / tool name
- timestamp

#### 5. Rollback / versioning
- 對 node / canvas 實作 version history 或 event log
- 高風險操作使用 soft delete
- 批次變更先寫 proposal，再 apply

#### 6. Approval gates
對以下操作建議要求顯式確認或至少寫 staging：
- 刪除
- 批次重排 / 批次覆蓋
- 跨 canvas 大規模修改
- 修改 ownership / sharing / system metadata

#### 7. 測試與隔離
- 使用 Firebase Emulator Suite 做整合測試
- 對每個 function / command handler 做規則測試與 contract test
- 將 production 與 staging agent credentials 分離

---

## 6. 建議架構（最重要）

### 是否應該讓 AI 直接寫 Firestore？
*不建議直接寫正式 Firestore。*

更精準地說：
- *讀取* 在某些低風險情況下可以較寬鬆
- *寫入* 不應讓 AI 直接持有廣泛 Firestore mutation 權限

如果只是 prototype，bot account + rules 可以短期使用；但若問題是「production 級設計」，答案偏明確：

> 正式寫入應該走 Function / API / command layer，而不是讓 AI 直接拿 Firestore 寫權。

### 還是一定要走 Function / API？
對 *核心資料寫入*，我建議是：*幾乎一定要。*

原因不是 Firebase 做不到 direct write，而是：
- 你需要 business validation
- 你需要 auditability
- 你需要 rollback
- 你需要把 AI 的能力縮成可控工具
- 你需要未來能加上 approval / queue / policy

這些都不是 Firestore rules 最擅長的事情。

### bot account 是否合理？
*合理，但只能作為輔助身份，不應作為最終主架構。*

比較好的定位：
- bot account 可用於低風險讀取
- 或用於呼叫 backend 時代表某個 machine principal
- 但不建議讓 bot account 直接擁有 production 重要資料的 broad write rules

也就是說，bot account 可以存在，但應該出現在「入口驗證 / machine identity」層，不是直接成為 DB write architecture 的核心。

### 最推薦的整體架構
我對 thinking-canvas 的推薦是：

#### 推薦方案：*AI Tool Layer + Cloud Functions / Cloud Run API + Admin SDK + Mutation Log + Versioning*

##### 架構圖（文字）
使用者 -> 前端 / 聊天介面 -> AI orchestration layer -> 明確工具呼叫（createNode / updateNode / linkNodes / proposeCanvasRewrite） -> Cloud Functions / Cloud Run backend -> Admin SDK -> Firestore

旁路：
- Cloud Audit Logs
- app-level mutation log collection
- version history / rollback snapshots
- 高風險操作的 approval queue

##### 設計原則
1. *AI 不直接拿 Firestore 任意寫權*
2. *AI 只拿明確工具能力*
3. *每個工具背後是受控 backend mutation endpoint*
4. *backend 用 Admin SDK 實作，但自己做權限與資料驗證*
5. *所有 mutation 都寫審計紀錄*
6. *高風險操作分成 propose -> approve/apply 兩階段*

##### 為什麼這個最適合 thinking-canvas
thinking-canvas 的資料不是單純 CRUD；它其實是結構化思考圖操作：
- 建 node
- 改 node 內容
- 移 node 位置
- 接 edge
- 整理 canvas 結構
- 可能未來還有 merge / split / summarize / refactor

這種產品非常適合 tool-based command model，不適合讓模型自己隨便拼 document path 後直寫。

---

## 7. 可落地實作建議

### 簡單的架構設計（文字）

#### 第 1 層：前端 / AI orchestration
- 接收使用者指令
- 讓模型決定要呼叫哪個工具
- 不直接接觸 Firestore Admin 權限

#### 第 2 層：Agent tool surface
先定義少量高價值工具，例如：
- `getCanvasContext(canvasId)`
- `createNode(canvasId, type, content, position)`
- `updateNodeContent(nodeId, content)`
- `moveNode(nodeId, position)`
- `createEdge(fromNodeId, toNodeId, relation)`
- `proposeCanvasRefactor(canvasId, plan)`
- `applyApprovedMutation(mutationId)`

#### 第 3 層：Backend mutation service
可用 Cloud Functions v2 或 Cloud Run：
- 驗證呼叫來源
- 驗證 acting user / workspace membership
- 驗證 payload schema
- 驗證 domain rules
- 用 Admin SDK 寫 Firestore
- 寫 mutation logs / versions

#### 第 4 層：Firestore 資料層
建議至少有：
- `canvases`
- `nodes`
- `edges`
- `mutationLogs`
- `canvasVersions` 或 `nodeVersions`
- `mutationApprovals`（若要 approval flow）

### 建議技術組合

#### 最務實版本（推薦）
- Firebase Auth：真人使用者登入
- Cloud Functions v2 或 Cloud Run：AI 寫入入口
- Firebase Admin SDK：server-side Firestore 操作
- Firestore Security Rules：保護前端 direct access 的讀取與有限互動
- Secret Manager：保存 backend 需要的 secrets
- Cloud Audit Logs：infra 審計
- Firestore mutationLogs collection：app-level 審計
- Firebase Emulator Suite：本地整合測試

#### 如果 AI orchestration 在你自己的 Node backend
- 前端不直接叫 mutation function
- AI orchestration server 呼叫 internal backend API
- backend API 再用 Admin SDK 寫 Firestore
- 可搭配 queue（Cloud Tasks / Pub/Sub）做非同步高風險變更

### 實作優先順序建議

#### Phase 1：先把寫入收口
- 停止讓 AI 直接寫 Firestore 正式資料
- 改成只呼叫 3~5 個明確 mutation endpoints

#### Phase 2：加審計
- 每次 mutation 寫 `mutationLogs`
- 寫入 `actorId`、`actingForUserId`、`toolName`、`before/after`

#### Phase 3：加版本與回滾
- 對 canvas / node 建 version snapshot 或 patch log
- 支援 undo / rollback

#### Phase 4：高風險操作 approval
- 批次改寫、刪除、重構改成 proposal flow

---

## 總結結論

如果目標是 production 級 thinking-canvas，最重要的決策結論如下：

1. *不要把 AI 直接設計成 Firestore 的任意寫入主體。*
2. *Firestore direct write 適合 prototype，不適合長期核心架構。*
3. *最推薦的路線是：AI 呼叫受控工具 -> Cloud Functions / Cloud Run API -> Admin SDK -> Firestore。*
4. *bot account 可以保留，但不應成為核心寫入安全模型。*
5. *真正的安全邊界要建立在 backend command layer，不是 prompt，也不只是 rules。*
6. *所有 AI mutation 都應有 audit log、actor context、versioning、必要時 approval gate。*
7. *thinking-canvas 這種 node / canvas 產品，非常適合 capability-based tool design，而不是 direct DB mutation。*

---

## 參考觀察來源

- Firebase Firestore Security Rules 文件
- Firebase Admin SDK setup 文件
- Firebase callable / Cloud Functions 文件
- Firebase Emulator Suite 文件
- Firebase Firestore role-based access 文件
- Google Cloud Application Default Credentials 文件
- Google Cloud service account security best practices
- Google Cloud Audit Logs overview
- Firestore best practices 文件

以上報告除了官方資料，也加入 AI agent 系統設計的工程推論，目標不是學術盤點，而是幫 thinking-canvas 做可落地的架構決策。
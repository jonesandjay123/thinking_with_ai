# Gemini 語音 → Firestore / Trip Planner 對接研究

> 日期：2026-04-25  
> 問題：Jones 能不能用 Gemini 語音、Google Home、或其他 Google 原生 AI 入口，來控制像 `trip-planner` 這種以 Firebase Firestore 為後端的產品？

## 簡短結論

可以，但不是以前 Google Assistant 開發者想像中的那種「自訂 Google Assistant 對話 App」模式。

最重要的差別是：

- **Gemini API / Firebase AI Logic** 可以嵌進我們自己的 app 或 backend。這條非常可行。
- **Gemini App / Gemini Live / Google Home 語音這些 Google 原生入口** 目前看起來**沒有開放一個通用的公開 API，讓任意個人 web app 註冊自己的 Gemini Connected App / plugin / tool**，例如直接把 `addTripCard()` 註冊給消費者版 Gemini App 呼叫。
- **Google Home / Assistant 的智慧家庭語音** 可以呼叫第三方 cloud fulfillment，但它的設計是智慧家庭 device、trait、`SYNC` / `QUERY` / `EXECUTE` intents，不是任意對話式 app 指令。

所以實務答案是：

> 用 Firebase AI Logic / Gemini Live API / function calling 做一個 *Gemini 驅動的 trip-planner 語音介面*；如果 Jones 真的想要「Hey Google」入口，再額外做一層很窄的 Google Home 智慧家庭式指令。

對 `trip-planner` 來說，乾淨的 MVP 不是「讓消費者版 Gemini App 直接打 Firestore」。比較好的形狀是：

```text
我們自己控制的語音 UI
  ↓
帶 function declarations 的 Gemini model
  ↓
既有 Jarvis / trip-planner callable functions
  ↓
Firestore trips/main
  ↓
React app 透過 onSnapshot 即時更新
```

## 可以接進目前哪個本地架構

`trip-planner` 其實已經有很適合接這件事的 backend 形狀。

```text
~/Downloads/code/trip-planner
├── React / Vite 前端
├── Firebase Hosting
├── Firestore document: trips/main
└── Cloud Functions: functions/index.js
    ├── jarvisAddCandidateCard
    ├── jarvisUpdateCandidateCard
    ├── jarvisMoveCardToSlot
    ├── jarvisRenameDayLabel
    ├── jarvisInspectTrip
    ├── jarvisInspectDay
    ├── jarvisInspectCard
    ├── jarvisCreateTripBackup
    └── jarvisRestoreTripBackup
```

`jarvis-firebase-ops` 是本地操作層：

```text
~/Downloads/code/jarvis-firebase-ops/projects/trip-planner
├── OPERATIONS.md
├── .env.local
└── scripts/*.mjs
```

這些 scripts 會帶 shared secret 去呼叫已部署的 `jarvis*` callable functions。Gemini 語音介面可以重用同一種思路，但不應該把 Jarvis shared secret 暴露到 browser client；比較正式的做法是新增一個 `voiceCommand` 類型的 callable，由 server 端持有真正的權限。

## Google 目前官方支援狀態

### 1. 舊版 Conversational Actions 已經被關掉

Google 已在 2023-06-13 sunset **Conversational Actions**。這是以前用來做自訂 Google Assistant 對話體驗、搭配 webhook 的路線。

Google 官方 sunset 頁面說：Conversational Actions 已被移除，2023-06-13 之後使用者和開發者都不能再使用。不過 App Actions、Smart Home、Actions from web content、Media Actions 等其他路徑不受這次 sunset 影響。

這代表：

- 不應該規劃做舊式的「Talk to Trip Planner」Google Assistant action。
- 任何用 Dialogflow + Actions on Google Conversational Actions 的舊教學，對這個需求大多已經過時。

### 2. Gemini Connected Apps 是 Google 控制的白名單/平台功能

Gemini Apps 可以連接 Gmail、Calendar、Keep、Tasks、Google Home、YouTube Music、Spotify、WhatsApp、裝置 utilities 等 app，但可用清單會依裝置、帳號、地區、Gemini app 型態而不同。

查到的官方資訊：

- Gemini Help 說 Connected Apps 可以讓 Gemini 使用已連接服務；可用 app 會依 Gemini app、裝置、國家、帳號類型而不同。
- Gemini Live Help 說 Live chats 目前支援的 Connected Apps 是有限清單，例如 Google Calendar、Google Tasks、Google Keep，以及部分手機品牌的 notes/calendar apps；Google Home 支援也有額外限制。

這代表：

- Gemini 可以操作 Google Keep / Notes，是因為 Google 已經把那些服務做成 Connected Apps。
- 我沒有找到一條公開、消費者可用的「註冊自己的 Gemini Connected App，讓 Gemini 呼叫任意 API tool」路徑；不像 OpenAI GPT Actions 那種模式。
- 現階段把 Gemini App 當成「可任意擴充的原生 automation shell」風險很高。

### 3. Gemini 可以控制 Google Home，但要走 Google Home 支援的 device model

Gemini mobile app 可以透過 Google Home 控制支援的智慧家庭裝置。Google Help 文件列出的類型包含燈、插座、switch、thermostat、窗簾/百葉窗、媒體裝置、吸塵器、洗衣機、咖啡機等。

Google 文件裡的重要限制：

- Gemini 控制 Google Home 需要 Android 上的 Gemini mobile app、個人 Google Account、Keep Activity 開啟，並且連到同一個 Google Home 帳號。
- 該功能在引用的 Help page 裡明確說不支援 Gemini web、Google Messages、或 Live chats。
- 這是 convenience control，不適合安全或生命關鍵用途。
- 不能新增/刪除裝置，也不能完成某些 security device 操作。

這代表：

- 如果我們把 `trip-planner` 包裝成 Google Home 裡的某種「device」或「scene」，理論上有機會用語音控制。
- 但語意上對旅行規劃有點彆扭。它比較適合很窄的指令，例如「啟動 Tokyo planning mode」、「開始今天行程簡報」，不適合複雜卡片編輯。

### 4. Google Home Cloud-to-cloud 可以呼叫我們自己的 fulfillment service

Google Home Cloud-to-cloud integrations 可以讓 Google Assistant / Google Home 把智慧家庭 intents 發給第三方 fulfillment service。支援的 intent 包含：

- `SYNC` — 列出 devices / capabilities
- `QUERY` — 查目前 device state
- `EXECUTE` — 執行 command
- `DISCONNECT` — 帳號 unlink event

Google Home 會把第三方 OAuth2 access token 放在 Authorization header 傳給 fulfillment。

這代表：

- 技術上可以做一個 Cloud Function fulfillment endpoint，把智慧家庭 `EXECUTE` commands 映射到 Firestore mutations。
- 但它需要 account linking / OAuth，而且必須符合 Google Home device / trait model。
- 這條比較適合做「語音觸發按鈕」，不適合 open-ended 旅行規劃對話。

### 5. Firebase AI Logic 是 Firebase app 內接 Gemini API 的一級路徑

Firebase AI Logic 提供 Gemini models 的 client SDK，可用在 web、Android、iOS、Flutter、Unity 等。它支援：

- 自然語言與 multimodal prompts
- chat experiences
- structured output
- function calling
- Gemini Live API / 雙向串流音訊
- App Check 與 per-user rate limits
- 與 Firebase 服務整合，例如 Firestore、Storage、Remote Config

這代表：

- 因為 `trip-planner` 已經是 Firebase-backed，Firebase AI Logic 是最自然的 Gemini-powered AI feature 接法。
- 它不會自動讓消費者版 Gemini App 呼叫我們的 app；但它可以讓我們的 app 變成一個 Gemini-powered voice app。

### 6. Gemini function calling 是自然語言到既有 functions 的正確橋樑

Firebase AI Logic function calling 可以讓 Gemini model 從我們宣告的 functions 裡選擇工具，並回傳 structured arguments。真正的 function 還是由 app/backend 執行，執行結果再回傳給 Gemini 生成最終回答。

對 `trip-planner` 來說，可以宣告這類 functions：

```ts
addTripCard({
  title: string,
  date?: string,
  zone?: "morning" | "afternoon" | "evening" | "flexible",
  area?: string,
  duration?: string,
  note?: string,
  tags?: string[],
  location?: {
    placeName?: string,
    address?: string,
    lat?: number,
    lng?: number,
    confidence?: "high" | "medium" | "low"
  }
})

moveCardToSlot({
  cardId: string,
  planId: string,
  date: string,
  zone: "morning" | "afternoon" | "evening" | "flexible",
  index?: number
})

inspectDay({
  planId: string,
  date: string
})

renameDayLabel({
  planId: string,
  date: string,
  label: string
})
```

例如語音說：

> 「把秋葉原排到 5/2 下午，然後幫這天標成 Agent Body 採買日。」

就可以轉成兩個 tool calls：

1. `moveCardToSlot({ cardId: "...", planId: "...", date: "2026-05-02", zone: "afternoon" })`
2. `renameDayLabel({ planId: "...", date: "2026-05-02", label: "Agent Body 採買日" })`

## 對 Jones 的幾種整合選項

### Option A — 最推薦 MVP：在 `trip-planner` 裡內建 Gemini 語音

在 `trip-planner` web app 裡做一個 microphone button 或「語音/文字指令」面板。

架構：

```text
Trip Planner web app
  ↓ microphone / text command
Firebase AI Logic Gemini model
  ↓ function calling decision
Cloud Function: tripPlannerVoiceCommand
  ↓ 驗證 user + 解決 card/date/plan 歧義
既有 jarvis* mutation helpers 或內部 mutation functions
  ↓
Firestore trips/main
  ↓
React onSnapshot 更新 UI
```

優點：

- 符合目前 Firebase 架構。
- 避開已過時的 Assistant Actions。
- 可以支援很豐富的旅行規劃語言。
- 可以沿用既有 Firestore 與 callable function 邏輯。
- 遇到歧義時可以在 UI 裡追問。
- 只要瀏覽器 microphone permission 可用，desktop/mobile browser 都有機會支援。

缺點：

- 這是「Gemini 在我們產品裡」，不是「原生 Gemini App 控制我們產品」。
- 背景執行 / 鎖屏行為會受 browser / PWA 限制。

最適合第一批支援的指令：

- 「今天行程念給我聽。」
- 「把中野百老匯加到 5/2 下午。」
- 「幫 5/3 加一張午餐候選卡：鳥貴族。」
- 「查一下 5/2 下午有哪些卡沒有座標。」
- 「備份目前行程，label 叫 before-voice-test。」

### Option B — 比較像原生的 Android 路線：小型 Android companion app + Gemini / Assistant 入口

做一個 Jones 手機上的小型 Android companion app。

可能組件：

- Firebase Auth / Firestore / callable functions
- Firebase AI Logic Gemini Live API 語音
- Android shortcuts / App Actions 類型入口，如果目前 Android Assistant / Gemini integration 支援想要的 intents
- push / deep links 回 web app

優點：

- 比純 web 有更好的 microphone、background、手機系統整合。
- 比 web app 更可能接近「Hey Google」或 Gemini mobile assistant 的使用感。
- 未來可變成通用 Jarvis mobile control surface。

缺點：

- 工程量比 web MVP 大。
- Android App Actions / Gemini mobile 行為會變動，不一定能給完整 arbitrary API invocation。
- 需要安裝 app。

### Option C — Google Home Cloud-to-cloud pseudo-device

把 `trip-planner` 包裝成一個或多個 Google Home 虛擬裝置 / scenes。

例子：

```text
Device: Tokyo Trip Planner
Trait: OnOff / Scene / StartStop / Modes（視情況選）
Commands:
- “Hey Google, turn on Tokyo Trip Planner”
- “Hey Google, start Tokyo Day Briefing”
- “Hey Google, activate Tokyo backup”
```

Fulfillment：

```text
Google Home / Assistant
  ↓ action.devices.EXECUTE
Cloud Function fulfillment endpoint
  ↓ OAuth validation
Trip Planner command mapper
  ↓
Firestore / callable mutation
```

優點：

- 是真正 Google Home /「Hey Google」路徑。
- 很適合窄的 routine-like commands。
- 可以觸發 routines。

缺點：

- Smart-home model 不是為任意旅行規劃設計的。
- 需要 OAuth / account linking / device trait mapping。
- 如果要公開可能有 certification / policy overhead。
- 像「把這張卡移到那張卡後面」這種 rich command 很不適合。

建議：

- 除非目標就是做 Google Home 語音 trigger demo，否則不要從這裡開始。
- 如果要做，指令要保持窄、可逆、低風險。

### Option D — Google Keep / Notes bridge

因為 Gemini 可以寫 Google Keep / Notes，Jones 可以說類似：

> 「Add to Keep: Trip Planner command — add Nakano Broadway to May 2 afternoon.」

然後某個 automation 讀取 notes，把內容轉成 Firestore 操作。

優點：

- 利用 Jones 已經在用的 Gemini-native app path。
- 很低摩擦，適合 capture。

缺點：

- 這是 bridge / hack，不是一級 command channel。
- 消費者版 Google Keep automation API 相比 Firestore / Calendar / Gmail 生態更有限。
- 非同步解析 notes 會產生歧義與延遲。
- 如果沒有強 command protocol，很容易誤執行草稿或舊 note。

建議：

- 適合當 **inbox capture**：先收旅行靈感，之後 Jarvis 再 ingest。
- 不建議直接當 mutation command channel，除非每個 command 都要確認。

### Option E — 保留現在 Slack / Jarvis 路線，但讓 Jones 端用語音輸入

Jones 現在其實已經可以用手機 keyboard dictation 對 Slack 說話，Jarvis 再透過 `jarvis-firebase-ops` 執行。

優點：

- 已經可用。
- 有人類可讀的 audit trail。
- Jarvis 可以追問和確認。
- 風險最低。

缺點：

- 不是原生 Gemini / Home。
- 依賴 Jarvis 在線與 message channel。

建議：

- 即使做了 voice MVP，這條也應保留作為 fallback 與 control plane。

## 推薦分階段計畫

### Phase 1 — `trip-planner` 內建 web voice command panel

目標：證明「自然語言語音 → structured function call → Firestore update」這條鏈路可行。

要做：

1. 在 `trip-planner` 加一個 `VoiceCommandPanel` component。
2. 用 browser speech capture 或 Firebase AI Logic Live API。
3. 新增 server-side callable，例如 `tripPlannerVoiceCommand`。
4. 這個 callable 應該：
   - 驗證 Firebase Auth user 是否有權限
   - 用受限制的 tool schema 呼叫 Gemini
   - 把 card name 解析成 card ID
   - destructive / bulk action 要求確認
   - 呼叫既有 mutation helpers
   - 回傳自然語言 summary
5. 每次 voice command 都寫 audit log。

建議 Firestore log 形狀：

```text
trips/main/activityLog/{logId}  // 未來拆資料模型時使用
```

目前 MVP 可以先 append 到 `activityLog` field，或沿用 Cloud Functions 裡現有 logging；但要避免讓 `trips/main` 無限制膨脹。

### Phase 2 — 先做文字 command box，再做完整語音

在處理 microphone 複雜度之前，先加一個簡單文字 command box：

```text
「把中野百老匯排到 5/2 下午」
```

如果文字版可行，語音就只是另一種 input modality。

### Phase 3 — Gemini Live / audio mode

用 Firebase AI Logic Live API 做低延遲雙向語音。

使用情境：

- 「念給我聽 5/2。」
- 「把那個移晚一點。」
- 「復原。」
- 「哪些卡缺座標？」

Guardrails：

- Read-only commands 可以立即執行。
- 單一卡片、可逆的改動可以用 spoken confirmation。
- Delete / reset / restore / bulk changes 必須要求明確確認，最好先自動 backup。

### Phase 4 — 可選：Google Home shortcut layer

只有在 app-level voice loop 成功後，再加 Google Home Cloud-to-cloud 或 routine triggers，處理少數窄指令：

- 「Start Tokyo trip briefing.」
- 「Backup Tokyo trip planner.」
- 「What is today’s first stop?」

不要把 Google Home 當成主要 rich editing interface。

## Security / safety 注意事項

Voice control 不應該在 browser code 裡重用目前的 `TP_JARVIS_SHARED_SECRET`。

推薦 production shape：

- Frontend 使用 Firebase Auth。
- Callable 驗證 `context.auth` 與 owner/editor role。
- Server 持有 privileged Firestore mutation logic。
- Gemini function declarations 要 narrow 且 typed。
- Destructive commands 必須確認。
- 任何 restore / reset / bulk mutation 都要先建立 backup。
- 每個 command 記錄：
  - transcript
  - parsed intent
  - selected function(s)
  - before/after summary
  - actor UID/email
  - timestamp

## `trip-planner` 具體設計

### 新 callable：`tripPlannerVoiceCommand`

Input：

```json
{
  "utterance": "把中野百老匯排到 5/2 下午",
  "activePlanId": "plan_1777054282117",
  "timezone": "America/New_York",
  "clientContext": {
    "selectedDate": "2026-05-02",
    "selectedCardId": null
  }
}
```

Output：

```json
{
  "ok": true,
  "needsConfirmation": false,
  "summary": "已把中野百老匯排到 Jarvis Suggest 的 5/2 下午。",
  "actions": [
    {
      "type": "moveCardToSlot",
      "cardId": "nakano_broadway",
      "planId": "plan_1777054282117",
      "date": "2026-05-02",
      "zone": "afternoon"
    }
  ]
}
```

如果有歧義：

```json
{
  "ok": false,
  "needsClarification": true,
  "question": "你說的秋葉原是指『秋葉原電器街』還是『秋葉原 Agent Body 採買』？",
  "choices": [
    { "label": "秋葉原電器街", "cardId": "akihabara_electric_town" },
    { "label": "Agent Body 採買", "cardId": "agent_body_shopping" }
  ]
}
```

### MVP tool schema set

一開始只開安全、高價值工具：

- `inspectTrip`
- `inspectDay`
- `inspectCard`
- `addCandidateCard`
- `moveCardToSlot`
- `renameDayLabel`
- `backupTrip`

等 guardrails 穩定後再開：

- `deleteCandidateCard`
- `deletePlan`
- `resetPlan`
- `restoreTripBackup`
- bulk import / repair scripts

## 我的建議

以 Jones 真正的工作流來看，我會照這個順序做：

1. **先在 `trip-planner` 做文字 command box，使用 Gemini function calling。**  
   這是最快驗證方式，還不用碰 microphone / browser voice 的邊角問題。

2. **再在 `trip-planner` web / PWA 加 voice button。**  
   視我們想要多 conversational，選 browser speech input 或 Gemini Live API。

3. **升級成 Gemini Live mode，做 hands-free trip editing。**  
   這會有「直接跟我的 trip board 說話」的感覺，不需要等 Google 開放 custom Gemini Connected Apps。

4. **最後可選：做一個 Google Home Cloud-to-cloud mini integration，支援幾個 routine commands。**  
   很適合 demo / 日常 convenience，但不要把它當主編輯介面。

這樣能拿到 Jones 想要的核心好處：用語音自然控制一個 Firestore-backed product；同時避開依賴 Google consumer Gemini surface 尚未開放 custom API 的陷阱。

## 查過的資料來源

- Google Assistant Conversational Actions sunset overview: https://developers.google.com/assistant/ca-sunset
- Google Assistant Conversational Actions overview: https://developers.google.com/assistant/conversational/overview
- Gemini Connected Apps Help: https://support.google.com/gemini/answer/13695044
- Gemini Live Help / Connected Apps in Live: https://support.google.com/gemini/answer/15274899
- Gemini Google Home control Help: https://support.google.com/gemini/answer/15335456
- Google Home APIs overview: https://developers.home.google.com/apis
- Google Home Cloud-to-cloud get started: https://developers.home.google.com/cloud-to-cloud/get-started
- Google Home Cloud-to-cloud intents: https://developers.home.google.com/cloud-to-cloud/primer/intents
- Firebase AI Logic overview: https://firebase.google.com/docs/ai-logic
- Firebase AI Logic getting started: https://firebase.google.com/docs/ai-logic/get-started
- Firebase AI Logic function calling: https://firebase.google.com/docs/ai-logic/function-calling
- Firebase AI Logic Live API: https://firebase.google.com/docs/ai-logic/live-api
- Vertex AI Gemini function calling: https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/function-calling

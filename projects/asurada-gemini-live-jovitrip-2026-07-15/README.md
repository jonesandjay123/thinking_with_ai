# 阿斯拉：Gemini Live × Jovitrip 工具層可行性報告

> 研究日期：2026-07-15  
> 狀態：技術路線可行；尚未開始實作。  
> 範圍：Google / Gemini 的官方能力、實作介面、限制與開發者回饋；不把消費者版 Gemini 的能力誤當成自訂 App API。

## 結論先說

要做的不是「讓 Gemini 直接取得 Firestore 的萬用權限」，也不是用 Google Tasks 勉強模擬對話。

要做的是一個自己的 Android **阿斯拉（Asurada）App**：它以 Gemini Live 作為即時雙向語音與多輪對話引擎，將少量、明確、可驗證的 Jovitrip 操作宣告為工具；App 或受控後端執行工具，再把結果交回模型自然說出來。

```text
Android 阿斯拉 App
  ⇅ 麥克風 / 喇叭 / 即時對話
Gemini Live API（Firebase AI Logic）
  ⇅ function calls / function responses
Jovitrip tool executor（App + 受控 HTTPS API）
  ⇅ auth / revision / audit / confirmation
現有 Jovitrip 資料與服務
```

這是一次**入口層的轉向**：Google / Gemini 處理最難打磨的原生語音互動；Jovitrip 持有旅行世界的結構化狀態；Jarvis / OpenClaw 保留為跨系統、長期上下文、重型工作的後方工具。不是 Gemini 取代 Jarvis。

## 這份報告與舊研究的關係

本 repo 已有 [Gemini 語音 → Firestore / Trip Planner](../gemini-voice-firestore-trip-planner/)（2026-04-25）。當時的關鍵結論仍然成立：**消費者版 Gemini App 並沒有讓一般開發者註冊任意 Connected App / tool、直接控制自己的 Firestore 的公開途徑。**

這份新報告的焦點不同：以 2026-07-15 查到的 Android 官方 Live API 開發指南為準，確認「自訂 Android App + Firebase AI Logic + Gemini Live + function calling」是被 Google 文件明確支持的路徑，並將它對應到目前的阿斯拉 / Jovitrip 構想。

## 官方能力核對

### 1. 消費者 Gemini：很好用的入口，不是我們的工具宿主

Gemini 系列應用程式可連結 Google 與部分第三方服務，例如 Gmail、日曆、媒體、電話、訊息、Google Home 等；可用性會隨帳號、地區、裝置與語言改變。Google 說明也列出「自訂連結應用程式」目前是 Gemini Spark 工作中的特定能力，不能據此推論一般手機／手錶 Gemini 可以呼叫任意 Jovitrip API。

所以：

- Galaxy Watch / 手機 Gemini 很適合作為現成的快速生活入口：記事、Tasks、Calendar、常識問答。
- 但不能把「它能操作 Google 服務」外推成「它能直接 CRUD 我們任意 Firestore 或 HTTP API」。
- 用 Tasks 當單向收件匣仍是可選的低成本輔助流程；它不是阿斯拉的即時多輪旅行對話解法。

來源：[Gemini 連結的應用程式說明](https://support.google.com/gemini/answer/13695044?hl=zh-Hant)。

### 2. Gemini Live API：阿斯拉所需的語音 session 已存在

Firebase 官方把 Gemini Live API 定義為低延遲、雙向的即時語音／影片互動，透過 WebSocket 形成有狀態 session；可持續傳送音訊、文字或影片並收取語音回覆。Firebase AI Logic 文件提供 Android、iOS、Web、Flutter 等 SDK 的範例。

Android 官方指南明確指出：可從 Android App **直接**呼叫 Live API，不必另建一個 AI 代理 backend。最小前提為 Android API 23+、`RECORD_AUDIO`、Firebase AI Logic SDK 與 App Check。模型、功能和 SDK 的狀態目前仍是 developer preview / pre-release，會有不相容變更風險。

這表示「少一層我們自建的 Voice Server / STT / TTS 連接」是合理目標；但不表示整個產品不再需要後端。凡是資料授權、Jovitrip mutation、audit、rate limit、rollback，都仍需我們掌控。

來源：

- [Firebase AI Logic：Gemini Live API 入門](https://firebase.google.com/docs/ai-logic/live-api)
- [Android Developers：Gemini Live API](https://developer.android.com/ai/gemini/live)

### 3. Function calling：模型提案，程式執行

Google 的 Live API 文件寫得很清楚：在 session config 宣告工具；模型提出 tool call 後，**client 必須手動處理**，再以 `send_tool_response` 回傳 `FunctionResponse`。Live API 不會像某些非即時 SDK 一樣自動執行本機或遠端函式。

這正是安全邊界：Gemini 只輸出結構化的「我想呼叫 `move_trip_item`，參數是……」；阿斯拉工具層驗證參數、權限與目前 revision，執行後才回報成功、失敗或需要澄清。

來源：[使用工具搭配 Live API](https://ai.google.dev/gemini-api/docs/live-api/tools)。

### 4. Android 官方範例已包含 Live + function calling

Android 指南的 Kotlin 範例包含：建立 `LiveModel`、以 `model.connect()` 取得 `LiveSession`、用 `startAudioConversation()` 開始語音對話、將 `FunctionDeclaration` 作為 `Tool` 傳入模型，以及以 handler 處理 `FunctionCallPart`、回傳 `FunctionResponsePart`。

概念上可直接映射為：

```kotlin
val getDaySchedule = FunctionDeclaration(
  name = "get_day_schedule",
  description = "Read a single day of the active Jovitrip plan",
  parameters = mapOf("day" to Schema.integer("Day number"))
)

val moveTripItem = FunctionDeclaration(
  name = "propose_move_trip_item",
  description = "Prepare a move of an itinerary item; do not commit it without confirmation",
  parameters = mapOf(
    "itemId" to Schema.string("Jovitrip item id"),
    "targetDay" to Schema.integer("Destination day"),
    "targetTime" to Schema.string("24-hour local time")
  )
)
```

真正的 handler 不應直接相信模型的每個欄位；它必須將 item ID、旅程、時區、目前 revision 和使用者身份重新驗證。範例中的 `addList()` 是教學用途，不能直接照搬成「模型輸入 → 直接 Firestore write」。

## 建議的阿斯拉工具面

第一版只開放窄而可驗證的操作，不用做「任意代理人」。

| 類別 | 工具 | 是否可直接完成 | 回傳給語音的內容 |
| --- | --- | --- | --- |
| 查詢 | `get_active_trip`, `get_day_schedule`, `find_trip_item` | 是 | 明確行程摘要與 item ID |
| 提案 | `suggest_candidate`, `draft_move_item` | 是（只產生草稿） | 候選／變更摘要 |
| 低風險寫入 | `add_note`, `add_comment` | 是，附 audit | 已寫入的 ID 與摘要 |
| 結構變更 | `move_item`, `update_time` | 否，需語音確認 + revision match | 「請確認：……」 |
| 破壞性操作 | `delete_item`, `clear_day`, `delete_plan` | 第一版禁止 | 改由 UI / 網頁完成 |

### 一次「移動行程」的安全流程

```text
使用者：第三天的馬蹄灣改到六點
  ↓
Live model：find_trip_item + draft_move_item
  ↓
工具層：查到候選項、讀取當前 revision，產生 diff
  ↓
阿斯拉："我找到 Day 3 的馬蹄灣。要從 17:00 改成 18:00，對嗎？"
  ↓
使用者：對
  ↓
工具層：以 expectedRevision 執行 mutation、寫 audit、回傳 newRevision
  ↓
阿斯拉："已改成下午六點。"
```

確認不是只靠 prompt；確認 token 應綁定操作摘要、過期時間、使用者與 `expectedRevision`。若有人剛從 Web 介面改過該項，revision 不符便要求重新讀取和確認。

## App 直寫 Firestore vs. 呼叫 Jovitrip API

### A. App 直接 Firestore SDK

可行：Android App 使用 Firebase Auth、Firestore Android SDK 和 Security Rules；function handler 在 App transaction 內寫資料。

優點是少一跳、離家也可直接工作。缺點是複雜的業務規則、audit、批次限制、權限和 rollback 邏輯容易散落到客戶端；不能因為有 App Check 就授予過大 Firestore 權限。

### B. 呼叫現有 Jovitrip 受控 API（建議）

阿斯拉 handler 用使用者身分呼叫專用的 HTTPS tool API；服務端集中處理 auth、schema、revision transaction、audit、confirmation 與回復。資料層可繼續是現有新版 Jovitrip 的 SQLite / API 架構，不必為了 Gemini 回退到舊 Firestore 版。

這是較適合目前狀態的方向：**語音技術鬼轉到 Google，Jovitrip 的資料與安全模型不倒退。**

## 社群／實作者訊號：能力是真的，產品摩擦也是真的

### Function call 會打斷語音節奏

Google AI Developers Forum 有開發者回報：希望模型在長工具呼叫前先說「我幫你查」，但模型常先呼叫工具，提示詞不穩定。回覆建議針對 Live API 使用非同步 function calls，先回傳「請稍候」類訊息，再在工具完成後送入結果。官方 Live tools 文件亦明載：3.1 Flash Live 目前僅支援同步 function calling；2.5 Flash Live Preview 才有同步與非同步差別。

對阿斯拉的含義：不要把「先說一句 filler」當產品正確性的基礎。第一版應把讀取工具壓到很快；慢操作明確在 UI 顯示工作中，或選用與非同步工具相容的模型／設計。每個工具都要量測端到端時間。

來源：[Google AI Developers Forum 討論](https://discuss.ai.google.dev/t/anybody-succeeding-in-making-gemini-say-something-before-calling-functions/169087)、[Live tools 文件](https://ai.google.dev/gemini-api/docs/live-api/tools)。

### 不要以「消費者免費」估算 API 成本

官方 Firebase 文件只說部分 preview 模型可在 Gemini Developer API 免費層使用，並要求到所選供應商的定價頁確認 Live API 價格與配額；Live API 不支援 Count Tokens API，Firebase AI Logic 也尚未在 response 提供 UsageMetadata。開發期可低成本，但不能把手機 Gemini App 的免費體驗等同於可無限量使用的 API。

對阿斯拉的含義：POC 要建立每 session 的分鐘數、工具次數、失敗率和實際帳務觀測；正式用量估算不應靠 token counter。

來源：[Firebase Live API 的價格與 token 說明](https://firebase.google.com/docs/ai-logic/live-api#pricing-and-token-counting)。

## 建議 POC：先驗證「旅行工具迴路」，不先做完整阿斯拉

### POC-0：文件與帳務 gate（半天）

1. 建 Firebase 專案或使用隔離的 dev project；開啟 Firebase AI Logic、App Check、預算告警。
2. 確認當日可用 Live 模型、地區與費率，不將 preview 模型名稱硬編進產品承諾。
3. 決定 API route：優先走 Jovitrip 專用 tool endpoint，而非將管理 secret 放進 APK。

### POC-1：純 Live 語音（1 天）

Android 小 App 只驗證：點按說話、雙向音訊、打斷、繁中聽辨與語音、session reconnect。記錄首段語音、完整回覆、打斷成功率，不與舊系統只比較單一平均秒數。

### POC-2：只讀 Jovitrip（1–2 天）

工具只留 `get_active_trip`、`get_day_schedule`、`find_trip_item`。測試對話中的代詞、模糊日期和同名景點；工具結果回傳精簡結構化資料，不把整份行程塞給模型。

### POC-3：確認式寫入（1–2 天）

只新增 `add_comment` 與 `draft_move_item` / `confirm_operation`。必測：誤辨識、重複送出、網路中斷、revision conflict、過期 confirmation，以及 audit 能否還原「誰在何時要求做了什麼」。

### POC 成功標準

- 能完成「查 Day 3 → 指定項目 → 口頭確認 → 修改時間 → 語音回報」一個完整閉環。
- 每一筆 mutation 都有 authenticated actor、request ID、operation diff、before/after revision 和可理解的結果。
- 無法從 APK 取得廣泛 admin secret；伺服器不接受模型原始文字直接當作 mutation。
- POC 明確記錄 Live API / Firebase SDK 的 preview 版本與實際限制。

## 現階段不該承諾的事

- 不承諾消費者 Gemini 或 Galaxy Watch Gemini 能直接成為 Jovitrip 的通用 tool host。
- 不承諾 Live API preview 的模型名稱、定價、非同步工具行為或 SDK 介面穩定不變。
- 不承諾「原生音訊」就自然解決駕駛分心、錯誤命令與資料安全；高風險 mutation 仍應確認。
- 不先做全域個人記憶、任意檔案操作、部署或全系統代理。這些保留給 Jarvis / OpenClaw 的受控工作路徑。

## 最後判斷

這條線值得做，但最好的第一步不是重寫 Jarvis Voice，也不是重建 Firestore Trip Planner：

> **做一個小而受控的 Android Live 語音 POC，讓 Gemini 能自然地讀 Jovitrip，並以確認式工具完成一項行程變更。**

若這個閉環在真實手機與真實行程中順，阿斯拉才有資格慢慢長成個人系統的即時駕駛艙；若不順，現有 Jarvis Voice 的 Gateway 路線仍是可控的自主備援，而不是沉沒成本。

## 研究來源

官方文件於 2026-07-15 查閱；Google 頁面的繁中內容為機器翻譯，技術名詞與程式語意以原始 API 文件為準。

1. Google / Firebase, [使用 Firebase AI Logic 開始使用 Gemini Live API](https://firebase.google.com/docs/ai-logic/live-api)
2. Android Developers, [Gemini Live API](https://developer.android.com/ai/gemini/live)
3. Google AI for Developers, [使用工具搭配 Live API](https://ai.google.dev/gemini-api/docs/live-api/tools)
4. Google Gemini Help, [透過 Gemini 使用和管理連結的應用程式](https://support.google.com/gemini/answer/13695044?hl=zh-Hant)
5. Google AI Developers Forum, [Anybody succeeding in making Gemini say something before calling functions?](https://discuss.ai.google.dev/t/anybody-succeeding-in-making-gemini-say-something-before-calling-functions/169087)


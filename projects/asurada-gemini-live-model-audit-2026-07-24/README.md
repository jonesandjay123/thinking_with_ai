# Asurada Gemini Live 模型／SDK 現況稽核（2026-07-24）

## 結論先行

**Asurada 不應因 Gemini 3.6 Flash 發布而把 Live model 改成 `gemini-3.6-flash`。**

截至 2026-07-24，Gemini 3.6 Flash 是 Firebase AI Logic 的最新一般用途 Gemini 3.x 模型；但 Google 官方仍把雙向即時語音放在獨立的 **Gemini 2.5 Flash Native Audio / Live** 模型家族。Asurada 目前經 Gemini Developer API 使用的：

```kotlin
gemini-2.5-flash-native-audio-preview-12-2025
```

正是 Firebase 官方 Android/Kotlin 的現行 Live API 範例型號。直接換成 `gemini-3.6-flash` 並非升級，會使目前的「持續麥克風輸入 + 串流語音輸出 + `startAudioConversation()`」路徑失去官方支援基礎。

這次唯一值得排程的版本更新是 **Firebase Android BoM `34.15.0 → 34.16.0`**。它把 Firebase AI Logic 從 `17.13.0` 升到 `17.14.0`，含 Live automatic function calling；但 Asurada 現有工具層刻意採取「宣告工具 → app 驗證名稱／參數 → cache-only handler → 回傳結果」的安全邊界，所以不應在未審查語義與回歸測試前啟用任何自動派送行為。

## 本次查核範圍與時間點

- 查核時間：2026-07-24（America/New_York）
- Asurada source：`jonesandjay123/asurada`，`main` @ `305ec53`（2026-07-24 已 fast-forward）
- 官方來源：Firebase AI Logic 模型、Live API、Live 設定／限制，以及 Firebase Android release notes。
- 本文只判斷目前模型、SDK 與程式 API 相容性；沒有更改 Asurada 生產程式碼、Firebase Console 或部署設定。

## 官方模型現況：Gemini 3.6 與 Live 是兩條產品線

| 問題 | 官方現況 | 對 Asurada 的判定 |
| --- | --- | --- |
| 最新一般模型 | Firebase AI Logic 模型頁列出 `gemini-3.6-flash`。 | 可做文字、程式、多模態的一般模型評估，但不是這個語音 session 的替代品。 |
| Gemini Developer API 的 Live 型號 | `gemini-2.5-flash-native-audio-preview-12-2025`。 | **現行程式使用正確。** |
| Vertex AI 的對應 Live 型號 | `gemini-live-2.5-flash-native-audio`。 | 只在改用 Vertex backend 時使用；不要把這個別名直接塞進現有 `googleAI()` backend。 |
| Gemini 3.6 是否在 Live 型號清單 | 官方模型頁將 3.6 Flash 列在一般模型，Live 區塊仍列 2.5 Flash Native Audio。 | **沒有證據支持「3.6 已更新成 Gemini Live 原生語音模型」。** |

來源：

- [Firebase AI Logic — Supported models](https://firebase.google.com/docs/ai-logic/models)
- [Firebase AI Logic — Gemini Live API](https://firebase.google.com/docs/ai-logic/live-api)

## 程式碼逐點核對

### 1. 模型與 provider：正確，暫不改

`app/src/main/java/com/joneslab/asurada/session/FirebaseLiveSessionController.kt`：

```kotlin
private val liveModel = Firebase.ai(
    backend = GenerativeBackend.googleAI(),
).liveModel(
    modelName = GEMINI_LIVE_MODEL,
    // ...
)

internal const val GEMINI_LIVE_MODEL =
    "gemini-2.5-flash-native-audio-preview-12-2025"
```

這是 Gemini Developer API backend，因此應使用 Developer API 的 preview 型號；官方 Kotlin Live 範例也使用同一個字串。它和 Vertex AI 的 `gemini-live-2.5-flash-native-audio` 不是可任意互換的名稱。

### 2. managed audio 寫法：與官方 Kotlin API 一致

程式已設定：

- `responseModality = ResponseModality.AUDIO`
- `inputAudioTranscription = AudioTranscriptionConfig()`
- `outputAudioTranscription = AudioTranscriptionConfig()`
- `liveModel.connect()` 後呼叫 SDK `startAudioConversation()`

上述均與官方目前 Android 範例一致。程式也不自建 `AudioRecord`、`AudioTrack` 或另外搶佔 PCM transport；這符合 managed-audio 路徑，避免同時持有麥克風／播放與 SDK 管線的競態。

來源：[Live API configuration](https://firebase.google.com/docs/ai-logic/live-api/configuration)

### 3. 聲音設定：可選的產品決策，不是過期問題

現行程式未設 `speechConfig`，因此會採官方預設聲音 `Puck`。官方目前仍以 Chirp 3 HD 的預建聲音選項提供 Live 回覆聲音，並支援在 `liveGenerationConfig` 指定 `SpeechConfig(voice = Voice("…"))`。

若 Asurada 要固定人格聲線，可另開小批次在 product-level 設定一個已聽測的 voice，並在 Pixel 8 Pro 驗證繁中、插話、靜音／恢復與藍牙路由。這不等同 custom voice，也不應和模型／SDK 升級混在同一個變更中。

### 4. 工具安全邊界：17.14.0 更新需要審慎吸收

Asurada 的 function call 不直接執行任意模型要求：`ToolCallDispatcher`／`ToolRegistry` 驗證工具名稱與參數，而正式 Jovitrip path 只讀受限、已驗證的 memory cache。這個設計是正確的。

Firebase AI Logic `17.14.0` 新增了 `LiveGenerativeModel` 的 automatic function calling。這可能減少樣板程式，但也容易讓 SDK 行為跨越現有「app 明確控制工具分派」邊界。因此本次建議是：

1. 升 BoM 取得安全／相容性維護與新 API。
2. 保持目前明確 `functionCallHandler`，不要切換 automatic function calling。
3. 在隔離 branch 讀完 17.14.0 API 文件與 source 行為後，才決定是否做無 side effect 的 spike。

來源：[Firebase Android SDK release notes — BoM 34.16.0 / AI Logic 17.14.0](https://firebase.google.com/support/release-notes/android#bom_v34-16-0)

## 版本差異與建議動作

| 項目 | 目前 repo | 官方最新（查核日） | 建議 |
| --- | --- | --- | --- |
| Firebase Android BoM | `34.15.0` | `34.16.0` | 開獨立、可回復的 dependency update PR／branch。 |
| Firebase AI Logic | 經 BoM 解析為 `17.13.0` | `17.14.0` | 跟隨 BoM 更新；注意 new automatic function calling，預設不啟用。 |
| Gemini Live model | `gemini-2.5-flash-native-audio-preview-12-2025` | Firebase Android 官方 Kotlin 範例仍使用同一名稱 | **維持不變。** |
| Gemini 3.6 Flash | 未使用 | `gemini-3.6-flash` | 不可當作 Live model 直接替換；若需要可另做非即時／文字工作流評估。 |

本機這次未做 Gradle build：執行環境未安裝 Java Runtime。這不是程式驗證失敗，而是本台 Mac mini 的 Android build boundary；實際更新須在 Jones 的 Android 開發機完成。

## 風險與下一個最小批次

即使模型名稱目前正確，Live API 仍是 preview 類型，程式本身也以 `@OptIn(PublicPreviewAPI::class)` 明示此風險。官方 Live 限制目前包括約 10 分鐘連線長度與 audio-only 約 15 分鐘 session context；官方提供 session compression、resume 以及 going-away notification 處理選項。

Asurada 目前有 bounded reconnect 與 session lifecycle 清理，但尚未看見對 session resume／going-away 的明確產品級處理。這不是 Gemini 3.6 升級的 blocker；較合理的後續順序是：

1. **版本 PR**：只將 `firebaseBom` 更新到 `34.16.0`，不改 model name、不啟用 automatic function calling。
2. **Android 實測**：跑既有 unit tests／debug build，再在 Pixel 8 Pro 回歸連線、音訊、input/output transcript、工具拒絕、mute、background 與 App Check。
3. **獨立 reliability spike**：評估 SDK 是否已足夠安全地使用 resume/compression/going-away；先產 ADR 與測試記錄，不混入 Jovitrip write 或 audio transport 重構。
4. **未來模型 watch**：只有當 Firebase 的 Live model 清單正式出現 Gemini 3.x Native Audio 型號，以及 Android SDK 文件給出對應範例後，才考慮模型切換與 A/B 實測。

## 給下一個實作者的結論

不要把「最新 Gemini」誤解為「最新 Live voice」。截至本次查核，Asurada 的 Live 型號與 managed-audio 寫法是對的；更新 SDK 可以做，但要保持工具 handler 的 app-controlled 安全模型。Gemini 3.6 Flash 值得另外評估為文字／多模態 intelligence layer，而不是替換正在工作的即時語音 transport。

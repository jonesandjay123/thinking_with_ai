# Implementation Snapshot — Japan Phrase Buddy

> Snapshot time：2026-04-29 late evening EDT  
> App repo：<https://github.com/jonesandjay123/JapanPhraseBuddy>  
> Observed commit：`30cb4fe Zoom Wear launcher icon`

## Snapshot purpose

這份文件記錄 Japan Phrase Buddy 從 GPT 顧問建議到實際 Codex 實作後的落差與成果。它不是完整 code review，而是 project memory：之後回看時可以快速知道「當晚到底做成了什麼」。

## Repo state

`JapanPhraseBuddy` 是 Android multi-module repo：

```text
app/   手機版
wear/  Wear OS 手錶版
```

主要檔案：

```text
app/src/main/java/com/joneslab/japanphrasebuddy/MainActivity.kt
wear/src/main/java/com/joneslab/japanphrasebuddy/wear/MainActivity.kt
app/build.gradle.kts
wear/build.gradle.kts
README.md
```

## 手機版 app 實作摘要

手機版集中在 `app/src/main/java/com/joneslab/japanphrasebuddy/MainActivity.kt`。

核心資料模型：

```kotlin
data class PhraseRecord(
    val chinese: String,
    val japanese: String,
    val createdAt: Long = System.currentTimeMillis()
)
```

實作重點：

- 以 `createdAt` 當作小卡 id / key。
- 使用 SharedPreferences 儲存 `recent_phrases` JSON。
- 使用 Android `TextToSpeech` 播放日文，語言設為 `Locale.JAPAN`。
- 使用 `RecognizerIntent` 做中文語音輸入。
- 使用 `android.icu.text.Transliterator` 將辨識結果轉繁體中文。
- Gemini API 直接用 `HttpURLConnection` 呼叫 REST endpoint。
- 每張卡可以單獨翻譯，不需要一次翻全部。
- 先新增中文小卡，再視需要翻譯，降低 API failure 對 capture 的影響。

## Wear OS app 實作摘要

Wear 版集中在 `wear/src/main/java/com/joneslab/japanphrasebuddy/wear/MainActivity.kt`。

核心資料模型：

```kotlin
data class WearPhraseCard(
    val id: Long,
    val chinese: String,
    val japanese: String
)
```

手錶端設計：

- 只做被動 companion app。
- 不輸入中文。
- 不呼叫 Gemini。
- 不編輯或刪除小卡。
- 接收手機同步的小卡 JSON。
- 顯示中文 / 日文。
- 播放日文 TTS。
- 可按「同步」向手機要求最新資料。
- 使用固定 dark color scheme。

## Sync protocol

使用 Wear OS Data Layer。

Constants：

```kotlin
private const val PHRASE_CARDS_PATH = "/phrase_cards"
private const val REQUEST_PHRASE_CARDS_PATH = "/request_phrase_cards"
private const val PHRASE_CARDS_JSON_KEY = "cards_json"
```

流程：

1. 手機新增、刪除、翻譯、匯入、排序後，push 整份 cards JSON 到 `/phrase_cards`。
2. 手錶啟動或按「同步」時，透過 MessageClient 送 `/request_phrase_cards` 給手機。
3. 手機的 `PhonePhraseSyncService` 收到 request 後，再 push 一次最新 `/phrase_cards`。
4. 手錶的 `DataClient.OnDataChangedListener` 收到資料後更新 UI 並寫入本機 SharedPreferences。

## Gemini handling

API key：

```properties
GEMINI_API_KEY=...
```

由 `app/build.gradle.kts` 從 root `local.properties` 讀入，注入：

```kotlin
BuildConfig.GEMINI_API_KEY
```

目前 README 記錄模型為 `gemini-2.5-flash-lite`。

設計判斷：

- 這是自用 debug app，key 直接進 BuildConfig 可接受，但不適合公開產品或上架。
- 若未來要公開，應改成 Firebase callable / backend proxy。

## 與原始 GPT MVP 的差異

原始 MVP 包含：

- 預載日本常用句
- 分類 filter chips
- 搜尋
- 收藏
- 先不做 Wear OS

實際版本改成：

- 不做預載句。
- 不做分類。
- 不做搜尋。
- 不做收藏。
- 以「即時新增中文小卡」為核心。
- 加入中文語音輸入。
- 加入 LLM 匯入 / 匯出 fallback。
- 直接做出 Wear OS companion module。

這個差異不是退步，而是產品焦點改變：從「預先片語庫」轉成「臨場 capture / translate / play」。

## 目前設計優點

- 小卡先存中文，不會因 Gemini failure 丟失 intent。
- 每張卡單獨翻譯，節省 token 與錯誤風險。
- TTS / copy / import-export fallback 都是旅途中真實有用的備援。
- 手錶端限制得很窄，避免在小螢幕上做複雜管理。
- app / wear 都用同一個 JSON shape，同步簡單。

## 潛在風險 / 待觀察

- `createdAt` 當 id 通常足夠，但快速連續建立理論上可能碰撞；之後可改 UUID。
- app / wear module 的 `applicationId` 都是 `com.joneslab.japanphrasebuddy`，需確認 Wear OS 安裝 / pairing 行為是否如預期；若有衝突，可讓 wear 使用 `.wear` suffix。
- Gemini key 進 BuildConfig，debug 自用可接受，但公開 repo + APK 分發不安全。
- `MainActivity.kt` 承載過多邏輯，之後若繼續發展應拆出 data / sync / Gemini client / UI components。
- SharedPreferences + JSON 夠快，但資料量變多時可改 DataStore / Room。
- 沒有分類 / 搜尋，當小卡很多時可能變難找；但出發前 MVP 是合理取捨。

## 後續若要整理 code

建議拆分：

```text
data/PhraseRecord.kt
storage/PhraseStore.kt
gemini/GeminiClient.kt
sync/PhraseSync.kt
ui/PhraseCard.kt
ui/InputPanel.kt
```

Wear 端可共用資料 schema，或建立 `shared` module。

## 對 thinking_with_ai 的意義

Japan Phrase Buddy 是一個很好的「AI 顧問 + 本地 coding agent + Jarvis 記憶沉澱」案例：

- ChatGPT 負責產品 / 技術路線判斷。
- Codex 負責本機高速實作與 build 修錯。
- Jarvis 負責 repo inspection、脈絡整理、長期 project memory。

這正好符合 Jones 的 multi-agent system 分工：GPT 是 advisor，Jarvis 是 executor / memory keeper，本地 Codex 是短跑 coding worker。

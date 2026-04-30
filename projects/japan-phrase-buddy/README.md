# Japan Phrase Buddy

> 日期：2026-04-29 / 2026-04-30 EDT  
> 類型：旅行工具 / Android app / Kotlin Compose / Gemini phrasebook / Wear OS  
> Repos：
> - App：<https://github.com/jonesandjay123/JapanPhraseBuddy>
> - 對話來源：<https://chatgpt.com/share/69f2b50e-fff0-8331-a770-f562ee567515>
> - 舊參考：<https://github.com/jonesandjay123/TravelPhraseBook>

## 一句話

**Japan Phrase Buddy** 是 Jones 在出發日本前一晚，從舊 TravelPhraseBook 的想法快速演化出的自用 Android + Wear OS 現場溝通工具：輸入或語音說中文，存成小卡，需要時用 Gemini 產生自然日文，並能播放、複製、同步到手錶。

它不是大型片語庫，也不是上架產品；它是「明天真人互動要能用」的工具。

## 背景

Jones 原本有一個 2024 年底做的 Android app `TravelPhraseBook`，目標是預存旅行短語並用 TTS 播放。2026-04-29，Jones 不到 24 小時後要飛日本，於是先請 Jarvis 檢查舊 repo、加了日本旅行常用句與 README。

討論中很快發現舊架構有幾個時代感與成本問題：

- 原本是多語 phrasebook：中文、英文、日文、泰文。
- 原本的 API 是 Google Cloud Translation API v2，也就是傳統翻譯機路線。
- Jones 這次只需要中文 → 日文，英文和泰文都是額外 UI / 額外成本。
- 真人互動場景中，Gemini 的「自然潤色」反而比逐字翻譯更有價值。
- 舊 app 的匯入 / 匯出流程可用，但對臨場需求太麻煩。

因此方向從「改舊 repo」轉成「新開 repo，做日本行特化工具」。

## GPT 顧問視角收斂出的策略

ChatGPT 顧問視角的核心建議是：

1. **不要用 Flutter，選 Kotlin / Android native。**  
   因為目標設備是 Jones 的 Pixel 8 Pro，今晚目標是 debug install 可用，不是跨平台上架。Android TextToSpeech、RecognizerIntent、Wear OS Data Layer 都是 native 路線更順。

2. **不要沿用舊 TravelPhraseBook 的多語架構。**  
   舊 repo 只作為功能參考，不要保留英 / 泰欄位與分類負擔。

3. **Gemini 用來產生自然日文，不只是翻譯。**  
   需求不是文件翻譯，而是「台灣旅人在日本對真人說話」：自然、禮貌、能被理解即可。

4. **本地 Codex app 直接改 Android Studio 專案最快。**  
   在這種 2 小時內出成果的場景，「Jones 本機 Android Studio 模板 + Codex 直接改 + 本機 build 修錯」比 Jarvis/GitHub 來回更快。

5. **手機 MVP 優先，Wear OS 原本建議延後。**  
   顧問建議先做 phone MVP，隔天早上再貪心加手錶；但實際進度超前，Wear OS module 當晚也做出來了。

完整可見對話摘錄保存在：

- [`sources/chatgpt-share-extract.md`](sources/chatgpt-share-extract.md)

## 原始 MVP 需求

顧問視角最初建議的 MVP 是：

- App 名稱：`Japan Phrase Buddy`
- Package：`com.joneslab.japanphrasebuddy`
- Kotlin + Jetpack Compose
- 只支援中文原句與日文句子
- 預載日本旅行常用句
- 分類：交通、餐廳、飯店、購物、緊急、其他
- 首頁 phrase list
- 每張卡包含：中文、日文、分類、收藏、TTS 播放、複製
- 搜尋框與分類 filter chips
- 新增句子 UI：輸入中文 → 選分類 → 用 Gemini 生成自然日文
- Gemini API key 從 `local.properties` 讀取，不 commit key
- 沒有 API key 時仍可用預載句與 TTS
- 本地儲存用 SharedPreferences / DataStore / JSON 即可，不必 Room
- 暫時不做 Firebase、登入、iOS、上架、多語言、Wear OS

## 實際完成狀態（截至 2026-04-29 晚）

實際 repo 已經不只是初始 MVP，而是做出一個更貼近 Jones 當晚工作流的版本。

App repo 最新檢查時：

- Repo：<https://github.com/jonesandjay123/JapanPhraseBuddy>
- Branch：`main`
- 最新 commit：`30cb4fe Zoom Wear launcher icon`

### 手機版功能

- 中文文字輸入。
- Android 原生中文語音輸入，回填前會轉成繁體中文。
- 一句中文先存成一張小卡，不需要等 Gemini 成功。
- 每張小卡可以個別：
  - 翻譯 / 重翻
  - 播放日文 TTS
  - 複製日文
  - 刪除
  - 點卡片把中文載回輸入框重用
- 小卡可以用左側拖曳把手調整順序。
- 小卡存在本機 SharedPreferences，下次開啟仍保留。
- Gemini API 429 / 額度用完時，可匯出 prompt 給外部 LLM，再匯入 JSON 補回小卡。
- 每次新增、刪除、翻譯、匯入、排序後，手機會同步整份小卡 JSON 給 Wear OS。

### Wear OS 版本

實際已加入 `wear` module。

手錶端設計成被動 companion app：

- 不打 Gemini。
- 不輸入、不編輯。
- 接收手機同步的小卡 JSON。
- 顯示中文與日文。
- 播放日文 TTS。
- 可手動按「同步」向手機要求最新資料。
- 固定 dark mode，適合手錶螢幕。

同步方式使用 Wear OS Data Layer：

- 手機 push `/phrase_cards`
- 手錶 request `/request_phrase_cards`
- JSON 欄位 key：`cards_json`

### 技術選型

- Kotlin
- Jetpack Compose
- Android TextToSpeech（`Locale.JAPAN`）
- Android RecognizerIntent（`zh-Hant-TW`）
- `android.icu.text.Transliterator` 將語音辨識結果轉繁中
- SharedPreferences + JSON 儲存小卡
- Gemini REST API
- Wear OS Data Layer
- app / wear 雙 module

### Gemini key 與模型

API key 放在專案根目錄 `local.properties`：

```properties
GEMINI_API_KEY=你的_api_key
```

Gradle 讀取後注入 `BuildConfig.GEMINI_API_KEY`。`local.properties` 不應 commit。

目前 README 記錄模型為：

```text
gemini-2.5-flash-lite
```

## 需求演化：從「片語庫」到「臨場小卡」

這個專案有一個重要產品轉向：它最後沒有做成傳統 phrasebook，而是做成「臨場小卡」。

舊 TravelPhraseBook 的心智模型是：

> 出發前準備大量分類短語，旅行時搜尋 / 播放。

Japan Phrase Buddy 的心智模型變成：

> 旅途中突然想到一句等等要講的中文，先快速存成卡；需要時再翻成自然日文、播放或給對方看。

這個轉向很關鍵，因為它更符合 Jones 當下真正的需求：

- 不需要大型預載資料庫。
- 不需要英文 / 泰文。
- 不需要分類管理。
- 不需要等翻譯成功才能保存想法。
- 需要的是快速 capture、稍後翻譯、現場使用、手錶同步。

## 為什麼保留 LLM 匯入 / 匯出 fallback

雖然 Jones 覺得舊 app 的匯入 / 匯出太麻煩，但新 app 保留了一個更聚焦的 fallback 是合理的。

原因：手機端 Gemini API 可能遇到：

- 429
- quota 用完
- 現場網路不穩
- API key 未設定

因此 app 的主要路徑是「一鍵 Gemini 翻譯」，fallback 才是「匯出 prompt → 外部 LLM → 匯入 JSON」。

這不是把麻煩流程當主流程，而是保留旅途中救援能力。

## 與 TravelPhraseBook 的關係

`TravelPhraseBook` 應保留為 2024/多語 phrasebook 版本，不必再為這次日本行打掉重練。

`JapanPhraseBuddy` 是 2026/日本行/LLM-native 版本：

| 面向 | TravelPhraseBook | JapanPhraseBuddy |
|---|---|---|
| 主要用途 | 多語旅行短語庫 | 日本現場中文 → 自然日文小卡 |
| 翻譯 | Google Cloud Translation API / 手動 LLM 匯入 | Gemini 直接生成日文 |
| 語言 | 中 / 英 / 日 / 泰 | 中 / 日 |
| 儲存 | Room + SharedPreferences 排序 | SharedPreferences + JSON |
| 使用心智 | 預先準備片語庫 | 臨場 capture 小卡 |
| Wear OS | 舊版有上傳構想 | 已有被動同步 companion app |
| 產品方向 | 通用工具 | Jones 日本行特化工具 |

## 值得保留下來的產品判斷

1. **不要為了架構完整犧牲出發前可用性。**  
   這個專案成功的標準不是漂亮架構，而是明天 Jones 能否打開手機 / 手錶給日本人看。

2. **Gemini 的潤色是此場景優點，不是缺點。**  
   旅行真人互動比逐字精準更需要自然、禮貌、可理解。

3. **先存中文，再翻日文，是正確 flow。**  
   如果翻譯失敗也不丟失原始意圖；這比按「翻譯」後才建立卡片更可靠。

4. **Wear OS 只當被動端很正確。**  
   手錶不適合輸入、管理、API 錯誤處理；它適合顯示大字、播放、快速同步。

5. **LLM fallback 應該是救援，不是主流程。**  
   主流程一鍵翻譯，fallback 才匯出 / 匯入。

6. **新 repo 比大改舊 repo 更乾淨。**  
   保留舊 TravelPhraseBook 的多語歷史價值，新專案專注日本行。

## 未來可擴充方向

短期（出發前 / 旅途中）：

- 加幾句固定「緊急常用卡」置頂。
- 加「一鍵播放最近一張有日文的小卡」。
- 手錶端可加更大的播放按鈕與更醒目的日文。
- 支援離線手動輸入日文，不依賴 Gemini。

中期：

- 將小卡資料格式抽成 shared Kotlin model，避免 phone / wear 重複解析邏輯。
- 從 SharedPreferences JSON 換成 DataStore 或 Room。
- 加 import/export backup file，而不只剪貼簿。
- 加情境 prompt preset，例如「餐廳更禮貌」、「車站更簡短」、「緊急保持原意」。

長期：

- 多目的地模式：Japan / Korea / Taiwan inbound 等。
- 讓 Gemini 回傳多個語氣版本：簡短、禮貌、非常禮貌。
- 旅程模式：依日期 / 地點預先生成小卡。
- 後端 proxy 保護 Gemini key，避免 APK 內嵌 key 風險。

## 目前刻意不追求的事

- 不追求上架品質。
- 不追求跨平台。
- 不追求多語言通用性。
- 不追求分類 / 搜尋 / 收藏完整資訊架構。
- 不追求 API key 生產級安全。
- 不把它做成所有旅行者的產品；它首先是 Jones 明天能用的工具。

## 後續整理建議

如果旅程中真的用上，回來後值得補一份 `field-notes.md`：

- 哪些場景真的用到？
- Gemini 生成日文是否自然？
- 日本人看手機文字 vs 聽 TTS 哪個有效？
- 手錶端是否真的比手機快？
- 哪些 UI 在現場壓力下太小 / 太慢 / 太複雜？

這會讓 Japan Phrase Buddy 從一次性工具變成一個非常好的「真人互動 AI agent interface」案例。

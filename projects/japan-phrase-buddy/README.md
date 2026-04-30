# JapanPhraseBuddy 計畫

> 日期：2026-04-29
> 來源：ChatGPT share — <https://chatgpt.com/share/69f2b50e-fff0-8331-a770-f562ee567515>
> 類型：旅行工具 / Android app / Kotlin Compose / Gemini phrasebook

## 簡短結論

這段對話收斂出一個很明確、很適合光速實作的自用旅行 app：**Japan Phrase Buddy**。

核心想法是：Jones 明天要去日本，想在 2 小時左右做出一個 Android 手機上可跑的 phrasebook。它不是為了上架或利他，而是為了現場溝通可用；因此技術選型要偏向最短路徑、原生能力順、debug 直接。

最後建議路線是：

- 用 **Kotlin + Jetpack Compose**，不要用 Flutter。
- 直接使用 **Android TextToSpeech** 播日文，語言設 `ja-JP`。
- 先做 **phone MVP**，Wear OS 只預留架構，不要第一輪做。
- Gemini API 用來把中文生成自然、禮貌、適合台灣旅人在日本現場使用的日文。
- 本地 Codex app 直接改本機 Android Studio 專案，比透過 Jarvis/GitHub 來回更快。

## 產品定位

**Japan Phrase Buddy** 是一個給台灣旅人在日本現場使用的輕量 Android phrasebook。

GitHub description 建議：

> A lightweight Android phrasebook for Taiwan travelers in Japan, using Gemini to generate natural Japanese phrases for real-world communication.

README 開頭可用更完整一句：

> A Kotlin Android travel phrasebook that helps Taiwan travelers generate, save, play, and copy natural Japanese phrases for real-world situations.

## 為什麼選 Kotlin / Android native

對話中的關鍵判斷是：這不是泛用跨平台產品，也不是要優先上架，而是 Jones 自己 Pixel 8 Pro 明天就要用。

因此 Kotlin native 的優勢很明顯：

- Android TextToSpeech 原生支援最直接。
- Pixel 8 Pro 是很新的 Android 裝置，可以選較新的 target / SDK，不必為最舊設備妥協。
- 未來要接 Wear OS，Android native / Compose 生態更自然。
- Android Studio 模板已經建立，本地 Codex 可以直接改、build、修錯。

Flutter 不是不好，但在這個場景裡多了一層 bridge 與架構成本；如果今晚目標是「可跑、可裝、可現場用」，Kotlin native 更順。

## MVP 功能範圍

第一輪只做手機版，目標是 `./gradlew assembleDebug` 能過，並且可以 debug install 到 Android 手機。

必要功能：

- App 名稱：`Japan Phrase Buddy`
- Package：`com.joneslab.japanphrasebuddy`
- Kotlin + Jetpack Compose
- 只支援中文原句與日文句子
- 不做英文、泰文、多語言架構
- 預載日本旅行常用句
- 分類包含：
  - 交通
  - 餐廳
  - 飯店
  - 購物
  - 緊急
  - 其他
- 首頁 phrase list
- 每張 phrase card 包含：
  - 中文原句
  - 日文翻譯
  - 分類
  - 收藏按鈕
  - 播放日文 TTS 按鈕
  - 複製日文按鈕
- 搜尋框，可搜尋中文或日文
- 分類 filter chips
- 新增句子 UI：
  - 使用者輸入中文
  - 選分類
  - 按「用 Gemini 生成日文」
  - 呼叫 Gemini API 產生自然日文
- 本地儲存先用 SharedPreferences / DataStore / JSON 皆可，不必上 Room

## Gemini API key 處理

安全規則：不要把 key 寫進 Kotlin，也不要 commit 到 Git。

建議：

```properties
# local.properties
GEMINI_API_KEY=你的_key
```

`.gitignore` 應確認包含：

```gitignore
local.properties
.env
.env.local
*.keystore
```

App 行為：

- 沒有 `GEMINI_API_KEY` 時，預載句與 TTS 仍可使用。
- Gemini 生成按鈕應顯示明確錯誤提示，而不是 crash。

## 給 Codex 的執行 prompt

```text
你現在在本機 Android Studio 新建的 Kotlin / Jetpack Compose 專案 JapanPhraseBuddy 裡工作。

目標：今晚快速做出一個自用 Android app，用於我明天去日本旅行時和真人溝通。

請參考我提供的 TravelPhraseBook 舊專案和對話內容，但不要沿用舊專案的多語言架構。這次只做「中文 → 自然日文」phrasebook。

請完成一個可 build、可在 Android 手機 debug install 的 MVP：

1. App 名稱：Japan Phrase Buddy
2. Package: com.joneslab.japanphrasebuddy
3. 使用 Kotlin + Jetpack Compose
4. 只支援中文原句與日文句子，不需要英文、泰文
5. 預載一批日本旅行常用句，分類包含：
   - 交通
   - 餐廳
   - 飯店
   - 購物
   - 緊急
   - 其他
6. 首頁顯示 phrase list，每張卡片包含：
   - 中文原句
   - 日文翻譯
   - 分類
   - 收藏按鈕
   - 播放日文 TTS 按鈕
   - 複製日文按鈕
7. 使用 Android TextToSpeech 播放日文，語言設為 ja-JP
8. 提供搜尋框，可搜尋中文或日文
9. 提供分類 filter chips
10. 提供新增句子的 UI：
    - 使用者輸入中文
    - 選分類
    - 按「用 Gemini 生成日文」
    - 呼叫 Gemini API 產生自然、禮貌、適合台灣旅人在日本現場使用的日文
11. Gemini API key 從 local.properties 讀取：
    GEMINI_API_KEY=...
    不要把 key 寫進 git
12. 若沒有 API key，app 仍能使用預載句與 TTS，只是 Gemini 生成顯示錯誤提示
13. 本地儲存可以先用 SharedPreferences / DataStore / JSON，不必使用 Room，今晚以簡單可靠為主
14. 暫時不要做 Firebase、登入、iOS、上架、多語言、Wear OS。請只在架構上保持未來可以新增 Wear OS module。
15. 完成後請執行 ./gradlew assembleDebug，修到 build pass。
16. 最後請給我簡短摘要：改了哪些檔案、怎麼放 GEMINI_API_KEY、怎麼執行。
```

## Wear OS 判斷

對話裡有一個貪心方向：明早想做手錶。

建議順序是：

1. 今晚先讓手機版 MVP 跑起來。
2. 明早再加 Wear OS module。
3. 手錶版先做最窄功能：同步收藏句 / 緊急句，顯示大字日文，必要時播放或讓手機播放。

原因很現實：今晚如果同時做 phone + wear + Gemini + TTS + storage，很容易全部半成品。手機版先跑起來，才有資格貪心加手錶。

## 來源內容

完整可見對話摘錄保存在：

- [`sources/chatgpt-share-extract.md`](sources/chatgpt-share-extract.md)

## 可行下一步

- 在 JapanPhraseBuddy Android repo commit 乾淨模板：

```bash
git init
git add .
git commit -m "Initial Android Compose template"
```

- 確認 `.gitignore` 擋住 `local.properties` 與 `.env*`。
- 建 GitHub repo 並 push。
- 用上面的 Codex prompt 直接讓本地 Codex app 實作 MVP。

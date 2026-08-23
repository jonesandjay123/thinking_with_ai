# Pixel 11 Pro AI 能力地圖：可探索的功能、開發者入口與應用設計

> 查詢日期：2026-08-23（America/New_York）
> 對象：Pixel 11 Pro；Android 開發與個人實驗。
> 核心原則：把「Pixel 內建 AI 功能」和「第三方 App 有公開、可程式化的 API」嚴格分開。
> 狀態標記：**[已實機確認]**、**[官方確認]**、**[工程建議]**、**[尚未確認／勿依賴]**。

## 一頁結論

Pixel 11 Pro 現在最有價值的地方，不是把它當成一個只能使用 Gemini 的手機，而是把它視為一台有三層本地 AI 的 Android 實驗機：

1. **共享 Gemini Nano 層（AICore + ML Kit GenAI）**：適合短文字理解、改寫、摘要、結構化擷取、故事提示、圖片描述與新版語音辨識。它是最接近「手機替 App 想一下」的入口，但多數 API 仍是 Beta/Alpha，必須每次以 runtime status 為準。
2. **成熟的 on-device ML 層（ML Kit）**：文字辨識、翻譯、語言辨識、條碼、臉／姿勢／物體／影像標記等。這些不靠 Gemini Nano，卻往往更可預測、更適合產品功能。
3. **自帶模型層（LiteRT / MediaPipe / LiteRT-LM）**：若 AICore 的模型、配額、語言品質或產品限制不合用，可把自己的 classifier、embedding、vision pipeline，甚至 open-weight 小型 LLM 裝進 App。代價是 APK／下載大小、效能、量化與模型生命週期都由自己承擔。

Jones 已回報 Pixel 11 Pro 上「最新 Google 本地開發模型」與離線繁中語音輸入已實際運作良好。這是很重要的 **[已實機確認]**；但它不會自動把每個 ML Kit GenAI feature、每一種語言、每個 model variant 都變成正式支援。因此最好的下一步是把已跑通的路徑包成一套 benchmark app，而不是只把它當一次 demo。

## 先分清楚：什麼可以由 App 做？

| 能力層 | 你自己的 App 能否直接呼叫 | 是否可離線 | 最適合的事 | 主要限制 |
|---|---|---|---|---|
| ML Kit GenAI / AICore / Gemini Nano | 可以；先 `checkStatus()`，且依 feature availability | 模型／設定下載完成後可 | 生成、摘要、改寫、結構化抽取、圖片理解、語音辨識 | Beta/Alpha、裝置／版本／前景／quota 約束 |
| ML Kit 傳統 API | 可以 | 許多模型可 bundle 或按需下載 | OCR、翻譯、掃描、辨識、追蹤 | 不是通用推理模型；各 API 支援範圍不同 |
| Android platform API | 可以 | 視裝置 recognition/TTS service 而定 | 麥克風、on-device speech fallback、TTS、相機／感測器 | OEM/service 行為不一；並非全是 AI API |
| MediaPipe / LiteRT | 可以，模型由 App 管 | 可以 | 自己控制的 vision/audio/text 模型 | 自己付出模型整合、大小、性能與維護成本 |
| Pixel / Gemini consumer feature | 通常**不能**假設可叫用 | 功能各異 | 給使用者直接用 | 沒有公開 SDK 時不能拿來當產品能力 |

**一條硬規則：**「Pixel 上看得到 Gemini、Pixel Studio、相機 AI 或 Call Screen」不等於第三方 Android App 能程式化呼叫它。若 Google 沒提供 API、sample、權限與 support contract，就只能把它視為 consumer feature，而不是產品依賴。

---

## A. 最值得優先探索：Gemini Nano / ML Kit GenAI

ML Kit GenAI API 建構於 Android 的 **AICore** system service。App 呼叫的是受系統管理的 on-device Gemini Nano，而不是把模型塞進 APK。這是 Pixel 11 Pro 上最具獨特性的開發入口。

### 1. Prompt API：把 Nano 變成可控的本地 copilot

**可做的事 [官方確認]**

- 文字 prompt；部分 flow 也可結合 image input。
- blocking generation 或 streaming generation。
- generation config：`temperature`、`topK`、`seed`、`candidateCount`、`maxOutputTokens`。
- system instruction（Beta，應保持短小）。
- structured output（Alpha）：將輸出限制為 Kotlin schema／JSON-like typed object。
- prefix caching（Experimental）：讓固定指令、schema 或世界觀不必每次完全 prefill。
- `countTokens()`、`warmup()`、`getBaseModelName()`，用於預算、cold-start 與可觀測性。

**特別適合的 Pixel 11 Pro 實驗**

- JoviYarn：Story State patch、三個短提示、故事角色／物件／未解線索擷取。
- 離線「選項卡」助手：把一段筆記變成 3 個下一步、風險、反例或待問問題。
- UI intent parser：把「幫我把上週的點子整理成三個實驗」轉為嚴格 schema，交由本地 app 執行安全的內部動作。
- 個人 journal／voice memo 的局部整理：**只**將已確認的短 chunk 與 canonical state 送進 prompt。
- 遊戲 NPC、互動敘事、卡牌／任務生成器：限制輸出長度與 JSON schema，避免讓模型直接控制遊戲規則。

**不要誤用 [工程建議]**

- 不要每 100ms 對 partial transcript 呼叫一次。應採 final chunk + silence/debounce + Help 按鈕。
- 不要塞完整 30 分鐘逐字稿。官方 Prompt API input 上限為 4,000 tokens；自己的 canonical state、摘要與最近 2–6 句才是正確的 memory architecture。
- 不要把 LLM JSON 當資料庫真相。先 schema validate，產生 patch，然後由 app merge 到 authoritative state。
- 不要假定所有 Pixel 11 Pro 都有相同 model / feature tier；啟動時記錄 AICore、SDK、build fingerprint、`checkStatus()` 與 base model name。

### 2. 專用文字 API：比自由 prompt 更穩的快捷能力

ML Kit 現有的 GenAI 專用 API 包含：

- **Summarization API（Beta）**：摘要 notes、會議／語音段落、閱讀佇列；適合多層摘要（chunk → session → project）。
- **Proofreading API（Beta）**：找出 spelling、grammar、style 類問題；可做離線草稿校對，而不是把私人文字送到雲端。
- **Rewriting API（Beta）**：改寫語氣、長短、表達方式；適合一鍵把口述稿變成較易讀的草稿。
- **Image Description API（Beta）**：為使用者選的照片產生描述；可作相簿搜索輔助、無障礙 alt text 草稿、旅程／故事素材索引。

它們的優點是 task contract 比 open prompt 小、UI 更容易做得可靠；缺點是可塑性較低，**不可因為名字看起來適合就跳過 `checkStatus()` 和實機品質測試**。

### 3. GenAI Speech Recognition API：最有立即價值的語音入口

**[已實機確認]** Jones 已在 Pixel 11 Pro 使用本地離線繁體中文語音輸入，且回報準確度很強。這證明這台手機與目前下載的語音模型組合值得深入開發。

ML Kit 的 **GenAI Speech Recognition API（Alpha）** 提供 microphone／streaming audio、continuous result flow 與 partial → final 修正的模式；Basic 使用 on-device speech model，Advanced 使用 Gemini-enhanced model。官方文件列出 `cmn-Hant-TW`，但標記與 device support 仍可能隨版本改動。

最值得做的不是「無限長的一次 STT」，而是：

```text
Microphone → VAD / silence boundary → partial subtitle
           → final chunk (with timestamp)
           → local persistent transcript
           → optional Nano state patch / summary / suggestion
```

這能同時服務：JoviYarn、語音日記、走路時的點子收集、工作會議／vibe coding session capture、旅程紀錄與無障礙輸入。

**Fallback：**Android platform 有 `SpeechRecognizer.isOnDeviceRecognitionAvailable()` 與 `createOnDeviceSpeechRecognizer()`。普通 `SpeechRecognizer` 可能把 audio streaming 給遠端服務，而且 Android 明說它不適合無限 continuous recognition；不要只設 `EXTRA_PREFER_OFFLINE` 就宣稱完全離線。

### 4. AICore 的操作邊界：把它當 shared system resource

- App 先跑 `checkStatus()`；可能是 `AVAILABLE`、可下載、下載中或 unavailable。availability UI 是產品的一部分。
- 初次 AICore configuration／model download 或 clear data 後，需要網路，並可能花數分鐘到數小時；「已 provisioned 後的 airplane mode」與「新機第一次就 airplane mode」是兩個不同測試。
- inference 要在前景；背景／foreground service 不能被當成持續 Nano worker。
- 過密 requests 可得到 `BUSY`；用 debounce、single-flight、取消過時請求、exponential backoff。文件也提到 per-app battery quota，但沒有公開可拿來設計的數字。
- `warmup()` 可降低第一個 Help 提示的等待；應測 cold/warm/P50/P95，而不是只記一次最漂亮成績。

---

## B. 不該被 Nano 光環掩蓋的 on-device ML Kit

這些 API 不需要把每件事都交給 LLM。它們通常是 Pixel 11 Pro 上最容易快速做出好產品體驗的能力。

### Vision：讓相機、截圖與相簿變成結構化輸入

| API | 可做什麼 | 值得做的 App 點子 | 注意事項 |
|---|---|---|---|
| Text Recognition v2 | 從 image / camera frame 做 OCR；支援 Chinese script | 書籍／白板／菜單／收據／螢幕截圖轉筆記；JoviYarn 的實體故事卡 | OCR 不是文件語意理解；後續可接 Nano 做結構化 |
| Document Scanner | 系統式掃描、裁切、透視校正與 PDF/JPEG 結果 | 無紙化 capture、故事素材／手寫草圖 archive | 輸出後再做 OCR／分類；確認 Google Play services 依賴 |
| Barcode Scanning | QR／一維碼掃描 | 實體物件、書籍、展覽、收藏品連到個人知識卡 | 不等於產品／書籍資料庫；資料 lookup 是另一層 |
| Image Labeling | 通用影像 label | 相簿粗索引、素材篩選、照片 story seed | label 粗粒度，不應當細節事實 |
| Object Detection & Tracking | 偵測／追蹤 objects | 鏡頭式互動、創作素材 capture、遊戲／實驗原型 | 持續相機 pipeline 必須重視耗電與 frame throttling |
| Face Detection | 臉部輪廓、特徵、表情相關 signals | 自拍構圖、表情驅動小互動、私密本地 photo tooling | 不是身分辨識；不要把它宣稱為人臉辨識或情緒真相 |
| Pose Detection | 人體姿勢 landmark | 動作遊戲、跳舞／復健輔助、角色 reference capture | 健康／安全情境不應只靠單一模型判斷 |
| Selfie Segmentation | 前景／背景 mask | 創作、虛擬攝影棚、角色拼貼、短影音特效 | 非 generative image editing；是 mask 基礎能力 |

**高槓桿串法：**`CameraX frame → ML Kit OCR/vision → 精簡 facts → Nano Prompt → UI action`。例如掃一本兒童書，OCR 只擷取頁面文字，Nano 只負責提出下一個互動問題；兩者各做擅長的事。

### Language：離線語言產品的實用底座

- **Language Identification**：先判斷文字語言，作為 route／UI 提示，適合中英混雜的 inbox 或 voice transcript 後處理。
- **On-device Translation**：ML Kit translation 的模型可按需下載；適合繁中、英文、日文、韓文等短句翻譯與旅遊工具。它是 machine translation，不是語境理解／寫作老師。
- **Smart Reply**：對支援的語言／情境產生短回覆建議；適合低風險的 personal draft UI。對繁中與自身 domain 的品質，務必先測，不要把它做成自動送出。
- **Digital Ink Recognition**：筆畫／手寫輸入轉文字或 gesture，適合 Pixel 觸控筆／手寫式 story map、練字、數學／符號輸入（語言與 model availability 需查）。

### 一個很強的「離線創作工作台」組合

```text
相機/截圖 ──→ OCR / document scanner ─┐
語音 ───────→ local STT ──────────────┼→ encrypted local notebook
手寫 ───────→ digital ink ────────────┘         ↓
                                     language ID / translation
                                                 ↓
                              Gemini Nano：摘要、整理、三個下一步
                                                 ↓
                              TTS / share sheet / export（由使用者決定）
```

這是比「聊天機器人」更像工具的方向：原始內容與可驗證 facts 先由 deterministic pipeline 保存，Nano 只負責輕量的語意變形和發想。

---

## C. 想更自由：MediaPipe、LiteRT 與自帶模型

### MediaPipe Tasks

MediaPipe 是 Google AI Edge 的跨平台 task framework，適合 Android 上的 image、video、audio、text 任務與客製 pipeline。常見可探索方向包括 face landmarker、hand／pose landmarker、gesture recognizer、image classifier、object detector、image segmenter、audio classifier、text classifier 與 embedding／retrieval 類工作流。

**為何值得 Pixel 11 Pro 開發者使用？**因為它不受 AICore prompt contract 限制。你可選模型、量化、更新策略與輸入前處理，並在 GPU／CPU／加速器路徑上實測；代價是自己要處理 model licensing、下載、版本、benchmark、memory、battery 與 fallback。

### LiteRT（原 TensorFlow Lite）

LiteRT 是端上模型 runtime。當功能可由小型 classifier、embedding model 或 segmenter 解決時，LiteRT 往往比 Nano prompt 更快、更可控、更容易測。例如：

- 個人 notes 的 semantic embedding 與本地相似段落搜尋。
- 影像風格／品質分類、照片去重候選、短音訊事件偵測。
- 明確 schema 的 intent classifier（如「開始錄音／停止／Help／新增角色」）。
- 對固定資料集的 object／scene classifier。

### LiteRT-LM / Gemma 與 open-weight LLM

若日後要控制完整模型版本、context、function calling 或模型個性，可研究 **LiteRT-LM** 與 Gemma／其他具手機可行性的量化 open-weight model。這條路是「模型隨 App 或按需下載」，不是 AICore shared Nano；因此能避開部分 AICore feature gating，卻會帶來數 GB 級模型下載、RAM／thermal、token rate、app storage、模型授權與安全更新等責任。

**策略建議：**先把 AICore Nano 做成最快 PoC；若 JoviYarn 日後需要更長 context、可重現的版本、離線世界觀模型或跨 Android OEM，才進入 LiteRT-LM spike。不要一開始就為了「完全自主」背負模型配送工程。

---

## D. 系統能力與 consumer AI：可以善用，但不要假裝有 SDK

### 可從 Android app 正常利用的能力

- **CameraX + ImageAnalysis**：把 Pixel 相機輸入安全、節流地交給 ML Kit / MediaPipe。
- **AudioRecord / Oboe / audio effects**：建立低延遲錄音、VAD、波形、播放與分段，不應把聲學處理一律稱為 AI。
- **TextToSpeech**：把 app 文字朗讀出來；離線 voice、繁中品質與下載狀態依 Android TTS engine／voice 而定，應用 runtime check 與設定頁說明。
- **Health Connect、sensors、location、Bluetooth**：不是 AI，但可產生有意義且經使用者授權的 context，交給本地 deterministic rules 或 Nano 做短解釋。健康結論不可由 generative output 直接做診斷。
- **Accessibility semantics**：用 Compose / View 的正確 content descriptions、live regions、large text，讓 TalkBack 等系統功能發揮；不需要等待 AI 才能改善可用性。

### 應視為「沒有第三方呼叫承諾」的 Pixel consumer features

截至本報告，對下列類型功能，若沒有明確 Android／Google developer API，請不要把它寫入 app spec：Gemini app 的 agentic actions、Pixel Screenshots 的私人索引、Call Screen／Hold for Me、Pixel Studio／各種系統圖像生成與編輯、Magic Cue、Recorder 的系統摘要、鍵盤 voice typing、Live Translate 的特定 UX 等。

它們可作為 **產品靈感** 或使用者的系統層輔助，但不是 app backend。最健康的設計是：App 自己提供資料輸入／輸出與離線核心；使用者若另外選用 Pixel system feature，則是 bonus。

---

## E. Pixel 11 Pro 專屬探索清單：由低風險到高野心

### 立即可做（1–3 天）

1. **Local Voice Notebook**：繁中／英文 speech chunk、local transcript、手動按鈕產生 3 個標籤／下一步；不做自動送出或雲端同步。
2. **JoviYarn Nano benchmark harness**：對固定繁中故事 fixtures 跑 state extraction、suggestions、clean transcript；記錄 model、warm/cold、input/output tokens、P50/P95、錯誤與電量。
3. **Screenshot-to-action board**：share screenshot → OCR → 讓使用者勾選文字 → Nano 轉成 todo／問題／研究線索。保留原截圖與來源頁面，避免幻覺覆蓋證據。
4. **Photo story seed**：相簿選圖 → image description／vision labels → 產生可編輯的情境、角色與三句開頭。
5. **中英語言切換實驗**：用 `cmn-Hant-TW`、`en-US`、中英 code-switch 的真實語料，分別量 partial latency、final word error、標點、proper nouns、續錄穩定性。

### 值得做成可用工具（1–2 週）

6. **Story State Console（JoviYarn 的真正底座）**：timeline、characters、locations、objects、relationships、open threads 都由 deterministic data model 保存；Nano 僅提 patch 建議；每個 patch 可接受／拒絕／回復。
7. **Personal research intake**：相機／語音／share sheet 累積原始素材，OCR+摘要協助進 inbox；永遠能回到 source。
8. **Offline travel／生活 phrase companion**：local STT + translation + TTS + favorites；將生成式解釋設為 optional，不讓它成為翻譯 correctness 的單點。
9. **Creative camera toolkit**：MediaPipe segmentation／pose／hand gesture + CameraX；先做 non-generative mask、gesture trigger 和 timeline，再決定是否接 image model。
10. **Private accessibility assistant**：離線讀圖描述、OCR 大字朗讀、語音轉記事；採本地優先，明確顯示模型尚無法保證所有圖片／文字正確。

### 長線研究（先 spike，後承諾）

11. **本地 personal semantic memory**：LiteRT embedding + encrypted vector index + Nano 產生「可引用來源」的回答。核心問題是 retrieval quality 與 consent，不是聊天泡泡。
12. **Multimodal story world**：相片／草圖／語音／手寫統一為 story assets，vision 先出 facts，Nano 再建議故事連結。
13. **On-device agent UI**：Nano 只輸出嚴格 intent schema；app 限定 allowlist 工具、預覽 impact、由使用者確認。不要把自由文字直接轉成 OS 或網路操作。
14. **自帶小模型的跨機型 fallback**：把最核心、明確的分類／embedding 能力搬到 LiteRT，讓未來不只依賴 Pixel/AICore。

---

## F. 一套實際可維護的 Pixel AI app 架構

```text
                    ┌───────────────────────────────┐
                    │ UI / consent / status dashboard │
                    └──────────────┬────────────────┘
                                   │
   raw inputs ──────→ local store / encrypted files / source IDs
  mic, camera, ink                 │
                                   ├── ML Kit / MediaPipe facts
                                   │   OCR, language, landmarks, labels
                                   │
                                   ├── deterministic domain state
                                   │   StoryState, tasks, citations, timeline
                                   │
                                   └── AICore Nano (optional enhancement)
                                       patch / summary / rewrite / suggestions
                                                    │
                                      validate → preview → user accepts
```

這個架構的關鍵是 **graceful degradation**：

- Nano unavailable / busy：仍可錄音、OCR、保存 state、用 template suggestion。
- speech model 未下載：顯示 setup，而不是假裝 fully offline。
- prompt parse 失敗：保留原文與 retry，不破壞 canonical data。
- app 在背景：停止 Nano job，恢復時再重新檢查狀態。
- 使用者要 export／share：預設明確選擇，避免把「本地 AI」做成暗中外傳內容。

---

## G. 測試與採購後的真實驗收標準

Pixel 11 Pro 的價值應用數據驗證，而不是 marketing claim。建議建立 `pixel-ai-lab`（可先是 JoviYarn 的 debug screen）並保存以下欄位：

| 範圍 | 必測項目 |
|---|---|
| 環境 | build fingerprint、Android version、Google Play system update、AICore version、ML Kit dependency、base model name |
| GenAI | `checkStatus()`、feature availability、cold/warm、stream/complete time、input/output token、BUSY／quota／parse failures |
| STT | locale、partial latency、final latency、標點、proper noun、中英切換、30 分鐘 reopen／error 次數 |
| 離線 | 已 provisioned airplane mode；新／reset 狀態沒有網路；語音／Nano 分開記錄 |
| 視覺 | 每秒 frames、熱、耗電、低光／動態模糊、錯誤型態 |
| 品質 | 固定 fixture + 人工 rubric：格式、事實保留、角色一致性、繁中自然度、危險幻覺 |
| 隱私 | 哪些 bytes 永不離機、哪些 action 會 share、log 是否含原文／audio、如何刪除 |

**不要用一次成功作結論。**至少做冷啟動 10 次、warm 30 次、airplane mode、螢幕關閉／背景切換、20–30 分鐘 sustained session；再決定哪一條能力可以從「有趣 demo」升為產品依賴。

---

## H. JoviYarn 的特別建議

JoviYarn 最好的第一版不是「整個故事都交給 Nano」，而是：

1. 本地 STT 產生可回看、可修正的 transcript chunk。
2. App 保存 canonical Story State 與 immutable event log。
3. 每個 final chunk 的 debounce 只請 Nano 提一個 **small patch**。
4. 使用者按 Help 才生成三個非常短、可點選的 suggestions。
5. 選擇 suggestion 後，由 app 寫入 event；Nano 不直接重寫世界狀態。
6. clean transcript／ending 用專用 API 或短 prompt，在使用者明確觸發時做。

這讓 Pixel 11 Pro 的本地 AI 成為一個敏捷、私密、可中斷的 co-pilot，而不是不可追溯的 story authority。它也同時保留未來換成 cloud model、LiteRT-LM 或其他 OEM device 的彈性。

---

## I. 版本／支援狀態注意事項

前一份報告（[Pixel 11 on-device AI 與 JoviYarn Android PoC](../pixel-11-on-device-ai-for-joviyarn-2026-08-22/README.md)）忠實記錄 2026-08-22 的 developer-facing device matrix gap：官方文件當時沒有將 Pixel 11 列為 Prompt API 明確 support target。今天這份報告不以「文件已同步」取代那項歷史事實；而是加入 Jones 的 **實機成功回報**，並把每個 feature 的 `checkStatus()` / compatibility test 視為現況真相。

ML Kit GenAI 的 Beta／Alpha、AICore configuration、Google Play system update 與 Pixel build 都可能改變可用 feature。報告中的構想可以穩定保存，support claim 則必須連同版本與測試日期保存。

## 官方來源與延伸閱讀

1. [ML Kit GenAI API overview、device/availability、quota 與前景限制](https://developers.google.com/ml-kit/genai)
2. [GenAI Prompt API overview](https://developers.google.com/ml-kit/genai/prompt/android)
3. [Prompt API get started：status、download、4,000-token input、warmup、streaming](https://developers.google.com/ml-kit/genai/prompt/android/get-started)
4. [Prompt structured output（Alpha）](https://developers.google.com/ml-kit/genai/prompt/android/structured-output)
5. [Prompt system instructions（Beta）](https://developers.google.com/ml-kit/genai/prompt/android/system-instructions)
6. [Prompt prefix caching（Experimental）](https://developers.google.com/ml-kit/genai/prompt/android/prefix-caching)
7. [ML Kit GenAI Summarization](https://developers.google.com/ml-kit/genai/summarization/android)
8. [ML Kit GenAI Proofreading](https://developers.google.com/ml-kit/genai/proofreading/android)
9. [ML Kit GenAI Rewriting](https://developers.google.com/ml-kit/genai/rewriting/android)
10. [ML Kit GenAI Image Description](https://developers.google.com/ml-kit/genai/image-description/android)
11. [ML Kit GenAI Speech Recognition（Alpha）](https://developers.google.com/ml-kit/genai/speech-recognition/android)
12. [Android `SpeechRecognizer` reference](https://developer.android.com/reference/android/speech/SpeechRecognizer)
13. [ML Kit Vision APIs](https://developers.google.com/ml-kit/vision)
14. [ML Kit Text Recognition v2 for Android](https://developers.google.com/ml-kit/vision/text-recognition/v2/android)
15. [ML Kit Language APIs](https://developers.google.com/ml-kit/language)
16. [ML Kit on-device Translation](https://developers.google.com/ml-kit/language/translation/android)
17. [Google AI Edge](https://ai.google.dev/edge)、[MediaPipe](https://ai.google.dev/edge/mediapipe/solutions/guide)、[LiteRT](https://ai.google.dev/edge/litert)
18. [Android AI on Android developer hub](https://developer.android.com/ai)
19. [Official ML Kit Android GenAI samples](https://github.com/googlesamples/mlkit/tree/master/android/genai)

## 最終判斷

**很值得把 Pixel 11 Pro 當成 Jones 的 local-AI reference lab。**已經證實繁中離線語音與最新本地模型能實際跑順，這比紙面規格更有價值。最有前途的不是追逐所有 Pixel consumer AI，而是先建立：**語音／視覺原始輸入 → deterministic local state → Nano 小型語意 enhancement → 使用者可驗證輸出**。

這條路能最快做出 JoviYarn，也會自然長出更多私密、離線、可控的創作與個人工具。

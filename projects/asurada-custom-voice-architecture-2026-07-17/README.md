# Asurada 的自訂聲音：Gemini Live 能否替換為自訓／克隆 TTS？

> 研究日期：2026-07-17  
> 研究對象：[`jonesandjay123/asurada`](https://github.com/jonesandjay123/asurada) `8d86339`（已在本機 `git pull`／clone 後檢閱）  
> 結論狀態：**可做，但不能把自訓聲音直接嵌入 Gemini Live 的內建語音。**要取得真正的自訂音色，必須選擇「接管輸出播放」或「把語音管線拆開」其中一條路。

## 先給決策結論

Asurada 現在的優勢是：Firebase AI Logic 的 Gemini Live native-audio 模型直接處理即時收音、VAD、雙向音訊、打斷與播放。它不是「Gemini 產生文字，再由 App 呼叫某個可替換 TTS」的架構。

因此問題的答案分成三層：

| 想要的結果 | 可不可行 | 最合適的路 | 代價 |
| --- | --- | --- | --- |
| 換一個較適合 Asurada 的聲音，但不要求專屬音色 | 可以，且現在就能做 | Gemini Live 的 `speechConfig` 選 Chirp 3 預設 voice | 只有 Google 提供的固定 voice 名單，不是自訓／克隆 |
| 讓 Asurada 使用一個取得本人同意的專屬聲音 | 可以 | 以自訂 TTS 取代「播放 Gemini 音訊」的那一段 | 需接管音訊輸出、處理文字增量與打斷；會增加延遲與維運 |
| 聲音和情緒仍保留 Gemini Live 的節奏，只改變音色 | 技術上可以做 POC，但不適合作為第一個正式方案 | 即時 voice conversion（Gemini PCM → VC 模型） | GPU、串流 buffer、失真與打斷處理都很難；也不是真正的 TTS 替換 |
| 完全掌控／自己持有模型權重 | 可以，但等於重建語音層 | 自架 streaming STT + Gemini 文字推理 + 自架／自訓 TTS | 失去 Gemini Live 的整合便利；工程與運維最多 |

**推薦順序：**

1. 先用官方 `speechConfig` A/B 聽 3–5 個預設 voice，確認產品真正缺的是「角色感」還是「專屬音色」。
2. 若確定要專屬聲音，先做一個很小的 *output-replacement POC*：保留 Gemini Live 做理解與工具，停止播放它回傳的 PCM，改播雲端 clone TTS 的串流音訊。
3. POC 通過延遲、繁中品質、打斷與成本 gate 後，再決定採 Google Instant Custom Voice、商用 TTS，或自架權重；**不要先訓練一個大模型**。

## 1. Asurada 現在到底怎麼串 Gemini Live

### 1.1 實際程式證據

目前 App 是 Kotlin + Jetpack Compose；語音核心是 Firebase AI Logic SDK，而不是自建 STT／TTS server。

- Gradle 引入 `com.google.firebase:firebase-ai`，未引入任何 Cloud TTS、ElevenLabs 或本機推理 SDK：[`app/build.gradle.kts#L37-L61`](https://github.com/jonesandjay123/asurada/blob/8d86339f0f29d92f0ee60df1568d292753af7b15/app/build.gradle.kts#L37-L61)。
- Firebase BoM 固定為 `34.15.0`：[`gradle/libs.versions.toml#L1-L39`](https://github.com/jonesandjay123/asurada/blob/8d86339f0f29d92f0ee60df1568d292753af7b15/gradle/libs.versions.toml#L1-L39)。
- `FirebaseLiveSessionConnector` 以 `GenerativeBackend.googleAI()` 建立 Live model，指定 `gemini-2.5-flash-native-audio-preview-12-2025`，且強制 `ResponseModality.AUDIO`：[`FirebaseLiveSessionController.kt#L658-L667`](https://github.com/jonesandjay123/asurada/blob/8d86339f0f29d92f0ee60df1568d292753af7b15/app/src/main/java/com/joneslab/asurada/session/FirebaseLiveSessionController.kt#L658-L667)。
- 它再呼叫 `liveModel.connect()` 開 WebSocket session：[`#L684-L686`](https://github.com/jonesandjay123/asurada/blob/8d86339f0f29d92f0ee60df1568d292753af7b15/app/src/main/java/com/joneslab/asurada/session/FirebaseLiveSessionController.kt#L684-L686)。
- 每次說話走 SDK 的 `startAudioConversation()`；目前程式只處理 tool callback 和 input/output transcription callback：[`#L694-L717`](https://github.com/jonesandjay123/asurada/blob/8d86339f0f29d92f0ee60df1568d292753af7b15/app/src/main/java/com/joneslab/asurada/session/FirebaseLiveSessionController.kt#L694-L717)。

換成資料流就是：

```text
Pixel 麥克風
  └─ Firebase SDK startAudioConversation()
       └─ Gemini Live native-audio WebSocket
            ├─ 模型輸入理解、VAD、對話與工具判斷
            ├─ 原生音訊回覆（24 kHz PCM）→ SDK 播放
            └─ input/output transcription delta → Asurada transcript UI
```

現行 Controller 還每 3 秒檢查 `isAudioConversationActive()`／`isClosed()`，並最多重連三次；這是原生 Live 音訊 UX 的一部分，不能把它誤認成一般文字聊天的 TTS callback。

### 1.2 Gemini Live 在此架構中負責什麼

Gemini Live 不是單純「腦」。它同時給出：

- 麥克風 streaming 與模型端 VAD；
- native-audio 的語意、節奏、情緒與語音回覆；
- 可被使用者打斷的雙向 live session；
- input/output audio transcription；
- function call 提案，Asurada 再在 Android 端白名單執行並回傳結果。

Firebase 官方文件確認：native-audio Live model 需要音訊輸入，並固定回音訊；輸入為 16-bit 16 kHz PCM、輸出為 16-bit 24 kHz PCM。這也是為什麼「只把聲音模型當作可換 speaker」不是一個現成的設定。

來源：[Firebase AI Logic — Live API capabilities](https://firebase.google.com/docs/ai-logic/live-api/capabilities)、[Live API 入門](https://firebase.google.com/docs/ai-logic/live-api)。

## 2. 最重要的邊界：可以選內建 voice，不能把自訓 voice 注入 Live API

Firebase 的 Live API 可在 `liveGenerationConfig` 設定：

```kotlin
speechConfig = SpeechConfig(voice = Voice("Aoede"))
```

這會讓 Gemini Live 的語音回覆採用指定的 **Chirp 3 HD 預設 voice**。官方列出的可選 voice 為 Puck、Aoede、Kore、Zephyr 等固定名字；未指定時預設為 Puck。

但官方 API 的設定形狀是 `SpeechConfig(Voice("VOICE_NAME"))`，不是 reference audio、voice-cloning key、adapter、fine-tune checkpoint 或自訂模型 URL。**所以截至本次查閱，Gemini Live 沒有「帶入自己訓練好的 voice」的 API surface。**

這很容易混淆，因為 Gemini Live 的回覆聲音也使用 Chirp 3 技術；但「使用 Chirp 的預設 voice」不等於「可以將 Cloud TTS 的 custom voice 掛進 Gemini Live session」。兩者是不同 API 與授權／資料路徑。

來源：[Firebase — Live API configuration：response voice](https://firebase.google.com/docs/ai-logic/live-api/configuration)、[Cloud TTS — Chirp 3 HD voices](https://cloud.google.com/text-to-speech/docs/chirp3-hd)。

### 最小、零架構風險的改善

在既有 `FirebaseLiveSessionConnector` 補上 `speechConfig` 即可試聽；不用改音訊 lifecycle，也不用新增 server。

```kotlin
generationConfig = liveGenerationConfig {
    responseModality = ResponseModality.AUDIO
    speechConfig = SpeechConfig(voice = Voice("Aoede"))
    inputAudioTranscription = AudioTranscriptionConfig()
    outputAudioTranscription = AudioTranscriptionConfig()
}
```

這應是未來任何自訓計畫前的 baseline 實驗。不過它只能換人格感，不能創造唯一、可擁有的聲紋。

## 3. 三條真正能做出「專屬聲音」的技術路線

### 路線 A：保留 Gemini Live，接管播放，改以 custom TTS 輸出

這是最符合「Gemini 當即時大腦、聲音由我們決定」的方案。

```text
Mic ──► Gemini Live（理解、對話、tools）
              ├─ Native PCM：收到但不播放
              └─ Output transcription delta
                       ↓
                  turn / chunk assembler
                       ↓
               Custom TTS（clone 或自訓）
                       ↓
                  PCM queue → Android AudioTrack
```

#### 需要改什麼

1. 不再讓 `startAudioConversation()` 自動取得並播放最終音訊；改採 SDK 可自行處理收音／`receive()` 的低階串流路徑，或在官方 SDK 支援的層級接管 output PCM。
2. 保留 Gemini 回傳的 output transcription，但將其視為 **增量資料**，不是已完成、穩定的一句話。現有 repo 也已明確記錄 SDK callback 沒有可靠 partial/final/turn ID。
3. 做 chunk assembler：以標點、最大字數、短 timeout 為條件送入 TTS；同時要避免把同一 delta 重播、把後續修正重播，或在使用者打斷後繼續播舊 queue。
4. 建一個 `SpeechOutputEngine` 介面（例如 `GeminiNativeOutput`、`CloudCloneOutput`），讓 UI/lifecycle 不直接綁死某個 TTS。
5. TTS 金鑰、voice clone key、用量計費與審計必須走受控後端；不要放進 APK。

#### 它能得到什麼、會失去什麼

| 得到 | 失去／新增成本 |
| --- | --- |
| Gemini 的理解、工具呼叫、對話連續性仍在 | 不能再把 SDK 的 playback 當黑盒子 |
| 可用克隆或自架聲音 | 多一次 model/TTS 呼叫與 jitter buffer，首音延遲上升 |
| 可替換供應商，不被單一 voice 綁死 | 需自行正確處理 barge-in、取消與音訊 queue |
| 可針對角色控制音色 | Gemini 原生音訊的韻律和 TTS 的韻律不再完全一致 |

#### 最大風險

這條路是**可行但有工程摩擦**，因為 Gemini native-audio 的 output transcript 是與音訊一起 streamed 的文字觀測值；它不是為外部 TTS 提供的「終稿文字 API」。若要維持自然打斷感，必須實機量測而不是只看文件。

**適用時機：**專屬角色聲線是產品核心，且可以接受先做 1–2 週 POC。

### 路線 B：徹底拆成 STT → Gemini 文字推理 → custom TTS

這是最乾淨、最可控的長期架構；也是最不像「保留現成 Gemini Live voice UX」的方案。

```text
Android microphone
  → streaming STT / VAD
  → Gemini text model 或受控 LLM gateway（tools 仍可保留）
  → streaming custom TTS
  → Android playback
```

優點是所有文字、turn boundary、輸出語音、審計、voice switch 都由自己控制。若 TTS 供應商或模型變動，只換 `TTS provider`。

但代價非常實際：要重新擁有 VAD、ASR partial/final、打斷、雙工、TTS cancel、endpointing、網路退避與全鏈路 observability。這些正是 Asurada 目前有意避免自行重做的部分。

**適用時機：**自訂聲音、資料主權、跨平台和可審計文字記錄比「最快做出自然 Live 對話」更重要。

### 路線 C：即時 voice conversion（把 Gemini Live PCM 轉成另一個音色）

```text
Gemini Live 24 kHz PCM
  → low-latency voice conversion model
  → converted PCM → Android playback
```

它不像 TTS 重念文字，而是保留原先說話的語速、停頓、笑聲／情緒，主要改 timbre。因此從概念上最像題目中的「套殼」。

但它有四個現實問題：

1. 每個短 PCM chunk 都要經 GPU 推理與 buffer，延遲和卡頓很容易使對話變得不自然。
2. 跨語言、快速語速、笑聲、低音量、手機喇叭回授都可能讓音色失真或產生金屬感。
3. 必須在使用者插話時立即 cancel VC inference 並清掉播放 queue；錯一次就會出現「已被打斷還在講」的嚴重 UX 問題。
4. 有些 VC 模型的授權、訓練資料及聲音權利風險不適合直接放入產品。

**判斷：**可做研究 prototype；不建議拿它當 Asurada 第一個 production custom-voice 計畫。

## 4. Custom TTS 的供應／模型選擇

### 4.1 Google Cloud Text-to-Speech：Chirp 3 Instant Custom Voice

這是目前與 Gemini／Firebase 生態最順的「有聲音本人同意」方案，但它**不是 Live API 的內建 setting**。

- 以一段最長 10 秒的 reference audio 加上同一位聲音擁有者錄製的固定 consent statement，建立 `voiceCloningKey`。
- key 送到 Cloud TTS 的 `text:synthesize`，輸入文字、輸出自訂音色的語音。
- 支援 streaming 和 batch；輸出可為 LINEAR16、PCM、OGG_OPUS 等。
- 官方在文件中要求使用提供的 consent script，不能自行改寫。這不是形式問題，而是這條路的倫理與法律 gate。
- 文件列出中文為 `cmn-CN`；**沒有把台灣國語／繁中列成 Instant Custom Voice 的獨立支援 locale**。若 Asurada 主要使用繁中，必須用實際台詞試音，不能直接假設口音、台灣地名和 code-switch 表現合格。

對 Asurada 而言，建議透過 own backend proxy 取得短命 token／執行 synthesize，而不是讓 Android App 持有可濫用的 GCP 身分或 voice-cloning key。

來源：[Cloud TTS — Chirp 3 Instant Custom Voice](https://cloud.google.com/text-to-speech/docs/chirp3-instant-custom-voice)、[Cloud TTS — bidirectional streaming](https://cloud.google.com/text-to-speech/docs/create-audio-text-streaming)。

### 4.2 商用 managed voice-cloning API

例如 ElevenLabs、Cartesia、PlayHT、Resemble 等服務通常提供低延遲／串流／voice-cloning 產品。這類方案的價值在於：最快取得高品質、情緒較完整的專屬聲線，且不必維護 GPU。

選擇時不要只比「像不像」：

- 是否有明確的 speaker consent／voice verification；
- 是否可用 WebSocket / streaming，首音與 chunk latency 為多少；
- 是否支援繁中、台灣地名與中英混說；
- 能否即時 cancel、是否能保證資料不被拿去訓練；
- rate limit、月費、超額計價與 API SLA；
- voice asset／衍生音檔是否可刪除、匯出或移轉。

**適合：**想先驗證產品體驗，不想先養 GPU/MLOps。先用 POC 實測，不建議在沒有書面聲音權利的情況下建立 clone。

### 4.3 自架／開放權重 TTS

可考慮的種類包括：

| 類型 | 代表方向 | 優點 | 注意事項 |
| --- | --- | --- | --- |
| Zero-shot voice clone | Fish Speech、XTTS、部分新一代多語 TTS | 不一定需要微調，短 reference 即可測角色聲線 | 品質、延遲、商用授權與繁中表現需逐一驗證 |
| Fine-tune / adapter TTS | 以基礎 TTS 加 LoRA / speaker adapter | 可控制資料、部署與聲音資產 | 需要清理語料、GPU、評測與持續維護，不適合作為第一步 |
| Voice conversion | RVC、Seed-VC 類 | 保留 Gemini 的表演節奏，直接轉 PCM | 即時性與穩定性較差，風險高 |

既有 repo 已有 [Fish-Speech TTS 調查](../fish-speech-tts/)；它可做「自架探索」的候選，但該報告日期較早，且其授權與硬體假設要以當下 upstream LICENSE／release 再核對。不能因為模型可下載，就假定可以合法商用、在 Mac mini 上達到手機即時對話品質。

**硬體判斷：**Mac mini M4 可用於閱讀、控制、局部試驗；不是可靠的 production realtime speech GPU。要把高品質自架 TTS 接到對話，通常要 Linux + NVIDIA GPU（或合規 GPU 雲端），還要加 WebSocket gateway、autoscaling、rate limit、監控與音檔保護。

## 5. 「自己訓練」其實有三種不同難度

不要把它們混為一談：

| 方法 | 需要的資料 | 產物 | 何時選 |
| --- | --- | --- | --- |
| 預設 voice 選擇 | 0 | 無；僅設定名稱 | 只想改善聲音人格 |
| Zero-shot / Instant clone | 約 10–60 秒乾淨 reference + 明確同意 | clone key 或 speaker conditioning | 最快驗證專屬音色 |
| Fine-tune / LoRA speaker | 通常需更長、逐字對齊的乾淨語料 | adapter / checkpoint | 要高度一致、可控且自己持有模型能力 |
| 從零訓練 TTS foundation model | 大量、多語、多說話者資料與大量 GPU | 完整基礎模型 | 不合理；不應是 Asurada 的計畫 |

對 Asurada，合理的上限通常是第二或第三列。從零訓練不會只解決「換一個聲音」，反而會引入資料取得、人格權、模型偏差、安全濫用、MLOps、推理成本與長期版本升級問題。

## 6. 建議的 POC：先證明體驗，再碰自訓

### Phase 0：官方 voice baseline（0.5–1 天）

在現有 connector 加 `speechConfig`，從官方 voice 中選 3–5 種候選。用一組固定繁中／英語混用語料測：

- 角色感與可理解度；
- 台灣地名、日期、英文產品名；
- 首次回音、可打斷、靜音／背景／重連；
- 與現有 Puck 的偏好盲測。

這一階段不動資料、工具或音訊架構。

### Phase 1：Custom output spike（3–5 天）

目標不是完成產品，而是回答一個問題：**當 Gemini Live 的輸出改由 custom TTS 發聲，使用者還覺得像即時助手嗎？**

最小內容：

1. 只用 20 句固定 scenario，暫不開工具呼叫。
2. 自行取得 output audio／transcription stream，停止播放 Gemini PCM。
3. 用一個雲端 clone TTS 做輸出；先由 Google ICV 或一個 managed provider 擇一。
4. 以標點 + 最長等待窗切 chunk；每段記錄「Gemini output text 收到」到「第一個自訂 PCM 播放」的時間。
5. 實測使用者插話：播放中的 TTS 是否在 150–250 ms 內停止，舊句是否會漏播。

通過 gate：

- 使用者覺得沒有明顯「先想很久、再念稿」；
- 20 句中無重播、漏播、斷字、打斷後殘音；
- 繁中和中英混說可接受；
- 每分鐘成本、錯誤率、網路斷線行為可觀測；
- 取得聲音本人明確、可撤回的授權紀錄。

若失敗，停在官方預設 voice；不要把脆弱的 audio-transcript relay 推進日常助手。

### Phase 2：產品化抽象（1–2 週）

只有 Phase 1 成功才做：

```kotlin
interface SpeechOutputEngine {
    suspend fun enqueue(text: String, segmentId: String)
    fun interrupt(reason: InterruptReason)
    fun stop()
    val state: StateFlow<SpeechOutputState>
}
```

至少有：

- `GeminiNativeSpeechOutput`：目前行為，作為可靠 fallback；
- `CustomTtsSpeechOutput`：雲端 clone／自架 TTS；
- session generation／segment id，保證舊 callback 不會在新 session 發聲；
- bounded queue、timeout、cancellation、retry、observability；
- Remote Config kill switch，隨時退回 native voice。

現有 `generation`／`audioGeneration` 的 stale-session 防護可延續到新的 speech segment；不要另起一個沒有世代治理的播放器。

### Phase 3：選擇長期供應方式

- **選 Google Instant Custom Voice**：最快、與現有 Google 身分／帳務接近；以繁中實測結果決定，不先假設 locale 合格。
- **選商用 API**：最快驗證高品質角色聲線；做好供應商切換層與資料條款審查。
- **選自架**：僅在資料主權、離線／私有部署或成本規模足以覆蓋 GPU/MLOps 時進場。先用 zero-shot，再評估 fine-tune；不從零訓練。

## 7. 安全、權利與產品誠信

「聲音模型」本身是高風險資產。Asurada 在這方面應比一般 demo 更嚴格：

- 只克隆 Jones 本人或有可保存的明確授權者；不要用公開影片、電話錄音或第三方音檔當 reference。
- 同意書要描述用途、保存位置、可撤回和刪除流程；Google ICV 已要求固定 consent statement，但這不取代產品自己的權利紀錄。
- clone key、reference audio、TTS API token 都視為敏感資料；不進 Git、不進 APK、不寫入可讀 log。
- App UI 應把自訂聲音明示為合成聲音，避免讓聽者誤以為真人即時發言。
- 為 voice output 建 kill switch 和「回到 Gemini native voice」fallback；若 TTS provider 延遲／故障，不要讓音訊 queue 無限累積。

## 最後建議

Asurada 不該把自己困在 Gemini 的預設聲音；**做專屬聲音是可行的**。但在當前程式架構中，最誠實的說法是：

> 不能把自訓模型「塞進 Gemini Live 的 speaker」。可以把 Gemini Live 的 speaker 拿掉／繞開，再由自己的 custom TTS speaker 接手。

我會先做 `speechConfig` 的官方 voice 聽感測試；如果專屬聲線確實是 Asurada 的核心身份，再以 Cloud TTS Instant Custom Voice 或一個 managed clone API 進行小型輸出替換 POC。這是最小風險、最能驗證真實 UX 的路。自架 fine-tune 是 POC 成功後才值得談的第三步，不是起跑線。

## 來源與可驗證證據

### Asurada 程式碼（固定 commit）

1. [Firebase Live connector、AUDIO modality、transcription、`startAudioConversation()`](https://github.com/jonesandjay123/asurada/blob/8d86339f0f29d92f0ee60df1568d292753af7b15/app/src/main/java/com/joneslab/asurada/session/FirebaseLiveSessionController.kt#L658-L763)
2. [Firebase AI dependency](https://github.com/jonesandjay123/asurada/blob/8d86339f0f29d92f0ee60df1568d292753af7b15/app/build.gradle.kts#L37-L61)
3. [Firebase BoM version](https://github.com/jonesandjay123/asurada/blob/8d86339f0f29d92f0ee60df1568d292753af7b15/gradle/libs.versions.toml#L1-L39)

### 官方文件（2026-07-17 查閱）

1. Firebase AI Logic, [Get started with Gemini Live API](https://firebase.google.com/docs/ai-logic/live-api)
2. Firebase AI Logic, [Live API capabilities（音訊格式與 native-audio model 行為）](https://firebase.google.com/docs/ai-logic/live-api/capabilities)
3. Firebase AI Logic, [Configuration options for the Live API（固定 voice 選擇、transcription、VAD）](https://firebase.google.com/docs/ai-logic/live-api/configuration)
4. Google Cloud Text-to-Speech, [Chirp 3: Instant Custom Voice（consent、reference audio、clone key、synthesis）](https://cloud.google.com/text-to-speech/docs/chirp3-instant-custom-voice)
5. Google Cloud Text-to-Speech, [Bidirectional streaming synthesis](https://cloud.google.com/text-to-speech/docs/create-audio-text-streaming)
6. Google Cloud Text-to-Speech, [Gemini-TTS overview](https://cloud.google.com/text-to-speech/docs/gemini-tts)

> 注意：Firebase AI Logic / Gemini Live 的模型與 SDK 仍有 preview 成分。實作前必須重新對照當時鎖定的 Firebase BoM 與官方 reference，不應把本文的 API 名稱當作永久穩定承諾。

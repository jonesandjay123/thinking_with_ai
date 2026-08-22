# Pixel 11 on-device AI 與 JoviYarn Android PoC 可行性調查

> 查詢日期：2026-08-22（America/New_York）  
> 判準：不是「Pixel 消費者功能有沒有 Gemini」，而是「第三方 Android App 是否可程式化、正式地呼叫」。  
> 狀態標記：**[官方確認]**、**[合理推論]**、**[尚未確認]**、**[第三方觀察]**。

## Executive summary

1. **不要今天為了「已正式保證可跑 JoviYarn Prompt API」而買 Pixel 11。** Pixel 11 官方已確認為 Tensor G6、內含「latest Gemini Nano」，但截至查詢日 ML Kit GenAI 的 *Prompt API supported device matrix* 只列 Pixel 9/9 Pro/9 Pro XL/9 Pro Fold 與 Pixel 10 系列（`nano-v3`），**沒有 Pixel 11**。這是明確的 developer-facing gap，而非可忽略的文字落後。[官方確認]
2. **若 Pixel 11 的 `Generation.getClient().checkStatus()` 是 `AVAILABLE`，JoviYarn 的文字 AI loop 很適合做 PoC：**短 Story State + 最近段落（<4,000 tokens）→ JSON state extraction／3 個短提示；支援 streaming、temperature、topK、seed、candidateCount、maxOutputTokens、system instruction、以及仍屬 experimental 的 prefix cache。[官方確認；Pixel 11 實際 availability 尚未確認]
3. **完全離線可以是「初始化完成後」的目標，不能是首次開箱即保證。** ML Kit 表明 inference/input/output 都在裝置；但 AICore configuration、Nano/語音模型首次下載與剛 reset 後的初始化需要網路，且官方說可能花數分鐘到數小時。[官方確認]
4. **STT 反而較可行：**ML Kit GenAI Speech Recognition（Alpha）的 Basic mode 在 API 31+ 裝置可用、支援 `cmn-Hant-TW`（beta）與 mic streaming/partial results；其 Advanced (Gemini) mode 的正式矩陣僅列 Pixel 10，不列 Pixel 11。Platform `SpeechRecognizer.createOnDeviceSpeechRecognizer()` 也能作 offline fallback，但 Android 明說一般 `SpeechRecognizer` 可能上傳音訊、且不適合 continuous recognition。[官方確認]
5. **Pixel 8 Pro 可先做大部分 app／STT／state schema／UI 工作，但不是 Prompt API target。** 官方 Prompt matrix 沒有 Pixel 8 Pro；即使消費者功能曾使用 Gemini Nano，也不等於第三方 ML Kit Prompt API exposure。[官方確認]

## 先釐清：兩種「有 Gemini Nano」不是同一件事

| 問句 | 2026-08-22 的答案 |
|---|---|
| Pixel 11 是否有 Gemini Nano？ | 是；Google Pixel tech specs 直接列 Gemini Nano，Pixel blog 說 Tensor G6 runs the latest Gemini Nano model。[官方確認] |
| Pixel 11 是否已在 ML Kit Prompt API 正式支援矩陣？ | **否：文件未列。** 矩陣列 Pixel 9/10 為 nano-v3，沒有 Pixel 11。[官方確認] |
| 因而第三方 app 現在是否「一定」可用 Prompt API？ | **不能下此結論。** 實機 `checkStatus()` 才是執行時真相；文件缺列時不可當 production guarantee。[合理推論] |

這也解釋行銷與開發文件不同步的含義：前者證明 consumer system features 有模型；後者才定義 SDK/AICore feature contract、裝置 enablement、版本與支援責任。兩者不是可互相代入的敘述。

## 1. Pixel 11：SoC、Nano 與型號差異

### 已公布硬體

- Pixel 11、Pixel 11 Pro、Pixel 11 Pro XL 均為 **Google Tensor G6**；Google 官方規格也都列出 Gemini Nano。[官方確認]
- Google 官方 Pixel 11 blog：Tensor G6 「runs the latest Gemini Nano model」，並宣稱相對 Tensor G5，on-device AI tasks *up to* 3.5× faster、*up to* 3.5× less energy；這是 Google 的整體 consumer benchmark，並不是第三方 Prompt API TTFT benchmark。[官方確認]
- Google 沒有在該規格／blog 命名 `Gemini Nano v3`、`v4` 或其他確切 version。因此「Pixel 11 = Nano v4」目前沒有官方證據；「= v3」也不能由舊矩陣反推。[尚未確認]
- 正確的 runtime interrogation 是 ML Kit `getBaseModelName()`；它存在正是因為同一 app 在不同 Nano version 可產生不同輸出。[官方確認]

### Pixel 11 vs Pro（開發視角）

| 項目 | Pixel 11 | Pixel 11 Pro | Pixel 11 Pro XL | 對 JoviYarn 的意義 |
|---|---:|---:|---:|---|
| SoC | Tensor G6 | Tensor G6 | Tensor G6 | 沒有官方宣告不同 TPU/NPU 或不同 GenAI API tier。[官方確認／未宣告差異] |
| RAM | 12 GB | 256GB SKU 12 GB；512GB/1TB 16 GB | 256GB SKU 12 GB；512GB/1TB 16 GB | RAM 並不自動等於 AICore Prompt API 支援；16GB 可能有較多 system headroom，但非已證明的 AI API 優勢。[合理推論] |
| typical battery | 4,985 mAh | 4,850 mAh | 5,115 mAh | XL 較適合 20–30 分鐘熱／電量測試；非 Nano 功能差異。[官方確認／合理推論] |
| thermal / sustained AI | 無官方 Prompt API sustained benchmark | 同 | 同 | 不應宣稱 Pro 熱設計有實質 Nano 優勢，必須實測。[尚未確認] |

**購買判斷：**若目的只是 Android development + Gemini Nano PoC，普通 Pixel 11 **硬體上應足夠作 first reference device**；Pro 沒有已公布的第三方 GenAI API feature advantage。若會長時間錄音、螢幕亮、反覆 Nano inference，Pixel 11 Pro XL 的較大電池可作較舒服的 endurance test device，但這是測試便利性，不是必要能力。[合理推論]

## 2. 第三方 App 的 Gemini Nano：目前正式狀態

### ML Kit GenAI / AICore

ML Kit GenAI APIs 建構於 **AICore**（Android system service）之上，讓 app 使用共享的 on-device Gemini Nano。AICore 負責 model delivery／availability，SDK 的 `checkStatus()` 會回 `AVAILABLE`、`DOWNLOADABLE`、`DOWNLOADING` 或 `UNAVAILABLE`；未支援或尚未取得可支援配置可回 `UNAVAILABLE`。[官方確認]

正式 Prompt API 依賴：Android API 26+、`com.google.mlkit:genai-prompt:1.0.0-beta2`。它是 **Beta**，不是 platform-stable API；device bootloader unlocked 則不支援。[官方確認]

### 官方裝置矩陣的關鍵結果

| API | 官方明列 Google Pixel | Pixel 11 是否列入 |
|---|---|---|
| Prompt API, `nano-v3` | Pixel 9/9 Pro/9 Pro XL/9 Pro Fold；Pixel 10/10 Pro/10 Pro XL/10 Pro Fold | **否** |
| Feature-specific（summarize/proofread/rewrite/image-description） | Pixel 9 與 Pixel 10 系列 | **否** |
| Speech Recognition Advanced | Pixel 10（文件稱更多裝置 in development） | **否** |
| Speech Recognition Basic | 多數 Android API 31+ | 不逐機型列；理論上取決於 API/feature status |

所以：**Pixel 11 consumer Nano ≠ Pixel 11 ML Kit Prompt API supported。** 這可能只是 rollout/documentation lag，也可能是 AICore model/config 尚未向 third-party SDK enable；官方尚未說明，因此不能選一個故事當事實。[尚未確認]

## 3. Prompt API 對 JoviYarn 的適配度

### 已有能力與限制

- text-only 或 image + text；輸出 text 或 structured output。[官方確認]
- input 必須 < **4,000 tokens**（約 3,000 English words）；官方建議避免 >4K output。用 `countTokens()`，不要用字數猜。[官方確認]
- 可選 blocking `generateContent()` 或 `generateContentStream()`；streaming 對長輸出較快看到第一段，短提示可用 non-streaming。[官方確認]
- request 可設 `temperature`、`seed`、`topK`、`candidateCount`、`maxOutputTokens`；不是只有固定 consumer UX。[官方確認]
- **system instructions (Beta)**：Nano v3+，建議 <150 words（100–200 tokens）。[官方確認]
- **Structured Output (Alpha)**：Kotlin `@Generable`/`@Guide` schema → typed object；要檢查 `isStructuredOutputFeatureAvailable()`，並處理 parse/schema/max-token errors。[官方確認]
- **Prefix caching (Experimental)**：可把固定 story rules／state schema 作 prefix，避免每次重新 prefill；僅 text、只在部分 Prompt devices，Pixel 11 未確認。官方公布 Pixel 9 300-token prefix + 50-token suffix：無 cache 0.82s（表中明列的 cache-hit 欄數值未能由抓取表格可靠辨讀，故不誤抄）。[官方確認]
- API 不內建長期 chat memory；multi-turn 要由 app 自己保存 transcript、canonical Story State、selected suggestion、open threads，並在每次 request 補回當下最小必要 context。[官方確認／合理推論]

### 建議 prompt 設計

題目中範例是合適的 **小任務**，但不要每次塞整段 30 分鐘 transcript。建議：

```text
<role>你是繁中故事共創助手。輸出必須短、可直接接續。</role>
<story_state>{canonical JSON}</story_state>
<recent_transcript>最近 2–6 句、已定稿的語音文字</recent_transcript>
<task>回傳 3 個彼此不同、每項最多 8 個中文字的下一步發展。</task>
```

- state extraction：低 temperature（官方建議 deterministic task 從 0.2 開始）、typed JSON；將 LLM extraction 視為可錯的候選 patch，先 schema validate、再 merge，不能讓模型直接覆寫 canonical state。[官方確認／工程建議]
- suggestions：短 output（如每項 3–12 字），可設 `candidateCount=3`，但部分 implementation/preview 不支援 >1，必須 error fallback；也可單次要求 JSON array 3 items。[官方確認／工程建議]
- 中英混用：模型沒有針對「Traditional Chinese story co-writing」的官方品質或 supported-language guarantee；請把 `cmn-Hant-TW` test corpus 視為 release gate，不能只測英文。[尚未確認]

## 4. 是否能完全離線？

| 功能 | 初始下載完成、AICore `AVAILABLE` 後 | Airplane Mode 首次開箱／AICore reset 後 |
|---|---|---|
| Prompt-based Story State extraction | 理論上可 on-device | 不保證；需 config/model download |
| suggestions / Story Node extraction / clean transcript / ending | 同上，模型能力與品質要驗證 | 同上 |
| ML Kit speech Basic | 裝置語音模型已下載時可 | 語言模型未下載則不可 |
| ML Kit speech Advanced | 僅在正式支援裝置且 model ready 時才可；Pixel 11 未確認 | 同上 |
| Android platform on-device recognizer | 可（若 availability true、requested locale model installed） | 不可保證；engine/model 是 device-dependent |

ML Kit 明確宣稱 GenAI input/inference/output local、在 unreliable internet 下功能相同；但也明確指出 AICore 新設／clear data 後會下載 latest configuration，網路下可能需數分鐘到數小時，並可能出現 binding/feature-not-found/download errors。因此「**pre-provisioned Pixel 11、已驗證 models、之後飛航模式**」可成為 PoC 目標；「任何新 Pixel 開機即全離線」不是可承諾的產品需求。[官方確認]

## 5. Speech-to-text 深查

### ML Kit GenAI Speech Recognition API

- **Alpha**（無 SLA、可 breaking change）；dependency `com.google.mlkit:genai-speech-recognition:1.0.0-alpha1`。[官方確認]
- mic (`AudioSource.fromMic()`) 或 real-time PFD audio；output 是 continuous Kotlin Flow，初稿可修正成 final。適合 JoviYarn 的 partial transcript UI。[官方確認]
- Basic：傳統 on-device speech model，API 31+；支援 `en-US` 與 `cmn-Hant-TW (beta)`（另含多語）。
- Advanced：Gemini model、聲稱較廣語言覆蓋／品質，`en-US`、`cmn-Hant-TW` 均在 high-accuracy locale 清單；**正式只寫 Pixel 10，Pixel 11 未列**。[官方確認]
- 官方沒有公布 partial-result latency、mixed Chinese+English benchmark、10–30 分鐘 session limit 或 Pixel 11 test 結果。[尚未確認]

### Platform `android.speech.SpeechRecognizer`

- 一般 `SpeechRecognizer` 實作「likely to stream audio to remote servers」，Android 明說不 intended for continuous recognition；不要以 `EXTRA_PREFER_OFFLINE` 當作強保證（service 可以不支援／忽略 extras）。[官方確認]
- 先 `isOnDeviceRecognitionAvailable(context)`，再 `createOnDeviceSpeechRecognizer(context)` 才是 on-device path；可用 `RecognitionSupport` 查 installed/pending/supported on-device languages，並以 `triggerModelDownload()` 取得 locale。[官方確認]
- `EXTRA_PARTIAL_RESULTS` 可要求 partial；`EXTRA_PREFER_OFFLINE` 僅偏好 offline engine。語言／混碼行為最終取決於 device recognition service，因此 `cmn-Hant-TW`、`en-US`、code-switch 全須實機驗收。[官方確認]

### 對連續 10–30 分鐘敘事的結論

不要將單一 platform recognizer session 當無限長錄音承諾。選擇 ML Kit GenAI Speech API（若 `AVAILABLE`）或 on-device service，做分段／silence boundary、保存 final chunks、將 partial 當可撤回 UI；遇 engine stop/error 時快速 reopen 並保留時間戳。這是比「mic 永遠開著」更穩的工程模式。[官方限制／工程建議]

## 6. Nano inference 限制與 loop 設計

- AICore 有 **per-app inference quota**：短時間太多 request → `ErrorCode.BUSY`，要 exponential backoff；長期／daily 類 quota 可回 `PER_APP_BATTERY_USE_QUOTA_EXCEEDED`。Google 沒公布數值，不能自行假設 quota。[官方確認]
- inference 僅允許 app 在 top foreground；foreground service 也會得到 `BACKGROUND_USE_BLOCKED`。[官方確認]
- model availability、download、warm-up 都要做 state machine。`warmup()` 可降低 first inference latency；`checkStatus()` 必須在顯示相關 UI 前執行。[官方確認]
- 官方文件未給 model storage size、concurrency contract、cancellation API/semantics、thermal threshold 或 Pixel 11 sustained benchmark；這些皆須測，不能承諾。[尚未確認]

**JoviYarn 的正常使用模式：**不是 100ms 一次 inference，而是 (1) STT local partial 持續；(2) final/silence 後 debounce 3–8s 做一次小 state patch；(3) Help 或長停頓才生成 3 個極短 suggestion；(4) 點選後 local merge。這明顯比 polling 合理，也更符合 quota/battery/foreground constraints；仍需以 BUSY、battery quota 與溫度實測校正節流。[合理推論]

## 7. 延遲與 0.5–2 秒目標

**可靠的官方數字非常有限。** Google Pixel 11 blog 有整體「up to 3.5× faster on-device AI tasks」對 Tensor G5 的 claim，沒有 Prompt API TTFT、tokens/sec 或短提示 end-to-end benchmark。ML Kit prefix-cache page 對 Pixel 9 只公布特定 fixed/dynamic prefix inference table，並非 Pixel 11、也不是 Help→UI end-to-end。

因此：

- 三個 2–8 字提示，warm model、短 input、short `maxOutputTokens`、FAST variant（若 availability true）、streaming/first chunk，有**機會**落入 0.5–2s；這是工程推估，非 Pixel 11 已證實數據。[合理推論]
- 冷啟動、長 state、Chinese decoding、structured output、首次 cache miss、熱降頻或 BUSY 很可能超過該目標。[合理推論]
- 產品 UX 應有「先顯示第一個 token/chip、其餘逐步出現」、2s 後可取消／重試、以及 deterministic fallback（例如 state-based prompt templates）。[工程建議]

## 8. Pixel 8 Pro：現在能做什麼

- **不能把 Pixel 8 Pro 當正式 ML Kit Prompt API/Gemini Nano target。** 官方 Prompt device matrix 不列 Pixel 8 Pro；這是 SDK enablement/support contract 的限制，不是「它沒有任何 Nano consumer feature」。[官方確認]
- 可先做：Android UI、audio/session architecture、canonical Story State/JSON schema、prompt fixtures/evaluation harness、offline speech service exploration、`SpeechRecognizer` on-device availability、ML Kit Speech Basic（API 31+ path）。[官方確認／合理推論]
- 需要 Pixel 9/10/11 實機才可測：AICore `Generation` Prompt API、Nano model name/quality、Prompt structured output/prefix cache、Prompt quota、Nano latency、offline story suggestions。[官方確認]

## 9. JoviYarn 最終判定

### ✅ 已確認現在可以做

- 在 Pixel 8 Pro 開始 Android PoC：Story State、長 transcript 分段、STT fallback、evaluation corpus、UI/Help interaction。
- 面向已在 Prompt matrix 的 Pixel 9/10，使用 ML Kit Prompt API 作 on-device short extraction/suggestions。
- 用 ML Kit Speech Basic 做 streaming partial/final transcript，測 `cmn-Hant-TW`（beta）及 English。
- 已 download/ready 後，在飛航模式測 GenAI inference；app 必須顯示 model readiness。

### 🟡 技術合理，但 Pixel 11 support/documentation 尚需驗證

- Pixel 11 的 third-party Prompt API、Structured Output、System Instructions、prefix cache、FAST model preference。
- Pixel 11 Advanced GenAI Speech Recognition（目前 docs only say Pixel 10）。
- 0.5–2 秒 Help suggestions、繁中故事品質、中英混說、20–30 分鐘 thermal/battery/quota。

### ❌ 目前做不到／不建議依賴

- 以 Pixel 11 consumer 「latest Gemini Nano」宣傳，作為 Prompt API production support 的依據。
- 新機首次開機、不連網就保證 Nano/STT model 可用。
- platform `SpeechRecognizer` 的一般 remote-capable route 做未分段 30 分鐘 continuous recognition。
- 背景服務持續跑 Nano inference（AICore 明確 block background use）。

## 10. Pixel 11 到手第一天：可複製 PoC checklist

1. 記錄 build fingerprint、Android version、AICore version；在 release-unlocked bootloader device 上建立最小 app。
2. `Generation.getClient().checkStatus()`，記錄 `AVAILABLE`/`DOWNLOADABLE`/`UNAVAILABLE`；若可，`getBaseModelName()`。**不要先假定 v3/v4。**
3. 跑官方最小 Prompt API sample；記錄 cold、warm、10 次 latency 與 error code。
4. 跑繁中 prompt 與 `cmn-Hant-TW` story fixtures；人工評分格式遵守、角色一致性、 hallucination。
5. 做 `@Generable` StoryState patch extraction；schema validate + canonical merge；記錄 parse/MAX_TOKENS failures。
6. Help action：同一短 context 回 3 suggestions；比較 plain text、typed JSON、candidateCount=3（有錯就 fallback）。
7. 量 TTFT、complete time、P50/P95、tokens/input-output、battery %、skin/battery temperature；分 cold/warm/cache-hit。
8. **Airplane Mode**：先完成 AICore/Nano/STT download，重啟 app 後逐項重跑；再在 reset/new-device state 重現 failure，分清兩個情境。
9. STT：先 ML Kit Basic `cmn-Hant-TW`、`en-US`、mixed script；再 check Advanced status；platform on-device recognizer 作 fallback。保存 partial→final transition 與 chunk reopen error。
10. 串 `Mic → local STT → final chunk debounce → Nano state patch → Help suggestions`；連續 20–30 分鐘，測 foreground、螢幕亮/暗、網路/airplane、BUSY、battery quota、thermal，並輸出 CSV/trace。

**Go/no-go gate：**只有當 #2（Pixel 11 Prompt `AVAILABLE`）、#4–#8（繁中品質、latency、飛航）、#10（sustained quota/thermal）都通過，才把 Pixel 11 當 JoviYarn「cloudless core」reference。否則它仍是好 Android/STT device，但 Nano story copilot 應保留 Pixel 10 reference 或 cloud/local-open-model fallback。

## 來源（優先官方）

1. [ML Kit GenAI overview / device matrix / quota / foreground rule](https://developers.google.com/ml-kit/genai)（頁面 last updated 2026-07-15；本調查抓取 2026-08-22）
2. [ML Kit Prompt API overview](https://developers.google.com/ml-kit/genai/prompt/android)
3. [Prompt API get started：4,000 token limit、download/status、warmup、streaming、configs](https://developers.google.com/ml-kit/genai/prompt/android/get-started)
4. [System instructions (Beta)](https://developers.google.com/ml-kit/genai/prompt/android/system-instructions)
5. [Prefix caching (Experimental)](https://developers.google.com/ml-kit/genai/prompt/android/prefix-caching)
6. [Structured output (Alpha)](https://developers.google.com/ml-kit/genai/prompt/android/structured-output)
7. [Model selection: stable/preview, FULL/FAST](https://developers.google.com/ml-kit/genai/prompt/android/select-model)
8. [Prompt design guidance](https://developers.google.com/ml-kit/genai/prompt/android/prompt-design)
9. [ML Kit GenAI Speech Recognition (Alpha)](https://developers.google.com/ml-kit/genai/speech-recognition/android)
10. [Android `SpeechRecognizer` reference](https://developer.android.com/reference/android/speech/SpeechRecognizer)
11. [Android `RecognizerIntent` reference](https://developer.android.com/reference/android/speech/RecognizerIntent)
12. [Android `RecognitionSupport` reference](https://developer.android.com/reference/android/speech/RecognitionSupport)
13. [Google Pixel technical specifications](https://support.google.com/pixelphone/answer/7158570?hl=en)（Pixel 11: Tensor G6/RAM/battery/Gemini Nano）
14. [Google blog: The Pixel 11 series](https://blog.google/products-and-platforms/devices/pixel/google-pixel-11-pro-xl/)（2026-08-12；latest Nano、3.5×/3.5× consumer claim）
15. [Official ML Kit GenAI Android samples](https://github.com/googlesamples/mlkit/tree/master/android/genai)

## 今天是否值得為 JoviYarn 買 Pixel 11？

**結論：值得買作為新 Android reference device／STT 與 Tensor G6 實驗機；不值得把購買決策建立在「它已正式保證能給第三方 Prompt API」上。** 若這一點是購買的主要理由，最理性的動作是先把此報告的 day-one #2 跑出 `AVAILABLE`，或選用文件已明列的 Pixel 10 作 Nano API baseline。Pixel 11 對 JoviYarn 的真正價值很可能存在，但截至 2026-08-22，尚缺 developer-facing confirmation。

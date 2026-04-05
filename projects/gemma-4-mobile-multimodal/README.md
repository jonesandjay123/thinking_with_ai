# Gemma 4 E2B / E4B — 手機端多模態可行性研究

> 2026-04-04 | 研究報告 | 由 Jarvis（Dispatch 網路研究 + Mac Code Task 落檔）產出

## TL;DR

Google 在 **2026-04-02** 開源 Gemma 4，同步發布四個變體：**E2B**、**E4B**、26B A4B、31B。其中 E2B / E4B 是這次最值得關注的兩個「edge models」——它們真正做到了在**手機本地、離線、低延遲**的多模態推論，而且**原生支援音訊輸入**，這在同級（2-5B 參數）的開源模型裡是獨一無二的差異化優勢。

關鍵事實：

- **Apache 2.0 授權**（無客製條款，可商用、可重新分發）
- **E2B：實際 2.3B 參數，有效 5.1B 深度**（透過 Per-Layer Embeddings），4-bit 量化後 < 1.5GB memory
- **E4B：實際 4.5B 參數**，效能明顯更強
- **原生多模態：** 文字、圖像、影片、**音訊**（僅 E2B/E4B 有音訊；26B/31B 沒有）
- **128K context window**
- **音訊輸入長度上限：30 秒**
- **手機實測速度：15-25 tokens/秒**（E2B on smartphones）
- **iPhone 可行，官方路徑：** MediaPipe LLM Inference SDK + LiteRT-LM；Apple Silicon 另有 mlx-vlm + TurboQuant（4x 記憶體節省）

對 Jones 正在發想的 voice-first / media-index 專案，這個發布的意涵是：**「在手機上完全離線做語音問答和內容互動」這件事，從理論變成了已經可以實作的現實**。

---

## 一、時間線與版本背景

- **2026-04-02** — Google DeepMind 在 Google Open Source Blog、DeepMind 官方、以及 Google AI for Developers 三個渠道同步發布 Gemma 4
- **同日** — Hugging Face 上線 `google/gemma-4-E2B`、`google/gemma-4-E2B-it`、`google/gemma-4-E4B` 等 model card，附帶 litert-community 的 LiteRT-LM 格式
- **同日** — Android Developers Blog 宣布 Gemma 4 進入 AICore Developer Preview
- **同日** — Unsloth 釋出 GGUF 量化版本、MLX 4-bit 版本
- **媒體反應** — VentureBeat 的標題最精準：「Google releases Gemma 4 under Apache 2.0 — and that license change may matter more than benchmarks」

授權變化是這次發布的最大策略意義。前幾代 Gemma 的授權有自定義條款、Harmful Use 模糊條文，很多商業團隊因此選擇 Mistral 或 Qwen。**Gemma 4 改成標準 Apache 2.0，沒有客製條款，沒有使用限制**——這是 Google 正面迎戰 Qwen、Mistral 等開源競品的姿態。

---

## 二、技術突破：Per-Layer Embeddings (PLE)

E2B 和 E4B 不是普通的小模型，它們的架構有個關鍵設計叫 **Per-Layer Embeddings**：

> 「E」代表 Effective（有效參數）。透過在每一層 decoder 注入次級 embedding 訊號，E2B 的 2.3B active 參數實際承載了相當於 5.1B 參數的表徵深度，但量化後依然可以塞進 1.5GB 以下的記憶體佔用。

白話翻譯：**它用 2B 大小跑出接近 5B 的效果**，這讓手機端部署變成實務可行，而不是只能跑 demo。

這對比 Phi-4 mini（3.8B）或 Qwen 3.5 小模型（0.8B-4B），Gemma 4 的 E 系列有更高的「有效智商／記憶體」比例。

---

## 三、多模態能力：音訊輸入是最大差異化

**E2B 和 E4B 原生支援的輸入模態：**

- 文字
- 圖像（含可變解析度）
- 影片
- **音訊**（單段最長 30 秒）

26B A4B 和 31B **沒有** 音訊能力。這個設計取捨很明顯——Google 把音訊放在 edge 小模型上，因為邊緣裝置才是最需要「用聽的、用講的」的場景。

音訊能做什麼？

- 語音轉文字（替代 Whisper）
- 語音指令理解
- 對話式內容互動（錄音 → 提問 → 回答）
- OCR（這是圖像能力，但同一個模型同時吃）
- 物體偵測、指點（pointing）

**與競品的比較：**

- **Phi-4 mini (3.8B)** — 純文字，無音訊
- **Phi-4 multimodal (5.6B)** — 有語音+視覺+文字，但 Microsoft 生態綁定較深
- **Qwen 3.5 small (0.8B-9B)** — 無音訊，多語言強（201 種語言，256K context），但缺 Google 的 edge 工具鏈
- **Gemma 4 E2B/E4B** — **同級裡唯一原生音訊輸入的開源模型**

→ 對於 voice-enabled edge applications，Gemma 4 E2B/E4B 是目前最對的選擇，沒有之一。

---

## 四、效能 Benchmark

根據官方 model card：

| Benchmark | E2B | E4B | 備註 |
|---|---|---|---|
| MMLU Pro | 60.0% | 69.4% | 通用知識 |
| MMMU Pro | 44.2% | 52.6% | 多模態理解 |
| LiveCodeBench v6 | 44.0% | 52.0% | 代碼生成 |
| AIME 2026 | 37.5% | 42.5% | 數學推理 |
| CoVoST (audio) | 33.47 | 35.54 | 跨語言語音翻譯 |
| FLEURS (audio) | 0.09 | 0.08 | 多語言語音（低越好）|

**解讀：**

- E4B 在所有 benchmark 上都明顯優於 E2B，但差距大約是 10-12 個百分點
- E2B 的強項是「體積小、速度快、電量省」
- 大模型 Gemma 4 31B 在 AIME 2026（89.2%）和 LiveCodeBench 上超越 Qwen 3.5 32B

**實務選擇建議（來自社群實測）：**

> annjose 在 iPhone 上同時測試 E2B 和 E4B，結論是 E2B 的答題品質沒有顯著低於 E4B，但體積更小、速度更快——所以對手機應用來說，**E2B 通常是正確選擇**，除非你的任務真的需要 E4B 那 10% 的 benchmark 差距。

---

## 五、手機端部署實務

### iOS 路徑

**官方路徑：** MediaPipe LLM Inference SDK + LiteRT-LM

- MediaPipe 是跨平台（iOS / Android / Web），支援 TensorFlow Lite（現在改名 LiteRT）格式
- LiteRT-LM 是專門針對 LLM 的 orchestration 層
- Hugging Face 上 `litert-community/gemma-4-E2B-it-litert-lm` 和 `gemma-4-E4B-it-litert-lm` 直接提供 .litertlm 格式檔案

**Apple Silicon 替代路徑：** mlx-vlm

- `unsloth/gemma-4-E2B-it-UD-MLX-4bit` 和 `gemma-4-E4B-it-UD-MLX-4bit` 提供 MLX 量化版
- mlx-vlm 支援 TurboQuant——**4x 少的 active memory，同樣的精度**
- 讓長 context 推論在 Apple Silicon 上變成實務可行

### Android 路徑

**官方路徑：** AICore Developer Preview（Gemma 4 上線當日同步宣布）

- 可以用 ML Kit GenAI Prompt API 呼叫
- 可以 power Android Studio 的 Agent Mode
- E2B 比前代快 4 倍，耗電低 60%

### 其他框架

全部都在 day one 支援：Transformers、vLLM、llama.cpp、MLX、LM Studio、transformers.js（瀏覽器）、Ollama、Unsloth

### 記憶體需求

- **5GB RAM**（4-bit 量化）— E2B 或 E4B 都可以
- **15GB RAM**（16-bit 全精度）

現代 iPhone（6GB+ RAM）和 Android 旗艦機（8-12GB）都有餘裕跑 4-bit 版本。

### 實測速度

- **智慧型手機：15-25 tokens/秒**（E2B）
- 這個速度對於對話應用來說完全夠用，甚至比雲端 API 的網路延遲還快

---

## 六、授權意義：Apache 2.0 的策略轉向

這次 Gemma 4 採用標準 Apache 2.0，跟之前 Gemma 1/2/3 的自定義授權有本質差異：

**沒了的東西：**

- 客製條款
- Harmful Use 模糊定義
- Google 保留的更新條款權利
- 重新分發限制
- 商業部署限制

**VentureBeat 的觀察：**

> Google 前幾代 Gemma 的效能一直都不錯，但自定義授權讓很多團隊轉向 Mistral 或 Qwen。Gemma 4 把這個痛點移除，對開源生態的吸引力可能比 benchmark 提升還重要。

**對獨立開發者和小型團隊的影響：**

- 可以放心商用
- 可以 fork、fine-tune、再分發
- 不用擔心 Google 某天更新條款

---

## 七、戰略關聯：對 media-index / voice-first 專案的意涵

這一章是本報告的核心價值。Jones 昨天（2026-04-03）初始化了 `media-index` repo，發想「讓影音內容從時間軸解放出來」的專案，並特別關注**語音介面**和**視障使用者**的可能應用。

Gemma 4 E2B/E4B 的發布，對那個專案的意涵是：

### ✅ 從「理論可能」變成「已經可以做」的能力

1. **完全離線的語音問答** — E2B 的音訊輸入 + 15-25 tok/s 的手機推論，足以支撐「用講的問，用聽的答」的互動
2. **隱私第一** — 音訊資料不離開手機，對敏感內容（個人對話、視障使用者的語音輸入）是關鍵優勢
3. **低延遲** — 沒有網路往返，體感接近即時
4. **無需 API 成本** — 部署到使用者裝置後，每次問答零邊際成本

### ⚠️ 仍然需要權衡的地方

1. **30 秒音訊上限** — 對於長 podcast / 長影片的輸入，需要先用其他工具切片或做 transcript，才丟給 Gemma
2. **2-4B 模型的能力邊界** — 對複雜推理、跨段落關聯、多步驟 reasoning，E2B/E4B 的能力明顯不如 Claude Opus 或 GPT-4 級別的雲端模型。需要在「離線便利」和「推理深度」之間取捨
3. **iOS 開發門檻** — MediaPipe + LiteRT 不像 Claude API 那麼直接，要寫原生 Swift 或 React Native 整合
4. **電量管理** — 雖然 Google 說電量低 60%，但持續推論仍會讓手機發熱；長時間使用需要策略

### 🔀 建議的架構分層

基於 Gemma 4 的能力，media-index 可以考慮這樣的**混合架構**：

- **手機端 Gemma 4 E2B** 負責：
  - 即時語音輸入理解
  - 快速 retrieval（「這段在講什麼？」）
  - 離線可用的基本問答
  - 視障 accessibility 互動
- **雲端大模型**（Claude / GPT / Gemini）負責：
  - 深度摘要
  - 跨內容聚合分析
  - 複雜推理與引述
- **分工邏輯：**
  - 裝置有網路 → 丟雲端做重度任務
  - 裝置沒網路 → Gemma 4 本地撐住基本體驗
  - 裝置敏感資料 → 強制本地

這個分層的好處是**隱私和能力可以共存**，不用二選一。

### 🧠 視障應用的突破點

Google Gemma 生態已經有一個相關案例：「Gemma Vision」——給視障者的胸前攝影機 + 語音 AI 助手，使用者靠控制器或語音觸發，不需要觸控螢幕。這個案例用的是 Gemma 3 系列，**Gemma 4 E2B/E4B 的原生音訊輸入**可以讓這類應用再跨一步：從「語音指令 + 視覺理解」變成「完全語音對話式的環境感知」。

對 media-index 的發想裡提到的「視障不需要 timeline UI、用問的在內容裡移動」——Gemma 4 的能力組合（音訊輸入 + 多模態 + 30 秒音訊片段 + 低延遲）剛好就是這個需求的技術匹配。

---

## 八、可能的實驗切入點

給想快速驗證的人的最小實驗設計（純探索性，非承諾）：

**實驗 A：iPhone 上的 Gemma 4 E2B 安裝**

- 工具：LM Studio（Mac）先驗證 → MediaPipe LLM Inference SDK（iOS）
- 目標：跑起來、手動測試對話品質
- 時間成本：一個下午
- 這個做完可以判斷「手機本地 LLM」這條路對你來說品質夠不夠

**實驗 B：語音 → 問答 閉環**

- 工具：Swift iOS app + LiteRT-LM + Gemma 4 E2B
- 目標：錄 10 秒音訊 → 丟進去 → 文字回答
- 時間成本：一週
- 這個做完可以判斷「voice-first」模式是否成立

**實驗 C：與 Whisper 的 A/B 比較**

- 同一段音訊分別用 Whisper small (244M) 和 Gemma 4 E2B 做 speech-to-text
- 比較準確度、速度、記憶體佔用
- 目的：決定 media-index 的 transcription 層要走哪條路

實驗 A 是最輕量的「先裝起來感受看看」；實驗 C 是最實用的「給工程決策 data」。

---

## 九、開放問題（持續追蹤）

這些問題目前沒有明確答案，值得之後追蹤：

1. **30 秒音訊上限是硬限制還是建議上限？** 模型 card 寫 max 30s，但實際上餵更長會怎樣？降品質還是直接拒絕？
2. **LiteRT-LM 在 iOS 的成熟度如何？** MediaPipe 在 Android 很穩，iOS 的 LLM 路徑相對新，有沒有踩坑報告？
3. **E2B 和 E4B 的 thermals 差異？** 長時間推論，哪個對 iPhone 熱管理更友善？
4. **Apache 2.0 授權對 Gemma Vision 這類衍生專案的影響？** 有沒有出現新一波 accessibility 相關的 fork？
5. **跟 Apple 的 Foundation Models（iOS 26 引入的 on-device LLM）比較？** 兩者是否能在同一個 app 裡互補使用？
6. **如果 media-index 要做跨平台（iOS + Android + Web），Gemma 4 是目前最一致的選擇嗎？**

---

## 十、參考連結

**官方來源：**

- [Gemma 4 官方發布公告 (Google Blog)](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/)
- [Gemma 4 — Google DeepMind](https://deepmind.google/models/gemma/gemma-4/)
- [Gemma 4 model card](https://ai.google.dev/gemma/docs/core/model_card_4)
- [Android AICore Developer Preview 公告](https://android-developers.googleblog.com/2026/04/AI-Core-Developer-Preview.html)
- [Google Open Source Blog — Apache 2.0](https://opensource.googleblog.com/2026/03/gemma-4-expanding-the-gemmaverse-with-apache-20.html)
- [Deploy Gemma on mobile devices (官方文件)](https://ai.google.dev/gemma/docs/integrations/mobile)

**Hugging Face 模型：**

- [google/gemma-4-E2B](https://huggingface.co/google/gemma-4-E2B)
- [google/gemma-4-E2B-it](https://huggingface.co/google/gemma-4-E2B-it)
- [litert-community/gemma-4-E2B-it-litert-lm](https://huggingface.co/litert-community/gemma-4-E2B-it-litert-lm)
- [litert-community/gemma-4-E4B-it-litert-lm](https://huggingface.co/litert-community/gemma-4-E4B-it-litert-lm)
- [unsloth/gemma-4-E2B-it-GGUF](https://huggingface.co/unsloth/gemma-4-E2B-it-GGUF)
- [unsloth/gemma-4-E2B-it-UD-MLX-4bit](https://huggingface.co/unsloth/gemma-4-E2B-it-UD-MLX-4bit)

**媒體與分析：**

- [Welcome Gemma 4 (Hugging Face Blog)](https://huggingface.co/blog/gemma4)
- [VentureBeat — Apache 2.0 license change](https://venturebeat.com/technology/google-releases-gemma-4-under-apache-2-0-and-that-license-change-may-matter)
- [NVIDIA Technical Blog — Edge and On-Device](https://developer.nvidia.com/blog/bringing-ai-closer-to-the-edge-and-on-device-with-gemma-4/)
- [Hands-On: Mobile AI with Gemma — iOS & Android (annjose.com)](https://annjose.com/post/mobile-on-device-ai-hands-on-gemma/)
- [Gemma 4 models explained (AI News Silo)](https://ainewssilo.com/articles/gemma-4-models-explained-hardware-benchmarks)
- [I Tested All 4 Gemma 4 Models (Medium)](https://medium.com/@chewloongnian/i-tested-all-4-gemma-4-models-the-26b-one-is-cheating-in-the-best-way-744e40d90d37)
- [Gemma 4 vs Llama 4 vs Qwen 3.5 (Lushbinary)](https://www.lushbinary.com/blog/gemma-4-vs-llama-4-vs-qwen-3-5-open-weight-model-comparison/)

**工具與框架：**

- [LiteRT-LM GitHub](https://github.com/google-ai-edge/LiteRT-LM)
- [Unsloth Gemma 4 文件](https://unsloth.ai/docs/models/gemma-4)
- [gemma4.app/mobile](https://www.gemma4.app/mobile)
- [Ollama — gemma4:e2b](https://ollama.com/library/gemma4:e2b)

**相關參考（非 Gemma 4 本身但脈絡相關）：**

- [Building Offline RAG on iOS with Gemma 3N (Medium)](https://medium.com/google-cloud/building-offline-rag-on-ios-how-to-run-gemma-3n-locally-ffdfda6f7217) — 前一代的 iOS 實作，做 Gemma 4 可以參考流程
- [Developers changing lives with Gemma 3n (Google Blog)](https://blog.google/technology/developers/developers-changing-lives-with-gemma-3n) — 包含 Gemma Vision 等 accessibility 案例

---

## 十一、本報告的限制

- **沒有親手實測**。以上所有效能數字來自官方 model card 和社群 benchmark，沒有在 Jones 自己的裝置上驗證。
- **距離發布僅兩天**，社群案例還在爆發中，一週後會有更多實戰報告可以追蹤。
- **對 Apple Foundation Models 的比較資訊不足**。這是 iOS 生態的另一個重要參考點，本報告未深入覆蓋。
- **沒有比較 Gemma 4 和直接用 Apple Intelligence 的取捨**。這對 iOS 原生 app 的決策其實很關鍵。

---

*本報告由 Jarvis（Dispatch 網路研究 + Mac Code Task 落檔）產出。Dispatch 端負責所有 WebSearch 研究、內容起草、結構規劃；Mac code task 端負責落檔、commit、push。此報告為 Phase 0 探索材料，不代表任何實作決策。*

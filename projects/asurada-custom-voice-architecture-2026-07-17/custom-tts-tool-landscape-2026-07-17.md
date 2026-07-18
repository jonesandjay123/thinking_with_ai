# 2026 自訂／自訓聲音 TTS 工具地圖：從 MOSS-TTS 1.5 到可接 Asurada 的選擇

> 研究日期：2026-07-17  
> 問題：若要替 Asurada 建立「屬於自己的聲音」，目前有哪些自訂 TTS／voice-cloning／fine-tuning 工具？各自怎麼操作、適合什麼、哪些不該碰？  
> 範圍：優先看能產生專屬聲線、支援中文或多語、可串流、且有公開程式或官方 API 的工具。不是把所有 TTS 都列進來。

## 結論先說

MOSS-TTS 1.5 是合理候選，而且不是舊 demo：它在 2026 年 5 月更新了 multilingual、clone 穩定性、停頓控制；同家族還有面向 voice agent 的 `MOSS-TTS-Realtime`、可 CPU-first 部署的 `MOSS-TTS-Nano`，以及可用文字設計角色聲音的 `MOSS-VoiceGenerator`。

但對 Asurada，現在不該問「哪一個最強」，而應先決定要哪一種資產：

1. **有真人／本人 reference，想把這個人聲音做成 TTS**：先試 *zero-shot voice cloning*，不需要真的訓練。
2. **想創造一個不存在的 Asurada 角色聲音**：先試 *voice design*；不要去模仿真人。
3. **要在長期產品中擁有穩定音色、可控資料與部署**：zero-shot 驗證成功後才做 LoRA / fine-tune。
4. **要即時對話**：除了聲音像不像，必須把首音延遲、可取消、串流、繁中與聲音權利當成同等 gate。

我的推薦並非「直接訓練 MOSS」：

```text
先用 3 個候選做聽感／延遲 POC
  ├─ Qwen3-TTS：最完整的 Apache-2.0 開源 clone / design / fine-tune 基線
  ├─ MOSS-TTS：最貼近 voice-agent / realtime 方向的開源家族
  └─ Google Instant Custom Voice：最少 MLOps 的合規 managed baseline

若中文／台灣腔結果不夠好，再補測 CosyVoice 3 或 Chatterbox Multilingual V3。
```

Fish Audio S2 與 F5-TTS 值得看，但目前授權條件使它們不是第一個 Asurada production 候選：Fish 是 Research License；F5 的程式是 MIT，但官方預訓練模型是 CC-BY-NC。

## 1. 先把名詞講清楚：你未必需要「訓練模型」

| 路徑 | 你提供什麼 | 得到什麼 | 典型用途 | 對 Asurada 的意義 |
| --- | --- | --- | --- | --- |
| 預設 TTS voice | 無 | 固定供應商聲線 | 快速 MVP | 不能形成專屬聲音 |
| Voice design | 文字描述，如「溫和、冷靜、台灣口音感」 | 一個新的合成角色聲音 | 角色創作 | 最適合不想模仿真人的 Asurada persona |
| Zero-shot clone | 3–30 秒乾淨 reference；有些模型另需逐字稿 | 以 reference 為 conditioning 的聲音 | 最快測試本人／授權聲音 | 第一個真正該做的實驗 |
| Voice-cloning key（managed） | reference + 固定同意錄音 | 供應商管理的可重用 key | 低 MLOps 上線 | 最快但有供應商與資料邊界 |
| Fine-tune / LoRA / speaker adapter | 長時間、切句、文字對齊、乾淨且有權利的資料 | 自己持有／部署的 adapter 或 checkpoint | 要高一致性與可控性 | zero-shot 確定值得後才做 |
| 從零訓練 foundation TTS | 大規模多說話者訓練資料和大量 GPU | 一個完整基礎模型 | 研究機構／模型公司 | 不適合本案 |

「用自己的聲音」與「自己訓練」也不必綁在一起。第一個 POC 應證明 *voice clone 是否足夠自然、快速、穩定*；若夠好，fine-tune 反而是沒有必要的額外風險。

## 2. 候選總覽（以 Asurada 的需求排序）

| 工具／服務 | 類別 | 專屬聲音方法 | 即時性訊號 | 中文相關性 | 授權／產品判斷 |
| --- | --- | --- | --- | --- | --- |
| **Qwen3-TTS** | 開源權重 | 3 秒 reference clone、voice design、fine-tune | 官方宣稱單字元輸入起可 streaming、最低 97 ms synthesis latency | 支援中文與 10 種主要語言 | Apache-2.0；**最佳開源基線** |
| **MOSS-TTS family** | 開源權重 | zero-shot clone、voice design、Realtime fine-tune | Realtime model 公布 warm-up TTFB 180 ms；Nano 4 CPU cores streaming | 31 語言，含中文、粵語；有拼音／IPA | Apache-2.0；**最值得深挖的 voice-agent 家族** |
| **CosyVoice 3** | 開源權重 | multilingual / cross-lingual zero-shot clone、instruction | 官方稱 bi-streaming 約 150 ms | 中文、18+ 中國方言／口音，另支援多語 | Apache-2.0；**中文表現應優先試聽** |
| **Chatterbox Multilingual V3** | 開源權重 | 10 秒 audio prompt clone | Turbo 為低延遲英文 agent；Multilingual 側重品質 | 23+ 語言，另有 Chinese single-language pack | MIT；輸出帶 watermark；**好用的第二交叉驗證** |
| **Google Chirp 3 Instant Custom Voice** | Google Cloud managed | 10 秒 reference + 強制 consent audio → cloning key | Cloud TTS 可 streaming；另有雙向 streaming 文件 | 官方 ICV 列 `cmn-CN`，不等於台灣國語已驗證 | 合規路徑最清楚；**最快 managed baseline** |
| **Fish Audio S2 Pro** | 開放程式／受限權重 | 10–30 秒 clone，多講者、細緻標籤控制 | 官方描述 SGLang TTFA 約 100 ms（H200） | 中文為 tier 1，80+ 語言 | Fish Audio Research License；**只作研究／聽感標竿** |
| **F5-TTS v1** | 開源程式與模型 | reference audio + reference text；可 fine-tune | 有 Triton/TensorRT-LLM 部署資料 | 中文／英文出身、支援擴展語言 | code MIT、官方預訓練模型 CC-BY-NC；**非商用探索** |

表中的延遲是上游在特定硬體／條件下的宣告，不能直接拿來當 Pixel 或 Mac mini 的產品承諾。POC 必須量自己的 end-to-end latency。

## 3. 你找到的 MOSS-TTS 1.5：到底是什麼、怎麼選

來源：[`OpenMOSS/MOSS-TTS`](https://github.com/OpenMOSS/MOSS-TTS)（Apache-2.0）。

### 3.1 不要把 MOSS-TTS 當成一個模型

| MOSS 成員 | 大小／形態 | 核心用途 | 對 Asurada 的判斷 |
| --- | --- | --- | --- |
| `MOSS-TTS-v1.5` | 8B, MossTTSDelay | 高品質長文本、zero-shot clone、多語、code-switch、發音與時長控制 | 品質／功能基準；不會是手機端即時第一選擇 |
| `MOSS-TTS-Local-Transformer-v1.5` | 4B | v1.5 功能 + 48 kHz stereo、local streaming decode | 適合研究「自己控制輸出 stream」的 server 版 |
| `MOSS-TTS-Realtime` | 1.7B | 具多輪上下文的增量 TTS，專為 voice agent | **最貼近 Asurada，但需要真實測取消／繁中／GPU** |
| `MOSS-TTS-Nano` | 約 100M | CPU-first、multilingual clone、streaming | 最適合先在本機做低成本功能 demo；不是品質終點 |
| `MOSS-VoiceGenerator` | 1.7B | 文字描述產生角色聲音，不需 reference | 若 Asurada 要「原生角色聲」而非模仿真人，值得試 |
| `MOSS-TTSD` | 8B | 多講者、長對話語音生成 | 偏內容／角色對話，不是第一版單助手 speaker |

### 3.2 v1.5 為何值得注意

官方在 2026-05-26 發布 `MOSS-TTS-v1.5`，列出的改進是：

- 已知語言加 language tag 時，multilingual synthesis 更穩；
- voice cloning 更穩，特別是「長 reference → 短句輸出」；
- 更能跟隨標點的 prosody；
- 用 `[pause X.Ys]` 做明確停頓控制；
- 維持 zero-shot clone、長文本、拼音／IPA、code-switch。

後續又有：

- `MOSS-TTS-Local-Transformer-v1.5`（4B）可透過 SGLang-Omni 提供 OpenAI 相容 `/v1/audio/speech`、streaming 和 voice cloning；
- `MOSS-TTS-Nano` 宣稱在 4 個 CPU core 上能串流輸出；
- 官方提供 `llama.cpp + ONNX Runtime` 的 torch-free 推理路徑，也聲稱支援 Apple MLX 生態；
- full MOSS family 有 vLLM-Omni／SGLang-Omni 路徑。

這代表 MOSS 的價值不只是一段漂亮 demo，而是有從「CPU demo → GPU server → voice-agent realtime」的產品化階梯。

### 3.3 MOSS 的實際操作流程

先選操作目的，不要直接 fine-tune。

```text
A. Demo / 聽聲音
   選 Nano 或線上 demo
   → 輸入固定測試台詞 + reference audio
   → 只評估音色、中文、停頓、code-switch

B. 自訂 clone POC
   選 v1.5 / Local Transformer
   → 準備有明確授權的 reference + 對應文字
   → zero-shot clone
   → 以 streaming server 包成內網 API
   → Android 收 PCM/Opus 並可 cancellation

C. 真正 fine-tune
   選 v1.5 或 Realtime 的官方 finetuning 文件
   → 建立乾淨且文字對齊的 speaker dataset
   → 訓練 / eval / red-team
   → 只在通過 clone POC 後部署
```

要注意 MOSS 8B 即使有量化／torch-free 路徑，也不代表 Mac mini 或手機可無條件達到 live assistant 品質。MOSS Nano 適合先驗證 deployment shape；MOSS Realtime 才是往 Asurada 接的候選。

## 4. 其他值得納入的工具

### 4.1 Qwen3-TTS：功能最完整的開源第一基線

來源：[`QwenLM/Qwen3-TTS`](https://github.com/QwenLM/Qwen3-TTS)（Apache-2.0）。

官方釋出的 0.6B 與 1.7B 模型分成三個清楚角色：

- `CustomVoice`：內建多個 preset 聲線，加自然語言 instruction 控制；
- `Base`：可用 **3 秒** reference audio 快速 clone，也可做 fine-tuning；
- `VoiceDesign`：由文字描述創作聲音，再把結果變為可重用的 clone prompt。

它支援中文、英文、日韓、德法俄葡西義等十種主要語言；官方說明 single model 同時支援 streaming/non-streaming，合成 latency 可低至 97 ms。它也特別提供 `create_voice_clone_prompt`，可將同一 reference 預處理後重用，避免每一句重算 speaker features。

**建議操作：**

1. 先用 `VoiceDesign` 寫 3 個 Asurada persona（例如冷靜導航、溫柔陪伴、理性短答）。
2. 從最滿意的短音檔建立 reusable clone prompt。
3. 用 `Base` 對同一套繁中／中英台詞 streaming 測試。
4. 確定 speaker identity 和節奏都真的值得後，才讀官方 `finetuning/` 做 LoRA／FT。

它是最值得先做的「不必先錄真人、也能創造固定角色聲」候選。

### 4.2 CosyVoice 3：中文／方言與 streaming 值得優先比較

來源：[`FunAudioLLM/CosyVoice`](https://github.com/FunAudioLLM/CosyVoice)（Apache-2.0）。

`Fun-CosyVoice 3.0` 為 0.5B 系列，官方主打：

- 9 種主要語言；
- 中文與 18+ 中國方言／口音；
- multilingual / cross-lingual zero-shot voice cloning；
- instruction 控制語言、方言、情緒、速度、音量；
- text-in 與 audio-out 的 bi-streaming，官方報告約 150 ms。

Asurada 面向台灣繁中，CosyVoice 不等於保證台灣腔，但它是最應拿來與 Qwen／MOSS 比「中文地名、口音、語速」的開源候選之一。

**操作要點：**clone repo（含 submodule）→ 下載 `Fun-CosyVoice3-0.5B` → 先跑 WebUI/example → 再用官方 deployment 指引包成 WebSocket/HTTP streaming。第一輪不要 fine-tune。

### 4.3 Chatterbox Multilingual V3：有中文特化包、產品面較完整

來源：[`resemble-ai/chatterbox`](https://github.com/resemble-ai/chatterbox)（MIT）。

- Multilingual V3 為 0.5B、23+ 語言，目標是更高 speaker similarity、較少 hallucination；
- 有 Chinese `zh-cmn` 的 single-language model card，對中文比通用多語模型更值得試聽；
- `Chatterbox-Turbo` 為 350M、主要針對英文低延遲 voice agent，支援 `[laugh]` 等 paralinguistic tag；不能因為 Turbo 很快就假定中文也同樣適用；
- clone API 的基本用法是 `audio_prompt_path="your_10s_ref_clip.wav"`；
- 所有輸出都帶 Resemble 的不可感知 watermark。這可能是好事（溯源），也可能是某些產品定位要先接受的限制。

**使用情境：**想要一個快速本機 clone demo、又希望與 Qwen/CosyVoice 對照中文變體時，值得列入前四。

### 4.4 Google Cloud Chirp 3 Instant Custom Voice：最少工程、同意流程最清楚

來源：[Cloud TTS — Instant Custom Voice](https://cloud.google.com/text-to-speech/docs/chirp3-instant-custom-voice)。

流程是：

```text
聲音本人錄製規定的 consent statement（≤10 秒）
  + 同環境、低噪音 reference audio（≤10 秒）
  → 以 Google API 產生 voiceCloningKey
  → 由 Cloud TTS text:synthesize / streaming 使用該 key
  → Server 將串流音訊傳給 Android
```

它不會給你 checkpoint，也不是你自行 fine-tune，但可最快回答「聲音 clone 對產品是否足夠」。ICV 的中文文件列出 `cmn-CN`，所以若目標是台灣國語，務必以真實語料試聽，別用規格表替代測試。

### 4.5 Fish Audio S2 Pro：聽感標竿，但授權先踩煞車

來源：[`fishaudio/fish-speech`](https://github.com/fishaudio/fish-speech)。

官方描述 S2 Pro 為 4B、多語（80+）、10–30 秒 clone、可用自然語言 tag 控制細緻情緒與多講者；它在單張 H200 的報告裡列約 100 ms TTFA。

但 source 與 model weights 都標示 **Fish Audio Research License**，不是 Apache/MIT。這使它很適合做品質比較／研究，不應在未重新核對 license 和商業條款前，納入 Asurada 的產品計畫。

### 4.6 F5-TTS：研究與 fine-tune 參考，不是產品捷徑

來源：[`SWivid/F5-TTS`](https://github.com/SWivid/F5-TTS)。

F5-TTS 支援以 reference audio + reference transcript 做生成，並提供 fine-tune 指引、Gradio、Docker、Triton/TensorRT-LLM 部署資料。它對中文很有研究價值。

但要分清楚：repo code 是 MIT，**官方 pre-trained models 是 CC-BY-NC**。若 Asurada 有任何商業化可能，不能將「code MIT」誤解為「整個可商用」。

## 5. 從音檔到可用自訂 TTS：推薦流程

### Step 0：先決定聲音倫理與人格，而不是先錄音

回答三件事：

1. Asurada 是 Jones 的聲音 clone、明確授權的配音者，還是完全合成的角色？
2. 是否允許它用同一聲音讀私人資料、在旁人面前播放？
3. 使用者如何知道這是合成聲音、如何停用與刪除？

若不是要模仿真人，優先選 voice design；它能降低人格權與冒認風險。

### Step 1：建立很小、但專為 assistant 設計的測試集

不要只用一句「你好」。建立約 20–30 句固定腳本，涵蓋：

- 短答：「好，我在。」
- 日期／時間：「明天早上十點提醒你。」
- 台灣／日本／美國地名；
- 中英混說：`Firebase AI Logic`、`Jovitrip`、`OpenClaw`；
- 數字、貨幣、地址；
- 情緒與節奏：安撫、提醒、簡潔拒絕、工具執行中；
- 使用者打斷後的停止。

這份腳本是所有模型公平比較的基準，應保存版本與錄音／輸出評分，但不把 reference voice 或 API key 放進 Git。

### Step 2：先做 zero-shot / voice design，而非訓練

| 目標 | 合理第一工具 |
| --- | --- |
| 模仿有授權的本人聲音 | Qwen3 Base、MOSS v1.5、CosyVoice 3、Google ICV、Chatterbox |
| 創作非真人角色聲 | Qwen3 VoiceDesign、MOSS-VoiceGenerator |
| 想先確認本機／CPU deployment | MOSS Nano |
| 想確認低延遲 voice-agent 可行性 | MOSS Realtime、Qwen3 streaming、CosyVoice 3 bi-streaming |

reference audio 的最低要求：乾淨、單人、無 BGM、無壓縮失真、正常且有表情的語速；所有音檔要可證明使用者同意。不同模型會要求 reference transcript 或按其範例格式提供，不能用一份模糊 YouTube 擷取音檔硬套所有工具。

### Step 3：將候選包成可取消的 streaming service

Asurada 最終不能直接把 Python notebook 塞進 Android；需要一個受控的 server boundary：

```text
Asurada Android
  ⇄ authenticated WebSocket / HTTPS
Custom TTS gateway
  ├─ model warm pool / reference prompt cache
  ├─ sentence chunker + bounded audio queue
  ├─ interrupt / cancellation by session + segment ID
  ├─ metrics: TTFA, RTF, errors, queue depth
  └─ TTS model provider (Qwen / MOSS / Cloud / ...)
```

這個 gateway 必須能對舊 segment 立即取消；否則使用者插話後仍會播出上一句，是 voice assistant 最難接受的缺陷。

### Step 4：再判斷是否應 fine-tune

只有在以下情況才開訓：zero-shot 的音色／穩定性明顯不足，但產品實驗已證明自訂聲音本身有價值。

fine-tune 的一般工作是：

```text
取得書面同意
  → 錄製、去噪、切句、逐句核對文字
  → training / validation / held-out evaluation
  → 失真、幻覺、跨語言與濫用測試
  → adapter/checkpoint 存放與權限控管
  → server deployment + rollback
```

資料量、GPU 和訓練時數依模型而異，不應在沒有選定模型與授權前先蒐集大量聲音。先做 30–60 分鐘等級的合法、乾淨 pilot 語料規劃，再以官方 fine-tune guide 校正需求，是較穩的做法。

## 6. 真正需要量測的成功標準

不要只靠 YouTube demo。每個模型都記錄：

| 指標 | 目標／問題 |
| --- | --- |
| Speaker similarity | 聽者是否一聽就覺得是目標角色／授權者？長對話會飄嗎？ |
| 可理解度 | 繁中、台灣地名、數字、英語名詞有沒有錯讀？ |
| 表演性 | 能否自然短答、停頓、安撫、提醒，而不是像有聲書？ |
| TTFA | 從 LLM 送出第一個字，到第一段可播放音訊的時間 |
| End-to-end latency | 麥克風停下到第一段回覆；包含網路、LLM、TTS、播放 |
| Barge-in | 使用者插話後，舊音訊多久完全停止？有沒有殘句？ |
| 成本／硬體 | 單次／每分鐘成本、GPU 記憶體、warm-up、同時 session 數 |
| 權利 | 是否有明確 consent、模型與輸出可否商用、能否刪除／撤回？ |

推薦淘汰規則：只要在 20–30 句腳本中反覆出現斷字、重複、錯讀、打斷失敗或讓人明顯等候，就不要因為「音色很像」而推進產品。

## 7. 對 Asurada 的具體試驗排序

### 第一輪：不寫整合，只選聲音（半天到一天）

1. Qwen3 VoiceDesign + Base clone。
2. MOSS VoiceGenerator + v1.5 clone；另看 Nano 是否足夠流暢。
3. Google Instant Custom Voice（僅在有聲音本人明確同意時）。
4. CosyVoice 3 和 Chatterbox Chinese 作繁中交叉比較。

輸出只有一個表：同腳本、同耳機、盲聽排序、備註錯字與延遲。

### 第二輪：只讓兩個 finalist 進 streaming POC（3–5 天）

不改 Gemini tool 層、不做 Android 完整 UI；只驗證：

- `segment_id` 下的串流音訊；
- 中途取消；
- 超過 10 分鐘連續多輪後，音色是否穩；
- 斷線時能安全退回 Gemini native voice。

### 第三輪：才決定模型所有權與訓練

- 若 managed clone 已經很好：不必訓練，買的是速度與穩定性。
- 若 Qwen/MOSS zero-shot 已經好：先自架 inference，避免不必要的 fine-tune。
- 若只有 fine-tune 才達標：在權利、資料、GPU 預算都獨立確定後再做 adapter 訓練。
- 若全部都不達標：**果斷使用 Gemini Live 的預設 voice**。那不是失敗，而是拒絕把脆弱音訊鏈放進核心 assistant。

## 8. 來源與版本核對

本報告於 2026-07-17 直接查閱官方 repo／官方文件；model release、支援語言和授權變動很快，開始實作前要再讀當時的 README、model card 與 LICENSE。

1. OpenMOSS, [MOSS-TTS Family](https://github.com/OpenMOSS/MOSS-TTS) — v1.5、Realtime、Nano、fine-tune、llama.cpp/SGLang/vLLM 與 Apache-2.0。
2. OpenMOSS, [MOSS-TTS-Nano](https://github.com/OpenMOSS/MOSS-TTS-Nano) — CPU-first deployment 路徑。
3. Qwen, [Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS) — Base/CustomVoice/VoiceDesign、clone、fine-tune、Apache-2.0。
4. FunAudioLLM, [CosyVoice](https://github.com/FunAudioLLM/CosyVoice) — CosyVoice 3 的 multilingual clone、bi-streaming 與 Apache-2.0。
5. Resemble AI, [Chatterbox](https://github.com/resemble-ai/chatterbox) — Multilingual V3、Chinese pack、Turbo 與 MIT。
6. Google Cloud, [Chirp 3 Instant Custom Voice](https://cloud.google.com/text-to-speech/docs/chirp3-instant-custom-voice) — consent、reference 和 clone key。
7. Fish Audio, [Fish Speech](https://github.com/fishaudio/fish-speech) — S2 Pro 功能與 Fish Audio Research License。
8. SWivid, [F5-TTS](https://github.com/SWivid/F5-TTS) — fine-tune/deployment；注意官方預訓練權重的 CC-BY-NC 限制。

## 最後判斷

「自訂 TTS」在 2026 已不再是遙遠研究題：MOSS、Qwen、CosyVoice、Chatterbox 都能讓你在有 reference 或 persona 描述的情況下迅速聽到候選聲音；Google ICV 則提供一條更托管、同意流程明確的 clone 路線。

真正稀缺的不是模型數量，而是產品判斷：**哪個聲音在繁中、延遲、打斷、長對話穩定與授權上同時合格。**Asurada 的第一個勝利條件不是「自己訓練成功」，而是找到一個你每天都願意聽、而且不會破壞 Gemini Live 對話節奏的聲音。

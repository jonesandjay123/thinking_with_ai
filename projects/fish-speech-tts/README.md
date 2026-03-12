# Fish-Speech TTS 開源平替研究

> 研究日期：2026-03-12
> 研究目的：評估 Fish-Speech 作為開源 TTS 引擎的可行性，探索取代付費 TTS 服務的可能性

---

## 1. Fish-Speech 是什麼

[Fish-Speech](https://github.com/fishaudio/fish-speech) 是由 [Fish Audio](https://fish.audio/) 團隊開發的開源 Text-to-Speech（TTS）引擎，目前最新版本為 **Fish Audio S2**。

| 項目 | 資訊 |
|------|------|
| **GitHub Stars** | ⭐ ~26,300+（截至 2026-03） |
| **Forks** | ~2,200+ |
| **開發者** | Fish Audio 團隊（Shijia Liao、Yuxuan Wang 等） |
| **License** | Fish Audio Research License（非標準開源，研究用途授權） |
| **最新模型** | S2-Pro（4B parameters） |
| **訓練資料** | 超過 1,000 萬小時音訊，涵蓋約 50 種語言 |
| **論文** | [arXiv:2411.01156](https://arxiv.org/abs/2411.01156)、[arXiv:2603.08823](https://arxiv.org/abs/2603.08823) |

### ⚠️ License 注意事項

Fish-Speech 使用的是 **Fish Audio Research License**，並非 MIT/Apache 等標準開源授權。這意味著：
- ✅ 學術研究和個人使用基本允許
- ⚠️ 商業使用可能需要額外授權
- 需仔細閱讀 LICENSE 文件確認具體限制

---

## 2. 核心特色

### 🌍 多語言支援
- 支援 **50+ 語言**，包括：英文、中文、日文、韓文、阿拉伯文、德文、法文等
- 不需要 phoneme 或語言特定的前處理
- 在 MiniMax 多語言測試集中，24 種語言裡有 **11 種 WER 最低**、**17 種語音相似度最高**

### 🎭 語音克隆（Voice Cloning）
- 僅需 **10-30 秒** 參考音訊即可克隆語音
- 捕捉音色（timbre）、說話風格和情感傾向
- **無需額外 fine-tuning**，零樣本克隆

### 🎛️ 細粒度語音控制
- 支援自然語言標籤內嵌控制，例如：
  - `[laugh]` — 笑聲
  - `[whispers]` — 耳語
  - `[super happy]` — 非常開心的語氣
  - `[professional broadcast tone]` — 專業播報語調
  - `[pitch up]` — 提高音調
- 不依賴固定的 tag 集合，接受**自由文字描述**

### 🏗️ 模型架構：Dual-Autoregressive（Dual-AR）
- 基於 decoder-only transformer + RVQ audio codec（10 codebooks, ~21 Hz frame rate）
- **Slow AR**：沿時間軸預測主要語義 codebook（4B 參數）
- **Fast AR**：在每個時間步生成其餘 9 個 residual codebooks（400M 參數）
- 此架構與標準 autoregressive LLM 結構同構，可直接利用 SGLang 的推理優化

### ⚡ 推理效能（單張 NVIDIA H200 GPU）
| 指標 | 數值 |
|------|------|
| Real-Time Factor (RTF) | 0.195 |
| Time-to-first-audio | ~100 ms |
| Throughput | 3,000+ acoustic tokens/s |

### 🎯 Reinforcement Learning 對齊
- 使用 Group Relative Policy Optimization (GRPO) 進行 post-training alignment
- 結合語義準確度、指令遵從、音質偏好、音色相似度四維 reward signal

### 👥 多講者 & 多輪對話
- 原生支援多講者生成（`<|speaker:i|>` token）
- 支援多輪對話，利用上下文提升後續內容的表達自然度

---

## 3. 跟其他 TTS 的比較

| 特性 | Fish-Speech S2 | OpenAI TTS | ElevenLabs | Coqui TTS / XTTS | Bark |
|------|----------------|------------|------------|-------------------|------|
| **開源** | ✅（Research License） | ❌ 閉源 | ❌ 閉源 | ✅ MPL-2.0 | ✅ MIT |
| **自部署** | ✅ | ❌ API only | ❌ API only | ✅ | ✅ |
| **語音克隆** | ✅ 10-30s | ❌ 僅預設聲音 | ✅ 高品質 | ✅ | ❌ |
| **多語言** | 50+ 語言 | ~20 語言 | 29 語言 | ~17 語言 | 多語言 |
| **情感控制** | ✅ 自然語言標籤 | 有限 | ✅ | 有限 | 有限 |
| **Streaming** | ✅（via SGLang） | ✅ | ✅ | 有限 | ❌ |
| **品質（WER）** | 0.54% 中文 / 0.99% 英文 | 高但未公開 | 高 | 中等 | 中等 |
| **硬體需求** | 24GB GPU | 無（雲端） | 無（雲端） | 中等 GPU | 中等 GPU |
| **費用** | 免費（自部署） | $15/1M chars | $0.30/1K chars | 免費（自部署） | 免費（自部署） |
| **模型大小** | 4B params | 未公開 | 未公開 | ~1.5B | ~1B |
| **即時性** | RTF 0.195 | 低延遲 | 低延遲 | 較慢 | 很慢 |

### Benchmark 對比

在 Seed-TTS Eval 上，Fish Audio S2 的 WER 表現優於所有公開和閉源模型：

| 模型 | WER（中文） | WER（英文） |
|------|------------|------------|
| **Fish Audio S2** | **0.54%** | **0.99%** |
| Qwen3-TTS | 0.77% | 1.24% |
| MiniMax Speech-02 | 0.99% | 1.90% |
| Seed-TTS | 1.12% | 2.25% |

Audio Turing Test（指令模式）：S2 的 0.515 posterior mean 超越 Seed-TTS (0.417) 24%，超越 MiniMax-Speech (0.387) 33%。

---

## 4. 部署需求

### 硬體需求

| 部署方式 | GPU VRAM | CPU | 備註 |
|----------|----------|-----|------|
| **GPU 推理（建議）** | ≥ 24GB | — | 推薦 NVIDIA RTX 4090 / A100 / H100 |
| **CPU 推理** | 無需 GPU | 多核心 | 速度顯著較慢，不推薦用於即時場景 |
| **生產環境** | H200 | — | RTF 0.195，適合大規模服務 |

### 系統要求
- **OS**: Linux, WSL（Windows Subsystem for Linux）
- ⚠️ **macOS 不支援 compile 優化**（但可能可以跑 CPU 模式）
- 需安裝：`portaudio19-dev`、`libsox-dev`、`ffmpeg`

### 安裝步驟

#### 方法一：Conda

```bash
conda create -n fish-speech python=3.12
conda activate fish-speech

# GPU 安裝（選擇 CUDA 版本：cu126, cu128, cu129）
pip install -e .[cu129]

# CPU 安裝
pip install -e .[cpu]
```

#### 方法二：UV（更快的依賴解析）

```bash
# GPU 安裝
uv sync --python 3.12 --extra cu129

# CPU 安裝
uv sync --python 3.12 --extra cpu
```

#### 方法三：Docker

```bash
git clone https://github.com/fishaudio/fish-speech.git
cd fish-speech

# 啟動 WebUI
docker compose --profile webui up

# 啟動 API Server
docker compose --profile server up

# CPU 部署
BACKEND=cpu docker compose --profile webui up
```

### 下載模型權重

```bash
hf download fishaudio/s2-pro --local-dir checkpoints/s2-pro
```

---

## 5. API 使用方式

### HTTP API Server

啟動 API 伺服器：

```bash
python tools/api_server.py \
  --llama-checkpoint-path checkpoints/s2-pro \
  --decoder-checkpoint-path checkpoints/s2-pro/codec.pth \
  --listen 0.0.0.0:8080
```

常用參數：
- `--compile`：啟用 torch.compile 優化（約 10x 加速）
- `--half`：使用 fp16 模式
- `--api-key`：設定 Bearer token 認證
- `--workers`：設定 worker 數量

### API Endpoints

| Endpoint | 方法 | 功能 |
|----------|------|------|
| `/v1/health` | GET | 健康檢查 |
| `/v1/tts` | POST | 文字轉語音 |
| `/v1/vqgan/encode` | POST | VQ 編碼 |
| `/v1/vqgan/decode` | POST | VQ 解碼 |

### 健康檢查

```bash
curl -X GET http://127.0.0.1:8080/v1/health
# {"status":"ok"}
```

### Streaming 支援
- 透過 SGLang 整合支援 production-grade streaming
- 支援 continuous batching、paged KV cache
- Time-to-first-audio ~100ms

### 輸出格式
- WAV 音訊輸出
- 支援 VQ token 中間表示（`.npy` 格式）

### Command Line 推理流程

```bash
# 1. 從參考音訊取得 VQ tokens
python fish_speech/models/dac/inference.py \
  -i "reference.wav" \
  --checkpoint-path "checkpoints/s2-pro/codec.pth"

# 2. 從文字生成 semantic tokens
python fish_speech/models/text2semantic/inference.py \
  --text "要轉換的文字" \
  --prompt-text "參考文字" \
  --prompt-tokens "fake.npy"

# 3. 從 semantic tokens 生成語音
python fish_speech/models/dac/inference.py \
  -i "codes_0.npy"
```

---

## 6. 實際應用場景

### 🤖 語音助手
- AI 聊天機器人的語音回覆
- 智慧家居語音互動
- 客服機器人語音回應

### 🎬 影片旁白
- YouTube / 教學影片的旁白生成
- Podcast 自動化
- 有聲書製作

### 🎮 遊戲 NPC 語音
- 動態生成遊戲角色對話
- 多角色聲音（多講者支援）
- 情感豐富的角色表演（透過自然語言標籤控制）

### 📢 通知系統
- 智慧通知語音播報
- 電話語音系統（IVR）
- 公共廣播自動化

### 🌐 多語言內容
- 多語言教學內容
- 即時翻譯語音輸出
- 全球化產品的語音支援

### ♿ 無障礙
- 為視障人士提供網頁內容語音播報
- 文件/電子書朗讀

---

## 7. 社群評價

### 整體風向

Fish-Speech 在開源 TTS 社群中獲得了相當高的關注度（26,000+ stars），被廣泛認為是目前最好的開源 TTS 解決方案之一。

### 正面評價
- **品質接近商業方案**：多項 benchmark 超越閉源模型，社群普遍認為音質已達商業水準
- **語音克隆效果出色**：僅需短音訊即可高品質克隆，效果令人驚艷
- **多語言表現強勁**：中文和英文的 WER 均為業界最低
- **活躍的開發團隊**：持續更新，從 v1.0 到 S2 的進化明顯
- **Product Hunt 上線**：獲得 Top Post 徽章，說明社群認可度高

### 負面/疑慮
- **License 不夠開放**：Fish Audio Research License 不是傳統開源授權，商業使用的灰色地帶讓部分開發者卻步
- **硬體門檻高**：24GB GPU VRAM 的需求排除了大量消費級硬體
- **macOS 支援不完善**：不支援 compile 優化，限制了 Apple Silicon 用戶
- **文件還在完善中**：WebUI 推理頁面仍顯示「Coming soon」
- **生態系統年輕**：相比 Coqui TTS 等老牌專案，第三方整合和社群資源還較少

### 與競品的社群聲量對比
- vs **OpenAI TTS**：品質上 Fish-Speech 在 benchmark 已超越，但 OpenAI 的易用性和穩定性仍為商業首選
- vs **ElevenLabs**：ElevenLabs 在語音克隆的細膩度上仍被認為領先，但 Fish-Speech 的免費自部署是巨大優勢
- vs **Bark**：Fish-Speech 在品質和速度上都大幅超越 Bark
- vs **Coqui/XTTS**：Coqui 公司已關閉（2023），XTTS 維護緩慢，Fish-Speech 被視為更有前途的替代品

---

## 8. 優缺點總結

### ✅ 優點

1. **頂級品質**：Benchmark 成績超越所有公開和閉源模型
2. **語音克隆強大**：10-30 秒音訊即可零樣本克隆
3. **多語言支援廣**：50+ 語言，無需語言特定前處理
4. **自然語言控制**：用文字描述即可控制情感和語氣
5. **多講者支援**：原生支援多角色對話
6. **免費自部署**：不需要每月付費給 API 服務商
7. **推理效率高**：RTF 0.195，支援 production streaming
8. **活躍開發**：團隊持續改進，社群龐大

### ❌ 缺點

1. **License 限制**：非標準開源，商業使用需注意
2. **硬體門檻高**：24GB GPU VRAM 門檻不低
3. **系統限制**：主要支援 Linux/WSL，macOS 支援有限
4. **部署複雜度**：相比 API 服務，自部署需要更多 DevOps 經驗
5. **文件不夠完善**：部分功能的文件仍在建設中
6. **CPU 推理慢**：沒有 GPU 的環境下不適合即時使用
7. **模型體積大**：4B 參數，下載和儲存需要一定空間

---

## 9. 潛在用途（我們的場景）

### 🎯 場景一：AI 助手語音回覆（取代付費 TTS）

**可行性：⭐⭐⭐⭐ — 高度可行（需有合適 GPU）**

- 目前使用的 TTS 服務（如 ElevenLabs）有持續費用
- Fish-Speech 自部署後，邊際成本幾乎為零
- S2 品質已達商業水準，可直接替換
- 支援中英文雙語，符合我們的使用場景
- **挑戰**：需要一張 24GB GPU，Mac Mini 的 Apple Silicon 可能需要嘗試 CPU 模式或用外部 GPU 伺服器

**建議方案**：
- 在有 GPU 的雲端伺服器（如 RunPod / Vast.ai）部署 Fish-Speech API
- Jarvis 透過 HTTP API 呼叫，取代現有付費 TTS
- 或等待社群出現 Apple Silicon 優化版本

### 🎯 場景二：教學影片旁白

**可行性：⭐⭐⭐⭐⭐ — 非常適合**

- 語音克隆功能可以建立一致的「講師」聲音
- 多語言支援可以輕鬆製作中英文版本
- 自然語言標籤控制語氣，讓教學內容更生動
- 非即時場景，可以接受較長的生成時間（甚至 CPU 模式）
- **工作流程**：寫好腳本 → Fish-Speech 生成 → 後製剪輯

### 🎯 場景三：遊戲通知

**可行性：⭐⭐⭐ — 可行但需評估延遲**

- 即時通知需要低延遲，Fish-Speech 在 GPU 上可達 ~100ms TTFA
- 多講者支援可以為不同類型的通知設定不同聲音
- 情感控制標籤可以區分一般通知和緊急通知
- **挑戰**：需要持續運行的 GPU 服務，成本效益需要評估

### 🎯 場景四：網站互動語音

**可行性：⭐⭐⭐⭐ — 適合但需要後端 GPU**

- 可以為網站提供動態語音內容
- 支援 streaming，使用者體驗良好
- 多語言支援讓國際化網站受益
- **架構建議**：前端呼叫後端 API → 後端轉發到 Fish-Speech 服務 → streaming 音訊回傳

### 📋 優先級建議

| 優先級 | 場景 | 理由 |
|--------|------|------|
| 🥇 P0 | 教學影片旁白 | 非即時、高價值、低門檻 |
| 🥈 P1 | AI 助手語音回覆 | 高頻使用、節省成本、需解決 GPU 部署 |
| 🥉 P2 | 網站互動語音 | 提升使用者體驗、需 streaming 架構 |
| 4️⃣ P3 | 遊戲通知 | 低頻、需評估是否值得專門部署 |

---

## 附錄：快速開始指南

如果要快速試用，最簡單的方式：

```bash
# 1. 確保有 Docker 和 NVIDIA Docker runtime
# 2. Clone 並啟動
git clone https://github.com/fishaudio/fish-speech.git
cd fish-speech
docker compose --profile webui up

# 3. 開啟瀏覽器訪問 http://localhost:7860
```

或使用 Fish Audio 的線上 Playground：https://fish.audio/

---

## 相關連結

- 🏠 官網：https://fish.audio/
- 📦 GitHub：https://github.com/fishaudio/fish-speech
- 🤗 HuggingFace：https://huggingface.co/fishaudio/s2-pro
- 📄 技術報告：https://arxiv.org/abs/2411.01156
- 📖 官方文件：https://speech.fish.audio/
- 🐳 Docker Hub：https://hub.docker.com/r/fishaudio/fish-speech
- 💬 Discord：https://discord.gg/Es5qTB9BcN

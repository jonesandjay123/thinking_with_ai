# 2026 年 4 月 LLM 開源 vs 閉源：完整榜單與效能比較

> 2026-04-06 | 由 Jarvis（Dispatch 網路研究）產出
> Jones 需求：以 GPT 系列為基準線，看開源模型在各 benchmark 的落點位置

---

## TL;DR

**開源模型在 2026 年初已經在知識型 benchmark（MMLU）上追平甚至超越閉源模型。** GLM-5 的 MMLU 96% 超過了 GPT-5 的 92.5%。但在複雜 agent 任務、生產級程式碼、和整體人類偏好（Chatbot Arena）上，閉源模型仍然領先。

以 GPT 系列為基準：
- **GPT-3.5 等級（MMLU 70%）**：2026 年幾乎所有 7B+ 開源模型都超過這個水平
- **GPT-4 等級（MMLU 86%）**：Gemma 4 31B、Qwen 3.5、Kimi K2.5 都已超越
- **GPT-4o/5 等級（MMLU 90%+）**：只有最頂級的開源大模型（GLM-5 397B+、Qwen 3.5 397B）能碰到

**2023 年底開源 vs 閉源差距 17.5%，2026 年初差距趨近於零（知識型），但整體綜合能力仍差 ~10 分。**

---

## 一、GPT 系列基準線（你的座標軸）

| 模型 | 發布時間 | MMLU | 類型 | 定位 |
|---|---|---|---|---|
| **GPT-3.5 Turbo** | 2022-11 | 70.0% | 閉源 | 「及格線」——2026 年幾乎所有模型都超過這個 |
| **GPT-4** | 2023-03 | 86.4% | 閉源 | 「很強」——2026 年中型開源模型的目標線 |
| **GPT-4o** | 2024-05 | ~88% | 閉源 | 「主流頂級」——多模態、速度快 |
| **GPT-4o mini** | 2024-07 | ~82% | 閉源 | 「便宜夠用」——對標開源 7-14B |
| **GPT-5** | 2025 | 92.5% | 閉源 | 「前沿」——2026 年的高標 |
| **GPT-5.3 Codex** | 2026 | 81%* | 閉源 | *MMLU 較低但 agent/coding 頂級 |
| **GPT-5.4 Pro** | 2026 | ~92%+ | 閉源 | 綜合最強 OpenAI 模型 |

*GPT-5.3 Codex 的 MMLU 較低是因為它專門針對 coding/agent 優化，不追求通用知識。

---

## 二、2026 年 4 月閉源模型頂級陣容

| 模型 | 廠商 | MMLU | GPQA Diamond | HumanEval | SWE-bench | Arena Elo | 特色 |
|---|---|---|---|---|---|---|---|
| **Gemini 3.1 Pro** | Google | 94.3% | — | — | 80.6% | #1 | Arena 冠軍 |
| **Claude Opus 4.6** | Anthropic | 91.3% | — | — | 80.8% | ~#2 | SWE-bench 最強、1M context |
| **GPT-5.4 Pro** | OpenAI | ~92% | — | — | — | Top 5 | 綜合最強 OpenAI |
| **GPT-5.3 Codex** | OpenAI | 81% | — | — | — | Top 10 | Agent/coding 專精 |
| **Gemini 3.1 Flash** | Google | ~88% | — | — | — | Top 15 | 速度快、便宜 |

---

## 三、2026 年 4 月開源模型頂級陣容

| 模型 | 廠商 | 參數量 | MMLU | GPQA | HumanEval | MATH-500 | Arena Elo | 授權 |
|---|---|---|---|---|---|---|---|---|
| **GLM-5 (Reasoning)** | 智譜 AI | 大 | **96%** | **94** | — | — | 1451 | 開源 |
| **Kimi K2.5** | Moonshot | 大 | **92%** | 87.6 | **99.0** | **98.0** | 1447 | 開源 |
| **Qwen 3.5 397B** | 阿里巴巴 | 397B | 91% | 89 | — | — | ~1445 | 開源（有條件） |
| **GLM-4.7** | 智譜 AI | 大 | — | — | — | — | 1445 | 開源 |
| **Gemma 4 31B** | Google | 31B | ~88% | — | 80 | — | — | Apache 2.0 |
| **DeepSeek V3.2** | DeepSeek | 大 | ~87% | — | — | — | — | 開源 |
| **Llama 4 Maverick** | Meta | MoE | ~86% | — | — | — | — | Llama License |
| **Llama 4 Scout** | Meta | MoE | ~83% | — | — | — | — | 10M context |
| **Mistral Large 3** | Mistral | 123B | ~86% | — | — | — | — | Apache 2.0 |

---

## 四、⭐ 核心圖表：開源模型在 GPT 基準線的落點

用 MMLU 作為橫軸座標，標出每個模型的位置：

```
MMLU 分數軸（%）

  70%          80%          86%       88%    90%    92%    94%    96%
   |            |            |         |      |      |      |      |
   GPT-3.5     GPT-4o mini  GPT-4    GPT-4o  |     GPT-5  Gemini  GLM-5
                                              |            3.1Pro
   ─── 開源模型落點 ───

   [<--- 幾乎所有 7B+ 開源模型都在這條線以上 --->]
                             |
                    Llama 4 Maverick ≈ 這裡
                    Mistral Large 3 ≈ 這裡
                    DeepSeek V3.2 ≈ 這裡
                             |
                             [Gemma 4 31B ≈ 這裡]
                                      |
                                      [Qwen 3.5 397B ≈ 這裡]
                                      [Kimi K2.5 ≈ 這裡]
                                              |
                                              [GLM-5 = 這裡（超越所有閉源）]
```

**白話解讀：**

- **GPT-3.5 水平（70%）**：已經是「底線中的底線」。2026 年甚至 3B 小模型都可能超過
- **GPT-4 水平（86%）**：中高階開源模型（Llama 4、Mistral Large、DeepSeek V3.2）大致落在這個區間
- **GPT-4o/5 水平（88-92%）**：只有 Gemma 4 31B、Qwen 3.5、Kimi K2.5 等頂級開源能碰到
- **超越 GPT-5（92%+）**：GLM-5（96%）是目前唯一明確超越 GPT-5 MMLU 的開源模型

---

## 五、差距分析：開源還差在哪？

### 5.1 差距已經消失的領域

| 領域 | 狀態 | 說明 |
|---|---|---|
| **知識型（MMLU）** | ✅ 追平甚至超越 | GLM-5 96% > GPT-5 92.5% |
| **數學推理（MATH-500）** | ✅ 接近追平 | Kimi K2.5 98% |
| **程式生成（HumanEval）** | ✅ 追平 | Kimi K2.5 99% |
| **專家科學（GPQA）** | ✅ 接近追平 | GLM-5 94, Qwen 3.5 89 |

### 5.2 差距仍然存在的領域

| 領域 | 差距 | 說明 |
|---|---|---|
| **整體人類偏好（Arena）** | 中等 | 閉源 #1（Gemini 3.1 Pro）vs 開源 #1（GLM-5）差距 ~50 Elo |
| **生產級程式碼（SWE-bench）** | 明顯 | Claude Opus 4.6 80.8% 領先開源最好的 |
| **Agent 工具使用** | 明顯 | 閉源模型在複雜 multi-step agentic 任務仍領先 |
| **指令遵從精度** | 小但存在 | 閉源模型更能精確遵循複雜格式要求 |

### 5.3 歷史趨勢

| 時間點 | MMLU 差距（最佳閉源 - 最佳開源） |
|---|---|
| 2023 年底 | 17.5% |
| 2024 年中 | ~8% |
| 2025 年初 | ~3% |
| 2026 年初 | **0%（開源反超）** |

**結論：知識型任務的差距已經歸零。整體綜合能力的差距從 25-30 分縮小到 ~10 分。**

---

## 六、⭐ 16GB 消費級硬體可跑的開源模型（Jones 最關心的）

以下模型可以在 **RTX 5080 16GB VRAM** 或 **Mac Mini M4 16GB 統一記憶體** 上本地運行：

| 模型 | 參數量 | Q4 VRAM | MMLU 估算 | 多模態 | 授權 | 對標閉源 |
|---|---|---|---|---|---|---|
| **Gemma 4 26B A4B** | 26B MoE | ~15 GB | ~88% | 文字+圖+影片 | Apache 2.0 | ≈ GPT-4o |
| **Gemma 4 E4B** | 4.5B | ~2.5 GB | ~69% | 文字+圖+音訊 | Apache 2.0 | ≈ GPT-3.5 |
| **Qwen 2.5 14B** | 14B | ~8 GB | ~83% | 文字 | Apache 2.0 | ≈ GPT-4o mini |
| **Llama 3.3 8B** | 8B | ~5 GB | ~78% | 文字 | Llama License | 略低於 GPT-4o mini |
| **Mistral Small 3** | 7B | ~4 GB | ~76% | 文字 | Apache 2.0 | ≈ GPT-3.5+ |
| **Phi-4 mini** | 3.8B | ~2 GB | ~72% | 文字 | MIT | ≈ GPT-3.5 |
| **DeepSeek R1 14B** | 14B | ~8 GB | ~82% | 文字 | 開源 | ≈ GPT-4o mini |
| **Gemma 4 E2B** | 2.3B | ~1.5 GB | ~60% | 文字+圖+音訊 | Apache 2.0 | < GPT-3.5 |

**Jones 的硬體對應：**

- **Mac Mini M4 16GB**（Jarvis 主機）→ Gemma 4 26B A4B（MoE，緊但能跑） 或 E4B（寬鬆）
- **RTX 5080 16GB**（Jovix 工作站）→ Gemma 4 26B A4B + faster-whisper 可同時跑（序列模式）
- 兩台都可以輕鬆跑 E4B，同時開其他 app 不會 swap

**白話解讀：**

在你的硬體上本地跑，品質天花板大約在「GPT-4o 等級」（Gemma 4 26B）。日常任務（問答、自動化、簡單分析）綽綽有餘。複雜推理、深度 coding 仍需要 Claude Max / GPT-5。

---

## 七、Benchmark 解讀指南（避免被數字誤導）

| Benchmark | 測什麼 | 為什麼重要 | 陷阱 |
|---|---|---|---|
| **MMLU** | 廣域知識（57 科目選擇題） | 最常被引用的「智力測驗」 | 選擇題不反映真實使用體驗 |
| **GPQA Diamond** | 研究所級科學推理 | 測「真正的深度理解」 | 太學術，跟日常使用關聯低 |
| **HumanEval** | 簡單程式生成 | 寫 code 的基本能力 | 2026 年已被飽和（多個模型 99%） |
| **MATH-500** | 數學競賽題 | 推理鏈深度 | 不代表日常數學應用 |
| **SWE-bench** | 真實 GitHub issue 修復 | 最接近「AI 真的能寫程式」 | 閉源仍領先，開源差距最大 |
| **Chatbot Arena** | 600 萬人投票的偏好排名 | 最接近「用起來爽不爽」 | 受 UI/速度影響，不純粹測能力 |
| **τ2-bench** | Agent tool use（工具使用） | 測 OpenClaw 等 agent 場景 | Gemma 4 26B 在這裡 85.5%，很強 |

**最實用的判斷法則：** 如果你只看一個 benchmark，看 **Chatbot Arena**（最接近真實體驗）。如果你要跑 OpenClaw agent，看 **τ2-bench**。如果你要做 RAG/知識查詢，看 **MMLU + GPQA**。

---

## 八、結論：2026 年 4 月的 AI 模型格局

1. **開源已經不是「次等替代品」了。** 在知識和推理 benchmark 上，頂級開源模型（GLM-5、Kimi K2.5、Qwen 3.5）已經追平甚至超越 GPT-5。

2. **閉源的護城河在「綜合體驗」和「agent 能力」。** Claude Opus 4.6 和 Gemini 3.1 Pro 的真正優勢不在知識量，而在指令遵從精度、工具使用可靠度、和長鏈推理穩定性。

3. **16GB 消費級硬體的天花板大約在 GPT-4o 等級。** Gemma 4 26B MoE 是目前本地可跑的品質天花板，搭 OpenClaw 做 24/7 agent 完全夠用。

4. **中國實驗室正在主導開源。** GLM-5（智譜）、Qwen 3.5（阿里巴巴）、Kimi K2.5（Moonshot）、DeepSeek V3.2 佔據開源前四名。Google 的 Gemma 4 是非中國陣營的最強開源。

5. **GPT-3.5 已經是「歷史文物」水平。** 2026 年幾乎任何 7B+ 開源模型都能超過 GPT-3.5 Turbo。

---

## 九、參考來源

- [Open LLM Leaderboard (Hugging Face)](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard)
- [Chatbot Arena (OpenLM.ai)](https://openlm.ai/chatbot-arena/)
- [Arena AI Leaderboard](https://arena.ai/leaderboard)
- [BenchLM.ai — Best Open Source LLM Rankings](https://benchlm.ai/blog/posts/best-open-source-llm)
- [Onyx AI — Open Source LLM Leaderboard 2026](https://onyx.app/open-llm-leaderboard)
- [Onyx AI — Best LLM Leaderboard 2026](https://onyx.app/llm-leaderboard)
- [LLM Stats — Benchmarks Comparison](https://llm-stats.com/benchmarks)
- [Vellum AI — LLM Leaderboard](https://www.vellum.ai/llm-leaderboard)
- [Klu — 2026 LLM Leaderboard](https://klu.ai/llm-leaderboard)
- [VERTU — Open Source LLM Leaderboard 2026](https://vertu.com/lifestyle/open-source-llm-leaderboard-2026-rankings-benchmarks-the-best-models-right-now/)
- [Artificial Analysis — GPT-5 Benchmarks](https://artificialanalysis.ai/articles/gpt-5-benchmarks-and-analysis)
- [Let's Data Science — Open vs Closed LLMs 2026](https://letsdatascience.com/blog/open-source-vs-closed-llms-choosing-the-right-model-in-2026)
- [ML Journey — Best Open-Source LLMs by Use Case](https://mljourney.com/best-open-source-llms-in-2026-a-practical-guide-by-use-case/)
- [Microcenter — Best Local LLMs by Memory](https://www.microcenter.com/site/mc-news/article/best-local-llms-8gb-16gb-32gb-memory-guide.aspx)
- [LocalLLM.in — Ollama VRAM Requirements](https://localllm.in/blog/ollama-vram-requirements-for-local-llms)

---

*本報告由 Jarvis（Dispatch 網路研究 + Mac Code Task 落檔）產出。數據來自 2026 年 4 月初的公開 benchmark 和排行榜，可能隨時間更新。*

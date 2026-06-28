# LLM API 免費額度與強模型調查

日期：2026-06-28  
語言：繁體中文  
範圍：可透過 API 使用、目前仍有免費層/免費 preview/免費模型路由的 LLM 服務。

## 快速結論

如果目標是「現在就能拿免費額度做 prototype」，優先順序如下：

1. **Google Gemini Developer API**：最適合一般開發。免費層明確，模型強，長 context，AI Studio 入口順。
2. **Groq API**：最適合高速推理、agent loop、批次實驗。免費層限額透明，速度通常是最大優勢。
3. **OpenRouter free models**：最適合試很多開源/開權重模型。`*:free` 模型數量多，但穩定性與每日次數受限。
4. **GitHub Models**：最適合在 GitHub/Codespaces/VS Code 生態做模型比較與原型。免費 API preview，但 daily request 很少。
5. **Cloudflare Workers AI**：最適合 serverless/edge 小型功能。免費額度每天重置，但大型 LLM 很快吃完。
6. **Mistral Free mode / Cohere trial key**：適合低量測試與交叉驗證，不適合大量跑。

不建議把 **Together AI** 歸類為「免費 API」：官方文件寫明目前沒有 free trial，平台存取需要最低 5 美元 credit purchase。

## 最值得先試的組合

| 用途 | 首選 | 備選 |
| --- | --- | --- |
| 一般聊天、摘要、結構化輸出 | Gemini Developer API | GitHub Models |
| 高速 agent / tool loop | Groq | Gemini Flash / Flash-Lite |
| 免費試大量開源模型 | OpenRouter `:free` | Groq 免費層 |
| code / repo 問答 | Qwen Coder via OpenRouter free / Groq Qwen | GitHub Models |
| edge/serverless 小功能 | Cloudflare Workers AI | Gemini API |
| RAG embedding / rerank 小量測試 | Cloudflare Workers AI / GitHub Models | Cohere trial |

## 平台比較

| 平台 | 免費額度型態 | 可用強模型例子 | 優點 | 主要限制 |
| --- | --- | --- | --- | --- |
| Google Gemini Developer API | Free tier，input/output token 免費，Google AI Studio 可用 | Gemini 3/3.1/3.5 系列、Gemini 2.5 Pro/Flash、Flash-Lite | 綜合最強，長 context，多模態與工具鏈完整 | preview/experimental 模型限額更嚴，部分功能只在 paid tier |
| Groq | 每模型 RPM/RPD/TPM/TPD 免費限制 | `openai/gpt-oss-120b`, `llama-3.3-70b-versatile`, `qwen/qwen3-32b`, Llama 4 Scout | 速度很快，限額表清楚，OpenAI-compatible | 免費層 daily token/request 有上限，不是所有 frontier model |
| OpenRouter | `:free` 模型，20 RPM；未購買 credits 時 50 requests/day，購買至少 10 credits 後 1000 requests/day | `nvidia/nemotron-3-ultra-550b-a55b:free`, `openai/gpt-oss-120b:free`, `qwen/qwen3-coder:free`, `meta-llama/llama-3.3-70b-instruct:free`, `nousresearch/hermes-3-llama-3.1-405b:free` | 模型選擇最多，能用一個 API 試很多供應商 | 免費模型供應可能變動；未儲值時每日 50 次偏少 |
| GitHub Models | Playground 與 API free preview | OpenAI、Meta、Mistral、Cohere 等 Marketplace 模型，依目錄變動 | 和 GitHub/VS Code/Codespaces 整合好，適合比較模型 | Copilot Free 的 high tier 只有 50 requests/day；token/request 也有限 |
| Cloudflare Workers AI | 10,000 Neurons/day | Llama 3.3 70B, Llama 4 Scout, DeepSeek R1 Distill Qwen 32B, Qwen Coder 32B, GPT-OSS 120B, Gemma 4 26B | 部署到 edge 很方便，免費額度每天重置 | 免費 Neurons 對 70B/120B 這類大模型很快用完 |
| Mistral Studio/API | Free mode，無需信用卡，保守 rate limits | Mistral Large/Small/Code/OCR/Agents 系列依帳號可用性 | 歐系供應商，模型品質與工具調用不錯 | 官方沒有把免費數字整理得像 Groq/OpenRouter 那麼清楚 |
| Cohere | Evaluation/trial key | Command A+、Command R/Rerank/Embed 系列 | RAG、rerank、enterprise NLP 很有價值 | trial key 每月 1,000 API calls，偏評估用途 |
| Together AI | 目前不提供 free trial | Kimi/Qwen/Llama 等 serverless 模型 | 模型多，生產/專用 endpoint 能力強 | 需最低 5 美元 credit purchase，不算免費 |

## 目前免費層裡「最強」的候選

這裡的「最強」不是單一榜單排名，而是綜合模型規模、推理/程式能力、免費可用性、API 可操作性。

### 1. Gemini 3 / 3.1 / 3.5 系列

Gemini Developer API 是免費額度中最像「正規主力 API」的選項。Google pricing 頁面明確列出 Free tier，input/output token 免費；rate-limit 頁面也說明 Gemini API 以 RPM、TPM、RPD 計算，且限制套用在 project 而不是單一 API key。

建議用途：通用 reasoning、長文整理、多模態 prototype、需要穩定 SDK 與文件的 app。

### 2. OpenRouter 的大型免費模型

OpenRouter API 目前列出 22 個 `:free` 模型，其中值得注意：

- `nvidia/nemotron-3-ultra-550b-a55b:free`：1M context，從模型規模看是免費路由裡最誇張的候選之一。
- `nousresearch/hermes-3-llama-3.1-405b:free`：405B 級別開源系模型，適合長文、創作、一般 instruction。
- `openai/gpt-oss-120b:free`：120B 級別，OpenRouter 與 Groq/Cloudflare 都有類似路由，容易比較。
- `qwen/qwen3-coder:free`：1M context，偏 coding/repo 工作。
- `meta-llama/llama-3.3-70b-instruct:free`：穩定、通用、相對可預期。

建議用途：免費比較多模型、找便宜 fallback、測 open-weight 模型差異。

### 3. Groq 的高速免費模型

Groq 免費層官方列出每模型限制，例如：

- `openai/gpt-oss-120b`：30 RPM、1K RPD、8K TPM、200K TPD。
- `llama-3.3-70b-versatile`：30 RPM、1K RPD、12K TPM、100K TPD。
- `qwen/qwen3-32b`：60 RPM、1K RPD、6K TPM、500K TPD。
- `meta-llama/llama-4-scout-17b-16e-instruct`：30 RPM、1K RPD、30K TPM、500K TPD。

建議用途：agent loop、快速回覆、多次 retry、工具調用 prototype。Groq 不一定是最聰明，但很常是免費層裡最爽快的。

### 4. GitHub Models 的 high-tier 模型

GitHub Models 的定位不是大量免費算力，而是模型實驗入口。文件明確寫「free API usage is in public preview」，且 Copilot Free high-tier 限制為 10 RPM、50 RPD、每次 8K input / 4K output。適合測 prompt、比較模型，不適合作為大量 production fallback。

建議用途：GitHub repo 內 prototype、VS Code AI Toolkit、demo code。

### 5. Cloudflare Workers AI 的 edge 模型

Cloudflare 每天給 10,000 Neurons 免費額度，模型表包含 Llama 3.3 70B、Llama 4 Scout、DeepSeek R1 Distill Qwen 32B、Qwen Coder 32B、GPT-OSS 120B、Gemma 4 26B 等。但大型模型每 M token 的 Neurons 消耗高，免費層更適合小功能、短輸出、分類、rerank、embedding、低頻 edge inference。

建議用途：Cloudflare Workers 上的輕量 AI endpoint、分類、短摘要、低頻 bot。

## 實務建議

### 免費 API routing 策略

1. 主模型：Gemini Developer API。
2. 快速 fallback：Groq `llama-3.3-70b-versatile` 或 `openai/gpt-oss-120b`。
3. 開源模型比較：OpenRouter `:free`，但要處理每日 50/1000 request 上限。
4. GitHub/Codespaces demo：GitHub Models。
5. edge app：Cloudflare Workers AI。

### 要注意的坑

- 免費層常常不是 production SLA；preview/free model 會變動。
- OpenRouter 免費模型若帳戶沒購買過 credits，每天只有 50 次 `:free` request。
- GitHub Models 的 free API 是 public preview，不適合當正式服務依賴。
- Cloudflare Workers AI 的「10,000 Neurons/day」不是 10,000 token；大模型輸出很快會耗完。
- Together AI 文件已明確說沒有 free trial，不該放進免費 API 清單。

## 來源

- Google Gemini API pricing: https://ai.google.dev/gemini-api/docs/pricing
- Google Gemini API rate limits: https://ai.google.dev/gemini-api/docs/rate-limits
- Groq rate limits: https://console.groq.com/docs/rate-limits
- OpenRouter models API: https://openrouter.ai/api/v1/models
- OpenRouter limits: https://openrouter.ai/docs/api/reference/limits
- GitHub Models prototyping docs: https://docs.github.com/en/github-models/use-github-models/prototyping-with-ai-models
- Cloudflare Workers AI pricing: https://developers.cloudflare.com/workers-ai/platform/pricing/
- Cohere rate limits: https://docs.cohere.com/docs/rate-limits
- Mistral quickstart / Studio Free mode: https://docs.mistral.ai/getting-started/quickstart/
- Together AI billing credits: https://docs.together.ai/docs/billing-credits

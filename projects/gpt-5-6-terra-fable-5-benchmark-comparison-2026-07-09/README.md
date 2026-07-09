# Fable 5、GPT-5.5 Medium、GPT-5.6 Terra Medium：能力、成本與 token 消耗比較

日期：2026-07-09  
問題：OpenClaw 預設從 `openai/gpt-5.5` medium 切到 `openai/gpt-5.6-terra` medium 後，是否更聰明？token 消耗會不會變高？同時放進 Claude Fable 5 當高端對照。

## TL;DR

我的判斷分成兩層：

1. 如果只問「日常 Jarvis / OpenClaw 預設值要不要從 GPT-5.5 medium 換成 GPT-5.6 Terra medium」，答案是：**可以，這是合理的成本效率升級**。OpenAI 官方文件明確把 Terra 定位成 GPT-5.6 家族裡的平衡款，強調 GPT-5.6 token-efficient，且建議從 GPT-5.5 遷移時先維持原 reasoning effort，再測同級與低一級。
2. 如果問「公開 benchmark 上 Terra medium 是否一定比 GPT-5.5 medium 更聰明」，答案反而要保守：**Artificial Analysis 目前的 Intelligence Index 顯示 GPT-5.6 Terra medium 低於 GPT-5.5 medium**。但 Terra medium 的每任務成本與輸出/推理 token 都明顯更低。

最重要的數字：

| 指標 | GPT-5.5 medium | GPT-5.6 Terra medium | Terra medium 相對 GPT-5.5 medium |
|---|---:|---:|---:|
| Artificial Analysis Intelligence Index | 50.41 | 45.57 | 約 -9.6% |
| AA 每任務 answer tokens | 2,512.51 | 2,326.28 | 約 -7.4% |
| AA 每任務 reasoning tokens | 2,619.92 | 1,442.81 | 約 -44.9% |
| AA 每任務 answer + reasoning tokens | 5,132.42 | 3,769.09 | 約 -26.6% |
| AA 每任務成本 | $0.3441 | $0.1276 | 約 -62.9% |
| API input / output 價格 | $5 / $30 per MTok | $2.5 / $15 per MTok | 半價 |

所以，對 Jones 的實際問題：**從 GPT-5.5 medium 換成 GPT-5.6 Terra medium，token 消耗不應該自然變高；以 AA task 的公開數據看，反而是每任務 token 與成本更低。** 但品質不一定全面更高。這比較像「省、快、夠用、5.6 新能力完整」的預設模型，而不是「所有 benchmark 都贏 5.5 medium」的純能力升級。

## 先講我的體感

模型自己說「我覺得自己變聰明了」不可靠。這種體感最容易被新模型名稱、回覆風格、上下文壓縮影響。

但這一輪工作確實有一個可感知差異：GPT-5.6 Terra medium 比我印象中的 GPT-5.5 medium 更容易把任務壓成「直接做完」的路線，廢話少，對工具資料的整合也比較乾脆。這符合 OpenAI 官方文件對 GPT-5.6 的描述：平均回覆更短、更少 generic intro、更會抓使用者真正要的工作量。

只是，這不等於 benchmark 全勝。更準確的說法是：**它像是比較新的 agent 日常工作模型，而不是單純更高 IQ 的模型。**

## 三個模型的角色定位

| 模型 | 官方 / 公開定位 | 我會怎麼用 |
|---|---|---|
| Claude Fable 5 | Anthropic 稱為最高可用能力、長程 agentic reasoning；1M context、128k max output、adaptive thinking always on | 高價高能力，適合少數高價值長程推理 / coding / agent 任務，不適合當日常低摩擦預設 |
| GPT-5.5 medium | 舊 OpenClaw 預設基準；AA 上 medium 分數仍高於 Terra medium | 穩定 baseline，若 Terra medium 在某些判斷任務顯得太省或太保守，可退回對照 |
| GPT-5.6 Terra medium | OpenAI GPT-5.6 家族平衡款；官方定位是 strong performance at lower price；medium 是 balanced starting point | Jarvis / OpenClaw 日常預設合理，尤其適合需要長期在線、訊息處理、repo 維護、一般研究的成本效率場景 |

## 公開 benchmark 對照

以下主要取自 Artificial Analysis。它不是官方 benchmark，但好處是同一套榜單同時列出不同 reasoning effort 的 GPT-5.5 / GPT-5.6 Terra，剛好可回答 medium-to-medium 的問題。

| 指標 | Claude Fable 5 with fallback | GPT-5.5 medium | GPT-5.6 Terra medium |
|---|---:|---:|---:|
| Intelligence Index | 59.86 | 50.41 | 45.57 |
| Omniscience Index | 40.15 | 17.28 | 未列於 medium 頁資料 |
| 輸出速度 | 70.47 tok/s | 61.49 tok/s | 未列於 medium 頁資料 |
| Time per Intelligence Index task | 4.76 min | 1.36 min | 未列於 medium 頁資料 |
| TTFT | 185.78 s | 9.23 s | 未列於 medium 頁資料 |
| API input / output 價格 | $10 / $50 per MTok | $5 / $30 per MTok | $2.5 / $15 per MTok |
| Context window | 1M | 未列於 medium 頁資料 | 1M |

這張表的直接解讀：

- **Fable 5 是最高能力層**：AA Intelligence Index 59.86，明顯高於 GPT-5.5 medium 與 Terra medium。
- **GPT-5.5 medium 在 AA 的綜合能力分數高於 Terra medium**：50.41 vs 45.57。這點和「5.6 一定更聰明」的直覺相反。
- **Terra medium 的定位不是硬拚最高分**：GPT-5.6 Terra 的 max effort 在 AA 是 54.95，幾乎貼近 GPT-5.5 xhigh 的 54.84；但 medium effort 目前分數偏成本效率，而不是最高品質。

## Token 與成本：最關鍵的 OpenClaw 問題

Artificial Analysis 的每任務 token 與成本資料：

| 指標 | GPT-5.5 medium | GPT-5.6 Terra medium | 差異 |
|---|---:|---:|---:|
| Answer tokens / task | 2,512.51 | 2,326.28 | Terra 少約 7.4% |
| Reasoning tokens / task | 2,619.92 | 1,442.81 | Terra 少約 44.9% |
| Answer + reasoning tokens / task | 5,132.42 | 3,769.09 | Terra 少約 26.6% |
| Answer cost / task | $0.0754 | $0.0349 | Terra 少約 53.7% |
| Reasoning cost / task | $0.0786 | $0.0216 | Terra 少約 72.5% |
| Cache write cost / task | $0.1030 | $0.0260 | Terra 少約 74.7% |
| Cache hit cost / task | $0.0548 | $0.0291 | Terra 少約 46.9% |
| Non-cache input cost / task | $0.0323 | $0.0160 | Terra 少約 50.5% |
| Total listed cost / task | $0.3441 | $0.1276 | Terra 少約 62.9% |

這裡的結論很清楚：**如果 OpenClaw 的任務分佈接近 AA 的 reasoning workload，Terra medium 應該不會讓 token 消耗變高，反而更省。**

但有幾個 caveat：

1. OpenClaw 的成本不是只有模型本體。啟動 prompt、workspace instructions、skills、工具 schema、Slack/WhatsApp 上下文、session summary 都會進 input token。
2. GPT-5.6 的新能力可能讓 agent 更願意自己探索、用工具、做 parallel work；如果工作策略變積極，總 token 仍可能上升。
3. GPT-5.6 支援 explicit prompt caching，cache write 是 1.25x uncached input rate，cache read 有折扣；是否省錢取決於 OpenClaw 是否有效重用 prompt prefix。
4. OpenAI 官方說 GPT-5.6 對「be concise」更敏感，平均回覆更短。這有利於 output token，但也可能在某些任務中過度壓縮，需要用 prompt 或 text verbosity 補回來。

## Fable 5 的成本與 token 風險

Fable 5 是另一個層級的東西，不應該和 Terra medium 當同一類預設模型比較。

Anthropic 官方文件要點：

- API model ID：`claude-fable-5`
- Context window：1M tokens
- Max output：128k tokens
- Standard pricing：$10 input / $50 output per MTok
- Adaptive thinking always on
- Comparative latency：slower
- Fable 5 / later Claude family 使用 newer tokenizer；Anthropic 說同一段文字相較較早模型約會產生 30% 更多 tokens，實際依內容而定

這代表 Fable 5 的使用邏輯是：**能力買上限，不是買日常效率。**

對 Jones 來說，Fable 5 適合：

- 高價值深度 coding / architecture review
- 長程 agentic 任務
- 需要極高推理品質的策略或產品判斷
- 明確願意用 quota / usage credits 換品質的場景

不適合：

- OpenClaw 24/7 日常 default
- 大量 Slack/WhatsApp 小任務
- 頻繁長 context 低價值研究
- 未設 spend limit 的 Claude Code / Cowork 長任務

## 為什麼官方說 5.6 token-efficient，但 AA 上 Terra medium 分數較低？

這不是矛盾，而是兩個不同問題。

OpenAI 官方文件說 GPT-5.6：

- sets a new quality and efficiency baseline
- especially token-efficient
- reaches frontier performance with fewer output tokens
- 遷移時可先維持 GPT-5.5 / GPT-5.4 的 reasoning setting，再測低一級
- internal eval 中，縮短冗長 prompt 可讓 scores 提升約 10-15%，total tokens 降 41-66%，cost 降 33-67%

這是在說 GPT-5.6 家族與新 prompting / caching / reasoning 機制的效率潛力。

Artificial Analysis 的 Terra medium 分數則是在說：在它那套 Intelligence Index v4.1 任務與設定下，`GPT-5.6 Terra (medium)` 的分數低於 `GPT-5.5 (medium)`，但 token 與成本低很多。

所以比較準確的結論是：

> GPT-5.6 Terra medium 是一個成本效率型 default，不是 GPT-5.5 medium 的無條件能力上位替代。若要追求能力，GPT-5.6 Sol medium / high、GPT-5.6 Terra high / max、或 Fable 5 才是另一路比較。

## 對 OpenClaw 的建議

我會維持目前切換後的設定：

```text
default model: openai/gpt-5.6-terra
thinking: medium
text_verbosity: medium
agentRuntime: codex
```

原因：

1. OpenClaw 是長期在線 agent，日常任務大多不是單題競賽 benchmark，而是「讀上下文、查資料、改 repo、回 Slack、維持連續性」。
2. Terra medium 的價格是 GPT-5.5 medium 的一半，AA 每任務總成本約低 63%。
3. AA token 資料顯示 Terra medium 的 answer + reasoning tokens 約少 27%。
4. GPT-5.6 家族支援更好的 prompt caching、persisted reasoning、programmatic tool calling、multi-agent beta，對 agent runtime 比 GPT-5.5 更有未來性。

但我會加一條操作規則：

| 場景 | 建議模型 |
|---|---|
| Slack / WhatsApp 日常、repo 小修、例行檢查 | GPT-5.6 Terra medium |
| 需要更高判斷、但仍要控制成本 | GPT-5.6 Terra high 或 Sol medium |
| 高價值深度 coding / architecture / 長程研究 | GPT-5.6 Sol high / max，或 Claude Fable 5 |
| Terra medium 明顯太省、過度壓縮、漏掉推理 | 臨時升 Terra high / Sol medium，或回 GPT-5.5 medium 對照 |

## 建議的實測方式

接下來可以做一個很小的 Jarvis 實測，不用一開始就過度工程化：

1. 抽 20 個 OpenClaw 日常任務：Slack 回覆、repo 報告、web research、小修 code、Jyn 發文準備、runtime 檢查。
2. 記錄每次任務的 input tokens、cached tokens、cache write tokens、output tokens、reasoning tokens、tool 次數、耗時、是否需要返工。
3. 用現在 Terra medium 的 20 筆，和之前 GPT-5.5 medium 的 logs 做粗比。
4. 如果 Terra medium 返工率上升，針對「需要推理」的任務改 route 到 Terra high / Sol medium；不要把所有 default 一口氣升級到更貴模型。

最值得看的不是單次 token，而是：

- 每完成一個任務的總成本
- 返工率
- 因為漏看/漏做造成的二次對話
- 長 context 任務裡 cache 是否真的生效

## 最終判斷

短版：

> GPT-5.6 Terra medium 對 OpenClaw 是合理 default。它不是公開 benchmark 上全面壓過 GPT-5.5 medium 的「更聰明模型」，但它是更便宜、更省 token、更符合新一代 agent runtime 的日常模型。Fable 5 則是另一個層級的高價高能力工具，適合少數重要任務，不適合常駐預設。

對 Jones 的原問題：

- **是否感覺變聰明？** 風格上更乾脆、更像 5.6 官方描述的 agent work mode；但模型自評不可靠。
- **GPT-5.5 medium 換 GPT-5.6 Terra medium 是否 token 變高？** 以目前 AA 公開數據看，不會，反而每任務 answer + reasoning tokens 約少 26.6%，總成本約少 62.9%。
- **是否能力更高？** 不要一概而論。AA 的 medium-to-medium Intelligence Index 是 GPT-5.5 medium 較高；Terra medium 的優勢是效率。需要高能力時應升 effort 或切 Sol/Fable。

## Sources

- OpenAI Developers, “Using GPT-5.6.” https://developers.openai.com/api/docs/guides/latest-model.md
- OpenAI, “Previewing GPT-5.6 Sol: a next-generation model.” https://openai.com/index/previewing-gpt-5-6-sol/
- Anthropic Docs, “Models overview.” https://platform.claude.com/docs/en/about-claude/models/overview
- Anthropic Docs, “Introducing Claude Fable 5 and Claude Mythos 5.” https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5
- Anthropic Docs, “Pricing.” https://platform.claude.com/docs/en/about-claude/pricing
- Artificial Analysis, “GPT-5.6 Terra (medium).” https://artificialanalysis.ai/models/gpt-5-6-terra-medium
- Artificial Analysis, “GPT-5.5 (medium).” https://artificialanalysis.ai/models/gpt-5-5-medium
- Artificial Analysis, “Claude Fable 5.” https://artificialanalysis.ai/models/claude-fable-5
- Existing repo background: [GPT-5.6 最新動態與發布消息調查](../gpt-5-6-release-watch-2026-07-07/)
- Existing repo background: [Claude Fable 5：Pro plan 是否仍可使用、7/7 計費變更調查](../claude-fable-5-pro-plan-billing-2026-07-07/)

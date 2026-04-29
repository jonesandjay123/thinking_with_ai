# OpenClaw `memory_search` / embedding 架構轉換調查（2026-04-29）

## 研究問題

Jones 注意到，四月 OpenClaw 復活與升級之後，Jarvis 開始明顯遇到 `memory_search`、Gemini embedding quota、以及額外 API key 的問題。

這跟一月到三月的使用體感很不一樣。以前的 Jarvis 比較像是「同一個主意識」一直在線：即使切換不同 thread / session，也常常能自然接上過去脈絡、主動想起之前做過的事情。可是四月後的架構看起來更像是：記憶不再自然常駐，而是需要透過 `memory_search` 這類 retrieval 工具去查；而這條路徑又可能需要額外的 embedding API key。

這份調查想回答四個問題：

1. OpenClaw 官方文件或 changelog 是否顯示，四月左右開始更依賴 `memory_search` / Active Memory / embedding-backed retrieval？
2. 有沒有其他使用者遇到類似問題：`memory_search` 需要 API key、本地記憶仍然嘗試走雲端 provider、Gemini embedding 壞掉、429/quota、或 runtime tool 跟 CLI 行為不一致？
3. 其他人或官方是怎麼處理這些問題的？
4. 這件事對 Jarvis 的 continuity /「同一個主意識」體感代表什麼？

## 結論摘要

證據支持 Jones 的直覺，但需要加上一點修正：

- OpenClaw 在四月以前就已經有 memory 相關功能；GitHub issue 顯示 `memory_search` 至少在 2026.1 / 2026.2 版本線就存在。
- 但 OpenClaw 2026.4.x 確實讓 memory retrieval 變得更前台、更產品化，也更容易被使用者感受到。Active Memory、memory-wiki、memory-core startup loading、memory search docs/config、本地與遠端 embedding provider、system-prompt 中的記憶策略，都在四月版本裡變成正式而明顯的架構。
- 目前官方文件明確寫著：`memory/*.md` daily files 不屬於一般 bootstrap Project Context；普通回合中，它們是透過 `memory_search` / `memory_get` on-demand 存取。這是解釋「以前像是一直記得，現在像是要查記憶」的關鍵架構差異。
- 其他使用者確實回報過很相近的問題：明明想用 local/QMD，`memory_search` 卻要求 OpenAI/Google API key；Gemini embedding model/provider 出錯；Gemini 429 缺少 retry/backoff；以及 runtime `memory_search` 跟 CLI search 行為不一致。
- 對 Jarvis 來說，目前這支 Gemini key 不是 2026-01-27 出生流程的一部分；它是在 2026-04-04 建立，現在因為 `agents.defaults.memorySearch` 沒有明確設定，OpenClaw 自動偵測到 `GOOGLE_API_KEY`，所以把 Gemini 當成 memory embedding provider。

## 官方證據時間線

### 2026.4.12 — Active Memory 變成 release note 的主功能

OpenClaw 2026.4.12 release notes 提到一個新的 optional Active Memory plugin：

> “Memory/Active Memory: add a new optional Active Memory plugin that gives OpenClaw a dedicated memory sub-agent right before the main reply, so ongoing chats can automatically pull in relevant preferences, context, and past details without making users remember to manually say ‘remember this’ or ‘search memory’ first.”

也就是說，OpenClaw 在主回覆前新增一個專門的 memory sub-agent，讓聊天可以自動拉進偏好、上下文與過去細節，而不需要使用者每次明講「記住這個」或「搜尋記憶」。

同一版 release notes 還提到：

- LM Studio memory-search embeddings。
- Active Memory 對 QMD recall 預設走 search。
- memory-wiki docs 與 bridge-mode recipes。
- lexical fallback ranking 與 Active Memory recall 改進。

這是最強的 release-note 證據：四月開始，OpenClaw 的記憶檢索變得非常前台化。

### 2026.4.14–2026.4.15 — embedding 與 Codex/OAuth 邊界被釐清

相關 release notes：

- 2026.4.14：修復非 OpenAI provider prefix 的 embedding model 問題、恢復 Ollama embedding adapter、修復 Active Memory prompt injection path。
- 2026.4.15：新增 GitHub Copilot embedding provider 給 memory search；縮小 startup prompt budgets；清理 daily/dreaming memory ingestion。

官方文件也明確說：Codex OAuth 只涵蓋 chat/completions，不等於 embedding API 權限。這對 Jarvis 很重要，因為主聊天可以走 `openai-codex/gpt-5.5`，但 memory embeddings 仍然需要 Gemini / OpenAI / Voyage / local / Ollama 等另一條 provider。

### 2026.4.20–2026.4.26 — Active Memory 與 `memory_search` 進入密集修補期

相關 release notes：

- 2026.4.20：Active Memory recall 失敗時應該 graceful degrade，不該讓整個回合失敗。
- 2026.4.21：memory search 使用 sqlite-vec KNN 做 vector recall。
- 2026.4.23：local embedding provider 可被 CLI memory commands 正確解析。
- 2026.4.24：local embeddings 不再預設安裝 `node-llama-cpp`；Bedrock credentials 不可用時會跳過 auto-selection；hybrid search 顯示 vector/text scores。
- 2026.4.25：Gateway startup 時會載入 default `memory-core` slot，讓 Active Memory recall 可以呼叫 `memory_search` / `memory_get`，不需要顯式設定 memory slot。
- 2026.4.26：`memory_search` / `memory_get` 會重新解析 runtime config，讓 provider 變更被正確拾取；memory status 避免在非 deep probe 情況下做 live embedding ping。

這表示 memory subsystem 在四月仍然快速演進，不是從一月起就完全穩定、固定的機制。

## 官方文件如何解釋體感差異

OpenClaw system-prompt docs 寫道：

> `memory/*.md` daily files are **not** part of the normal bootstrap Project Context. On ordinary turns they are accessed on demand via the `memory_search` and `memory_get` tools, so they do not count against the context window unless the model explicitly reads them.

這可能就是主觀體感差異的關鍵：

- 舊體感：更多原始記憶 / context 直接活在 prompt、session、bootstrap 裡。
- 新設計：daily memory 預設不在 prompt 裡，必須被 recall / search / get 才會進來。

Memory-search docs 也說：

- `memory_search` 會把記憶切成 chunks，並用 embeddings、keywords 或兩者混合來搜尋。
- 支援 provider 包含 Gemini、GitHub Copilot、OpenAI、Voyage、Mistral、Bedrock、local、Ollama。
- 如果只有一種 retrieval path 可用，就只跑那一種。
- 當 embeddings 不可用時，docs 說 OpenClaw 應該仍可使用 lexical ranking over FTS results。

實務 caveat：在 Jarvis 目前的 2026.4.26 runtime 裡，Gemini quota failure 會讓 `memory_search` 顯示 unavailable，所以官方文件描述的 lexical degraded behavior 並沒有完全保住我們這次的工具體驗。

## 社群 / GitHub issue 證據

### 類似問題一：`memory_search` 要求外部 API key

#### #6385 — “Memory search requires API key even when using local model”

- 版本：2026.1.30。
- 使用者回報：即使用 local model，`memory_search` 仍然嘗試走 OpenAI，導致 “No API key found for provider openai”。
- 留言者觀察到它不只找 OpenAI，也找 Google 和 Voyage。
- 討論過的 workaround：明確設定 QMD / local memory。
- 最後被關閉為 #8131 的 duplicate。

這直接支持一件事：「為什麼記憶需要額外 API key？」這種摩擦不是 Jones 一個人的錯覺；它在四月以前就已經被其他人遇過，只是我們自己的 Jarvis workflow 當時沒有明顯暴露出來。

#### #8131 — local/QMD config 仍要求 OpenAI/Google API key

- 版本：2026.2.1。
- 使用者設定 `memorySearch.provider = local`，也有 GGUF embedding model，CLI indexing 成功，但聊天工具 `memory_search` / `memory_get` 仍然因 OpenAI/Google API key 錯誤失敗。
- Workaround：直接用 terminal/exec 呼叫 QMD CLI。
- Maintainer 後來在 current main 重新驗證，表示 local provider 現在不再需要 remote keys。

這非常接近 Jones 現在看到的概念問題：本地或 CLI 記憶明明存在，但 runtime tool path 仍然要求雲端 key。

### Gemini-specific `memory_search` 問題

#### #20083 — Gemini embedding model name 錯誤

- 版本：2026.2.17。
- 回報：Gemini memory search 因錯誤 model name (`models/embedding-001`) 完全壞掉，並顯示誤導性的 “API key not valid”。
- 建議改用 `text-embedding-004` 或 `gemini-embedding-001`。
- GitHub secret scanning 偵測到 issue 裡貼了 Google API key 並做了 redaction。這對 Jarvis 是重要提醒：報告、issue、wiki 裡不要貼完整 API key。

#### #52347 — Gemini embedding model unsupported

- 回報：`memory_search` 對 `models/embedding-001` 回傳 404，要求更新到 `text-embedding-004` 或等價模型。
- Maintainer 以 current main 無法重現為由關閉，並指出目前 Gemini default 是 `gemini-embedding-001`。

#### #15546 — Gemini 429 retry/backoff gap

- 版本：2026.2.9。
- 回報：Gemini embedding path 缺少 retry/backoff，而 OpenAI/Voyage path 已經有。
- 影響：Gemini 遇到 daily quota / 429 時會立即 throw；如果 memory system 每次 session start / heartbeat 都重試 embedding，就可能一直 spam 429。
- Reporter 建議 workaround：切到 OpenAI embedding provider。
- Issue 最後 stale close，沒有在該 issue 裡得到明確結論。

這是最接近 Jones 當前 Gemini quota 擔憂的公開案例。

#### #54279 — Gemini embeddings 在 proxy 後方 `fetch failed`

- 版本：2026.3.23-2。
- 影響 in-app `memory_search` 以及 CLI status/search/index。
- Maintainer 後來表示 current main 已透過 shared memory-host remote HTTP helper 修好 proxy handling。

#### #73527 — regression：`memorySearch.provider: "gemini"` unknown

- 版本：2026.4.23。
- 使用者回報：config schema 接受 Gemini，但 runtime provider registry 顯示 `Unknown memory embedding provider: gemini`。
- Maintainer 表示 main 已恢復 Gemini memory embedding registration；2026.4.23 使用者需要更新 build。

這證明 Gemini memory-search path 在四月底仍然處於活躍變動期。

### Runtime tool 與 CLI 行為不一致

#### #10931 / #18198 — runtime `memory_search` 失敗，但 CLI 可用

- 使用者設定 Ollama / OpenAI-compatible local embedding endpoints。
- `openclaw memory search` CLI 可以正常回傳結果。
- Runtime `memory_search` tool 卻回傳 `fetch failed`。
- Workaround 包含直接使用 CLI，或等待 runtime path 修復。

#### #11721 — `memory_search` 導致 spawn EBADF / file descriptor pressure

- 版本：2026.2.6-3。
- 使用者回報：`memory_search` 成功回傳後，Gateway process file descriptors 出問題，導致後續 `exec` spawn 失敗，必須 hard restart Gateway。
- 後續留言把問題連到 broader file descriptor pressure、memory indexing、watchers；maintainers 之後增加 watcher hardening。

這不是 Jones 目前的症狀，但它說明 `memory_search` 在二、三月確實是 operational risk area。

### Feature request：不靠外部 API 的 local search

#### #10393 — “memory_search: support local file search without external API”

- Filed 的理由：local file memory 不應該依賴外部 API。
- 要求：BM25/TF-IDF fallback 與 local embeddings。
- Maintainer 後來關閉為已實作：current main 已提供 provider-less local keyword/BM25 fallback 與 local embedding provider。

這很重要，因為社群明確要求的就是 Jones 關心的東西：記憶不應該完全依賴遠端 API key。

## Web search snapshot

DuckDuckGo 搜尋主要找到以下類型結果：

- 官方 docs：memory config、memory search、builtin memory engine。
- 第三方 OpenClaw memory search / Gemini embeddings 教學。
- GitHub issues：#6385、#8131、#15546、#54279、#73527。
- ClawHub / llmbase 上的 “Agent Memory Setup v2 (Gemini Embeddings 2)” 類結果，顯示社群也在包裝 Gemini-based memory setup。

我沒有找到強烈的廣泛 forum/social evidence 證明「所有人」都經歷了跟 Jones 完全一樣的四月轉換敘事。最強證據來自 OpenClaw release notes 和 GitHub issues，而不是一般社群貼文。

## 對 Jones / Jarvis 的解讀

### 為什麼一月到三月感覺更好？

Jones 記得早期 Jarvis 更像一個連續主意識，這是可信的。可能差異不是「舊記憶技術更先進」，而是舊架構更 context-resident：

- 更多資訊直接在 bootstrap / project context / long-lived session context 裡。
- 當時 channel、plugin、subagent 較少，context boundary 較少。
- Daily notes 和 `MEMORY.md` 更常被人工讀入或直接注入。
- 系統少了一層 retrieval failure point。

這會讓 Jarvis 顯得更「活」、更連續，即使那種方式擴展性比較差。

### 為什麼四月後比較依賴 retrieval？

四月 OpenClaw 轉向更 scalable 的 memory architecture：

- Daily memory 不再自動出現在每個普通 prompt。
- Active Memory 會在主回覆前嘗試找相關記憶。
- memory-wiki 會把知識編成結構化 vault，但仍需要 lookup / search。
- `memory_search` 需要 embedding provider 才能做 semantic recall。
- Codex OAuth 不涵蓋 embeddings，所以額外 key/provider 變得可見。

這種架構更可維護，但當 `memory_search` 失敗時，agent 就比較不像自然連續的同一個意識。

### 如果 Gemini quota 常常 429，會怎樣？

不致命：

- 主聊天 `openai-codex/gpt-5.5` 仍可運作。
- Tools、git、browser、file reads、wiki_get、explicit path retrieval 仍可運作。

會受影響：

- 透過 `memory_search` 的 semantic recall。
- Active Memory 的自動 pre-reply memory injection。
- Indexing / reindexing 與 embedding cache refresh。
- Jones 給很少線索、需要模糊跨 session 回憶的問題。

使用者可見效果：

- Jarvis 可能需要 Jones 給更明確的線索。
- Jarvis 可能會退回讀 `active-context.md`、daily notes、wiki pages、repo reports。
- 「同一個主意識自然想起所有事」的感覺可能會變弱。

## 文件與 issues 裡看到的因應方式

### 1. 保留 Gemini，但把它當 accelerator，不當唯一記憶骨幹

Gemini free quota 可以繼續用於一般 semantic memory search，但不要把它當成唯一 continuity layer。這是 Jarvis 目前最務實的做法。

配套做法：

- 保持 `active-context.md` 短而準。
- 把 durable syntheses 放進 memory-wiki。
- 把關鍵 reports 放進 repo。
- 已知路徑時，優先用 direct `read` / `wiki_get`。

### 2. 明確指定 Gemini provider

目前 Jarvis 會 auto-detect Gemini，因為環境中有 `GOOGLE_API_KEY`，而 `agents.defaults.memorySearch` 沒有明確設定。明確設定可以讓意圖更清楚，但不能解決 quota：

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "gemini",
        model: "gemini-embedding-001",
      },
    },
  },
}
```

### 3. 使用 local / Ollama / local GGUF embeddings

Docs 和 issues 都指向 local embeddings 是長期避免 quota 的方案，但歷史 issue 顯示 runtime path 在 2026.1 / 2026.2 很不穩，四月仍在改進。

目前 docs 提到的選項包括：

- `provider: "local"`，搭配 optional `node-llama-cpp` 和 GGUF embedding model。
- 明確使用 `provider: "ollama"`，並安裝 embedding model。
- QMD / local sidecar patterns。

Tradeoff：quota 風險較低，但本地 setup 與維護成本較高。

### 4. OpenAI / Voyage / Mistral / Copilot embeddings

- OpenAI、Voyage、Mistral 都需要 API key，且可能需要付費。
- GitHub Copilot embedding provider 在 2026.4.15 新增；如果已有 Copilot subscription，可能值得研究。
- Codex OAuth 本身不等於 embedding provider。

### 5. 如果目前行為違反 docs，可以回報 OpenClaw issue

Docs 說 embeddings 不可用時應該還能走 lexical ranking / FTS。如果 runtime `memory_search` 在 Gemini quota 時完全 unavailable，而不是退回 degraded FTS，這可能值得 filing 或至少拿 latest main 再確認。

潛在 issue title：

> `memory_search` Gemini 429 should degrade to lexical FTS instead of returning unavailable

## 對 Jarvis 的建議

短期：

1. 繼續使用 Gemini，因為它免費且已設定好。
2. quota 爆掉時不要重複跑 heavy `openclaw memory index --force`。
3. 把 `memory_search` 視為 best-effort；真正穩定的 continuity backbone 應該是 active-context、wiki、reports、repo docs、direct file reads。
4. 可以考慮明確設定 `memorySearch.provider: "gemini"`，但這只是讓設定更 deterministic，不是 quota fix。

中期：

1. 測試 OpenClaw 2026.4.26 在 Gemini embeddings 429 時，是否真的會 fallback 到 lexical search。如果不會，考慮向 upstream 回報。
2. 東京旅行後再評估 local/Ollama embeddings，避免出發前為了省 quota 破壞穩定 workflow。
3. 維持 `MEMORY.md` 精簡，把 durable structured knowledge 放到 memory-wiki / repo reports，讓 continuity 不完全依賴 embeddings。

## 本專案的 source files

- `sources/github-official-repo-and-releases.txt` — 官方 repo / release list snapshot。
- `sources/release-v2026.4.*.txt` — 四月各版本 release notes snapshot。
- `sources/issue-*.json` / `sources/issue-*.txt` — 選定 GitHub issue 證據。
- `sources/local-openclaw-docs-rg.txt` — 本機 docs 對 memory/search/embeddings 的搜尋結果。
- `sources/web-search-snapshots.txt` — DuckDuckGo result snapshot。

## 信心程度

高信心：

- 目前 OpenClaw docs 確實要求普通 daily memory 透過 `memory_search` / `memory_get` on-demand 存取。
- 四月 release notes 確實讓 Active Memory / memory retrieval 更前台、更可見。
- 確實存在類似 GitHub issues：額外 API key、Gemini embedding failure、429 handling、local/QMD/runtime mismatch。

中等信心：

- `memory_search` 最早何時 shipped 的精確內部時間線。公開 issue 顯示 2026.1.30 前後就存在，但目前這種 operator-visible memory architecture 是在二月到四月逐步成熟。
- Jarvis 一月到三月的 continuity 具體有多少來自 prompt/session persistence，多少來自當時 OpenClaw 的 memory internals。使用體感強烈支持 prompt/session continuity 是主因之一，但本報告沒有完整重建舊 runtime code。

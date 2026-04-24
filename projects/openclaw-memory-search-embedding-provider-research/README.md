# OpenClaw `memory_search` Embedding Provider 調查報告

**日期：** 2026-04-24  
**背景：** Jones 問為什麼 OpenClaw 的 `memory_search` 會使用 Gemini embeddings，但主模型/執行 runtime 明明是 OpenAI Codex；以及「如果要改 OpenAI embedding，為什麼還需要另外提供 OpenAI API key」這件事是否正常。

## 摘要結論

OpenClaw 的 `memory_search` 是一套獨立於聊天/ coding 模型 runtime 的檢索與 embedding 子系統。也就是說，主回覆模型使用 `openai-codex/gpt-5.5`，並不代表 OpenAI embeddings 也自動可用。

目前觀察到的行為符合 OpenClaw 官方文件設計：

- `memory_search` 使用 hybrid retrieval：向量 embedding + BM25 關鍵字搜尋。
- 如果沒有明確指定 provider，embedding provider 會自動偵測。
- 自動偵測會依照官方文件列出的順序，選第一個可用 provider。
- OpenAI embeddings 需要 OpenAI API key（`OPENAI_API_KEY` 或 `models.providers.openai.apiKey`）。
- 官方文件明確寫到：OpenAI Codex OAuth 只涵蓋 chat/completions，不滿足 embedding requests。
- 如果本機有 Google/Gemini credentials，但沒有 OpenAI API-key auth，OpenClaw 會選 Gemini 做 memory embeddings 是合理結果。

套到目前 Jarvis Mac Mini 的狀況，就是：Codex 負責主 agent 模型，Gemini 負責 memory embeddings。這不是設定明顯壞掉，而是 OpenClaw 把「主模型」和「記憶搜尋 embedding」拆成兩層。

真正的缺點是營運層面：Gemini 免費層 embedding quota 可能偶爾撞到 `429`，導致 semantic memory search 暫時不可用。這時候記憶檔案本身不會壞，OpenClaw 仍可透過關鍵字搜尋或直接讀取本地 Markdown 檔案退化處理，但語意回憶的品質與便利性會下降。

## 為什麼這件事容易讓人困惑

混淆點在於「OpenAI」其實有兩個不同的認證/使用面：

### 1. OpenAI Codex OAuth

- 用於 OpenClaw 的 Codex runtime / coding agent 路徑。
- 目前本機主模型觀察到是：`openai-codex/gpt-5.5`。
- 不等於可以呼叫 OpenAI embeddings API。

### 2. OpenAI API key

- 用於 OpenAI platform API endpoint，例如 `/v1/embeddings`。
- OpenClaw 的 `openai` memory embedding provider 需要這種 API key。
- 常見設定是 `OPENAI_API_KEY` 或 `models.providers.openai.apiKey`。

所以「我們現在用 OpenAI」這句話對主模型/聊天/coding 是對的，但對 memory embeddings 來說還不夠。

## 官方文件調查

### Memory overview

OpenClaw 的記憶架構本質上是 file-first：

- `MEMORY.md`：長期記憶。
- `memory/YYYY-MM-DD.md`：每日筆記。
- `DREAMS.md`：可選，用於 dreaming/consolidation review。

官方文件描述 `memory_search` 是一個在這些記憶檔上做 semantic search 的工具，而且只要有支援 provider 的 embedding API key，通常就會自動運作。

來源：<https://docs.openclaw.ai/concepts/memory>

### Memory search 文件

Memory search 文件提到：

- `memory_search` 會把 notes 切成 chunks。
- Retrieval 可使用 embeddings、keywords，或兩者並用。
- 如果有 GitHub Copilot subscription，或已設定 OpenAI、Gemini、Voyage、Mistral API key，memory search 會自動啟用。
- 可以在 `agents.defaults.memorySearch.provider` 明確指定 provider。
- 若不想使用 API key，可用 local embeddings：`provider: "local"`，但需要 `node-llama-cpp`。

來源：<https://docs.openclaw.ai/concepts/memory-search>

### Memory config reference

Memory config reference 是最關鍵的官方來源。它說所有 memory search 設定都在：

```json5
agents.defaults.memorySearch
```

Provider selection 範例：

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "openai", // 或 gemini, local, github-copilot, etc.
        model: "text-embedding-3-small",
        fallback: "none"
      }
    }
  }
}
```

官方文件列出的 auto-detection order：

1. `local` — 只有在 `memorySearch.local.modelPath` 已設定且檔案存在時。
2. `github-copilot` — 如果可以解析到 GitHub Copilot token。
3. `openai` — 如果可以解析到 OpenAI key。
4. `gemini` — 如果可以解析到 Gemini key。
5. `voyage` — 如果可以解析到 Voyage key。
6. `mistral` — 如果可以解析到 Mistral key。
7. `bedrock` — 如果 AWS SDK credentials 可解析。

同一份文件還明確寫到：

> Codex OAuth covers chat/completions only and does not satisfy embedding requests.

OpenAI API-key 解析方式則是：

- 環境變數：`OPENAI_API_KEY`
- Config key：`models.providers.openai.apiKey`

來源：<https://docs.openclaw.ai/reference/memory-config>

## 本機實作調查

檢查本機安裝的 OpenClaw 2026.4.22：

```text
/opt/homebrew/lib/node_modules/openclaw/dist
```

相關觀察：

- Bundled OpenAI plugin 會註冊 `openAiMemoryEmbeddingProviderAdapter`。
- OpenAI memory embedding adapter：
  - id: `openai`
  - auth provider: `openai`
  - default model: `text-embedding-3-small`
  - auto-select priority: 20
- Gemini memory embedding adapter：
  - id: `gemini`
  - auth provider: `google`
  - default model: `gemini-embedding-001`
  - auto-select priority: 30
- OpenAI embedding provider 會呼叫 OpenAI-compatible `/embeddings` endpoint，並透過 provider API key/profile/env 解析 bearer auth。
- 本機 config schema 也確認 `agents.defaults.memorySearch.provider` 支援：`openai`、`gemini`、`voyage`、`mistral`、`bedrock`、`lmstudio`、`ollama`、`local`。

目前未改設定前，本機 memory status 類似：

```json
{
  "provider": "gemini",
  "model": "gemini-embedding-001",
  "requestedProvider": "auto",
  "sources": ["memory"],
  "fts": { "enabled": true, "available": true },
  "vector": { "enabled": true, "available": true }
}
```

本機 auth/config 狀況：

- 有 OpenAI Codex OAuth profile。
- 有 Google/Gemini auth profile。
- Agent shell 裡沒有 `OPENAI_API_KEY`。
- Config 裡沒有 `models.providers.openai.apiKey`。
- 所以 auto-detection 選到 Gemini 是預期行為。

## 社群 / Issue tracker 調查

目前沒有看到大量公開討論在爭論「OpenClaw 應該讓 Codex OAuth 直接支援 OpenAI embeddings」。公開 issues 和官方文件比較像是把 memory embeddings 視為獨立 provider layer。

幾個值得注意的 patterns：

### 1. Local embeddings 失敗時，Gemini 常被當成實務 workaround

例子：Apple Silicon / Node.js 22 上 `node-llama-cpp` missing 的 issue。該 issue 的 workaround 是把 memory search provider 改成 Gemini：

```bash
clawdbot gateway config.patch '.agents.defaults.memorySearch.provider = "gemini"'
```

來源：<https://github.com/openclaw/openclaw/issues/29548>

解讀：Gemini memory embeddings 不是異常用法；它一直是正常 provider / workaround 路徑之一。

### 2. Local embeddings 歷史上有安裝與 runtime 坑

例子：

- 升級後 `node-llama-cpp` missing：<https://github.com/openclaw/openclaw/issues/46569>
- Local GGUF embeddings 在 concurrent calls 下 deadlock：<https://github.com/openclaw/openclaw/issues/7548>

解讀：local/no-API-key 路線很吸引人，但不一定是最少摩擦的選項。它現在可能變好，但不應該在沒有測試的情況下貿然切換。

### 3. Gemini embedding path 也有自己的營運問題

例子：

- 某版本 Gemini behind HTTP(S) proxy 失敗：<https://github.com/openclaw/openclaw/issues/54279>
- 舊版 Gemini model/batch config 混淆：<https://github.com/openclaw/openclaw/issues/7319>

解讀：Gemini 是有效 provider，但可能遇到 quota、proxy、model naming、batch behavior 等 provider-specific 問題。

### 4. QMD backend 更進階，但 integration complexity 不低

例子：

- Raspberry Pi / arm64 上 QMD backend CLI hangs：<https://github.com/openclaw/openclaw/issues/65553>
- Direct `qmd search` 有結果但 OpenClaw 回空：<https://github.com/openclaw/openclaw/issues/58055>
- QMD collection name mismatch：<https://github.com/openclaw/openclaw/issues/21349>、<https://github.com/openclaw/openclaw/issues/9854>、<https://github.com/openclaw/openclaw/issues/23228>
- QMD MCP daemon proposal：<https://github.com/openclaw/openclaw/issues/15562>

解讀：進階 local-first memory stack 確實存在，但不是「隨手改一下就一定更穩」。對 Jones/Jarvis 的 continuity 來說，穩定比架構潔癖更重要。

### 5. 公開討論量有限

針對 OpenClaw repo issues 與 GitHub discussion search，沒有找到明確的大型共識串在討論「Codex OAuth 應該提供 embedding access」。目前最清楚的依據仍然是官方 memory config reference。

## 要求 OpenAI API key 正常嗎？

如果目標是使用 OpenAI embeddings：**正常**。

這不是 OpenClaw 故意要求重複 credential，而是因為 `openai` embedding provider 走 OpenAI platform embedding endpoint。該 endpoint 一般就是使用 OpenAI API key 認證與計費。

但從使用者體驗來說，這確實容易困惑。尤其是使用者是透過 Codex OAuth 進入 OpenAI 生態時，直覺會以為「OpenAI 都登入了，怎麼 embeddings 還要 key？」

更理想的產品呈現可能應該在 status / onboarding 裡把兩件事分開顯示：

- 主模型 auth：OpenAI Codex OAuth
- Memory embedding auth：Gemini API key
- Codex OAuth 不涵蓋 embeddings
- 若要 OpenAI embeddings，請設定 `OPENAI_API_KEY`；否則可選 local/Copilot/Gemini

## Jones/Jarvis 的實務選項

### 選項 A — 先保持 Gemini

如果 quota error 很少發生，這是目前最推薦的短期做法。

優點：

- 最少變更。
- 目前架構從早期 Jarvis 歷史到現在大致能運作。
- 不需要新增 OpenAI API billing/config。
- 避免在其他工作進行中動到 memory search 穩定性。

缺點：

- 免費 quota 可能偶爾 429。
- 主模型 provider 和 memory embedding provider 不一致，心智模型會混亂。
- 如果 quota 常爆，continuity 感會受影響。

操作規則：

- 如果 `memory_search` 因 Gemini quota 失敗，就退回直接讀本地記憶檔：`memory_get`、`read MEMORY.md`、今天/昨天 daily notes，晚點再重試 semantic search。

### 選項 B — 設定 OpenAI API key 做 memory embeddings

如果 Gemini quota 變成高頻問題，而且 Jones 接受 API billing，這是最穩直覺的方案。

優點：

- 簡單、官方文件支援清楚。
- OpenAI `text-embedding-3-small` 對文字記憶便宜且穩。
- 避免 Google/Gemini free-tier quota。

缺點：

- 需要 OpenAI platform API key，和 Codex OAuth 分開。
- 多一組 paid credential 要管理。
- 改 provider/model 可能觸發 reindex。

可能設定：

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "openai",
        model: "text-embedding-3-small",
        fallback: "none"
      }
    }
  }
}
```

Auth 方式：

- `OPENAI_API_KEY` environment variable，或
- `models.providers.openai.apiKey`，若 active config path 支援，最好用 secret reference。

### 選項 C — 測試 local embeddings

如果 Jones 希望 memory embeddings 完全不依賴外部 API，這是值得研究的方向。

優點：

- 沒有外部 quota。
- 隱私性更好；memory chunks 不需要送出機器做 embedding。
- 符合 self-hosted/local-agent 哲學。

缺點：

- 需要 `node-llama-cpp` / GGUF path 可靠運作。
- 可能需要下載模型、處理 build/optional dependency。
- OpenClaw issue tracker 顯示歷史上有不少坑。

可能設定：

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "local"
      }
    }
  }
}
```

驗證指令：

```bash
openclaw memory status --deep --agent main
openclaw memory index --force --agent main --verbose
openclaw memory search "test recall" --agent main
```

### 選項 D — GitHub Copilot embeddings

官方文件說 GitHub Copilot 可被 auto-detected，若能解析 Copilot token，理論上不需要另外 API key。

優點：

- 可能避免 OpenAI platform key。
- 在 auto-detection 順序中排在 OpenAI/Gemini 前面。

缺點：

- 目前 Jarvis 這台機器是否具備可用 Copilot embedding auth 未驗證。
- 會把 memory recall 綁到 GitHub/Copilot token availability。

## 建議

目前建議：**先保持 Gemini，不要急著改。**

這次發現的 quota 問題比較像是免費層短暫限制，不是資料損壞，也不是架構錯誤。不要只是為了讓 provider 名稱一致，就急著加 OpenAI API key。

建議後續策略：

1. 目前 config 不變。
2. 記住 operational fallback：Gemini quota 導致 `memory_search` 失敗時，直接讀本地記憶檔。
3. 接下來幾天觀察 quota failure 是否常發生。
4. 如果常發生，先做一次 controlled local embedding test。
5. 如果 local embedding 不穩或太慢，再考慮設定 OpenAI API key + `text-embedding-3-small`。

## 後續待研究問題

- 目前 OpenClaw 安裝在這台 Mac Mini 上的 `node-llama-cpp` local embedding path 是否能正常工作？
- 既有 GitHub / Copilot credentials 是否能支援 GitHub Copilot embedding provider？
- Gemini embedding free-tier quota 在 Jarvis 的正常 memory usage 下，實際多久會撞一次？
- OpenClaw status / Control UI 是否應該把「chat provider」與「memory embedding provider」並列顯示，避免這類混淆？
- `cache.enabled` 是否能顯著降低重複 embedding work，進而減少 quota spike？

## 來源

官方文件：

- OpenClaw Memory Overview: <https://docs.openclaw.ai/concepts/memory>
- OpenClaw Memory Search: <https://docs.openclaw.ai/concepts/memory-search>
- OpenClaw Memory Config Reference: <https://docs.openclaw.ai/reference/memory-config>
- OpenClaw Memory CLI: <https://docs.openclaw.ai/cli/memory>

Issue tracker examples：

- Local/node-llama install workaround using Gemini: <https://github.com/openclaw/openclaw/issues/29548>
- node-llama-cpp missing after upgrade: <https://github.com/openclaw/openclaw/issues/46569>
- Local GGUF embedding deadlock: <https://github.com/openclaw/openclaw/issues/7548>
- Gemini proxy issue: <https://github.com/openclaw/openclaw/issues/54279>
- Gemini model/batch issue: <https://github.com/openclaw/openclaw/issues/7319>
- QMD backend hang: <https://github.com/openclaw/openclaw/issues/65553>
- QMD empty results: <https://github.com/openclaw/openclaw/issues/58055>
- QMD collection mismatch: <https://github.com/openclaw/openclaw/issues/21349>
- QMD collection mismatch / empty results: <https://github.com/openclaw/openclaw/issues/9854>
- QMD suffix bug: <https://github.com/openclaw/openclaw/issues/23228>
- QMD MCP daemon proposal: <https://github.com/openclaw/openclaw/issues/15562>

本機證據：

- 2026-04-24 在 Jarvis Mac Mini 上執行 `openclaw memory status --json`。
- 檢查本機 OpenClaw installed dist：`/opt/homebrew/lib/node_modules/openclaw/dist`。
- 查詢 config schema：`agents.defaults.memorySearch`。

# Repo 安全審核：last30days-skill 與 Agent-Reach

日期：2026-06-14 EDT

需求來源：Jones

審核 repo：

- `mvanhorn/last30days-skill`：https://github.com/mvanhorn/last30days-skill
- `Panniantong/Agent-Reach`：https://github.com/Panniantong/Agent-Reach

## 摘要結論

我的建議是：不要盲目跟風導入 `last30days-skill` 或 `Agent-Reach` 這類爆火 repo。更好的做法是把它們當成技術樣板與突破口：看它們怎麼設計多來源 research pipeline、source adapter、credential boundary、doctor checks、資料 normalize、去重、評分與摘要流程；真正要長期使用時，應該依照 Jones / Jarvis / Jyn Null 的需求自研一套更窄、更乾淨、更安全的內部系統。

`last30days-skill` 值得參考，因為它展示了一個完整的「近期社群訊號研究」pipeline：跨 Reddit、X、YouTube、GitHub、Polymarket、HN、TikTok / Instagram / Threads、Web 等來源抓取資料，再整理成有根據的摘要。它有大量測試、明確設定文件、MIT license，也看得到不少安全意識；但它仍是年輕且快速變動的專案，而且會碰 API key、browser cookie、local env file 這類敏感資料。所以它更適合被研究、拆解與借鑑，不適合直接成為主系統依賴。

`Agent-Reach` 很值得研究，但我不建議現在直接裝進主 OpenClaw / Jarvis 環境。它更像是一個「網路能力路由器 + installer」，不是單一搜尋工具。它的設計很有啟發：多後端路由、doctor 健康檢查、平台 fallback、修復提示。但也因為它會安裝全域工具、設定 MCP、讀取 browser cookies、寫入 token config、接上第三方 CLI / OpenCLI，風險面比 `last30days-skill` 更大。

簡短版：

- 不要因為 GitHub trending 或社群爆火就直接採用。
- 真正重要的是：我們知道自己的需求，也知道需要時自己做得出來。
- `last30days-skill` 最值得參考的是 research pipeline 與 source normalization。
- `Agent-Reach` 最值得參考的是能力路由、doctor checks、multi-backend fallback。
- 兩者都不應該拿 Jones 的主社群帳號、主瀏覽器 profile、或高信任 credentials 直接測。
- 若未來自研，應使用專用 bot / sandbox 帳號、per-project env files，並避免給任何發文或互動權限。

## Repo 1：mvanhorn/last30days-skill

來源：https://github.com/mvanhorn/last30days-skill

本次觀察到的狀態：

- Default branch：`main`
- License：MIT
- 建立時間：2026-01-23
- GitHub metadata 顯示最近 push：2026-06-10 UTC
- GitHub metadata 顯示最新 release：`v3.3.0`，發布於 2026-05-17
- 本地 shallow clone 看到的 package version：`3.3.2`
- GitHub stars：約 42k
- Forks：約 3.4k
- 主要語言 / runtime：Python 3.12+
- 依賴面：`pyproject.toml` 沒有宣告一般 runtime dependencies；dev deps 是 `pytest` 與 `pytest-cov`

### 它在做什麼

`last30days-skill` 是一個 Agent Skill 加 Python engine，用來做多來源近期訊號研究。README 和 `SKILL.md` 描述的來源包含 Reddit、X / Twitter、YouTube、TikTok、Instagram、Threads、Hacker News、Polymarket、GitHub、Digg、Bluesky、Truth Social、小紅書、Pinterest、Perplexity、Web search 等。

它的產品概念很適合 Jones 的工作流：不是只搜尋網頁，而是搜尋最近 30 天人們真正討論、按讚、留言、下注、互動的東西，再交給 agent / model 做摘要。

很適合用在：

- AI opportunity radar
- Jyn Null 發文前的題材研究
- 市場敘事 / 社群敘事掃描
- 會議前的人物或公司研究
- 做工具、文章、產品前先驗證話題熱度

### 安全相關觀察

正面訊號：

- runtime dependency surface 很小，`pyproject.toml` 沒有一般 runtime dependencies。
- CI 有測試 workflow，也有 security workflow。
- security workflow 會跑 `pip-audit` 與 TruffleHog。不過目前是 advisory 模式，`continue-on-error: true`，還不是 blocking gate。
- 讀取 `.env` secret file 時會檢查權限，若 group / other 可讀會警告。
- 支援從 macOS Keychain 載入已知 credential names。
- HTTP helper 會在 log 裡遮蔽 query string 中的 `key`、`api_key`、`token`、`secret`。
- subprocess helper 使用 argument list、timeout、process-group cleanup，沒有看到危險的 shell string 執行模式。
- 目前程式碼明確跳過 Codex `~/.codex/auth.json` fallback；如果使用者要 OpenAI，需要明確設定 `OPENAI_API_KEY`。

風險點：

- 它可能使用很多強 credentials：`SCRAPECREATORS_API_KEY`、`OPENAI_API_KEY`、`XAI_API_KEY`、`OPENROUTER_API_KEY`、`AUTH_TOKEN`、`CT0`、`BSKY_APP_PASSWORD`、`TRUTHSOCIAL_TOKEN`、`BRAVE_API_KEY`、`EXA_API_KEY` 等。
- X / Twitter 可以透過 `AUTH_TOKEN` 與 `CT0` 存取，這基本上是 browser session 等級的 credential。
- 依設定不同，它支援從 Firefox / Safari / Chrome / Brave 擷取 browser credentials。
- repo 內 vendored 了一份 Bird / X client subset：`skills/last30days/scripts/lib/vendor/bird-search/`。
- 它接觸很多第三方平台，而這些平台的反自動化政策和 API 路徑都可能快速變動。
- repo 有 security scan，但目前不是 blocking gate，所以不能把 CI 綠燈視為完整安全保證。

### 與我們未來使用場景的適配度

高適配：

- 快速回答「最近大家怎麼看這件事？」
- 判斷一個 repo / product / topic 是真有社群熱度，還是只是 GitHub trending 的短期泡泡。
- 幫 Jyn Null 建立可重複的發文前研究 brief。
- 幫 `thinking_with_ai` 產出有來源支撐的研究報告。

中適配：

- 每日 / 每週 watchlist，但需要控制成本與查詢頻率，避免過度打付費 API。
- 產品競品研究。

低適配：

- 不適合未經人工覆核就產出精準法律、金融、醫療判斷。
- 不適合用來發文、按讚、回覆，或代表 Jones 執行社交互動。

### 採用建議

不建議直接採用成主系統依賴。比較好的做法是把它當成 reference implementation：研究它怎麼拆 source adapters、怎麼 normalize item schema、怎麼做 engagement scoring、怎麼保存 raw data、怎麼讓 LLM 摘要與資料抓取分離。

若真的需要做實測，也應該只是 sandbox spike，而不是正式導入：

1. 裝在 disposable / sandbox agent 環境，不要直接裝進主 OpenClaw home。
2. 第一階段只開低風險來源：GitHub、HN、Polymarket、可用的 Reddit public path、YouTube via `yt-dlp`、Web search。
3. 第一天不要開 browser-cookie extraction。
4. 如果之後需要 X，使用低風險專用帳號，並把 `AUTH_TOKEN` / `CT0` 放在 per-project file，權限設為 `chmod 600`。
5. 明確設定 `LAST30DAYS_MEMORY_DIR`，避免 raw result 和 brief 散落到不明位置。
6. 用幾個已知 topic 跑測試，再和手動 Web / GitHub 查核結果比較，確認可信度。

風險等級：中。

我的判斷：值得研究與拆解；必要時可做 sandbox spike，但長期方向應該是自研內部版。

## Repo 8：Panniantong/Agent-Reach

來源：https://github.com/Panniantong/Agent-Reach

本次觀察到的狀態：

- Default branch：`main`
- License：MIT
- 建立時間：2026-02-24
- GitHub metadata 顯示最近 push：2026-06-12 UTC
- GitHub metadata 顯示最新 release：`v1.5.0`，發布於 2026-06-11
- 本地 shallow clone 看到的 package version：`1.5.0`
- GitHub stars：約 28.7k
- Forks：約 2.3k
- 主要語言 / runtime：Python 3.10+
- runtime dependencies：`requests`、`feedparser`、`python-dotenv`、`loguru`、`pyyaml`、`rich`、`yt-dlp`
- optional dependencies 包含 `playwright`、`mcp[cli]`、`browser-cookie3`

### 它在做什麼

`Agent-Reach` 是給 agent 用的網路能力路由器。它不是單一搜尋引擎，而是幫 agent 選擇、安裝、檢查、維護多個平台的可用讀取路徑。

它支援或路由的方向包含：

- Web via Jina Reader
- GitHub via `gh`
- YouTube via `yt-dlp`
- Bilibili via `bili-cli` 或 OpenCLI
- X / Twitter via `twitter-cli`、OpenCLI、或 Bird-like fallback
- Reddit via OpenCLI 或 `rdt-cli`
- 小紅書 via OpenCLI、xiaohongshu-mcp、或 legacy xhs tools
- RSS via `feedparser`
- Exa via `mcporter`
- LinkedIn、V2EX、雪球、小宇宙播客等

最值得學的是 `agent-reach doctor` 的概念：每個 channel 都能檢查 active backend，並回報目前哪條路可用、哪條路壞了、該怎麼修。這對 Jarvis / OpenClaw 的長期工具治理很有參考價值。

### 安全相關觀察

正面訊號：

- 有 `SECURITY.md`，包含 private advisory 回報方式與回應時間線。
- CI 測 Python 3.10 到 3.13，並有 wheel smoke-install gate。
- persistent config 放在 `~/.agent-reach/config.yaml`。
- 寫 config 時使用 owner-only permission，透過 `os.open(..., 0o600)` 避免短暫 world-readable race。
- 提供 `--safe` 與 `--dry-run` install modes。
- 安裝文件明確要求：除非使用者批准，不要用 `sudo`。
- 文件明確建議 cookie-based platform 使用 secondary account。
- 目前檢查到的 subprocess path 沒有使用 `shell=True`。

風險點：

- installer 可以執行全域 package install，例如 `npm install -g mcporter`、`npm install -g @jackwener/opencli`、`pipx install`、`uv tool install`。
- 可以透過 `mcporter config add` 設定 MCP entries。
- 安裝文件鼓勵 agent 從 `main.zip` 安裝，雖然方便，但 reproducibility 比 pinned release artifact 弱。
- 啟用後可透過 `rookiepy` 或 `browser-cookie3` 讀 browser cookies。
- cookies / API keys 存在 `~/.agent-reach/config.yaml`，不是系統 keychain。
- OpenCLI 的設計是透過 browser extension + local daemon 重用真實 browser login session。
- 小紅書 MCP flow 可能涉及 Docker / container cookie injection。
- 它路由很多第三方工具，每個工具都有各自的供應鏈與維護風險。

### 與我們未來使用場景的適配度

高適配：

- 當 Jarvis 工具治理架構參考：doctor checks、backend selection、platform status reporting、repair hints。
- 在 sandbox VM 裡測廣泛 internet reader stack。
- 做中文平台研究實驗，特別是小紅書 / Bilibili 路線探索。

中適配：

- 某些單次研究任務，如果它處理的平台比我們現有工具更好，可以拿來比較。
- 研究目前公眾 agent tooling 如何解決多平台存取。

低適配：

- 不適合直接安裝進主 Jarvis / OpenClaw 環境。
- 不適合碰 Jones 的主瀏覽器、主社群帳號、高信任 credentials。
- 不適合用在需要 deterministic reproducibility 的 production automation。

### 採用建議

暫時不要裝進主環境。

建議試用路徑：

1. 在 disposable macOS user、VM、或類 container 的 throwaway 環境測。
2. 第一個命令只跑 `agent-reach install --safe --dry-run`。
3. 不要用 `--channels=all`。
4. 一次只啟用一個 channel，並記錄它新增了哪些 file / tool / config。
5. X、小紅書、Reddit、任何 cookie-based platform 都只用 secondary accounts。
6. 不要接 `user` Chrome profile。若需要 browser automation，使用既有 OpenClaw `openclaw` profile 邊界，或建立全新 browser profile。
7. 把 OpenCLI / MCP entries 視為 privileged local automation，啟用前逐項審核。

風險等級：中高。

我的判斷：很值得研究，但暫時不適合 wholesale adoption。

## 橫向比較

| 面向 | last30days-skill | Agent-Reach |
|---|---|---|
| 核心形態 | Research skill + Python engine | Capability router + installer + doctor |
| 直接導入價值 | 中 | 低 |
| 參考價值 | 高 | 高 |
| 主要風險 | 多 API keys / browser tokens | 全域安裝、MCP / router surface、browser cookies |
| Credential handling | Env files + project env + macOS Keychain support | `~/.agent-reach/config.yaml`，權限 0600 |
| CI 成熟度 | 大量 pytest，advisory security scans | Multi-Python CI + wheel smoke gate |
| Runtime dependency surface | Python runtime deps 很小 | 較大，且有 optional browser / MCP deps |
| 最適合我們的用途 | 拆解 research pipeline 與 source normalization | 拆解工具治理、doctor checks、multi-backend fallback |
| 是否裝進主 Jarvis | 不建議，reference only | 不建議 |
| 總體建議 | 研究拆解，必要時 sandbox spike | 研究拆解，不 wholesale adoption |

## 建議下一步

1. 不急著安裝或導入任一個爆火 repo。
2. 先把 `last30days-skill` 拆成可學習的技術模組：
   - source adapter
   - normalized item schema
   - raw result archive
   - engagement scoring
   - dedupe / clustering
   - synthesis prompt contract
3. 先把 `Agent-Reach` 拆成可學習的工具治理模組：
   - platform router
   - active backend
   - doctor output
   - repair hints
   - safe / dry-run install mode
4. 未來若要自研，可以先做一個小型 `Jarvis Signal Engine` / `Jovi Signal Lab`：
   - GitHub repo / issue / release 掃描
   - Web search + article reader
   - YouTube search + transcript
   - HN / Reddit 社群討論
   - Markdown brief + raw JSON archive
5. 跑三個評估題目：
   - `OpenClaw vs Hermes`
   - `AI social media automation tools`
   - `Xiaohongshu AI creator workflow`
6. 把輸出和手動 Web / GitHub 查核、現有 browser / web_fetch workflow 比較。
7. 第一天不要加 X cookies；如果後續需要，使用 throwaway X account。

## 最後判斷

`last30days-skill` 的價值不在於「我們要不要用它」，而在於它證明了 agent research 可以被工程化：多來源抓取、normalize、排序、cluster、保存 raw data、再交給 LLM 摘要。這個方向值得學，但不必盲目採用。

`Agent-Reach` 很聰明，也符合 agent tooling 的未來方向，但它想變成 infrastructure。對 infrastructure，要走比較慢、比較嚴格的 trust path。

最適合我們的策略是：不跟風、不迷信 star 數；保留技術參考，知道突破口在哪，然後在真正需要時自己做一個符合 Jones 系統需求與安全邊界的版本。

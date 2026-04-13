# Anthropic Managed Agents — 服務調查報告

> **最後更新：** 2026-04-11
> **發布日期：** 2026-04-08（public beta）
> **由 Jarvis（Dispatch 研究）產出**

---

## TL;DR

**Managed Agents 是 Anthropic 新推出的「agent 代管服務」**——你定義 agent 的任務、工具、護欄；Anthropic 處理其他所有事情：orchestration、sandbox 執行環境、error recovery、credential 管理、auto-scaling、session 持久化。

一句話定義：

> **「你寫 agent 邏輯，我幫你跑 agent 基礎設施。」**

定價：**標準 API token 費 + $0.08 / session-hour**（很便宜——一個 agent 跑一整天是 $1.92）。

Public beta 2026-04-08 開放，所有 Anthropic API 帳號都能用。Notion、Asana、Rakuten 已在生產環境使用。

---

## 一、為什麼這個服務值得關注

### 問題背景

建一個「能真正在生產環境跑」的 AI agent 有很多工程黑洞：

- **Sandbox 安全：** 你不會想讓一個 LLM 在你主機上直接跑 bash
- **Credential 管理：** agent 需要存取 API key、OAuth token，但不能讓 LLM 直接看到
- **Session 持久化：** 長時間任務（跑幾小時甚至幾天）中途斷了要能接回來
- **Error recovery：** agent 呼叫的外部 API timeout 了、rate limit 了，要自動重試
- **Scaling：** 100 個用戶同時觸發 agent，你的 infra 撐不撐得住
- **監控 / 可觀測性：** agent 做了什麼、花了多少 token、哪裡出錯

Anthropic 的說法是：**以上這些東西，企業工程團隊通常花 3-6 個月才能搭好**，還沒開始寫一行 agent 邏輯。Managed Agents 把這些全包了。

### 對手對照

| 面向 | 自己建（Claude API + 自架 infra） | Anthropic Managed Agents | OpenAI Agents API | LangGraph Cloud |
|---|---|---|---|---|
| Agent 定義 | 你全寫 | 你寫邏輯，infra 它管 | 你寫邏輯，infra 它管 | 你畫 graph + 寫邏輯 |
| Sandbox | 自己弄 Docker / Firecracker | **Anthropic 提供，untrusted by design** | OpenAI Code Interpreter | 自己搞 |
| Credential 管理 | 自己建 vault | **內建 vault + OAuth** | 有限支援 | 自己建 |
| Error recovery | 自己寫 | **內建，斷電可接回** | 有限 | 自己寫 |
| 模型選擇 | Claude 全系列 | Claude 全系列 | 只能 OpenAI | 任何模型 |
| 定價 | Token 費 | Token + $0.08/hr | Token + 有 infra 費 | Token + LangSmith 費 |
| Vendor lock-in | 低（自己控制） | **高（Anthropic only）** | 高（OpenAI only） | 低（model-agnostic） |
| 上手時間 | 3-6 個月 | **~30 分鐘** | ~1 小時 | ~1 天 |

---

## 二、核心架構：Brain / Hands / Session

Anthropic 的工程部落格詳細描述了這套架構的設計哲學：**把 agent 虛擬化成三個解耦的組件**。

### 🧠 Brain（大腦）

- 就是 Claude 本體 + orchestration harness
- 負責：看 prompt → 決定要呼叫什麼 tool → 讀結果 → 決定下一步
- Brain 不直接執行任何東西——它只「想」和「指揮」

### 🤲 Hands（手）

- 一個 sandboxed、ephemeral 的執行環境
- 可以跑 bash、Python REPL、操作檔案、呼叫 MCP 工具
- **Sandbox 被視為 untrusted** — 即使 agent 被 jailbreak、sandbox 裡也拿不到真正的 credential
- Credential 怎麼辦？有個 **credential vault**，agent 透過 proxy 存取外部服務，vault 代為注入 token，sandbox 本身看不到 token string

### 📋 Session（記憶 / 日誌）

- 一個 append-only 的 event log，紀錄 agent 做的每一件事
- 活在 Claude 的 context window **之外**（不會佔 token）
- Brain 可以透過 `getEvents()` 查詢特定片段
- **Error recovery 的核心：** 如果 agent 中途掛掉，新的 session 可以接上舊的 event log 繼續跑

### 架構圖（簡化版）

```
使用者 / 你的系統
    ↓ API call（定義任務 + 工具 + 護欄）

┌─────────────────────────────────────────┐
│  Anthropic Managed Agents Infrastructure │
│                                          │
│  ┌──────────┐    ┌──────────┐           │
│  │ 🧠 Brain │◄──►│ 📋 Session│           │
│  │ (Claude)  │    │ (Event   │           │
│  │ + Harness │    │  Log)    │           │
│  └────┬─────┘    └──────────┘           │
│       │ tool calls                       │
│       ▼                                  │
│  ┌──────────┐    ┌──────────┐           │
│  │ 🤲 Hands │◄──►│ 🔐 Vault │           │
│  │ (Sandbox) │    │ (Creds)  │           │
│  │ bash/py/  │    │ OAuth    │           │
│  │ MCP tools │    │ tokens   │           │
│  └──────────┘    └──────────┘           │
│                                          │
└─────────────────────────────────────────┘
    ↓ 結果回傳
使用者 / 你的系統
```

**為什麼這個設計很強：**

1. **Brain 從不觸碰 credential** — 即使模型嘗試讀 vault，也做不到（結構性安全，不依賴 model alignment）
2. **Sandbox 可以被銷毀重建** — 每個 session 的 sandbox 是 ephemeral 的，做完就丟
3. **Session 可以跨 sandbox 傳遞** — 舊 sandbox 死了，新的 sandbox 可以繼續
4. **Git 操作的 clever trick：** access token 只在 clone 時注入到 git remote URL 裡，agent 在 sandbox 裡 `git push/pull` 時完全透明運作，不知道 token 長什麼樣

---

## 三、主要功能

### 已上線（public beta）

| 功能 | 說明 |
|---|---|
| **Managed hosting** | Anthropic 幫你跑，不用自己搞 server |
| **Auto-scaling** | 多用戶同時觸發自動擴展 |
| **Built-in monitoring** | OpenTelemetry 支援，可接你的 observability stack |
| **Error recovery** | 中途斷線可接回（靠 session event log） |
| **Credential vault** | OAuth token、API key 安全存放 |
| **MCP 原生支援** | 接任何 MCP 相容工具 |
| **RBAC** | Role-based access control |
| **Group spend limits** | 團隊預算控管 |
| **Usage analytics** | 用量分析 dashboard |
| **Zoom MCP connector** | 內建 Zoom 整合 |

### Research Preview（還在實驗）

| 功能 | 說明 |
|---|---|
| **Sub-agent 協調** | Agent 可以自己開 sub-agent 來平行處理複雜任務 |
| **自動 prompt 品質優化** | 自動迭代改善 prompt 的回應品質 |

---

## 四、定價

**極其簡單：**

| 費用 | 金額 |
|---|---|
| **Claude API token 費** | 照標準價（Opus / Sonnet / Haiku 各自計費） |
| **Session-hour 費** | **$0.08 / 小時**（agent 在跑的時候才計費） |

### 範例計算

- **一個 agent 跑 30 分鐘：** token 費 + $0.04
- **一個 agent 跑 24 小時：** token 費 + $1.92
- **100 個 agent 同時跑 1 小時：** token 費 + $8.00

**Session-hour 費非常便宜。** 真正的成本在 token 費——如果你的 agent 用 Opus 4.6 做大量推理，token 才是大頭。

### 跟自架比

自己搞 sandbox + credential vault + monitoring + auto-scaling 的雲端費用，通常遠超 $0.08/hr。Anthropic 這個定價基本上是在說：**不要自己建基礎設施了，用我的。**

---

## 五、早期客戶

| 公司 | 用途 |
|---|---|
| **Notion** | 已整合到 Notion AI 產品 |
| **Asana** | 任務自動化 |
| **Rakuten** | 日本電商客服 / 商品管理 |

Anthropic 聲稱多家客戶已經在生產環境使用，不只是 pilot。

---

## 六、跟 Jones / Jarvis 架構的潛在關聯

### 現在的 Jarvis 架構

Jones 目前的 Jarvis 跑在 Mac Mini M4 + Claude Desktop + Dispatch（Cowork mode）。Code task 在 Mac host 上執行，需要本地 shell、git、filesystem 存取。

### Managed Agents 能改變什麼？

| 現在的痛點 | Managed Agents 能幫嗎？ |
|---|---|
| **Code task 卡 allow prompt** | ✅ Managed agent 不需要桌面 allow——它跑在雲端 |
| **Dispatch sandbox 不能碰 GitHub** | ✅ Managed agent 有 credential vault，可以直接操作 git repo |
| **Sleep 時 agent 能不能繼續跑** | ✅ Session 持久化 + error recovery，你睡覺它繼續 |
| **多 agent 平行任務** | ⚠️ Sub-agent 還是 research preview |
| **本地 GPU 推論（Gemma 4）** | ❌ Managed Agents 只用 Claude，不能接本地模型 |
| **零成本** | ❌ 不行——有 token 費 + session-hour 費 |

### 適合 Jones 的場景

- **24/7 cron 任務**（如果 Jones 不想依賴 Mac Mini 24/7 開機）——Managed Agent 在雲端跑，不需要本地硬體
- **masterplan-jarvis 的 context 汲取**——可以設一個 Managed Agent 定期 pull repo、讀新 context、更新 Dossier、push
- **media-index 的 YouTube 掃描**——定時觸發 agent 抓新影片 → Whisper → 抽取 → 存 JSON（但 Whisper 要跑在 sandbox 裡，不確定支援多好）

### 不適合的場景

- **需要本地 GUI 操控**（computer-use）——Managed Agents 沒有桌面
- **需要本地 Gemma 4 推論**——只能用 Claude
- **教學 / vibe coding**——學生需要的是互動式 IDE，不是雲端 agent

### 結論

**現階段 Managed Agents 對 Jones 來說是「知道就好」的程度，不需要立刻切換。** 原因：

1. Jones 現在的 Dispatch + Code Task 架構**免費**（走 Claude Max 訂閱）；Managed Agents 有額外費用
2. Jones 很多任務需要 Mac Mini 本地操控（computer-use、git 本地操作、Gemma 4）
3. Managed Agents 的 sub-agent 功能還在 research preview

**但未來如果 Jones 想把某些「不需要本地硬體」的定期任務雲端化（例如每天自動更新 Dossier），Managed Agents 是一個非常自然的升級路徑。**

---

## 七、技術深潛參考（給想深入了解的人）

### Anthropic 官方

- [Scaling Managed Agents: Decoupling the Brain from the Hands](https://www.anthropic.com/engineering/managed-agents) — Anthropic 工程部落格，架構設計原文
- [Code Execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) — MCP 整合的技術細節

### 媒體報導

- [SiliconANGLE: Anthropic launches Claude Managed Agents](https://siliconangle.com/2026/04/08/anthropic-launches-claude-managed-agents-speed-ai-agent-development/)
- [The New Stack: Anthropic wants to run your AI agents for you](https://thenewstack.io/with-claude-managed-agents-anthropic-wants-to-run-your-ai-agents-for-you/)
- [The Decoder: Anthropic launches managed infrastructure for autonomous AI agents](https://the-decoder.com/anthropic-launches-managed-infrastructure-for-autonomous-ai-agents/)
- [The Register: Anthropic will let your agents sleep on its couch](https://www.theregister.com/2026/04/09/anthropic_offers_to_host_ai/)
- [CNBC: AI threat's relentless flogging of software stocks](https://www.cnbc.com/2026/04/09/anthropic-new-ai-agent-software-stocks-selloff.html) — Managed Agents 發布後的股市影響

### 深度分析

- [Complete Guide to Building with Claude Managed Agents](https://atalupadhyay.wordpress.com/2026/04/11/from-building-to-deploying-the-complete-guide-to-anthropics-claude-managed-agents/)
- [Architectural Genius of Managed Agents (Epsilla Blog)](https://www.epsilla.com/blogs/anthropic-managed-agents-decoupling-brain-hands-enterprise-orchestration)
- [DEV.to Deep Dive](https://dev.to/bean_bean/claude-managed-agents-deep-dive-anthropics-new-ai-agent-infrastructure-2026-3286)
- [MindStudio: What Is Managed Agents](https://www.mindstudio.ai/blog/what-is-anthropic-managed-agents)

### 比較分析

- [AI Agent Frameworks Comparison 2026 (Fungies.io)](https://fungies.io/ai-agent-frameworks-comparison-2026-langchain-crewai-autogen/)
- [AI Agent Frameworks: State of the Art (Medium)](https://medium.com/@roberto.g.infante/the-state-of-ai-agent-frameworks-comparing-langgraph-openai-agent-sdk-google-adk-and-aws-d3e52a497720)

---

## 八、修訂紀錄

- **2026-04-11 v1** — 初版。由 Jarvis（Dispatch 研究）產出。涵蓋架構、功能、定價、早期客戶、競品比較、Jones 適用性分析

---

*本報告是 living document，Anthropic Managed Agents 仍在 public beta，功能和定價可能隨時變動。*

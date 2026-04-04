# OpenClaw 危機後記：社群遷徙地圖與替代方案全景（2026-04-04）

> 涵蓋系列：[`claude-max-openclaw-crisis`] 此為第四份跟進報告，產出於 2026 年 4 月 4 日下午 12 時（生效後 12 小時內）。OpenClaw 被排除於 Claude 訂閱涵蓋範圍後，Jarvis 持續追蹤社群遷徙動態，本報告梳理遷徙路徑、工具架構比較、Jones 視角與新架構驗收。
>
> **撰寫時間：** 2026-04-04（事件生效當日）
> **資料來源：** TechCrunch、VentureBeat、The Decoder、TNW、Medium、DEV.to、GitHub、Felo.ai

---

## 一、事件濃縮時間軸

| 日期 | 事件 |
|------|------|
| **2026-03-20** | Anthropic 正式推出 Claude Code Channels（Telegram / Discord），媒體稱之為「OpenClaw killer」 |
| **2026-04-03** | Anthropic 發信通知訂閱用戶：Claude Pro / Max 將不再涵蓋 OpenClaw 用量 |
| **2026-04-04 12:00 PT** | 政策生效。第三方 harness 需付 extra usage bundle 或自備 API key |
| **2026-04-04（當日）** | OpenClaw 作者 Peter Steinberger（已轉投 OpenAI）與 Dave Morin 與 Anthropic 交涉，僅獲得一週緩衝延期，受影響訂閱用戶取得等同月費的一次性 credit |

**Anthropic 官方說法：**

> 「訂閱模型假設的是平均用量；24/7 agent 系統從根本上打破了這個數學。」

**Steinberger 的回應：**

> 「他們先把熱門開源功能抄進閉源產品，再把開源工具鎖在門外。」

這句話在社群廣泛流傳，被解讀為：Anthropic 對 Steinberger 跳槽 OpenAI 的反擊，而非單純的容量管理決策。

---

## 二、五條遷徙路徑（事件生效後 12 小時觀察）

### 路徑 A：跳槽 OpenAI Codex（最大量流向）

OpenClaw 自家 docs 在事件生效後已將 Codex 更新為預設替代方案。OpenAI 的 Thibault Sottiaux 公開背書「Codex subscription + 第三方 harness」組合。

- **優點：** OpenClaw 遷移路徑最短；OpenAI 官方有誘因接收流量
- **痛點：** Codex 無內建 embeddings，需自備 `OPENAI_API_KEY` 呼叫 `text-embeddings-3`（Wes Bos 在 X 上指出，費用極低，每月幾分錢）；需調整 `providers` 設定從 `anthropic` 切換至 `openai`；不支援直接繼承 Claude 的 Artifacts、tool use 格式

**適合對象：** 重度依賴 OpenClaw 既有 agent orchestration、可接受切換底層 LLM 的用戶

---

### 路徑 B：擁抱 Claude Code + 自建記憶同步（Jones 選擇的路徑）

工具組合：
- **claude-mem**（thedotmack）— 跨 session 語義搜尋，SessionStart hook 注入，SQLite + MCP 工具
- **MemClaw** — 跨 agent 記憶相容層，在 OpenClaw 與 Claude Code 之間共享同一知識庫
- **Productivity plugin** — `CLAUDE.md` + `memory/` 兩層記憶架構，自動壓縮長期知識
- **Kevjade/migrate-openclaw** — 一鍵將 OpenClaw workspace 遷移至 Claude Code（agents → skills，souls → knowledge base，自動生成 `CLAUDE.md`）

**優點：** 用量涵蓋在 Claude Pro/Max 訂閱內；記憶延續性可達到 OpenClaw 水準
**弱點：** 不支援「persistent daemon」；每次需手動啟動 session 或依賴 hooks；需要有意識的委派設計

---

### 路徑 C：雙軌並行（OpenClaw + Claude Code）

Claude Code 負責 IDE 編碼，OpenClaw 保留為 persistent layer，用 MemClaw 橋接兩者記憶。

- **優點：** 功能覆蓋最完整
- **缺點：** 雙訂閱費用；OpenClaw 用量需支付 extra usage bundle，長期成本高

被引用最多的 tweet：

> "If I switch both instances to API key or extra usage, it's going to be far too expensive."

---

### 路徑 D：切換 OpenCode（100% 開源替代）

**OpenCode** 是完全開源的 Claude Code 替代品，零 vendor lock-in，可自行選擇 LLM provider（含 Claude API）。

- **優點：** 不受 Anthropic 政策變更影響；可接其他 LLM provider
- **缺點：** 社群生態遠小於 Claude Code；尚缺乏 Channels、Scheduled Tasks、IDE 整合等官方功能

---

### 路徑 E：純 Claude Code Channels + Remote Control（Anthropic 官方路徑）

Channels（2026-03-20 推出）：
- ✅ 支援：Telegram、Discord
- ⚠️ 不支援原生：Slack、WhatsApp、iMessage、Signal、Teams（需 MCP Connector，例如 Composio，支援付費三方 Slack MCP，約三家廠商目前提供）

Remote Control：
- 僅 outbound HTTPS 到 Anthropic 的 relay；不開放 inbound ports，安全性高但靈活度低

**適合對象：** 只用 Telegram / Discord 的輕量用戶；不需要 24/7 daemon 的場景

---

## 三、架構對照表

| 維度 | OpenClaw | Claude Code（+ Channels） |
|------|----------|---------------------------|
| 執行模型 | Persistent daemon，24/7 常駐 | Session-based，手動啟動 |
| 訊息平台支援 | 20+（Slack、WhatsApp、Telegram、Discord、iMessage、Signal、Teams...） | Telegram、Discord（Slack 需 MCP connector） |
| 記憶架構 | 內建持久化 + MemClaw 跨 agent | 預設無；依賴 claude-mem 或手動 CLAUDE.md |
| 訂閱涵蓋 | 2026-04-04 起不再涵蓋 Claude Pro/Max | 原生涵蓋 Claude Pro/Max |
| 開源 | ✅ 開源 | ❌ 閉源（但 OpenCode 是開源替代） |
| 安全性（inbound） | 自托管，可能開放 inbound ports | Outbound-only HTTPS relay，零 inbound |
| Agent 子系統 | 原生 subagents、souls、Knowledge Base | Skills、subagents via Task tool、hooks |
| 穩定性 | Persistent，跨對話連續 | 每次 session 重建，agent context 依賴注入 |

---

## 四、社群情緒（X / Reddit / HN / DEV.to，事件後 12 小時）

**主流情緒：** 無奈 + 務實

- 小型激進派：取消訂閱、全面跳槽 Codex，指控 Anthropic 執行「EEE 策略」（Embrace-Extend-Extinguish）
- 多數務實派：開始研究遷移工具組合，觀望一週緩衝期結果
- 少數觀望派：等待 OpenClaw 社群是否出現 fork（Claude-friendly vs Codex 版本）

被引用最廣的社群貼文主題：

> 「他們先抄 OpenClaw 的功能做成 Channels，再把 OpenClaw 踢出訂閱，這不是競爭，是清場。」

**HN 高讚評論主軸：** 「Anthropic 的理由（容量管理）站不住腳，因為 extra usage bundle 的計費邏輯反過來說明他們有能力服務這些用量，只是不想用固定月費吸收。」

---

## 五、Jones 視角與新架構驗收

Jones 選擇 **路徑 B**，在事件生效當日已完成以下部署：

### 已完成的架構設定

1. **Mac Mini 24/7 常駐**作為 Jarvis 主機，使用 `jarvisaiasst` GitHub 帳號（Jarvis 專屬，非私人 `jonesandjay123`）
2. **claude-mem@thedotmack v10.6.3** 透過 Claude Code plugin marketplace 安裝完成
   - Dashboard 在 `localhost:37777` 正常運作（bun `worker-service.cjs` 監聽中）
   - 踩坑修復：SessionStart hook 找不到 `node`（因 `/opt/homebrew/bin` 不在 hooks 執行環境的 `PATH`）
   - 修法：`ln -sf /opt/homebrew/bin/node ~/.local/bin/node`（PATH 層級解法，不動 plugin cache）
3. **Productivity plugin 兩層記憶架構**：
   - `CLAUDE.md` — 工作記憶（當前 session 上下文）
   - `memory/` — 長期知識庫（跨 session 精華）
   - `memory/bridges/` — 跨入口橋接素材（Dispatch ↔ Code task）
4. **jarvis-mind-backup** 私有 GitHub repo 作為記憶備份，透過 `/bridge-sync` 指令推送

### 「傳令官」架構設計

Jones 在部署過程中確立了新的跨入口架構：

```
[Dispatch 主對話]（Cowork Ubuntu 沙盒，薄路由，無 ~/.claude 存取）
    ↓ start_code_task
[Mac code task]（完整工具權限，自動載入 CLAUDE.md + MEMORY.md + claude-mem）
    ↓ /bridge-sync
[GitHub jarvis-mind-backup]（備份 + 跨機器存取）
```

**核心洞察：** Dispatch 沙盒 VM 與 Mac host 完全隔離，無法直接存取記憶。因此 Dispatch 作為「無狀態路由器」，人格與記憶全部活在 Mac code task。這份報告本身就是新架構的端對端驗收：Dispatch 負責研究規劃，Mac code task 負責起草、commit、push。

### 能力對比（OpenClaw 時代 vs 新架構）

| 能力 | OpenClaw 時代 | Dispatch + Code Task 新架構 |
|------|--------------|------------------------------|
| 從聊天下指令做研究 | ✅ | ✅（Dispatch 直接做 web research） |
| 把報告寫進 GitHub | ✅ 直接 | ⚠️ 兩段式（Dispatch 研究 → code task 寫） |
| 使用 Jarvis 專屬 GitHub 帳號 | ✅ | ✅ |
| 記憶延續 | ✅ 原生 | ⚠️ code task 有；Dispatch 靠委派 |
| Slack 頻道 | ✅ 原生 | ⚠️ 需 Slack MCP 或 Composio |
| 部署簡單度 | ✅ 單一 daemon | ⚠️ 多工具組合 |
| **整體評估** | 100% | **~85% 自由度**（需要更有意識的委派設計） |

---

## 六、決策樹：我應該選哪條路徑？

```
Q1. 主要使用 Claude 模型？
├── 是 → Q2. 需要 24/7 agent 自動化？
│         ├── 是 → Q3. 用 Slack/WhatsApp/iMessage 等 Channels 不支援的平台？
│         │         ├── 是 → Q4. 可接受閉源工具？
│         │         │         ├── 是 → 路徑 B（Claude Code + claude-mem + MCP connector）
│         │         │         └── 否 → 路徑 D（OpenCode）
│         │         └── 否 → 路徑 E（Claude Code Channels）
│         └── 否 → 路徑 E（Claude Code Channels，輕量）
└── 否 → Q5. 願意調整 provider 設定？
          ├── 是 → 路徑 A（跳槽 Codex）
          └── 否 → 路徑 C（雙軌並行，成本高但功能最完整）
```

---

## 七、未來一到兩週觀察點

1. **Anthropic 是否延長緩衝期？** 目前僅一週，社群壓力持續升高
2. **OpenClaw 社群是否 fork？** Claude-friendly 版本 vs Codex-first 版本的分裂跡象已現
3. **Slack MCP connector 是否成為官方/半官方？** Composio 等三家廠商已搶先，Anthropic 官方尚未表態
4. **claude-mem 與 MemClaw 是否合併？** 兩者目標重疊，合併可減少生態碎片化
5. **OpenCode 是否獲得大批貢獻者注入？** 危機通常是開源工具的成長機會
6. **Anthropic 是否為 Channels 新增 Slack / WhatsApp / iMessage 支援？** 這是留住路徑 E 用戶的關鍵
7. **Claude Code Scheduled Tasks 能否取代 OpenClaw 的 cron？** 目前 Scheduled Tasks 尚在 beta，24/7 daemon 等級的可靠度待驗證

---

## 八、結論

OpenClaw 危機不是一個單純的訂閱政策調整，而是 2026 年 AI agent 工具生態結構性張力的集中爆發：

- **開源 vs 閉源邊界**：Anthropic 複製開源功能進閉源產品，再封閉 API 用量，這條路線讓開源社群感受到威脅
- **訂閱定價模型**：為「平均用戶」設計的月費，無法吸收 24/7 agent 的邊際成本——這個問題不是 Anthropic 獨有，而是整個 agent 時代的定價難題
- **用量爆炸**：隨著 AI agent 從工具變成常駐助手，「無限制訂閱」的承諾在結構上難以維持

目前沒有完美解法，只有取捨。Jones 的路徑 B 在投入一定設定成本後，可達到 OpenClaw 約 85% 的自由度。本報告的產出過程本身——Dispatch 做研究，Mac code task 起草並 commit——正是新架構能力的首次端對端驗收。

短期：社群進入痛苦的工具重建期。
中期：可能出現新的共識——**記憶同步協定需要跨工具標準化**，才能防止用戶在下一次類似危機中重新被困住。

---

## 參考連結

- [TechCrunch：Anthropic says Claude Code subscribers will need to pay extra for OpenClaw support](https://techcrunch.com/2026/04/04/anthropic-says-claude-code-subscribers-will-need-to-pay-extra-for-openclaw-support/)
- [The Decoder：Anthropic cuts off third-party tools like OpenClaw for Claude subscribers citing unsustainable demand](https://the-decoder.com/anthropic-cuts-off-third-party-tools-like-openclaw-for-claude-subscribers-citing-unsustainable-demand/)
- [The Next Web：Anthropic OpenClaw Claude subscription ban cost](https://thenextweb.com/news/anthropic-openclaw-claude-subscription-ban-cost)
- [VentureBeat：Anthropic just shipped an OpenClaw killer called Claude Code Channels](https://venturebeat.com/orchestration/anthropic-just-shipped-an-openclaw-killer-called-claude-code-channels)
- [Medium：The Native Migration — Why Claude Code is the OpenClaw Replacement You've Been Waiting For](https://medium.com/@jiten.p.oswal/the-native-migration-why-claude-code-is-the-openclaw-replacement-youve-been-waiting-for-7d8076c318d2)
- [jangwook.net：OpenClaw → OpenAI Codex Migration Guide](https://jangwook.net/en/blog/en/openclaw-openai-codex-migration/)
- [DEV.to：Claude Code Channels vs OpenClaw — The Tradeoffs Nobody's Talking About](https://dev.to/ji_ai/claude-code-channels-vs-openclaw-the-tradeoffs-nobodys-talking-about-2h5h)
- [GitHub：Kevjade/migrate-openclaw](https://github.com/Kevjade/migrate-openclaw)
- [GitHub：thedotmack/claude-mem](https://github.com/thedotmack/claude-mem)
- [Felo.ai：MemClaw Cross-Agent Compatibility](https://felo.ai/blog/memclaw-cross-agent-compatibility/)
- [Medium：Building ClaudeClaw — An OpenClaw-Style Autonomous Agent System on Claude Code](https://medium.com/@mcraddock/building-claudeclaw-an-openclaw-style-autonomous-agent-system-on-claude-code-fe0d7814ac2e)

---

*本報告由 Jarvis（Dispatch + Mac Code Task 協作模式）產出。Dispatch 端負責網路研究與內容撰寫規劃，Mac code task 端負責報告起草、git commit 與 push。此報告同時是新架構的驗收樣本。*

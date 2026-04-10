# AI 編碼平台額度與模型橫向比較

> **最後更新：** 2026-04-07
> **維護狀態：** 🟢 Living document — 政策與額度變動頻繁，請回頭檢查更新日期
> **用途：** 給教授 AI vibe coding 的學習者選擇平台、規劃流量、比較工具時的參考

---

## 為什麼需要這份文件

AI 編碼工具的免費／付費制度在 2025-2026 變動極度頻繁——Antigravity 在 4 個月內被砍 4 次配額、Cursor 2025-06 改成 credit-based、Claude 2025-08 加週配額、Codex 2026-04-02 改 token-based 計價。**任何一份「寫死的攻略」都會在幾週內過時。**

這份文件的目的不是給一個「標準答案」，而是：

1. 列出**當前主要五個平台**的最新狀況（2026-04 快照）
2. 說明**額度如何計算**（訊息？請求？5 小時？週？Token？）
3. 告訴你**不同平台之間的流量是獨立還是共享**（關鍵！）
4. 提供**多管齊下的策略**（Q1：當一個平台流量爆了，能無痛切換到另一個嗎？）
5. 註明每個資訊的**來源時間戳**，讓未來更新時知道哪裡要重查

**這份是 living document，任何時候發現某個平台有重大政策變動，就回來這裡加上更新日期和新資料。**

---

## TL;DR

**五大平台（2026-04 快照）：**

| 平台 | 免費方案 | 最便宜付費 | 核心模型 | 推薦用途 |
|---|---|---|---|---|
| **Google Antigravity** | 有（頻繁被砍） | Google AI Pro $20/月 | Gemini 3.1 Pro、Claude 4.6 | Google 生態 IDE 體驗 |
| **Gemini CLI** | 1000 req/day（Pro 100、Flash 900） | Google AI Pro $20/月 | Gemini 2.5 Pro / Flash | 終端機快速使用 |
| **Claude Code** | ❌ 無永久免費 | Pro $20/月（40-45 msg/5hr）| Claude Opus 4.6、Sonnet 4.6 | 品質最高的深度編碼 |
| **OpenAI Codex CLI** | ❌ 無（要 ChatGPT Plus 起跳） | ChatGPT Plus $20/月 | GPT-5.3 Codex、GPT-5.4 | OpenAI 生態使用者 |
| **Cursor** | Hobby（2000 補全 + 50 慢速 Premium） | Pro $20/月 | Claude / GPT / Gemini 全接 | 多模型切換彈性 |

**🔑 關鍵洞察：**

1. **Antigravity 和 Gemini CLI 在同一個 Google 帳號下是獨立 quota pool**——兩個都可以用到爆，等於雙倍
2. **Claude Code 沒有永久免費版**，新帳號只有 ~$5 試用金
3. **OpenAI Codex CLI 也沒有免費版**，最低要 ChatGPT Plus $20/月
4. **Cursor 的免費 Hobby 是四個平台裡唯一真正「免費可持續」的選項**
5. **多管齊下策略：** 同一個 Google 帳號開 Antigravity + Gemini CLI 雙軌、搭配 Cursor Hobby 當備胎，即使零付費也能撐一天的教學

---

## 一、Google Antigravity

### 基本資訊

- **定位：** Google 的新 AI IDE，2025-11 發表，對標 Cursor 的競品
- **狀態：** 2026-04 仍是 public preview
- **開發商：** Google

### 可用模型

- Gemini 3.1 Pro
- Gemini 2.5 Flash
- Claude Sonnet 4.6
- Claude Opus 4.6
- GPT-OSS 120B
- 其他第三方模型（透過 Google 代理）

### 免費方案

**注意：** Antigravity 的免費額度在 2025-12 到 2026-03 之間被砍了 4 次，體感極度不穩定：

| 時間點 | 事件 |
|---|---|
| 2025-12 | 免費版被砍 92% |
| 2026-02 | 圖片配額緊縮 |
| 2026-03 | 引入 credit 系統 |
| 2026-03 | 連 Ultra 用戶都被突襲砍額度 |

目前免費方案：
- 有「每 5 小時刷新」的基本額度，但有「週配額天花板」
- Gemini 3.1 Pro 可用但限制嚴格
- 部分日額度估計 20-100 requests

**如果要教學用，不建議把 Antigravity 當成唯一平台。**

### 付費方案

| 方案 | 價格 | 特色 |
|---|---|---|
| **Google AI Pro** | $20/月 | Antigravity + Gemini Code Assist 的 Pro 等級額度 |
| **Google AI Ultra** | $250/月 | 最高額度，但 2026-03 連 Ultra 都被突襲砍過 |

付費方案的 quota 每 5 小時刷新。

### 特色 & 警告

- ✅ 一個訂閱同時涵蓋 Antigravity + Gemini Code Assist + Gemini CLI
- ✅ 可以在 IDE 裡直接用 Gemini 3.1 Pro
- ⚠️ **政策極不穩定** — 不要押寶這一個
- ⚠️ 仍是 public preview，有當機回報

### 資料時間戳：2026-04-07

---

## 二、Gemini CLI

### 基本資訊

- **定位：** Google 官方的終端機 CLI 工具
- **跟 Antigravity 的關係：** 共享後端（Gemini Code Assist），但額度**獨立計算**
- **開發商：** Google

### 可用模型

- Gemini 2.5 Pro（稀有配額）
- Gemini 2.5 Flash（大宗配額）
- Gemini 3.1 Pro（需 Google AI Pro/Ultra 訂閱）

### 免費方案（個人 Google 帳號）

**總量：1,000 requests/day**，但內部有兩個子配額：

| 模型 | 每日配額 | 每分鐘 | Token/分 |
|---|---|---|---|
| Gemini 2.5 Pro | **100 req/day** | 5 RPM | 250K TPM |
| Gemini 2.5 Flash | 900 req/day（剩餘） | 60 RPM | — |

**重要：**
- 配額是 **project 層級**，不是每個 API key——多開 key 不會疊加
- 「1000/day」聽起來很多，但只有 100 次是用 Pro 模型，其餘都是 Flash
- 課堂教學前記得看清楚當前是哪個模型

### 付費方案

- **Google AI Pro（$20/月）**：升級 Pro 配額，跟 Antigravity 共用訂閱
- **Google AI Ultra（$250/月）**：最高配額

### 🔑 跟 Antigravity 額度的關係

**確認：Antigravity 和 Gemini CLI 的 quota pool 是獨立的。**

- Antigravity 有自己的 credit-based 系統
- Gemini CLI 跟 Gemini Code Assist 共用 RPM/RPD/TPM 系統
- **同一個 Google 帳號登入兩邊，額度分開計算**
- 社群工具（如 opencode-antigravity-auth）甚至設計「Antigravity 爆了自動 fallback 到 Gemini CLI」的機制，證明兩邊池獨立

**這是教學上非常重要的情報：** 如果 Antigravity 額度卡住，可以立刻切到 Gemini CLI 繼續用，不用換帳號、不用等刷新。等於**免費版就有雙倍容量**。

### 特色 & 警告

- ✅ 免費版 1000/day 總量聽起來很大
- ✅ 跟 Antigravity 雙軌並行的最大贏家
- ⚠️ Pro 模型只有 100/day，要注意模型選擇
- ⚠️ project-level 配額，不能用多 key 作弊

### 資料時間戳：2026-04-07

---

## 三、Claude Code

### 基本資訊

- **定位：** Anthropic 官方的 agentic 編碼工具
- **特色：** 業界公認品質最高的深度編碼
- **開發商：** Anthropic

### 可用模型

- **Claude Opus 4.6**（旗艦）
- **Claude Sonnet 4.6**（主力）
- **Claude Haiku 4.5**（輕量）
- Claude Opus 4.6（1M context beta）

### 免費方案

**⚠️ 沒有永久免費版。**

- 新帳號在 platform.claude.com 註冊可拿到約 **$5 試用金**（API credit）
- 用完就需要訂閱或加值

**這意味著：** Claude Code 不適合「完全免費」的教學路徑。

### 付費方案

| 方案 | 價格 | 5 小時 rolling window | 適合誰 |
|---|---|---|---|
| **Pro** | $20/月 | ~40-45 messages | 個人輕量使用 |
| **Max 5x** | $100/月 | ~225 messages | 認真使用者 |
| **Max 20x** | $200/月 | ~900 messages | 重度開發 |
| **API** | Pay per token | 無上限 | 團隊 / 大量 |

### 特色 & 警告

- ✅ 品質最高 — Claude Opus 4.6 在 SWE-bench 領先其他模型
- ✅ 1M context 可用（Opus 4.6 beta）
- ✅ Rolling window 制度（不是每日重置），爆了等 5 小時
- ✅ 2025-08 起有週配額上限
- ❌ **沒有免費版**，教學時要學生有預算或只當示範看
- ⚠️ 2026-04-04 Anthropic 宣布第三方工具（如 OpenClaw）使用 Claude Pro/Max 配額需額外付費

### 資料時間戳：2026-04-07

---

## 四、OpenAI Codex CLI

### 基本資訊

- **定位：** OpenAI 的開源 CLI 工具，本地執行、呼叫 OpenAI API
- **重要：** **沒有獨立訂閱**——必須包在 ChatGPT 訂閱裡
- **開發商：** OpenAI

### 可用模型

- **GPT-5.4**（旗艦 frontier 模型）
- **GPT-5.3 Codex**（專門 coding 優化）
- **GPT-5.3 Codex Spark**（Pro 方案獨享的 research preview）
- **GPT-5.2 / GPT-5.2 Thinking**

### 免費方案

**⚠️ 沒有免費 Codex CLI 方案。**

- ChatGPT 免費版有 10 messages / 5 hours，但**不能用 Codex**
- 必須訂閱 ChatGPT Plus 起跳才能用 Codex

### 付費方案（ChatGPT 訂閱包含 Codex）

| 方案 | 價格 | Codex 用量 | GPT-5.2 messages |
|---|---|---|---|
| **ChatGPT Plus** | $20/月 | 30-150 msg/5hr | 160 msg/3hr |
| **ChatGPT Business** | $30/用戶/月 | 類似 Plus | — |
| **ChatGPT Pro** | $200/月 | 300-1500 msg/5hr（**5x Plus**） | 更高 |

**2026-04-02 更新：** Codex pricing 改為 **token-based**，跟 API 計價對齊（不再是每訊息計價）。適用所有 Plus / Pro / Business / Enterprise。

### 特色 & 警告

- ✅ GPT-5.4 和 GPT-5.3 Codex 是目前 agent/coding benchmark 最強之一
- ✅ Codex CLI 是開源工具（MIT），本地執行，可稽核
- ✅ 可以用 OAuth 流程把 Codex 接到 IDE、編輯器、自訂工具
- ❌ **完全沒有免費方案**，$20/月是最低門檻
- ⚠️ 2026-04 剛改 token-based 計價，實際「多少訊息」會視 prompt 複雜度變化

### 資料時間戳：2026-04-07

---

## 五、Cursor

### 基本資訊

- **定位：** 多模型可切的 AI 編輯器（fork 自 VS Code）
- **特色：** 可以同一個介面切 Claude / GPT / Gemini，不綁定單一廠商
- **開發商：** Anysphere (Cursor)

### 可用模型

- Claude Opus 4.6、Sonnet 4.6
- GPT-5.4、GPT-5.3 Codex
- Gemini 3.1 Pro、Gemini 2.5 Pro
- 本地模型（透過 OpenAI-compatible endpoint 自訂）

### 免費方案（Hobby）

- **2,000 個 tab completions / 月**
- **50 個 slow premium requests / 月**
- 無需信用卡
- ⚠️ 「evaluation-grade」 — 官方承認是給試用的，不是給認真開發

**但這仍是五大平台裡唯一真正持續免費的選項。**

### 付費方案

| 方案 | 價格 | 特色 |
|---|---|---|
| **Pro** | $20/月 | 無限 Tab completions、$20 premium credits pool |
| **Pro+** | $60/月 | 3x credits |
| **Ultra** | $200/月 | 20x credits |
| **Teams** | $40/user/月 | Pro + 團隊功能 |
| **Business/Enterprise** | 客製 | 自訂 SLA |

### 2025-06 重大改制

Cursor 在 2025-06 把 pricing model 從「固定 fast request 配額」改成 **usage-based credit pool**，跟底層 API 實際成本對齊。原因是太多用戶用 frontier 模型，flat-rate 無法持續。

**影響：** 舊的 Cursor 教學裡「每月 500 fast requests」之類的說法已**過時**。現在是 credit pool 制。

### 特色 & 警告

- ✅ 免費 Hobby 夠做課堂示範和輕量 side project
- ✅ 多模型彈性是其他平台沒有的
- ✅ VS Code 生態相容（多數擴充直接用）
- ⚠️ Premium credits 會不知不覺用完，教學時要注意 prompt 效率
- ⚠️ 政策變動比 Claude Code 更頻繁

### 資料時間戳：2026-04-07

---

## 六、⭐ 多管齊下策略（給教學者）

### 目標：用零付費或最低成本達成一堂完整的 vibe coding 課

### 策略 A：全免費三軌並行（推薦給預算有限的教學情境）

1. **主力：Antigravity**（Gemini 3.1 Pro 可用，體驗 IDE agent 流程）
2. **備胎 1：Gemini CLI**（Antigravity 爆了切這裡，獨立 quota pool，1000/day Flash + 100/day Pro）
3. **備胎 2：Cursor Hobby**（再爆了切這裡，2000 completions + 50 premium）

**爆的順序保護：** 同一堂 2-3 小時的課，同時用光三個的機率極低。而且 Antigravity 和 Gemini CLI 是**同一個 Google 帳號**登入，切換只要換工具，不用換帳號。

### 策略 B：花 $20 買穩定性

挑一個付費，取代策略 A 的「爆了切」手動流程：

- **追求品質：Claude Code Pro $20/月** — 40-45 messages/5hr，品質最高但爆了要等 5 小時
- **追求 Google 生態：Google AI Pro $20/月** — 同時升級 Antigravity + Gemini CLI + Code Assist
- **追求多模型：Cursor Pro $20/月** — 同一介面切 Claude/GPT/Gemini，適合示範對比
- **追求 OpenAI 生態：ChatGPT Plus $20/月** — 包含 Codex CLI + ChatGPT 本體

### 策略 C：花 $100-200 買重度使用

如果真的高頻使用：

- **Claude Max 5x $100/月** — 225 msgs/5hr，品質最高
- **ChatGPT Pro $200/月** — 包含 Codex Pro 額度 + GPT-5.3 Codex Spark
- **Cursor Ultra $200/月** — 20x credits，多模型

### ❌ 別做的事

1. **不要同時買多個** — 除非你真的有用爆一個才考慮加第二個
2. **不要把 Antigravity 當唯一平台** — 它的政策穩定性是所有平台裡最差的
3. **不要靠 Claude Code 新帳號 $5 試用做教學** — 太少，會尷尬
4. **不要示範 OpenAI Codex CLI 的「免費版」** — 它不存在

---

## 七、比較速查表

### 額度計算單位對照

| 平台 | 計算單位 | 重置週期 | 備註 |
|---|---|---|---|
| Antigravity | Credits（不穩定） | 5 小時 + 週上限 | 常被突襲砍 |
| Gemini CLI | Requests / Tokens | 每天 | Pro/Flash 分開算 |
| Claude Code | Messages | 5 小時 rolling | + 週上限（2025-08 起） |
| OpenAI Codex | Messages / Tokens | 5 小時 | 2026-04 改 token-based |
| Cursor | Completions + Credit Pool | 每月 | Credit 對齊底層 API 成本 |

### 免費方案能用的日常量對照

| 平台 | 一天大約能寫多少 code？ | 能完整做完一個 side project 嗎？ |
|---|---|---|
| Antigravity（免費） | 20-100 次請求，不穩定 | 可能中途中斷 |
| Gemini CLI（免費） | 100 Pro + 900 Flash = 1000 次 | 可以，但複雜任務靠 Flash |
| Claude Code（免費） | 無 | ❌ |
| OpenAI Codex（免費） | 無 | ❌ |
| Cursor Hobby | 2000 補全 + 50 slow premium | 可以做簡單 project |

### 付費 $20/月檔位比較

| 平台 | $20/月能拿什麼 | 最大優勢 |
|---|---|---|
| Google AI Pro | Antigravity + Gemini CLI 雙 Pro 等級 | 雙軌並行 |
| Claude Pro | 40-45 msgs/5hr + 週上限 | 品質最高 |
| ChatGPT Plus | 30-150 Codex msgs/5hr + ChatGPT 本體 | GPT-5 生態 |
| Cursor Pro | 無限 Tab + $20 premium credits | 多模型彈性 |

---

## 八、需要回頭更新時注意的地方

**下次來更新這份文件前，請先重新確認：**

- [ ] Antigravity 的配額政策（最容易變動）
- [ ] Gemini CLI 的 Pro / Flash 每日量（數字可能改）
- [ ] Claude Code 的 Pro/Max 每 5 小時數字
- [ ] Codex CLI 的 token-based 計價細節（2026-04 剛改，可能還在調整）
- [ ] Cursor 的 credit pool 匯率
- [ ] 任何新進入者（例如 GitHub Copilot Coding Agent、Zed AI、Continue.dev 付費方案等）

**每次大更新完，請把本文件的「最後更新」日期往後推，並在下方新增「修訂紀錄」。**

---

## 九、修訂紀錄

- **2026-04-07 v1（初版）** — 由 Jarvis（Dispatch 研究）產出初版，涵蓋 Antigravity / Gemini CLI / Claude Code / OpenAI Codex CLI / Cursor 五大平台

---

## 十、參考來源（2026-04-07 快照）

**Google Antigravity：**
- [Antigravity Pricing 2026 (vibecoding.app)](https://vibecoding.app/blog/google-antigravity-pricing-2026)
- [Antigravity Rate Limits 官方 Blog](https://blog.google/feed/new-antigravity-rate-limits-pro-ultra-subsribers/)
- [Antigravity Quota Cut 事件分析 (apiyi.com)](https://help.apiyi.com/en/google-antigravity-quota-cut-policy-changes-developer-impact-guide-en.html)

**Gemini CLI：**
- [Gemini CLI Quotas 官方 (geminicli.com)](https://geminicli.com/docs/resources/quota-and-pricing/)
- [Gemini API Rate Limits 官方](https://ai.google.dev/gemini-api/docs/rate-limits)
- [Dual Quota System 證實 (opencode-antigravity-auth)](https://lzw.me/docs/opencodedocs/NoeFabris/opencode-antigravity-auth/platforms/dual-quota-system/)

**Claude Code：**
- [Claude Code Pricing 2026 (nxcode.io)](https://www.nxcode.io/resources/news/claude-code-pricing-2026-free-api-costs-max-plan)
- [Claude Help Center — Using Claude Code with Pro/Max](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan)
- [Claude Max Plan Explained (IntuitionLabs)](https://intuitionlabs.ai/articles/claude-max-plan-pricing-usage-limits)

**OpenAI Codex CLI：**
- [Codex Pricing 官方 (developers.openai.com)](https://developers.openai.com/codex/pricing)
- [OpenAI Codex Pricing 2026 (uibakery.io)](https://uibakery.io/blog/openai-codex-pricing)
- [ChatGPT Pro $100 5X Codex Limit (VentureBeat)](https://venturebeat.com/orchestration/openai-introduces-chatgpt-pro-usd100-tier-with-5x-usage-limits-for-codex)
- [Codex Rate Card (OpenAI Help)](https://help.openai.com/en/articles/20001106-codex-rate-card)

**Cursor：**
- [Cursor 官方 Pricing](https://cursor.com/pricing)
- [Cursor Models & Pricing Docs](https://cursor.com/docs/models-and-pricing)
- [Cursor Pricing Explained (vantage.sh)](https://www.vantage.sh/blog/cursor-pricing-explained)

**綜合比較：**
- [AI Coding Tools Pricing Comparison 2026 (nxcode.io)](https://www.nxcode.io/resources/news/ai-coding-tools-pricing-comparison-2026)

---

*本報告是 living document，由 Jarvis（Dispatch 研究 + Mac Code Task 落檔）維護。任何人看到政策變動都歡迎回頭更新此檔案。*

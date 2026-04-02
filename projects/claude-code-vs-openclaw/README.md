# Claude Code vs OpenClaw — 比較評估

**建立日期**：2026-04-02
**目的**：評估 Jones 是否應該從 OpenClaw 切換到 Claude Code，或並行使用

---

## 一、背景

Jones 自 2026/01 開始使用 OpenClaw（Jarvis），兩個月來建立了深度的使用習慣。但最近遇到幾個痛點：
- OpenClaw 4.1 更新後 exec approval 機制突然收緊，需要手動修復
- 偶爾漏接訊息
- 更新越來越複雜

同時，Claude Code 近期快速發展（甚至源碼都洩漏了），功能越來越強。值得評估是否切換。

---

## 二、核心差異比較

| 維度 | OpenClaw (Jarvis) | Claude Code |
|------|------------------|-------------|
| **本質** | 多通道 AI agent 平台（Slack/Telegram/WhatsApp/LINE） | 終端機/IDE 內的 coding agent |
| **互動方式** | 隨時隨地透過聊天 app 對話 | 需要開終端機或 VS Code |
| **24/7 可用** | ✅ Gateway 常駐，cron job 自動跑 | ❌ 需要手動啟動 session |
| **多頻道** | ✅ Slack + Telegram + WhatsApp + LINE | ❌ 只有終端機/IDE/桌面 app |
| **Cron 排程** | ✅ 內建，每天自動跑任務 | ❌ 需要自己用 OS 的 crontab 搭配 |
| **記憶系統** | ✅ MEMORY.md + memory/*.md + 向量搜尋 | ✅ CLAUDE.md（每輪重讀，40K 字元） |
| **Sub-agent** | ✅ 內建 sessions_spawn | ✅ 內建子 agent（共享快取，5個花1個的錢） |
| **檔案操作** | ✅ Read/Write/Edit | ✅ Read/Write/Edit |
| **Shell 執行** | ✅ exec 工具 | ✅ 直接在終端跑 |
| **瀏覽器控制** | ✅ browser 工具 | ❌ 沒有（需外掛 MCP） |
| **訊息推送** | ✅ 主動傳訊到 Slack/Telegram | ❌ 無法主動推送 |
| **Skills/插件** | ✅ 豐富的 skill 生態系 | ✅ 插件系統（hooks + plugins） |
| **成本** | Claude Max $100/月（含 OpenClaw 免費） | Claude Max $100/月 或 API 付費 |
| **穩定性** | ⚠️ 偶爾更新後設定被重置 | ⚠️ npm 發布出過兩次 source map 洩漏 |

---

## 三、Jones 的使用場景分析

### Jones 最常用的功能

| 使用場景 | OpenClaw 能做？ | Claude Code 能做？ |
|---------|:-:|:-:|
| **Slack 隨時對話** | ✅ | ❌ 要開電腦打開終端 |
| **WhatsApp/Telegram 接收訊息** | ✅ | ❌ |
| **凌晨 3 點 cron 自動爬蟲** | ✅ | ❌ 要自己搭 crontab + CLI |
| **派 sub-agent 跑 27 個 context** | ✅ | ✅ 而且快取更省 token |
| **git commit + push** | ✅ | ✅ |
| **curl Firestore API** | ✅ | ✅ |
| **瀏覽器操作** | ✅ | ❌ |
| **Heartbeat 主動關心** | ✅ | ❌ |
| **跨裝置使用（手機上聊天）** | ✅ | ❌ |
| **coding（寫程式、debug）** | ✅ | ✅✅ 更強（專為此設計） |

### 結論：Claude Code 無法完全替代 OpenClaw

**Claude Code 更強的地方**：
- 純 coding 場景（寫程式、debug、重構）— 這是它的主戰場
- Sub-agent 快取機制更省 token
- CLAUDE.md 的分層設計更成熟

**OpenClaw 不可替代的地方**：
- 多通道隨時可達（Slack/WhatsApp/Telegram）— 你不可能隨時開終端機
- Cron job 自動排程 — Claude Code 沒有內建
- 主動推送訊息 — Claude Code 做不到
- 瀏覽器控制 — Claude Code 沒有
- Jones 的整套生態系已經建在 OpenClaw 上（memory、skills、cron、多通道）

---

## 四、從洩漏源碼看 Claude Code 的優勢

根據洩漏的 51 萬行源碼分析：

1. **CLAUDE.md 每輪重讀** — 40K 字元額度，分層結構（全域/專案/模組/私人）
2. **5 種壓縮策略** — 對長對話的處理更成熟
3. **多 Agent 架構** — Explore/Plan/Verification 分離，子 Agent 共享快取
4. **95% 是 harness** — 安全檢查、權限控制、工具編排的工程非常深

---

## 五、建議策略

### 不要切換，並行使用

| 場景 | 用什麼 |
|------|--------|
| 日常對話、討論、brainstorm | **OpenClaw**（Slack 隨時聊） |
| Cron job、自動爬蟲、排程任務 | **OpenClaw** |
| 接收多通道訊息 | **OpenClaw** |
| 純 coding 專案（新 repo、大重構） | **Claude Code**（終端機裡更順） |
| Polymarket agent 開發 | **Claude Code**（coding 為主） |

### 怎麼並行？
- 兩者都用同一個 Claude Max 訂閱
- OpenClaw 繼續當「隨時可達的 Jarvis」
- Claude Code 當「需要深度 coding 時的加速器」
- 不衝突，互補

---

## 六、下一步行動

- [ ] 在 Mac 上安裝 Claude Code CLI：`curl -fsSL https://claude.ai/install.sh | bash`
- [ ] 用 Claude Code 試做 Polymarket agent 的第一個 prototype
- [ ] 比較體驗差異後再決定長期策略
- [ ] OpenClaw 繼續維持現有的 cron + 多通道 + memory 生態

---

*結論：Claude Code 是更強的 coding 工具，但 OpenClaw 是更完整的「生活 AI agent」。兩者互補，不需要二選一。*

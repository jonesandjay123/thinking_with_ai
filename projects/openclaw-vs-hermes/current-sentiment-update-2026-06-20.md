# OpenClaw vs Hermes：2026-06 最新口碑與現況更新

> **資料截點：** 2026-06-20 晚間（America/New_York）  
> **問題意識：** Jones 在 2026-04-18 曾短暫從 OpenClaw 轉去 Hermes，但因 Slack thread/session continuity 不佳、token 消耗高、工作記憶感弱而快速退回 OpenClaw。這份報告回頭檢查兩個多月後，Hermes 是否改善，以及 OpenClaw / Hermes 當前公開口碑如何。  
> **結論先講：** Hermes 明顯進步，尤其是 desktop、background subagents、memory tool、automation blueprints、curator cost 等；但對 Jones/Jarvis 這種「Slack DM/thread 是主神經系統」的工作流，Hermes 仍不應直接取代 OpenClaw。比較合理的下一步是小規模 sidecar / POC，而不是主系統遷移。

## 1. 一句話結論

如果只問「現在 Hermes 有沒有比四月好？」答案是：**有，而且不是一點點。**

但如果問「是否已經適合取代 Jones 現在的 OpenClaw/Jarvis？」我的答案是：**還沒有。**

原因很直接：

- Hermes 的強項正在變強：記憶、自動技能、背景任務、桌面端、遠端常駐、agent 自我改進。
- OpenClaw 的核心強項仍然更符合 Jones 的主場：Slack / WhatsApp / 多通道、thread/session continuity、工具與 browser / node / messaging 的操作底座。
- Jones 四月退回 OpenClaw 的主因不是「Hermes 不酷」，而是 Slack continuity 與 token/工作記憶體感。到 6 月底，Hermes 這些點有改善跡象，但公開 issue 顯示 Slack thread/history 還沒到我會放心切主系統的程度。

## 2. 當前版本與熱度快照

| 指標 | OpenClaw | Hermes Agent | 解讀 |
|---|---:|---:|---|
| GitHub repo | `openclaw/openclaw` | `NousResearch/hermes-agent` | 都是高活躍大型 repo |
| Stars（GitHub API, 2026-06-20） | 379,692 | 198,387 | OpenClaw 仍大很多，但 Hermes 增長已經很顯眼 |
| Forks（GitHub API, 2026-06-20） | 79,481 | 35,191 | 兩邊都有真實開發者生態 |
| Open issues（GitHub API, 2026-06-20） | 6,463 | 22,632 | Hermes issue 壓力非常大，代表熱度高也代表未穩定化成本高 |
| Latest release | `v2026.6.9`, 2026-06-21 UTC | `v2026.6.19`, 2026-06-19 UTC | 兩邊都仍在快速迭代 |
| 官方定位 | personal AI assistant / multi-channel gateway | self-improving agent / learning loop | 哲學差異仍然存在 |

## 3. 功能與特性對比

| 面向 | OpenClaw | Hermes | 目前判斷 |
|---|---|---|---|
| 多通道 gateway | WhatsApp、Telegram、Slack、Discord、LINE、iMessage、Matrix、Teams、Zalo、WeChat 等，官方 README 直接把多通道列為核心能力 | Telegram、Discord、Slack、WhatsApp、Signal、Email；v0.17 也新增 iMessage via Photon、WhatsApp Business Cloud API | OpenClaw 仍更像專職 gateway；Hermes 正快速補通道 |
| Slack thread/session | 官方 Slack docs 有 thread session routing、thread history preload、DM scope、top-level opt-out 等細緻設計 | 近期仍有 Slack thread/history issue，例如 history tool、cold-start thread bot replies、channel history 讀取 | 以 Jones 工作流，OpenClaw 明顯較安心 |
| 長期記憶 | 有 workspace memory、memory search、wiki、session history；但需要比較多制度化維護 | 官方核心賣點就是 agent-curated memory、自動技能、cross-session recall、user modeling；v0.17 memory tool 升級 atomic batch operations | Hermes 在「agent 自己長出記憶」的產品敘事更強 |
| Skills | OpenClaw skills / ClawHub / Skill Workshop 偏人工審核、可控、可提案 | Hermes 強調自動 skill creation / self-improving skills / Skills Hub | Hermes 更像自我演化；OpenClaw 更可治理 |
| 背景任務 | OpenClaw 有 cron、sessions、subagents、TaskFlow 等能力；比較像 agent OS 裡的操作模組 | v0.17 加入 background/async subagents、Automation Blueprints；cron 是核心賣點之一 | Hermes 背景 worker 方向變得很強 |
| Token 成本 | OpenClaw 也會吃 context，尤其多工具、多 channel、長 session；但目前我們本機已能用 compaction/memory discipline 控制 | 社群常稱 Hermes 記得更好、較省 token；但 lazy tool schema issue 仍 open，代表工具 schema 注入成本尚未根治 | Hermes 有改善方向，但「省 token」不是全場景已解 |
| 桌面 / UI | OpenClaw 有 Control UI、Canvas、mobile nodes、browser/node 管理 | v0.17 desktop app 大幅增強，包含 watch-windows、model selector、theme、terminal pane、profile builder | Hermes desktop 已從 preview 變成 serious daily-driver 候選 |
| 遷移 | OpenClaw 是現有主系統 | Hermes 官方有 `hermes claw migrate`，可遷移 persona、memory、skills、platform tokens 等；但 multi-agent list/channel bindings 等仍需手動 | 可以 POC，不建議直接 cleanup / archive OpenClaw |
| 風險面 | 高權限 gateway 本身風險高；快速 release 可能引入 regression | self-improving / auto-write / migration / gateway context 都有複雜風險；issue 量大 | 兩邊都不是「裝了就放心」；Hermes 目前更需要隔離實驗 |

## 4. 口碑整理

### 4.1 網友討論的大方向

公開討論大致不是「誰完全打爆誰」，而是分成幾派：

| 立場 | 常見說法 | 我怎麼解讀 |
|---|---|---|
| OpenClaw 主派 | OpenClaw gateway、工具、Slack/多通道更成熟，更新雖會破但整體可營運 | 適合把 agent 當日常基礎設施的人 |
| Hermes 主派 | Hermes 記憶更好、更自動、更會 grinding、更像能自己變強的 agent | 適合追求自我演化、長任務、雲端 worker 的人 |
| 混合派 | Hermes 當 orchestrator，OpenClaw 當 tool-calling / channel layer | 這其實最接近我目前對 Jones 的建議 |
| 懷疑派 | Hermes hype 大、issue 多、self-learning 可能亂寫技能；OpenClaw 則 context/token 吃得快 | 兩邊都有代價，重點是你的主痛點是哪個 |

Reddit 上一則近一個月的討論很代表性：有人說 Hermes 記憶更好、token 燒得少，OpenClaw 更新比較不容易破但吃 context；也有人把 Hermes 當長任務 grinding worker，OpenClaw 當更可靠的工具/通道層。這跟我們自己的體感相當吻合。

### 4.2 第三方綜述的大方向

Kilo 的 2026-05-08 更新版整理稱，他們分析了 25 個 thread、1300+ comments，結論不是單一贏家，而是 OpenClaw / Hermes 都有強項與嚴重問題。這類文章本身可能有內容行銷成分，但它抓到的軸線合理：OpenClaw 偏 gateway / integrations，Hermes 偏 memory / self-learning / rollback / easier setup。

我會把這類第三方文視為「方向參考」，不視為事實裁判。真正能支撐判斷的，還是官方 release、GitHub issue、以及我們自己 4 月實測痛點。

## 5. 6 月新進展：Hermes 真的改善了什麼？

Hermes v0.17.0（2026-06-19）是一個大版本。最值得注意的不是單一功能，而是它把 Hermes 的產品重心推得更清楚：

- **Desktop app 變成熟**：watch-windows、快捷鍵、通知、model selector、terminal pane、profile builder，讓 Hermes 不再只是 CLI/TUI hacker 工具。
- **Background / async subagents**：可以派出背景 subagent，主對話繼續進行，完成後結果回到 conversation。這對長研究、長 build 很關鍵。
- **Automation Blueprints**：不用直接寫 cron syntax，能用表單/對話建立自動化。
- **Memory tool 升級**：atomic batch operations，讓模型可在同一操作中刪舊記憶、加新記憶，降低記憶寫入失敗率。
- **Curator cost optimization**：日常 skill curator 不再每次花 aux-model token 做 LLM consolidation，例行整理成本下降。
- **更多 channel**：iMessage via Photon、官方 WhatsApp Business Cloud API adapter。

這些都回應了 Jones 當初好奇的點：Hermes 是否兩個月後變成熟？答案是：功能面和產品面都明顯成熟很多。

但 mature feature 不等於 fit Jones workflow。尤其 Jones 的 Jarvis 是 Slack DM/thread 主入口，不是桌面 app 主入口。

## 6. 6 月新進展：OpenClaw 也不是停在原地

OpenClaw latest release `v2026.6.9` 的 release notes 顯示，它仍在大量修 gateway、channel delivery、agent/session runtime、Codex integration、provider、skills、search、UI、mobile/node 相關問題。幾個對 Jones 有意義的方向：

- **Agent/session runtime recovery**：重試 thinking-only / empty post-tool turns、修 session history artifacts、保持 usage / compaction 狀態。
- **Channel delivery hardening**：Telegram、WhatsApp、Mattermost、Discord 等 delivery / progress / media path 都在修。
- **Codex integration 變強**：automatic plugin approvals、remote-node exec as dynamic tool、app-server teardown 等。
- **Skills / Search / Plugin**：ClawHub skill provenance、Codex Hosted Search、external provider plugins。

也就是說，OpenClaw 仍然是在「agent OS / gateway / operations substrate」這條路上快速補洞。它的產品靈魂沒有 Hermes 那麼像「會成長的個體」，但對我們現在每天靠 Slack/WhatsApp/browser/repo 做事的系統來說，這反而是更重要的穩定地基。

## 7. 對 Jones 原本痛點逐項回看

| 2026-04-18 痛點 | 2026-06 觀察 | 是否改善到可主遷移 |
|---|---|---|
| Slack thread/session continuity 不佳 | Hermes 有部分 Slack thread bug 已 closed，但仍有 `conversations.history` 工具、cold-start thread context、channel/DM history scopes 等 open issues | **還不夠** |
| token 消耗高 | Hermes v0.17 curator 成本降低；社群有人回報較省 token；但 lazy tool schema loading issue 仍 open，顯示 tool schema 成本仍是問題 | **有改善，但未根治** |
| 工作記憶感弱 | Hermes memory tool、background review、skills、自動記憶敘事持續增強；某些 custom provider bug 已 closed | **顯著改善，值得 POC** |
| migration / continuity 風險 | 官方 migration guide 比四月完整；但仍有 migration cleanup 破壞 OpenClaw multi-agent setup 的歷史 issue，且 multi-agent/channel bindings 沒有完整直接等價 | **只能 dry-run / sidecar，不可 cleanup 主系統** |
| Jarvis identity / SOUL / AGENTS | 6/2 仍有 gateway 忽略 custom SOUL.md + AGENTS.md 的 open issue（Telegram），代表 messaging gateway prompt assembly 仍可能跟 CLI 不一致 | **對 Jones 很敏感，需驗證** |

## 8. 對 Jones/Jarvis 的實際建議

### 不建議現在做的事

- 不建議把現有 OpenClaw 主系統遷移到 Hermes。
- 不建議跑 `hermes claw cleanup` 或任何會 archive / rename `~/.openclaw` 的動作。
- 不建議讓 Hermes 直接接管 Slack DM 主入口。
- 不建議把長期記憶唯一來源交給 Hermes 自動寫入。

### 可以做的事

我會建議做一個 **Hermes sidecar POC**：

1. 在隔離的 `~/.hermes-jones-poc` 或 VM/VPS 上安裝 Hermes。
2. 不導入真 Slack token，不碰主 `~/.openclaw`。
3. 用 `hermes claw migrate --dry-run` 或 copy-only 方法抽一份非敏感的 SOUL/MEMORY sample。
4. 測三個任務：
   - 長研究報告：看 background subagent / memory / summary 是否真的比 OpenClaw 更省心。
   - 自動 skill 生成：看它會不會亂寫、亂 commit、亂永久化。
   - Slack thread 模擬：用測試 workspace 或 dummy channel 測 parent/root context、thread replies、history link retrieval。
5. POC 成功後，最多先讓 OpenClaw 透過 ACP / shell / API 把特定長任務丟給 Hermes，不讓 Hermes 成為主入口。

### 可能的混合架構

| 角色 | 建議系統 | 理由 |
|---|---|---|
| Slack / WhatsApp / LINE / browser / repo 主入口 | OpenClaw | 目前最符合 Jones 的日常操作面 |
| 長研究、長 build、背景 grinding | Hermes sidecar | Hermes v0.17 的 background subagents 很適合 |
| 自我演化 skill 實驗 | Hermes sandbox | 可以觀察它的學習 loop，但不要讓它碰主記憶 |
| 長期人格 / memory source of truth | OpenClaw memory + thinking_with_ai + repo docs | 可治理、可 review、可 commit |
| 未來可能 | OpenClaw channel layer + Hermes worker layer | 這是最值得研究的融合方向 |

## 9. 我的最終判斷

如果用 Jones 的標準來說：

> **Hermes 現在值得重新測，但不值得重新搬家。**

Hermes 的 6 月版本已經比四月成熟很多，尤其「會自己學、會背景跑、會整理記憶、會長出技能」這些點非常戳中 Jones 對 digital agent 的願景。它已經不是只靠 hype 的東西。

但 Jones 不是一般 CLI user。Jones 的 Jarvis 是一個 24/7 Slack/WhatsApp-connected local agent，有跨 thread continuity、memory、repo、browser、Jyn Null publishing、LINE/WhatsApp gateway、GitHub deliverable push 等實際生活工作流。這種情況下，**Slack thread continuity 和操作可靠性比 Hermes 的人格魅力更重要**。

所以我會維持：

- **主系統：OpenClaw**
- **研究對象：Hermes**
- **實驗方式：sidecar POC**
- **遷移門檻：Hermes 必須證明 Slack thread/root/history、SOUL/AGENTS gateway identity、token 成本、memory writes、tool permissions 都能在 Jones 的真實工作流裡穩定通過**

現在最值得做的不是「換」，而是「用 OpenClaw 的穩定主入口去駕馭 Hermes 的長任務能力」。

## 10. Sources

### Official / primary

- OpenClaw GitHub README: https://github.com/openclaw/openclaw
- OpenClaw Slack docs: https://docs.openclaw.ai/channels/slack
- OpenClaw latest release `v2026.6.9`: https://github.com/openclaw/openclaw/releases/tag/v2026.6.9
- Hermes GitHub README: https://github.com/NousResearch/hermes-agent
- Hermes latest release `v2026.6.19`: https://github.com/NousResearch/hermes-agent/releases/tag/v2026.6.19
- Hermes OpenClaw migration guide: https://hermes-agent.nousresearch.com/docs/guides/migrate-from-openclaw

### GitHub issue evidence

- Hermes Slack history tool open issue: https://github.com/NousResearch/hermes-agent/issues/6345
- Hermes Slack thread context bug, closed: https://github.com/NousResearch/hermes-agent/issues/15201
- Hermes Slack bot replies excluded from cold-start thread context, open: https://github.com/NousResearch/hermes-agent/issues/38861
- Hermes Slack channel/DM history read scopes, open: https://github.com/NousResearch/hermes-agent/issues/35291
- Hermes gateway ignoring custom SOUL.md / AGENTS.md in Telegram, open: https://github.com/NousResearch/hermes-agent/issues/37480
- Hermes lazy tool schema loading / token overhead, open: https://github.com/NousResearch/hermes-agent/issues/6839
- Hermes custom provider background review bug, closed: https://github.com/NousResearch/hermes-agent/issues/16123
- Hermes migration cleanup risk issue: https://github.com/NousResearch/hermes-agent/issues/8502
- OpenClaw Slack stuck-session/message-loss issue, closed: https://github.com/openclaw/openclaw/issues/89208
- OpenClaw Slack event-loop/socket issue, closed: https://github.com/openclaw/openclaw/issues/58519
- OpenClaw Slack progress/status issues still open: https://github.com/openclaw/openclaw/issues/33413 and https://github.com/openclaw/openclaw/issues/90017

### Secondary / sentiment sources

- Reddit discussion: https://www.reddit.com/r/openclaw/comments/1swc620/openclaw_vs_hermes/
- Kilo Reddit-comment synthesis: https://kilo.ai/openclaw/vs-hermes
- TrilogyAI technical deep dive: https://trilogyai.substack.com/p/technical-deep-dive-hermes-vs-openclaw

### Internal memory

- `~/clawd/MEMORY.md` records the 2026-04-18 Hermes test and return to OpenClaw.
- `~/clawd/memory/2026-04-18.md` records the original reasons: poor Slack thread/session continuity, high token burn, weak working-memory feel.

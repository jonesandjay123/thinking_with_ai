# OpenClaw 危機 10 天後追蹤：社群實際妥協方案（2026-04-14）

> 本報告為《claude-max-openclaw-crisis》系列的第三篇追蹤。距離 2026-04-04 Anthropic 正式斷供 OpenClaw 使用 Claude Pro/Max 訂閱配額已經 10 天。社群已從震驚和抗議進入「務實接受」階段——本篇記錄大家實際選了什麼路、花了多少錢、體驗怎麼樣。
>
> 撰寫時點：2026-04-14
> 資料來源：Reddit、Hacker News、Medium、TechCrunch、OpenClaw 官方文件、pricepertoken.com 社群排行榜

---

## TL;DR

事隔 10 天，社群的反應可以用一句話總結：

> **「60% 的人轉去了 Codex，剩下的要嘛付 API 費、要嘛跑本地模型、要嘛搞多模型路由。幾乎沒有人真的離開 OpenClaw 本身。」**

五條路徑和大致佔比（根據社群討論推估）：

| 路徑 | 佔比 | 月成本 |
|---|---|---|
| **A：轉 OpenAI Codex 訂閱** | ~40% | $0（包在 ChatGPT Plus/Pro） |
| **B：Claude API 付費（per-token）** | ~25% | $3-200+/月 |
| **C：Claude CLI 後端繞道** | ~10% | $0（走 Claude Code 訂閱額度） |
| **D：本地模型（Ollama + Gemma 4 等）** | ~15% | $0 |
| **E：多模型智慧路由** | ~10% | 混合（$10-50/月） |

---

## 一、10 天間的重大事件補充

### Steinberger 被 Anthropic 暫時封號（4/10）

**TechCrunch 報導：** Anthropic 在 4/10 暫時封鎖了 OpenClaw 創始人 Peter Steinberger 的 Claude 存取權限。原因未完全說明，但時間點緊接在他公開批評 Anthropic 的「EEE 策略」之後。

社群反應：
- 加深了「Anthropic 在報復 Steinberger」的觀感
- 加速了部分用戶轉向 OpenAI 的決心
- Steinberger 後來恢復存取，但此事件被視為 AI 平台治理的一個警示案例

### OpenClaw 官方文件更新

- 官方 providers 頁面已把 **OpenAI Codex 列為預設推薦**
- 新增 「Claude CLI backend」作為繞道方案
- 新增免費模型清單（Qwen 3.5 via OpenRouter、Gemma 4 via Ollama）

### 轉換補貼到期提醒

Anthropic 提供的一次性 credit 需在 **2026-04-17** 前領取（Pro $20、Max $100-200），30% extra usage 折扣有效期 90 天。**還沒領的人剩 3 天。**

---

## 二、五條路徑的實際體驗

### 路徑 A：轉 OpenAI Codex 訂閱（最多人選）

**為什麼最多人選：**

- OpenClaw 官方 docs 直接推薦
- OpenAI 的 Thibault Sottiaux 公開背書「Codex 訂閱 + 第三方 harness」組合
- ChatGPT Plus $20/月 或 Pro $200/月 包含 Codex 額度
- **不需要改 OpenClaw 本身的設定太多**——換 provider 就好

**實際操作：**

1. OpenClaw 設定檔裡把 `provider` 從 `anthropic` 改成 `openai`
2. 模型改成 `gpt-5.4` 或 `codex`
3. 注意：Codex 不帶 embeddings，需要另外設 `OPENAI_API_KEY` 跑 `text-embeddings-3`（成本極低，每月幾美分）

**用戶回報：**

- GPT-5.4 的 coding 品質跟 Claude Opus 4.6 接近，但「風格不同」——Opus 更精準、GPT 更快
- 有些用戶反映 GPT 在 tool calling 上偶爾不穩定（比 Claude 差一點）
- 整體滿意度中上，**但有一種「被迫搬家」的不爽感**

### 路徑 B：Claude API 付費（per-token）

**兩種子路徑：**

**B1. 純 API key（console.anthropic.com）**

| 模型 | Input | Output |
|---|---|---|
| Sonnet 4.6 | $3/M tokens | $15/M tokens |
| Opus 4.6 | $15/M tokens | $75/M tokens |

**真實成本體驗（用戶實測）：**

| 使用強度 | 月成本估算 |
|---|---|
| 輕度（1-2 個短任務/天） | $3-15 |
| 中度（每天 2-4 小時） | $20-60 |
| 重度開發者 | $200+ |

有用戶直接比較：**同一個 code review 任務，extra usage bundle 花了 $20，用 API key 只花 $5.30。** API key 幾乎在所有情境都比 extra usage 便宜。

**B2. Extra Usage Bundle**

- Anthropic 提供的官方轉換方案
- 比 API key 貴（但操作更簡單，不用自己管 key）
- 有 30% 折扣（限時 90 天）
- **社群共識：不推薦，API key 更划算**

### 路徑 C：Claude CLI 後端繞道

**OpenClaw 官方推薦的 workaround：**

原理：不直接呼叫 Anthropic API，而是**把請求委派給本地安裝的 `claude` CLI binary**。`claude` CLI 用你的 Claude Code 訂閱認證，自動處理 auth、token refresh、session management。

**操作步驟：**

1. 確認本地有安裝 `claude` CLI（Claude Code）
2. OpenClaw 設定裡把 backend 改成 `claude-cli`
3. 不需要 API key，走你的 Pro/Max 訂閱額度

**風險：**

- ⚠️ 本質上還是在用 Claude 訂閱跑 OpenClaw——只是繞了一道
- **Anthropic 可能隨時堵住這個漏洞**
- 目前（4/14）還能用，但沒有人保證明天還行

### 路徑 D：本地模型（零成本）

**最熱門的組合：**

| 模型 | 工具 | 適合什麼 |
|---|---|---|
| **Gemma 4 26B** | Ollama | 日常 coding、tool calling 強（τ2-bench 85.5%） |
| **Qwen 3.5 27B** | OpenRouter（免費） | SWE-bench 72.4%，接近 GPT-5 Mini |
| **Gemma 4 E4B** | Ollama | 超輕量，長對話 |

**社群使用心得：**

- 大多數人**混合使用**：本地模型做 routine 工作，遇到硬題丟雲端
- Gemma 4 的 function calling 確實很強，OpenClaw 的 skill 系統跑起來順
- **中文能力是軟肋**——Gemma 4 和 Qwen 3.5 的中文都不如 Claude Opus

### 路徑 E：多模型智慧路由（Pro 玩法）

**概念：**

不綁死單一模型，根據任務類型自動選 model：

```
使用者下指令
  ↓
Opus 4.6 當 orchestrator（決定這題要誰做）
  ↓ 簡單任務 → Sonnet 4.6 或 GPT-5 Mini（便宜）
  ↓ 複雜推理 → Opus 4.6（貴但強）
  ↓ Routine 自動化 → 本地 Gemma 4（免費）
  ↓ 驗證輸出 → Opus 快速 review
  ↓
結果回傳使用者
```

**工具：**

- **OpenRouter**：一個 API key 接 200+ 模型，自動路由
- **Haimaker**：類似概念，混合本地和雲端
- **自己用 OpenClaw 的 model routing 功能**

**成本：** 比純 Opus 便宜 50-70%，品質損失 5-10%。

---

## 三、社群模型偏好排名（4/14 快照）

根據 pricepertoken.com 的社群投票和 Reddit 討論：

| 排名 | 模型 | 定位 | 備註 |
|---|---|---|---|
| 1 | **Claude Opus 4.6** | 品質之王 | 貴 + Anthropic 會 ban 重度用戶 |
| 2 | **GPT-5.4（thinking medium+）** | 最佳日常驅動 | 速度快、穩定 |
| 3 | **MiniMax M2.7** | 性價比黑馬 | 便宜、品質意外地好 |
| 4 | **Gemma 4 26B** | 最佳本地選擇 | 免費、tool calling 強 |
| 5 | **Qwen 3.5 27B** | 最佳免費雲端 | OpenRouter 免費、coding 強 |
| 6 | **Claude Sonnet 4.6** | Opus 的平價版 | API 便宜但品質差一截 |

**關鍵觀察：**

- **Claude Opus 仍是品質冠軍，但它的「不可替代性」正在快速下降**
- GPT-5.4 和 MiniMax M2.7 已經能處理 80% 的日常任務
- 本地模型（Gemma 4 + Qwen 3.5）**在有預算約束的情境下完全可用**
- **沒有一個模型能完全取代 Opus**，但「混合使用」已經是主流策略

---

## 四、跟 Jones 的關聯

Jones 在 4/4 當天就選了**路徑 B 的變體**——接受 Claude Code 的 session-based 工作流 + 自建記憶同步機制（claude-mem + CLAUDE.md + memory/）+ Dispatch 作為遠端介面。

10 天後的檢視：

| 維度 | Jones 當時的選擇 | 10 天後社群共識 | 是否需要調整 |
|---|---|---|---|
| **主力模型** | Claude Opus 4.6（Max 訂閱） | Opus 仍是品質王，但 GPT-5.4 / MiniMax 在追上 | 不用改，Max 訂閱夠用 |
| **記憶系統** | claude-mem + CLAUDE.md + memory/ | 社群也開始用類似方案（MemClaw、Letta） | 方向正確 |
| **本地模型** | Gemma 4 on Mac Mini（失敗）→ 5080 Jovix | 本地模型成為主流 Plan B | ✅ 正確，之後在 Jovix 上跑 |
| **OpenClaw** | 暫不復活 | 社群大多仍在用 OpenClaw，只是換了 provider | Jones 可考慮復活 OpenClaw + Codex 或 local Gemma 4 |

**10 天後的新建議：**

1. **OpenClaw + 本地 Gemma 4 on Jovix** 這條路線現在社群已有大量教學和實證，值得 Jones 排上日程
2. **Claude CLI 後端繞道** 這個方式 Jones 現在理論上可以用（他有 Max 訂閱 + 本地 claude CLI），但有被堵住的風險
3. **多模型路由** 如果 Jones 之後想省錢或做更複雜的 agent 任務，OpenRouter 一個 key 接 200+ 模型是最靈活的方案

---

## 五、參考來源

- [OpenClaw After the Claude Ban: $0, $15, or $360/mo?](https://findskill.ai/blog/openclaw-claude-cutoff/)
- [Fix OpenClaw Missing Auth After Anthropic Changes (Medium)](https://medium.com/@ulmeanuadrian/anthropic-just-cut-off-my-ai-agents-heres-how-i-fixed-it-in-20-minutes-741384d27080)
- [OpenClaw + Claude Code Costs 2026: API Key vs Pro vs Max](https://www.shareuhack.com/en/posts/openclaw-claude-code-oauth-cost)
- [Anthropic temporarily banned OpenClaw's creator (TechCrunch)](https://techcrunch.com/2026/04/10/anthropic-temporarily-banned-openclaws-creator-from-accessing-claude/)
- [Breaking: Claude Ends Subscription Coverage (BotLearn)](https://www.botlearn.ai/news/claude-bans-openclaw-most-people-doing-and-talking-20260404)
- [Best Models for OpenClaw April 2026 (haimaker)](https://haimaker.ai/blog/best-models-for-clawdbot/)
- [Free AI Models for OpenClaw (LumaDock)](https://lumadock.com/tutorials/free-ai-models-openclaw)
- [OpenClaw Providers 官方文件](https://docs.openclaw.ai/concepts/model-providers)
- [Price Per Token — OpenClaw Model Rankings](https://pricepertoken.com/leaderboards/openclaw)
- [Anthropic Banned Claude on OpenClaw: $0 Local Alternative](https://nervegna.substack.com/p/anthropic-banned-claude-subs-on-openclaw)

---

*本報告由 Jarvis（Dispatch 研究 + Mac Code Task 落檔）產出。距離 4/4 公告 10 天的社群快照。*

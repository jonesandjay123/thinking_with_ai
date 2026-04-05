# Polymarket Agent — Claude Code 架構可行性分析

**建立日期**：2026-04-04
**報告性質**：OpenClaw → Claude Code 遷移後的可行性重評估
**前提**：本報告不取代 `README.md`，而是作為架構遷移決策文件，針對「用 Claude Code 實作原 README 計畫」進行逐項評估。
**協作模式**：Dispatch Jarvis 研究框架 + Mac Code Task 讀取原文交叉評估 + 起草 commit

---

## TL;DR 結論

**✅ 高度可行，強烈建議推進。**

原 README 規劃的 Pipeline 五步驟，在 Claude Code 架構下每一步都有直接對應的實作路徑。Polymarket 的技術堆疊（CLOB API、SDK、WebSocket）完全不依賴 OpenClaw，遷移後不只「能做」，而且還新增了幾個原本沒有的能力。

在 **Claude Max 訂閱 + 本機 Mac Mini** 的前提下，主要瓶頸已從「成本與 rate limit」轉移到「工程複雜度」（watchdog 設計、context 管理、風險控制）。這是一個更健康的瓶頸——全部都是可解的工程問題。

**推薦方案：方案 α（本機 Mac Mini + Claude Code CLI）** 作為 Phase 1 起點，以 $50 預算上限跑第一個 pipeline。Max 訂閱的高配額讓方案 α 極具優勢，Phase 2 之前不需要考慮 VPS。

---

## 一、原 README 需求對照表

根據 README 的 MVP Pipeline（Step 1-5）和 Loop 架構，逐項評估在 Claude Code 下的支援狀況：

| 需求 | 描述 | Claude Code 狀態 | 備註 |
|------|------|-----------------|------|
| 資訊蒐集 — Polymarket 熱門問題 | 抓市場清單、odds、metadata | ✅ 直接支援 | Gamma API + `py-clob-client` 直接呼叫 |
| 資訊蒐集 — Reddit/Twitter | 爬 social 資訊輔助判斷 | ⚠️ 需替代方案 | Twitter API 付費；Reddit 有限制；建議改用 Brave Search API（Jones 已有 key）+ RSS |
| 資訊蒐集 — 新聞 | 時事搜尋 | ✅ 直接支援 | Claude Code 內建 `WebSearch`；Brave Search API |
| LLM 判斷 — 機率估算 | 對每個問題估算機率 | ✅ 直接支援 | Claude Opus 4.6 / Sonnet 4.6 直接呼叫，比 OpenClaw 時代更新 |
| 市場比較 — 抓現價 | 讀 CLOB 當前 orderbook | ✅ 直接支援 | `py-clob-client` 或 REST API |
| 決策邏輯 — 差距門檻 | 差距 > N% 才下注 | ✅ 直接支援 | Python script，在 Claude Code session 內跑 |
| 下注執行 — 真實下單 | 透過 CLOB API 送出訂單 | ✅ 直接支援 | `py-clob-client` + Ethereum 私鑰簽名 |
| 記錄 — Decision Log | 完整 log 回頭分析 | ✅ 直接支援 | CSV / SQLite，`claude-mem` 跨 session 記憶 |
| 優化 — 分析 log 調整策略 | 用歷史 log 改進決策 | ✅ 直接支援 | `claude-mem` 自動壓縮，跨 session 學習策略 |
| 24/7 持續運行 | 不依賴人工介入 | ⚠️ 需設計 | 本機 Mac Mini 可行，但需加 watchdog；VPS 更穩 |
| 遠端監控 / 暫停 | 手機查狀態、下指令 | ✅ 新能力 | Dispatch + code task；Telegram channel（原 OpenClaw 沒有原生支援）|
| 跨 session 記憶延續 | 學習不因重啟而消失 | ✅ 已建好 | `claude-mem` v10.6.3 + memory/ + CLAUDE.md 已驗證 |

**紅燈只有一個**：Twitter/Reddit 爬蟲在新架構下需要替代方案（見第六節對策）。其他全部可行。

---

## 二、技術堆疊對照（OpenClaw vs Claude Code）

| 元件 | 原 OpenClaw 設想 | Claude Code 對應 | 遷移難度 |
|------|----------------|-----------------|---------|
| **Reasoning Engine** | OpenClaw + Claude / DeepSeek | Claude Code 直呼 Opus 4.6 / Sonnet 4.6 | 🟢 無縫 |
| **Polymarket API 呼叫** | OpenClaw Skill 包裝 | 直接 `import py-clob-client`，沒有中間層 | 🟢 更直接 |
| **Memory System** | OpenClaw Souls + MemClaw | `claude-mem` + `memory/` + `CLAUDE.md` | 🟢 已就位 |
| **Alerting / 通知** | OpenClaw webhooks | Claude Code Channels（Telegram/Discord 原生）+ Slack MCP | 🟢 更完整 |
| **持續排程** | OpenClaw 排程機制 | `Scheduled Tasks`（`/schedule` skill）或 Mac `cron` | 🟡 需設定 |
| **資訊蒐集 wrapper** | OpenClaw 整合 Reddit/Twitter | Brave Search API + RSS + WebSearch | 🟡 需重寫 |
| **背景 agent** | OpenClaw 常駐進程 | `/loop` + watchdog 腳本 | 🟡 需設計 |
| **遠端操控** | 無 | Dispatch → code task | 🟢 新能力 |

**關鍵洞察**：Polymarket 的三大 API（CLOB、Gamma、Data）本身是標準 REST + WebSocket，跟任何 AI 框架完全解耦。OpenClaw 從未是技術必需品，而是一個管理入口。Claude Code 不只替代，而且更直接。

---

## 三、三種部署方案比較

### 方案 α：本機 Mac Mini（推薦 Phase 1）

```
Mac Mini 24/7
  └── Claude Code CLI
        ├── /loop 15m  →  heartbeat agent（掃市場 + 評估）
        ├── Scheduled Tasks  →  每日 8AM EST 報告
        ├── claude-mem  →  跨 session 記憶策略
        └── Telegram Channel  →  推播警示 + 遠端監控
```

| 項目 | 評估 |
|------|------|
| 硬體成本 | $0（已有 Mac Mini 24/7）|
| Claude 成本 | Max 訂閱 $100-200/月（已有）；日常 agent 跑在訂閱配額內，**額外 API 成本趨近於零** |
| Polymarket 手續費 | 僅下注時發生，Phase 1 乾跑 $0 |
| 私鑰安全 | 留在本機 Keychain，不上傳 |
| 穩定性 | ⚠️ Mac 重啟 / 網路中斷會停 |
| 啟動速度 | 最快（環境已就位）|
| 適用場景 | Phase 1-2 全程；Phase 3 前不需要升級 |

### 方案 β：Hetzner VPS（Phase 2 可考慮）

```
Hetzner CX22 ~$5/月
  └── Docker container
        ├── Claude Code CLI（headless）
        ├── supervisord watchdog
        └── Telegram bot bridge
Mac Mini 負責：下指令、審核、記憶備份
```

| 項目 | 評估 |
|------|------|
| 硬體成本 | VPS ~$5-8/月 |
| Claude 成本 | ⚠️ VPS 上的 Claude Code CLI 無法共享 Max 訂閱配額，**需要另外付 API 費用** $30-100/月（看強度）|
| 私鑰安全 | ⚠️ 需謹慎處理，建議加 Vault |
| 穩定性 | ✅ 99.9% uptime，獨立於家用網路 |
| 啟動速度 | 需要 CI/CD 配置，約 1 週 |
| 適用場景 | 家用網路嚴重不穩，或 Phase 3 策略強度到需要 scale 時 |

> **重要提醒**：Jones 是 Max 訂閱用戶，方案 β 的主要吸引力（省 Claude API 費）在這個情況下**反而消失**——VPS 上跑的 Claude Code 是 API 計費，不吃 Max 配額。除非有網路隔離或穩定性需求，否則方案 β 在 Max 訂閱下性價比低於方案 α。

### 方案 γ：混合模式（Phase 3 選項）

- Mac Mini 跑：日常市場監控 + 低頻決策 + memory 管理
- VPS 跑：高頻交易執行層 + 時間敏感下單
- 成本最高，架構最複雜，等真的有策略 edge 再考慮

**建議路徑（Max 訂閱版）**：α（Phase 1-2 全程）→ 若家用網路不穩才考慮 β → γ 只有 Phase 3 真正 scale 時

---

## 四、已知限制與對策

### 1. Context Window 管理

**問題**：一個 heartbeat agent 掃 20 個市場 + 搜尋新聞，可能吃掉 15K-30K token。跑幾輪就撞上限。
**對策**：
- 硬性 session 上限：每 session 50K token 就寫進度到 `memory/`，結束重開
- 用 Sonnet 4.6 做日常掃描（便宜），只在真正需要深度推理時呼叫 Opus
- 每次 heartbeat 只處理「差距 > 門檻候選」，而非掃全部市場

### 2. Rate Limit

**實際狀況（Max 訂閱）**：原本擔心的「Pro plan 45 messages / 5hr」**對 Jones 不適用**。

| Max tier | 配額 | 適用 |
|----------|------|------|
| Max $100/月（5x Pro）| ~200-225 messages / 5hr | 日常 Polymarket agent 綽綽有餘 |
| Max $200/月（20x Pro）| ~900 messages / 5hr | 幾乎不可能撞牆 |

單純的 Polymarket 監控 loop（掃市場 → LLM 判斷 → 記錄）每輪約 5-10 messages，以每 30 分鐘一輪計算，一天約 240-480 messages，在 $100 Max tier 的日配額以內。**撞 rate limit 的主要風險只在同時開多個 parallel subagent 或密集 tool call 的情境。**

**對策（保留基本設計，不因為配額高就放任）**：
- Checkpoint/graceful exit 仍然必要——不是為了省配額，而是為了應對 context window 耗盡
- 非交易時段（深夜）可以放心拉長 heartbeat 間隔（節省 context，不是節省配額）
- 若在 Max 配額內跑完整個 24/7 loop，額外 API 費用趨近於零（只需 Max 本身的月費）

### 3. 背景 Agent 停止問題

**問題**：GitHub issue #41461 記錄，Claude 可能謊稱已停止但實際上沒停。長時間任務尤其危險。
**對策**：
```bash
# watchdog 腳本範例
while true; do
  if ! pgrep -f "claude" > /dev/null; then
    # 重啟 agent
    claude --dangerously-skip-permissions /loop 30m polymarket-heartbeat
  fi
  sleep 300
done
```
- 每次 agent 啟動寫 PID 到 `/tmp/polymarket-agent.pid`
- Telegram channel 每 30 分鐘收到 heartbeat；15 分鐘沒收到就警告

### 4. Twitter/Reddit 爬蟲

**問題**：原 README Step 1 提到「爬 Reddit / Twitter」，但 Twitter API 免費層已失效，Reddit 也有限制。
**對策**：
- **Brave Search API**（Jones 已有 Keychain `brave-api-key`，2000 req/月）：搜「market_name site:reddit.com」
- **RSS feeds**：Reuters、AP、BBC，用 `feedparser` 解析，免費無限制
- **Polymarket 官方 Gamma API**：已經包含市場熱度、交易量排名，可替代「熱門問題」爬蟲
- **結論**：Twitter 直接爬的需求可以 80% 被 Brave Search 替代，剩下 20% 靠 WebSearch

### 5. 成本估算（Max 訂閱版）

**核心前提**：Jones 是 Claude Max 用戶。在訂閱配額內運行的 agent，**不產生額外 API 費用**。成本主要是 Max 本身的月費。

| 使用情境 | 月成本估算（Max 訂閱） |
|---------|----------------------|
| Phase 1：手動觸發 + 偶爾掃描 | Max $100-200（已有）+ $0 Polymarket 手續費（乾跑）|
| Phase 2：每 30 分鐘 heartbeat，方案 α（本機）| Max 月費（已有）+ Polymarket 手續費（小額）|
| Phase 2：每 30 分鐘 heartbeat，方案 β（VPS）| Max 月費 + VPS $5-8 + **額外 Claude API $30-100**（VPS 上不走 Max 配額）|
| Phase 3：全自動 24/7 + 多 subagent，方案 α | Max 月費；若衝破週配額上限才加購 extra usage |

**結論**：相比「API 計費」情境下可能高達 $3,000-10,000/年的年成本，Max 訂閱方案（$1,200-2,400/年）讓整個計畫的財務可行性大幅提升，且成本上限清晰可控。

**Phase 1 建議**：先乾跑（paper trading），不實際下注，只記錄 decision log。驗證 pipeline 正確性比賺錢更重要。

---

## 五、Jones 現有環境加成

相比從零開始，Jones 這台機器已經有以下優勢，保守估計省去 2 週啟動成本：

| 已就位 | 說明 |
|--------|------|
| Mac Mini 24/7 | 主機已存在，不用租 VPS |
| `claude-mem` v10.6.3 | hook 修好、dashboard 在 localhost:37777 |
| Productivity plugin | 雙層記憶系統已初始化 |
| bridges/ 跨入口記憶 | Dispatch ↔ Mac code task 協作已驗收 |
| Gmail + Slack MCP | 通知管道已連接 |
| `jarvisaiasst` GitHub 帳號 | Polymarket agent repo 可直接 push |
| Brave Search API key | Keychain `brave-api-key`，資訊蒐集替代方案已有 |
| `~/.claude/settings.json` permissions | 剛才大幅擴充，減少操作摩擦 |
| Dispatch 遠端監控 | 從手機下指令查 agent 狀態的基礎設施已在 |

---

## 六、Claude Code 專屬能力（OpenClaw 沒有的升級）

這些是遷移後新獲得的能力，值得在實作中善用：

- **`/schedule` skill**：原生排程，不需要 cron wrapper 或外部排程服務
- **`Skills` 系統**：可以寫 `polymarket-agent` skill，把 domain knowledge（API endpoint、策略邏輯、prompt 模板）封裝成可複用模組
- **`claude-mem` 跨 session 學習**：每次 session 的 decision log 自動壓縮進語義搜尋庫，下一個 session 可以問「我上次對 Trump 相關市場的判斷準確率如何？」
- **Dispatch 手機監控**：從 Claude app 直接問「Polymarket agent 現在狀態如何？上次下注是什麼？」——這需要 agent 定期寫 status 到 memory
- **MCP tools 整合**：Slack MCP 可以讓 agent 在重要事件（差距 >15%）時主動發 DM，不依賴 polling

---

## 七、具體實作 Roadmap

### Phase 1：Pipeline 驗證（目標 2 週，$0 實際資金）

**成功指標**：跑 10 個預測、有完整 decision log、能回答「我的準確率是多少？」

```
Week 1：
  □ 安裝 py-clob-client，能抓 top 20 市場清單和 odds
  □ 寫 prompt 模板：給定 [市場標題 + 描述]，請 Sonnet 估算機率
  □ 手動跑 5 個市場的 pipeline（Step 1-4），記錄到 decision_log.csv
  □ 驗證 Gamma API：確認市場 metadata 質量

Week 2：
  □ 加入 Brave Search：搜尋 [市場關鍵字]，擷取前 5 個結果餵給 LLM
  □ 半自動化：寫 Python 腳本把 Step 1-4 串起來，Claude Code 執行
  □ 跑滿 10 個 prediction，分析 decision log
  □ 決定是否進入 Phase 2
```

### Phase 2：真實下注實驗（目標 1 個月，$50 預算硬上限）

**成功指標**：完整的 autonomous loop 跑過至少 20 個市場；有 decision quality 分析

```
□ 完成 Polymarket 帳號 + 入金（$50 上限）
□ 設定私鑰管理（macOS Keychain，不明文儲存）
□ 加入 py-clob-client 真實下單邏輯（先從 $1 最小單位開始）
□ 設定 watchdog 腳本 + Telegram 通知
□ 用 /schedule 排程：每日 8AM 掃市場 + 下注
□ 月末：輸出完整分析報告（ROI、準確率、策略改進方向）
```

### Phase 3：策略迭代（依 Phase 2 結果決定）

```
□ 若 Phase 2 準確率 > 60%：加大倉位、優化 prompt、考慮方案 β（VPS）
□ 若 Phase 2 準確率 < 50%：分析失敗模式，調整資訊來源或推理策略
□ 可選：加入 Kalshi（合法美國版），同樣架構橫向擴展
□ 可選：寫 polymarket-agent Skill，方便任何 session 即時載入
```

---

## 八、風險清單

| 風險 | 嚴重度 | 對策 |
|------|--------|------|
| 私鑰洩漏 | 🔴 高 | 只存 macOS Keychain；code 裡用環境變數；不 commit 到 GitHub |
| 成本失控（Claude API） | 🟢 低 | Max 訂閱內基本無額外費用；風險僅在 Phase 3 密集 subagent 場景，加購 extra usage 前設提醒 |
| Agent 持續運行中斷 | 🟡 中 | watchdog 腳本 + Telegram heartbeat |
| 策略連續虧損超出預算 | 🟡 中 | 程式碼內寫死 $50 上限；每筆最大 $5 |
| Polymarket API 版本變動 | 🟢 低 | 有官方 `Polymarket/agents` repo 可參考；SDK 有版本鎖 |
| 有效市場 — 沒有 edge | 🟡 中 | 在 README 已有清醒認識；Phase 1 先驗證能力，不假設有 edge |
| Mac Mini 重啟斷線 | 🟢 低 | Phase 1 可接受；Phase 2 加 watchdog |

**最高優先風險**：私鑰管理。必須在寫任何下單程式碼之前就設好，不能事後補。

---

## 九、結論

原 README 規劃的 autonomous agent money loop，在 Claude Code 架構下不只可行，而且技術路徑更清晰：

1. **Polymarket API 本身跟 OpenClaw 無關**——這是最好的消息，完全不需要遷移任何 Polymarket 相關邏輯
2. **唯一需要替代的是資訊蒐集**（Twitter 爬蟲 → Brave Search + RSS），已有現成方案
3. **24/7 持續運行**需要 watchdog 設計，但 Mac Mini 已在跑，Phase 1 夠用
4. **新增能力**（Dispatch 遠端監控、Scheduled Tasks、claude-mem 跨 session 學習）讓這個系統比原 OpenClaw 版本更完整

最後，保留 README 裡 Jarvis 說的那句話作為 Phase 1 的精神：

> 「追求的是『系統跑起來』的成就感。」

---

## 參考連結

- [Polymarket/agents — 官方 Agent 架構參考](https://github.com/Polymarket/agents)
- [Polymarket 開發者文件](https://docs.polymarket.com/)
- [CLOB API 介紹](https://docs.polymarket.com/developers/CLOB/introduction)
- [Polymarket API 完整指南](https://agentbets.ai/guides/polymarket-api-guide/)
- [polymarket-apis PyPI](https://pypi.org/project/polymarket-apis/)
- [discountry/polymarket-trading-bot — CLAUDE.md 方案實例](https://github.com/discountry/polymarket-trading-bot/blob/main/CLAUDE.md)
- [Building an Automated Polymarket Trading System with Claude Code (Medium)](https://medium.com/@rvarkarlsson/building-an-automated-polymarket-trading-system-with-claude-code-1982ff60cc74)
- [Two-Layer AI System for Polymarket + Kalshi (Dev Genius)](https://blog.devgenius.io/just-built-a-two-layer-ai-system-that-trades-polymarket-and-kalshi-while-i-sleep-heres-the-aa59ead275f6)
- [Polymarket Development Suite Skill on mcpmarket.com](https://mcpmarket.com/es/tools/skills/polymarket-development-suite)
- [Always-On AI Agent Server Setup](https://okhlopkov.com/always-on-ai-agent-server-setup/)
- [How to Run Claude Code 24/7 Without Burning Context Window](https://dev.to/gentic_news/how-to-run-claude-code-247-without-burning-your-context-window-48ci)
- [Claude Code issue #41461 — 背景 agent 停止問題](https://github.com/anthropics/claude-code/issues/41461)

---

## 修訂紀錄

- **2026-04-04 v1** 初版
- **2026-04-04 v1.1** 更正 rate limit 章節（Jones 實際為 Claude Max 訂閱，非 Pro），重新評估成本與部署方案推薦

---

*本報告由 Jarvis（Dispatch + Mac Code Task 協作模式）產出。Dispatch 端負責研究與架構分析框架，Mac code task 端負責讀取原 README、交叉評估、起草與 commit。*

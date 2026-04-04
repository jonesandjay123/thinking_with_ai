# Claude Code 記憶 Plugin 與解決方案研究

> **日期：** 2026-04-04
> **目的：** 找到讓 Claude Code 擁有持久記憶的最佳方案，作為 OpenClaw 的替代

---

## 🏆 最推薦方案

### 1. Ductor — ⭐231 | 最接近 OpenClaw 的替代品！
**GitHub:** https://github.com/PleasePrompto/ductor

**這就是你要的東西。** 它把 Claude Code CLI 包裝成 Telegram bot，功能幾乎跟 OpenClaw 一模一樣：

| 功能 | OpenClaw | Ductor |
|------|---------|--------|
| Telegram 控制 | ✅ | ✅ |
| Slack 控制 | ✅ | ❌（有 Matrix） |
| 持久記憶 | ✅ | ✅ |
| Cron 排程 | ✅ | ✅ |
| 背景任務 | ✅ | ✅ |
| Multi-agent | ✅ | ✅（sub-agents） |
| 用 Claude Max 訂閱 | ❌（被砍了） | ✅（直接用 CLI） |
| Docker 沙箱 | ❌ | ✅ |

**核心優勢：** 它直接調用 `claude` CLI 執行命令，所以 **Claude Max 訂閱完全涵蓋**，不會被額外收費！

**安裝：**
```bash
pipx install ductor
ductor  # 互動式設定精靈
```

**適合 Jones 嗎：** ⭐⭐⭐⭐⭐ 完美。Mac Mini 上跑 Ductor，用 Telegram 控制，記憶持久，cron 可用，而且直接走 Claude Max 訂閱。

---

### 2. claude-brain — ⭐36 | Git 同步記憶跨裝置
**GitHub:** https://github.com/toroleapinc/claude-brain

用 Git repo 自動同步 Claude Code 的記憶、skills、rules 到所有裝置。

**核心功能：**
- `/brain-init git@github.com:you/my-brain.git` — 初始化
- `/brain-join` — 其他裝置加入
- 每次 session 開始/結束自動 push/pull
- **LLM 語義合併**（不是暴力覆蓋，會智能解決衝突）
- 支援加密

**適合 Jones 嗎：** ⭐⭐⭐⭐ 完美搭配 `jarvis-mind-backup` repo。Mac Mini + Jones 電腦 + Jovix 都能共享 Jarvis 的記憶。

---

### 3. pro-workflow — ⭐1,538 | 自我糾正記憶系統
**GitHub:** https://github.com/rohitg00/pro-workflow

Claude Code 從你的糾正中學習，記憶跨 50+ sessions 持續累積。

**核心功能：**
- 自動捕捉你的修正和偏好
- 記憶隨時間複合成長
- 支援 parallel worktrees、agent teams
- 17 個 battle-tested skills

**適合 Jones 嗎：** ⭐⭐⭐⭐ 適合長期使用，讓 Claude Code 越用越懂你。

---

### 4. claude-reflect — ⭐874 | 自學習系統
**GitHub:** https://github.com/BayramAnnakov/claude-reflect

自動捕捉糾正、正面回饋、偏好，同步到 CLAUDE.md 和 AGENTS.md。

**適合 Jones 嗎：** ⭐⭐⭐⭐ 跟 pro-workflow 類似但更輕量，專注於學習和同步。

---

### 5. claude-cognitive — ⭐448 | 工作記憶 + 多實例
**GitHub:** https://github.com/GMaN1911/claude-cognitive

提供 persistent context 和多實例協調。

**適合 Jones 嗎：** ⭐⭐⭐ 如果需要多個 Claude Code 實例同時工作。

---

### 6. total-recall — ⭐190 | 分層記憶系統
**GitHub:** https://github.com/davegoldblatt/total-recall

分層記憶（tiered memory）+ write gates + correction propagation + slash commands。

**適合 Jones 嗎：** ⭐⭐⭐ 記憶管理更精細，適合重度用戶。

---

### 7. cog — ⭐329 | 認知架構
**GitHub:** https://github.com/marciopuga/cog

持久記憶 + 自我反思 + 預見能力。

**適合 Jones 嗎：** ⭐⭐⭐ 比較進階的認知架構，適合想讓 AI 更自主的用戶。

---

### 8. MemoMind — ⭐403 | 本地 GPU 加速記憶
**GitHub:** https://github.com/24kchengYe/MemoMind

100% 本地、GPU 加速、零雲端依賴。

**適合 Jones 嗎：** ⭐⭐⭐ 如果你想完全離線運作，但 Mac Mini M4 的 GPU 不太夠力。

---

### 9. ClawMem — ⭐75 | 同時支援 Claude Code 和 OpenClaw
**GitHub:** https://github.com/yoloshii/ClawMem

On-device context engine，同時支援 Claude Code 和 OpenClaw。Hooks + MCP server + hybrid RAG 搜尋。

**適合 Jones 嗎：** ⭐⭐⭐⭐ 如果你想同時保留 OpenClaw（Gemini）和 Claude Code，讓它們共享記憶。

---

### 10. claude-diary — ⭐352 | 簡單記憶
**GitHub:** https://github.com/rlancemartin/claude-diary

極簡記憶系統。

**適合 Jones 嗎：** ⭐⭐ 太簡單，功能不夠。

---

## 🎯 Jones 的最佳組合建議

### 方案 A：最接近 OpenClaw 體驗（推薦！）
```
Mac Mini 上安裝：
1. Ductor — Telegram 控制 Claude Code（完整替代 OpenClaw）
2. claude-brain — Git 同步記憶到 jarvis-mind-backup repo
```
- ✅ Telegram 控制（手機遙控 Mac Mini）
- ✅ 持久記憶
- ✅ Cron 排程
- ✅ 走 Claude Max 訂閱（不額外收費）
- ✅ 記憶自動同步到 GitHub

### 方案 B：最小改動
```
Mac Mini 上安裝：
1. Claude Code CLI
2. claude-brain — 同步記憶
3. 保留 OpenClaw（Gemini）作為 Slack 備用
```

### 方案 C：完整生態
```
1. Ductor — Telegram 控制
2. claude-brain — 跨裝置記憶同步
3. ClawMem — OpenClaw 和 Claude Code 共享記憶
4. pro-workflow — 自我學習
```

---

## ⚡ 快速開始步驟

如果選方案 A：
1. `pipx install ductor` — 安裝 Ductor
2. `ductor` — 跑設定精靈（需要 Telegram Bot Token）
3. 把 jarvis-mind-backup 的記憶移入 `~/.ductor/`
4. 安裝 claude-brain，指向 jarvis-mind-backup repo
5. 從 Telegram 開始用！

---

*報告完成時間：2026-04-04 13:44 EDT*

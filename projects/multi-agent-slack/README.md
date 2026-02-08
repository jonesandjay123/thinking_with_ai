# Multi-Agent Slack Workspace

## 狀態：📋 規劃中

## 目標
讓 Jarvis、Juni、Jovix 三個 agent 能在同一個 Slack Workspace 內協作對話。

## 背景
目前每個 agent 各有獨立的 Slack Workspace，各自以該 Workspace 的 OpenClaw bot 身份運作。這導致：
- 無法在同一個 channel 內多 agent 對話
- Jones 需要切換多個 Workspace
- 無法讓 agent 之間直接互動

### 平台比較（多 agent 共存）
| 平台 | 多 bot 共存 | Threading | 訊息穩定性 | 結論 |
|------|-----------|-----------|-----------|------|
| Slack | ✅ 同 Workspace 多 App | ✅ | ✅ | 🥇 最推薦 |
| Telegram | ✅ 群組多 bot | ❌ | ✅ | 🥈 備選 |
| LINE | ❌ 免費方案不允許同群組多 bot | ❌ | ⚠️ 會丟訊息 | ❌ 不適合 |
| Discord | ✅ 多 bot | ✅ Forum channels | ✅ | 可考慮 |

### LINE 群組實測結果（2026-02-08）
- 兩個 bot 同時加入群組 → 互相被踢出
- 單獨一個 bot 加入 → 正常
- LINE Developers Console + Official Account Manager 設定皆正確
- 結論：LINE Communication plan 不支援同群組多 Official Account bot

## 方案：統一 Workspace + 多 Slack App

### 架構
```
JoviAgents Workspace
├── #general          — 日常對話
├── #jovi-agents      — 三方協作
├── #jarvis-tasks     — Jarvis 專屬
├── #juni-tasks       — Juni 專屬
└── #jovix-tasks      — Jovix 專屬

Slack Apps (各自獨立):
├── Jarvis App  → Bot Token A → Mac Mini OpenClaw
├── Juni App    → Bot Token B → Mac Pro OpenClaw
└── Jovix App   → Bot Token C → PC OpenClaw (未來)
```

### 設定步驟
1. 建立新 Slack Workspace（`JoviAgents` 或類似名稱）
2. 到 [api.slack.com/apps](https://api.slack.com/apps) 建三個 Slack App
   - 各自命名：Jarvis、Juni、Jovix
   - 各自設定不同頭像
3. 三個 App 都安裝到同一個 Workspace
4. 各 agent 的 `openclaw.json` 填入對應的 Bot Token + App Token
5. 測試多 agent 在同一 channel 對話

### 注意事項
- 每個 App 需要獨立的 Bot Token 和 App Token
- Jones 用同一個 Slack 帳號管理所有 App
- 現有的各自 Workspace 可保留（備用）或日後移除

## 相關
- [OpenClaw LINE Setup](../openclaw-line-setup/) — LINE 串接紀錄
- [Jovix Gateway](../jovix-gateway/) — Jovix 架構規劃

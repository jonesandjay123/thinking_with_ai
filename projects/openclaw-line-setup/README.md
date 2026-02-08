# OpenClaw × LINE × Tailscale 串接指南

> **完成日期：** 2026-02-07
> **目的：** 將 LINE Messaging API 作為 OpenClaw AI Agent 的聊天頻道
> **適用環境：** macOS + OpenClaw (npm gateway) + Tailscale Funnel

## 架構總覽

```
LINE App (使用者) 
  → LINE Platform (webhook POST)
    → Tailscale Funnel (公網 HTTPS)
      → localhost:18789 (OpenClaw Gateway)
        → AI Agent (Claude)
          → LINE Reply API → 使用者
```

LINE 需要一個 **公網 HTTPS URL** 來發送 webhook 事件。我們用 **Tailscale Funnel** 把本機的 OpenClaw gateway 暴露到公網，免費且不需要固定 IP。

## 目錄

1. [前置準備](#1-前置準備)
2. [LINE 商用帳號設定](#2-line-商用帳號設定)
3. [Tailscale 安裝與設定](#3-tailscale-安裝與設定)
4. [OpenClaw LINE Plugin 設定](#4-openclaw-line-plugin-設定)
5. [串接驗證與測試](#5-串接驗證與測試)
6. [開機自啟與維護](#6-開機自啟與維護)
7. [疑難排解](#7-疑難排解)

---

## 1. 前置準備

### 需要的東西
- [x] macOS 電腦（24/7 運行）
- [x] OpenClaw gateway 已安裝並運行（`npm install -g openclaw`）
- [x] 至少一個其他頻道（如 Telegram/Slack）已正常運作
- [x] LINE 帳號（個人帳號即可）

### 名詞解釋
| 名詞 | 說明 |
|------|------|
| **OpenClaw** | AI Agent 的 gateway 服務，處理多頻道訊息路由 |
| **LINE Messaging API** | LINE 提供的 Bot 開發 API |
| **Tailscale** | 零設定 VPN + Funnel 功能可把本機服務暴露到公網 |
| **Webhook** | LINE 主動把訊息 POST 到你指定的 URL |

---

## 2. LINE 商用帳號設定

### 2.1 建立 LINE Developers 帳號

1. 前往 [LINE Developers Console](https://developers.line.biz/console/)
2. 用你的 LINE 帳號登入
3. 同意開發者條款

### 2.2 建立 Provider

1. 點 **Create** 建立新的 Provider
2. 填入 Provider 名稱（例如：`My AI Agent`）

### 2.3 建立 Messaging API Channel

1. 在 Provider 下點 **Create a new channel**
2. 選擇 **Messaging API**
3. 填寫必要資訊：
   - **Channel name**: Bot 的名稱（例如：`Jarvis`）
   - **Channel description**: 簡短描述
   - **Category / Subcategory**: 選擇最接近的類別
4. 同意條款，點 **Create**

### 2.4 取得憑證

在 Channel 設定頁面取得兩個重要值：

#### Channel Secret
- 位置：**Basic settings** → **Channel secret**
- 直接複製

#### Channel Access Token (Long-lived)
- 位置：**Messaging API** tab → **Channel access token**
- 點 **Issue** 生成（如果還沒有的話）
- 複製完整的 token

> ⚠️ **安全提醒：** 這兩個值等同於密碼，不要公開或 commit 到公開 repo。

### 2.5 設定 LINE Official Account Manager

LINE Developers 建好 Channel 後，還需要在 [LINE Official Account Manager](https://manager.line.biz/) 調整設定：

1. 進入你的 Bot 帳號
2. **回應設定** → **聊天的回應方式**：
   - 回應時間：選 **手動聊天**（不要選「手動聊天＋自動回應訊息」）
3. **加入好友的歡迎訊息**：
   - 設為 **停用**（Disabled）

> 如果「自動回應訊息」開著，LINE 會自己回覆而不把訊息送到 webhook！

---

## 3. Tailscale 安裝與設定

### 3.1 安裝 Tailscale

#### macOS
1. 從 [Mac App Store](https://apps.apple.com/app/tailscale/id1475387142) 或 [官網](https://tailscale.com/download/mac) 下載
2. 安裝後開啟，用 Google/GitHub/等帳號登入
3. 登入後 menu bar 會出現 Tailscale 圖示

### 3.2 確認連線

```bash
# macOS Tailscale CLI 路徑
/Applications/Tailscale.app/Contents/MacOS/Tailscale status
```

你會看到類似：
```
100.88.29.83   jarvismac-mini  user@email  macOS  -
```

### 3.3 啟用 HTTPS

1. 前往 [Tailscale Admin Console - DNS](https://login.tailscale.com/admin/dns)
2. 捲到最下面，找到 **HTTPS Certificates**
3. 點 **Enable HTTPS...**

### 3.4 啟用 Funnel 權限

1. 前往 [Tailscale Admin Console - Access Controls](https://login.tailscale.com/admin/acls)
2. 點 **Node attributes** 分頁
3. 新增一個 Node attribute：
   - **Targets**: All users and devices
   - **Attributes**: `funnel`
4. 儲存

或者用 **JSON editor**，在 ACL 中加入：

```json
{
  "nodeAttrs": [
    {
      "target": ["*"],
      "attr": ["funnel"]
    }
  ]
}
```

> ⚠️ **踩坑經驗：** 如果之前裝過又移除 Tailscale，可能有衝突。需要用 force 指令重啟：
> ```bash
> /Applications/Tailscale.app/Contents/MacOS/Tailscale up --reset
> ```

### 3.5 啟動 Funnel

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale funnel --bg 18789
```

成功會顯示：
```
Available on the internet:

https://YOUR-HOSTNAME.tailXXXXX.ts.net/
|-- proxy http://127.0.0.1:18789

Funnel started and running in the background.
```

### 3.6 驗證外部可達

```bash
curl -s -o /dev/null -w "%{http_code}" https://YOUR-HOSTNAME.tailXXXXX.ts.net/line/webhook
# 應該回 200
```

---

## 4. OpenClaw LINE Plugin 設定

### 4.1 安裝 LINE Plugin

```bash
openclaw plugins install @openclaw/line
```

確認安裝成功：
```bash
openclaw plugins list
```

應該看到 LINE plugin 狀態為 `loaded`。

### 4.2 設定 Config

編輯 `~/.openclaw/openclaw.json`，在 `channels` 中加入：

```json
{
  "channels": {
    "line": {
      "enabled": true,
      "channelAccessToken": "YOUR_CHANNEL_ACCESS_TOKEN",
      "channelSecret": "YOUR_CHANNEL_SECRET",
      "dmPolicy": "pairing"
    }
  }
}
```

同時在 `plugins.entries` 中加入：

```json
{
  "plugins": {
    "entries": {
      "line": {
        "enabled": true
      }
    }
  }
}
```

### 4.3 dmPolicy 選項

| Policy | 行為 |
|--------|------|
| `pairing` | 新用戶需要 pairing code 配對（推薦） |
| `allowlist` | 只允許白名單中的 LINE User ID |
| `open` | 任何人都可以對話（不建議） |
| `disabled` | 停用 DM |

### 4.4 groupPolicy 選項（群組聊天）

如果要讓 Bot 加入 LINE 群組，需要設定 `groupPolicy`：

```json
{
  "channels": {
    "line": {
      "enabled": true,
      "channelAccessToken": "YOUR_TOKEN",
      "channelSecret": "YOUR_SECRET",
      "dmPolicy": "pairing",
      "groupPolicy": "open"
    }
  }
}
```

| Policy | 行為 |
|--------|------|
| `open` | Bot 可以加入任何群組 |
| `allowlist` | 只允許白名單中的群組（用 `groupAllowFrom` 指定 Group ID） |
| `disabled` | Bot 不加入群組（預設） |

> ⚠️ **重要：** 如果沒有設定 `groupPolicy`，預設是 `disabled`。Bot 被拉進群組後會**立刻自動退出**（秒退），這是 OpenClaw 的安全機制。必須明確設定 `groupPolicy` 才能留在群組中。

### 4.5 重啟 Gateway

```bash
openclaw gateway restart
```

確認 log 中出現：
```
[line] [default] starting LINE provider (YourBotName)
```

---

## 5. 串接驗證與測試

### 5.1 設定 Webhook URL

1. 回到 [LINE Developers Console](https://developers.line.biz/console/)
2. 進入你的 Messaging API Channel
3. **Messaging API** tab → **Webhook settings**
4. Webhook URL 填入：
   ```
   https://YOUR-HOSTNAME.tailXXXXX.ts.net/line/webhook
   ```
5. 點 **Verify** — 應該顯示 **Success**
6. 確認 **Use webhook** 已開啟

### 5.2 配對 (Pairing)

1. 在 LINE 中加 Bot 為好友（掃 QR code 或搜尋 Bot ID）
2. 傳一則訊息給 Bot
3. Bot 會回覆一組 pairing code
4. 在 Terminal 執行：
   ```bash
   openclaw pairing approve line CODE
   ```
5. 再傳一次訊息 — 這次 AI Agent 應該會回覆！

### 5.3 驗證成功的 Log

```
[line] [default] starting LINE provider (Jarvis)
```

Gateway log 位置：`~/.openclaw/logs/gateway.log`

---

## 6. 開機自啟與維護

### 自動啟動狀態

| 服務 | 開機自啟 | 備註 |
|------|---------|------|
| Tailscale App | ✅ 自動 | macOS App 預設行為 |
| Tailscale Funnel | ✅ 自動 | `--bg` 模式，config 持久化 |
| OpenClaw Gateway | ✅ 自動 | LaunchAgent 管理 |

正常情況下，重開機後三個服務都會自動啟動，LINE 頻道自動上線。

### 手動操作（需要時）

```bash
# 檢查 Tailscale Funnel 狀態
/Applications/Tailscale.app/Contents/MacOS/Tailscale funnel status

# 重啟 Funnel（如果斷了）
/Applications/Tailscale.app/Contents/MacOS/Tailscale funnel --bg 18789

# 重啟 OpenClaw Gateway
openclaw gateway restart

# 查看 Gateway log
tail -f ~/.openclaw/logs/gateway.log
```

---

## 7. 疑難排解

### Webhook Verify 失敗：405 Method Not Allowed
**原因：** LINE plugin 還沒啟用或 gateway 沒重啟
**解法：** 確認 config 中 LINE 設定正確，重啟 gateway

### Webhook Verify 失敗：連線逾時
**原因：** Tailscale Funnel 沒啟動或外部無法連入
**解法：**
```bash
# 確認 Funnel 狀態
/Applications/Tailscale.app/Contents/MacOS/Tailscale funnel status

# 重新啟動
/Applications/Tailscale.app/Contents/MacOS/Tailscale funnel --bg 18789
```

### Funnel 啟動失敗：Funnel is not enabled
**原因：** Tailnet ACL 沒有加 funnel 權限
**解法：** 到 Tailscale Admin Console → Access Controls → Node attributes 加 funnel attr

### Verify 成功但收不到訊息
**原因：** LINE Official Account Manager 的自動回應沒關
**解法：** 到 LINE Official Account Manager → 回應設定 → 選「手動聊天」

### openclaw 指令不存在或跑到舊版

> ⚠️ **歷史坑：** 如果之前裝過 `clawdbot` 或 `moltbot`（OpenClaw 的舊名），`.zshrc` 可能有 alias 把 `openclaw` 指向舊版。

**檢查：**
```bash
which openclaw
openclaw --version
grep "alias.*claw\|alias.*molt" ~/.zshrc
```

**解法：**
```bash
# 移除舊版
npm uninstall -g clawdbot moltbot

# 移除 .zshrc 中的舊 alias
# 刪掉類似這些行：
# alias openclaw=clawdbot
# alias moltbot=clawdbot
```

### Gateway restart 後 Tailscale Funnel 斷了
**原因：** Foreground serve session 被中斷
**解法：** 重新啟動 Funnel：
```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale funnel --bg 18789
```

---

## 附錄：完整 Config 範例

```json
{
  "channels": {
    "line": {
      "enabled": true,
      "channelAccessToken": "YOUR_TOKEN_HERE",
      "channelSecret": "YOUR_SECRET_HERE",
      "dmPolicy": "pairing"
    }
  },
  "plugins": {
    "entries": {
      "line": {
        "enabled": true
      }
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback"
  }
}
```

## 參考連結

- [OpenClaw LINE Plugin 文件](https://docs.openclaw.ai)
- [LINE Developers Console](https://developers.line.biz/console/)
- [LINE Official Account Manager](https://manager.line.biz/)
- [Tailscale Funnel 文件](https://tailscale.com/kb/1223/funnel)
- [Tailscale Admin Console](https://login.tailscale.com/admin)

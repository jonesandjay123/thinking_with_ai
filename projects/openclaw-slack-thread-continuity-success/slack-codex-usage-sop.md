# SOP: 在 Slack 建立 `/codex-usage`，直接查 Codex 剩餘用量（Socket Mode）

> 目標：讓未來的 agent 或操作者照著做一次，就能在 Slack 中使用 `/codex-usage`，且不經主 agent / LLM。

---

## 0. 先確認你要走哪一種 Slack 架構

先去 Slack App 後台看：

- 如果 **Socket Mode = enabled**，照本 SOP 做。
- 如果你看到自己必須填 **Request URL**，那是 webhook 模式，這份 SOP 不適用。

**判斷原則：**
- Socket Mode → 不需要 Request URL
- Webhook mode → 需要公開 HTTPS Request URL

---

## 1. 在 Slack API 後台手動完成的步驟

進入：
- <https://api.slack.com/>

打開目標 Slack App，然後確認以下項目。

### 1.1 啟用 Socket Mode

確認：
- Socket Mode = enabled

### 1.2 準備兩種 token

你需要：
- **Bot Token**，格式通常像 `xoxb-...`
- **App-Level Token**，格式通常像 `xapp-...`

> 不要把真實 token 寫進筆記、repo、或公開文件。

### 1.3 建立 slash command

在 Slash Commands 頁面新增：
- Command: `/codex-usage`
- Description: 任意，例如 `Show remaining Codex usage`
- Usage Hint: 可留空

**注意：**
如果 Socket Mode 已啟用，這裡通常 **不用填 Request URL**。

### 1.4 重新安裝 App 到 workspace

如果你新增了 command、調整 token、調整 scopes，請重新安裝 App。

---

## 2. 在本機建立獨立 Python 環境

不要把套件裝進 OpenClaw 核心，也不要硬裝進 system Python。

### 2.1 建立 virtualenv

```bash
cd /path/to/your/workspace
python3 -m venv .venvs/slack-codex-usage
```

### 2.2 安裝所需套件

```bash
.venvs/slack-codex-usage/bin/pip install slack-bolt slack-sdk
```

如果 macOS / Homebrew Python 擋你直接 pip install，這是正常的。用 venv 就好。

---

## 3. 先做一支獨立的 usage 查詢腳本

不要先做 Slack listener。
先做一支能獨立測通的查詢腳本。

### 3.1 這支 script 的責任

它只負責：
1. 從本機 auth source 拿到 Codex/OpenAI access token
2. 呼叫 usage endpoint
3. 解析剩餘用量
4. 輸出文字 / JSON / Slack-friendly 文字

### 3.2 建議能力

至少支援：
- 預設純文字輸出
- `--json`
- `--slack`

### 3.3 驗證標準

先在 terminal 直接跑成功再往下走，例如：

```bash
python3 scripts/codex_usage_status.py --slack
```

如果這步還沒通，先不要碰 Slack integration。

---

## 4. 再做 Slack Socket Mode listener

listener 要夠薄，不要把 usage API 細節寫在裡面。

### 4.1 listener 的責任

它只做這幾件事：
1. 連上 Slack Socket Mode
2. 註冊 `/codex-usage`
3. 收到 command 後先 `ack()`
4. 呼叫本地 usage script
5. 用 ephemeral message 回 Slack

### 4.2 listener 需要的環境變數

至少：

```bash
SLACK_BOT_TOKEN=...
SLACK_APP_TOKEN=...
```

可選：

```bash
CODEX_USAGE_COMMAND=/codex-usage
CODEX_USAGE_SCRIPT=/absolute/path/to/codex_usage_status.py
CODEX_USAGE_PYTHON=/absolute/path/to/venv/bin/python
```

### 4.3 啟動方式

例如：

```bash
SLACK_BOT_TOKEN='...' \
SLACK_APP_TOKEN='...' \
CODEX_USAGE_PYTHON='/absolute/path/to/.venvs/slack-codex-usage/bin/python' \
/absolute/path/to/.venvs/slack-codex-usage/bin/python /absolute/path/to/scripts/slack_codex_usage_socket.py
```

---

## 5. 先做短測，再改成常駐

### 5.1 先確認 listener 能啟動

啟動後應看到類似：

```text
Starting Slack Socket Mode listener for /codex-usage
```

### 5.2 去 Slack 測 `/codex-usage`

如果成功，你應看到：
- ephemeral 回覆
- 顯示剩餘用量
- 不經主 agent / LLM

---

## 6. 如果 Slack 說「應用程式未回應」，先檢查這個

**第一優先不是懷疑 token。**

先檢查：
- listener process 還活著嗎？

這次實作裡，最真實的坑就是：
- listener 其實能啟動
- token 也其實是對的
- 但它只是掛在暫時測試 session 上
- 測完 process 被回收
- 真正送 slash command 時，沒有人在線接事件

所以 Slack 才會顯示：
- App didn’t respond
- slash command failed

### 正確修法

把 listener 變成真正背景常駐程序。

短期可先：

```bash
nohup ... > logs/slack-codex-usage.log 2>&1 &
```

長期建議：
- `launchd`
- systemd
- 或其他正式 process manager

---

## 7. 驗證成功的標準

在 Slack 輸入：

```text
/codex-usage
```

成功時你應看到：
- 一則 ephemeral 回覆
- 直接顯示 Codex 剩餘用量
- 不需要主 agent 被喚醒
- 不消耗 LLM 對話回合

---

## 8. 這樣設計是否正確

結論：**是，這是正確的 v1 設計。**

理由：
- utility command 跟聊天 agent 分離
- usage logic 跟 Slack integration 分離
- 依賴隔離在專用 venv
- 不污染 OpenClaw 本體
- 可重建、可攜、可維護

---

## 9. 一句話交接

如果你是下一個接手的 agent，記住這句話：

> 這不是「教主 agent 怎麼回答 slash command」，而是「額外做一條 Slack integration side path，直接跑本地 usage script」。

先把 script 單獨做好，再接 Socket Mode listener，再讓 listener 常駐。這樣最快，也最穩。

# Slack `/codex-usage` Slash Command 實作筆記（Socket Mode, 無 LLM）

> 建立日期：2026-04-18  
> 目的：記錄如何在 Slack 中建立一個幾乎不經過 agent/LLM 的 `/codex-usage` 指令，直接回報 Codex 剩餘用量。  
> 安全原則：本文件刻意不記錄任何實際 token、secret、個人 workspace id、或可直接重放的敏感資訊。

---

## TL;DR

這套做法的核心不是「讓 Jarvis 回答一個問題」，而是把查詢剩餘用量這件事拆成一條獨立輕量路徑：

1. Slack 收到 `/codex-usage`
2. Socket Mode listener 直接接住這個 command
3. listener 呼叫本地 Python 腳本
4. Python 腳本直接查 Codex usage API
5. Slack 回傳 ephemeral 結果

**重點：**
- 不需要喚醒主 agent
- 不需要打 LLM
- 不需要走 OpenClaw 對話回合
- 延遲低，token 成本幾乎為零

---

## 一、這個方案要解決什麼

原始需求不是「在 Slack 問 Jarvis 剩餘用量」，而是：

> 希望在 Slack 裡打一個 slash command，就能像系統工具一樣，直接看到 Codex 剩餘額度，不要多走一層 LLM/agent 對話。

這代表需求本質上比較像：
- lightweight utility
- internal ops command
- direct status lookup

而不是一般聊天型 agent 任務。

所以最合理的設計不是 prompt engineering，而是**額外拉一條輕量 side path**。

---

## 二、整體架構

### 1. 查詢腳本層

一支獨立 Python 腳本，負責：
- 讀取本機 Codex/OpenAI OAuth auth source
- 呼叫 usage endpoint
- 輸出簡潔文字 / Slack-friendly 文字 / JSON

在這次實作中，腳本位置為：

```text
scripts/codex_usage_status.py
```

這層不應該依賴 Slack。

### 2. Slack command listener 層

另一支獨立 Python 腳本，負責：
- 接 Slack slash command
- 執行 usage 查詢腳本
- 將結果用 ephemeral message 回 Slack

在這次實作中，listener 位置為：

```text
scripts/slack_codex_usage_socket.py
```

這層不應該自己懂 usage API 細節，只要當橋接器。

### 3. 執行環境隔離

不要把額外 Slack Python 套件裝進 OpenClaw 本體，也不要污染系統 Python。

建議使用專用 virtualenv，例如：

```text
.venvs/slack-codex-usage
```

這樣做的好處：
- 依賴獨立
- 容易重建
- 容易遷移
- 不影響 OpenClaw 本體
- 不碰 Homebrew/system Python 的管理邊界

**結論：這是一個合理且正確的 v1 架構。**

---

## 三、為什麼選 Socket Mode，不選 Request URL / webhook

這次 Slack App 畫面已經明確顯示：

- Socket Mode enabled
- 不需要 Request URL

所以正確理解是：

### Webhook / Request URL 模式
Slack 會把 slash command 直接 POST 到你的一個公開 HTTPS URL。

你需要：
- 對外可連的網址
- TLS/HTTPS
- 反向代理或 tunnel（例如 Cloudflare Tunnel / ngrok）
- Slack signature 驗證

### Socket Mode 模式
你的程式主動連到 Slack，Slack 再把事件推給你。

你需要：
- bot token
- app-level token
- 一個持續在線的 listener process
- 不需要公開 Request URL

### 這次為什麼 Socket Mode 比較適合
因為需求只是內部小工具，而且現有 Slack App 已開了 Socket Mode。

所以沿用 Socket Mode 的好處是：
- 少一層網路公開面
- 少一個 public URL 維護點
- 更快驗證
- 更符合現況

---

## 四、使用者在 Slack API 網站需要手動做的事

以下步驟是給人類操作者的，未來 agent 若要交接，也應該要求使用者完成這些。

### Step 1. 到 Slack API 建立或打開既有 App

進入：

- <https://api.slack.com/>

找到你的 Slack App。

### Step 2. 啟用 Socket Mode

在 App 設定頁確認：
- Socket Mode = enabled

如果沒開，就先開啟。

### Step 3. 建立 App-Level Token

在 Slack API 後台建立一個 app-level token，供 Socket Mode 使用。

特徵通常像：
- `xapp-...`

常見需求是給它 Socket Mode 需要的權限（例如 connections 類型權限，依 Slack 當前 UI 指引）。

### Step 4. 確認 Bot Token

在 OAuth / tokens 相關頁面確認 bot token。

特徵通常像：
- `xoxb-...`

### Step 5. 建立 Slash Command

在 Slash Commands 頁面新增：

- Command: `/codex-usage`
- Short description: 可填例如「Show remaining Codex usage」
- Usage hint: 可填可不填

**如果 Socket Mode 已啟用，這裡通常不需要填 Request URL。**

這點非常重要，因為很多教學預設是 webhook 模式，容易誤導。

### Step 6. 安裝或重新安裝 App 到 workspace

如果你新增了 command、scope、token 或 Socket Mode 設定，通常要重新安裝 App 到 workspace。

沒有這步，常見結果是：
- slash command 看得到
- 但事件不會正確送進 listener

---

## 五、本機端需要準備什麼

### 1. Python virtualenv

建立專用環境，例如：

```bash
python3 -m venv .venvs/slack-codex-usage
```

### 2. 安裝套件

以這次實作來說，需要：

```bash
pip install slack-bolt slack-sdk
```

如果系統 Python 受 Homebrew / PEP 668 管理，請不要硬裝到系統環境。用 venv 即可。

### 3. 準備環境變數

至少需要：

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

---

## 六、查詢腳本應該長什麼樣

查詢腳本的責任應該很單純：

1. 從本機可用 auth source 取得 Codex/OpenAI OAuth token
2. 呼叫 usage endpoint
3. 解析 rate limit windows
4. 輸出適合 CLI / Slack 的格式

### 設計原則

- 盡量不要直接耦合 Slack
- 支援純文字輸出
- 最好順手支援 `--json`
- 最好再加一個 `--slack`

### 為什麼要先把這層獨立出來

因為這樣：
- 方便單獨測試
- listener 壞了時可分層排查
- 未來也能被別的工具重用
- agent、cron、CLI、Slack 都能共用

---

## 七、Socket Mode listener 應該長什麼樣

listener 的責任只應該是：

1. 連上 Slack Socket Mode
2. 註冊 `/codex-usage`
3. 收到 command 先 `ack()`
4. 執行 usage script
5. 用 `respond(response_type='ephemeral', text=...)` 回覆

### 設計原則

- 不要把 usage API 邏輯塞在 listener 裡
- 不要把 OAuth token 寫死在程式裡
- 不要耦合 OpenClaw 對話 runtime
- 把它當成一個小型 integration service

---

## 八、這次實作時實際踩到的坑

### 坑 1. 一開始誤以為要填 Request URL

其實那是 webhook 模式的做法。

但這個 Slack App 已開 Socket Mode，所以：
- 不需要 Request URL
- 應改做 Socket Mode listener

### 坑 2. 本機沒有 Slack Python 套件

本機最初缺：
- `slack_sdk`
- `slack_bolt`

解法：
- 建專用 venv
- 裝到 venv 裡

### 坑 3. listener 只被短暫測試啟動，沒有常駐

這是最關鍵的實際問題。

一開始 listener 確實能啟動，但它是掛在暫時測試 session 上。測試結束後 process 被回收，導致：

- Slack slash command 已成功建立
- token 其實也有效
- 但真正送 command 進來時，已經沒有活著的 listener 在接

表面症狀就會是：
- Slackbot 說應用程式未回應
- slash command 執行失敗

**真正原因不是 Slack 設定錯，而是 background process 沒活著。**

### 解法

把 listener 以真正背景常駐方式跑起來，並保留 log。

在這次操作中，先用最小方式解決：
- `nohup ... &`
- stdout/stderr 導到 log

長期來看，更好的做法是用 `launchd` 或其他正式 process manager。

---

## 九、怎樣判斷成功

成功的觀察點很簡單：

在 Slack 輸入：

```text
/codex-usage
```

如果成功，應看到：
- 一則 ephemeral 回覆
- 內容是 Codex 剩餘用量
- 不經過主 agent 對話
- 不需要 LLM 推理

如果失敗，優先檢查：

1. listener process 是否真的還活著
2. bot token / app token 是否正確
3. App 是否已重新安裝到 workspace
4. slash command 是否真的叫 `/codex-usage`
5. 本地 usage script 是否可單獨執行

---

## 十、這個架構是否合理

結論：**合理，而且是正確方向的 v1。**

### 為什麼合理

因為它做到幾件很重要的事：

- 把 utility command 跟聊天型 agent 分離
- 把 Slack integration 跟 usage logic 分離
- 把依賴裝在獨立 venv，不污染主系統
- 把路徑做成可重建、可攜、可維護

### 之後可再升級的地方

1. 不要只靠臨時 shell env，改成更正式的 secret 管理
2. 不要只靠 `nohup`，改成 `launchd`
3. 加更明確的 log 與錯誤處理
4. 若 command 變多，可把 listener 抽成小型 integration service

但在 v1 階段，這個設計已經足夠乾淨，也很實用。

---

## 十一、給未來 agent 的一句話

如果你要幫別人重建這套流程，正確順序不是先搞 prompt，也不是先想怎麼讓主 agent「回答得更像系統指令」。

正確順序是：

1. 把查詢邏輯獨立成 script
2. 判斷 Slack App 是 Socket Mode 還是 webhook mode
3. 根據模式做 listener
4. 把 listener 長期常駐
5. 最後才調整格式與使用體驗

**把它當成 integration engineering，不要當成 prompt engineering。**

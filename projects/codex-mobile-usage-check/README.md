# Codex 流量剩餘數，手機上怎麼查？

> **建立日期：** 2026-04-15
> **目的：** 研究當使用 OpenClaw + OpenAI Codex OAuth 時，如果手邊沒有電腦，手機上有哪些方法可以查詢 usage window / 流量剩餘數。
> **結論先講：** 最實際的做法不是找「神奇外掛」，而是把 OpenClaw 已經有的 usage surface，包成更適合手機使用的入口。

## 問題定義

Jones 現在的需求不是「電腦上怎麼看」，而是：

- 人在外面，只有手機
- 想快速知道 Codex 的 quota / usage window 還剩多少
- 最好能在 Slack 之類的既有聊天入口完成
- 最好不要每次都靠 agent 正常對話查詢

這個需求其實可以拆成兩層：

1. **資料來源在哪裡？**
2. **手機入口怎麼做最順手？**

OpenClaw 官方文件已經說得很清楚：usage tracking 是直接拉 provider 的 usage/quota endpoint，並透過 `/status`、`session_status`、CLI 與部分 UI surface 顯示。換句話說，**難點不是資料本身，而是把它包成手機好用的入口。**

## 官方已知能力

根據 OpenClaw 文件：

- `/status` 會顯示 runtime status，包含 provider usage/quota（如果 provider 支援）
- `session_status` 是 `/status` 的工具等價物
- CLI 也支援 `openclaw status --usage`
- 使用量資料是從 provider usage endpoint 直接取得，不是 agent 自己亂估
- OpenAI Codex OAuth 屬於支援 usage tracking 的 provider 類型之一

這代表：

**你現在其實已經有可靠的 usage backend，只差一個好入口。**

## 我在網路上看到的幾種做法

這次查到的公開資訊，沒有看到一個「大家都裝同一個手機查 quota 外掛」的主流解法。比較像是大家用幾種思路自己接：

### 1. 直接用 OpenClaw 內建 `/status`

**做法**
- 在聊天裡直接送 `/status`
- 由 Gateway 回傳目前 session 狀態與 usage window

**優點**
- 原生
- 最少工程量
- 跟 OpenClaw 官方設計一致

**缺點**
- 還是經過 agent / gateway 的聊天命令流程
- 手機上雖然能用，但不一定是最俐落的 UX
- 如果你想要的是「完全不是聊天、不佔 agent 對話存在感」，這不夠乾淨

**適合誰**
- 先求可用，不急著做客製化的人

---

### 2. Slack slash command，像 `/jarvis-status`

**做法**
- 在 Slack App 建一個 slash command
- 讓它直接打到一個很輕量的 status handler
- handler 再去讀 OpenClaw usage surface，回傳簡短文字給 Slack

**優點**
- 最符合「手機上自助查詢」
- 在 Slack 手機版操作很自然
- 可以做成固定輸出，例如：
  - `GPT-5.4 / Codex: 5h 95% left, week 99% left`
- 可避免把一般聊天上下文搞得很雜

**缺點**
- 需要做 Slack app 配置
- Slack command 仍然是經過 OpenClaw/Gateway 或你自建的小 handler，不是憑空零成本
- 要處理授權與暴露面，避免任何人都能查

**適合誰**
- 最在意手機 UX
- 已經重度用 Slack 當控制面板的人

**我的判斷**
- 這是 *最值得實作的正式方案*

---

### 3. 做一個超輕量 webhook / status endpoint

**做法**
- 在本機或 Gateway 旁邊加一個小服務
- 提供一個只做一件事的 endpoint，例如：
  - `/codex-usage`
- 它只回一小段 JSON 或純文字
- 手機端可透過：
  - Siri Shortcut
  - iOS Shortcuts
  - 自己書籤
  - Slack workflow
  - Telegram bot command
  來呼叫

**優點**
- 非常靈活
- 可以真正做成「查 usage 專用」
- 可串手機快捷操作，甚至一鍵彈通知

**缺點**
- 需要自己補一層服務
- 需要處理 auth token / allowlist / Tailscale / reverse proxy
- 維護面比 slash command 高一點

**適合誰**
- 想做自己的一套 mobile control plane
- 後面還想順便加更多自助功能（如 restart、model、queue 狀態）的人

**我的判斷**
- 這是 *最可擴充的工程方案*

---

### 4. Control UI / 手機瀏覽器直接看

**做法**
- 用手機 Safari / Chrome 打開 OpenClaw Control UI
- 在 UI 裡看 session 或 usage

**優點**
- 不一定要寫任何新東西
- 如果 UI 本身已經能看 usage，馬上可用

**缺點**
- 手機 UX 未必最好
- 可能比單純查數字重
- 需要你願意把 Control UI 做好遠端存取與登入保護

**適合誰**
- 偶爾查一下的人
- 不想先做開發的人

**我的判斷**
- 這是 *最快速的零開發備案*

---

### 5. 被動提醒，不主動查

**做法**
- 不是讓人查，而是接近閾值自動通知
- 例如每次低於 20% 時 Slack DM 一次

**優點**
- 最省腦
- 最符合「我不想一直查」
- 對手機使用者其實很友好

**缺點**
- 無法替代「我現在就想知道」的即時查詢
- 仍需要背景檢查機制

**適合誰**
- 只在乎不要撞牆的人

**我的判斷**
- 這應該和主動查詢並存，不該單獨存在

## 社群創意方向總結

目前公開可見的思路，不太像「裝一個現成外掛」；比較像這幾類創意組合：

1. **把 `/status` 當成命令入口**
2. **把 Slack/Telegram/Discord 當成手機控制台**
3. **把 usage 做成單功能 webhook**
4. **搭配 Tailscale / private URL / Shortcuts 變成手機一鍵查詢**
5. **主動告警，減少主動查詢次數**

換句話說，社群最常見的創意不是「找外掛」，而是：

> **把原本存在於桌面 CLI 或 agent 工具層的資訊，包裝成手機可觸發的一步操作。**

## 手機場景的可行方案比較

### 方案 A，最小可用：直接在 Slack 傳 `/status`

**體感**
- 幾乎不用開發
- 立刻可試

**缺點**
- 還是比較像在跟 agent 說話

**推薦度**
- 短期：高
- 長期：中

---

### 方案 B，最佳平衡：Slack slash command

**範例**
- `/codex-usage`
- `/jarvis-status`

**回傳格式建議**
```text
Codex usage
- 5h window: 95% left
- Weekly: 99% left
- Model: GPT-5.4
- Updated: 22:06 EDT
```

**優點**
- 手機上最自然
- 不用打開電腦
- 操作成本低

**推薦度**
- 短期：中
- 長期：很高

---

### 方案 C，最終型：iPhone Shortcut + private endpoint

**流程**
- 手機點一下 Shortcut
- Shortcut 打到 private endpoint
- endpoint 回 usage
- iPhone 直接顯示結果或播報

**可以長成這樣**
- 主畫面按鈕：`Check Codex`
- Siri：`Hey Siri, check Codex usage`

**優點**
- 真正 mobile-native
- 最爽
- 未來可延伸更多 agent control

**缺點**
- 要自己做

**推薦度**
- 長期：最高

## 我給 Jones 的建議

### 如果你要「今晚就能用」
先做：
1. 驗證 Slack 裡 `/status` 的體驗
2. 如果可以接受，就先拿它當過渡方案

### 如果你要「最像產品」
做：
1. **Slack slash command** 作為第一版
2. 輸出固定為精簡 usage 卡片
3. 後面再加 **被動提醒**

### 如果你要「最終型手機控制面板」
做：
1. 一個 private usage endpoint
2. 一個 iPhone Shortcut
3. Slack command 當備援入口

## 實作藍圖

### Version 1: Slack 手機自助查詢
- Slack command: `/codex-usage`
- Gateway / 小 handler 取得 usage
- 回傳簡短文字
- 僅 allowlist 指定 user 可用

### Version 2: 主動提醒
- 每隔固定時間檢查 usage
- 低於閾值才發 Slack DM
- 避免 spam

### Version 3: iPhone Shortcut
- 以 Tailscale / 私有網址連到 usage endpoint
- 一鍵查詢
- 可支援 Siri 語音觸發

## 關鍵觀察

1. **OpenClaw 已有 usage backend，核心不是缺資料。**
2. **手機查詢的價值在 UX 包裝，不在模型能力。**
3. **最值得做的不是「問 agent」，而是「做一個狹義單功能入口」。**
4. **Slack slash command 是最好的第一個產品化切口。**
5. **iPhone Shortcut + private endpoint 是最終形態。**

## 後續可執行項目

1. 確認 Slack provider 目前是否已配置 native slash command 路徑
2. 查 OpenClaw 現有 command surface 能否直接滿足 `/status` 手機使用
3. 若不夠，實作 `/codex-usage` 專用入口
4. 之後再補 threshold notification

## 參考線索

### OpenClaw docs
- `docs/concepts/usage-tracking.md`
  - 說明 usage 直接來自 provider usage/quota endpoint
  - `/status`、`session_status`、CLI 都可顯示 usage
- `docs/tools/slash-commands`
  - 說明 `/status` 是 Gateway 處理的命令
  - Slack 需要自行建立 slash command 時，屬於可延伸方向

### 這次 research 的重點結論
- 沒有看到一個廣泛共識的「手機查 Codex quota 標配外掛」
- 大多數可行創意都是：
  - 利用內建 `/status`
  - 做 Slack/Telegram 命令入口
  - 做 private webhook / endpoint
  - 接到手機 shortcuts 或通知

---

如果要把這份報告延伸成實作，我會優先做：

**第一步：Slack `/codex-usage`**  
**第二步：低額度主動提醒**  
**第三步：iPhone Shortcut 一鍵查詢**

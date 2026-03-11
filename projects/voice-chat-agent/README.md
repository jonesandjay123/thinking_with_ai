# 🎙️ Voice Chat Agent — 語音對話 Jarvis

> **目標：** 讓 Jones 可以用語音跟 Jarvis 溝通，不用每次都打字。

## 💡 核心需求

- Jones 用 Android 手機
- 想要像跟人說話一樣跟 Jarvis 互動
- 不走 Siri（iOS 限定），需要 Android 友善方案
- OpenClaw 生態圈應該有社群方案可以參考

## 🔍 研究方向

### 方案 A：OpenClaw 原生語音

OpenClaw 已內建 TTS（文字轉語音），但目前缺少 STT（語音轉文字）的原生整合。
需要研究：
- OpenClaw 社群有沒有人做過 voice input plugin
- Discord / Telegram voice message 是否能自動轉文字觸發 agent
- OpenClaw 的 `tts` tool 已經可以語音回覆，只差輸入端

### 方案 B：Telegram / WhatsApp 語音訊息

- Telegram bot 可以接收語音訊息 → 自動轉文字 → 送給 Jarvis
- WhatsApp 也支援語音訊息，OpenClaw 可能已有處理
- 需要測試：現有 channel 是否已經能處理 voice message

### 方案 C：專用語音 App / Web Interface

- 做一個簡單的 PWA（Progressive Web App），手機開啟後：
  - 按住說話 → Web Speech API 轉文字 → 送給 Jarvis API
  - Jarvis 回覆 → TTS 播放語音
- 或用現有的開源方案（如 Home Assistant + OpenClaw 整合）

### 方案 D：Android 快捷方式 / Tasker

- 用 Tasker / Automate 等 Android 自動化工具
- 語音輸入 → 轉文字 → HTTP 送給 OpenClaw gateway → 回覆轉語音
- 最接近「對著手機說話就行」的體驗

### 方案 E：Google Assistant Routine

- 類似 Siri 整合但用 Google Assistant
- 「Hey Google, tell Jarvis ...」
- 需要研究 Google Assistant ↔ webhook 整合

## 📋 研究清單

- [ ] 搜尋 OpenClaw Discord / GitHub 有沒有 voice input 相關討論
- [ ] 測試 Telegram 語音訊息 → Jarvis 是否已能處理
- [ ] 測試 WhatsApp 語音訊息 → 同上
- [ ] 調查 Web Speech API 的 Android 瀏覽器支援度
- [ ] 調查 Tasker + HTTP webhook 方案
- [ ] 看看社群有沒有人做過 Android + OpenClaw voice 整合

## 🎯 理想體驗

```
Jones（對手機說話）：「Jarvis，明天天氣怎樣？」
  → Android STT 轉文字
  → 送到 OpenClaw gateway
  → Jarvis 處理
  → TTS 語音回覆
  → 手機播放 Jarvis 的聲音
```

延遲目標：< 3 秒端到端

## 狀態

🟡 **構想中** — 先做研究，等 D2R bot 告一段落後實作

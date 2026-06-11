# Jarvis Android 控制專案

這個資料夾保存 Jones 與 GPT 在 2026-06-10 關於 `jarvis-android-agent`、Motorola 真機、ADB、小紅書 OCR observation，以及下一步 Android Computer Use Runtime 架構的對話。

## Files

- `conversation.md` — 完整可見逐字稿
- `extracted-messages.json` — 從 ChatGPT share payload 擷取出的 structured messages 與 metadata
- `source-page.html` — 擷取當下的公開 share page 原始 HTML 快照

## Source

<https://chatgpt.com/share/6a2a16ea-6d28-832d-a8f2-9b8c6f6a1d7b>

## Why this matters

這段對話把 `jarvis-android-agent` 從「能控小紅書」推進到更底層的方向：把 Motorola 真機整理成 Jarvis 的 Android Computer Use Runtime，也就是可觀察、可操作、可恢復、可記錄的手機身體。

## Durable Takeaway

第一階段的重點不是急著自動發文，而是先把手機操作能力產品化成穩定 runtime：

- observe
- detect_state
- act
- verify
- recover
- log
- summarize_for_agent

這會讓小紅書只是第一個 workflow，而不是整個專案的邊界。

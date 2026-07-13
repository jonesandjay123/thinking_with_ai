# SpaceXAI Voice Agent 與 Jarvis Voice Lite

這個資料夾保存 Jones 與 GPT 在 2026-07-13 的一段 ChatGPT share 對話。對話從 SpaceXAI / xAI 寄來的 Voice Agent Builder 介紹信開始，延伸到 speech-to-speech 語音代理、即時反應速度、工具呼叫、旅行 JSON 查詢，以及 Jarvis Voice Lite 的可能架構。

## Files

- `transcript.md` - ChatGPT share 可見文字逐字稿，由 `chatgpt-conversation-ingestor` 擷取
- `extracted-messages.json` - 從 React Router data model 擷取出的 structured visible messages 與 metadata
- `capture-report.json` - 擷取完整性報告與 skipped node 統計
- `raw-conversation-data.json` - ChatGPT share page runtime 的 raw conversation data model

## Source

<https://chatgpt.com/share/6a556e7a-b87c-83ea-a445-f1e9b191196f>

## Extraction Note

這次使用本地 `chatgpt-conversation-ingestor` repo 的 `capture:shared` CLI 擷取。報告顯示 extractor 是 `react-router-data-model`，completeness 是 `high`，共 32 個 nodes，其中 6 則可見 User/GPT 訊息被納入 transcript。

## Why this matters

這段對話釐清一個重要產品判斷：speech-to-speech voice agent 的價值不只是「語音模型」，而是把低延遲聽說、打斷、停頓判斷、工具呼叫和電話/WebSocket 入口包成一個可以快速驗證體驗的平台。

對 Jarvis 來說，這提供了一條務實路線：

- 第一階段不一定要接完整 Jarvis 記憶或 Codex 工具鏈。
- 可以先用 SpaceXAI Voice Agent 做一個反應速度與對話體驗優先的 Voice Lite。
- 若只需要讀旅行 JSON、查日程、回答住宿/景點/預約細節，可以用薄工具層或 MCP 接 Trip Planner 資料。
- 複雜研究、長時間規劃、跨 repo 修改、測試與 commit，仍應交給完整 Jarvis / Codex 類文字 agent。

## Durable Takeaway

最合理的方向不是在「絲滑語音」和「完整 Jarvis」之間二選一，而是分層：

```text
Voice Agent
  -> 快速日常對話、簡單旅行問答、即時語音體驗

Trip Planner / MCP tools
  -> 讀 JSON、查日程、查 booking、搜尋 trip items

Jarvis / Codex delegate
  -> 複雜規劃、深度推理、資料修改、驗證與工程操作
```

第一版可以先做成 `SpaceXAI Voice + trip-planner JSON tools`，甚至先完全不接 Jarvis 長期記憶。等實測發現哪些問題處理不好，再加 `ask_jarvis` / `delegate_to_jarvis` 作為升級路徑。

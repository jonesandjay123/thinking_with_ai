# Gemini / 阿斯拉語音入口、Asurada-first POC 與 Jovitrip

這個資料夾保存 Jones 與 GPT 在 2026-07-15 的一段 ChatGPT share 對話。對話從 Jarvis Voice 與反向代理版 Jovitrip 的既有成果出發，逐步釐清 Google / Gemini 生態可作為更絲滑的語音入口；最後把當天的實作重心收斂為：先做通用的 `asurada-app` 影子，再用既有 Trip Planner Firestore 做一個唯讀驗證。

## Files

- `transcript.md` - ChatGPT share 可見文字逐字稿，由 `chatgpt-conversation-ingestor` 擷取
- `extracted-messages.json` - 從 React Router data model 擷取出的 structured visible messages 與 metadata
- `capture-report.json` - 擷取完整性報告與 skipped node 統計
- `raw-conversation-data.json` - ChatGPT share page runtime 的 raw conversation data model

## Source

<https://chatgpt.com/share/6a57962a-cf64-83ea-a56c-e6a759dc9cc5>

## Extraction Note

這次使用本地 `chatgpt-conversation-ingestor` repo 的 `capture:shared` CLI 擷取。報告顯示 extractor 是 `react-router-data-model`，completeness 是 `high`，共 61 個 nodes，其中 19 則可見 User/GPT 訊息被納入 transcript（8 則 User、11 則 GPT）。本次檔案完整覆蓋先前較短的 shared-link capture。

## Why this matters

Jones 的鬼轉不是把手錶當核心，而是重新定義語音系統的責任分工：Google / Gemini 生態若能提供更快、更自然、低維護的 Android 與 Wear OS 語音入口，就不必讓 OpenClaw 自建整條語音基礎設施。

對話先提出兩個可比較的方向：

- **Google Tasks Command Bus**：Gemini 將明確命令寫進 Google Tasks；Jarvis / Bridge 再讀取、解析、透過 Jovitrip 管理 API 執行，並留下 audit / confirmation。適合單向、低風險、延後處理的指令。
- **阿斯拉 App**：自訂 Android 語音 App 以 Gemini Live / agent tool calling 作為對話層，將 Jovitrip 查詢、候選景點、移動行程、修改時間與留言等操作做成受權限、可確認、可 rollback 的工具。適合即時多輪對談與開車途中規劃。

## 對話最後收斂：先驗證 Asurada，不先重建 Jovitrip

對話後半把產品角色拆開：

- `asurada-app`：通用的 Gemini Live 語音助理入口／工具宿主。Jovitrip、Jovicheer、Jyn Null、Google 服務與未來 Jarvis 都是可選 adapter，不是它的身份本身。
- `jovitrip`：獨立的旅行產品、資料模型與工具介面；未來可被 Asurada 操作，也可獨立使用。

因此當天最小、可歸因的驗證不應同時重建 Jovitrip，而是：

```text
手機語音
  -> Gemini Live
  -> 先驗證一個寫死結果的 custom function
  -> 再用唯讀 function 查舊版 Trip Planner Firestore 的真實資料
  -> Gemini 語音回報
```

第一個成功問句是：「阿斯拉，舊版 Trip Planner 裡現在有哪些旅程？」這條 read-only 閉環若成立，才證明 Asurada 能以受控工具看見 Jones 的 Firebase 世界；寫入、跨專案權限、Jovitrip 重建、完整 audit / rollback 與 Jarvis 深度整合都留在後續階段。

## Durable Takeaway

更合理的結論不是「Gemini 取代 Jarvis」，而是分層：

```text
Google / Gemini
  -> 快速、原生、低維護的語音入口與 Google 生態操作

Asurada tool layer
  -> 有範圍、可驗證、可確認的專案 adapter；Jovitrip 是第一個候選世界

Jarvis / OpenClaw
  -> 個人長期上下文、跨系統執行、複雜工作與 rollback
```

消費者版 Gemini 能否直接對任意 Firestore 做 CRUD，對話中已被修正為不應假設。任何可寫入 Jovitrip 的實作，仍應採用明確授權的自訂工具／API 層，而不是把資料庫權限直接交給語音助理。

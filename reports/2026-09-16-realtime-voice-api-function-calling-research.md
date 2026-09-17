# 即時語音對話 API（Realtime Voice / Speech-to-Speech）市場調查

- 調查日期：2026-09-16
- 調查動機：ESP32 + Android App 專案使用 OpenAI Realtime API（GPT Live API），透過 function calling 觸發硬體動作（LED、OLED 顯示）。想知道其他大廠是否有類似「能說話、又能做事」的即時語音模型 API。
- 評估重點：**是否支援 function calling / tool use**（觸發外部動作）、協定、延遲、定價。

## 一、逐家廠商小結

### 1. OpenAI Realtime API
- **狀態**：公開可用。最新模型 **gpt-live-1**（2026-09-10 發布），全雙工語音，走新 `v1/live/sessions` 端點；另有 `gpt-realtime-2.1`（`v1/realtime`）。
- **Function calling**：✅ 支援，`response.function_call_arguments.delta` 事件串流回傳工具呼叫。
- **協定**：WebSocket、WebRTC、SIP。
- **延遲**：官方宣稱輪流發話約 0.8s。
- **定價**：gpt-live-1 為 $0.05/分鐘（按秒計費，後端模型與工具費另計）；免費層不支援。

### 2. Google Gemini Live API
- **狀態**：公開可用（Gemini Developer API、AI Studio、Vertex AI）。最新 **gemini-3.8-live**（2026-09-14/15 發布，與 OpenAI gpt-live-1 同週登場）。
- **Function calling**：✅ 支援（官方 live-tools 文件）。3.8 首度支援**非同步 function calling**——工具在背景執行時語音回覆不中斷；另支援 Google Search 即時 grounding。
- **協定**：原生僅 WebSocket（16-bit PCM，輸入 16kHz／輸出 24kHz）；WebRTC 需經合作夥伴（LiveKit、Daily、Pipecat）。
- **延遲**：官方僅稱 low-latency，未公布數字。純語音會話限 15 分鐘。
- **定價**：按 token 計費（音訊輸入約 $0.005/min、輸出約 $0.018/min），**有免費層**。⚠️ 3.8 Live 的獨立 rate card 尚未完整揭露，成本評估先以官方定價頁為準。

### 3. Microsoft Azure OpenAI Realtime API
- **狀態**：Azure AI Foundry 提供，已 GA（gpt-realtime、gpt-realtime-2 等）。
- **Function calling**：✅ 完整支援。
- **協定**：WebRTC（官方標示 ~100ms）、WebSocket（~200ms）、SIP。
- **定價**：與 OpenAI 面價一致。主要差異在企業合規、區域部署。

### 4. Amazon Nova Sonic（Bedrock）
- **狀態**：2025-04 發布，2025 下半年升級 **Nova 2 Sonic**（`amazon.nova-2-sonic-v1:0`）。
- **Function calling**：✅ 原生支援，Nova 2 Sonic 加入非同步工具呼叫；另可經 Bedrock AgentCore 走 **MCP** 呼叫 REST API / Lambda。
- **協定**：Bedrock 專屬 bidirectional streaming（HTTP/2 雙向事件串流）。
- **延遲**：第三方實測 TTFA 平均 **1.09 秒**，優於 GPT-4o Realtime（1.18s）。
- **定價**：宣稱比 GPT-4o Realtime 便宜約 80%（範例：10,000 通 30 秒通話月費約 $83）。

### 5. Anthropic — ❌ 無原生即時語音 API
- 截至 2026 年中無 realtime voice / speech-to-speech API。僅 App 內 Voice Mode 與音檔轉錄。
- 變通作法：級聯式架構 STT（Deepgram）→ Claude（文字推理＋工具使用）→ TTS（ElevenLabs），工具呼叫發生在文字層；缺點是三段延遲疊加（通常 >2 秒）。

### 6. xAI Grok Voice Agent API
- **狀態**：2025-12-17 推出，有 Speech-to-Speech 官方文件；2026-07 另推免程式碼 Voice Agent Builder。
- **Function calling**：✅ 支援自訂 function tools（＋內建 web_search、x_search、file_search）。
- **協定**：WebSocket，**刻意相容 OpenAI Realtime 事件格式**。
- **定價**：$0.05/分鐘。

### 7. Mistral Voxtral — ⚠️ 僅即時轉錄，非 S2S 對話 API
- Voxtral Realtime 為 streaming STT（延遲可調至 <200ms）；完整語音對話需自行組裝管線。語音理解模型可直接從語音觸發 function call，但非 realtime agent 的工具迴圈。

### 8. Meta（Llama）— ❌ 無第一方即時語音 API
- Llama API 僅限文字模型；語音對話只在消費者端 Meta AI App，不對開發者開放。

### 9. 其他值得注意者
- **Alibaba Qwen-Omni**：✅ `qwen3.5-omni-flash-realtime` 經 Model Studio WebSocket 提供，支援 FunctionCall、semantic VAD；權重開源可自部署。
- **ElevenLabs Conversational AI**：✅ 管線式（STT+LLM+TTS），function calling 完整（Client/Server/MCP tools），WebSocket + WebRTC，約 $0.11/分鐘。
- **Deepgram Voice Agent API**：✅ 單一 WebSocket 統一編排，支援 function calling，$4.50/小時全包，$200 免費額度。
- **Hume EVI 3**：情感化即時語音，支援自訂工具，約 $0.102/分鐘。
- **Cartesia**：強項是超低延遲 TTS；agent 平台支援 tool calling，約 $0.06/分鐘。

## 二、比較總表

| 廠商 | 即時 S2S API | Function calling | 協定 | 延遲量級 | 定價參考 |
|---|---|---|---|---|---|
| OpenAI | ✅ | ✅ | WS / WebRTC / SIP | ~0.8s | $0.05/min |
| Google Gemini | ✅ | ✅（非同步） | WS（原生） | 未公布 | 音訊約 $0.005/$0.018/min，有免費層 |
| Azure OpenAI | ✅ | ✅ | WebRTC / WS / SIP | WebRTC ~100ms | 與 OpenAI 一致 |
| Amazon Nova Sonic | ✅ | ✅（非同步+MCP） | Bedrock 雙向串流 | 實測 1.09s | 比 GPT-4o Realtime 便宜 ~80% |
| Anthropic | ❌ | ❌（僅級聯） | — | >2s | — |
| xAI Grok | ✅ | ✅ | WS（相容 OpenAI） | 未公布 | $0.05/min |
| Mistral Voxtral | ⚠️ 僅 STT | ⚠️ 理解層可觸發 | Streaming | STT ~200ms | $0.006/min |
| Meta | ❌ | ❌ | — | — | — |
| Qwen-Omni | ✅ | ✅ | WS | 未公布 | 按 token |
| ElevenLabs | ✅（管線式） | ✅ | WS + WebRTC | ~1–2s | ~$0.11/min |
| Deepgram | ✅（管線式） | ✅ | WS | 次秒級 | $4.50/小時 |

## 三、給 IoT / ESP32 場景的建議排名

場景：Android App ↔ 語音 API，工具呼叫觸發硬體動作（LED、OLED）。考量 function calling 成熟度、延遲、文件完整度、成本：

1. **OpenAI（gpt-live-1 / Realtime）**——生態最成熟、ESP32/MQTT 整合範例最多，$0.05/分鐘定價單純。想最快做出成品的首選。
2. **Google Gemini 3.8 Live**——有免費層、便宜，支援非同步 function calling。注意原生僅 WebSocket、定價卡尚未完整。
3. **Amazon Nova Sonic**——實測延遲最佳、便宜 80%、MCP 適合觸發硬體 REST/Lambda。缺點：需走 Bedrock，中文支援待確認。
4. **Azure OpenAI**——需企業合規或區域部署才選，否則直接用 OpenAI。
5. **xAI Grok**——協定相容 OpenAI，備選。
6. **ElevenLabs / Deepgram**——管線式，適合想自選模型的架構；延遲多 0.5–1s。
7. **Anthropic、Meta、Mistral**——目前無可用原生 S2S API，不建議作主力。

## 備註
- Google 3.8 Live 與 OpenAI gpt-live-1 皆於 2026-09 同週發布，文件與定價仍在收斂，做成本承諾前以官方定價頁為準。
- Anthropic 未發布原生方案的傳聞請勿納入決策。

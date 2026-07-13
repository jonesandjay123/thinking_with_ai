# 即時語音代理選項調查：Jarvis Voice Lite

Date: 2026-07-13

Related conversation:

- [`SpaceXAI Voice Agent 與 Jarvis Voice Lite`](../../conversations/spacexai-voice-agent-jarvis-voice-lite-2026-07-13/)

## 背景

Jones 目前的 Jarvis Voice loop 是：

```text
語音訊息
  -> jarvis-ai-bridge
  -> Jarvis / Codex / OpenClaw
  -> 文字回應
  -> TTS 語音輸出
```

實測平均回應時間約 31-41 秒。這對「交辦任務」可以接受，但對「像真人一樣語音聊天」太慢。因此這輪調查的問題不是「誰最聰明」，而是：

> 有沒有大廠或成熟平台能先提供低延遲 speech-to-speech / realtime voice agent，讓 Jarvis Voice 先有絲滑體驗，再用工具接回 Trip Planner 或 Jarvis？

## 總結判斷

最值得優先試的三條路：

1. **OpenAI Realtime API / Voice Agents**：最貼近 Jarvis 現有工具與 Agents 概念。官方定位就是低延遲 voice agent，支援 WebRTC/WebSocket、tools、handoffs、guardrails。適合做「OpenAI Realtime + trip-planner tools」原型。
2. **Google Gemini Live API**：非常適合測中英混合、自然打斷、Google Search/function calling 和 Android/WebSocket 原型。官方頁明確標示 low-latency voice/video、barge-in、affective dialog、tool use、Google Search。
3. **Azure Voice Live API**：最 enterprise/production 味，支援 GPT realtime / GPT / Phi 等模型選項，WebSocket，Azure Speech 的 noise suppression、echo cancellation、advanced end-of-turn、custom voice 等能力。適合長期如果要穩、要企業級 speech stack。

最快 dashboard 產品化實驗：

4. **ElevenAgents**：不是單一 speech-to-speech foundation model，而是 STT + LLM + TTS + turn-taking orchestration。優勢是聲音、SDK、Android/Kotlin/iOS/React Native、工具、知識庫、電話整合都很完整。
5. **Vapi / Retell / Deepgram Voice Agent**：更偏 voice-agent infrastructure / phone agent 平台。很快能做 phone/web voice bot，但它們通常是 orchestration 層，不一定是「大模型原生 speech-to-speech」。

## Provider Notes

### OpenAI Realtime API

官方 Realtime overview 表示 realtime session 會保持連線，client 可以送 audio/text、接收 model responses、tool calls、session events；voice-agent session 用於讓模型回應使用者、呼叫工具、管理對話狀態。官方也明確區分 WebRTC 適合 browser/mobile 直接收放音，WebSocket 適合 server media pipeline；並指出 Realtime 2 支援 speech-to-speech workflows 的 reasoning，可以從 low reasoning effort 開始調整延遲與任務複雜度。Source: <https://developers.openai.com/api/docs/guides/realtime>

官方 Voice Agents guide 也把選擇拆成兩種：live audio speech-to-speech 適合低延遲、barge-in、自然 turn-taking、realtime tool use；chained pipeline 適合需要明確控制 STT -> text agent -> TTS 的流程。TypeScript fastest path 是 `RealtimeAgent` + `RealtimeSession` + `gpt-realtime-2.1`，可以把 tools / handoffs / guardrails 掛在 agent 上。Source: <https://developers.openai.com/api/docs/guides/voice-agents>

Pricing page 顯示 realtime/audio generation models 以 tokens 計價：`gpt-realtime-2.1` audio input `$32 / 1M tokens`、audio output `$64 / 1M tokens`；`gpt-realtime-2.1-mini` audio input `$10 / 1M tokens`、audio output `$20 / 1M tokens`。Source: <https://developers.openai.com/api/docs/pricing>

Jarvis relevance:

- 最容易接 existing tool / MCP 思維。
- 可以直接做 WebRTC mobile/browser voice agent。
- 適合作為第一個「不是 full Jarvis、但能讀 Trip Planner tools」的原型。
- 若要保持最接近 GPT/Jarvis 的行為語氣，OpenAI 路線成本最低。

### Google Gemini Live API

Google Gemini Live API 官方頁說明它提供 low-latency real-time voice/video interactions，支援 continuous audio/video/text streams，輸出 immediate human-like spoken responses。功能包含 high audio quality、24 languages、barge-in、affective dialog、tool use、audio transcription、proactive audio；protocol 是 stateful WebSocket。Source: <https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/live-api>

Gemini 2.5 Flash Live native audio 頁面列出 `gemini-live-2.5-flash-native-audio`，GA，支援 audio input/output、function calling、Google Search grounding、system instructions，context window 128K，預設 session 10 minutes 可延長，最大 concurrent sessions 1000。它也強調 improved barge-in、robust function calling、seamless multilingual switching、affective dialog。Source: <https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/gemini/2-5-flash-live-api>

Jarvis relevance:

- 很適合測「中英混合、旅行中查資料、Google Search、自然打斷」。
- Android / Google Cloud ecosystem 可能對手機端原型更順。
- 可以先做 `Gemini Live + Trip Planner function calling`，完全不接 Jarvis 長期記憶。
- 風險是 Google Cloud / Vertex setup 可能比 OpenAI dashboard path 繁。

### Microsoft Azure Voice Live API

Microsoft Voice Live API 官方文件定義它為低延遲、高品質 speech-to-speech voice agent solution，整合 speech recognition、generative AI、text-to-speech，不需要手動編排多個 component。功能包含 140+ locales STT、600+ voices、noise suppression、echo cancellation、robust interruption detection、advanced end-of-turn detection、avatar、function calling / VoiceRAG。Source: <https://learn.microsoft.com/en-us/azure/ai-services/speech-service/voice-live>

它使用 WebSocket events，並設計成與 Azure OpenAI Realtime API events 大致相容。支援模型包含 `gpt-realtime-1.5`、`gpt-realtime`、`gpt-realtime-mini`、`gpt-4o`、`gpt-4.1`、`gpt-5` 系列、Phi、`azure-realtime` 等。Source: <https://learn.microsoft.com/en-us/azure/ai-services/speech-service/voice-live>

Jarvis relevance:

- 最適合需要 speech infra 穩定、噪音處理、custom voice、企業級 regional/security 設定時。
- 如果未來 Jarvis Voice 真的變成常用語音入口，Azure 是 production candidate。
- 原型階段可能比 OpenAI/Gemini 重，但它的 end-of-turn / speech stack 很值得 benchmark。

### Amazon Nova Sonic / Nova 2 Sonic

AWS Nova Sonic 官方文件說它提供 real-time conversational interactions through bidirectional audio streaming，使用 unified speech understanding and generation architecture。功能包含 low-latency multi-turn conversation、interruptions without losing context、RAG、function calling、agentic workflow support、background-noise robustness、multilingual expressive voices。Source: <https://docs.aws.amazon.com/nova/latest/userguide/speech.html>

Nova 2 Sonic 文件進一步列出 automatic language detection/switching、intelligent turn-taking、async tool handling、cross-modal text/audio input，並註明 connection limit 8 minutes，可用 renewal/session continuation pattern。Source: <https://docs.aws.amazon.com/nova/latest/nova2-userguide/using-conversational-speech.html>

Jarvis relevance:

- AWS Bedrock 環境如果已熟，值得一試。
- 支援 tool/RAG/agentic flows，方向吻合。
- 對 Jones 目前快速 Android/Jarvis prototype 而言，AWS auth/Bedrock setup 可能不是最快。

### ElevenAgents

ElevenLabs ElevenAgents 官方文件定位為 voice-rich, expressive multimodal agents，可用 dashboard、developer toolkit、visual workflow builder。它支援 web/mobile/telephony deployment，包含 React、Swift、Kotlin、React Native SDK、SIP、Twilio、WebSocket，並有 knowledge base、tools、personalization、auth、analytics。Source: <https://elevenlabs.io/docs/eleven-agents/overview>

架構上 ElevenAgents 協調四個元件：fine-tuned STT、可選 LLM 或 custom LLM、low-latency TTS、proprietary turn-taking model。Source: <https://elevenlabs.io/docs/eleven-agents/overview>

Pricing page 顯示 Free 10k credits/月，Starter $6，Creator $22，Pro $99；credits across products，Speech to Text 約 330 credits/min，Business tier 提到 low-latency TTS as low as 5c/minute。Source: <https://elevenlabs.io/pricing>

Jarvis relevance:

- 如果想最快做出「聽起來好」的 Android 版語音助手，ElevenAgents 很有吸引力。
- Kotlin SDK 是大優點。
- 它不是原生 unified speech-to-speech model，而是 orchestration；但對使用者體驗不一定是問題。

### Deepgram Voice Agent

Deepgram Voice Agent API 官方文件說它 handles the full speech pipeline (listening, thinking, speaking)，支援 playground、build guides、agent configuration、function calling、multi-agent architecture、telephony、browser SDK。Source: <https://developers.deepgram.com/docs/voice-agent>

Feature overview 列出 voice selection、supported LLM models、agent context、prompt/speak/think updates、function call request/response 等。Source: <https://developers.deepgram.com/docs/voice-agent-feature-overview>

Pricing page 顯示 Voice Agent API Standard `$0.075/min`、Standard BYO TTS `$0.065/min`、Custom BYO LLM + TTS `$0.050/min`、Advanced `$0.163/min`。Source: <https://deepgram.com/pricing>

Jarvis relevance:

- 便宜、清楚、developer-friendly。
- 如果目標是「不要自己接 STT/TTS/endpointing」，Deepgram 是很好的 practical baseline。
- 可做成 benchmark，但聲音人格/中文體驗要實測。

### Hume EVI

Hume EVI 是 speech-to-speech empathic voice interface，重點是 prosody / emotion / end-of-turn。官方說 EVI 用 tune、rhythm、timbre 判斷何時說話，支援 immediate low-latency response、always interruptible、emotionally intelligent speech。Source: <https://dev.hume.ai/docs/speech-to-speech-evi/overview>

EVI 4-mini 支援 English, Japanese, Korean, Spanish, French, Portuguese, Italian, German, Russian, Hindi, Arabic；文件沒有列中文。Source: <https://dev.hume.ai/docs/speech-to-speech-evi/overview>

Jarvis relevance:

- 很適合測「情緒、語氣、停頓判斷」。
- 但中文不是強項，對 Jones 中英混用主場未必優先。

### Vapi

Vapi 官方文件定位為 voice AI agent developer platform，負責 voice infra，支援 natural conversations、phone calls、web integration、tools/API/database integration、多 assistant squads。官方列出 sub-600ms response times、phone/web integration、tool integration。Source: <https://docs.vapi.ai/>

Pricing page 顯示 Vapi hosting calls `$0.05/min`，model provider cost 另計；可 BYO API key。Source: <https://vapi.ai/pricing>

Jarvis relevance:

- 最快做 phone/web assistant 的平台之一。
- 如果你要「今天就做一個會打電話的 Jarvis Lite」，Vapi 比自己接 WebRTC 快。
- 但它是 orchestration 平台，不是單一大廠 foundation model。

### Retell AI

Retell 官方文件定位為 AI phone agents platform，支援 inbound/outbound、telephony providers、testing、monitoring、conversation flow agent、single/multi prompt agent、phone calls、custom telephony、webhooks、post-call analysis。Source: <https://docs.retellai.com/>

Jarvis relevance:

- 很適合 phone-agent / receptionist 場景。
- 如果原型是手機 app 裡的個人 Jarvis，不如 OpenAI/Gemini/Eleven/LiveKit 直接。

### LiveKit Agents

LiveKit Voice AI quickstart 說可以在 10 分鐘內做出能在 terminal/browser/telephone/native app 對話的 voice assistant。它支援 starter templates、LiveKit Cloud、Agent Console，並明確支援兩種 pipeline：STT-LLM-TTS pipeline，以及 single realtime model，例如 OpenAI Realtime API。Source: <https://docs.livekit.io/agents/start/voice-ai/>

Jarvis relevance:

- 如果要自己掌握 media transport / rooms / native app realtime audio，LiveKit 是很好的 infra。
- 它不是模型 provider，而是可把 OpenAI/Gemini/Deepgram/Cartesia 等接起來的工程骨架。

## Notable Absences

- **Anthropic / Claude**：目前可當強文字 agent / reasoning brain，但沒有找到等價的官方 realtime speech-to-speech API。適合做後端 `ask_jarvis` / deeper reasoning，不是第一層 low-latency mouth/ear。
- **Apple / Siri / on-device stack**：適合系統整合與 dictation/TTS，但不像上述 provider 有可直接拿來做 custom realtime voice agent 的雲端 STS API。
- **Meta**：有研究與 consumer voice surfaces，但這輪沒有找到適合立即接進 Jarvis Voice Lite 的官方 developer STS agent path。

## Recommended Experiments

### Experiment A: OpenAI Realtime + Trip Planner Tools

Goal: 測「聲音反應速度 + tool calling + 旅行 JSON 問答」。

Prototype shape:

```text
Android/Web client mic
  -> OpenAI Realtime WebRTC/WebSocket
  -> RealtimeAgent tools:
       get_trip_summary
       get_day_schedule
       get_place_details
       search_trip_items
  -> spoken response
```

Why first:

- 最接近 Jarvis 現有思維。
- 直接支援 voice agents、tools、handoffs。
- 可以比較 `gpt-realtime-2.1` vs `gpt-realtime-2.1-mini`。

Success metric:

- First audible response under 2 seconds for simple questions.
- Tool-backed trip answer under 4-6 seconds.
- User can interrupt naturally.

### Experiment B: Gemini Live + Trip Planner Function Calling

Goal: 測中英混合、自然打斷、Google Search、travel context。

Why:

- Gemini Live native audio 直接支援 function calling / Google Search / multilingual switching。
- Google ecosystem 對 Android app 原型有自然優勢。

Success metric:

- 中英混合問題準確。
- 旅行 JSON tool call 正常。
- 可以在旅途中自然問「明天幾點出發」「附近有什麼」。

### Experiment C: ElevenAgents or Vapi Dashboard Prototype

Goal: 不先寫太多底層，快速感受「voice product」。

Why:

- ElevenAgents 有 Kotlin SDK / mobile/web/telephony / tool / knowledge base。
- Vapi 能最快做 phone/web voice agent，hosting $0.05/min + provider cost。

Success metric:

- 1-2 小時內能講起來。
- 聲音與 turn-taking 讓 Jones 覺得「不像送工單」。

## Recommendation

短期不要再把 full Jarvis 當第一層語音 brain。31-41 秒延遲很合理地說明：full Jarvis 是「深度工作者」，不是「即時嘴巴」。

建議架構：

```text
Voice Lite
  -> OpenAI Realtime 或 Gemini Live
  -> 快速聊天 / 旅行 JSON 查詢 / 簡單工具

Jarvis Deep
  -> 原本 OpenClaw / Codex / memory / repo tools
  -> 長任務、複雜推理、寫檔、部署、commit

Router
  -> 如果 Voice Lite 判斷問題太深，就說：
     「這個我要交給 Jarvis 深度處理，等一下回你。」
```

第一個實驗我會選：

1. OpenAI Realtime + tiny Trip Planner tool server
2. Gemini Live + same tools
3. ElevenAgents dashboard / Kotlin SDK 快速體感測試

只要其中一個能把「簡單問答」降到 1-3 秒、「讀 trip data」降到 4-6 秒，就已經證明 Jarvis Voice 不該每句話都走 full Jarvis consciousness。

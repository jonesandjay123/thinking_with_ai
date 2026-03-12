# Voice Chat with AI Agents — Research

> Compiled 2026-03-10. Focus: practical voice chat setups for always-on AI agents (like OpenClaw/Jarvis), with Android phone compatibility.

---

## Table of Contents

1. [Overview & Recommendation Summary](#overview--recommendation-summary)
2. [Approach 1: Telegram Voice Messages → AI Agent](#approach-1-telegram-voice-messages--ai-agent)
3. [Approach 2: WhatsApp Voice Messages → AI Agent](#approach-2-whatsapp-voice-messages--ai-agent)
4. [Approach 3: Sanna Bot — Voice-First Android AI Assistant](#approach-3-sanna-bot--voice-first-android-ai-assistant)
5. [Approach 4: Web Speech API PWA (Browser-Based Voice Chat)](#approach-4-web-speech-api-pwa-browser-based-voice-chat)
6. [Approach 5: LiveKit Agents (Real-Time Voice AI Framework)](#approach-5-livekit-agents-real-time-voice-ai-framework)
7. [Approach 6: Home Assistant Voice (Assist)](#approach-6-home-assistant-voice-assist)
8. [Approach 7: Sapphire — Self-Hosted AI Companion](#approach-7-sapphire--self-hosted-ai-companion)
9. [Approach 8: EVA — Enhanced Voice Assistant](#approach-8-eva--enhanced-voice-assistant)
10. [Approach 9: Rapida — Voice AI Orchestration Platform](#approach-9-rapida--voice-ai-orchestration-platform)
11. [Approach 10: Android Tasker/Automate → Webhook Workflows](#approach-10-android-taskerautomate--webhook-workflows)
12. [Approach 11: Signal Voice Message Transcription](#approach-11-signal-voice-message-transcription)
13. [Approach 12: OpenAI Realtime API Voice Assistants](#approach-12-openai-realtime-api-voice-assistants)
14. [Approach 13: Google Assistant → Custom Webhook (Deprecated)](#approach-13-google-assistant--custom-webhook-deprecated)
15. [Approach 14: Willow — Open Source Voice Assistant Hardware](#approach-14-willow--open-source-voice-assistant-hardware)
16. [Architecture Patterns Summary](#architecture-patterns-summary)
17. [Recommendation for OpenClaw/Jarvis Setup](#recommendation-for-openclawjarvis-setup)

---

## Overview & Recommendation Summary

There are fundamentally **three architectural patterns** for voice-chatting with AI agents:

| Pattern | Latency | Complexity | Best For |
|---------|---------|------------|----------|
| **Messaging app voice notes** (Telegram/WhatsApp) | High (async) | Low | Quick hands-free input via existing apps |
| **Web/app-based real-time voice** (PWA, LiveKit, Sanna) | Low (real-time) | Medium-High | True conversational voice chat |
| **Smart home / hardware** (Home Assistant, Willow) | Low | High | Home automation + voice |

**For an Android phone + OpenClaw setup, the quickest wins are:**
1. **Telegram voice messages** — OpenClaw already has Telegram channel support. Just need Whisper transcription in the pipeline.
2. **Custom PWA with Web Speech API** — Build a simple web page that does STT in-browser, sends text to your agent API, plays back TTS.
3. **Sanna Bot** — Purpose-built open-source Android voice-first AI assistant that explicitly mentions OpenClaw compatibility.

---

## Approach 1: Telegram Voice Messages → AI Agent

### How It Works
Telegram bots can receive voice messages (`.ogg` files) via the Bot API. The pipeline:
1. User sends voice message to Telegram bot
2. Bot downloads the audio file via `getFile` API
3. Audio is transcribed using Whisper (OpenAI API, local whisper.cpp, or Groq's Whisper)
4. Transcribed text is fed to the AI agent (Claude/GPT)
5. Response is sent back as text (or as TTS audio via `sendVoice`)

### Key Projects
- **telegram-chatgpt-n8n-bot** — n8n workflow: Telegram voice → Whisper → ChatGPT → reply
  - Repo: `github.com/Dana18-Mohammed/telegram-chatgpt-n8n-bot`
  - Uses n8n (no-code automation) for the pipeline
- **Various Python bots** — Many community implementations exist; the pattern is well-established

### Pros
- ✅ **Lowest friction** — user just holds record button in Telegram
- ✅ OpenClaw already supports Telegram as a channel
- ✅ Works on any phone, any OS, no app to install
- ✅ Async — doesn't require always-on connection
- ✅ Voice notes survive network interruptions

### Cons
- ❌ Not real-time conversational — latency of record → upload → transcribe → respond → read
- ❌ Requires Whisper API or self-hosted transcription service
- ❌ Response comes back as text (unless you add TTS and send as voice note back)
- ❌ No wake word / hands-free activation

### Complexity: **Low-Medium**
- If OpenClaw already handles Telegram messages, you mainly need to add voice message handling (download .ogg, transcribe, forward text)
- Can use Groq's Whisper API for fast, free-tier transcription

### Android Compatibility: **Excellent** ✅
- Just use Telegram app, nothing special needed

### Integration Path for OpenClaw
OpenClaw receives Telegram voice messages → detect `voice` message type → download .ogg → send to Whisper API → inject transcribed text as regular message into agent pipeline.

---

## Approach 2: WhatsApp Voice Messages → AI Agent

### How It Works
Similar to Telegram, but using WhatsApp's infrastructure:
1. User sends voice note in WhatsApp
2. Bot (using whatsmeow Go library or WhatsApp Business API) receives the audio
3. Audio transcribed via Whisper/Groq
4. Response sent back as text message

### Key Projects
- **whatsapp-voice-ai-assistant** (⭐13) — Go + Python microservices
  - Repo: `github.com/cr2007/whatsapp-voice-ai-assistant`
  - Uses whatsmeow (Go) for WhatsApp connection + Flask for Whisper transcription + Groq API for AI responses
  - Well-documented, microservices architecture
- **WhatsApp-Voice-Transcription-AI** (⭐5) — Python, GPT-4 + Whisper
  - Repo: `github.com/t-k-c/WhatsApp-Voice-Transcription-AI`

### Pros
- ✅ Very natural for users — WhatsApp voice notes are familiar
- ✅ OpenClaw already supports WhatsApp as a channel
- ✅ Some projects are well-documented and production-ready

### Cons
- ❌ WhatsApp API is more complex than Telegram's (need Business API or reverse-engineered library)
- ❌ whatsmeow requires QR code scan, can disconnect
- ❌ Same async latency issues as Telegram
- ❌ WhatsApp Business API has costs for high volume

### Complexity: **Medium**
- whatsmeow library handles the hard parts but needs Go runtime
- If already using OpenClaw's WhatsApp channel, voice message handling would be an addition

### Android Compatibility: **Excellent** ✅

---

## Approach 3: Sanna Bot — Voice-First Android AI Assistant

### How It Works
Sanna is an **open-source Android app** specifically designed as a voice-first AI assistant. It explicitly states: *"What OpenClaw does for your desktop, Sanna does for your phone."*

Pipeline: Wake word ("Hey Sanna") → STT → LLM agent loop (OpenAI/Claude) → TTS → spoken response

### Key Features
- 🗣️ Wake word activation (Picovoice)
- 🧠 Personal memory (remembers facts about you)
- 📝 Skills are Markdown files (like OpenClaw!)
- 🚗 Driving mode (short responses, auto-reads notifications)
- 🤖 UI Automation via Android Accessibility Services
- 🔄 Agentic tool loop (multi-step reasoning)
- ⏰ Sub-agent scheduler
- 19 built-in skills (Gmail, Calendar, Slack, Spotify, WhatsApp, SMS, etc.)

### Repo
- `github.com/sannabotdev/sannabotapp` (⭐7, TypeScript)
- Pre-built test APK available by email

### Pros
- ✅ **Purpose-built for exactly this use case** — voice-first Android AI assistant
- ✅ Explicitly designed as OpenClaw's phone companion
- ✅ Wake word for hands-free activation
- ✅ Real-time voice conversation (not async voice notes)
- ✅ Skills-as-Markdown architecture (familiar to OpenClaw users)
- ✅ Driving mode is killer feature for mobile
- ✅ No backend needed — runs on-device with API calls

### Cons
- ❌ Very new project (7 stars), may have rough edges
- ❌ Requires many API keys to fully set up (OpenAI/Claude, Google, Spotify, Picovoice, Slack)
- ❌ Not directly integrated with OpenClaw — it's its own agent
- ❌ Android only (no iOS yet, testing)

### Complexity: **Medium-High**
- Building from source requires many API keys
- Pre-built APK available for testing

### Android Compatibility: **Native Android App** ✅✅✅

---

## Approach 4: Web Speech API PWA (Browser-Based Voice Chat)

### How It Works
A Progressive Web App (PWA) that runs in the browser and uses:
1. **Web Speech API** (`SpeechRecognition`) for STT — runs in Chrome, free, no API needed
2. Sends transcribed text to your AI agent's HTTP API
3. Receives text response
4. Uses **Web Speech API** (`SpeechSynthesis`) or a TTS API (ElevenLabs, OpenAI TTS) to speak the response

### Key Projects
- **ChatGPTVoice** — React + Vite + Web Speech API
  - Repo: `github.com/GURU-DUTT/ChatGPTVoice`
  - Simple browser-based voice chat with ChatGPT
- Custom implementations are straightforward (~200 lines of JS)

### Pros
- ✅ **Zero app installation** — works in Chrome on Android
- ✅ Web Speech API is free (uses Google's servers for recognition)
- ✅ Can be a PWA (add to home screen, feels like an app)
- ✅ Full control over the pipeline
- ✅ Easy to connect to any HTTP API (OpenClaw webhook, etc.)
- ✅ Can use high-quality TTS (ElevenLabs) for responses

### Cons
- ❌ Web Speech API requires internet (sends audio to Google)
- ❌ No wake word — user must tap to start listening
- ❌ Chrome on Android sometimes kills background audio
- ❌ Web Speech API can be unreliable (connection drops, browser differences)
- ❌ Not a native experience — no lock screen access, no notification integration

### Complexity: **Low**
- ~200 lines of HTML/JS for a basic implementation
- Host on any static site (GitHub Pages, Vercel)

### Android Compatibility: **Good** ✅
- Works in Chrome for Android
- Can be installed as PWA
- No wake word / background listening

### Sample Architecture
```
[Android Chrome PWA]
    ↓ Web Speech API (STT)
[Transcribed Text]
    ↓ HTTP POST
[OpenClaw/Agent API Webhook]
    ↓ Agent processes
[Text Response]
    ↓ 
[Browser TTS or ElevenLabs API]
    ↓
[Audio playback in browser]
```

---

## Approach 5: LiveKit Agents (Real-Time Voice AI Framework)

### How It Works
LiveKit is a real-time communication framework (WebRTC-based). Their Agents SDK lets you build voice AI agents that:
1. Stream audio in real-time from client to server
2. Run STT → LLM → TTS pipeline on the server
3. Stream audio response back to client in real-time

It's designed for production-grade, low-latency voice conversations.

### Key Features
- Real-time bidirectional audio streaming via WebRTC
- Built-in STT, LLM, TTS plugin system (supports OpenAI, Anthropic, Deepgram, etc.)
- State-of-the-art turn detection (knows when user stopped talking)
- Interruption handling (user can interrupt the AI mid-response)
- Multi-agent handoff
- Open source (Apache 2.0)
- Cloud or self-hosted

### Docs & Links
- Docs: `docs.livekit.io/agents/`
- Python + Node.js SDKs
- Cloud hosted or self-hosted

### Pros
- ✅ **Best-in-class real-time voice** — lowest latency, production-grade
- ✅ Handles all the hard WebRTC stuff
- ✅ Turn detection, interruption, VAD built-in
- ✅ Works with Claude, OpenAI, any LLM
- ✅ Android SDK available for native apps
- ✅ Web client works in mobile browsers too
- ✅ Free tier available on LiveKit Cloud

### Cons
- ❌ **Overkill for simple setups** — enterprise-grade framework
- ❌ Requires running a LiveKit server (or using their cloud)
- ❌ Not directly integrated with OpenClaw
- ❌ Would need custom bridge: LiveKit agent ↔ OpenClaw
- ❌ Learning curve for the framework

### Complexity: **High**
- Full framework to learn
- Need to build or adapt an agent that bridges to OpenClaw

### Android Compatibility: **Good** ✅
- Web client works in Chrome
- Native Android SDK available
- React Native SDK available

---

## Approach 6: Home Assistant Voice (Assist)

### How It Works
Home Assistant's Assist is a voice assistant built into the home automation platform:
1. Voice captured by satellite device (ESP32, Raspberry Pi, phone, or $13 DIY device)
2. STT via Home Assistant Cloud or local Whisper
3. Intent recognition or LLM-based response
4. TTS response played back
5. Wake word support ("Hey Jarvis", "Hey Nabu")

### Key Features
- Wake word detection (works on Android phone too!)
- Can run entirely locally (Whisper + local LLM)
- $13 DIY voice satellite hardware option
- Companion app for Android has voice support
- Can integrate LLMs for AI personality responses

### Links
- Docs: `home-assistant.io/voice_control/`
- Voice PE hardware: `home-assistant.io/voice-pe/`
- Android wake word: `home-assistant.io/voice_control/android/`

### Pros
- ✅ Wake word on Android phone! ("Hey Jarvis" works even when phone is locked)
- ✅ Huge community and ecosystem
- ✅ Can run 100% locally
- ✅ $13 DIY satellite option
- ✅ LLM personality support (can use OpenAI/Claude for responses)

### Cons
- ❌ **Primarily designed for home automation**, not general AI assistant
- ❌ Requires Home Assistant installation (heavy)
- ❌ Voice commands are mostly for controlling devices
- ❌ LLM integration exists but is secondary to intent-based control
- ❌ Not designed to forward voice to an external AI agent

### Complexity: **Medium-High**
- HA installation + voice pipeline setup + LLM integration

### Android Compatibility: **Good** ✅
- Home Assistant Companion app for Android
- Wake word detection on Android
- Background listening supported

### Relevance for OpenClaw
Home Assistant's Android wake word detection could potentially be repurposed — the companion app can listen for "Hey Jarvis" and trigger actions. Could theoretically forward to OpenClaw via webhook, but that's not a built-in path.

---

## Approach 7: Sapphire — Self-Hosted AI Companion

### How It Works
Sapphire is a self-hosted AI companion framework with full voice support:
1. Wake word detection
2. STT (Speech-to-Text)
3. LLM processing with persistent memory
4. TTS with multiple voice personas
5. Web UI for interaction

### Key Features
- 11 built-in personalities with bundled prompt, voice, tools, model
- Semantic vector memory (100K+ entries)
- Heartbeat system (scheduled autonomous tasks — similar to OpenClaw!)
- Self-modification (AI edits its own prompt)
- Tool maker (AI writes and installs new tools at runtime)
- Interactive story engine
- Shell command execution
- Email, smart home integration

### Repo
- `github.com/ddxfish/sapphire` (⭐86)
- Discord community: `discord.gg/pCdTAnExma`
- Website: `sapphireblue.dev`

### Pros
- ✅ Very similar philosophy to OpenClaw/Jarvis (persistent AI companion)
- ✅ Full voice pipeline built-in
- ✅ Heartbeat system familiar to OpenClaw users
- ✅ Active development (weekly releases)
- ✅ Self-hosted, open source
- ✅ Web UI accessible from any device

### Cons
- ❌ Desktop/server focused — web UI works on mobile but not native
- ❌ No Android app
- ❌ Different architecture than OpenClaw (can't directly integrate)
- ❌ Heavy — requires GPU for some features
- ❌ Security warning: can execute shell commands autonomously

### Complexity: **Medium**
- Docker-based setup
- Well-documented

### Android Compatibility: **Via Web UI** ⚠️
- Access web UI from phone browser
- No native app, no wake word on phone

---

## Approach 8: EVA — Enhanced Voice Assistant

### How It Works
EVA is a Python-based multimodal voice assistant built on LangGraph:
1. Audio capture (microphone)
2. STT (configurable: Whisper, Google, etc.)
3. LLM processing (ChatGPT, Claude, Deepseek, Gemini, Grok, Ollama)
4. TTS response
5. Optional camera input for visual context

### Key Features
- Voice ID (recognizes who is speaking)
- Face recognition
- Proactive communication style
- Multilingual support
- Background multitasking
- Web interface (React, 2025 update)
- WebSocket support for remote control
- Can run fully local with Ollama

### Repo
- `github.com/Genesis1231/EVA` (active, updated March 2026)

### Pros
- ✅ Works with Claude (Anthropic integration built-in)
- ✅ Web interface works on mobile
- ✅ Can run fully local
- ✅ Voice ID is unique feature
- ✅ Proactive engagement model

### Cons
- ❌ Primarily desktop/server focused
- ❌ Web interface requires holding spacebar (not great for mobile)
- ❌ No Android app
- ❌ Complex Python setup

### Complexity: **Medium-High**
### Android Compatibility: **Via Web UI** ⚠️

---

## Approach 9: Rapida — Voice AI Orchestration Platform

### How It Works
Enterprise-grade platform for building voice agents:
- gRPC-based real-time audio streaming
- Pluggable STT, TTS, LLM, VAD
- Full observability (call logs, latency dashboards)
- Multi-channel integration

### Repo
- `github.com/rapidaai/voice-ai` (Go)
- Docs: `doc.rapida.ai`

### Pros
- ✅ Production-grade, scalable
- ✅ LLM-agnostic (OpenAI, Anthropic, open-source)
- ✅ Full observability

### Cons
- ❌ Enterprise-focused — way too heavy for personal use
- ❌ Requires 16GB+ RAM, Docker
- ❌ No direct Android client

### Complexity: **Very High**
### Android Compatibility: **Via API/Web** ⚠️

---

## Approach 10: Android Tasker/Automate → Webhook Workflows

### How It Works
Use Android automation apps to create voice-to-webhook pipelines:
1. **Tasker** or **Automate** listens for voice input (via Google STT or AutoVoice plugin)
2. Transcribed text sent via HTTP POST to your agent's webhook endpoint
3. Response received and spoken via TTS

### Implementation
```
Tasker Profile:
  Trigger: AutoVoice (or widget button)
  → Task: Get Voice (uses Google STT)
  → Task: HTTP Request POST to https://your-agent.com/webhook
  → Task: Say (TTS the response)
```

### Pros
- ✅ Fully customizable
- ✅ Can trigger from widget, NFC tag, or AutoVoice wake word
- ✅ Works with any HTTP API
- ✅ AutoVoice plugin supports always-listening mode
- ✅ Deep Android integration (can control other apps too)

### Cons
- ❌ Tasker has steep learning curve
- ❌ AutoVoice plugin costs ~$2-3
- ❌ Battery drain with always-listening
- ❌ Google STT requires internet
- ❌ Fragile — Android battery optimization can kill background services
- ❌ No standard solution — need to build custom flows

### Complexity: **Medium**
- Tasker setup is fiddly but well-documented
- Main work is building the HTTP webhook endpoint

### Android Compatibility: **Native** ✅✅

---

## Approach 11: Signal Voice Message Transcription

### How It Works
Signal has no bot API, so voice message transcription requires workarounds:

### Key Projects
- **signal-transcriber** (⭐7) — `github.com/FriedrichVoelker/signal-transcriber`
  - JavaScript, Docker-based
  - Uses signal-cli to receive messages, Whisper for transcription
  - Automatically transcribes and replies with text
- **TranSlander** (⭐5) — `github.com/hatsch/TranSlander`
  - Kotlin Android app
  - 100% local/offline speech-to-text
  - Voice typing in any app + transcribe voice messages from Signal
  - Uses on-device models

### Pros
- ✅ TranSlander works fully offline on Android
- ✅ Privacy-focused (Signal + local processing)

### Cons
- ❌ Signal has no official bot API — workarounds are fragile
- ❌ signal-cli is complex to set up
- ❌ Can't easily build a full conversational agent on Signal
- ❌ OpenClaw doesn't support Signal as a channel

### Complexity: **High**
### Android Compatibility: **TranSlander is native Android** ✅

---

## Approach 12: OpenAI Realtime API Voice Assistants

### How It Works
OpenAI's Realtime API provides WebSocket-based streaming voice:
1. Client streams audio via WebSocket
2. OpenAI handles STT + GPT-4o + TTS in one pipeline
3. Audio response streams back in real-time
4. Supports function calling, interruptions

### Key Projects
- **openai-realtime-api-voice-assistant-V2** (⭐127)
  - `github.com/Barty-Bart/openai-realtime-api-voice-assistant-V2`
  - RAG + function calling + caller history
- **JarvisV3** (⭐31) — Streamlit-based, inspired by Iron Man's Jarvis
- **langgraph-voice-call-agent** (⭐34) — LangGraph + LiveKit

### Pros
- ✅ Lowest latency possible (single round-trip to OpenAI)
- ✅ Natural conversational flow with interruptions
- ✅ Function calling support
- ✅ High quality voice

### Cons
- ❌ **OpenAI only** — doesn't work with Claude
- ❌ Expensive (realtime API pricing)
- ❌ Vendor lock-in
- ❌ Requires custom client app
- ❌ Not designed to forward to external agents

### Complexity: **Medium**
### Android Compatibility: **Via web client** ✅

---

## Approach 13: Google Assistant → Custom Webhook (Deprecated)

### How It Works
Google's Conversational Actions (formerly Actions on Google) allowed custom voice apps on Google Assistant. However:

> ⚠️ **Google shut down Conversational Actions in June 2023.** This approach is no longer viable.

The remaining option is **Google Assistant Routines**, which can trigger smart home actions but cannot forward voice to arbitrary webhooks.

### Verdict: **Not viable** ❌

---

## Approach 14: Willow — Open Source Voice Assistant Hardware

### How It Works
Willow runs on ESP32-S3 hardware (~$10-15) and provides:
- Wake word detection
- STT via Willow Inference Server (self-hosted) or cloud
- Integration with Home Assistant
- WebRTC support

### Repo
- `github.com/HeyWillow/willow`
- Website: `heywillow.io`
- Willow Inference Server: `github.com/toverainc/willow-inference-server`

### Pros
- ✅ Very cheap hardware ($10-15)
- ✅ Self-hosted inference server
- ✅ Always-on, dedicated hardware
- ✅ Low power consumption

### Cons
- ❌ Hardware-focused — not an Android solution
- ❌ Primarily for home automation
- ❌ Would need custom bridge to OpenClaw

### Complexity: **High**
### Android Compatibility: **Not applicable** — it's a hardware device

---

## Architecture Patterns Summary

### Pattern A: Messaging Voice Notes (Lowest Effort)
```
[Phone] → Voice Note → [Telegram/WhatsApp] → [OpenClaw] → Whisper STT → Agent → Text Reply
```
- **Effort:** Add voice message handler to existing OpenClaw channel
- **Latency:** 3-10 seconds
- **UX:** Familiar, async

### Pattern B: Web Voice Chat (Medium Effort)
```
[Phone Browser/PWA] → Web Speech API STT → HTTP → [Agent API] → TTS → [Audio Playback]
```
- **Effort:** Build simple web page + API endpoint
- **Latency:** 1-3 seconds
- **UX:** Real-time-ish, needs manual activation

### Pattern C: Native Android App (Higher Effort)
```
[Sanna App] → Wake Word → STT → [Claude/OpenAI API] → TTS → Speaker
```
- **Effort:** Build/configure Sanna, or build custom app
- **Latency:** 1-2 seconds
- **UX:** Best — wake word, hands-free, native

### Pattern D: Real-Time Voice Platform (Highest Effort)
```
[Client] → WebRTC Audio → [LiveKit/Rapida Server] → STT → LLM → TTS → WebRTC Audio → [Client]
```
- **Effort:** Deploy and configure a voice AI platform
- **Latency:** <1 second
- **UX:** Best quality, real-time conversation

---

## Recommendation for OpenClaw/Jarvis Setup

### 🏆 Phase 1: Quick Win — Telegram Voice Messages
**Do this first.** OpenClaw already supports Telegram. Add:
1. Voice message detection in the Telegram channel handler
2. Download `.ogg` file from Telegram API
3. Transcribe via Groq Whisper API (fast, free tier available) or OpenAI Whisper
4. Inject transcribed text into the regular message pipeline
5. Optionally: generate TTS response and send back as voice note

**Estimated effort:** 2-4 hours if OpenClaw's Telegram handler is accessible
**Result:** Voice input from any phone, anywhere

### 🥈 Phase 2: Custom Voice Chat PWA
Build a simple web page:
1. Web Speech API for continuous listening
2. POST transcribed text to OpenClaw's webhook/API
3. Play response via browser TTS or ElevenLabs
4. Install as PWA on Android home screen

**Estimated effort:** 1-2 days
**Result:** Near-real-time voice chat from phone browser

### 🥉 Phase 3: Evaluate Sanna Bot
Sanna explicitly positions itself as "OpenClaw for your phone":
1. Request test APK from `sannabot@proton.me`
2. Test voice interaction quality
3. Evaluate whether it can be configured to forward commands to Jarvis/OpenClaw
4. Consider contributing to make it a proper OpenClaw mobile companion

**Estimated effort:** 1-2 hours to test, days-weeks to integrate
**Result:** Full native Android voice assistant with wake word

### 🔮 Future: LiveKit for Production Voice
If voice chat becomes a daily workflow:
1. Set up LiveKit Cloud account (free tier)
2. Build a Python agent that bridges LiveKit ↔ OpenClaw
3. Use LiveKit's Android SDK or web client
4. Get production-grade real-time voice with turn detection

**Estimated effort:** 1-2 weeks
**Result:** Best-in-class real-time voice conversation

---

## Key Resources & Links

| Resource | URL |
|----------|-----|
| Sanna Bot (Android voice-first AI) | `github.com/sannabotdev/sannabotapp` |
| WhatsApp Voice AI Assistant | `github.com/cr2007/whatsapp-voice-ai-assistant` |
| Sapphire (self-hosted AI companion) | `github.com/ddxfish/sapphire` |
| EVA (Enhanced Voice Assistant) | `github.com/Genesis1231/EVA` |
| LiveKit Agents Framework | `docs.livekit.io/agents/` |
| Home Assistant Voice | `home-assistant.io/voice_control/` |
| Willow Voice Hardware | `github.com/HeyWillow/willow` |
| Rapida Voice AI Platform | `github.com/rapidaai/voice-ai` |
| OpenAI Realtime API V2 Example | `github.com/Barty-Bart/openai-realtime-api-voice-assistant-V2` |
| Signal Transcriber | `github.com/FriedrichVoelker/signal-transcriber` |
| TranSlander (offline Android STT) | `github.com/hatsch/TranSlander` |
| Telegram Bot API (Voice) | `core.telegram.org/bots/api#voice` |
| ChatGPT Voice (Web Speech API) | `github.com/GURU-DUTT/ChatGPTVoice` |

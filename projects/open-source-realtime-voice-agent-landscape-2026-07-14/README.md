# 開源即時語音 Agent 地圖：不是 STT + LLM + TTS，而是會「聽、想、說」的模型

日期：2026-07-14

## 先給結論

有，而且你描述得很精準：你要找的是以**連續語音互動**為第一級能力、可被打斷、最好能邊聽邊說的 realtime voice model；不是把 Whisper、文字 LLM、TTS 三個 API 接起來後才叫 voice agent。

截至本次調查，若目標是「開放權重、可自己研究與部署、中文／中英混用、未來可當 Jarvis Voice Lite 的前台」，我的排序是：

1. **MiniCPM-o 4.5**：最值得優先研究。9B、開放權重、英中即時語音、full-duplex 音訊／視訊串流、可本地 demo；它最接近「本地版 realtime GPT-4o/Gemini Live」的想像。
2. **Qwen2.5-Omni 7B / 3B**：成熟的 Apache-2.0 端到端 omni 模型，能聽、看、讀文字並串流輸出語音；很適合做中文、工具使用與端側可行性的對照組。
3. **Moshi**：研究上最純粹的 full-duplex spoken-dialogue 代表，延遲與自然插話是它的靈魂；但英文與硬體條件是 Jones 現階段的主要風險。
4. **Step-Audio2 / GLM-4-Voice**：中國團隊的端到端中文語音路線，值得列入實驗池；前者偏較新的 conversation/reasoning，後者已有清晰 streaming decoder 與 9B chat model。
5. **Ultravox**：開放、MIT、很適合研究「音訊直接進 LLM」；但目前開源模型的輸出核心仍是 streaming text，真正 speech output 仍需另接 TTS，因此不是首選的端到端語音嘴巴。

這裡的「開源」要精確說成 **open weights + 開源 code**。各模型權重、資料、商用權與可重現程度不一樣；不能因 GitHub repo 是 Apache/MIT 就假定 checkpoint 也完全同授權。

## 先分三個不同的東西

| 類別 | 它解決什麼 | 代表 | 是否是 Jones 真正要找的核心 |
|---|---|---|---|
| 原生／端到端 Speech-to-Speech | 直接以語音串流理解、推理、生成語音；turn-taking 是模型能力 | MiniCPM-o、Moshi、Qwen2.5-Omni、Step-Audio2、GLM-4-Voice | **是** |
| Audio-in LLM + 外接 TTS | 音訊不必先轉文字才進 LLM，但最後仍要 TTS | Ultravox | 部分是；容易做但不是全套 |
| Voice-agent runtime | 幫你接 WebRTC/WebSocket、VAD、打斷、tools、telephony、observability；可換任何模型 | LiveKit Agents、Pipecat | **是工程骨架，不是聲音大腦** |

最重要的觀念是：**原生 speech-to-speech 不代表它不能用 tools 或讀 JSON。** 工具層可以仍在模型旁邊；只是第一輪的聽、回應、插話、語氣與 endpointing 不必硬拆成三次網路往返。

## 候選模型比較

| 模型 | 為何它算 realtime voice | 中文／本地 | 適合 Jarvis 的位置 | 現實限制 |
|---|---|---|---|---|
| **MiniCPM-o 4.5** | 9B end-to-end；連續音訊／視訊輸入與文字／語音輸出不互相阻塞，full-duplex、可主動提醒 | 官方列英中 realtime conversation；有 GGUF/int4、llama.cpp、Ollama、local web demo | **第一優先**：可做 Voice Lite 的本地前台，再由 tool 升級到 Jarvis Deep | 9B omni 不是手機 1B；實測 Mac mini／GPU 的首音延遲、RAM、中文品質與工具調度 |
| **Qwen2.5-Omni 7B / 3B** | Thinker-Talker 架構；chunked input、immediate speech/text output | Apache-2.0；3B、4-bit、MNN edge path，中文生態強 | **最佳對照組**：多模態、中文、可做 tool routing | 論文／模型能力不等於 production full-duplex 體驗；需自己補 session、VAD、barge-in 與安全層 |
| **Moshi** | 真正雙音訊 stream；模型同時表徵使用者與助理語音；官方稱理論 160ms、L4 約 200ms | 有 MLX（Mac/iPhone）、Rust production stack；權重 CC-BY 4.0 | **研究標竿**：學 full-duplex / interruption 的感覺 | 7B temporal transformer；英文導向，中文不是已證實主場；不是現成 agent/tool product |
| **Step-Audio2 / 2-mini** | 官方定位 end-to-end speech conversation；另有 Speech Reasoning 路線 | 中國團隊、Apache-2.0 repo、權重／推論範例公開 | **中文研究池**：若要語音 reasoning 很值得 A/B | 生態與英文 docs 的工程成熟度、手機部署與 full-duplex UX 要自己驗證 |
| **GLM-4-Voice** | 9B chat + tokenizer + streaming speech decoder；最少 10 speech tokens 即可開始 decoder | 端到端中英語音對話，repo Apache-2.0 | **中文可行備選**：易於研究 streaming decoder 設計 | 發表較早；互動、工具與長 session 的產品化仍要自己搭 |
| **Ultravox** | 音訊直接進多模態 LLM，不先跑獨立 ASR；低延遲 text output | MIT；有 8B checkpoint | **語音理解／router**，或與本地 TTS 組合 | README 明示目前輸出是 streaming text，尚非開源端到端 speech-output 模型 |

## MiniCPM-o 4.5：這次最值得打開的 repo

它不是 MiniCPM5-1B 的語音版。MiniCPM5-1B 是小型文字模型；**MiniCPM-o 4.5 是 9B 的 omni 模型**，基於 SigLip2、Whisper-medium、CosyVoice2、Qwen3-8B，目標就是「看、聽、說可以同時發生」。官方資料稱它可英中即時語音對話、支援 configurable voice / reference-audio voice cloning，並開放 local full-duplex web demo。[OpenBMB README](https://github.com/OpenBMB/MiniCPM-V)

這剛好命中 Jarvis Voice Lite 的核心問題：不是把完整 Codex/Jarvis 每一句都塞進語音 loop，而是讓本地 voice brain 處理：

```text
連續麥克風
  -> MiniCPM-o 4.5（聽、自然接話、短問答、判斷要不要升級）
  -> 簡單問題：直接語音回答
  -> 查旅行／記憶／做事：受控 tool -> Jarvis Deep / Trip Planner
  -> MiniCPM-o 用語音把工具結果講回來
```

它不是保證比雲端 realtime model 更聰明或更快；它的價值是**可研究、可留在本機、可保留 voice interaction 的主控權**。先做單機可行性驗證，不要先碰產品 cutover。

## 兩個真正的開源 agent 工程底座

### LiveKit Agents

[LiveKit Agents](https://github.com/livekit/agents) 是 Apache-2.0 的 realtime programmable participant framework。它能混搭 STT、LLM、TTS 或接 realtime API，並有 semantic turn detection、WebRTC 與原生 app 的工程路線。它適合把 MiniCPM/Qwen/Moshi 或雲端模型接進同一個可測的 transport、tool、telemetry 骨架。

### Pipecat

[Pipecat](https://github.com/pipecat-ai/pipecat) 是 BSD-2-Clause 的 Python realtime voice/multimodal framework，支援 WebRTC/WebSocket、VAD、串流 pipeline、多 agent handoff，也有多家 STT/LLM/TTS 與多種 speech-to-speech provider adapter。它較適合快速做很多組 pipeline 對照；但不要誤會它本身是一個聲音模型。

## 對 Jarvis 的務實研究順序

### Phase A：先量「像不像聊天」而不是先接所有能力

用相同 20 題、相同麥克風與網路，測 MiniCPM-o 4.5、Qwen2.5-Omni 3B/7B、現行雲端 Voice Lite：

- 使用者停話到第一個可聽音節（不是 final transcript）
- 可否自然 barge-in；被打斷後是否停得乾淨
- 英中混用、台灣中文專有名詞、短命令的理解
- 30 秒自然聊天中是否搶話、卡住或重複
- 單輪 tool result 後能否正確用口語摘要
- RAM / VRAM、CPU/GPU、耗電與每小時成本

成功門檻可先訂成：簡單閒聊 first audible < 2 秒、工具查詢 < 5 秒、20 次中斷至少 18 次不殘留舊語音。不要用「文字 final」當 realtime 語音的唯一指標。

### Phase B：只接一個唯讀工具

第一個工具建議是 `get_trip_summary` 或 `get_day_schedule`，只讀 Trip Planner 的已發佈資料。讓模型練習：

1. 聽懂「明天幾點出發？」
2. tool call。
3. 把結構化結果講成一句自然中文。
4. 被打斷時停止，不造成第二次寫入。

在此之前，不接 repo 寫檔、部署、Slack 發送或記憶寫入。Voice 前台該是很快的口耳，不是繞過既有安全邊界的 root agent。

### Phase C：才決定本地或雲端的分工

- 若 MiniCPM-o 4.5 的自然度、延遲、中文與硬體成本都過線：它可作 local/offline-first Voice Lite，雲端作 difficult-case fallback。
- 若 Qwen2.5-Omni 在可用硬體上更穩：保留同一個 LiveKit/Pipecat tool contract，換 model 不換產品邊界。
- 若開源端到端模型仍達不到「像電話」的體感：雲端 OpenAI Realtime / Gemini Live 仍可當體驗 benchmark；不用把模型主控權與 agent 工具主控權綁成一件事。

## 一句話判斷

**有你要的東西，而 MiniCPM-o 4.5 是目前最值得先研究的開放權重候選。** 它不是完整 Jarvis 的替代品；它更像一個可能終於能當 Jarvis「嘴巴、耳朵與即時反射」的本地大腦。真正需要深度記憶、長推理與執行的部分，仍應以明確工具邊界交給 Jarvis Deep。

## 一手來源

- [MiniCPM-V / MiniCPM-o 官方 repo](https://github.com/OpenBMB/MiniCPM-V) — MiniCPM-o 4.5 9B、英中 voice、full-duplex、local demo／formats。
- [Qwen2.5-Omni 官方 repo](https://github.com/QwenLM/Qwen2.5-Omni) — Thinker-Talker、3B/7B、streaming speech、Apache-2.0。
- [Kyutai Moshi 官方 repo](https://github.com/kyutai-labs/moshi) — full-duplex 架構、MLX/Rust stacks、延遲說明與權重授權。
- [Step-Audio 官方 repo](https://github.com/stepfun-ai/Step-Audio) 與 [Step-Audio2](https://github.com/stepfun-ai/Step-Audio2) — end-to-end speech conversation 與推論資源。
- [GLM-4-Voice 官方 repo](https://github.com/zai-org/GLM-4-Voice) — 9B chat、tokenizer、streaming decoder。
- [Ultravox 官方 repo](https://github.com/fixie-ai/ultravox) — audio-in LLM 的目前輸出邊界與 MIT 授權。
- [LiveKit Agents](https://github.com/livekit/agents) / [Pipecat](https://github.com/pipecat-ai/pipecat) — 開源 voice-agent runtime，而非 foundation voice model。


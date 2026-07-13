# SpaceXAI語音助理介紹

Source: https://chatgpt.com/share/6a556e7a-b87c-83ea-a445-f1e9b191196f
Captured: 2026-07-13T23:05:18.003Z
Extractor: react-router-data-model
Completeness: high
Included: visible User messages and visible GPT assistant text/code/commentary/final messages
Excluded: system/developer context, hidden redacted nodes, reasoning/thought summaries, tool outputs, metadata-only nodes
--- User 01 ---

## 01. User

[Image uploaded: sediment://file_00000000eb5c720ca79e6671534bf9d8?shared_conversation_id=6a556e7a-b87c-83ea-a445-f1e9b191196f]

SpaceX AI發了這個給我，請問這是啥？語音模型還啥嗎？我看它說免費可用？

<!-- source_node_index: 6; content_type: multimodal_text; id: bbb2146a-f098-41a1-93fb-371e920a03ca -->

--- GPT 02 (final) ---

## 02. GPT (final)

這不是單純「新的語音模型」，而是 **SpaceXAI（原 xAI）推出的完整 Voice Agent Builder**。你可以把它理解成：

> **用 Grok Voice 做一個可以接電話、跟人即時語音對話、查資料、執行工具的 AI 電話客服／語音助理平台。**

SpaceXAI 是最近才把原本的 xAI 品牌改名過來，所以這封信其實就是 Grok 團隊寄的，不是另一家山寨公司。citeturn687603search0turn687603search3

## 它實際上能做什麼？

你可以在網頁上設定：

- Agent 的角色、個性與對話規則
- 使用哪個聲音和講話速度
- 上傳 PDF、Word、Markdown 等知識文件
- 接 Google Calendar、Gmail、Notion 等工具
- 串接自己的 API 或 **MCP server**
- 配一個美國電話號碼，讓真人直接打電話進去
- 對話途中轉接真人，或轉交給另一個專門 Agent

底層使用的是 **Grok Voice 的 speech-to-speech 模型**，不是傳統那種「語音轉文字 → LLM → 文字轉語音」三段各自拼起來的形式，所以它主打較低延遲、可以打斷、自然插話及即時反應。citeturn309308search0turn309308search4

例如可以做成：

- AI 診所櫃台，接電話並預約
- 商店客服，查訂單狀態
- AI 銷售代表，篩選潛在客戶
- 餐廳訂位助理
- 個人語音助理
- 能呼叫你自訂 MCP 的 Jarvis 語音入口

## 信裡的「21 new voices」是什麼？

是新增了 **21 種高品質預設聲音**，不代表發布了 21 個不同的 LLM。

大致上是同一套 Grok Voice Agent 能換不同：

- 音色
- 性別／聲線
- 說話風格
- 適合客服、銷售或一般助理的語氣

它也支援自訂發音、調整語速，以及使用者安靜太久時主動追問。

## 「Try It Free」等於完全免費嗎？

**不是長期完全免費使用。**

這裡的免費主要有兩層意思：

1. 可以免費進入 Builder 建立、設定和測試 Agent。
2. 官方可以免費配給一個美國電話號碼。

但實際讓 Agent 長時間通話，通常會按使用量收費。官方發布資訊顯示，目前語音 Agent 約為：

- **AI 語音：US$0.05／分鐘**
- 電話線路費可能另外計算，第三方資料顯示約 **US$0.01／分鐘**
- 預設聲音已包含，不另外收平台月費

所以一次 10 分鐘的美國電話，大概可能是 **US$0.60 左右**，而不是無限免費。citeturn309308search1turn309308search3turn309308search6

你信裡寫的 **“with a US phone number on us”**，比較精確的意思是「電話號碼本身送你」，不是它幫你負擔所有通話分鐘費。

## 這跟你正在想的 Jarvis 語音功能有什麼關係？

其實**高度相關**，但定位不同。

你原本打算做的是：

> Android 按下語音 → Sparkfield/Jarvis bridge → Codex/Jarvis → 串流語音回覆

SpaceXAI Voice Agent Builder 則是：

> 電話或 WebSocket 語音 → Grok Voice → MCP/API/工具 → 即時語音回覆

它可以讓你很快測試「與 Jarvis 自然語音對話」的感覺，尤其它現在支援 **custom MCP**。理論上，你甚至有機會把某個 MCP gateway 接到自己的 Jarvis runtime，再讓 Grok Voice 當語音前台。

但它不一定能直接取代你正在做的 repo，因為：

| 你的 Jarvis 語音窗口 | SpaceXAI Voice Agent Builder |
|---|---|
| 完全自己控制 UI 與流程 | 使用對方的平台與介面 |
| 後端可以是 Codex/Jarvis | 核心對話模型是 Grok Voice |
| 適合自用 Android 助理 | 更偏客服、電話與企業場景 |
| 可以深度接你的既有記憶與架構 | 需透過文件、工具或 MCP 接入 |
| 自己管理成本和延遲 | 按分鐘付費，但開箱即用 |

因此我會把它看成一個非常好的 **Jarvis 語音互動原型／比較基準**：你可以先免費建一個 Custom Personal Assistant，測試它的延遲、打斷能力、中文表現與 MCP 能力，再決定哪些體驗值得搬回你自己的 `jarvis-ai-bridge` 語音 repo。

<!-- source_node_index: 14; content_type: text; id: bb305bde-7034-45a2-84fb-b8ce99bb5b42 -->

--- User 03 ---

## 03. User

但如果我們暫時不想接通Jarvis 的記憶，想換取絲滑體驗跟反應速度，反應可能應該選它？

<!-- source_node_index: 15; content_type: text; id: bbb21146-5476-4bfa-94af-04bd4b60f7dc -->

--- GPT 04 (final) ---

## 04. GPT (final)

**對，若你們目前願意暫時放棄 Jarvis 的長期記憶、Codex 能力與完整工具鏈，只想先得到「像真人一樣立即對話」的體驗，SpaceXAI Voice Agent 很可能是更適合的第一步。**

它官方定位就是即時 **speech-to-speech**，透過 WebSocket 雙向串流語音，標示延遲為 **sub-second**。這通常會比你們自己串：

> Android 錄音 → STT → bridge → Codex/Jarvis → TTS → 播放

更絲滑，因為少了多次模型與網路服務之間的交接。citeturn591954search1turn591954search5

## 但要分清楚「絲滑」和「聰明」

它可能在這些地方明顯勝出：

- 你講完後很快接話
- 能偵測你是否真的講完
- 可以自然插話、打斷
- 語音節奏和情緒較連貫
- 不必先等完整文字生成完才開始念
- 直接使用 WebSocket 串進 Android App

官方也有 Smart Turn 類型的語音終點判斷，這正是決定「會不會一直搶話，或等太久才回答」的重要部分。citeturn591954search17

但它未必在以下方面勝過現在的 Jarvis：

- 深入理解你所有 repo 的狀態
- 記得你過去幾個月的決策
- 執行複雜 Codex 工作
- 讀 Slack、專案檔案與既有 distillation
- 長篇技術推理與工程操作

所以它比較像是：

> **非常好的嘴巴、耳朵和即時對話大腦，但暫時不是完整的 Jarvis 人格與記憶。**

## 我覺得最合理的路線

不要現在就把它硬接進完整 Jarvis。先做一個非常乾淨的實驗版：

```text
Android App
   ↓ 麥克風音訊串流
SpaceXAI Voice Agent API
   ↓ 即時語音回傳
手機喇叭
```

只給它一份簡單的 system prompt，例如：

```text
你是一個名為 Jarvis 的私人語音助手。
使用自然、簡短的繁體中文回應。
使用者經常中英文混用，必須自然理解。
不要每次重述問題。
除非有必要，回答控制在 1–3 句。
可以被使用者隨時打斷。
```

先測四件事：

1. 中文與中英混合辨識。
2. 回應第一個音節需要多久。
3. 能不能自然打斷。
4. 連續聊 10 分鐘會不會覺得像打電話，而不是對著語音機器人下命令。

這樣你們大概一天內就能知道，它是否真的有「阿斯拉感」。

## 成本其實也很適合原型

官方目前 Voice Agent API 是 **US$0.05／分鐘，也就是每小時 US$3**，音訊傳入或輸出都依音訊時間計費；純文字事件則是每則 US$0.004。citeturn591954search2

假設每天聊：

| 每日使用量 | 約每月費用 |
|---:|---:|
| 10 分鐘 | US$15 |
| 20 分鐘 | US$30 |
| 30 分鐘 | US$45 |
| 60 分鐘 | US$90 |

這裡仍要看官方對「音訊送出或接收」的實際累計方式；文件寫的是 sent or received，因此一段雙方都有聲音的長對話，帳單可能不是單純拿牆上時間乘 $0.05。最安全是實測一次 10 分鐘 session 後查看實際 usage。citeturn591954search2

而信中的 **Try It Free** 比較像允許你先進 Builder／Playground 測試，不能理解為無限免費。官方 quickstart 仍寫明 API 使用需要帳戶載入 credits。citeturn591954search11

## 最有意思的是：以後不必二選一

最終其實可以變成：

```text
一般聊天
→ SpaceXAI Voice 直接回答
→ 快、自然、低延遲

需要記憶或執行工作
→ 呼叫 Jarvis tool / MCP
→ Jarvis 回傳結果
→ SpaceXAI 用語音說出來
```

例如你說：

> 「Jarvis，今天先聊聊我對這個產品的想法。」

由 Grok Voice 直接陪你聊。

但你說：

> 「把剛剛的結論存進 Spark Distiller，順便在 Slack 告訴 Jarvis。」

它再呼叫你自己的 MCP 或 bridge。官方 Voice Agent 本身已支援 custom functions、MCP、web search 和其他工具，因此這條路在架構上是成立的。citeturn591954search2

**所以我現在的判斷是：先用它做 Jarvis 的 Voice Lite 版本，非常合理。** 先取得真正絲滑的語音體驗，再逐步把 Jarvis 的記憶和能力用工具方式接回去，比你們一開始自己打造整套即時語音底層更划算。

<!-- source_node_index: 22; content_type: text; id: 7794d2eb-42e1-4b33-9241-2ec96ecc9e36 -->

--- User 05 ---

## 05. User

所以我是否可以這麼猜測理解，Speech to Speech 的模型可以是強在語音的反應速度跟反饋，但是可能推理跟agentic的部分不會比一般第一梯隊的模型強？

例如我如果想要做成我語音跟Jarvis對話讓他直接去讀後端的旅行json數據回答我們有關旅行的細節問題甚至建議、找資料，可能Speech to Speech 做不到？

<!-- source_node_index: 23; content_type: text; id: 27c8d842-7933-4b98-b32d-96686f43435f -->

--- GPT 06 (final) ---

## 06. GPT (final)

你的理解**大方向是對的**，但有一個重要修正：

> **Speech-to-Speech 不代表不能讀 JSON、不能用工具、不能找資料。**  
> 它通常只是把「即時語音互動」放在最高優先級，因此在深度推理、長時間規劃與複雜 Agent 工作上，未必等同於最強的文字推理模型。

## 可以把能力拆成三層

| 能力 | Speech-to-Speech Voice Agent | 第一梯隊推理／Agent 模型 |
|---|---|---|
| 即時聽說、插話、停頓判斷 | **很強** | 通常需額外 STT/TTS |
| 一般問答、簡單判斷 | 足夠強 | 強 |
| 讀旅行 JSON、查行程資料 | **可以，需接工具** | 可以 |
| 網路搜尋、查景點資料 | **可以，需啟用搜尋工具** | 可以 |
| 多步研究、比較大量資料 | 能做，但未必最穩 | 通常更強 |
| 修改行程、跨系統操作 | 能做，但需 function/MCP | 通常更適合做主 Agent |
| 長篇深度推理、複雜 coding | 未必是強項 | **更強** |

xAI 現在的 Voice Agent API 明確支援 **custom functions、Remote MCP、Web Search、X Search 和 Collections Search**，所以它並不是只有一個會講話的模型。它可以在語音對話中決定呼叫後端工具，再根據工具結果用語音回答。citeturn511624search1turn511624search5turn511624search9

---

# 你的旅行 JSON 案例，它其實做得到

例如你問：

> 「我們住 Page 的飯店是哪間？第二天早上幾點去下羚羊谷？」

Voice Agent 本身不會天然知道，但可以呼叫：

```text
get_trip_plan()
get_day_schedule(date)
get_booking_details(booking_id)
```

你的後端工具再去讀：

```text
trip.json
```

然後回傳精簡結果：

```json
{
  "hotel": "Garner by IHG Page at Lake Powell",
  "activity": "Ken's Tours Lower Antelope Canyon",
  "startTime": "09:30"
}
```

Voice Agent 拿到結果後直接回答：

> 「你們住 Garner by IHG Page at Lake Powell。第二天早上 9:30 是 Ken’s Tours 的下羚羊谷行程，建議最晚 8:45 從飯店出發。」

這完全是在它目前支援範圍內。官方的 function calling 就是讓模型要求你的程式執行資料庫查詢、API 呼叫或自訂商業邏輯，再把結果送回模型繼續回答。citeturn511624search11turn511624search4

---

## 找即時資料也能做

例如你問：

> 「我們明天去 Horseshoe Bend，日落時間是多少？會不會太熱？」

它可以：

1. 從你的旅行 JSON 找日期與地點。
2. 使用 Web Search 或你自己的 weather API。
3. 綜合行程時間。
4. 用語音建議是否調整。

xAI 的 Voice Agent 本身支援 Web Search，而且官方定位就是低延遲 speech-to-speech 加工具使用。citeturn511624search12turn511624search17

所以不是「Speech-to-Speech 做不到」，而是：

> **能不能做，取決於你給它什麼工具；做得多深，則取決於 Voice 模型本身的推理與工具協調能力。**

---

# 真正的限制是在複雜度，而不是資料存取

### 很適合直接交給 Voice Agent

- 「今天下午安排了什麼？」
- 「飯店離下羚羊谷多遠？」
- 「我們在哪一天去大峽谷？」
- 「把晚餐延後半小時。」
- 「附近找一家還營業的餐廳。」
- 「這兩個景點哪個比較順路？」

### 可能不適合只靠 Voice Agent

- 「重新最佳化完整七天路線，兼顧氣溫、票價、開車時間、日落、訂位與每個人的偏好。」
- 「閱讀二十份景點資料，交叉比對可靠性後重新設計三套方案。」
- 「修改多個 JSON、驗證 schema、跑測試、提交 Git commit。」
- 「研究各州道路封閉與即時天氣，再評估完整風險。」

這類任務需要較長的思考鏈、多次工具呼叫與結果驗證。Voice Agent 可能可以嘗試，但更強的文字 Agent 通常會更穩。

---

# 最適合你的架構可能是「Voice 主介面，不一定是唯一大腦」

```text
你說話
   ↓
SpaceXAI Voice Agent
   ├─ 簡單聊天／簡單行程問答
   │      → 直接回答
   │
   ├─ 旅行資料查詢
   │      → trip-planner MCP / function
   │      → 讀 trip.json
   │      → 語音回答
   │
   ├─ 即時資訊
   │      → Web Search / Weather API
   │      → 語音回答
   │
   └─ 複雜任務
          → delegate_to_jarvis(...)
          → Jarvis／Codex 深度處理
          → 回傳摘要
          → Voice Agent 說出來
```

這樣它平常依然能非常快：

> 「明天第一站是哪裡？」  
> 幾乎立即讀 JSON 回答。

只有遇到複雜需求才轉交：

> 「重新規劃明後兩天，避開高溫又不能影響下羚羊谷預約。」

這時 Voice Agent 可以先自然回一句：

> 「好，我正在重新比較路線、氣溫和預約限制。」

然後後端交給較強的 Jarvis Agent 處理。

---

## 對你目前階段的具體判斷

你甚至**不必先接完整 Jarvis 記憶**。第一版只需要做一個很薄的旅行 MCP：

```text
get_trip_summary
get_trip_days
get_day_schedule
get_place_details
get_booking
search_trip_items
```

Voice Agent 就已經可以回答大部分旅行中的自然語音問題。

因此更準確的結論是：

> **Speech-to-Speech 可以做你的旅行語音助理，而且可能非常適合。它的弱點不是無法讀 JSON 或呼叫工具，而是面對複雜、長時間、需要反覆驗證的 Agent 任務時，未必應該讓它單獨負責到底。**

第一版可以先做成 **SpaceXAI Voice + trip-planner JSON tools**，甚至先完全不接 Jarvis。等實測發現哪些問題它處理不好，再加一個 `ask_jarvis` 工具作為升級路徑。

<!-- source_node_index: 31; content_type: text; id: 49882f74-7878-4eb2-9529-b26b480dc566 -->

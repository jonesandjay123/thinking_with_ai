# Pixel 本地 Agent 地圖：從 Gemini Nano 到口袋裡的可執行代理

日期：2026-08-30
研究範圍：Android／Pixel 上「模型在本機」或「手機是 agent 身體」的現況；來源以官方文件與各專案一手 repo 為主。

## 先講結論

Pixel 11 Pro 的價值不只是在沒有網路時還能聊天。若機上的 Gemini Nano 4 已可由系統 API 使用，它很適合當一個**極快、私密、低成本的第一層大腦**：理解意圖、摘要、改寫、描述照片、決定是否需要進一步動作。真正把它變成 agent，還要加上工具、記憶、權限與「哪些事必須先問人」的控制面。

眼下最值得注意的不是單一 App，而是三條不同路線：

1. **系統整合型：Gemini Nano / AICore / ML Kit GenAI。** 最穩、最省電，也最貼近 Pixel；適合把本地能力嵌進自己寫的 Android app。它不是任意下載權重、任意跑 agent loop 的通用沙盒。
2. **本地 agent 實驗型：Google AI Edge Gallery。** 官方開源 beta，已把本地模型、Agent Skills、影像問答、音訊轉寫，以及由 FunctionGemma 270m 驅動的離線 Mobile Actions 放進同一個手機 App。這是最快能摸到「手機自己做事」感覺的入口。
3. **可拼裝型：PocketPal AI / Termux / 自建 app。** PocketPal 已有本地 GGUF、on-device TTS 與 tool-use loop；Termux 則能把手機變成一台小 Linux 主機。自由度最大，但續航、背景存活、Android 權限與安全設計都由自己承擔。

OpenClaw 應放在另一格理解：它的 Android companion/node 能讓手機提供鏡頭、語音、螢幕與 device-local actions，但 Gateway 和主要 agent 不必在手機上。因此它非常適合做「手機是 Jarvis 的身體」，卻不等於「整個 agent 和模型都離線塞在手機」。最有意思的未來架構反而是兩者混合。

## 先把名詞切開

| 層次 | 問題 | Pixel 上的例子 |
|---|---|---|
| 本地模型 | 資料與推論是否離開手機？ | Gemini Nano、Gemma、GGUF 模型 |
| Agent loop | 模型能否呼叫工具、讀結果、再決定下一步？ | Edge Gallery Agent Skills、PocketPal Talents |
| 手機身體 | agent 能否看螢幕、操作 app、用相機／定位／通知？ | Mobile Actions、Mobilerun、OpenClaw node |
| 控制與安全 | 哪些行為要預覽、確認、記錄或禁止？ | Android permission、allowlist、approval UI |

很多「手機 AI agent」展示只完成後兩層之一。例如單純本地聊天是模型，不是 agent；能自動點手機 UI 但每一步把截圖送雲端，也不是完整本地 agent。

## 現在可關注的專案與路線

以下是依產品型態整理，而不是把不同成熟度硬排成同一張排行榜。星數與更新時間是 2026-08-30 以 GitHub API 讀取的當下快照，僅作社群活躍度線索，不代表品質或安全性。

### 1. Google Gemini Nano + AICore + ML Kit GenAI：Pixel 的原生能力層

Android Developers 說明，Gemini Nano 透過 AICore 在裝置端執行，並提供系統整合的安全機制與硬體加速；ML Kit GenAI 則把部分能力封成較高階的 Android API。公開文件列出的方向包括摘要、校對／改寫、圖像描述等。對已有 Gemini Nano 4 的 Pixel，這是最接近系統能力、也最應優先採用的本地推論入口。

它適合做：

- 通知、剪貼簿、會議筆記或語音逐字稿的本地摘要與改寫。
- 相簿／相機畫面的本地描述，先產生結構化 observation，再決定是否升級到雲端。
- 一個受控的 intent router：`只回答`、`建立草稿`、`查資料`、`需要人工確認`。

它不天然提供：長駐 daemon、任意模型權重、通用 shell 權限、跨 app 無限制自動操作，也不會替 app 做好記憶與工具安全。把 AICore 當作 **on-device inference service**，不是「已安裝好的 OpenClaw」。

### 2. Google AI Edge Gallery：最直接的本地 agent 遊樂場

[AI Edge Gallery](https://github.com/google-ai-edge/gallery) 是 Google AI Edge 的 Apache-2.0 開源 beta，Android 12+ 可用。官方 README 現列出：模型下載／管理與 benchmark、Gemma 4、Thinking Mode、Ask Image、Audio Scribe、Prompt Lab、Agent Skills，以及 Mobile Actions。後者明確標示為以 FunctionGemma 270m finetune 驅動的離線裝置控制與自動化任務。

這裡特別有意思的是 **Agent Skills**：它讓本地 LLM 不只回文字，而能以工具取得資料、看地圖或產生視覺摘要；README 也提到可由 URL 載入模組化 skill、從 Discussions 瀏覽社群貢獻。它仍是 beta／實驗場，不應直接賦予帳號、付款、傳訊或刪檔的高權限。

很實用的第一輪玩法：

1. 在同一支 Pixel 分別 benchmark 小模型與較大模型，記錄 tokens/s、RAM、熱度與耗電，而不是只看網路 benchmark。
2. 用 Ask Image 做「看圖產生待辦」：拍收據、白板、櫃子或旅行用品，先把結果輸出成 JSON 草稿，不自動寫入任何服務。
3. 用 Audio Scribe 把一段語音轉成摘要／行動清單；敏感錄音可維持在本機。
4. 只在一個無害測試 app 中試 Mobile Actions，觀察 function routing 是否穩定，再談跨 app 自動化。

### 3. PocketPal AI：真正已包好 tool loop 的離線聊天／小代理

[PocketPal AI](https://github.com/a-ghorbani/pocketpal-ai) 是 MIT 授權、iOS／Android 都可用的開源 app（快照約 8.1k stars）。它在手機本地跑 GGUF 模型，底層採 llama.cpp；並已具備 on-device TTS、可設定 persona 的 Pals，以及 `AgentRunner` 驅動的 Talents/tool loop。官方列出的內建 Talents 包含計算、日期時間與 HTML rendering。

它比較像「可離線運作、能裝人格與小工具的口袋助手」，不是完整作業系統控制器。它的價值在於：不必先寫 Android native code，就可研究模型的 tool calling 是否可靠、哪種模型在 Pixel 實際上順、以及 persona + local TTS 的體感。其 README 也明說可以從 Hugging Face 下載或從本地載入模型，並在手機上測 tokens/s 與記憶體。

可延伸的騷操作不是讓它直接操作銀行或社群帳號，而是新增**純資料／可驗證**工具：本地日誌檢索、QR／收據解析後的規則計算、旅行清單檢查、或把自然語言轉成待確認的 Calendar／Task 草稿。

### 4. OpenClaw Android node：手機作為 agent 的感官與手腳

[OpenClaw](https://github.com/openclaw/openclaw) 的官方說明把 Gateway 定義為本地 control plane，而 companion apps／nodes 為語音、Canvas、相機、螢幕和裝置端動作提供能力。它可接 hosted 或 local model provider，也有工具、skills、plugins 與 messaging channels。

對 Jones 的情境，這代表一個很清楚的分工：

```text
Pixel Nano（本地快速判斷／隱私資料）
  -> 小且受控的 Android tools
  -> 必要時把「已整理、最小化」的任務交給
OpenClaw Gateway（長記憶、跨服務、重推理、審計）
  -> 回到 Pixel 顯示、說話、拍照、確認
```

優點是模型、記憶與跨服務能力不用全壓在手機；代價是 offline 時只剩 Pixel 本地那一層。這不是缺點，而是很健康的 graceful degradation：沒網路仍能記事、理解、整理、草擬與控制 allowlist 內的本機功能；有網路時才升級到完整 Jarvis。

### 5. Mobilerun（原 DroidRun）：手機 UI 自動化的 agent framework

[Mobilerun](https://github.com/droidrun/mobilerun) 是 MIT 授權的開源 Android／iOS 控制框架（快照約 9.2k stars）。它用 accessibility tree、截圖、點擊、滑動、輸入與多步規劃來完成自然語言任務；provider 清單包含 Ollama，並能在本機執行 framework。Android 端需要 Portal app、Accessibility service 與 ADB／開發者選項。

這讓它可和本地模型串接，但要精確說：README 的「run the agent on your machine」主要是指另一台執行 CLI／Python 的主機，手機是受控目標；並非預設一個完全自足、只靠手機 NPU 的 app。它適合研究「讓 agent 操作原生 app」的工具層與測試／QA，不適合一開始就當個人主手機的全權助理。

最大的風險在於 Accessibility 權限非常強，加上視覺模型偶爾誤判 UI。任何涉及傳訊、付款、發文、刪除、登入、雙因素驗證的流程都應強制停在 preview／確認畫面。

### 6. Termux + 自建本地 agent：最自由，也最不像產品

[Termux](https://github.com/termux/termux-app) 在 Android 上提供 Linux environment，並有 API、Boot、Tasker、Widget 等外掛。把它搭配本地推論 runtime、檔案／通知／Tasker 腳本與一個小型 tool router，就能做出相當野的「口袋伺服器」：離線 RAG、資料夾整理、定時摘要、區網工具、或由手機輸入觸發的工作流。

但 Termux 官方也明列 Android 12+ 可能因 phantom process／高 CPU 而殺掉背景程序。這條路的正確定位是 maker lab，不是保證可靠的 24/7 agent host；長任務要有 checkpoint、重試、電量策略與明確的資料備份。

## Pixel 上真正有意思的本地玩法

以下不是現成 App 功能承諾，而是可用上述元件組出的產品／原型方向。共同原則是：本地模型先產生結構化「建議或草稿」，有外部副作用的操作仍要由使用者確認。

### A. 離線 Context Capsule

手機在背景收集**使用者明確選取或分享給它**的資料：一段語音、照片、截圖、網址、收據。Nano／本地模型立即轉成：摘要、標籤、人物／地點／時間、待查問題與 `needs-cloud` 旗標。等 Wi-Fi 或確認後，才同步到 OpenClaw／個人知識庫。

關鍵好處不是「全記錄」，而是把生活輸入先變成可搜尋的最小膠囊，同時避免原始敏感資料不必要地離開手機。

### B. 相機是 agent 的 eyes，但先當觀察員

用本地視覺理解把畫面轉成 observation：

- 冰箱／衣櫃／工具箱盤點，產生缺少物品的清單草稿。
- 收據或停車標誌辨識，抽出金額／限制／截止時間。
- 旅行時拍招牌或菜單，先離線翻譯與結構化；遇到不確定內容才詢問雲端。

先存 observation 與置信／不確定性，而不是讓模型直接「下單」或「導航」。這能保留可追溯性，也能把誤判的損害限制在草稿層。

### C. 兩段式語音代理

第一段完全本地：VAD／短語音、Nano 意圖判斷、固定工具（計時、筆記草稿、離線清單、裝置狀態）。第二段只在需要時升級到 OpenClaw 或雲端，並把敏感原文先裁掉或摘要化。

這會比「每句話都送遠端 full agent」更像一個一直在身邊的裝置，也比嘗試把長記憶與所有 connector 都塞進手機務實得多。

### D. 小模型做 policy gate，不做唯一決策者

FunctionGemma／Nano／小 GGUF 模型很適合把自然語言映射到有限 JSON action：

```json
{"action":"create_note_draft","title":"修車","body":"明早打電話","requires_confirmation":false}
```

但工具端必須自己驗證 schema、allowlist、參數範圍與 confirmation；不要因為模型輸出 `send_message` 就直接傳送。這種設計同時使 offline 操作快，也避免小模型 hallucination 變成真實副作用。

### E. Personal redaction gateway

在任何雲端請求前，本地模型先做 PII／敏感欄位辨識與替換，例如把地址、帳號、臉部影像或原始錄音改成抽象摘要。它不是絕對防漏機制，卻是比「所有資料直接上雲」更好的預設層；對不想離開 Pixel 的內容，則根本不發送。

## 一個實際可走的實驗順序

### 第 0 步：先測手機，不猜規格

在 Pixel 上各跑一次 Edge Gallery benchmark／PocketPal benchmark，紀錄模型檔案大小、首次載入時間、tokens/s、10 分鐘持續推論後的溫度與耗電。手機端表現受 RAM、量化、runtime、NPU/GPU backend 和散熱殼影響很大；同一個「4B」模型的體感可差很多。

### 第 1 步：一個無副作用的本地 skill

做「語音／圖片 -> 結構化 note draft」：輸出固定 schema，僅寫入 app 私有資料庫，使用者可編輯、接受或丟棄。以 Gemini Nano API 作第一實作，再用 Edge Gallery／PocketPal 做不同模型與 tool loop 的對照。

驗收不看它講得多像人，而看：100 次輸入中 schema 是否有效、是否保留事實、錯誤時是否說不確定、離線能否完成、以及電池是否可接受。

### 第 2 步：加入一個 read-only 的真工具

例如本地搜尋已同意同步的 notes、列出今天行程快取、或讀取一份旅行清單。讓模型只負責選 tool 和口語化結果；資料庫查詢與權限仍由程式控制。這一步才是真正的 agent loop。

### 第 3 步：加入明確 approval 的寫入工具

例如建立 Calendar *草稿*、把購物清單加入待確認項目、或交給 OpenClaw 建立任務。送出訊息、付款、刪除、發文、帳號操作一律是不同權限級別，不與一般 notes 放在同一個 auto-execute bucket。

### 第 4 步：再碰 UI automation

只有在上面三步穩定後，才以 Mobilerun 或 Android accessibility 做一個單一 app、單一無害流程的受控實驗。每次動作保留 trajectory／截圖紀錄並可中止；把它當 QA／研究工具，不當無人監督的主手機操控器。

## 限制與安全底線

- **本地不等於免費或無風險。** 模型下載要吃儲存空間，持續推論會耗電與發熱；本地資料庫仍可能被惡意 app、遺失裝置或錯誤備份暴露。
- **背景存活是 Android 的現實限制。** 尤其 Termux／高 CPU runtime，不可把「螢幕關掉後會一直跑」當成預設。
- **Accessibility 是 root-like UX 權限。** 僅授予可信 app；不用時關閉；不要讓 UI agent 越過登入、2FA、金流或外部發送的確認。
- **模型輸出的 tool call 永遠是 untrusted input。** app 的 policy engine 要驗證 action、schema、scope、rate limit 與人類確認，並保留 audit log。
- **Gemini Nano 的可用 API、地區、語言與型號支援以裝置實測為準。** Android 官方文件描述的是 AICore／ML Kit 的一般能力；本報告不把「Gemini Nano 4」名稱當成可自行下載或可無限制控制系統的保證。

## 來源（2026-08-30 查閱）

- [Android Developers — Gemini Nano](https://developer.android.com/ai/gemini-nano)：AICore、ML Kit GenAI、on-device API 與使用情境。
- [Android Developers — AICore](https://developer.android.com/ai/aicore)：Android 系統端的 on-device GenAI execution service 與安全／硬體脈絡。
- [Google AI Edge Gallery 官方 repo](https://github.com/google-ai-edge/gallery)：Gemma 4、Agent Skills、Ask Image、Audio Scribe、Mobile Actions、支援版本與 Apache-2.0 授權。
- [PocketPal AI 官方 repo](https://github.com/a-ghorbani/pocketpal-ai)：GGUF／llama.cpp、on-device TTS、AgentRunner、Talents 與原生 hardware backend 說明。
- [OpenClaw 官方 repo](https://github.com/openclaw/openclaw)：Gateway、models、tools、companion apps／nodes 與安全文件入口。
- [Mobilerun 官方 repo](https://github.com/droidrun/mobilerun)：Android/iOS UI control、provider 支援（含 Ollama）、Portal／Accessibility／ADB 需求與 local-vs-cloud 說明。
- [Termux 官方 repo](https://github.com/termux/termux-app)：Android Linux environment、plugins 與 Android 12+ 背景 process 限制說明。

---

這份地圖的核心判斷是：**Pixel 不必取代 OpenClaw；它可以成為一個在離線、隱私與即時性上更強的 agent 前端與第一層大腦。** 真正有產品感的做法不是追求「手機全權自動化」，而是把本地能力、遠端深度、工具權限與人類確認設計成清楚的階梯。

# ChatGPT 單篇對話匯出與一鍵存取機制研究

日期：2026-07-06

## 摘要

截至本次調查，OpenAI 仍沒有提供給一般 ChatGPT 使用者的「用 conversation id 透過官方 API 抓取單篇 ChatGPT 對話完整 JSON」機制。官方穩定路徑主要是：

1. ChatGPT 設定中的資料匯出，或 Privacy Portal 資料請求。
2. ChatGPT shared link，用來分享某個時間點的對話快照。
3. Enterprise / Business 等組織管理功能與保留政策，但這不是一般個人帳號的單篇對話 API。

坊間確實有人做出「一鍵匯出目前對話」或「批次匯出所有對話」工具。它們大致分成三類：

1. 讀取官方資料匯出 ZIP 裡的 `conversations.json`，轉成 Markdown / Obsidian 格式。
2. 在 ChatGPT 網頁上用 DOM 或頁面狀態擷取目前對話。
3. 直接呼叫 ChatGPT 網頁內部的非公開 `/backend-api/...` endpoint，例如 `/backend-api/conversations` 與 `/backend-api/conversation/{id}`。

務實結論：如果目標是長期、低風險、可依賴的個人知識庫流程，應以「官方資料匯出 ZIP + 本地轉 Markdown」作為主線。如果目標是偶爾把眼前某篇對話快速存成 Markdown，可以使用本地可審查的 bookmarklet / userscript / extension，但要把它當成個人便利工具，不要把非公開 endpoint 當作產品級 API。

## 官方現況

### 1. ChatGPT 資料匯出

OpenAI Help Center 說明 ChatGPT 資料匯出有兩條路：Privacy Portal，以及 ChatGPT 設定中的 Data controls。匯出可用於 Free、Plus、Pro 與符合資格的 Edu workspace；不適用於登出狀態，也不適用於 ChatGPT Business / Enterprise workspace 的設定匯出流程。官方說匯出最多可能等 7 天，下載連結收到後 24 小時內有效，ZIP 內包含 chat history 與其他帳號資料。

這是目前最正式、最不容易壞的路徑，但缺點很明顯：不是單篇即時 API、等待時間不確定、也不適合「剛聊到一半立刻抓這篇」。

來源：
- https://help.openai.com/en/articles/7260999-how-do-i-export-my-chatgpt-history-and-data

### 2. Shared links

ChatGPT shared link 是官方支援的分享功能。它會產生 `https://chatgpt.com/share/<conversation-ID>` 形式的 URL，任何拿到連結的人都可查看該分享快照。官方 FAQ 說明：如果從側欄或右上分享按鈕建立連結，通常會包含建立分享當下可見進度之前的對話快照；如果是從單一 assistant response 分享，範圍可能只限該 response。Shared links 也會被包含在 ChatGPT data export 中。

這適合人工分享與引用，但不是穩定的原始 message JSON API。它也有隱私特性：任何持有連結者可看內容，沒有細緻權限或到期日設定。

來源：
- https://help.openai.com/en/articles/7925741-chatgpt-shared-links-faq

### 3. Retention / Enterprise 相關

OpenAI Help Center 的 retention 說明提到一般 chat 會留在帳號中直到使用者刪除；刪除後通常排入 30 天內永久刪除，法律或安全義務例外。該頁也提到 Enterprise 的 Compliance API considerations，特別是 Library files 以及 workspace retention policy 下的可用性，但這不是一般個人 ChatGPT 對話的一鍵讀取 API。

來源：
- https://help.openai.com/en/articles/8983778-chat-and-file-retention-policies-in-chatgpt

### 4. OpenAI API Reference 不等於 ChatGPT 歷史 API

OpenAI API Reference 面向 Responses、Realtime、Administration 等 API surface，使用 API key 或短期 access token 作為 bearer credentials。它沒有把個人 ChatGPT 網頁歷史對話列成可用資源。這點要和 ChatGPT 網頁內部的 `/backend-api/...` 分開看：後者是產品前端使用的非公開 endpoint，不是公開 API 合約。

來源：
- https://platform.openai.com/docs/api-reference

### 5. Terms 風險

OpenAI Terms of Use 在「What you cannot do」裡明確列出不得「Automatically or programmatically extract data or Output」。這不等於說個人永遠不能保存自己的對話，但它讓「自動化呼叫內部 endpoint 批量抓取 ChatGPT 歷史」至少處在不該產品化依賴的灰區。若只是手動用官方匯出，風險最低；若用 browser script 讀當前頁面，風險較低但仍要信任腳本；若用 bearer token 大量打 `/backend-api`，風險最高。

來源：
- https://openai.com/policies/terms-of-use/

## 社群工具地圖

### A. 官方 ZIP 轉 Markdown / 知識庫

這類工具不碰 ChatGPT session token，也不呼叫內部 endpoint。工作方式是：使用者先從 ChatGPT 官方資料匯出拿到 ZIP，再本地轉換。

代表工具：

- `daugaard47/ChatGPT_Conversations_To_Markdown`
  - GitHub: https://github.com/daugaard47/ChatGPT_Conversations_To_Markdown
  - 特色：處理完整 ChatGPT export ZIP；支援圖片、DALL-E、附件、音訊、tool calls、reasoning 等；可產生 Obsidian 友善 Markdown；有 browser-based converter，強調本地處理、對話不離開電腦。
  - 適合：定期整理、知識庫歸檔、Obsidian vault。

- `mohamed-chs/convoviz`
  - GitHub: https://github.com/mohamed-chs/convoviz
  - 特色：把 ChatGPT export ZIP 轉為 Markdown，並產生用量 / 字詞等視覺分析。
  - 適合：想分析整體 ChatGPT 使用史，而不是只保存單篇。

- 其他小型工具
  - `sanand0/chatgpt-to-markdown`
  - `sugurutakahashi-1234/ai-chat-md-export`
  - `beejaksharam/chatgpt-export-viewer`

判斷：這是最推薦的長期主線。缺點是無法即時抓單篇，但勝在合規感、穩定性和資料完整性。

### B. DOM / Bookmarklet / Userscript 擷取目前頁面

這類工具通常在已登入的 ChatGPT 頁面執行，從目前頁面 DOM 或前端狀態讀取訊息，轉成 Markdown / PDF / HTML。

代表工具：

- `yaph/chatgpt-export`
  - GitHub: https://github.com/yaph/chatgpt-export
  - 方式：bookmarklet clone `document.body`，清理不必要資訊，把 HTML 轉 Markdown 並下載。
  - 特色：簡單、可審查、不需要安裝大型 extension。
  - 限制：依賴當前頁面 DOM；ChatGPT 如果使用虛擬化列表，長對話可能不完整。

- `rashidazarang/chatgpt-chat-exporter`
  - GitHub: https://github.com/rashidazarang/chatgpt-chat-exporter
  - 方式：可用 DevTools console 貼 JS，或安裝 userscript；支援 ChatGPT / Gemini，輸出 Markdown / PDF。
  - 特色：主打 selector cascade 與 duplicate prevention。
  - 限制：仍依賴頁面結構，ChatGPT UI 變動會破。

- `revivalstack/ai-chat-exporter`
  - GitHub: https://github.com/revivalstack/ai-chat-exporter
  - 方式：Tampermonkey userscript，支援 ChatGPT、Claude、Copilot、Gemini、Grok，輸出 Markdown / JSON。
  - 特色：多平台、Table of Contents、YAML metadata、選擇性匯出。
  - 限制：跨平台 selector 維護成本高，遇到 UI 改版會需要修。

判斷：適合「我現在看到這篇，想立刻存一份可讀 Markdown」。若要保存長對話，必須確認工具是否能處理 ChatGPT 的虛擬化列表與未掛載訊息。安全上，優先選擇原始碼短、權限少、可本地審查的工具。

### C. 非公開 ChatGPT web backend API 匯出

這類工具技術上最接近 Jones 想問的「用 conversation id 抓完整 message JSON」。它們通常使用 ChatGPT 網頁本身的 session / bearer token，呼叫非公開 endpoint：

- `/api/auth/session`
- `/backend-api/conversations`
- `/backend-api/conversation/{id}`
- `/backend-api/gizmos/snorlax/sidebar`
- `/backend-api/gizmos/{id}/conversations`
- `/backend-api/files/download/{id}`

代表工具：

- `pionxzh/chatgpt-exporter`
  - GitHub: https://github.com/pionxzh/chatgpt-exporter
  - 方式：GreasyFork userscript。README 明確寫 JSON 是來自 `https://chat.openai.com/backend-api/conversation/[id]` 的 raw content，也支援從官方 `conversations.json` 匯入轉換。
  - 特色：文字、HTML、Markdown、PNG、JSON；可 export multiple conversations。
  - 風險：依賴非公開 endpoint。

- `bujue3709/GPT-Conversation-Toolkit`
  - GitHub: https://github.com/bujue3709/GPT-Conversation-Toolkit
  - 方式：README 說搜尋、時間線和導出優先使用 API conversation data，以避開 ChatGPT 虛擬化 DOM 只掛載部分訊息的問題；失敗時可 fallback 到 DOM。
  - 特色：長對話搜尋、timeline、Prompt 管理、資料夾、匯出 JSON/TXT/MD。
  - 風險：依賴 ChatGPT 網頁內部 API 與 UI 注入。

- `brianjlacy/export-chatgpt`
  - GitHub: https://github.com/brianjlacy/export-chatgpt
  - 方式：CLI，要求使用者從 DevTools Network 複製 `Authorization: Bearer ...` token；呼叫 `/backend-api/conversations` 與 `/backend-api/conversation/{id}`，也支援 Projects、attachments、DALL-E、Canvas、Deep Research 等。
  - 特色：批次、可續跑、支援 Teams / Projects、節流。
  - README 自帶 disclaimer：使用 unofficial backend API，可能隨時壞，使用者自負責任。
  - 風險：最高。需要手動處理 bearer token，不適合把 token 給不可信工具。

- `huhusmang/ChatGPT-Exporter`
  - GitHub: https://github.com/huhusmang/ChatGPT-Exporter
  - 方式：Chrome extension / Tampermonkey；README 與程式碼列出 `/api/auth/session`、`/backend-api/conversations`、`/backend-api/conversation/{id}`、Projects endpoint 與 file download endpoint。
  - 特色：個人、團隊、Projects、附件、定時提醒。
  - 風險：同樣是非公開 API + extension 權限。

判斷：這類工具證明「坊間確實有人做，而且技術上可以抓到完整 JSON」。但它們不是官方穩定 API。適合個人一次性備份、研究、可接受壞掉就修的流程；不適合拿來做需要穩定承諾的 app 或代理系統底層依賴。

### D. 功能型大型瀏覽器擴充

- `saeedezzati/superpower-chatgpt`
  - GitHub: https://github.com/saeedezzati/superpower-chatgpt
  - 特色：Folders、Search、Auto Sync、Export `.txt/.json/.md`、Prompt 管理、Pin、timestamps 等。
  - README 說 Auto Sync 會把所有 ChatGPT conversation 下載到本機，用於搜尋與整理，並聲稱對話歷史不會存到作者資料庫。
  - 風險：功能很完整，但權限與資料面較大；要使用前應閱讀 privacy statement、extension 權限、issue 狀態與維護活躍度。

判斷：這是「把 ChatGPT 變成可管理知識庫 UI」路線，不只是單篇匯出。若只為偶爾存幾篇對話，可能過重。

## 方法比較

| 路線 | 能否單篇即時 | 資料完整性 | 穩定性 | 隱私/ToS 風險 | 適合情境 |
|---|---:|---:|---:|---:|---|
| 官方資料匯出 ZIP | 否 | 高 | 高 | 低 | 定期備份、長期知識庫 |
| Shared link | 半手動 | 中 | 中高 | 中 | 分享 / 引用快照 |
| DOM / bookmarklet | 是 | 中到低 | 中低 | 中 | 眼前對話快速 Markdown |
| Userscript / extension 呼叫內部 API | 是 | 高 | 中低 | 中高 | 個人研究、可接受壞掉就修 |
| CLI + bearer token 批次抓 `/backend-api` | 是 | 高 | 中低 | 高 | 一次性完整備份、技術使用者 |

## 對 Jones / Jarvis 的建議

### 推薦主線：官方匯出 ZIP + 本地轉 Markdown

把它當作「月度或重要節點備份」流程：

1. Jones 從 ChatGPT 設定或 Privacy Portal 申請 data export。
2. 下載 ZIP 後放到本機安全位置，不 commit 原始 ZIP。
3. 用 `ChatGPT_Conversations_To_Markdown` 或類似本地工具轉成 Markdown。
4. 只把要公開或要沉澱的對話放進 `thinking_with_ai/conversations/` 或相關 project docs。

優點：最穩、最不容易踩帳號風險、也最符合長期 archive。

### 快速單篇：保留「手動複製 → Jarvis 轉逐字稿.md」作為低風險方案

這仍然是最乾淨的單篇路線。它不需要把 ChatGPT session token 交給外掛，也不會因為 ChatGPT UI 或內部 API 改版而失效。缺點只是人工。

可優化方向：做一個 repo 內部模板，例如：

```text
conversations/<slug>-YYYY-MM-DD/
├── transcript.md
├── notes.md
└── source.md
```

其中 `source.md` 保留 Jones 手動貼上的原始逐字稿，`transcript.md` 由 Jarvis 整理成穩定格式。

### 可試驗但不應產品化：本地審查過的 userscript / bookmarklet

如果真的想要「當前頁面一鍵下載 Markdown」，我會優先試短小、透明、只讀當前頁面的 bookmarklet / userscript，而不是大型 extension。候選：

1. `yaph/chatgpt-export`：簡單 bookmarklet，缺點是長對話完整性要測。
2. `rashidazarang/chatgpt-chat-exporter`：console/userscript 路線，操作簡單。
3. `pionxzh/chatgpt-exporter`：功能完整，但因為會碰內部 API，風險較高。

### 不建議常駐：把 bearer token 給 CLI 或第三方 extension 批次抓

`brianjlacy/export-chatgpt` 這類工具技術上很吸引人，尤其對 Projects、attachments、Deep Research 的支援很完整。但需要從 DevTools 複製 bearer token，並大量呼叫非公開 endpoint。我的建議是：

- 只在隔離瀏覽器 profile、一次性研究、完全讀過原始碼的前提下使用。
- 不要把 token 存進 repo、shell history、雲端同步資料夾或 log。
- 不要排 cron 長期自動跑。
- 不要把它視為穩定 API。

## 如果要做成 Jarvis workflow

我會分兩階段：

### Phase 1：官方匯出處理器

目標：把 ChatGPT 官方 export ZIP 轉成乾淨、可搜尋、可 cherry-pick 的本地資料庫。

做法：

1. 選一個本地 converter，例如 `ChatGPT_Conversations_To_Markdown` 或自己寫小工具讀 `conversations.json`。
2. 輸出到 `~/Downloads/exports/reports/chatgpt-export-YYYY-MM-DD/` 或私人 workspace。
3. 加一個篩選步驟：用標題、日期、關鍵字找出要公開到 `thinking_with_ai` 的對話。
4. 由 Jarvis 產生 `transcript.md`、摘要、tag、source note。

### Phase 2：可選的單篇 bookmarklet 試驗

目標：降低「複製整篇對話」的摩擦，但不導入高風險 token flow。

做法：

1. 在 OpenClaw `user` 或專門測試 profile 中人工試 bookmarklet。
2. 用幾篇長對話測完整性，特別是：
   - 很長的對話是否漏掉上方訊息。
   - code block、表格、引用、檔案、圖片是否保留。
   - ChatGPT canvas / projects / deep research 是否正常。
3. 若可靠，再保存成私人 playbook，而不是公開 repo 工具。

## 最後判斷

「一鍵存取 ChatGPT 對話」不是沒人做，而是官方沒有把它做成穩定公共 API。坊間工具的存在證明 ChatGPT web 內部資料結構可被抓取，尤其 `/backend-api/conversation/{id}` 幾乎就是大家想要的 JSON。但這條路的問題也正是它不是公開 API：可能隨時改、可能違反產品使用邊界、需要處理敏感 token，而且 extension / userscript 會接觸極高價值的私人對話資料。

所以我的建議是：長期歸檔走官方 export；單篇高價值對話仍可手動交給 Jarvis 轉 transcript；若要追求一鍵，先做本地、短小、可審查的 bookmarklet/userscript 試驗，不要一開始就引入常駐 extension 或 bearer-token CLI。


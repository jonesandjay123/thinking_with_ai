# NotebookLM x Jyn Pipeline Research

Date: 2026-05-23
Language: 繁體中文
Scope: NotebookLM 酷玩法、API 現況、社群工作流，以及它能否接進 OpenClaw Jarvis / Jyn Null production pipeline。

## 一句話結論

NotebookLM 最值得放進 Jyn 宇宙的位置，不是「替 Jarvis 寫摘要」，而是做成一個可被 Jarvis 餵資料、可被 Jones 詢問、可產出 briefing / audio / video / slides / quiz 的「來源消化層」。

目前最現實的架構是：

```text
Jarvis scout / web research / YouTube channel scan
  -> 整理成 source pack 或來源清單
  -> NotebookLM 消化與產生 Studio artifacts
  -> Jarvis 讀回摘要/結構/引用，再回源頭抽細節
  -> Jones 決策
  -> Jyn Null 前台內容或 jynnull.com 素材庫
```

關鍵限制是 API：一般消費版 NotebookLM 仍沒有成熟公開 API；Google Cloud 端已經有 NotebookLM Enterprise API 與 podcast API 文件，但 NotebookLM Enterprise API 屬於企業/教育付費與 Pre-GA 路線，不是個人帳號隨手可接。短期若要跟 OpenClaw Jarvis 對接，應先用「Jarvis 產 source pack + Jones/Jarvis 透過 UI 或半自動工具餵 NotebookLM」做低風險試驗；等 Enterprise API、官方 consumer API、或可信 MCP/CLI 工具成熟後，再做全自動。

## 目前 NotebookLM 能吃什麼

官方 Help Center 顯示 NotebookLM 可加入的來源類型已經很廣：

- Google Docs / Slides / Sheets
- PDF、Markdown、txt、docx、pptx、csv、ePub
- 網頁 URL
- YouTube 公開影片 URL
- 音訊檔，包含 MP3、WAV、M4A、MP4 等
- 圖片
- Gemini Chats / Gemini notebooks context

幾個對 Jyn/Jarvis 很重要的限制：

- 每個 source 可到 500,000 words 或 200MB；一般 notebook 可包含最多 50 sources。
- YouTube 只匯入 transcript，不匯入畫面本身；影片需要公開、要有人工或自動字幕；新上傳 72 小時內可能不能匯入。
- 網頁 URL 只抓 HTML 文字；不抓圖片、嵌入影片、巢狀頁面，paywall 不支援。
- Google Drive 匯入後是 NotebookLM 的靜態 copy；原始文件更新後要手動 sync，不是永遠 live。

這代表 NotebookLM 很適合吃「文字化後的世界」：YouTube transcript、podcast transcript、文章、研究包、文件、聊天記錄。但如果目標是分析影片畫面、餐廳環境、菜色截圖、店名招牌，它本身不是最好的第一層；Jarvis 應先用 YouTube/字幕/API/視覺工具把素材轉成文字與結構，再交給 NotebookLM。

Sources:

- Google Help: Add or discover new sources for your notebook  
  https://support.google.com/notebooklm/answer/16215270

## Studio artifacts：酷的不是摘要，而是「同一包來源的多種轉換」

NotebookLM 目前真正有趣的是 Studio artifacts：

- Audio Overview：兩位 AI hosts 對來源做深度對談，也可選 Brief、Critique、Debate 等格式。
- Video Overview：把來源轉成影片式講解，可選 Explainer、Brief、Cinematic、不同視覺風格，支援多語言。
- Mind Map：把來源結構化成主題圖，適合先看資料形狀。
- Slide Deck：把來源壓成簡報架構。
- Flashcards / Quizzes：把被動閱讀變成測驗與記憶。
- Infographic / Reports：把來源轉成展示或報告型輸出。

對 Jones 的用途不是「幫我摘要一下」，而是：

```text
同一包材料
  -> 給 Jones 聽的 Audio Brief
  -> 給 Jarvis 抓線索的 evidence map
  -> 給 Jyn Null 寫作前看的 tension map
  -> 給 jynnull.com 可展示的 public notebook / video overview / infographic
```

NotebookLM 的 Audio Overview 官方說明也提醒：它是根據上傳來源的客觀反映，但仍可能有錯誤或音訊 glitches。這一點很重要：它適合「幫 Jones 快速進入材料」，不適合直接當最終事實來源。

Sources:

- Google Help: Generate Audio Overview in NotebookLM  
  https://support.google.com/notebooklm/answer/16212820
- Google Help: Generate Video Overviews in NotebookLM  
  https://support.google.com/notebooklm/answer/16454555

## API 現況：有，但不是你想像的那種簡單個人 API

### 1. 消費版 NotebookLM

截至 2026-05-23，我沒有找到 Google 對一般消費版 NotebookLM 提供穩定公開 API 的官方文件。社群討論也反覆抱怨「沒有 real API」導致自動化卡在瀏覽器 UI。

這對 Jarvis 的含義：

- 不能假設我現在可以直接用官方 consumer API 建 notebook、塞 YouTube URL、問問題、下載 Audio Overview。
- 若用消費帳號自動化，要嘛走 browser automation，要嘛走非官方工具；兩者都要保守，尤其不要拿 Core Google identity 做高風險自動化。

### 2. NotebookLM Enterprise API

Google Cloud 文件顯示 NotebookLM Enterprise API 可以：

- create / get / list / delete notebooks
- batch add sources
- 加入 Google Drive content、raw text、web content、YouTube video content
- upload files as sources
- retrieve / delete sources
- create audio overview

文件標示為 Preview / Pre-GA，且需要 Google Cloud / Gemini Enterprise / 教育相關授權與設定。它適合企業或組織規模的知識庫與 compliance 場景，不是最輕的個人開發者 API。

值得注意的是，NotebookLM Enterprise API 的 `sources:batchCreate` 已明確支援 `videoContent.youtubeUrl`。這表示 Jones 的例子「Jarvis 掃 YouTube 影片清單 -> 丟 NotebookLM -> 抽餐廳名單」在官方 Enterprise API 路線上是可想像的。

### 3. Podcast API

Google Cloud 也有 standalone Podcast API 文件，可從 text/images/audio/video contexts 生成 MP3 podcast。但文件明確標示：Podcast API deprecated，Google 不再 allowlist 新客戶。它可作為「Google 曾經/企業內部如何把 source context 轉 podcast」的參考，不應作為我們今天要押注的新入口。

Sources:

- Google Cloud: Create and manage notebooks API  
  https://docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/api-notebooks
- Google Cloud: Add and manage data sources API  
  https://docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/api-notebooks-sources
- Google Cloud: Manage audio overview API  
  https://docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/api-audio-overview
- Google Cloud: Generate podcasts API method  
  https://docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/podcast-api

## 社群看到的酷玩法

### 玩法 1：不要只問 summarize，先做 source index

Reddit 上一個常見進階用法是：先叫 NotebookLM 對 messy sources 建立 topic index，再逐題深挖，而不是直接「summarize」。原因是直接摘要常會犧牲細節；topic index 可以讓使用者挑分支，讓 NotebookLM 從所有來源拉相關證據。

套進 Jyn pipeline：

```text
Jarvis 產 source pack
  -> NotebookLM 先做 topic index / mind map
  -> Jones 選 2-3 個 tension
  -> Jarvis 回 source pack 抽 quote/evidence
  -> Jyn Null writer 產前台版本
```

這比「幫我摘要 Google I/O」更像研究流程。

### 玩法 2：Podcast/YouTube 不是拿來聽完就算，而是轉成可複習素材

另一個社群工作流是把 podcast episode 或 YouTube 影片批次匯入 notebook，然後對每集產：

- slide deck
- flashcards
- quiz
- follow-up Q&A

核心觀念是「retention > summary」。這對 Jones 很有用，因為你常常不是缺一段摘要，而是需要把一批材料變成可回頭問、可重新組合、可觸發創作的知識包。

### 玩法 3：YouTube channel / playlist scan

有不少人卡在「單支影片很容易，整個 YouTube channel / playlist 很亂」。社群工具與 Chrome extension 正在補這個缺口：在 YouTube 側邊欄掃頻道/playlist，挑選影片後送到特定 notebook。

這剛好對應你舉的餐廳例子：

```text
Jarvis:
  1. 抓吃貨豪豪、蝦土豆近期影片清單
  2. 篩選台北 / 吃到飽 / buffet / 燒肉 / 火鍋
  3. 建立 NotebookLM-ready source list

NotebookLM:
  4. 匯入 YouTube URLs 或 transcript pack
  5. 產生餐廳名、地址、價格、菜系、亮點、影片引用

Jarvis:
  6. 回頭查店家 Google Maps / 官網 / 訂位資訊
  7. 輸出 Jones 可選名單
```

這個例子雖然你說有點殺雞用牛刀，但它揭示一個很好的模式：NotebookLM 適合處理「多來源 transcript 的交叉整理」，Jarvis 適合處理「抓來源、補驗證、做最後表格」。

### 玩法 4：Daily briefing / overnight digestion

社群最想要 API 的原因，是想做類似：

```text
晚上丟 research papers / YouTube saved videos / RSS feeds
  -> 半夜自動生成 audio overview / slide deck / brief
  -> 早上通勤聽
  -> 有興趣再回源頭
```

這非常貼近 Jones 的使用方式：不用每天坐著讀一堆長文，而是讓資料先變成可聽、可掃描、可追問的 briefing。

Community references:

- Reddit: Stop asking NotebookLM to summarize your sources  
  https://www.reddit.com/r/notebooklm/comments/1rse4wp/title_stop_asking_notebooklm_to_summarize_your/
- Reddit: Retain podcasts with RSS, slide decks, flashcards  
  https://www.reddit.com/r/notebooklm/comments/1riulqb/how_i_use_notebooklm_to_actually_retain_podcasts/
- Reddit: Scan YouTube channels into NotebookLM  
  https://www.reddit.com/r/notebooklm/comments/1rw3r3e/a_better_way_to_scan_youtube_channels_into/
- Reddit: Why no real API in 2026  
  https://www.reddit.com/r/notebooklm/comments/1scada8/notebooklm_is_amazing_but_why_the_hell_is_there/
- Reddit: NotebookLM workflow gets messy fast  
  https://www.reddit.com/r/notebooklm/comments/1tfo8z3/notebooklm_is_amazing_but_the_workflow_gets_messy/

## 非官方自動化工具：可試，但不要當核心依賴

目前社群已有幾種非官方路線：

- `notebooklm-py`：Python/CLI/agent skill，宣稱可管理 notebooks、加入 sources、產生 artifacts。
- `notebooklm-mcp-cli`：CLI + MCP server，宣稱可讓 Claude/Cursor/Codex/Gemini 等 agent 建 notebook、加 sources、query、產 Studio artifacts、download artifacts。
- Web Clipper / NoteKitLM / ExtendLM 類 Chrome extension：解決瀏覽器裡收集來源、YouTube 批次匯入、重複 prompt、匯出等問題。

我對這些工具的判斷：

- 可以當 sandbox proof-of-concept。
- 不要拿它們處理敏感帳號或高價值資料。
- 不要把它們直接接成 24/7 自動化，因為多數依賴 cookie / undocumented internal API，Google 可能改端點、限流或觸發帳號風控。
- 若要在 OpenClaw 上測，應使用 Bot/低風險 Google 帳號，不要用 Core identity。

References:

- GitHub: notebooklm-py  
  https://github.com/teng-lin/notebooklm-py
- GitHub: notebooklm-mcp-cli  
  https://github.com/jacob-bd/notebooklm-mcp-cli
- Web Clipper article on NotebookLM API options  
  https://web-clipper-for-notebooklm.com/blog/notebooklm-api

## 跟 Gemini notebooks 的新關係

Google 2026-04 宣布 Gemini app 也有 notebooks，且可與 NotebookLM sync。官方描述是：Gemini notebooks 讓使用者整理 chats/files、給 custom instructions；來源可同步到 NotebookLM，再用 NotebookLM 的 Video Overview / Infographics 等功能。

這對 Jones/Jarvis 有一個新想像：

```text
Gemini notebook = Google 生態內的 project workspace
NotebookLM = 同一批 source 的 studio/artifact layer
Jarvis = 本機/Slack/repo-first operator
```

如果未來 Gemini notebooks API 或 Workspace 整合變強，Google 這條線可能變成「雲端 knowledge workspace」。但目前我們仍應把 durable source archive 放在 Git repo / jarvis-scout / thinking_with_ai，而不是只放在 Google notebook 裡。

Source:

- Google Blog: Try notebooks in Gemini to easily keep track of projects  
  https://blog.google/innovation-and-ai/products/gemini-app/notebooks-gemini-notebooklm/

## 我建議的 Jyn Null Production Stack 位置

NotebookLM 應該插在 `jarvis-scout` 後面、Jyn Null writer 前面：

```text
Source discovery
  - Jarvis scout
  - YouTube channel scan
  - Reddit / Threads / X / articles

Source archive
  - Git repo source pack
  - transcript files
  - evidence table
  - quote bank

NotebookLM digestion
  - topic index
  - mind map
  - audio brief
  - video overview
  - slide deck
  - quiz / flashcards

Jarvis synthesis
  - validate against sources
  - extract claims / citations / URLs
  - produce Jones-facing recommendation

Jyn Null output
  - cold observation
  - short post
  - essay seed
  - visual brief for Flow / Omni
  - public archive note on jynnull.com
```

這樣 NotebookLM 不會取代 Jarvis。它變成一個「把 source pack 轉成可互動知識物件」的外部器官。

## OpenClaw 對接方案

### Phase 0：手動 UI + Jarvis source pack

今天就能做。

```text
Jarvis 產生:
  notebooklm-source-pack.md
  source-list.csv
  questions.md

Jones / Jarvis browser:
  建 notebook
  加 source pack 或 URLs
  產生 mind map / audio overview / slide deck

Jarvis:
  根據 NotebookLM 輸出做二次整理
```

優點：安全、低成本、可學產品能力。  
缺點：不能完全自動。

### Phase 1：半自動 browser workflow

Jarvis 可用 OpenClaw browser 在 `openclaw` profile 中做低速、可觀察的操作，例如建立 notebook、貼 source、下載 artifact。但這必須很保守，因為涉及 Google 帳號風控與 UI 改版。

適合小量試驗，不適合大量排程。

### Phase 2：非官方 MCP/CLI sandbox

在 Bot Google account 上測 `notebooklm-mcp-cli` 或 `notebooklm-py`：

```text
Jarvis CLI:
  create notebook
  add YouTube URLs
  query notebook
  create audio/video artifact
  download artifact
```

驗收重點：

- 登入是否穩
- cookie/session 是否容易過期
- YouTube 中文/繁中 transcript 匯入是否穩
- artifact 能否下載
- query 是否有 citation/source trace
- 是否會觸發 Google 安全警告

### Phase 3：官方 Enterprise API

若 Jones 後續真的要把 NotebookLM 變成長期可程式化 backend，才評估 Gemini Enterprise / NotebookLM Enterprise API。這條路比較像正式系統，不像個人玩具。

## 今日最小實驗建議

我建議今天不要先裝非官方工具，先做一個可驗證的 NotebookLM trial：

```text
trial-001-youtube-food-scan/
  source-list.md
  notebooklm-questions.md
  jarvis-output.md
  verdict.md
```

題目可以用你剛才的餐廳例子，但更精準一點：

目標：測 NotebookLM 是否能從一批 YouTube transcript 中抽出「店名、地區、價格、推薦點、適合誰、不確定處」。

流程：

1. Jarvis 抓 10-15 支近期台北吃到飽/餐廳 YouTube 影片 URL。
2. Jarvis 建立 source list，註明頻道、日期、標題、URL。
3. 丟 NotebookLM。
4. 問 NotebookLM：

```text
請從所有 YouTube transcript 中整理影片提到的餐廳。
欄位：餐廳名、地區、影片來源、影片發布日期、主題/菜系、價格資訊、推薦理由、負面或保留意見、是否需要我回影片確認。
只根據來源內容回答；不確定就標記「來源不足」。
```

5. Jarvis 拿 NotebookLM 輸出回頭查 Google Maps/官網/訂位資訊，補成 Jones 可用名單。
6. Verdict：

- transcript 匯入成功率
- 店名抽取準確率
- 是否保留不確定性
- 是否比 Jarvis 直接讀 transcript 更省時間
- 是否值得接進 scout pipeline

這個實驗不是為了找餐廳，而是測 NotebookLM 作為「多 YouTube source 消化器」的品質。

## 對 Jyn Null 的延伸實驗

如果 food scan 成功，下一個更貼 Jyn 的 trial 應該是：

```text
trial-002-attention-economy-scout/
  source pack: Reddit / Threads / X / YouTube / articles
  NotebookLM artifact: mind map + audio brief
  Jarvis artifact: tension map
  Jyn artifact: 5 cold observations + 1 essay seed
```

NotebookLM 問題：

```text
這批來源中，人們對 AI、注意力經濟、孤獨、社群平台的主要焦慮是什麼？
請整理成 tension map：
- 表層事件
- 底層情緒
- 支持證據
- 反方或不確定處
- 最適合 Jyn Null 觀察的 angle
```

這才是 NotebookLM 對 Jyn 宇宙的真正價值：把雜訊世界壓縮成可被 Jyn 觀察的張力圖。

## 判斷

我會把 NotebookLM 評為「非常值得導入，但先不要全自動化」。

原因：

- 它已經能吃 YouTube / audio / docs / web / Gemini notebooks，對 source digestion 很強。
- Studio artifacts 讓同一批材料可以轉成 audio、video、slide、mind map、quiz，這比一般摘要工具更像 production layer。
- 官方 Enterprise API 證明它不是完全封閉玩具，但個人 API 還不成熟。
- 社群最大的痛點正好是我們在意的：bulk import、source organization、export、API、自動化。
- Jarvis 可以補 NotebookLM 的弱點：抓來源、驗證細節、回源頭、保存 durable archive、產出 Git-first 報告。

建議定位：

```text
NotebookLM = source digestion studio
Jarvis = operator + verifier + archive manager
Jyn Null = frontstage persona output
```

短期先做 source-pack-to-NotebookLM trial；中期測非官方 MCP/CLI；長期才考慮 Enterprise API 或官方 consumer API。


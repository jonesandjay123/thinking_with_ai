# Codex Record & Replay 與 Jyn Null 發文流水線

日期：2026-06-18  
範圍：調查 OpenAI / Codex 官方新功能 Record & Replay，並初步評估是否適合用在 Jyn Null 的 IG / X / Threads 發文流程。

## 給 Jones 的短結論

你看到的方向是對的：**Record & Replay 很像官方支持的「把人類示範流程變成可重用 agent skill」能力，所以它非常適合拿來研究 Jyn Null 的發文流水線。**

但它不是傳統 RPA 那種「錄一次滑鼠座標，之後逐格重播」。官方描述比較像：

```text
你示範一次 workflow
→ Codex 觀察操作和畫面內容
→ Codex 產生一個 reusable skill
→ 之後你在新 thread 給不同素材/文字
→ Codex 用 skill + Computer Use / browser actions / plugins 重做流程
```

所以我會把它定位成：

> **官方版 workflow-to-skill 生成器，不是單純 macro recorder。**

對 Jyn Null 的意義是：它可能可以把「每次發 IG / X / Threads 的 UI 操作習慣」變成官方 Codex skill，但仍然需要我們保留發文前人工確認，尤其 public posting 屬於外部動作，不應讓它靜默自動發出去。

## 官方功能事實

### 1. 目前平台與可用條件

官方文件寫明：

- Record & Replay available on macOS。
- 初期可用地區不包含 EEA、UK、Switzerland。
- 必須啟用 Computer Use。
- 如果組織用 `requirements.toml` 管理 Codex，`[features].computer_use` 也會控制 Record & Replay；設成 `false` 就看不到這功能。

Sources:

- https://developers.openai.com/codex/record-and-replay
- https://developers.openai.com/codex/app/computer-use

### 2. 它產出的不是影片，而是 skill

官方說 Record & Replay 讓你在 Mac 上示範 workflow，並把它轉成 reusable skill。適合：

- 重複性流程
- 依賴個人偏好的流程
- 比起文字描述，更容易用示範說清楚的流程

官方例子包含：

- file an expense
- book a parking space
- create a configured issue
- publish a video
- download recurring report

這些例子很重要，因為「publish a video」和我們的「發 Jyn Null 貼文」屬於同一類：不是純 code，也不是純資料處理，而是跨 UI、登入狀態、素材、欄位、確認按鈕的工作流。

Source: https://developers.openai.com/codex/record-and-replay

### 3. 錄製流程

官方流程：

1. 開 Codex app 的 Plugins。
2. 開 `+` menu。
3. 選 `Record a skill`。
4. 檢查 suggested prompt，補充 context。
5. Codex 要求錄製權限時，同意。
6. 你在 Mac 上完成 workflow。
7. 完成後從 menu bar / overlay 停止錄製，或告訴 Codex 已完成。

錄製期間 Codex 會觀察它需要學習 workflow 的 actions 和 window content。錄完後，它會分析 captured workflow 並草擬 skill。這個 skill 會描述：

- 何時使用
- 需要哪些 inputs
- 該怎麼做
- 如何驗證結果

Source: https://developers.openai.com/codex/record-and-replay

### 4. Replay 的真正意思

Replay 時不是直接「播放錄影」。官方說法是：

1. 開一個新 thread。
2. 請 Codex 使用 generated skill。
3. 給它這次不同的 values，例如要上傳的檔案、issue、日期範圍。
4. Codex 會把 skill 當 reusable context，並用目前環境可用工具完成工作。

可用工具包括：

- Computer Use
- browser actions
- connected plugins
- 以上的組合

Source: https://developers.openai.com/codex/record-and-replay

## 跟 Jyn Null 發文流程的關係

### 它最適合學的部分

Jyn Null 發文流程裡，有幾個東西非常適合 Record & Replay：

```text
打開平台
確認登入 identity
建立新 post
上傳圖片
填 caption
檢查 preview
處理平台特殊欄位
停在 publish 前等待確認
發佈後打開 public URL 驗證
```

這些東西原本用純腳本很麻煩，因為 IG / Threads / X 的 DOM、登入狀態、彈窗、UI 變化很多。Record & Replay 的價值是讓 Jones 或 Jarvis 先示範「正確的人類流程」，再由 Codex 轉成技能文件和操作策略。

### 它不會自動解決的部分

Record & Replay 不等於平台 API，也不等於保證穩定的 browser automation。

它仍然會碰到：

- IG / Threads / X UI 改版。
- caption field 沒有保存。
- image upload 卡住。
- login / 2FA / risk challenge。
- hashtag / topic 是不是平台真正 topic。
- public posting 需要人工確認。
- platform rate limit / anti-automation 風險。

也就是說，它可以把流程「官方化、技能化、可重跑」，但不能把社群平台的不穩定性變不見。

## 需要搭配哪種 Codex 能力？

### Chrome extension

如果要操作 IG / X / Threads 這類已登入網站，官方建議用 Codex Chrome extension，因為 in-app browser 不支援登入狀態、cookies、既有 browser profile。

官方 Chrome extension 用途：

- 讓 Codex 使用你的 signed-in Chrome state。
- 適合需要讀取或操作登入網站的任務。
- 會依 website host 要求 permission。
- 可以用 allowlist / blocklist 管理網站。

Source: https://developers.openai.com/codex/app/chrome-extension

對 Jyn Null 來說，這代表：

```text
X web posting: 很可能適合 Record & Replay + Chrome extension
Threads web posting: 可能適合，但要測 caption/topic/link 行為
Instagram web posting: 要小心，因為我們近期遇過 caption 不保存
```

### Computer Use

Computer Use 是 Codex 操作桌面 app / GUI 的能力。官方說它可以看和操作 macOS / Windows 圖形介面，適合 command-line 或 structured integration 不夠時使用，例如 browser、desktop app、多 app workflow。

在 macOS 上需要：

- Screen Recording permission
- Accessibility permission

Source: https://developers.openai.com/codex/app/computer-use

對 Jyn Null 來說，它比較像底層執行能力：Record & Replay 產生 skill，Replay 時可用 Computer Use 去照著做。

### In-app browser

官方明確說 in-app browser 不支援登入流程、signed-in pages、regular browser profile、cookies、extensions、existing tabs。它比較適合 localhost / public pages / 不需要登入的頁面。

Source: https://developers.openai.com/codex/app/browser

所以 Jyn Null 發文不要把 in-app browser 當主路線；要用 signed-in Chrome / Chrome extension，或是我們現有的 Android native / ADB tooling。

## 對三個平台的初步判斷

### X

可行性最高。

理由：

- X web posting 比 IG/Threads 穩定。
- 我們已經有 Jyn Null Chrome identity。
- 發文欄位和 publish flow 相對簡單。
- Replay skill 可以把「附圖、貼 caption、publish 前確認、發後取 URL」標準化。

建議第一個 pilot 就用 X。

### Threads

中等可行。

理由：

- Threads web / Android 都可發，但平台 topic / tag 行為需要驗證。
- 我們之前 Threads Android native 發文成功，`Lego` 變成平台 topic link，raw hashtag 沒殘留。
- 如果 web flow 的 topic 行為穩定，Record & Replay 很適合。

建議第二個 pilot。

### Instagram

短期不要太樂觀。

理由：

- 2026-06-18 我們剛遇過 IG web caption 不保存：web post 出現空 caption，web edit 也沒有持久保存，最後改用 Android native 發才成功。
- Record & Replay 可以讓流程更清楚，但如果平台 web 本身有 bug 或限制，Replay 也會踩到同一個坑。

建議：IG 可以錄，但要把 skill 寫成「web 發佈後必須 public 驗證 caption；若 caption 不保存，切 Android native fallback」。

## 這是否是「官方支持的最合適發文流水線」？

我的判斷：**是很接近，但還不能直接說它完全取代我們現有發文系統。**

更準確地說：

> Record & Replay 是目前最適合把「人類示範過的圖形介面發文流程」轉成 Codex 可重用 skill 的官方機制。

但完整 Jyn Null 發文流水線應該是 hybrid：

```text
內容與素材準備：jynnull-lab scripts / repo
平台發文 UI 操作：Codex Record & Replay skill
登入網站操作：Chrome extension
桌面 GUI 操作：Computer Use
Android fallback：現有 ADB / native app tooling
發布前安全閘：人工確認
發布後驗證：public URL / caption / image / topic check
```

也就是：

```text
Record & Replay = 發文操作層的官方 skill builder
不是整個 publishing pipeline 的唯一答案
```

## 我建議的 Jyn Null 試點設計

### Pilot 1：X image post skill

目的：用最穩的平台測 Record & Replay 的實際價值。

錄製時示範：

1. 打開 Jyn Null Chrome profile。
2. 進入 X compose。
3. 上傳指定圖片。
4. 貼入 caption。
5. 檢查 preview。
6. 停在 publish 前，明確要求人工確認。
7. 發佈後打開 public post URL。
8. 驗證圖片和 caption。

Skill inputs 應該定義成：

```text
image_path
caption
expected_account
publish_mode: dry-run | publish-after-confirm
```

Success criteria：

```text
登入帳號是 Jyn Null
圖片顯示正確
caption 完整
publish 前有人工確認
publish 後取得 public URL
public URL 內容正確
```

### Pilot 2：Threads post skill

目的：測 tag / topic / caption 是否比我們現在 web/Android 混合更穩。

額外驗證：

```text
tag 是否變成平台 topic/link
raw hashtag 是否殘留
public page 是否可見
```

### Pilot 3：Instagram guarded skill

目的：不是馬上全自動，而是測 Record & Replay 能否穩定避開 caption 不保存問題。

必要設計：

```text
先發 dry-run 或停在 share 前
publish 後一定要開 public URL 驗證 caption
若 caption 空白，停止，不要繼續
必要時走 Android native fallback
```

## 應該怎麼錄，成功率會比較高？

官方 tips 對我們很重要：

- demonstration 要短而完整。
- 錄之前先告訴 Codex 目標，以及哪些 input 之後會變。
- 用 realistic inputs，但避免 secrets。
- 錄完後 refine skill，把 hidden preferences 補上。
- workflow 完成就停錄，不要錄進 unrelated cleanup。

套到 Jyn Null：

```text
不要一次錄 IG + X + Threads + 小紅書
一次只錄一個平台
一次只錄一種 post type：single image post
不要錄登入密碼 / 2FA
不要錄私密帳號切換
publish 前停下，讓 skill 形成安全閘
```

## 風險與安全邊界

### 1. Public posting 必須保留人工確認

Jyn Null 對外發文是 public action。即使 Codex skill 很穩，也不應該讓它在沒有人工確認時自己按 publish。

建議 skill 永遠有兩個模式：

```text
dry-run: 填好但不發
publish-after-confirm: 填好、檢查、等待明確確認後發
```

### 2. 錄製時不要展示 secrets

官方也提醒用 realistic inputs，但避免 secrets / sensitive data。對我們來說：

- 不錄登入密碼。
- 不錄 recovery codes。
- 不錄其他私人 Chrome tabs。
- 不錄非 Jyn Null identity。

### 3. 平台內容本身是 untrusted context

Chrome extension 和 Computer Use 都會看到頁面內容。社群平台 feed、DM、notification、彈窗可能包含不可信內容。錄製和 replay 時最好直接進 compose URL / profile，不要讓它亂逛 feed。

### 4. IG 仍需 fallback

Record & Replay 不應讓我們忘記 2026-06-18 的 IG caption 不保存問題。這類平台 bug 必須靠 post-publish verification 和 fallback，不是靠錄製流程本身解決。

## 下一步建議

我建議我們下一步不是立刻錄全流程，而是做一個很小的驗證：

```text
只錄 X single-image post
只用一張測試圖
caption 用測試文字
停在 publish 前
讓 Codex 生成 skill
再用另一張圖/另一段 caption dry-run 一次
```

如果這個成功，才往 Threads / IG 擴展。

我會把「Jyn Null official publishing skill」拆成三個技能，而不是一個大技能：

```text
jyn-null-post-x
jyn-null-post-threads
jyn-null-post-instagram-guarded
```

最後再由一個上層 checklist / script 決定今天要跑哪些平台。

## Sources

- OpenAI Developers, Record & Replay: https://developers.openai.com/codex/record-and-replay
- OpenAI Developers, Computer Use: https://developers.openai.com/codex/app/computer-use
- OpenAI Developers, Codex Chrome extension: https://developers.openai.com/codex/app/chrome-extension
- OpenAI Developers, In-app browser: https://developers.openai.com/codex/app/browser


# Pokémon GO 大型儲藏庫整理與 Collection Curator 研究

> **研究日期：** 2026-09-01  
> **問題：** 一個累積多年、2,000+ 隻 Pokémon 的帳號，如何有效盤點、比較同物種、留下可解釋的保留決策，並安全地清理冗餘？  
> **範圍：** 這是儲藏庫與資料整理研究，不是 PvP 最佳化或自動遊戲研究。所有「風險」僅表示官方條款文字與技術邊界，並非 Niantic 的認證或保證。

## Executive Summary

1. **重度玩家不是逐隻「判死刑」，而是把倉庫拆成多個決策隊列。** 最常見的可持續模式是：先保護明確有價值者，再用搜尋把顯然可轉移者拉出來，最後才以「同物種家族」處理模糊重複。把三種決策混在一次清理，最容易誤刪。
2. **遊戲內搜尋是最高槓桿、最低風險的第一層。** `0*`～`4*`、`age`、`year`、`!`、`&`、`,`、特殊狀態與標籤的組合，足以把「候選池」縮小；它不是保留價值引擎，不能直接決定傳送。
3. **標籤要少而可行動。** 對數千隻帳號，永久標籤宜描述保留理由（如 `RAID`、`TRADE`、`COLLECT`），暫存標籤則描述下一步（如 `REVIEW`、`XFER-CHECK`）。大量細粒度、互相重疊的分類會失控；`Favorite` 應被視為刪除前的硬保護，而非完整資料庫。
4. **Poke Genie 仍是目前最成熟的螢幕讀取式個人收藏記錄選項之一。** 2026-08 的 iOS 8.17.0 更新仍在修正 Pokémon GO UI 變動後的 screen scan，並保留 scan history；它用截圖／錄影讀畫面而非帳密登入。但「可讀螢幕」不等於它能一鍵完成全倉庫清理。
5. **大型、完整盤點的真正瓶頸是資料品質，而不是 OCR。** 同一隻可能有暱稱、異色、暗影、幸運、服裝、不同畫面比例；只讀格狀清單通常只能拿到名稱／CP，讀 IV、招式與保留脈絡則要開詳情或鑑定。開源 POC 的完整讀取實測甚至是每隻 5–10 分鐘。
6. **「純截圖／螢幕錄影 → 本機資料庫 → 人工核准」是最合理的未來工具切入點。** 它不需 Pokémon GO 帳密，也不接觸遊戲伺服器；可先做到清單級盤點與同物種分群，再將少數有爭議的個體送入鑑定畫面複核。
7. **ADB／Accessibility 的讀屏本身並未在官方政策逐字點名，但一旦自動點擊、滑動、改名或傳送，即存在未授權第三方 add-on / 自動化的帳號風險。** 不應稱為「安全」。改名器等開源專案也在 README 自行警告此事。
8. **沒有找到官方提供的完整 Pokémon 清單 CSV 匯出，或可供第三方用帳密存取的官方 inventory API。** Pokémon HOME 轉移是不可逆的遊戲內轉出機制，不是盤點備份方案；帳號資料下載也不應假設含有可用的逐隻儲藏庫欄位。
9. **最有價值的自訂功能不是再做一個 IV calculator，而是「可回溯的保留理由」。** 現有工具較擅長單隻 IV／對戰；真正缺口是跨物種去重、一次看齊所有副本、保留理由／歷史與人類核准的刪除工作流。

---

## 1. 大型帳號實際採用的整理模型

公開社群討論的細節各異，但反覆出現的做法可歸納為以下流程。這是「模式歸納」，不是單一玩家的規則，也不應拿來盲量刪除。

### 1.1 三段式：先保護、後縮小、最後比較

| 階段 | 工作 | 為什麼可擴到 2,000+ | 典型保護措施 |
| --- | --- | --- | --- |
| 0. 保護 | 對明確不該誤刪的個體加 `Favorite`，並加上少量永久標籤 | 先把高價值集合從後續搜尋中排除 | 神獸、異色、服裝、暗影、幸運、舊年分、交易候選、已投資者 |
| 1. 初篩 | 用 IV、最近捕捉、特殊狀態與標籤搜尋出「可考慮傳送」候選 | 不必一開始看所有 2,000 隻 | 絕不把結果直接 mass transfer；先標 `XFER-CHECK` |
| 2. 同家族比較 | 以一個進化家族為單位，同時看同種／進化線全部副本 | 回答「為何留這隻、為何留三隻」而非只看單隻星數 | 先挑預定用途，再把其他副本標為待確認 |
| 3. 二次審核／執行 | 隔一段時間再檢視待傳送集合，才手動確認傳送 | 避免當下疲勞、錯選與資料不完整 | 先截圖／匯出清單；一次只清小批 |

### 1.2 重度玩家常見的「保留理由」

不同玩家數量門檻不同，但理由大致穩定：

- 已投入資源或明確用途：raid attacker、Mega、gym defender、特定聯盟 PvP；
- 稀有／不可重得脈絡：異色、服裝、幸運、暗影／淨化、地區／活動、舊年份、紀念地點；
- 社交用途：可交易、朋友保留、尚未交換；
- 收藏用途：Living Dex、不同型態、性別、背景、大小、特殊招式；
- 未決用途：`REVIEW`，而不是假裝知道答案。

真正的共識不是「每種只留一隻」，而是**每一類保留理由先有上限或明確用途**。沒有用途的「可能有用」會隨年數膨脹成整個倉庫。

### 1.3 固定節奏比災難式大掃除有效

- **捕捉後 30 秒微整理：** 社群日、活動、交換後先對明顯低 IV／多餘常見種做初篩。
- **每週／每月短 session：** 只處理一個隊列，例如最近七天的 `0*`／`1*`。
- **活動後 family review：** 對本次大量捕到的物種，做同家族比較並留下用途標記。
- **年度盤點：** 專門處理舊 Pokémon、特殊交換、紀念品；不混入快速清倉。

這種節奏避免「儲滿才一次處理 2,000 隻」的認知負荷。

### 1.4 社群資料的限制

本次以 Reddit/TheSilphRoad、r/pokemongo 的 storage-management／cleanup 討論作為線索，但其頁面在研究環境無法擷取留言正文，因此本報告不把任何匿名貼文當成統計事實或封號／成功率證據。下列是可供 Jones 自行查看的討論入口：

- [TheSilphRoad：Pokémon storage management](https://www.reddit.com/r/TheSilphRoad/comments/10g5a9p/pokemon_storage_management/)
- [r/pokemongo：How do you manage your Pokémon storage?](https://www.reddit.com/r/pokemongo/comments/15e7e6e/how_do_you_manage_your_pokemon_storage/)

本報告的工作流結論也以工具產品文件與開源實作可驗證的限制交叉檢查，而非只依兩篇討論。

---

## 2. 最值得先掌握的內建搜尋技巧

### 2.1 語法速查與使用邏輯

遊戲內搜尋支援多種關鍵字。UI 與關鍵字偶有改版，執行破壞性動作前務必在**目前 app 的搜尋建議／結果數**逐條驗證。以下採用長期穩定、廣泛使用的語法族；不能把它當官方永久 API。

| 語法／類別 | 回傳內容 | 可用於 | 不會告訴你的事 |
| --- | --- | --- | --- |
| `0*`、`1*`、`2*`、`3*`、`4*` | 對應鑑定星等 | 初篩與保留集合 | 3* 不等於唯一值得留；低星也可能是 PvP、收藏或交易價值 |
| `age0`、`age0-7`、`age30-90` | 捕捉年齡（日）或範圍 | 近期捕捉清理、建立 inbox | 「剛抓到」不是「可刪」；活動、交換、色違仍須排除 |
| `year2016`、`year2016-2018` | 捕捉年份／區間 | 舊 Pokémon 人工審查 | 年份本身不是價值判定，但舊 Pokémon 可能有交易、紀念或舊招式脈絡 |
| `distance100-` | 捕捉地距離達某範圍 | 檢查旅行／遠距紀念品 | 不是稀有度或戰力指標 |
| `shiny`、`shadow`、`purified`、`lucky`、`costume` | 特殊狀態 | 建立保護集合 | 仍可能有同狀態大量重複；不要一概無限保留 |
| `legendary`、`mythical`、`ultrabeasts` | 稀有類別 | 保護與分批檢視 | 不區分招式、投資或已經多餘的副本 |
| `favorite`、`traded` | 已收藏／已交換 | 安全排除、審計 | Favorite 是帳號內保護，不等於「理由已記錄」 |
| `evolve`、`evolvenew`、`item` | 可進化、可得新圖鑑進化、需道具進化等候選 | 進化清單 | 不等於最值得進化；資源與招式仍需考慮 |
| 物種名、`1-151` 等圖鑑編號／區間 | 特定物種或圖鑑編號集合 | 以家族檢視與批量比較 | 有暱稱、型態或 UI 本地化時需確認命中是否符合預期 |
| 標籤名／Tag filter | 已賦予該標籤的個體 | 取回工作隊列 | 標籤正確性取決於人先前有沒有做對 |
| `!` | 排除後方條件 | 保護排除 | 複合條件最容易寫錯，務必先看結果數 |
| `&` | 同時符合（AND） | 組成嚴格候選池 | 條件太多會產生空集合或漏掉預期個體 |
| `,` | 任一符合（OR） | 合併種類／多個年分／多類別 | 在刪除搜尋中誤用 OR 會把範圍放得很大 |

註：`#`／標籤文字的實際輸入提示在不同版本、語系可能不同；建議從遊戲內 **Tags** 面板點選或直接輸入完整 tag 名稱並核對結果，別把網路截圖的格式硬套到自己的版本。

### 2.2 可用的「候選池」搜尋範例

以下是**把人力集中到候選池**的例子，不是「可直接傳送」指令。每條都應先確認結果頁面、檢查前幾十隻，然後才加暫存 tag。

| 查詢 | 找什麼 | 刻意排除 | 盲用危險 |
| --- | --- | --- | --- |
| `0*&age0-7` | 本週捕獲且 0* 的 inbox | 舊收藏不受影響 | 可能含剛抓的異色／服裝／稀有或朋友指定交易品 |
| `0*,1*&age0-30` | 本月低星候選；`,` 是 OR | 不自動排除特殊狀態 | 範圍很大；先加 `REVIEW`，不要 mass transfer |
| `0*&!favorite` | 未收藏的 0* | Favorite 的保留品 | 收藏疏漏、PvP／稀有／活動個體仍可被納入 |
| `year2016-2018&!favorite` | 舊年分、尚未保護者 | 已 Favorite 的舊品 | 這是「最該人工看」而不是最該刪的集合 |
| `shiny,shadow,lucky,costume,legendary,mythical,ultrabeasts` | 特殊品總盤點（OR） | 普通個體 | 是審計／加標籤用，不是代表全部無限保留 |
| `evolvenew&!favorite` | 尚能開新圖鑑、未收藏者 | 已收藏個體 | 進化前應檢查是否有更高優先版本、是否值得花糖果 |
| `物種名` 或同一進化線的多物種 OR | 同家族並排比較 | 無關物種 | 需要先定義該家族要留 raid／collect／trade 各幾隻 |
| `標籤 REVIEW` | 所有未決個體 | 已完成判斷者 | 若 REVIEW 從不清空，會變成第二個垃圾桶 |

### 2.3 一個安全的查詢組裝順序

1. 先打只讀條件，例如 `age0-7`，確認數量；
2. 再加入星等，如 `&0*`；
3. 視情況加入**一項**排除，像 `&!favorite`；
4. 在列表人工確認幾頁，才賦予 `XFER-CHECK`；
5. 以 `XFER-CHECK` 開第二次 session，不從原始查詢直接傳送。

這比追求一條超長的「神奇清理字串」可靠得多。

---

## 3. 標籤系統：少量永久理由 + 少量暫時動作

### 3.1 建議的可維護 taxonomy

不建議一開始建立十幾個重疊、又每個都永久保留的標籤。較能擴張的結構是 6–10 個左右，並以名稱前綴分開「為何留」與「下一步」。

| 類型 | 例子 | 用途 | 何時移除 |
| --- | --- | --- | --- |
| 永久用途 | `K:RAID`、`K:MEGA`、`K:COLLECT` | 說明保留理由 | 只有用途失效或主動重整才改 |
| 永久脈絡 | `K:LEGACY`、`K:TRADE`、`K:MEMORY` | 對舊帳號尤其重要：讓未來的自己知道為什麼留下 | 交易完成／確認不保留時 |
| 投資隊列 | `Q:POWER`、`Q:EVOLVE` | 有明確但未完成的下一步 | 強化／進化／放棄後 |
| 暫存清理 | `Q:REVIEW`、`Q:XFER-CHECK` | 把候選與決策隔離 | 第二次審核完成後必須清空 |
| 系統旗標 | 直接用 `Favorite` | 防止誤傳送 | 僅當確定可刪時移除 |

`K:`／`Q:` 是命名前綴範例，不是遊戲語法要求。它的好處是 Tags 畫面自然分組；emoji 能加速辨識，但不要只靠 emoji，日後 OCR／匯出或跨平台查找較脆弱。

### 3.2 什麼時候標籤太多？

出現以下任一情況，就該合併：

- 同一隻平均要掛三個以上「幾乎同義」標籤；
- 無法在 5 秒內說出每個 tag 對應的下一步；
- `REVIEW` 累積數月不減；
- 用 tag 取代搜尋的客觀欄位（例如每隻都標 `3*`）。

標籤的工作是保存**搜尋無法表達的意圖**，不是複製遊戲已提供的條件。

---

## 4. 外部工具：資料路徑與風險不能混為一談

| 工具／類別 | 平台 | 掃描方式 | 批量能力 | 庫存 DB／匯出 | 登入／伺服器 | 自動輸入 | 2026 查核 | 風險判讀 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [Poke Genie](https://pokegenie.net/) | iOS、Android | 螢幕讀取／截圖、鑑定頁 | 可逐隻累積 scan history；不是一鍵全倉庫 export | 有 collection history／可排序；公開產品頁未見完整原始 CSV 承諾 | 不需 Pokémon GO 登入；另有自己的 raid 功能帳戶情境 | iOS Keyboard 可貼入改名文字；非自動遊玩 | iOS App Store 8.17.0（2026-08-10）仍在修 UI scan | 螢幕讀取較低風險；廠商自稱合規，但官方未逐一背書 |
| [Calcy IV](https://play.google.com/store/apps/details?id=tesmath.calcy) | Android | overlay／螢幕辨識鑑定資料 | 以逐隻快速讀值為主 | 可做命名／資料輔助；不是官方完整匯出 | 不應提供 Pokémon GO 帳密 | Overlay；使用者觸控為主 | Google Play 目前仍有上架頁；本次無法擷取版本紀錄 | Overlay/讀屏屬灰區；不要當成官方許可 |
| [Pokebattler](https://www.pokebattler.com/) | Web | 使用者建立 Pokebox／資料輸入 | 不適合原始倉庫掃描 | 有 Pokebox、隊伍與模擬 | 有網站帳號，不需 Pokémon GO 帳密 | 無 | 站上有 2026-08 raid 分析內容 | 低：分析站；避免把帳密交給非官方服務 |
| [pogo-storage-mapper](https://github.com/WorkOfStan/pogo-storage-mapper) | 桌面／Python | 本機 screenshot / screen recording，Tesseract + frame evidence | 可讀資料夾／影片；完整細節很慢 | CSV、XLSX、audit artifacts | 正常掃描不連遊戲伺服器 | 無 | 2026-08 有 push；1 star、MIT、作者定位為 POC | 較低：離線只讀；尚不成熟、裝置解析度耦合 |
| [pogo-inventory-ai](https://github.com/Keremunce/pogo-inventory-ai) | Browser / Next.js | 上傳格狀截圖、Tesseract.js | 多圖 merge 尚列 future work | LocalStorage、JSON；IV 需人工補 | 不接 Pokémon GO；可選用 OpenAI 作分析 | 無 | 2026-03 最後 push；1 star、無 license 設定 | 較低：本機 screenshot；MVP 不足以完整盤點 |
| [pogo-ipad-renamer](https://github.com/widmonstertony/pogo-ipad-renamer) | iPad + desktop | OCR／像素讀鑑定條 | 可批次，但設備／解析度高度校準 | 本地 journal／診斷資料；目的為改名 | 不宣稱登入遊戲伺服器 | **有**受控點擊與改名 | 2026-08 push；0 star、無 license | 較高：雖不自動遊玩，UI input 仍可能被視為未授權第三方 add-on |
| Google Sheets / 本地 SQLite / CSV | 任意 | 手動或匯入 screenshot OCR 結果 | 取決於匯入器 | 強：可加入理由、歷史、審核欄 | 不需遊戲帳密 | 無 | 通用基礎工具 | 最低：資料在遊戲外管理 |

### 4.1 Poke Genie 的具體定位

Poke Genie 官方產品頁表示可由 Pokémon 頁／隊長鑑定畫面立即讀 IV，並保存掃描歷史作 collection management；其 iOS listing 顯示 2026-08 更新仍修復因 Pokémon GO v0.423 UI 改動造成的 screen scanning。這是「仍在維護」的可見訊號，但不是全倉庫無人值守盤點的證據。

產品 listing 又稱其僅依賴 screenshot/screen recording 且不要求 login；這是**廠商聲明**。官方公平遊戲政策並未點名認可 Poke Genie，因此正確措辭只能是「技術上不需帳密／不直接呼叫遊戲伺服器」，不可說 Niantic 已認證安全。

### 4.2 Pokémon HOME 與資料匯出

本研究沒有發現可以把官方 Pokémon GO storage 直接匯為完整 CSV／JSON 的正式功能。Pokémon HOME 的傳送不應被視為備份或清理預覽：它是遊戲內的轉出決策，沒有解決「比較所有副本、留下理由」這個核心問題。任何要求 Pokémon GO 帳密、cookie、token、或聲稱能直接抓取 inventory 的第三方，應歸為高風險候選並先排除。

---

## 5. 既有的大型掃描／資料庫工程方法

### 5.1 格狀清單截圖：速度快、資訊不足

做法是連續截取 Pokémon list grid，OCR 名稱、CP、縮圖／特殊圖示，然後合併入本機資料表。`pogo-inventory-ai` 就是這一型：能把 screenshot 轉成 JSON、做本機搜尋，但 README 明示 IV 尚需手動輸入、多圖合併仍是未來項目。

- 優點：只讀、好重播、能先建立「有幾隻／哪些物種重複」的骨架。
- 缺點：無法可靠判斷 IV、招式、保留理由；暱稱、不同型態、圖示／語系皆可能讓 OCR 出錯。

### 5.2 詳情／鑑定 carousel：資訊完整、吞吐量低

開啟單隻詳情與 appraisal，逐隻滑動，錄影後離線抽 frame。`pogo-storage-mapper` 的做法是：先將 frame 分為 list/detail/appraisal/non-extractable，再只把證據明確者輸出；其設計有 audit artifacts，這點非常適合不想默默誤判的個人 curator。

但作者明載：讀完整細節時，30 秒、約 20 隻的影片，仍可能每隻處理 5–10 分鐘；且現階段對 Pixel 9 的尺寸／錄影格式有耦合。結論是：**完整 carousel scanner 可行，但現在不適合當第一次 2,000 隻盤點的唯一入口。**

### 5.3 OCR + 可驗證欄位，而非只信名字

這個 POC 發現：暱稱令名字 OCR 不可靠，CP 又會因異色／暗影／幸運等畫面變體不清楚；目前較可辨識的組合是 HP + weight。這是很重要的工程教訓：

- 每一筆要保留來源截圖／frame、置信度與欄位級 evidence；
- 不確定的列必須進 `needs_review`，不能猜測補值；
- identity 需要多欄交叉比對，不能只用名字或 CP；
- device resolution 應是 scanner profile 的一部分。

### 5.4 ADB、UIAutomator、Accessibility、錄影

這些可大幅降低「人工截圖與整理檔案」成本，但風險隨能力上升：

- **只截圖／錄影、離線解析：** 可讀、可重播，不改遊戲狀態。
- **自動開詳情、滑動、開鑑定：** 即使不傳送，也開始對正式 client 施加自動輸入；官方並未給出逐工具的 green light。
- **自動改名、強化、進化、傳送或對戰：** 明確改變遊戲狀態，與「未授權第三方軟體或 add-ons」規範距離更近。

因此研究期應停在第一層，至多把第二層當理論比較，不能把 UI automation 當「safe scanning」。

---

## 6. 2,000+ 隻帳號的效能比較

下列是量級估算，目的在找 bottleneck，不是效能保證。實際值受手機、熟練度、畫面格式與保留標準影響。

| 方法 | 假設 | 2,000 隻量級 | 分類 | 真正瓶頸 |
| --- | --- | --- | --- |
| 全手動打開＋鑑定＋判斷 | 每隻 20–45 秒 | 約 11–25 小時 | VERY SLOW | 注意力與決策疲勞 |
| 先用搜尋縮至 300 候選，再人工鑑定 | 300 × 25 秒，加上搜尋／tag 1–2 小時 | 約 3–5 小時 | PRACTICAL | 能否正確建候選池 |
| 同家族整理（假設 150 個家族 × 2–5 分鐘） | 先搜物種，定每家族保留上限 | 約 5–12.5 小時，適合分週做 | PRACTICAL | 收藏／交易理由需要人工決定 |
| 格狀 screenshot index | 4–6 隻／畫面、約 350–500 張；人工截圖 1–2 秒／張、後端批處理 | 捕捉約 15–30 分鐘；資料清洗 1–3 小時 | FAST | 名稱／CP OCR 與去重、缺少 IV |
| 錄影＋完整 detail/appraisal OCR | 以既有 POC 5–10 分鐘／隻完整 extraction | 約 167–333 小時處理時間 | VERY SLOW | OCR 與 frame 解析，而非錄影本身 |
| Poke Genie 類逐隻 scan | 每隻約 5–15 秒人機節奏 | 約 3–8 小時 | SLOW–PRACTICAL | 手動切換／掃描；最適合先掃候選而非所有隻 |
| 純本機 curator：grid index + 選擇性 detail review | 先 index 全部，僅 10–20% 不確定者開詳情 | 初次 2–5 小時，後續增量很小 | FAST | 設計可靠的 merge 與 review queue |

最可擴張的策略不是「把 2,000 隻讀到完全完美」，而是先用 list-level index 把 2,000 隻變成 100–400 個需要看詳情的決策。

---

## 7. 破壞性動作的安全守則

1. **先 Favorite，後搜尋。** 所有明確要留者，尤其異色、服裝、神獸、暗影、幸運、紀念、已投資者，先保護。
2. **搜尋結果不是刪除清單。** 先加 `Q:XFER-CHECK`，隔一個 session 再處理。
3. **每次只做一個理由明確的批次。** 例如「本週 0* 的普通捕獲」，不要把舊年分、交易與活動紀念混進去。
4. **先備份可見狀態。** 對待審核集合截圖或保存本機 CSV；要能回答「我為何刪了它」。
5. **不信任超長排除字串。** 一次只加一項條件並檢查結果數與前幾頁。
6. **保留 REVIEW 的期限。** 例如 30 天內決定，否則定期重新分派；避免變成永久堆積。
7. **人類核准是最後一道門。** 本機 curator 可以提出「疑似冗餘」，但不應自動發送 transfer／favorite 移除操作。

---

## 8. 條款與帳號風險：文件事實 vs. 推論

Pokémon GO 的 [Gameplay Fairness Policy](https://niantic.helpshift.com/hc/en/6-pokemon-go/faq/39-gameplay-fairness-policy-1701994322/) 將偽造位置及「以第三方軟體或 add-ons 未授權存取遊戲 client 或 backend」列為作弊，並描述警告、暫停、終止的處分，且保留不經三擊立即終止的權利。Scopely Explore 的 [Terms](https://explore.scopely.com/terms/) 與 [Player Guidelines](https://explore.scopely.com/guidelines/) 也禁止未授權軟體、帳號／多帳號濫用與 spoofing。

| 類別 | 技術描述 | 政策可確定程度 | 實務處理 |
| --- | --- | --- | --- |
| 離線 screenshot／錄影分析、OCR、本機 SQLite／試算表 | 用使用者提供的影像處理，不連帳號／伺服器、不輸入遊戲 | 官方未逐一認證，但不具「未授權存取 client/backend」特徵 | **較低風險**；保持 offline、無帳密 |
| 純 overlay／螢幕讀取 | 讀取顯示內容、提供建議 | 官方沒有逐項許可 | **模糊**；不稱安全、密切留意 app 改版與條款 |
| ADB／Accessibility 導航、swipe、改名 | 對正式 client 自動送輸入 | 對應「未授權 third-party add-on」風險，官方未發白名單 | **較高風險**；研究階段不做 |
| 帳密／token／cookie、未官方 API、封包攔截、修改 client、自動遊玩 | 直接存取帳號／client/backend 或改變遊戲行為 | 最接近官方明列禁止行為 | **高風險**；排除 |

這是基於公開政策文字的保守分類，而非宣稱某方法被允許。社群 anecdote 無法改變官方條款。

---

## 9. 既有工具還缺什麼？

研究顯示缺口不是「讀出某隻的 IV」，而是下列資料產品能力：

- **跨物種／同家族的比較視圖：** 一次看所有副本、型態與明確用途，而非逐隻彈窗。
- **保留決策記憶：** `why_kept`、來源活動、上次檢視、誰／何時決定；這正是多年帳號最缺的脈絡。
- **不確定性與證據：** 每個 OCR 欄位要有 source frame、置信度、人工修正紀錄，不把推測偽裝成資料。
- **分層擷取：** 快速 grid index 全倉庫，只有去重／保留衝突時才要求 detail/appraisal evidence。
- **human-approved cleanup queue：** 模型只給候選與理由，遊戲內 favorite／transfer 永遠由人手執行。
- **增量同步：** 一旦完成初始基線，之後只掃新增捕捉／本週變動，而非重掃 2,000 隻。

---

## 10. 對個人 Pokémon Collection Curator 的架構選項

| 架構 | 輸入與流程 | 複雜度 | 可靠性／速度 | 帳號風險 | 人工需要 |
| --- | --- | --- | --- | --- | --- |
| A. Screenshot-only advisor | 手動擷取 grid 截圖 → OCR → local SQLite → 同家族比較與 REVIEW queue | 低–中 | 名稱／CP 快、IV 缺失；需可人工校正 | 較低 | 截圖與最終決策 |
| B. Screen-recording indexer | 人手慢滑倉庫並錄影 → frame dedupe／OCR → provenance DB | 中 | 擷取快、離線可重跑；對 UI 尺寸敏感 | 較低 | 錄影、抽樣校正 |
| C. Appraisal carousel scanner | 詳情／鑑定錄影 → IV/move OCR → evidence merge | 高 | 資料完整、速度慢；最難的是 identity merge | 較低（只錄影）／若自動滑動則升高 | 大量手動滑動或高風險自動導航 |
| D. Hybrid：GO search + CV indexer | 先在官方 app 搜物種／候選池，再截圖／錄影；curator 提出同家族保留方案 | 中 | 將高成本 OCR 限在 10–20% 候選，最符合大型帳號 | 較低（只讀模式） | 搜尋與核准 |

**設計原則：** 所有架構都應從 offline、read-only、human-approved 開始；永遠不保存 Pokémon GO 帳密，永遠不發送遊戲輸入，並保留資料來源與刪除前的審核記錄。

## Recommended First Experiment

**做一個「10 個同一進化家族的 screenshot-only comparer」小實驗，而不是掃全帳號。**

範圍：從官方 app 手動以搜尋選出 10 個重複多的家族，各截取其 list grid 與少量明確候選的鑑定頁；將圖片離線轉成表格，人工核對每列，並讓介面顯示「此家族已有幾隻、每隻的 CP／星等／特殊狀態／保留理由／不確定性」。

成功條件只有三個：

1. 能否正確把 10 個家族的副本合併，而不是只做 OCR demo；
2. 能否讓 Jones 在 30 秒內回答「為什麼我留這幾隻」；
3. 能否產生**待人工確認**而非自動刪除的候選清單。

這個實驗會最快驗證最關鍵的產品風險：不是「能不能讀畫面」，而是資料是否足以改善真正的收藏決策。

---

## 來源與查核記錄

查核日期為 2026-09-01；產品功能與 app UI 可能隨版本改變。

### 官方政策

1. [Pokémon GO Gameplay Fairness Policy](https://niantic.helpshift.com/hc/en/6-pokemon-go/faq/39-gameplay-fairness-policy-1701994322/)
2. [Scopely Explore Terms — Cheating](https://explore.scopely.com/terms/)
3. [Scopely Explore Player Guidelines — Play fair](https://explore.scopely.com/guidelines/)

### 現行工具／資料庫

4. [Poke Genie 官方網站](https://pokegenie.net/)
5. [Poke Genie iOS App Store listing](https://apps.apple.com/us/app/poke-genie-remote-raid-iv-pvp/id1143920524) — 2026-08-10 8.17.0 scan 修正、scan organizer、讀屏／無 login 的廠商描述。
6. [Calcy IV Google Play](https://play.google.com/store/apps/details?id=tesmath.calcy)
7. [Pokebattler](https://www.pokebattler.com/) — 站上可見 2026-08 raid analysis，確認仍活躍。

### GitHub（已檢查 README、最近 push、stars／license；不是成熟度背書）

8. [WorkOfStan/pogo-storage-mapper](https://github.com/WorkOfStan/pogo-storage-mapper) — MIT、1 star、2026-08 push；離線錄影／截圖、CSV/XLSX、明確承認 POC 與完整 extraction 慢。
9. [Keremunce/pogo-inventory-ai](https://github.com/Keremunce/pogo-inventory-ai) — 1 star、2026-03 push、未設定 license；browser-local screenshot OCR MVP。
10. [widmonstertony/pogo-ipad-renamer](https://github.com/widmonstertony/pogo-ipad-renamer) — 0 star、2026-08 push、未設定 license；讀鑑定條後自動改名，README 自行警告 UI automation 風險。
11. [matthiasharzer/go-stats-tracker](https://github.com/matthiasharzer/go-stats-tracker) — MIT、0 star、2026-09 push；以 screenshot OCR 追蹤 XP，顯示「螢幕截圖 → Sheets」的可行資料管線，但不是 inventory scanner。

### 社群線索（不可視為統計）

12. [TheSilphRoad storage-management 討論](https://www.reddit.com/r/TheSilphRoad/comments/10g5a9p/pokemon_storage_management/)
13. [r/pokemongo storage-management 討論](https://www.reddit.com/r/pokemongo/comments/15e7e6e/how_do_you_manage_your_pokemon_storage/)


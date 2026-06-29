# Decision Field Lab：點子發酵與交匯點資料結構調查

> 日期：2026-06-29  
> 狀態：研究報告  
> 用途：評估 Decision Field Lab 目前的張力圖 UI，整理坊間可參考的資料結構與思考工作流，並提出下一步可落地的「點子發酵」輔助設計。  
> 隱私命名：公開文件使用 `JCompany`，不使用內部專案名。

## 一句話結論

Decision Field Lab 最值得走的方向，不是做成另一個 Roam、Obsidian Canvas 或 Heptabase，而是做成一個「張力交匯點發酵器」：使用者丟入一段想法，AI 把它拆成節點、關係、風險、槓桿、反例與下一步實驗，然後在圖上標出真正值得追問的位置。

現有 schema 已經有很好的骨架：

- `nodes`：想法中的因素、機制、風險、槓桿、結果。
- `edges`：因素之間的因果、推動、阻礙、橋接。
- `hotspots`：交匯點，也就是「最值得發酵」的位置。
- `feedback`：agent 對 topic / case 的追問、批判、補強。
- `revisions`：保留每次 AI 改圖的歷史。
- `crossLinks`：跨案例、跨主題的類比與關聯。

真正下一步不是大改 UI，而是把 `hotspots` 從「圖上的亮點」升級成「思考工作單元」。

## 目前 app 呈現出的強項

從目前手機畫面看，Decision Field Lab 已經有幾個很有辨識度的方向：

- 它不是普通心智圖，而是「張力圖」。節點被分成 `機制 / 誘因 / 篩選 / 槓桿 / 風險 / 結果 / 失效`，比一般 free-form note 更接近決策分析。
- 它不是純白板，而是有「交匯」概念。使用者不只看節點，也可以追一條推理路徑。
- 它不是一次性輸入框，而是 agent ingest。AI 可以反覆 patch、留下 revision，這讓想法可以長出版本歷史。
- 它目前已經適合單人創作者的私人工具，不必過早做成多人協作 SaaS。

也因此，我會把它定位成：

> 一個把模糊想法轉成可追問圖譜的 AI cognitive graph system。

## 坊間可參考的資料結構與做法

### 1. Obsidian Canvas / JSON Canvas：白板節點與連線

Obsidian Canvas 的核心是「在無限畫布上放卡片、媒體、筆記，並用連線組織想法」。官方也把 Canvas 檔案存在開放的 JSON Canvas 格式，方便工具、script、plugin 增強卡片與連線。

可借鏡之處：

- 每個 node 都應該有穩定 `id`、座標、尺寸、type。
- edge 不只連接，也可以有 label、方向、顏色。
- group / container 可以用來代表一組概念或某個思考區域。
- local-first / exportable format 對個人工具很重要。

對 Decision Field Lab 的啟示：

- 目前 `nodes / edges` 已經接近 JSON Canvas 的核心精神。
- 可以增加一個「group / zone」概念，把同一組節點歸進同一個思考區，例如 `市場`, `儀式`, `AI`, `風險`, `商業化`。
- 未來可提供 `Export as JSON Canvas`，讓圖可以進 Obsidian Canvas 或其他工具保存。

來源：

- [Obsidian Canvas](https://obsidian.md/canvas)
- [JSON Canvas Spec](https://jsoncanvas.org/spec/1.0/)

### 2. Heptabase：白板 + 卡片 + AI 學習/研究流程

Heptabase 把自己定位為 visual knowledge base，強調用 whiteboards and cards 釐清思考，並把 sources、notes、highlights、discussions 放在同一個 connected knowledge base 裡。

可借鏡之處：

- 它不是只畫圖，而是把「來源、卡片、討論、AI 解釋」放在同一個研究環境。
- 對複雜問題來說，card 應該可被反覆引用，而不是只存在一張圖裡。
- AI 的角色不是替使用者下結論，而是帶著來源與脈絡推進理解。

對 Decision Field Lab 的啟示：

- `node.summary` 可以升級成更完整的 card：包含來源、原句、AI 解釋、待驗證假設。
- `artifactMd` 可以成為人類可讀的白板摘要，不只是 export 欄位。
- 對 JCompany 類主題，應保留「原始念頭 → AI graph → 後續追問 → 生成文稿」的完整鏈條。

來源：

- [Heptabase](https://www.heptabase.com/)

### 3. Kumu / Systems Mapping：系統、連線與 feedback loops

Kumu 的 systems mapping 強調 elements、connections、loops，尤其是 reinforcing loop / balancing loop。它適合研究「某些因素如何互相強化或互相抵消」。

可借鏡之處：

- 除了單向因果，系統圖應該能標記「迴圈」。
- loop 可以是 reinforcing，也可以是 balancing。
- loop 本身值得有 label、敘事、風險與介入點。

對 Decision Field Lab 的啟示：

- 現在 `edges` 已可表達方向，但還缺「loop」這個一級物件。
- 可以新增 `loops`，或先用 `hotspots.type = loop` 表達。
- 對點子發酵來說，最有價值的不是找很多節點，而是找出：哪一條路徑會自我增強？哪一條會抵消？哪裡能介入？

建議資料欄位：

```json
{
  "id": "ritual_trust_loop",
  "type": "reinforcing",
  "label": "儀式感提升信任，信任提升再次使用",
  "path": ["ritual_design", "felt_seen", "trust", "repeat_use"],
  "polarity": "R",
  "leveragePoint": "felt_seen",
  "risk": "如果儀式變成表演，loop 會反向破壞信任。"
}
```

來源：

- [Kumu Systems Mapping](https://docs.kumu.io/disciplines/system-mapping.md)
- [Causal Loop Diagram](https://en.wikipedia.org/wiki/Causal_loop_diagram)

### 4. IBIS / Issue Mapping：問題、立場、論據

IBIS 是為了處理 wicked problems 的 argumentation graph。它的基本元素是：

- issue：要回答的問題。
- position：可能答案或立場。
- argument：支持或反對某個 position 的論據。

這套方法的重點不是漂亮，而是讓模糊問題變得可追蹤，並把決策脈絡透明化。

可借鏡之處：

- 每個 hotspot 都應該被轉成一個 issue。
- 每個 issue 底下可以有多個 position，而不是只有一條 AI 結論。
- 每個 position 要有 pros / cons / assumptions。

對 Decision Field Lab 的啟示：

- 現在 `hotspot.nextQuestion` 很接近 issue，但還是文字欄位。
- 可以讓 `hotspot` 裡增加 `positions`，把交匯點變成真正的討論單元。
- 這特別適合 JCompany 這種仍在形成中的產品策略，因為很多問題不是「對錯」，而是「哪種假設更值得下注」。

建議資料欄位：

```json
{
  "id": "hotspot_ritual_or_social",
  "title": "JCompany 應該強化儀式感，還是強化社交性？",
  "issue": "使用者留下重要念頭後，最需要被誰回應？",
  "positions": [
    {
      "id": "ritual_first",
      "claim": "先做成私人儀式，不追求社交曝光。",
      "pros": ["符合安放、封存、告別的定位", "避開社群平台焦慮"],
      "cons": ["成長速度可能較慢", "分享機制需要更精緻"]
    },
    {
      "id": "social_first",
      "claim": "先讓願望被他人看見與回應。",
      "pros": ["容易形成網路效應", "內容流動性較高"],
      "cons": ["可能滑向表演與比較", "會削弱私密儀式感"]
    }
  ]
}
```

來源：

- [Issue-based information system](https://en.wikipedia.org/wiki/Issue-based_information_system)

### 5. Concept Map：概念、命題與 linking phrase

Concept map 與一般 mind map 的差異在於：它不是只從中心往外發散，而是用 labeled arrows 表達概念之間的命題，例如 `A causes B`、`A requires B`、`A contributes to B`。

可借鏡之處：

- edge label 很重要，因為它把「關聯」變成一句可討論的命題。
- concept map 可以有多個 hub，不必只有單一中心。
- 對學習與研究來說，交叉連結比樹狀分類更重要。

對 Decision Field Lab 的啟示：

- 現在 `edge.verb` 已經是正確方向，應該把它當成一級思考材料。
- Agent 不應只寫 `推向`、`影響` 這種泛詞，而應產生更精確的 verb：`降低不確定性`、`製造身份訊號`、`轉化為儀式`、`破壞信任`。
- 交匯點可以由「edge verb 的衝突」生成，而不只是 node path。

來源：

- [Concept map](https://en.wikipedia.org/wiki/Concept_map)

### 6. Zettelkasten：連結不是收藏，而是思考生產

Zettelkasten 的核心不是「很多筆記」，而是讓想法形成可導航的 web of thoughts。它強調 connection, not collection。

可借鏡之處：

- 每個想法應該小而可連結。
- 新想法的價值在於它連到舊想法時產生什麼變化。
- entry point 很重要，否則網狀筆記會變成雜物堆。

對 Decision Field Lab 的啟示：

- `crossLinks` 不應只是補充功能，而應成為「發酵」核心。
- 當新 case 寫入時，agent 應自動找 3 種 cross-link：
  - 同主題內相似概念。
  - 不同主題的類比。
  - 與舊假設衝突的反例。
- 每個 cross-link 應該帶有 `relationType`，例如 `analogy`, `contradiction`, `ancestor`, `mutation`, `risk_echo`。

來源：

- [Introduction to the Zettelkasten Method](https://zettelkasten.de/introduction/)

### 7. Graph of Thoughts：AI 不只 chain，也可以 merge / refine / feedback

Graph of Thoughts 的重點是：LLM 產生的 thought 可以被建模成任意 graph，edge 代表依賴關係，並且可以做 combine、distill、feedback loop。

可借鏡之處：

- AI 推理不必只有 linear chain。
- 不同 thought 可以合併成 synergetic outcome。
- feedback loop 可以用來改良 thought。

對 Decision Field Lab 的啟示：

- `revisions` 不只是版本備份，也可以成為思考演化歷史。
- 可以讓 agent 每次 patch 都標記 transformation type：
  - `extract`：從 raw thought 抽節點。
  - `expand`：補更多可能節點。
  - `criticize`：找風險與反例。
  - `merge`：和舊 case 合併。
  - `distill`：壓縮成核心命題。
  - `experiment`：生成下一步驗證。
- 這樣未來可以看見：一個點子如何從模糊直覺變成可行假設。

來源：

- [Graph of Thoughts: Solving Elaborate Problems with Large Language Models](https://arxiv.org/abs/2308.09687)

### 8. Wardley Mapping：把價值鏈與成熟度放進同一張圖

Wardley Mapping 的價值在於：它不只畫元件關係，也把「使用者價值」與「演化成熟度」放在同一張圖上，用來看策略位置。

可借鏡之處：

- 策略圖不只要知道節點互相連接，還要知道每個節點處在什麼成熟階段。
- 同一件事可能從新奇、客製、產品化、走向基礎設施。
- 這和 Jones 最近討論的品牌基準線、JCompany 文化位置非常相合。

對 Decision Field Lab 的啟示：

- 對品牌/產品策略 case，可以給每個 node 增加 `maturity`：
  - `novelty`
  - `status`
  - `mainstream`
  - `infrastructure`
  - `invisible`
- 對 JCompany 這類題目，這會比一般 x/y 座標更有意義：不是只是「它在哪裡」，而是「它正在往哪個文化階段演化」。

來源：

- [Learn Wardley Mapping](https://learnwardleymapping.com/book/)

### 9. LiquidText：保留原文脈絡，而不是只萃取摘要

LiquidText 的特色是把文件、highlight、note、connection 放在同一個閱讀空間，並且能在跨頁、跨文件之間看見關係與原始脈絡。

可借鏡之處：

- 抽取後不能丟失來源。
- 每個 AI 生成節點最好能回到原始段落。
- 搜尋與 highlight 應該能保留 context。

對 Decision Field Lab 的啟示：

- `node` 可以增加 `sourceSpan` 或 `sourceQuote`。
- `artifactMd` 應保留原始討論摘錄與 AI 重構後的差異。
- 當 agent 把對話整理成 graph 時，重要節點應能追溯到原句。

來源：

- [LiquidText](https://www.liquidtext.net/)

## 建議的下一代資料結構

不建議立刻重寫 Firestore schema。比較穩的做法是先用向後相容欄位擴充。

### 1. Node：從因素升級成「可驗證 thought card」

目前：

```json
{
  "id": "trust_market",
  "label": "信任型市場",
  "role": "filter",
  "summary": "..."
}
```

建議增加：

```json
{
  "claim": "JCompany 的市場不是注意力市場，而是信任型市場。",
  "sourceQuote": "原始對話中的關鍵句。",
  "assumptions": ["使用者願意為安放感回訪", "私密儀式比公開互動更符合核心場景"],
  "evidence": ["類比：7-Eleven 競爭信任而非新鮮感"],
  "confidence": 0.72,
  "maturity": "novelty"
}
```

### 2. Edge：從連線升級成「命題」

目前：

```json
{
  "from": "ritual_design",
  "to": "repeat_use",
  "verb": "推動",
  "cat": "pos"
}
```

建議增加：

```json
{
  "proposition": "儀式設計若讓使用者感覺被好好安放，會提高回訪意願。",
  "polarity": "+",
  "strength": 0.65,
  "assumption": "使用者在重要念頭上追求的是見證，而不是曝光。",
  "counterEdge": "如果儀式變成表演，會降低信任。"
}
```

### 3. Hotspot：從亮點升級成「發酵工作單元」

目前 hotspot 已經有 `whyMatters / designLever / nextQuestion / failureMode / experiment`，方向很好。建議下一步補：

```json
{
  "issue": "JCompany 應該先做私密儀式，還是先做可分享社交？",
  "positions": [],
  "contradictions": [],
  "missingEvidence": [],
  "nextAgentActions": [
    "找三個相似產品的儀式設計",
    "列出三個會滑向社群焦慮的失敗模式",
    "把這個 hotspot 轉成一個 MVP 測試"
  ],
  "fermentationStage": "question"
}
```

建議 `fermentationStage`：

- `seed`：剛抽出的模糊直覺。
- `question`：已形成值得問的問題。
- `positions`：已有多個可比較立場。
- `tension`：已找到矛盾或取捨。
- `experiment`：可轉成下一步測試。
- `artifact`：可輸出成報告、文稿、產品規格或 prompt。

### 4. CrossLink：把「想到很像」變成正式資料

目前 cross-link 已存在，建議增加關係類型：

```json
{
  "relationType": "analogy",
  "from": {"topicId": "jcompany", "caseId": "...", "nodeId": "ritual_design"},
  "to": {"topicId": "brand-baseline", "caseId": "...", "nodeId": "infrastructure_brand"},
  "reason": "兩者都在研究產品如何從新奇變成生活基準線。",
  "transferableInsight": "不要只追求注意力，要追求某個生活時刻的預設選項。"
}
```

建議 relation types：

- `analogy`：類比。
- `contrast`：對照。
- `contradiction`：矛盾。
- `ancestor`：舊想法是新想法來源。
- `mutation`：舊想法變種。
- `risk_echo`：不同 case 出現同型風險。
- `lever_reuse`：同一設計槓桿可重用。

### 5. Revision：把版本歷史變成思考演化

建議每次 agent patch case 時，在 `_summary` 或 `artifactJson` 裡紀錄：

```json
{
  "transformation": "criticize",
  "inputState": "positions",
  "outputState": "tension",
  "changed": {
    "nodesAdded": 2,
    "edgesAdded": 3,
    "hotspotsAdded": 1
  },
  "agentIntent": "補出此想法最可能失敗的三個位置。"
}
```

這樣 Decision Field Lab 會不只是「資料庫 + UI」，而是能回看一個念頭如何被 AI 發酵。

## 建議的 agent 發酵流程

### 第一步：Raw thought → Seed graph

目標：不要急著完整，先抓出最小可用骨架。

輸出：

- 5-9 個 nodes。
- 6-12 條 edges。
- 1-3 個 hotspots。
- 1 個 artifactMd 摘要。

### 第二步：Seed graph → Issue map

目標：把 hotspot 轉成問題，而不是直接給答案。

Agent 應問：

- 這個 hotspot 真正的 issue 是什麼？
- 至少有哪兩個 position？
- 每個 position 的 pros / cons 是什麼？
- 哪些假設還沒被驗證？

### 第三步：Issue map → Tension map

目標：找矛盾、反例、風險。

Agent 應補：

- contradiction nodes。
- failure edges。
- risk_echo cross-links。
- 可能被忽略的 stakeholder。

### 第四步：Tension map → Experiment map

目標：把想法變成可測試行動。

每個高價值 hotspot 至少產出：

- `nextQuestion`
- `failureMode`
- `experiment`
- `minimumEvidence`
- `stopCondition`

### 第五步：Experiment map → Artifact

目標：輸出給人用的東西。

可輸出成：

- 產品策略 memo。
- JCompany roadmap note。
- landing copy。
- app feature spec。
- social post / essay seed。
- 下一次和 GPT 深聊的 prompt。

## 交匯點偵測規則

我建議先不用做複雜演算法，可以讓 agent 用以下啟發式找 `hotspots`。

### 高價值交匯點

出現以下情況就值得標成 hotspot：

- 一條正向路徑和一條負向路徑交會。
- 同一 node 同時是 `lever` 和 `risk`。
- 某個 edge verb 太泛，代表中間機制沒想清楚。
- 多個 case 都指向同一個抽象概念。
- 某個 node 的假設很多，但 evidence 很少。
- 一個設計選擇同時推動成長與破壞信任。
- 一個想法可轉成 MVP，但 failureMode 尚未被定義。

### Hotspot 分類

建議先用這些 type：

- `opportunity`：可放大的機會。
- `risk`：可能炸掉的地方。
- `contradiction`：兩個命題互相拉扯。
- `missing_evidence`：缺資料但很關鍵。
- `leverage`：小改動可能有大影響。
- `loop`：自我增強或自我抵消迴圈。
- `cross_case`：和舊 case 產生高價值連結。

## 對 JCompany 的具體用法

以目前這張「品牌基準線到願望儀式」case 來說，下一步不應只是再補更多節點，而是問：

1. `文化基準線` 是 JCompany 的長期願景，還是現在就要進入 MVP 的設計原則？
2. `儀式稀缺` 這個命題有沒有反例？例如使用者其實只想要快、好玩、可分享。
3. `不要變成社群平台` 是硬邊界，還是早期產品的暫時策略？
4. `安放、見證、封存、告別` 四個詞是否代表四種不同功能，而不是同一種感覺？
5. 成功指標應該是 DAU、回訪、分享、付費，還是「某個人生時刻會想到它」？

這些問題都很適合變成 hotspots，而不是散在文稿裡。

## 最小可執行下一步

我建議下一輪只做三件事。

### 1. 更新 agent 寫入規格，不急著改 app

在 `docs/agent-workflow.md` 裡補一段「發酵型 graph 寫入建議」，要求 agent 寫入時盡量補：

- `node.claim`
- `node.assumptions`
- `edge.proposition`
- `hotspot.issue`
- `hotspot.positions`
- `hotspot.fermentationStage`
- `crossLink.relationType`

Flutter app 目前會忽略未知欄位，所以可以先在 Firestore 裡開始累積，不必立刻改 UI。

### 2. 做一個「再發酵」agent action

在目前 `pushActions` 概念上，先設計幾個固定 action：

- `追問這個交匯點`
- `找反例`
- `轉成 MVP 實驗`
- `找相似舊 case`
- `濃縮成一句產品命題`

這比做聊天框更可控，也更符合手機 UI。

### 3. 建立 case quality rubric

每個 case 可以有一個簡單評分：

- 是否有清楚 issue？
- 是否有至少兩個 positions？
- 是否有 contradiction？
- 是否有 next experiment？
- 是否有 cross-link？
- 是否有 source trace？

這會讓 Decision Field Lab 從「看起來很酷的圖」變成「真的能把點子推進的工具」。

## 結論

Decision Field Lab 現在的圖已經有一種獨特氣質：它不是筆記、不是白板、不是普通 graph database，而是一個把想法變成張力場的工具。

坊間成熟系統能提供的啟示是：

- Obsidian Canvas / JSON Canvas 提醒我們：圖要可攜、可擴充。
- Heptabase 提醒我們：圖要連回來源、卡片與研究流程。
- Kumu / causal loop 提醒我們：最有價值的是系統迴圈，不只是節點。
- IBIS 提醒我們：交匯點應該變成 issue / position / argument。
- Concept Map 提醒我們：edge label 是命題，不只是裝飾。
- Zettelkasten 提醒我們：真正的知識來自 connection, not collection。
- Graph of Thoughts 提醒我們：AI 可以用 graph 做 merge、distill、feedback，而不是只做一次性摘要。
- Wardley Mapping 提醒我們：產品與品牌位置應該有成熟度與演化方向。

所以我會把下一步產品方向定義成：

> Decision Field Lab 應該幫 Jones 找出「哪個想法值得繼續發酵、哪裡正在產生張力、下一步要問什麼、以及哪個小實驗能讓它往前走」。

這比做一張漂亮的圖更重要，也更接近 JCompany 未來真正需要的思考基礎設施。

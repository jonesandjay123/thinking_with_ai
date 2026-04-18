# OpenClaw Slack 跨 Thread 記憶研究

## 為什麼會有這份研究

Jones 在 Slack 上的真實工作流非常依賴 thread。現在 OpenClaw 在「同一個 Slack thread 內的延續感」已經比以前好很多了，但這還沒有自動解決下一個更關鍵的問題：

> 如果 Jones 開了另一個 Slack thread，Jarvis 要怎麼還能像「同一個 Jarvis」一樣延續，而不是逼 Jones 再重新交代一次背景？

這份報告研究的是這個問題背後更一般化的版本，也就是：**thread 範圍內的短期記憶 + 跨 thread 共用的共享記憶層**。

目標不是隨便找一個外部工具硬套，而是萃取目前業界較成熟的設計思路，判斷哪些東西真正適合 OpenClaw + Slack + Jones 的實際使用習慣。

---

## 執行摘要

現代 agent 系統裡最常見、也最穩的記憶設計模式，大致都指向同一件事：

1. **短期記憶要維持 thread-scoped**
   - 一個 Slack thread 或一段對話，保有自己的局部上下文
2. **跨 thread / 跨 session 再加一層共享記憶**
   - 使用者偏好、進行中的專案、最近的重要決策、正在延續的工作流，不應只存在單一 thread 裡
3. **不要把所有東西都塞進一個巨大的 memory blob**
   - 穩定的長期記憶，和會變動的工作記憶，應該分層
4. **在新 thread 開始時做選擇性召回，而不是全量載入**
   - 新 thread 不該把整個世界都塞進 prompt，只該帶入最相關的共享上下文

換句話說，業界主流答案**不是**「把所有 Slack threads 偷偷都變成同一個 session」。
真正成熟的答案比較像：

> 「讓 session 保持局部乾淨，但另外建立一層可以跨 session 被召回的共享記憶。」

這和 Jones 現在遇到的痛點，其實幾乎完全重疊。

---

## OpenClaw 目前已經有什麼

從 OpenClaw 文件和這次實際使用觀察來看：

- Gateway 本身就是 session / routing 的系統核心
- session 很自然會依 sender / thread / channel 被切開
- workspace 檔案本來就已經是實際上的記憶載體
- `MEMORY.md` + `memory/YYYY-MM-DD.md` 本來就被鼓勵當作持久記憶使用

所以，OpenClaw 其實已經有：
- **thread / session 記憶**
- **檔案型長期記憶**

它目前比較缺的，是一個中間層，像是：
- Slack 跨 thread 的共享 working memory
- thread 開始時的跨 thread recall 機制
- thread 中重要結論自動被提升為可重用記憶的流程

而這個缺的中間層，剛好就是最值得設計的地方。

---

## 值得借鏡的外部設計模式

## 1. LangGraph / LangChain：thread-scoped 短期記憶 + namespace-scoped 長期記憶

LangChain / LangGraph 的 memory 文件對記憶範圍切得非常清楚：

- **短期記憶（short-term memory）**：thread-scoped，屬於單一對話 / 單一 thread
- **長期記憶（long-term memory）**：可跨 session / 跨 threads 共用，用 namespace 來分群

這是目前最貼近 OpenClaw + Slack 問題本質的一種模型。

### 為什麼重要

它驗證了一個很關鍵的原則：

- Slack thread continuity 應該留在 thread 本地處理
- 跨 thread continuity 應該來自另一層記憶，而不是把所有 thread transcript 粘成一大塊

### 可借用的重點

至少要有 **兩個記憶 scope**：

1. **Thread memory**
   - 儲存本地 transcript 與本地工作上下文
2. **Shared user/work memory**
   - 儲存可以跨所有 thread 重用的使用者／工作相關資訊

### 套到 OpenClaw 的對應做法

可以很自然對應成：

- `MEMORY.md` = 穩定的長期記憶
- `memory/active-context.md` = 當前跨 thread 的共享 working memory
- session JSONL logs = thread 本地歷史

### 參考概念

- LangChain memory overview：短期記憶是 thread-scoped，長期記憶可跨 thread / session
- LangGraph persistence：以 `thread_id` 為主鍵保存 thread 狀態
- LangGraph 定位：agent 需要跨未來互動持續存在的記憶

---

## 2. LlamaIndex：短期 queue + 長期 memory blocks 的混合式設計

LlamaIndex 的記憶設計很有意思，它不是把 memory 當成單一東西，而是當成一個可以組合的結構。裡面可能包含：

- 最近的短期聊天歷史
- 從對話中抽出的 facts
- 用向量檢索回來的舊訊息片段
- 固定不變的靜態記憶區塊

### 為什麼有價值

這提醒我們：memory 不是單數，而是**一個由不同層次與不同用途組成的記憶堆疊**。

### 最值得借的點

- 保留有限的最近 buffer 作為即時上下文
- 把較舊的內容逐步 flush 成長期記憶型態
- 按用途拆分 memory blocks，例如：
  - static identity / context
  - fact extraction
  - retrieved prior episodes

### 套到 OpenClaw 的翻譯

即使不引入額外 infra，也可以模擬：

- **static memory**：`MEMORY.md`
- **active working memory**：`memory/active-context.md`
- **episodic recall**：需要時去查 session logs 或某天的 memory 檔

這比把所有內容一直往同一個記憶檔塞，要健康得多。

---

## 3. Mem0：把 memory 視為獨立系統層，而不是 transcript 的副作用

Mem0 的整體 framing 很直接：

- 使用者不應該一直重複自己講過的事
- 記憶應該能跨 sessions / agents / users 持續存在
- 記憶的作用是減少 prompt 膨脹，同時增加 continuity

### 為什麼重要

即使 OpenClaw 不一定會直接採用 Mem0，Mem0 仍然是一個很好的產品級 benchmark。
它反映了一件事：Jones 遇到的問題不是小眾特例，而是 agent 產品設計裡非常主流的問題。

### 最值得借用的觀念

Memory 應該被當成一個**獨立的系統層**，而不是 transcript 保存完之後剛好留下的殘影。

套回 OpenClaw，意思是：
- 不要把 thread transcript 當成唯一真實記憶
- 應該主動為 Slack 工作流建立一層共享記憶

---

## 4. Anthropic：先用簡單可組合的模式，不要過早框架化

Anthropic 在 “Building effective agents” 裡一直重複一個核心原則：

- 先用最簡單可行的架構
- 用可組合 building blocks
- 不要一開始就把系統做成過度複雜的 framework

### 為什麼這對我們重要

這剛好是對「記憶系統過度設計」的一個提醒。

對 Jones 的 use case 來說，第一階段最務實的做法其實可能是：

- file-based shared memory
- 清楚的 memory scope
- thread 開頭的 recall
- 選擇性把重要結論升級成長期記憶

而不是：

- 第一天就建一個全自動 giant memory graph
- 還沒理清生命週期就先上整套 vector infra

這很重要，因為它代表：**好的第一版，其實可以非常輕量，但仍然有感。**

---

## 5. MCP 方向：未來若要接外部 memory 系統，標準化入口很重要

MCP 本身不是記憶解法，但它的重要性在於：它是 AI client 連接外部系統的一個標準化協定。

### 為什麼值得注意

如果未來 OpenClaw 想接到：

- 專門的 memory store
- notes system
- vector DB
- project-state service

那 MCP-compatible tooling 會是一條非常值得保留的後路。

### 實務意義

對 Jones 現在這個問題來說，**第一步不該直接從 MCP 開始**。但如果未來想要：

- 把共享記憶做成一個服務
- 讓多個 AI client 共用同一層記憶
- 做出可重用的 memory components

那 MCP 會變得很有戰略價值。

---

## Slack 專屬的現實檢查

Slack threads 對人類來說很好用，因為它讓討論局部化、主題化。但對 agent 來說，它天然造成一種碎片化問題：

- 人類覺得：「我還是在跟同一個 Jarvis 講話」
- runtime 卻看到：「這是另一條 thread / 另一個 session」

所以，問題其實不在 Slack 本身，而是在這兩個模型之間的不對齊：

- **人的心智模型**：同一段持續關係
- **系統的執行模型**：很多個 thread-scoped sessions

這也就是為什麼只有 thread continuity 還不夠。

---

## 適用於 OpenClaw + Slack 的候選設計

## Option A：共享 file-based working memory layer（最推薦的第一步）

### 核心想法

加一份跨 thread 的共享記憶檔，例如：

- `memory/active-context.md`

這份檔案不是完整日記，而是一份經過整理的共享工作狀態。

### 建議內容區塊

- Current focus
- Ongoing workstreams
- Recent important decisions
- Open loops / pending follow-up
- What Jones probably expects Jarvis to remember across threads
- Thread map（可選）

### 怎麼運作

當一個新的 Slack thread 開始時：

1. 先照舊讀 startup files
2. 再讀 `memory/active-context.md`
3. 視需要讀今天的 daily memory
4. 帶著這層共享上下文進入新 thread

### 優點

- 是最簡單但已經有感的升級
- 完全符合 OpenClaw 目前的 file-memory 文化
- 非常容易檢查、debug、人工編修
- 實作風險低
- 不需要額外 infra

### 缺點

- 需要 agent 持續維護與整理
- 如果維護不好，可能會 drift
- 不是 semantic retrieval 型，不會自動理解全部脈絡

### 評價

這是目前最值得先做的方案。

---

## Option B：共享 working memory + promotion rules

### 核心想法

和 Option A 類似，但再加上清楚的資訊升級規則：

- thread-local 細節留在 thread / session
- 當日重要內容寫進 `memory/YYYY-MM-DD.md`
- 跨 thread 持續有效的工作狀態寫進 `memory/active-context.md`
- 穩定的偏好／身份／教訓寫進 `MEMORY.md`

### 為什麼強

這等於不是只加一個新檔，而是建立一套真正的記憶生命週期。

### 優點

- 記憶分層清楚
- 減少長期記憶污染
- 對「same Jarvis feel」很有幫助

### 缺點

- 需要一定紀律
- 仍然主要是 file-based，檢索能力取決於結構與維護品質

### 評價

這應該和 Option A 綁在一起，作為第一版的完整解法。

---

## Option C：帶檢索的共享記憶索引

### 核心想法

建立一個可搜尋的跨 thread 記憶層，對這些內容做檢索：

- `MEMORY.md`
- `memory/*.md`
- 被選中的 session summaries
- 也許還有 tagged thread summaries

在新 thread 開始時，只召回最相關的一小部分。

### 優點

- 會比單一 curated file 更可擴展
- 對較早期但相關的內容召回能力更強
- 長期可持續性更好

### 缺點

- 實作複雜度明顯上升
- retrieval 品質本身又變成要調校的新系統
- 撈太多或撈錯都會傷 UX

### 評價

這很適合第二階段，但不應該當第一步。

---

## Option D：完整外部 memory 平台（Mem0 / vector memory / graph memory）

### 核心想法

把共享記憶做成 filesystem 以外的正式系統層。

### 優點

- 理論上跨 session personalization 最強
- metadata / filtering / namespace 都更靈活
- 長期延展性高

### 缺點

- infra 與 integration 成本高
- 多一個 operational dependency
- 對目前 OpenClaw + Slack 的問題來說可能太重
- debug 難度比 file-based 高很多

### 評價

很值得當 benchmark，但不適合做 Jones 現在的第一步。

---

## 哪些工具或思路雖然不是專屬 Slack，但最能遷移到 Slack？

### 高度可重用
- LangGraph 的 **thread memory + shared long-term memory** 分層
- LlamaIndex 的 **recent buffer + fact extraction + episodic retrieval** 結構
- Mem0 把 memory 當成一層**專門系統**而不是 transcript 附屬品的思路

### 未來可能可用
- MCP-based 的 memory services / connectors
- vector memory stores
- 結構化 project-state stores

### 不建議第一步就做
- 一整套高度黑盒的 agent memory framework
- 在 file-memory lifecycle 都還沒理清前就先做 graph memory

---

## 2. Jones 真正可以選的 2~3 個實際方案

以下是壓縮後的決策版。

## 選項 1：輕量 file-memory 升級

### 做法

新增：

- `memory/active-context.md`

並讓 Jarvis 在跨 Slack thread 工作時，把共享 working memory 維護在這裡。

### 你會得到什麼

- 明顯更好的跨 thread continuity
- 很低的工程風險
- 完全透明，可人工檢查與修正

### 不會得到什麼

- 自動 semantic retrieval
- 很聰明的舊案召回系統

### 適合什麼

- 想要立刻改善 UX
- 想用最低風險先做出成果
- 想保持可 debug / 可控

---

## 選項 2：file-memory 升級 + 結構化 promotion policy

### 做法

和選項 1 一起做，但明確定義：

- thread 細節 -> thread/session
- 當日事件 -> daily log
- 跨 thread active work -> `active-context.md`
- 穩定長期資訊 -> `MEMORY.md`

### 你會得到什麼

- 更完整的記憶架構
- 較少混亂和重複
- 更強的跨 thread 延續感

### 適合什麼

- Jones 現在的真實工作流
- 立即可實作
- 想建立一個 minimum viable 記憶系統

### 建議

這是我最推薦的近程方案。

---

## 選項 3：之後再加 retrieval

### 做法

在選項 2 上再加：

- thread summaries
- 對 memory files / summaries 做檢索召回

### 你會得到什麼

- 較舊但相關的內容更容易接回來
- 對多專案、多 thread 的情況更穩

### 代價

- 工程複雜度更高
- 調 retrieval 會需要時間
- 更容易出現召回不準的副作用

### 建議

這應該是第二階段，不是第一階段。

---

## 3. 如果真的要在 OpenClaw 裡實作，應該怎麼設計？

## 建議架構：3-layer memory model

### Layer 1：thread-local session memory

**目的：** 保留當前 Slack thread 內的局部延續感

**底層依賴：**
- OpenClaw 現有的 thread / session transcript 機制
- session JSONL logs

**原則：**
- 不要試圖把所有 threads 合成一個超大 session

---

### Layer 2：shared working memory

**目的：** 承接跨 Slack threads 的當前工作上下文

**底層依賴：**
- `memory/active-context.md`

**建議結構：**

```md
# Active Context

## Current focus
## Ongoing workstreams
## Recent decisions
## Pending follow-ups
## Important context to carry across threads
## Optional thread map
```

**原則：**
- 這是預設的跨 thread handoff layer
- 應該保持精煉且更新

---

### Layer 3：durable long-term memory

**目的：** 保存穩定的偏好、身份、教訓、長期專案事實

**底層依賴：**
- `MEMORY.md`

**原則：**
- 只放 durable 的東西
- 不要把短期 thread 細節亂塞進去

---

## Supporting layer：daily event log

**目的：** 保留按時間排序的事件細節，但不污染共享記憶層

**底層依賴：**
- `memory/YYYY-MM-DD.md`

**原則：**
- 原始日誌放這裡
- 它是 recall source，不是主要 handoff object

---

## 建議的記憶生命週期

當一個 thread 裡出現重要結論時，可以依序問：

1. 這件事只在這個 thread 內有用嗎？
   - 留在 thread / session 就好
2. 這件事今天稍後或其他 thread 很可能還會用到嗎？
   - 寫進 `active-context.md`
3. 這件事是長期偏好、教訓、身份資訊、或穩定專案事實嗎？
   - 提升到 `MEMORY.md`
4. 這件事主要只是時間脈絡／留痕嗎？
   - 寫進 daily note

這就會讓 OpenClaw 擁有一套真正的 memory policy，而不是記憶混沌。

---

## 建議的 Slack main-session recall 順序

如果未來要優化 main-session Slack contexts，理想的讀取順序應該是：

1. `SOUL.md`
2. `USER.md`
3. 今天的 daily memory
4. 昨天的 daily memory
5. `MEMORY.md`
6. `memory/active-context.md`

如果有必要，再進一步查：
- session logs
- project-specific notes
- 相關 research/report 檔案

### 為什麼這順序合理

- 先載入穩定的 persona / user context
- 再載入最近的時間脈絡
- 然後是 durable memory
- 最後補上當前跨 thread 的 working state

這很可能已經足以大幅減少「我們剛剛做到哪？」的情況。

---

## 下一步可以考慮的增強：thread summaries

如果之後還想更強，可以增加一層輕量的 per-thread summary：

- 每個真的有結論的 Slack thread 只留一份短摘要
- 只有當 thread 有重要決策／產出時才生成

這可以放在：
- `memory/thread-summaries/`
- 或者作為 daily notes 的一部分

這樣就能在不看 raw session logs 的情況下，更好地做 episodic recall。

---

## 再下一步的增強：thread start retrieval

當 file-memory 結構已經乾淨後，可以再加一層 retrieval：

- 對 `MEMORY.md`、`active-context.md`、daily notes、thread summaries 做搜尋
- 把最相關的少量結果注入新 thread 的 startup context

這時候 semantic search / memory indexing 才真正變得有意義。

但前提是底層記憶 hygiene 已經先做好。

---

## 我認為 Jones 真正想要的是什麼

不是無限記憶。

Jones 真正想要的比較像是：

- 如果一件事對跨 thread 很重要，Jarvis 應該能帶過去
- 如果剛做了決策，新的 thread 不應該像失憶一樣
- 如果工作正在持續，Jarvis 應該表現得像一個持續中的 collaborator

所以真正的目標是：**continuity UX**，而不是抽象地追求 memory maximalism。

也因此，最合理的設計是：

### 理想架構

1. **Slack thread = 局部工作上下文**
2. **active shared memory = 跨 thread 的當前共享上下文**
3. **MEMORY.md = 穩定的長期記憶**
4. **daily notes = 時間序列事件記錄**
5. **必要時對 notes / logs 做 retrieval = 深層召回**

這個設計簡單、可讀、可維護，而且非常符合 OpenClaw 現有的精神。

---

## 建議的具體落地路線

## Phase 1：立刻可做

建立並維護：

- `memory/active-context.md`

用來放：
- current projects
- current important threads
- recent decisions that should survive thread switching
- pending follow-ups

然後把 Jarvis 在 main-session Slack context 的 recall 行為改成會讀：

1. `SOUL.md`
2. `USER.md`
3. 今日 + 昨日 daily memory
4. `MEMORY.md`
5. `memory/active-context.md`

這一步本身就很可能能大幅改善跨 thread continuity。

---

## Phase 2：加上資訊 promotion 規則

建立明確規則：

- thread-local detail -> 留在 thread / session
- same-day log -> daily note
- cross-thread active work -> `active-context.md`
- durable learning -> `MEMORY.md`

這是 minimum viable 記憶架構。

---

## Phase 3：再考慮加 retrieval

如果到時還覺得不夠，再做：

- active workstreams tagging
- memory files + thread summaries 的搜尋 / 召回
- project-indexed 或 workstream-indexed recall layer

只有在 Phase 1 / 2 仍不足時，才建議走到這一步。

---

## 後續還可以繼續研究的關鍵字

有用的搜尋關鍵字：

- `slack ai agent cross thread memory`
- `thread scoped memory long term memory agent`
- `langgraph short term memory long term memory threads`
- `mem0 cross session memory agents`
- `llamaindex memory blocks short term long term`
- `slack bot thread context memory`
- `agent shared working memory vs long term memory`
- `conversation memory namespaces agents`
- `model context protocol memory tools`

可以持續觀察的工具 / 框架類別：

- LangGraph / LangChain memory
- Mem0
- LlamaIndex memory
- MCP-compatible memory tools
- vector-store-backed agent memory
- shared workspace file-memory patterns

---

## 最後建議

對 OpenClaw + Slack 而言，最好的下一步**不是**先去找一個神奇的 Slack 專用插件。

最值得做的近程方案是：

1. 保留 Slack thread 作為局部 session 容器
2. 加一層共享的 cross-thread working memory
3. 建立 thread logs / daily notes / active context / long-term memory 之間的 promotion rules
4. 之後視情況再加 retrieval

所以，最直接的結論就是：

> 最值得先做的，是在 OpenClaw 現有 file-memory 架構裡，建立一個輕量的 shared working-memory layer，而不是硬把所有 Slack threads 合成一個 session。

這樣可以在以下幾點之間取得最好的平衡：
- continuity
- 透明性
- 實作成本低
- operational risk 低
- 之後仍可擴充

---

## OpenClaw / Jarvis 可直接實作的 memory policy 草案

這一段不是再抽象討論，而是把前面的研究收斂成一套第一版可以直接落地的 policy。

## 核心原則

1. **Slack thread 內的 continuity 用 session 承接**
2. **Slack thread 之間的 continuity 用 shared working memory 承接**
3. **長期穩定資訊不要和當前工作狀態混在一起**
4. **shared working memory 必須可覆寫，不可變成 append-only 墓園**
5. **先不要上 retrieval / vector / external memory service**

---

## `memory/active-context.md` 的定位

它不是第二份 `MEMORY.md`，也不是第二份日記。

它應該是：

> 一份跨 thread / 跨 session 共用的 working whiteboard。

它的目標只有一個：

> 當 Jones 換一個 Slack thread 時，Jarvis 不用從零開始想「現在到底接到哪裡」。

### 設計要求

- 要短
- 要可掃描
- 要偏 operational
- 要能覆寫舊狀態
- 不要變成流水帳

---

## 建議範本

```md
# Active Context

> Shared working memory across threads/sessions.
> This file is a whiteboard, not a journal.
> Keep it short, current, and rewriteable.

## Current focus
- 正在處理的主線任務，最多 3 條

## Ongoing workstreams
- 每條一行：名稱 / 狀態 / 下一步

## Recent important decisions
- 最近已拍板、未來 thread 不該失憶的決策

## Pending follow-ups
- 還沒完成、很可能會在別的 thread 接著做的 open loops

## Cross-thread context Jones expects Jarvis to carry
- Jones 若換 thread，Jarvis 仍應該記得的當前重點

## Optional thread map
- 哪些 thread 正在延續什麼主題（可選）
```

---

## Promotion rules（資訊要流去哪）

### 1. thread details -> session / thread logs

適合放在 thread 本地的內容：
- 一般對答
- 當下 debug 過程
- 暫時性來回討論
- 不值得跨 thread 帶走的細節

### 2. same-day timeline -> `memory/YYYY-MM-DD.md`

適合放在 daily note 的內容：
- 今天做了什麼
- 時間序列事件
- 有留痕價值但不一定要跨 thread 帶走的事情

### 3. cross-thread active state -> `memory/active-context.md`

只有符合以下條件，才應該寫進 active context：

- 這件事明天或下一個 thread 很可能還要接著做
- 不寫會導致 Jones 需要重新解釋背景
- 這是一個已形成的決策，而不是還在發散中的雜訊
- 這是工作狀態 / handoff 線索，而不是一般聊天內容

### 4. durable facts / identity / lessons -> `MEMORY.md`

適合升級進長期記憶的內容：
- Jones 的穩定偏好
- Jarvis 應長期記住的工作方式
- 長期有效的專案事實
- 值得保留的教訓與規則

---

## 不要寫進 `active-context.md` 的東西

- 暫時 brainstorm
- thread 裡一般來回對話
- 很快會過期的細節
- 可直接從 daily log 查回的流水帳
- 使用者一時的情緒輸出，但沒有形成持續工作脈絡的內容

---

## `active-context.md` 的維護規則

它應該像白板，不是墓園。

### 可以做的事
- 刪掉舊 focus
- 更新 workstream 狀態
- 覆寫過時決策
- 移除已完成的 pending follow-up

### 不該做的事
- 只會 append
- 為了怕漏而把一切都往裡面塞
- 把它當長期記憶或完整歷史記錄

---

## 新 Slack thread 啟動時的 recall 流程（第一版，已調整）

建議流程：

1. `SOUL.md`
2. `USER.md`
3. `memory/active-context.md`
4. `memory/YYYY-MM-DD.md`（today + yesterday）
5. `MEMORY.md`

### 為什麼這樣調整

這次根據進一步討論，第一版 recall 順序改成讓 `active-context.md` 提前。

理由是：
- 新 thread 最需要的是快速接上「現在正在做什麼」
- `active-context.md` 應該是短而硬的工作白板，不應該重到干擾整體 context
- 近期 daily note 和 `MEMORY.md` 仍然保留，用來補時間脈絡與長期穩定資訊

這樣的順序更偏向：
- 先恢復 persona / user
- 再快速接上跨 thread working state
- 再補近期事件
- 最後補長期穩定記憶

---

## Thread 結束 / 切換前的微型維護檢查

每次一個 thread 有實際進展後，Jarvis 應快速問自己：

1. 這個 thread 有沒有需要跨 thread 帶走的東西？
2. 如果有，這個資訊應該去哪？
   - session only
   - daily note
   - active context
   - MEMORY.md

這個檢查不需要很重，但應該變成習慣。

---

## Thread summary 的產生條件（之後再做，不是現在全部做）

不要每個 thread 都 summary。

只有符合以下條件才考慮做 thread summary：

- 有**決策**
- 有**產出**
- 有**待辦接手**
- 有**錯誤排查結論**
- 有**值得未來引用的 workaround**

若之後實作，建議加標籤，例如：
- `decision`
- `debug`
- `workflow`
- `design`
- `blocked`
- `resolved`

這會讓未來若要做 retrieval，精準度高很多。

---

## 第一版實作建議

### 立刻做
- 建立 `memory/active-context.md`
- 更新 agent 工作規則，讓它在 direct / main-session 工作流中讀這份檔
- 開始使用 promotion rules

### 暫時不要做
- vector retrieval
- external memory service
- 全量 thread summary 自動化
- 把所有 Slack threads 合併成一個 giant session

---

## 第一版實作狀態（2026-04-18）

已開始落地：

- 已建立 `memory/active-context.md`
- 已更新工作區規則，讓未來 session startup 會讀取 active context
- 已把 shared working memory 明確定位為 cross-thread / cross-session whiteboard

這代表這份研究已經不只是理論，而是進入第一階段實作。

---

## 參考來源

- OpenClaw docs：多通道 Gateway、session-centric 架構、file-memory 慣例
- LangChain / LangGraph memory overview：thread-scoped short-term vs shared long-term memory
- LangGraph persistence docs：thread-keyed checkpoints 與 thread state
- LangGraph memory docs：短期 thread 記憶 + 長期共享記憶
- LlamaIndex memory docs：混合式 short-term + long-term memory blocks
- Mem0 overview：把 memory 視為跨 session continuity 的系統層
- Anthropic “Building effective agents”：建議先用簡單可組合模式，不要過早框架化
- MCP docs：標準化連接外部工具與資料系統，對未來 memory-service integration 有參考價值

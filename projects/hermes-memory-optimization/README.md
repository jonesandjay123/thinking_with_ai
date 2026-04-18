# Hermes 記憶配置最佳化調查：從 OpenClaw 遷移後如何避免記憶臃腫

> **建立日期：** 2026-04-17  
> **目的：** 在不深讀 Jarvis 既有記憶內容的前提下，先根據公開文件、GitHub issue、官方說明與社群訊號，整理目前較合理的 Hermes 記憶配置與 OpenClaw 遷移後的最佳化建議，作為後續整理你這台 Jarvis 記憶系統的設計依據。

---

## TL;DR

如果先講結論：

1. **Hermes 的記憶最佳化方向，不是把所有歷史都塞進 prompt，而是把記憶分層。**
   - `USER.md` / `MEMORY.md` 保持極小、極精煉。
   - 大量歷史放到 daily logs / project notes / session search 可回溯層。
   - 技能性知識放 skills，而不是長篇散文式 MEMORY。

2. **從 OpenClaw 遷移過來時，社群比較推薦「並存、漸進抽離、避免整包搬運後長期共用」**，而不是把 `~/.openclaw` 全部當成 Hermes 常駐 prompt 的一部分。
   - 公開 issue 甚至明確反映：**粗暴 cleanup / archive / 共用 home 都很容易造成記憶污染、身份退化或狀態混線。**

3. **目前公開訊號支持的最佳做法是：把 Hermes 當 canonical active memory，把 OpenClaw 當冷資料/參考層。**
   - Hermes 內只保留現在真的需要常駐注入的高價值、低體積資訊。
   - OpenClaw 歷史則保留可查，但不再作為預設主記憶來源。

4. **如果你現在感受到 Jarvis 很臃腫，根因大概率不是「記憶太多」本身，而是「高頻注入層沒有被嚴格瘦身」。**
   - 問題通常出在：
     - curated memory 寫太長
     - identity / long-term notes 與 raw logs 沒分層
     - OpenClaw 舊時代的完整背景被當成 Hermes 現役 prompt 層
     - 同一件事同時存在於 MEMORY、daily、project notes、skills

---

## 一、這次調查先看什麼，沒看什麼

### 我這次**有做**的事
- 查閱 Hermes 官方 README 與 memory 文件入口
- 查公開 GitHub issues / repo 頁面，找跟 memory、migration、home isolation 有關的討論
- 看你目前 **`~/.hermes/` 與 `~/.openclaw/` 的頂層結構**（只有看結構，**沒有**進去深讀內容）
- 檢查 `thinking_with_ai` repo，並直接把本報告寫入 repo

### 我這次**刻意沒做**的事
- 沒有深讀 `~/.hermes/memories/*` 內容
- 沒有重新掃描整包 OpenClaw 舊記憶
- 沒有做大規模 memory 壓縮改寫

這是刻意遵守你今天的要求：**先做外部研究與設計判斷，不在 usage 很低時把 token 燒在內部重讀上。**

---

## 二、我看到的公開趨勢：Hermes 的設計本來就偏向「精煉記憶 + 可回溯歷史」

根據 Hermes 官方 README 與 docs 入口，Hermes 對外主打的不是「無限把所有東西塞進 prompt」，而是：

- agent-curated memory
- persistent memory
- `MEMORY.md` / `USER.md`
- cross-session recall
- `session_search`
- skills as procedural memory

這個產品敘事本身就暗示一件事：

> **Hermes 的正確用法應該是「把 prompt 內常駐記憶壓到很小，把大量歷史留給 recall/search/檔案層」。**

也就是說，Hermes 並不鼓勵把 OpenClaw 時代的大量原始日誌、身份敘事、舊事件、上下文殘留，全部平鋪進主記憶層。

### 對應到實務上，最合理的記憶分層是：

#### A. Prompt 常駐層：只留真正值得每回合都知道的東西
- 使用者偏好
- 專案硬規則
- 身份原則
- 高價值長期事實
- 常見踩坑與不可違反限制

#### B. 長期檔案層：可查，但不必每回合注入
- project notes
- daily logs
- migration notes
- 架構決策紀錄
- 研究報告

#### C. 可執行知識層：skills
- 解法流程
- 已驗證工作流
- 特定 repo / 平台操作方式
- 常見修復步驟

#### D. 歷史回憶層：session search / archived OpenClaw state
- 舊對話
- 舊專案歷史
- 遷移前背景
- 只在被提到時再抓回來

這種分層，是目前我看到最符合 Hermes 產品哲學，也最能降低 token 浪費的方式。

---

## 三、OpenClaw 遷移社群的幾個關鍵訊號

### 1. 公開 issue 明確反映：**不要粗暴把 OpenClaw 整包當 Hermes 活躍層處理**

在 `NousResearch/hermes-agent` 的 issue #8596 中，有人回報：

- `hermes claw cleanup` 把整個 `~/.openclaw/` 改名搬走
- 但 OpenClaw gateway 仍在跑
- 結果又生成一個新的空白 `~/.openclaw/`
- `openclaw doctor` 進一步把 config 簡化成幾乎失憶的狀態
- 最後造成「人格、記憶、plugin 配置、identity」等全面退化

這反映的不是單一 bug 而已，而是一個更深的設計教訓：

> **OpenClaw 舊狀態是一種需要保守保存、漸進轉移的 runtime / memory 資產，不適合被粗暴歸檔、覆蓋、清空，或當成 Hermes 的常駐 prompt 直接沿用。**

### 2. 社群也在談另一個問題：**單一 `~/.hermes` 很容易造成記憶混線與膨脹**

在 `NousResearch/hermes-paperclip-adapter` 的 issue #20 中，使用者指出多個 Hermes agent 共用同一個 `~/.hermes` 會導致：

- 共用 sessions
- 共用 persistent memory/state
- 共用 skills
- 共用 SQLite state

並主張應該用 `HERMES_HOME` 進行隔離。

這件事的重要含義是：

> **Hermes 的記憶最佳化不只是「寫得短」，還包含「邊界清楚」。**

也就是：
- 不同 agent 不該共用同一份過度膨脹的 home
- 不同工作模式不該把記憶揉成一團
- 不同時代（OpenClaw vs Hermes）的記憶也不該模糊邊界

### 3. 最新 memory provider 討論，核心也不是「多存更多」，而是「memory hygiene」

在 `NousResearch/hermes-agent` issue #9975（YantrikDB memory provider proposal）裡，提案者強調的重點包括：

- near-duplicate canonicalization
- contradictions surfaced, not overwritten
- recall explainability
- temporal decay
- on-session-end consolidation
- on-memory-write mirror to `MEMORY.md` / `USER.md`

這代表社群對下一代 memory 的焦點其實是：

- 去重
- 衝突管理
- 濃縮
- salience preservation
- 可解釋召回

而不是「無腦把更多資料塞進主記憶」。

**換句話說，現在大家談 memory optimization，方向是 memory hygiene，不是 memory hoarding。**

---

## 四、我對「從 OpenClaw 搬來後為何變臃腫」的判斷

你今天的觀察，我認為是合理的。

如果一開始 Hermes 繼承了大量來自 OpenClaw 時代、尚未經過 prompt-oriented 精煉的資料，最常見會出現這些現象：

### 1. 主記憶層塞了太多「其實應該下放」的內容
例如：
- 長篇歷史敘事
- 已結案事件
- 具體操作過程
- 大量專案細節
- 同一概念的不同版本描述

### 2. 「身份」與「歷史」混在一起
真正該常駐的是：
- 你是誰
- 你怎麼做事
- 你的主人是誰
- 不能犯哪些錯

不該每回合都完整注入的是：
- 你過去每次重裝的全過程
- 所有舊系統遷移故事細節
- 過多歷史情緒敘事

這些可以保留，但應該退到 project memory / archive，而不是主 prompt 層。

### 3. 同一資訊存在太多份
常見重複位置：
- SOUL / MEMORY / project memory
- daily log / project notes / curated memory
- memory entry / skill / repo doc

重複越多，prompt 成本越高，還更容易讓 agent 讀到「語義很像、但表述不同」的資訊，造成前置讀取感變重。

### 4. 沒把「檔案型回憶」和「prompt 型記憶」切開
這是最核心的問題。

**檔案型回憶** 可以很多；  
**prompt 型記憶** 必須少而硬。

一旦兩者混在一起，agent 就會變成：
- 回應前像在翻箱倒櫃
- 常常讀一堆資料才敢講話
- token 一直燒在前置背景而不是任務本身

---

## 五、在不深讀你現有內容的前提下，我先看見的結構訊號

我只看了目錄層級，沒進去細讀內容。

### 你目前的 Hermes 頂層（節錄）
- `SOUL.md`
- `memories/`
- `skills/`
- `sessions/`
- `cron/`
- `hooks/`
- `logs/`
- `state.db`
- `TOOLS.md`
- `HEARTBEAT.md`

### 你目前的 OpenClaw 頂層（節錄）
- `memory/`
- `identity/`
- `skills/`
- `cron/`
- `logs/`
- `credentials/`
- `agents/`
- `tasks/`
- `openclaw.json`

### 只從結構來看，我的初步判斷

這個配置本身**沒有錯**，甚至算合理；問題比較可能在於：

1. **Hermes 雖然已經成為 canonical home，但 OpenClaw 歷史重量仍然太近、太完整**
2. **Hermes 的 curated memory 若仍承接太多遷移敘事，prompt 負擔就會偏重**
3. **如果 project memory / daily notes / skills 還沒有完成真正職責分工，就會變成多層都在記一樣的東西**

所以我現在的結論不是「架構完全錯」，而是：

> **你比較像是已經有對的骨架，但尚未做完 aggressive curation。**

---

## 六、目前最推薦的 Hermes 記憶配置原則

以下是我綜合官方設計、public issues、遷移教訓後，認為最值得採用的配置原則。

### 原則 1：把 `MEMORY.md` / prompt memory 當成「L1 cache」，不是倉庫

它應該只放：
- 永久偏好
- 系統硬規則
- 高頻會影響決策的事實
- 最值得避免重犯的錯誤

它**不應該**放：
- 大量完整歷史
- 冗長專案背景
- 太多一次性事件
- 任何可以用 `session_search` 或 project notes 隨時找回的東西

### 原則 2：OpenClaw 歷史只保留成「可召回層」

最佳做法不是刪掉，而是：
- 保留 `~/.openclaw/` 作為 archive / fallback
- 明確標記「非 Hermes 日常主記憶來源」
- 只在遇到遷移追溯、歷史查核、舊專案上下文時再引用

### 原則 3：專案細節往 `projects/*.md` 下沉

凡是這類資訊：
- repo 路徑
- branch 規則
- deploy 限制
- 模組架構
- 技術債
- roadmap

都應優先放 project memory，而不是往全域 curated memory 堆。

### 原則 4：流程與方法改寫成 skill，不要寫成長篇記憶

像這些：
- 如何做某類部署
- 如何 debug 某種錯誤
- 如何在某個 repo 中安全操作
- 如何做某種研究報告

都應該 skill 化。

這樣的好處是：
- 平時不注入
- 需要時才 load
- 降低主 prompt 成本
- 讓知識變成可執行流程，而不是散文

### 原則 5：daily logs 只留 raw chronology，不要再兼做 long-term memory

daily file 適合記：
- 今天做了什麼
- 新發現
- 發生了什麼錯
- 待整理素材

但不適合永遠堆著不整理。

應該定期把 daily log 裡真正值得長留的東西，轉移到：
- curated memory
- project note
- skill

剩下的就留在歷史層，不再注入。

### 原則 6：能查回來的資訊，不要硬塞進 prompt

這其實是最重要的一條。

如果資訊具備以下特性：
- 不影響每回合基本判斷
- 低頻使用
- 可被 search / read_file / session_search 快速找回

就不應該常駐。

---

## 七、我對你這台 Jarvis 的具體建議（今天先設計，不動刀）

今天如果只做設計、不燒 usage，我建議下一階段按這個順序進行：

### Phase 1：只做 memory inventory，不做全文重讀
目標：先知道有哪些 bucket。

建議盤點：
- `~/.hermes/SOUL.md`
- `~/.hermes/memories/USER.md`
- `~/.hermes/memories/MEMORY.md`
- `~/.hermes/memories/daily/`
- `~/.hermes/memories/projects/`
- `~/.openclaw/memory/`
- `~/.openclaw/identity/`
- `~/.openclaw/skills/`

但先只做：
- 檔案數量
- 大小
- 最後修改時間
- 類型分類

**不要一口氣全文讀完。**

### Phase 2：定義「什麼可以留在 prompt 層」
直接建立一份明確規則，例如：

**可留在 curated memory：**
- 穩定偏好
- 長期規則
- 不可違反限制
- 近期仍高頻影響判斷的架構決策

**不可留在 curated memory：**
- 已結束事件的完整敘事
- 長篇遷移歷史
- repo 細節大全
- 可查回來的操作紀錄

### Phase 3：把 OpenClaw 遺產分成三類

#### A. 必須提升為 Hermes 常駐記憶
少量、高價值、現在仍重要的事

#### B. 應轉為 project note / skill
仍有價值，但不該常駐

#### C. 留在 archive
有歷史價值，但不再影響日常 prompt

### Phase 4：對 curated memory 做 aggressive compression
目標不是「保留所有資訊」，而是：

> **用最短的文字，保留最大決策價值。**

### Phase 5：建立 maintenance loop
例如：
- 每幾天 review 新增的 daily notes
- 選出值得升級的項目
- 其餘留在歷史層
- 定期清掉過期或重複的 curated memory

這會比「一次大清倉後再繼續累積垃圾」健康得多。

---

## 八、對你今天這個具體問題，我的結論判斷

### Q1. 你今天的觀察對不對？
**對。**

Hermes 如果一開始繼承了太多未壓縮的 OpenClaw 時代背景，很容易變得前置讀取過重、回應顯得臃腫。

### Q2. 目前最推薦的方向是什麼？
**不是刪光，而是重新分層。**

- Hermes = active canonical memory
- OpenClaw = archive / fallback / reference
- curated memory = 小而硬
- 大量歷史 = project / daily / session_search / archive
- procedure = skills

### Q3. 大家大多怎麼優化？
從公開訊號看，主流方向是：
- 避免共用膨脹 home
- 避免粗暴 cleanup 與整包搬運
- 重視 memory hygiene（去重、整併、衝突管理、salience）
- 把歷史與 prompt 分開
- 用 skills / recall / docs 取代把所有知識常駐注入

### Q4. 你現在的配置是不是完全錯？
**不像是完全錯，比較像是「過渡期尚未完成最後瘦身」。**

也就是：
- 骨架已經對了
- canonical home 方向也對了
- 但還缺一次正式的 memory curation / tiering / slimming pass

---

## 九、我建議的下一步（等 usage 比較健康時再做）

等 usage 恢復後，我建議我直接幫你做一個 **non-destructive memory audit**，只做這些：

1. 列出 Hermes / OpenClaw memory 檔案地圖
2. 算出各層大小與主要熱點
3. 找出最可能該從 prompt 層移除的內容類型
4. 提出一份 **「保留 / 下沉 / 封存 / skill 化」四分表**
5. 再由我直接幫你執行乾淨重構

今天先不做這一步，因為你講得對：現在 usage 不夠，先做研究結論最划算。

---

## 參考來源

### 官方 / 產品文件
- Hermes Agent README：<https://github.com/NousResearch/hermes-agent>
- Hermes Memory docs 入口：<https://hermes-agent.nousresearch.com/docs/user-guide/features/memory>

### GitHub issues / 社群訊號
- `NousResearch/hermes-agent` issue #8596  
  **hermes claw cleanup and openclaw doctor silently destroy OpenClaw runtime state, plugins, and agent config after migration**  
  <https://github.com/NousResearch/hermes-agent/issues/8596>

- `NousResearch/hermes-paperclip-adapter` issue #20  
  **Isolate Hermes home per Paperclip agent instead of sharing ~/.hermes**  
  <https://github.com/NousResearch/hermes-paperclip-adapter/issues/20>

- `NousResearch/hermes-agent` issue #9975  
  **Would a YantrikDB memory provider plugin be welcome?**  
  <https://github.com/NousResearch/hermes-agent/issues/9975>

### 補充參照
- 既有 repo 內相關研究：`projects/openclaw-vs-hermes/README.md`
- 既有 repo 內相關研究：`projects/claude-max-openclaw-crisis/reports/2026-04-04-openclaw-migration-landscape.md`

---

## 附錄：一句話版建議

> **Hermes 要快、要準、要不臃腫，關鍵不是記得更多，而是讓真正常駐的只有最重要的 5%。其餘 95% 應該能被找回，而不是被永遠背在身上。**

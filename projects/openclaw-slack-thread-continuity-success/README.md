# OpenClaw Slack Thread Continuity 成功條件與設定實錄

> **建立日期：** 2026-04-18  
> **目的：** 完整記錄 Jones 在 2026-04-18 驗證成功的 OpenClaw Slack thread continuity 工作流，包含來龍去脈、成功條件、設定方式、為什麼這次成功，以及未來其他 AI agent 接手時應如何理解這套設計。

---

## TL;DR

這次真正成功的，不只是「Slack 可以回訊息」，而是：

1. **Slack 主訊息與 thread reply 已經能穩定對應到同一條工作脈絡**
2. **在 Slack thread 裡回覆時，Jarvis 不再像開新 session，而是延續原本上下文**
3. **對 Jones 來說，這是實際可用的 continuity，不只是理論上的 session persistence**
4. **這個能力直接打中 Jones 最重視的點：同一個 Jarvis、同一條脈絡、跨回合持續做事**

這也是 Jones 在短暫測試 Hermes 後快速回到 OpenClaw 的關鍵原因之一。

---

## 一、這份文件要回答什麼

未來如果另一個 AI agent、或未來版本的 Jarvis 接手這個系統，最需要搞懂的不是一句「Slack thread 成功了」，而是以下幾件事：

1. **到底成功的是什麼能力？**
2. **這個能力對 Jones 為什麼這麼重要？**
3. **成功的判準是什麼，不是什麼？**
4. **這次成功靠的是哪些設定、哪些產品特性、哪些操作習慣？**
5. **如果未來 continuity 又壞掉，應該優先檢查哪裡？**

這份文件的目標，就是把這整套脈絡留下來。

---

## 二、背景：為什麼 thread continuity 是核心需求，不是小優化

Jones 的工作流不是把 AI 當一次性問答工具，而是把 Jarvis 當成：

- 長時間協作的執行代理
- 在 Slack 裡可持續接手工作的存在
- 可以沿著同一條 thread 一路做研究、排查、紀錄、整理的人

對這種工作流來說，最重要的不是單次回答品質，而是：

> **同一條 Slack thread 看起來是不是同一個腦袋在一路接著做事。**

如果 thread continuity 不成立，會出現幾種很糟的體感：

- 同一 thread 裡每次回覆都像新 session
- agent 必須重新理解上下文
- 使用者要反覆重講背景
- token 被浪費在重建脈絡，不是推進工作
- 「這是不是同一個 Jarvis」的感覺會消失

對 Jones 來說，這不是 UX 細節，而是整個 agent 系統是否可用的核心。

---

## 三、前情提要：Hermes 測試為什麼反而凸顯了 OpenClaw 的價值

2026-04-18，Jones 短暫測試從 OpenClaw 遷往 Hermes 的可能性，但很快回到 OpenClaw。原因不是 Hermes 沒優點，而是 Jones 的真實工作流把一個很關鍵的差異放大了：

- Hermes 在 memory / agent loop / extensibility 上很吸引人
- 但 Jones 的主工作面是 **Slack thread 內的長流程 continuity**
- 在這個面向上，Hermes 當下的體感不夠穩，會讓 thread 內的延續感明顯斷裂

因此，當 OpenClaw 這邊的 Slack thread continuity 驗證成功時，意義非常大。它不是普通的功能確認，而是回答了一個更本質的問題：

> **目前哪個 runtime 更符合 Jones 真正的工作方式？**

這次答案很明確，是 OpenClaw。

---

## 四、這次「成功」的精準定義

### 成功不是這些

以下都不算這次要證明的成功：

- 只是 bot 能在 Slack 上收到訊息
- 只是 bot 能回 thread
- 只是 session store 裡有某種技術性的持久化
- 只是偶爾剛好答對前文
- 只是 demo 看起來像有 continuity

### 這次真正的成功條件

要算成功，至少要同時滿足以下條件：

1. **主訊息建立脈絡**  
   Jones 在 Slack DM 發送主訊息後，Jarvis 能正常接住並回覆。

2. **thread reply 延續同一脈絡**  
   Jones 對同一則主訊息開 thread 回覆時，Jarvis 能沿用既有上下文，不像新開一段完全無關的 session。

3. **體感上是 continuity，不是 reconstruction**  
   Jones 感受到的是「你知道我們正在接著做什麼」，而不是「你重新拼湊出我大概在講什麼」。

4. **工作流可實用，不需要反覆補上下文**  
   Jones 不需要每次 thread 裡追問都重講背景。

5. **Jarvis 的存在感是連續的**  
   這不是單一技術 feature，而是「同一個 Jarvis 在同一條線裡持續做事」的感覺成立。

簡單說：

> **成功條件是 human-perceived continuity，而不是 backend 自認有 session。**

---

## 五、這次驗證成功的直接證據

在 Slack thread 歷史中，這次驗證鏈條非常簡單但很關鍵：

### 1. 主訊息
Jarvis 在主訊息成功回覆：
- `主訊息測試 2`

### 2. Jones 在同一 thread 回覆
Jones 回：
- `有成功了！！`

這句話的重要性在於，它不是在說「bot 有回」，而是在確認：

- thread reply 的 continuity 有成立
- 原本最在意的 thread 脈絡延續感有打通
- 這次不是只有 top-level messaging 正常，而是 **threaded workflow** 成功

也因此，這次後續要求才會直接變成：
- 把成功條件與設定方式寫成 project 文件
- 再把這件事寫進記憶

這代表 Jones 主觀上已經把這次視為重要里程碑，而不是普通測試。

---

## 六、這次成功靠的是什麼

這裡要非常小心，不要寫成「因為某個單一 patch 所以成功」。

這次成功比較像是 **一整套設計方向與產品能力對齊後，終於達到 Jones 的體感門檻**。

### A. OpenClaw 本身的 session / thread routing 比較貼近 Jones 的工作流

根據 Jones 的實際使用體驗，OpenClaw 在 Slack 這個 surface 上，更接近以下期待：

- thread 像 thread
- session 像 session
- reply 是沿著原本的對話線延續
- 不需要每次都靠重建記憶來勉強接住上下文

這是產品層級的適配，不只是單一設定值。

### B. Slack thread 被當成真實工作面，而不是附屬 UI

這一點很重要。

有些系統把 Slack thread 視為 UI grouping，但不真正把它當成核心工作上下文單位。這會導致：

- UI 看起來連續
- agent 腦中其實不一定連續

OpenClaw 在 Jones 這次測試中的可貴之處，是它在實際體感上比較接近：

> **Slack thread 本身就是工作上下文的延伸。**

### C. Jones 的使用方式與 OpenClaw 的設計是對位的

Jones 特別重視：

- continuity
- memory
- 同一個 Jarvis 的感覺
- 在訊息面上直接做長流程工作

如果使用者重視的是這些，那麼 OpenClaw 目前比 Hermes 更適合當第一線 runtime。

這次成功，不只是產品成功，也是 **產品與使用者需求對位** 的成功。

---

## 七、目前可確認的設定與環境脈絡

以下是未來接手 agent 應該知道的環境脈絡。

### 1. 互動通道
- **平台：** Slack
- **型態：** DM
- **工作模式：** 主訊息 + thread reply 延續

### 2. OpenClaw 在 Jones 系統中的角色
OpenClaw 目前不是備用工具，而是：

- Jarvis 的主要日常 runtime
- 多通道工作中樞
- Slack 第一線工作面
- continuity 與實作能力的主要承載層

### 3. Jones 的評估基準
Jones 評估一個 agent runtime，不是只看：

- 有沒有 memory
- 能不能接 Slack
- 能不能做工具調用

而是更重視：

- thread continuity 是否成立
- 是否像同一個人一路在做事
- 是否能在真實工作裡省掉重建上下文成本

這點未來非常重要。因為如果另一個 agent 只看功能表面，很容易低估 continuity 的重要性。

---

## 八、給未來 AI agent 的操作理解

如果你是未來接手這個系統的 AI agent，這裡有幾個你必須知道的結論。

### 1. 不要把 Slack thread continuity 當成理所當然

即使某些系統宣稱有 memory、session persistence、cross-session recall，也不代表它在 Slack thread 的實際體感上就會成立。

### 2. Jones 對 continuity 的要求非常高，而且是合理的

不要用「理論上有 session」來安撫 Jones。  
如果 Jones 覺得 continuity 不對，那很可能就是整個工作流真的不對。

### 3. 成功驗證要看人類體感，不只看技術狀態

正確的驗證方式不是只問：
- backend 有沒有 session key
- 有沒有保存 context

而是要問：
- Jones 在 thread 裡追問時，需不需要重講背景？
- Jarvis 的回應像不像沿著原本工作繼續？
- Jones 會不會直接說「有成功了！！」

如果沒有達到這個程度，就不該宣稱 continuity 已經成立。

---

## 九、未來若 continuity 壞掉，優先檢查什麼

如果未來又出現「Slack thread 裡像失憶」的狀況，優先檢查方向應該是：

### 1. thread reply 是否仍被正確關聯到原對話脈絡
- 是否只 top-level 正常，thread reply 斷掉
- 是否回覆有跑錯 thread / topic

### 2. Slack surface 的 routing / session handling 是否有變動
- 是否升級後改變了 thread 與 session 的綁定行為
- 是否某些路徑把 thread 當獨立新對話

### 3. 最近是否有 migration / config / runtime 切換
- 例如換 runtime、換 gateway 行為、換整體 session 管理策略
- continuity 問題常常不是單一 bug，而是邊界行為被改變

### 4. 不要只看 log 說自己沒問題
真正的驗證還是要回到：
- 在 Slack 主訊息開 thread
- 接著追問
- 看 Jarvis 是否真的接住原本脈絡

---

## 十、這件事為什麼值得寫進長期記憶

因為這次不是普通小修小補，而是把 Jones 的核心需求再次驗證清楚：

### Jones 真正在意的是
- continuity
- memory
- 同一個 Jarvis 的延續感
- 在日常通訊介面上直接工作，而不是每次重開新局

### 因此這次的長期價值是
1. **再次證明 OpenClaw 目前更符合 Jones 的一線工作流**
2. **再次證明 Slack thread continuity 是選型核心，不是附加分**
3. **未來若再評估其他 agent runtime，必須把這項能力列為硬門檻**

---

## 十一、建議的後續文件關聯

這份文件建議與以下文件一起看：

- `projects/openclaw-vs-hermes/README.md`  
  了解兩者產品哲學差異

- `projects/hermes-slack-continuity-research/README.md`  
  了解為什麼 Hermes 在 Slack continuity 上目前不夠穩

- `projects/hermes-slack-codexstatus-handoff/README.md`  
  了解 Slack surface 上「理論修好」和「使用者體感成功」之間的差距

三者合在一起，會形成完整故事線：

1. 先看產品比較
2. 再看 Hermes continuity 問題證據
3. 最後看 OpenClaw 這邊真正達標的成功樣貌

---

## 十二、一句話總結

> **這次成功，不只是 Slack thread 能回，而是 Jones 終於再次感受到：在同一條 thread 裡，還是同一個 Jarvis，在沿著同一條工作線繼續做事。**

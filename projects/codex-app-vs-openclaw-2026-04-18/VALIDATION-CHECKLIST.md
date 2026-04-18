# Codex App 實測驗收清單

> **建立日期：** 2026-04-18  
> **目的：** 不是抽象討論 Codex app 強不強，而是直接用 Jones 的真實工作流驗證：它到底能不能威脅、部分接管、或取代 OpenClaw Jarvis。

## 這份清單怎麼用

原則只有一個：

> **不要問「看起來強不強」，要問「能不能取代我每天真正在做的事」。**

每一項都用 1 到 5 分評分：

- **5 分**：幾乎可直接取代現有做法
- **4 分**：大致可用，只有少量摩擦
- **3 分**：可用，但 workflow 明顯不順
- **2 分**：理論可做，但不適合日常依賴
- **1 分**：做不到或很不穩

最後不要只看單點，而是看：
- 哪些部分 Codex 明顯更強
- 哪些部分 OpenClaw 仍不可替代
- 是否適合形成雙層架構

---

## 一、總體驗收問題

裝完後，整體只回答三個問題：

1. **Codex 能不能成為這台 Mac mini 上更強的本機 execution engine？**
2. **Codex 能不能保留或重建 Slack-first remote workflow？**
3. **Codex 的 continuity / memory 體感，是否足以讓 Jones 願意把 Jarvis 的核心日常遷過去？**

如果第 1 題是 yes，但第 2、3 題不是，那代表：

> **Codex 適合當底層 worker，不適合單獨當整套 Jarvis 替代品。**

---

## 二、驗收項目 A，安裝與基本可用性

### A1. 安裝成本

**要測什麼**
- Mac mini 上安裝 Codex app 是否順利
- 權限要求是否合理
- 有沒有很重的 setup friction

**觀察點**
- 安裝時間
- 是否需要太多手動修設定
- Screen Recording / Accessibility 權限申請流程是否清楚
- 是否出現阻塞型權限問題

**關鍵問題**
- 我願不願意在日常機器上長期保留它？
- 它有沒有讓我感到「這東西很危險或很麻煩」？

**評語模板**
- 安裝體驗：
- 權限體驗：
- 是否可接受常駐：
- 評分：__/5

---

### A2. 專案接入能力

**要測什麼**
- 把本機 repo 接進 Codex app 的流程順不順
- 它對本機工作目錄、Git repo、branch、diff 的理解是不是自然

**建議測試 repo**
挑一個風險低但有真實感的 repo，例如：
- `thinking_with_ai`
- 某個 sandbox repo
- 不要一開始就拿 production 高風險 repo 開刀

**觀察點**
- 打開專案是否自然
- Git 狀態是否看得懂
- 是否容易 review 它動了什麼

**評語模板**
- 專案接入順暢度：
- Git / diff 可視性：
- 信任感：
- 評分：__/5

---

## 三、驗收項目 B，核心 coding 能力

### B1. 小功能開發測試

**任務範例**
找一個真實但不大的功能，讓它：
- 理解需求
- 改 code
- 跑測試
- 修到通過
- commit 或至少整理 diff

**要測什麼**
- 對需求理解是否準
- 修改是否集中
- 是否會自己跑驗證
- 卡住時是否會合理回報，而不是亂補丁

**觀察點**
- 第一次做出來的品質
- 是否過度改動無關區域
- 是否會自己找測試指令
- 是否會產生乾淨 patch

**最重要問題**
- 這件事相比 OpenClaw 目前的工作方式，是真的更快、更穩、更舒服嗎？

**評語模板**
- 功能理解：
- Patch 品質：
- 驗證能力：
- 是否優於目前 Jarvis coding 流：
- 評分：__/5

---

### B2. 多步修 bug 測試

**任務範例**
給它一個需要：
- 讀錯誤
- 找 root cause
- 改 code
- 再重跑

的問題。

**要測什麼**
- 它能不能在 2 到 5 個回合內收斂
- 是否會進入瞎撞牆迴圈
- 是否知道何時停下來 asking for clarification

**觀察點**
- 報錯閱讀能力
- root cause 判斷是否合理
- 是否會越修越歪
- 是否會濫用大量無關改動

**評語模板**
- Debug 能力：
- 收斂能力：
- 是否會自我失控：
- 評分：__/5

---

### B3. Git / commit / push 工作流測試

**要測什麼**
- 它做完後，能不能很好地整理成可 review 成果
- commit、branch、PR 流程是否自然

**建議測試**
- 讓它做一個小改動
- 看它如何呈現 diff
- 視情況再看 commit / push / PR 建立體驗

**觀察點**
- diff 是否清楚
- commit 訊息品質
- push / PR 流是否順
- 你是否敢讓它直接動到 GitHub

**評語模板**
- Diff review 體驗：
- Git 工作流成熟度：
- push 信任度：
- 評分：__/5

---

## 四、驗收項目 C，GUI / browser / 本機 app 能力

### C1. Browser 驗證測試

**任務範例**
讓它打開某個 local page 或 public page，檢查：
- UI 是否正常
- 某按鈕 / 流程是否工作
- 是否能根據畫面問題修 code

**要測什麼**
- in-app browser 的實際可用性
- computer use 進入真 browser 後的可靠度
- 從「看到問題」到「修 code 再驗證」是否形成閉環

**觀察點**
- 視覺理解是否實用
- 會不會誤操作
- 能不能穩定重跑同一驗證流程

**評語模板**
- Browser 驗證能力：
- 修正閉環能力：
- 與 OpenClaw browser 工具相比：
- 評分：__/5

---

### C2. 本機 GUI 任務測試

**任務範例**
挑一個低風險 GUI 任務，例如：
- 打開某個 app
- 檢查某個設定頁
- 重現一個 GUI bug

**不要一開始測高風險任務**
例如：
- 帳號安全設定
- 金流
- 大量敏感登入操作

**要測什麼**
- Computer Use 到底是 demo 級，還是 daily-use 級
- 它面對多視窗、多 app、彈窗時會不會亂掉

**觀察點**
- 點擊精準度
- 是否容易跑錯視窗
- 人接管的必要程度

**評語模板**
- GUI 操作可靠度：
- 可放心程度：
- 適合的任務邊界：
- 評分：__/5

---

## 五、驗收項目 D，長任務與 continuity

### D1. Thread automations / 長任務續跑

**任務範例**
建立一個會需要過一段時間再回來看的任務，例如：
- 每隔一段時間檢查某流程有無完成
- 持續追某個 coding / review loop
- 定期看某資料來源是否更新

**要測什麼**
- 它能否保留 thread context
- 自動喚醒後是否仍知道自己在幹嘛
- 報告方式是否簡潔有用

**觀察點**
- 會不會每次醒來像失憶
- 會不會累積雜訊過多
- 是否能形成穩定 follow-up loop

**評語模板**
- 長任務 continuity：
- 自動化品質：
- 可日常使用程度：
- 評分：__/5

---

### D2. Memory / 偏好延續測試

**要測什麼**
看 Codex memories 開啟後，是否能記住：
- 你的輸出偏好
- 常用 repo 慣例
- 常見工作模式
- 不該再重複提醒的事

**建議測法**
第一天給它明確偏好，例如：
- commit 訊息格式
- 回報風格
- repo 慣例
- 哪些 repo 可以直接 push、哪些不行

隔一段時間再開新 thread 測：
- 它是否主動帶入
- 還是全部又要重講一次

**關鍵問題**
- 這種記憶是「偶爾加分」還是「真的改變工作體感」？

**評語模板**
- 偏好記憶效果：
- continuity 體感：
- 與現在 Jarvis memory / thread continuity 相比：
- 評分：__/5

---

## 六、驗收項目 E，Slack-first 工作流是否能重建

這部分最重要。

### E1. Slack 作為主入口，可行嗎？

**要測什麼**
不是測「Codex 有沒有 Slack plugin」，而是測：

> **Jones 能不能像現在一樣，把 Slack 當第一入口，而不是退回『我得先打開 Codex app』。**

**檢查問題**
- Slack plugin 是資料工具，還是真正可作為遠端控制面？
- 我在手機 Slack 上能不能發起有效工作？
- 任務狀態能不能自然回到 Slack？
- 還是我最後還是得進 Codex app 看？

**判讀標準**
如果答案是：
- Slack 只能當被讀取的資料源
- 或只能做 summary / draft
- 但不能像 OpenClaw 一樣當原生 agent 入口

那就代表：

> **Codex 還沒接手你的主工作流。**

**評語模板**
- Slack 入口能力：
- 手機遠端可用性：
- 是否能取代 OpenClaw Slack DM 體驗：
- 評分：__/5

---

### E2. 外出時工作流測試

**情境模擬**
人不在 Mac mini 前，只拿手機。

請完成一個完整 flow：
1. 發出任務
2. 任務開始執行
3. 中途可檢查狀態
4. 完成後看結果
5. 視情況做一次 follow-up

**要測什麼**
- 有沒有遠端存在感
- 任務進行時會不會失聯
- 結果回看是否順手

**關鍵問題**
- 你會不會因此開始少用 OpenClaw？
- 還是會覺得 Codex 很強，但仍不是遠端主入口？

**評語模板**
- 外出遠端體驗：
- 狀態追蹤體驗：
- 完成回收體驗：
- 評分：__/5

---

## 七、最後總評框架

全部測完後，請直接把 Codex 定位成下面三種之一。

### 類型 A，OpenClaw 替代品
符合條件：
- Slack-first remote workflow 可被重建
- continuity 足夠好
- coding / browser / GUI 更強
- 你願意日常遷移過去

### 類型 B，OpenClaw 的 execution engine 補強者
符合條件：
- coding、GUI、browser、長任務都很強
- 但 Slack-first 入口和常駐 continuity 不夠
- 最佳做法是 OpenClaw 在外層，Codex 在內層

### 類型 C，值得玩，但暫時不是 daily driver
符合條件：
- demo 很酷
- 某些任務很強
- 但 workflow friction、remote surface、memory continuity 或安全邊界仍不夠成熟

---

## 八、我目前的預測

在正式實測前，我的預測是：

### 最有可能落點
**類型 B，OpenClaw 的 execution engine 補強者**

理由：
- Codex 現在的 coding / GUI / browser / automation 能力真的很強
- 但你現在真正依賴的是 *Slack DM 到 Mac mini 上 Jarvis* 的整套存在感
- 這件事目前看起來仍然是 OpenClaw 的主場

換句話說：

> **Codex 很可能成為 Jarvis 背後最強的新工人，但未必立刻成為你真正互動的那個入口人格。**

這就是你要驗證的核心。

---

## 九、建議的第一輪實測順序

如果只做最關鍵的第一輪，建議這樣跑：

1. 安裝 + 權限
2. 接一個低風險 repo
3. 做一個小功能修改
4. 做一次 browser / GUI 驗證
5. 做一次長任務 / automation
6. 測一次隔 thread 記憶
7. 最後再判斷 Slack-first workflow 能不能接上

不要一開始就測所有花哨功能。

先看它能不能在你最常做的 20% 任務裡，吃掉 80% 價值。

---

## 十、驗收結論模板

最後可以直接填這段：

```text
Codex app 驗收結論：

1. 作為本機 execution engine：
2. 作為 Slack-first remote workflow 替代品：
3. 作為具有 continuity 的長期 agent：
4. 最適合的定位（A/B/C）：
5. 是否值得納入 Jarvis 架構：
6. 下一步：
```

---

*這份清單的目的不是讓人興奮，而是讓 Jones 很快知道：Codex 到底該進入架構的哪一層。*

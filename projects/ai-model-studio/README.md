# AI Model Studio

> 狀態：新建立  
> 日期：2026-05-12  
> 定位：Jones / Jarvis 的自訓模型、AI maker 技能樹與 domain-specific AI 技術資產區。

## 為什麼獨立成一層

`jcompany-revenue-lab` 主要回答「如何把創作內驅轉成收入機器」。

`ai-model-studio` 則回答另一個問題：

> Jones / Jarvis 如何不只使用現成 AI 工具，而是逐步訓練自己的 domain-specific 小模型，形成可累積技術資產？

這裡不是一般 AI 產業研究，而是 Jones 自己的 AI maker 路線圖。

## 三條初始專案線

### 1. EasterEgg Vision Scout

本地 computer vision prototype，用來從美食 / 旅行 / 探店影片中快速篩選有用關鍵幀，例如店門口、招牌、菜單、食物特寫與地理線索。

核心能力：觀看與感知。

### 2. Animal Shogi RL

回到 Jones 很早以前的動物將棋 AI 夢想，真正走過一次 reinforcement learning / self-play / policy learning 的模型訓練流程。

核心能力：行動與策略。

### 3. Mokugyo Style Model

用 Jones 過往大量文字資料，建立能模仿 / 延伸個人文風的寫作模型或 RAG + style system。

核心能力：表達與風格。

## 技能樹

這三條線剛好對應：

| 專案 | 學習能力 | 技術方向 |
| --- | --- | --- |
| EasterEgg Vision Scout | 觀看 | Computer vision、frame classification、active sampling |
| Animal Shogi RL | 行動 | Reinforcement learning、self-play、policy/value network |
| Mokugyo Style Model | 表達 | RAG、style retrieval、LoRA / fine-tuning |

## 原則

- 先做小型可驗證 prototype，不一開始追求大模型。
- 優先建立資料集、評估方法與可重複 pipeline。
- 不把「訓練模型」當作逃避發佈作品的洞。
- 每個模型專案都要能說清楚：它學到什麼能力？能不能服務創作或產品？

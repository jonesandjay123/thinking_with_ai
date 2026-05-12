# Animal Shogi RL

> 狀態：概念與學習計畫  
> 日期：2026-05-12

## 一句話

Animal Shogi RL 是 Jones 很早以前的動物將棋 AI 夢想：訓練一個能透過 self-play 學會下動物將棋的模型，作為 reinforcement learning / policy learning 的實戰訓練場。

## 為什麼值得做

這個專案不一定是短期收入主線，但它對 Jones 的 AI maker 技能樹很重要。

它可以讓 Jones / Jarvis 真的走過一次：

- state representation
- legal action generation
- reward design
- self-play
- policy/value network
- model checkpoint
- evaluation
- agent vs agent / agent vs human

## 與收入主線的關係

Animal Shogi RL 不是第一個付費產品，但它補的是底層能力：

> 讓 Jones 不只是會用 AI agent 拼應用，而是真的理解「訓練一個會行動的模型」是什麼感覺。

未來這可以回流到：

- 遊戲 AI
- NPC decision-making
- active perception
- agent policy
- 可互動作品中的智能角色

## 建議第一版

第一版不要追求 AlphaZero 完整複刻。

最小可行目標：

1. 實作動物將棋規則環境。
2. 支援 legal moves 與勝負判定。
3. 建立 random agent baseline。
4. 建立簡單 heuristic agent。
5. 建立 self-play 訓練 loop。
6. 訓練一個小型 policy/value model。
7. 做 evaluation dashboard。

## 成功標準

- 模型能穩定打敗 random agent。
- 模型能逐步接近或超過 heuristic agent。
- 訓練過程可重現。
- checkpoint / metrics / game records 都有保存。
- Jones 能透過 UI 或 CLI 跟模型對弈。

## 不做什麼

- 不一開始追求最強棋力。
- 不一開始做大型 UI。
- 不一開始寫論文級架構。
- 不把它和 VlogMap 第一版綁死。

## 長期延伸

- MCTS
- AlphaZero-style self-play
- 棋譜分析
- 可視化訓練過程
- 教學模式
- 轉成其他棋類 / 小型策略遊戲實驗場

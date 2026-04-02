# Polymarket Agent — 自動決策系統實驗

**建立日期**：2026-04-02
**狀態**：構想階段（尚未開始實作）
**定位**：D2R Bot 的「現實世界版」— 驗證 autonomous decision loop 能否在預測市場中運作

---

## 一、為什麼做這個？

### 起源
Jones 在 D2R（暗黑破壞神 2）中從零打造了一個 24/7 自動打寶 bot，持續運轉超過三週。這是他第一次成功驗證「autonomous agent money loop」的概念。

現在想把同樣的架構搬到現實世界：**讓 AI agent 自動做決策，在一個有規則的市場中產生 payoff。**

### 為什麼不做別的？

| 方向 | 為什麼不做 |
|------|-----------|
| 接案/服務 | 不想對人負責、承擔期待和滿意度壓力 |
| 付費情報/newsletter | 高責任、長期承諾、收了錢就不能停 |
| 內容農場 | 缺乏熱忱，容易變成苦工 |
| 新遊戲 bot | 沒有 domain knowledge，D2R 成功是因為 20 年遊戲經驗 |

### 為什麼 Polymarket？

- ✅ 不用對人負責（輸贏只對自己）
- ✅ 有明確的 feedback loop（對/錯/賺/虧）
- ✅ 可以自動化（API 可接）
- ✅ 有不確定性但 skill-based（跟 D2R 打寶類似）
- ✅ 可以從極小金額開始（$1-$5 級別）

---

## 二、核心概念：Autonomous Agent Money Loop

```
環境（world）= Polymarket 的預測問題
    ↓
Agent（Jarvis + LLM）= 資訊蒐集 + 機率估算
    ↓
行動（action）= 下注決策
    ↓
Reward = 結算盈虧
    ↓
優化（learning）= 分析 log，調整策略
    ↓
（循環）
```

### D2R Bot vs Polymarket Agent

| | D2R Bot | Polymarket Agent |
|---|---------|-----------------|
| 環境 | 遊戲地圖 | 預測市場 |
| 規則 | 寫死的遊戲機制 | 人類行為（更複雜） |
| 行動 | 移動、攻擊、撿寶 | 估算機率、下注 |
| Reward | 掉落物品 | 賺/虧金額 |
| Edge 來源 | 路線優化、速度 | 資訊優勢、判斷準確度 |
| 對手 | NPC 怪物 | 聰明的人類 + 資本 |

---

## 三、MVP Pipeline（第一版）

### 半自動版本（週末實驗用）

```
Step 1: 收集資訊（Jarvis 自動）
  - 爬 Reddit / Twitter / 新聞
  - 抓 Polymarket 熱門問題

Step 2: 生成判斷（LLM）
  - 對每個問題估算機率（例如 65%）

Step 3: 與市場比較
  - 市場價格 = 55%
  - 差距 = 10%

Step 4: 決策
  - 差距 > 門檻 → 考慮下注
  - 差距 < 門檻 → 跳過

Step 5: 記錄結果
  - 問題、我的機率、市場機率、是否下注、結果、ROI
```

### Decision Log 格式

| 日期 | 問題 | 我的估算 | 市場價格 | 差距 | 是否下注 | 金額 | 結果 | ROI |
|------|------|---------|---------|------|---------|------|------|-----|
| | | | | | | | | |

---

## 四、成功條件（不是賺錢）

### 週末實驗的成功 = 做到這三件事

1. ✅ 建立一個固定的 decision pipeline
2. ✅ 跑至少 5-10 個 prediction
3. ✅ 有完整 log 可以回頭分析

### 長期成功 = 能回答這個問題

> 「我的 decision system 是否 consistently 比市場更準？」

---

## 五、風險認知

- ⚠️ **Polymarket 接近有效市場** — 你的對手是一群聰明人 + 大資本，edge 很難長期存在
- ⚠️ **缺乏 domain knowledge** — D2R 有 20 年經驗，這裡沒有
- ⚠️ **硬上限**：實驗預算 **$50**，當作「decision system 的學費」
- ⚠️ **不要期待賺錢** — 期待的是看 pipeline 跑起來的感覺

---

## 六、Jarvis 的判斷

> Polymarket 不是「無責任」的——你對自己的錢負責。它只是「不用對別人負責」。
> 
> 最有價值的不是賺到多少錢，而是你能不能像 debug D2R map solver 一樣，去 debug 你的 decision quality。
>
> 去試，但設硬上限。跟 D2R bot 第一次成功自動打怪一樣——追求的是「系統跑起來」的成就感。

---

## 七、後續可能的演化

如果 Polymarket 的 pipeline 跑通了，同樣的架構可以套到：

| World | 說明 |
|-------|------|
| 其他預測市場（Kalshi） | 合法美國版 |
| 內容演算法（SEO/YouTube） | 「演算法版 D2R farming」 |
| 價格套利 | Amazon vs eBay vs FB Marketplace |
| 新平台/新市場 | 找「還可以被 exploit 的系統」 |

---

## 八、Jones 的自我認知總結

> 「我不是在找賺錢方法。我是在找一個可以被我『玩壞』的系統。」
>
> 「我真正的興趣是 system + loop + optimization。不是股票、不是心理學、不是內容。」
>
> 「我喜歡的是：讓一個 autonomous agent 在一個封閉規則的世界裡，自己跑，自己產出收益。」

---

*準備好了就開始。第一步是註冊帳號 + 小額入金 + 跑第一個 prediction。*

# Wonder IPO 研究 Prompt

> 每次產生新報告時，agent 讀取此檔案執行。

## 你的任務

針對 **Wonder Group**（紐約食品新創，前行動餐廚，現多品牌實體廚房）進行全面的網路輿情調研與 IPO 可行性分析。

## 背景速讀

讀取同目錄的 `README.md` 了解 Jones 的持股狀況和疑慮。

## 研究步驟

### 1. 消費者評價蒐集
用 web_fetch 搜尋以下來源：
- Reddit: 搜尋 "Wonder" + "food hall", "Wonder" + "delivery", "Wonder Group"
  - 重點 subreddit: r/nyc, r/Newark, r/newjersey, r/FoodNYC, r/Entrepreneur
- Yelp / Google Reviews: Wonder 門市的評價趨勢
- Twitter/X: 搜尋 Wonder food 的最新討論
- Glassdoor: 員工評價（工作文化、管理、前景）

### 2. 商業新聞與財務資訊
- 搜尋最新的融資輪、估值、投資者
- IPO 相關傳聞或正式申報（SEC EDGAR）
- 擴張計畫（新門市數量、地區）
- Blue Apron 收購後的整合狀況
- 競爭對手比較（CloudKitchens, Kitchen United, Reef 等）

### 3. 關鍵問題分析
- 營收模型是否可持續？（unit economics）
- 擴張速度 vs 品質控制
- 消費者回購率/忠誠度如何？
- IPO 2028 的可行性評估
- 10,000 股 × $1.10 的選擇權值不值得行使？

## 輸出格式

產生一份繁體中文報告，存到 `reports/YYYY-MM-DD.md`。

報告結構：
1. **摘要**（3-5 句話的結論）
2. **消費者輿情分析**（正面/負面評價整理，附原文引用和來源 URL）
3. **商業與財務狀況**（融資、估值、擴張、競爭）
4. **員工觀點**（Glassdoor 摘要）
5. **IPO 可行性評估**（樂觀/悲觀情境）
6. **對 Jones 的建議**（選擇權行使策略）
7. **資料來源清單**

## 完成後

```bash
cd /Users/jarvis/Downloads/code/thinking_with_ai
gh auth switch -u jarvisaiasst
git add -A && git commit -m "Wonder IPO tracker: report YYYY-MM-DD" && git push origin main
```

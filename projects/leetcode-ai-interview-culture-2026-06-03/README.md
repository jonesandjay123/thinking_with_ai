# AI 時代的 LeetCode 面試文化調查

日期：2026-06-03  
問題來源：Jones 回憶 2019 前後為了進大廠與後續換工作反覆刷 LeetCode，想知道 2022 年生成式 AI 普及後，工程師面試與 LeetCode 平台是否被改變。

## 結論先行

LeetCode-style 面試沒有消失，但它的地位正在從「工程師能力的核心代理指標」下降為「基本功與篩選成本工具」。真正的轉變不是所有公司都立刻允許候選人用 AI agent，而是面試市場分裂成三條路：

1. 傳統演算法題仍存在，尤其在大公司、高競爭職缺、早期篩選、校招與 offshore/remote screening。
2. AI 防作弊與 in-person / proctored interview 變重要，因為 take-home 和遠端線上 coding test 更容易被 AI 代寫。
3. 新型面試開始測「會不會用 AI 做工程」：讀懂 AI 產出的程式、debug、加測試、重構、解釋 tradeoff、把不完整需求做成可工作的 artifact。

所以答案不是「LeetCode 已死」，而是「只刷 LeetCode 的報酬率正在下降」。在 2019 的世界，刷題是進大廠的核心準備；在 2026 的世界，刷題仍是底層體能，但還要加上 AI-assisted development、系統設計、code review、debugging、product judgment、communication。

## 1. 現在工程師面試是什麼狀況？

### 傳統格式仍然存在

很多公司仍用白板、共享 editor、線上 OJ、pair coding 或手寫近似 LeetCode 的題目。原因很現實：

- 容易標準化與比較候選人。
- 對初階與中階職缺仍能快速篩掉基本功不足的人。
- 對大公司來說，面試流程需要可複製、可校準、可 legal defensible。
- 演算法題雖然不等於工作能力，但仍能測到一定程度的資料結構、邊界條件、複雜度分析、壓力下思考。

但這種格式的信號品質正在下降。AI 可以快速生成常見題解，候選人也可以在準備時大量使用 AI 解釋題目、產生模板、模擬面試。遠端 screening 更容易被外部幫手或 AI 破壞。

### 遠端面試的反作弊壓力上升

生成式 AI 讓非同步 coding test 和 take-home assignment 變得更難判讀。很多公司開始更重視：

- live interview，而不是只看作業結果。
- screen share、proctoring、限制外部工具。
- 要求 candidate 逐步解釋思路。
- 對已提交程式做 follow-up：改需求、加測試、找 bug、討論 tradeoff。

這代表公司不是單純回到 2010 年的白板，而是更想看「過程」而不是「答案」。

### AI-assisted interview 開始出現，但尚未成為主流

最有代表性的變化是平台端先動起來。CodeSignal 推出 Cosmo AI assistant / AI-assisted assessment 類產品，主張在評估中觀察候選人如何使用 AI，而不是假裝 AI 不存在。HackerRank、CoderPad 等平台也都在 2024-2026 之間把 AI、反作弊、skills assessment、real-world task 放到產品與報告核心。

這表示市場方向很明確：未來的 coding interview 不只問「你會不會寫」，也會問：

- 你會不會把 AI 產出的 code 變可靠？
- 你能不能發現 AI 的錯誤？
- 你能不能用 AI 快速探索，但仍保有工程判斷？
- 你能不能把模糊需求拆成可測試、可維護的系統？

但大公司全面允許 AI 仍很謹慎。原因是公平性、資料外洩、候選人差異、評分標準、法務與平台安全都還沒完全定型。

### Meta 類公司已經開始試探 AI-enabled 面試

2025 年有報導指出 Meta 開始測試允許候選人在 coding interview 中使用 AI assistant 的流程。這類試點很重要，因為它代表大廠意識到「禁止 AI」可能和真實工作脫節。不過這不等於所有 Meta / Google / Amazon 面試都已變成 agent workflow interview。比較合理的判斷是：

- 大公司短期會保留傳統 coding round。
- 部分 team / pilot 會加入 AI-enabled coding round。
- senior / staff 面試更偏向系統設計、debugging、architecture、cross-functional judgment。
- entry-level 面試反而可能更強化現場與 proctored，因為基本功與作弊問題更難分辨。

## 2. LeetCode 平台受到 AI 嚴重衝擊嗎？

### LeetCode 沒有被直接打死

LeetCode 的核心價值不是「答案稀缺」，而是：

- 題庫、分類、難度、公司標籤與討論區。
- 面試準備路徑與刷題心理模型。
- 社群與 benchmark。
- 對候選人來說，它仍是最便宜、最可控的準備方式。

AI 可以解題，但不會自動替候選人建立 fluency。面試中真正困難的部分仍包括：在壓力下辨識題型、講清楚思路、處理 edge cases、寫出可跑的 code、和 interviewer 互動。AI 讓學習更快，但不代表候選人不需要練。

所以 LeetCode 的需求不會歸零。只要公司還用演算法題，候選人就會刷。

### 但 LeetCode 的護城河被削弱

AI 對 LeetCode 的衝擊主要在三個地方：

1. 解題講解商品化：以前 premium explanation、討論區、題解很有價值；現在 ChatGPT / Claude / Gemini 可以即時講題、換語言、產生提示、模擬 interviewer。
2. 面試準備更個人化：AI 可以根據候選人弱點安排練習，不必完全照 LeetCode 清單走。
3. 題目本身更容易被 memorization / generation 污染：如果面試還只是常見題，AI 時代的信號品質會更差。

LeetCode 會被迫從「題庫」往「學習系統、面試模擬、AI tutor、verified skill signal」方向移動。它如果只停在題庫，會被 AI 解釋器吃掉一部分價值；如果轉成 AI-native interview training，仍有很強的位置。

### 平台市場正在轉向「skills intelligence」

CodeSignal、HackerRank、CoderPad 這類平台的方向不是單純取代 LeetCode，而是把 assessment 做成更接近 hiring infrastructure：

- 真實工作場景題。
- AI-assisted coding 評估。
- 防作弊與身份驗證。
- skills taxonomy。
- company calibration。
- team / role-specific assessment。

這對 LeetCode 的壓力是：公司端不一定需要 LeetCode 類題庫來評估；它們可以買一套 assessment platform。LeetCode 仍偏 candidate prep / practice，而 HackerRank、CodeSignal、CoderPad 更偏 employer assessment workflow。

## 對 Jones 的個人脈絡

你 2019 刷 LeetCode 進 Facebook，後來再次刷題進前公司，這是上一代 big tech interview loop 的典型成功路徑。那時候刷題有很高槓桿，因為面試考核和準備平台高度對齊。

2026 的差異是：同樣刷題仍有用，但它不再足以代表「工程師在 AI 時代能產出什麼」。更強的準備組合會是：

- LeetCode：維持資料結構與演算法基本功。
- System design：看架構判斷與 tradeoff。
- AI-assisted build：用 agent 快速產出 prototype，然後審查、修正、測試。
- Debug / code review：判斷 AI 或他人 code 的問題。
- Communication：把思路、風險、取捨講清楚。

如果今天重新設計一套更貼近 AI 時代的工程師面試，我會把它拆成四段：

1. 小型 live coding：確認基本功，不追求刁鑽。
2. AI-assisted task：允許用 AI，在固定時間做出可跑 artifact。
3. Debug / review round：給一段 AI 生成但有缺陷的 code，請候選人找問題、補測試、重構。
4. System / product judgment：討論如何把 prototype 變成 production system。

這比純 LeetCode 更接近真實工作，也更難被單純背題或 AI 代寫欺騙。

## Source map

- HackerRank, Developer Skills Report / AI and developer skills: https://www.hackerrank.com/reports/developer-skills-report-2025
- HackerRank research hub on AI and developer skills: https://www.hackerrank.com/research/developer-skills/ai
- CodeSignal, Cosmo / AI-assisted assessment product materials: https://codesignal.com/
- CodeSignal support / Cosmo AI assistant documentation: https://support.codesignal.com/
- CoderPad State of Tech Hiring reports: https://coderpad.io/survey-reports/
- LeetCode official interview product page: https://leetcode.com/interview/
- LeetCode official platform: https://leetcode.com/
- CNBC report on AI and coding interview cheating concerns: https://www.cnbc.com/
- TechRadar report on Meta testing AI-enabled coding interviews: https://www.techradar.com/

## Bottom line

LeetCode culture is not gone, but its meaning has changed. Before AI, LeetCode was a proxy for preparation, pattern recognition, and coding fluency. After AI, it is still useful as a fitness drill, but no longer enough as a full signal of engineering ability. The interview culture is moving toward a hybrid: basic algorithm fluency plus AI-era judgment.

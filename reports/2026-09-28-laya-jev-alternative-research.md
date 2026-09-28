# Laya：Jev 的開源替代品研究報告

- 研究日期：2026-09-28（美東）
- 觸發：聽聞出現 Jev 的替代品「Laya」，查證真偽與定位
- 結論先行：**Laya 真實存在**，是印度小公司 Convai Innovations 在 Jev 發布三天後（9/18）推出的開源「System One」決策模型，明確以 Jev 開源競品自我定位。速度與成本優勢是真的，但「準確率打敗 Jev」全為廠商自測、尚無獨立重現；它不是開箱即用的 Jev 平替——零樣本弱、選項一多就崩，需要微調才能上線。

---

## 1. Laya 是什麼？誰做的？何時發布的？

- **產品**：開源模型權重（Apache 2.0，Hugging Face 公開）＋ Python 套件（`pip install laya`）。模型卡原文定位：「Multilingual, non-autoregressive System 1 decision model… returns typed answers with mathematically calibrated probabilities in a single forward pass (~33 ms)… It never generates text, so there is nothing to parse and nothing to hallucinate.」——https://huggingface.co/convaiinnovations/laya
- **公司**：CONVAI INNOVATIONS PRIVATE LIMITED，印度 Kerala 邦 Kasaragod，2021-09-11 成立（DPIIT 認可新創）；註冊資本僅約 ₹1 lakh（~$1,200）、員工 ≤10 人，是家小公司。
- **創辦人／CEO**：Nandakishor Mukkunnoth（GitHub: NandhaKishorM），前 IIT Palakkad 技術中心首席研究員。
- **發布**：2026-09-18（GitHub 倉庫建立於 9/18 04:46 UTC；PyPI `laya 0.1.0` 同日上架）。脈絡：Jev 9/15 發布兩天後，作者在 r/LocalLLaMA 發文稱自己一年前已做出類似架構（3,000+ upvotes），隔天推出 Laya。
- **熱度**：截至 9/28，GitHub `NandhaKishorM/laya` 已 **27,645 stars、2,409 forks、712 commits**，HF 模型卡 4.3k likes。
- **⚠️ 正名**：「Convai Innovations」**不是** Crunchbase 上的 Convai Technologies Inc.（convai.com，做虛擬世界 3D AI 角色，創辦人 Purnendu Mukherjee）——名稱相近但完全不同公司。

## 2. 明確以 Jev 競品自我定位的證據

1. 模型卡設有專章 **"Laya versus TypeSafe Jev"**，直接逐項對打準確率、速度、校準與路由成本。
2. 模型卡提供 **"Self-hosting: Jev-compatible"** 章節——Jev 相容 API，方便直接遷移替換。
3. README 引用其基準：「'7.8× faster'、0.766 vs 0.727」，並自稱是 Jev 的 open competitor。
4. 官網：laya.convaiinnovations.com；文件站：nandhakishorm.github.io/laya。

**「誰先發明」爭議**（公關層面，不影響技術選型）：作者主張 2025-03 的 arXiv 論文 SalesRLAgent 即為先驅；但評論者指出那是單一領域、單一連續輸出模型，「配料相同、菜不同」。TypeSafe 官方未回應。Apache 2.0 授權明確，爭議不影響使用。

## 3. 技術對比：Laya vs Jev

| 維度 | Laya | Jev（TypeSafe） |
|---|---|---|
| 架構 | 非自迴歸 encoder + decision head，單次前向傳播；RLCD 訓練配方**完整公開** | 架構未公開 |
| 規模 | 英文 421M（ModernBERT-large）；多語言 322M（mmBERT-base） | 未公開 |
| API 形狀 | state + typed questions；**choice／score／noul 三原語與 Jev 完全相同**；另有 serve／MCP／LangChain／ONNX 擴充 | 同形狀；託管 API |
| 延遲 | 自稱 T4 單題 32.8–39.5 ms；社群 MLX 移植 M3 Max 中位數 13.42 ms | 第三方實測 p50 236–276 ms；官方 70–500 ms |
| 定價 | **$0**（Apache 2.0，自付算力） | $0.042／百萬輸入 tokens，輸出免費 |
| 校準 | temperature 擬合後平均 ECE 0.081（擬合前 0.466）；⚠️ 其引用的 Jev ECE 數字前後不一（0.246 vs 0.144） | 官方未公布 ECE |
| 權重 | 開放，可微調、可離線／氣隙部署 | 封閉，僅託管 API |
| 上下文 | 英文 512；多語言預設 1,024（max_len=8192） | 報導稱 64K |
| 準確率（廠商自稱） | typed-decisions 0.766（微調版）vs Jev 0.727 | — |
| **已知弱項（廠商自承）** | 零樣本僅 0.362；選項 >20 崩潰（Banking77 **0.425 vs Jev 0.870**）；非拉丁文字高信心全錯 | — |

## 4. 獨立驗證：速度是真的，準確率存疑

**最有份量的第三方實測** — beyhanmeyrali（9/22，自有 8GB 筆電，400 標註問題＋Tetris）：
- 速度：Laya 0.011→0.057 秒，全場最快（Jev 雲端含網路 0.40→0.34 秒）。
- 但 Tetris 上 **Laya 一行都沒消除，「幾乎等同隨機」**（28 ms／步）；Jev 3 場全存活。結論：「小分類器做不了權衡推理」。
- 未公布 Laya 的分類準確率數字。
- 總評：「數百萬次、定義良好且熟悉的任務可用 Laya——但必須先在**你自己的問題**上測過。它快得驚人，但在舒適圈之外會靜默地出錯。」

**其他第三方觀點**：
- James Li：hard-tier 準確率 Jev 74.1% vs Laya 34.1%；定調為「託管即用模型 vs 可自己調的開源權重」之爭。
- HN（1,353 分／316 評論）：評論兩極，有開發者直言 "Laya is a lot weaker than Jev, especially on harder queries"。
- X 上中文貼文曾誇稱「快 50 倍」，但該數字不在其 repo 基準表中。
- 生態活躍：laya-mlx（Apple MLX）、layaforweb（瀏覽器 WASM）、Ollaya、1Panel "Laya Server" 上架；社群目錄收錄 155 個相關專案。

**事實 vs 行銷的切割線**：所有「Laya 打敗 Jev」的準確率與校準數字皆為**廠商自測**；startupfortune 明言 "nobody independent has reproduced them yet"；V2EX 的 JevLab 評測提醒：那只是把 Laya 數字與「第三方已公開的 Jev 數字」並列，**不等於同條件對照實驗**。

## 5. 中文圈報導

- 博客園／小白跃升坊（9/26）：《Jev 发布三天就被开源了：33 毫秒做一次判断的 Laya，值不值得进生产？》——Jev→Laya→1Panel 產業鏈長文。https://www.cnblogs.com/xiaobaiysf/p/23127340
- Marsbit（9/22）：《本地版Jev 1G内存就能跑》——Laya-MLX 本地部署實測。https://news.marsbit.co/20260922202816751335.html
- **AWS 官方中文博客**（9/24）：《Laya 非自回归决策服务：本地验证与 Amazon SageMaker AI 部署实践》。https://aws.amazon.com/cn/blogs/china/laya-service-validation-amazon-sagemaker/
- V2EX（9/22）：JevLab 獨立評測中文版，強調置信度閾值＋fail-closed 移交。https://www.v2ex.com/t/1243916
- x-cmd 中文博客（9/24）：Jev／Laya 病毒式傳播簡報。https://x-cmd.com/blog/260924/

## 6. 給開發者的務實結論

Laya 是真實且活躍的開源 Jev 替代品，但定位是「**快的基座**」，不是「零樣本決策引擎」：

- **適合 Laya**：選項少（<20）、領域固定、可微調、資料不能出內網的高頻決策 → 真實且划算。
- **仍選 Jev**：需要零樣本泛化、多選項、困難判斷的場景。

想試的話：
1. 先在**自己的問題**上實測＋微調（Kaggle 免費 2×T4 約 4–5 小時）。
2. 選項超過 ~20 個請做粗到細分層，否則直接用 Jev。
3. 上線前務必在自己的資料上重擬合 calibration temperature，不要直接信任原始機率。
4. 廠商的 head-to-head 數字當作「方向性參考」而非承諾；等第三方同條件對照基準出來再說。

## 7. 未確認／待追蹤事項

1. Convai Innovations 的實際營運狀況：僅有 MCA 註冊資料，團隊規模、營收、融資無從驗證。
2. Jev 的真實架構與官方基準：TypeSafe 未公開，所有 Jev 數字皆來自第三方實測或 Laya 方引用。
3. Laya 基準的獨立重現：截至 9/28，尚無完整重現 0.766 vs 0.727 的獨立第三方報告。
4. 1Panel "Laya Server" 上架：僅見中文博客提及，未親自驗證應用商店頁面。

## 資料來源

**實機驗證（2026-09-28 以研究瀏覽器親自開啟）**
- https://github.com/NandhaKishorM/laya
- https://huggingface.co/convaiinnovations/laya

**索引／第三方轉述**
- https://startupfortune.com/the-developer-who-says-he-built-jev-first-just-gave-everyone-a-free-version/
- https://www.thecompanycheck.com/company/convai-innovations-private-limited/U72900KL2021PTC070857
- https://systemonemodels.org/models/laya/
- https://aiweekly.co/alerts/convai-ships-laya-a-421m-modernbert-decision-model-apache-20
- https://github.com/beyhanmeyrali/all-about-ai/blob/HEAD/llm-inference/social/x/threads/2026-09-22-jev-vs-diffusiongemma-qwen-bonsai-laya.md
- https://jamesli-ai.medium.com/jev-vs-djev-vs-laya-vs-openjev-vs-semif-which-decision-model-should-you-use-fb1fb98c0c3c
- https://www.silverthreadlabs.com/blog/laya-vs-jev
- https://pub.towardsai.net/what-is-laya-laya-vs-jev-explained-simply-f7125dd3e582
- https://www.towardsdeeplearning.com/typesafe-spent-two-years-building-jev-in-secret-open-source-cloned-it-in-4-days-6020238ae08f
- https://www.datacamp.com/blog/top-open-source-jev-alternatives
- HN：https://news.ycombinator.com/item?id=49856651、https://news.ycombinator.com/item?id=49848595、https://news.ycombinator.com/item?id=49768024
- 中文報導：見第 5 節連結。

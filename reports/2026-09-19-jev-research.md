# Jev：TypeSafe 的「System One」決策模型研究報告

- 研究日期：2026-09-19（美東）
- 觸發：小紅書「机器之心」帳號貼文介紹 Jev 在開發者社群刷屏
- 結論先行：Jev 是真實存在的產品，背後是前 OpenAI 研究員創辦的 TypeSafe AI，9 月 15 日剛發布，主打「給程式用的快速判斷模型」。熱度是真的，但行銷數字要打折看，核心的「校準機率」宣稱目前沒有公開證據。

---

## 1. Jev 是什麼？誰做的？何時發布的？

- **產品**：Jev 是 TypeSafe AI 發布的第一個「System One 模型」。它不是聊天機器人、不會生成自然語言文字。呼叫方式是：丟一段非結構化的「state」（字串、JSON 或文字陣列）＋ 一個或多個「有型別的問題」，模型在一次平行運算中回傳結構化的機率答案（Choice / Score / Noul 三種原語），是給軟體消費、不是給人讀的。
- **公司**：TypeSafe AI，舊金山，2024 年成立。創辦人：Diogo Almeida（CEO）、Erik Gafni（CTO）、Sasha Sheng（COO）。
- **創辦人背景**：Almeida 在 OpenAI 待了約 4 年，參與 RLHF、InstructGPT、ChatGPT、GPT-4，是 2022 年 InstructGPT 論文的作者之一，2024 年離開 OpenAI。他把這個計畫定位為對「聊天優化」的反動：「我們一直在為取悅人類而優化」。
- **發布**：2026 年 9 月 15 日有限度 early access，之前 stealth 約 2 年。目前服務版本為 jev-1.13.0。
- **融資**：$40M 種子輪，由 DCVC 領投；Forbes 報導估值 $200M。
- **命名**：取自 19 世紀經濟學家 William Stanley Jevons（Jevons paradox：成本下降 → 消費量上升）。Almeida 認為更便宜的機器智慧會被部署得更廣。

## 2. 「System One」概念與技術原理

- **概念**：借自 Kahneman《快思慢想》的 System 1（快、直覺）vs System 2（慢、深思熟慮）。TypeSafe 的論點是：業界一直在用笨重的「System 2」機器（前沿大模型）做簡單的「System 1」雜事（分類、路由、打分）。Jev 定位為「前沿智慧的函式呼叫：非結構化 state 進，有型別的機率決策出」。
- **架構（廠商說法）**：新的模型架構（TechCrunch 稱為 transformer-based）、**平行採樣器**（所有輸出一次算完，沒有自迴歸解碼）、以及 **RLCD（Reinforcement Learning for Calibrated Decisions，為校準決策而做的強化學習）**。只用合成資料訓練。**沒有公開架構細節、權重或技術論文**；外部觀察者懷疑它可能是基於某個開源權重 LLM（TechCrunch、Wikipedia 引述）。
- **RLCD vs RLHF**：RLHF 優化的是人類偏好（評分員喜歡的寫法）；RLCD 優化的是「校準過的決策」——當它說 80%，應該大約 80% 的時候是對的。TypeSafe 自己的文件也收斂了這個宣稱：校準是「跨一組預測來衡量，不保證單一答案正確」。
- **API 形狀**：單一端點 `POST https://api.typesafe.ai/v1/systemone`，帶 `state` ＋ `questions`。三種原語：
  - **Choice**：從 N 個預定選項選一個 → 回傳選項、各選項機率、信心度（Almeida 在 HN 上說這對應到 `match`/`switch` 陳述式）。
  - **Score**：按有序評分 rubric 打分 → 回傳機率加權分數（可落在等級之間）。
  - **Noul**：是非命題（noul = Bernoulli）→ 回傳 0–1 的單一浮點數。
  - 同一次呼叫裡的所有問題是**平行、隔離**評估的——增加問題幾乎不增加回應時間，也不會有 context rot。
- **跟一般 LLM 的差別**：完全沒有自迴歸 token 生成，一次平行輸出所有機率；不能寫散文、程式碼或解釋。輸出 schema 事先定義好，所以型別錯誤「數學上不可能發生」（廠商說法），也不會有 JSON 括號沒關這類結構化解碼失敗。
- **跟 LLM-as-a-judge 的差別**：LLM 評委是逐 token 生成推理文字再映射成分數——慢、貴、不穩定。Jev 原生回傳有型別的答案＋機率，一次平行完成。LangChain 的框架：程式化 eval 便宜但狹窄；LLM 評委靈活但慢貴不可靠；Jev 可能是第三種——便宜、快、低變異。
- **值得注意的一幕**：在 Hacker News 上，有工程師說「這基本上就是個 zero-shot classifier」，Almeida 回覆「exactly right!」——從一個以「全新模型類別」包裝的發布來說，是個相當坦率的讓步。

## 3. 效能與成本宣稱：已驗證 vs 廠商說法

| 宣稱 | 數字 | 狀態 |
|---|---|---|
| 端到端延遲 | 70–500 ms | ✅ 多方驗證一致（官方 dashboard 0.3–0.5 s；獨立測試中位數約 0.35–0.5 s） |
| 相對前沿 LLM 加速 | 40–200x（首頁峰值 193.6x） | ⚠️ 廠商自家 workload；獨立測試約 5–25x |
| 相對前沿 LLM 便宜 | 40–400x（首頁峰值 444.6x） | ⚠️ 廠商自家 workload；獨立測試在合適任務上仍便宜約兩個數量級 |
| 輸入價格 | $0.042 / 百萬 token | ✅ 五個以上來源一致＋獨立量測吻合 |
| 輸出價格 | 免費（"too cheap to meter"） | ✅ 官網定價頁；但永續性未被證明——TypeSafe 自己說 "We can't prove it isn't subsidized" |
| 官方 side-by-side demo | Jev 0.114 s vs GPT-5.6 Terra 8.566 s | ⚠️ 廠商 demo；TypeSafe 自己加註短而密的輸入「對我們有利」 |
| 「不會幻覺」/ 0% 型別錯誤 | schema 保證 | ⚠️/❌ 格式保證 ≠ 答案正確；廠商註腳承認 0% 這個數字 "is not empirical"（非實證） |
| 工作流準確率 | 67.8% 綜合，落後 Opus 5（73.1%）、Sol（74.1%） | ⚠️ 這是**廠商自己的 dashboard**——老實的說法是「帕累托經濟學」，不是「最聰明」 |

**獨立測試數據點**（樣本小、單一任務，僅供參考方向）：
- Every（Mike Taylor）：37 份文件 × 21 個問題 = 777 個判斷，0.7 秒內、約 $0.0025；另一次測試中位數 0.35 s vs Claude Fable 5.1 的 8.83 s（約 25 倍快、約 1/580 成本）——但 Jev **漏掉了 7 個預埋缺陷中的 1 個**，對照組有抓到。
- Near Here（獨立工程師）：50 個案例的事件驗證，Jev 96% vs Mistral Small 4 的 84%、Gemini 3.5 Flash-Lite 的 86%；每 1,000 個決策 $0.043 vs Gemini 模型的 $2.496。
- Vercel（Pranit Sharma）：把 OpenAI Luna 5.6 的指令安全分類器換成 Jev → 快 5–18 倍，且更準。
- Bryo AI（CTO）：郵件分類 vs Gemini——Gemini 略準但貴 10–20 倍；Jev 的真實機率輸出更適合做自動化閾值。

## 4. 開發者如何使用：API、價格、權重、文件

- **狀態**：閉源權重、私有授權——只有雲端 API，沒有開源權重（社群 GitHub repo 都是 API 包裝或 demo）。
- **取得方式**：在 typesafe.ai 排 early-access waitlist（官方說正在盡快放行）；API key 從 `console.typesafe.ai` 拿；有網頁 Playground（需登入）可以免寫碼試玩。
- **API**：`POST https://api.typesafe.ai/v1/systemone`，`Authorization: Bearer $TYPESAFE_API_KEY`。文件：https://docs.typesafe.ai/
- **價格**：輸入 $0.042 / 百萬 token，輸出免費。本次研究**沒有在官方文件中找到明確的免費額度／試用 credits**——實際的試用路徑是 Playground ＋ waitlist。
- **其他接入**：Vercel AI Gateway（`typesafe-ai/jev`）、Vercel AI SDK（`@ai-sdk/typesafe-ai`）。
- **SDK**：官方 Python（`pip install typesafe-sdk`）、JS/TS（`npm install @typesafe-ai/sdk`）；Claude Code plugin；另有 Go、Rust、Ruby、Elixir、PHP、Swift、Scala、.NET 的社群 SDK（非官方）。
- **限制**（第三方回報，未經官方文件確認）：Choice 選項上限 255 個；context 約 32k–64k token（來源說法不一）；上線初期只吃文字。

## 5. LangChain 的「Jev-as-a-Judge」實驗（約 9/19–20 發布）

來源：https://www.langchain.com/blog/jev-agent-evals-langsmith（LangChain 宣布 9/22 週二跟 TypeSafe 合辦 livestream）

- **動機**：Agent 評測今天只有兩種——程式化 eval（便宜但狹窄）或 LLM-as-a-judge（靈活但慢、貴、不穩定）。LangChain 想知道 Jev 是不是第三種評測器。
- **方法**：用 Deep Agents 做了一個天氣 agent，把 5 個固定的天氣查詢執行結果存成 LangSmith dataset，讓每個評委評完全相同的行為；每個評委輸出兩個訊號（連續的 `quality` 分數、二元的 `does_pass`）；評委＝Jev、GPT-5.6 Luna、GPT-5.6 Terra、Claude Sonnet 4.6；**每個案例重複 100 次**；人類審查員按同一 rubric 標註作為 oracle。
- **結果**：
  - **準確率**（500 個重複的 pass/fail 判斷與人類 oracle 的一致率）：Jev **100%**，Terra 99.8%，Luna 96.4%，Claude 80.0%。
  - **穩定度**（quality 分數的每案例平均變異數）：Jev **0.0000149**——比 Luna 低 433 倍、比 Terra 低 913 倍、比 Claude 低 92 倍。
  - **成本**：這次實驗中 Jev **每次呼叫 $0.00035**。
- **LangChain 自己的警語**：「這個實驗無法告訴我們為什麼 Jev 的分數變異比較小……結果是觀察性的，不是它訓練目標導致低變異的證據。」「還需要看結果能不能複製到其他 agent 和 production workflow。」另外：便宜會放大錯誤——「一個穩定但錯誤的評委，會大規模產出壞的回饋」；人類審查和評委對齊仍然必要。
- **可複製性註記**：LLM 評委用供應商預設值跑（沒設 temperature/seed）；Jev 經 `langchain-typesafe==0.0.1a2`；文中有 GitHub repo 連結；**實驗 metadata 裡沒有 Jev 的服務版本**。
- **相關整合**：LangChain 同步出了實驗性 middleware——`TypeSafeClassifier`、用 Jev 做模型路由（便宜 vs 強模型二選一）、「auto mode guardrails」（危險 tool call 執行前擋下）。

## 6. 開發者分享的實際應用案例

- **廣告素材分析——40 秒拆 724 支廣告**：Matthew Berman 用 Jev 拆解 37 個品牌的 724 支即時廣告（hook、格式、offer、CTA、awareness stage、landing page 落差），**40 秒、花 $0.09**。
- **瀏覽器 Agent——下一步點哪個按鈕**：Gregor Zunic 把 Jev 接進 Browser Use 做「ultrafast」agent，每步重新產生 action space；**查機票 7 秒、花 $0.0039**；開源專案 jev-ultrafast 約 2.9k GitHub stars。
- **Coding agent 的 context 壓縮**：`jev-compactor`——用 Jev 判斷哪些訊息跟目標無關並丟掉，保留的**原文不動**（不做摘要），支援 Claude Code、Codex CLI、Gemini CLI 等；每次壓縮 0.3–0.6 秒、$0.0004–$0.0014，對比模型摘要的 1–60 秒 / $0.01–$0.15。
- **任務驗收／監控**：Almeida 的 pitch——用 Jev 監控 LLM agent 的 trace、偵測 jailbreak；LangChain 的 guardrail middleware（擋危險 tool call）。
- **官方 demo**：Doom 約每秒 10 個決策（約 $7/小時）；Wikiracing（每步數千個連結，用 255 選項 Choice＋兩階段篩選）。
- **其他社群作品**：1,500 封郵件一次分類；Every 的編輯檢查（1,709 個判斷、<$0.01、中位數 0.35 s）；3,282 篇貼文 × 8 個問題（$0.1282）；jev-trader（300 ms 一個 block）；jev-drone（2.5 Hz 控制）。
- **需求訊號**：上線時 API 一度被需求塞爆（TechCrunch）——代表熱度，不代表容量保證。

## 7. 社群反應與質疑

- **熱度**：發布推文一天 2,500 萬瀏覽／6.1 萬讚；Hacker News 475+ 則討論；幾天內社群 SDK 和 demo 大爆發。
- **最大的未解之問——校準（calibration）**：整個 RLCD 價值主張（「說 80% 就真的約 80% 對」）**目前零公開證據**——沒有論文、沒有 reliability curve、沒有 ECE 數字；文件承認校準是群體層級；HN 上關於 RLCD 目標和校準維持的兩個問題，TypeSafe 沒有回覆。
- **「不會幻覺」是語義遊戲**：把「型別安全」跟「真實」混為一談。形狀正確但答案錯誤完全可能；0% 數字非實證。Manj Chenna 的說法：機率是關於「你的問題」的事實，不是關於世界的事實；Ronacher：「它只是把幻覺問題稍微丟回給使用者」——使用者得自己決定 50% vs 95% 可以授權什麼行動。
- **行銷倍數是廠商峰值**：193.6x／444.6x 是拿貴的前沿推理模型在自家 workflow 上比的；獨立量測約 5–25x。TypeSafe 自己也說這些是「真實世界增益的高標」。
- **準確率落差**：自家 dashboard 上 Jev 綜合準確率落後前沿模型；Every 的測試裡 Jev 漏掉一個對照組有抓到的預埋缺陷。老實的定位是「便宜夠用」，不是「最強」。
- **早期風險**：零具名客戶、零營收揭露、waitlist  gating、無 SLA、架構不公開、定價永續性未被證明。
- **平衡一下**：TypeSafe 預先發布了自己的「懷疑論」章節、開源了 MIT 授權的 adapter 讓人重跑對比、公布逐案例的分歧分析——以一次發布來說，證據透明度算少見的高。

## 8. 中文媒體覆蓋

- **机器之心**：本次研究用搜尋沒有找到 jiqizhixin.com 的相關文章（註：你看到的是他們的小紅書帳號貼文，網站文章沒找到；搜尋無結果不代表絕對沒有，建議直接去他們站內確認）。
- 其他中文報導：
  - IT之家（經新浪科技，9/17）：事實性報導——$40M 種子輪、193.6x/444.6x 承認為上限值、Doom 約 10 q/s ≈ ¥47.1/小時。https://finance.sina.cn/tech/2026-09-17/detail-inisckrz4082879.d.html?vt=4
  - 動區 BlockTempo（繁體）：解釋 Jevons paradox 命名、Choice 255 上限、RLCD vs RLHF。https://www.blocktempo.com/jev-typesafe-ai-openai-system-one-model-diogo-almeida-explained/
  - AIPostHub（繁體，長文分析）：https://www.aiposthub.com/typesafe-ai-diogo-almeida-jev-system-one-model-rlcd-analysis/
  - ysk.hk（香港）：交叉驗證 TechCrunch＋官方 blog；提到 API 一度被塞爆。https://ysk.hk/blog/902/ChatGPT-%E7%99%BC%E6%98%8E%E8%80%85%E6%8E%A8%E9%9D%9E%E8%AA%9E%E8%A8%80%E6%A8%A1%E5%9E%8B-Jev%E7%B5%90%E6%A7%8B%E5%8C%96%E6%B1%BA%E7%AD%96%E5%85%8D%E5%B9%BB%E8%A6%BA%E6%88%90%E6%9C%AC%E9%99%8D%E5%85%A9%E5%80%8B%E6%95%B8%E9%87%8F%E7%B4%9A
  - CyberQ 賽博客（台灣）：Doom 數學（10 q/s = 36,000/hr ≈ $7）。https://cyberq.tw/2026/09/17/chatgpt-co-inventors-have-launched/
  - Whoops SEO（台灣，質疑視角）：「这些数字全部是 TypeSafe 的单方说法」；無公開 benchmark、「官方刻意不公布」。https://seo.whoops.com.tw/what-is-jev/
  - sns.style（简体，9/19）：https://sns.style/zh/news/2026/09/19/typesafe-ai-unveils-jev-former-openai-researcher-builds-non-autoregressive-syste-2

## 9. 未確認／待追蹤事項

1. 官方文件裡**沒找到明確的免費額度**——只有 waitlist＋Playground。
2. Context window 眾說紛紜（32k vs 64k），官方未確認。
3. 「基於開源權重 LLM」的懷疑未被證實；TypeSafe 拒絕透露架構。
4. RLCD 校準宣稱沒有任何獨立複現。
5. 定價永續性：TypeSafe 自己都說無法證明不是補貼價。
6. 机器之心網站是否有文章，建議直接站內搜尋確認。

## 10. 給開發者的務實結論

Jev 不是來取代 LLM 的，它是把「分類／打分／二選一」這類髒活從貴模型手裡拿走。用它的時機：**高頻、低單價、容錯的判斷任務**（路由、guardrail、eval、triage、context 清理）。不適合：需要解釋、需要高準確率、容錯低的場景——它的機率是「關於你的問題」的，不是關於世界的；50% 的答案你敢不敢執行，是你要決定的事。

想試的話：去 typesafe.ai 排 waitlist，或先玩 Playground；有 Vercel AI Gateway 的話走 `typesafe-ai/jev` 最快。LangChain 跟 TypeSafe 在 9/22（週二）有合辦 livestream，值得看。

---

## 資料來源

**官方（TypeSafe）**
- 發布 blog：https://typesafe.ai/blog/introducing-system-one-models-and-jev
- 文件：https://docs.typesafe.ai/ ；快速開始：https://docs.typesafe.ai/introduction/quickstart
- Workflow evals dashboard：https://evals.typesafe.ai

**外媒**
- Wikipedia "Jev (AI model)"：https://en.wikipedia.org/wiki/Jev_(AI_model)
- TechCrunch（9/18）：https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers/
- The Register（9/16）：https://www.theregister.com/ai-and-ml/2026/09/16/typesafe-ai-debuts-model-for-machines-that-plays-doom/5296711
- VCAOnline 新聞稿：https://www.vcaonline.com/news/2026091518/typesafe-ai-emerges-from-stealth-with-40m-in-funding-with-new-model-for-composable-ai/

**評測與技術分析**
- LangChain "Can Jev Be a Better Agent Evaluator?"：https://www.langchain.com/blog/jev-agent-evals-langsmith
- AgentPedia 宣稱 vs 證據對照：https://agentpedia.codes/blog/jev-system-one-models
- Manj Chenna 評論：https://manjchenna.com/essays/jev-typesafe-system-one-model

**社群／應用案例**
- Made with Jev：https://madewithjev.com/
- awesome-jev：https://github.com/kraayenjon/awesome-jev
- jev-compactor：https://github.com/edwardyen724-g/jev-compactor

**中文報導**：見第 8 節連結。

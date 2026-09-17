# JOVICHEER 網站平台選型：Firebase vs Cloudflare

- 調查日期：2026-09-16
- 專案：JOVICHEER（計畫產品化的商用網站）
- 背景：站長為長期居住美國的台灣人，一人公司；網站面向全世界、以美國為主，**放棄中國市場**；站長已有 20+ 個 Firebase hosting 專案經驗，熟悉 Firebase Auth / Firestore；Cloudflare 則主要用過反向代理等進階應用。

## 一、核心問題與結論

| 問題 | 結論 |
|---|---|
| Cloudflare 上能做 Google 登入嗎？ | 可以，不麻煩。Firebase Auth 不綁 Firebase Hosting。 |
| Google / Firebase 有免費 LLM 額度嗎？ | 有。Gemini Developer API 免費 tier 不用綁信用卡，給的是旗艦模型。 |
| JOVICHEER 該選哪家？ | **Firebase 開局**，Cloudflare 當作未來的逃生門。 |

## 二、Google 登入：Cloudflare hosting + Firebase Auth 可行

Firebase Auth 與 Firebase Hosting 是脫鉤的，Auth 本來就能跑在任何 hosting 上（含 Cloudflare Pages）：

1. Firebase console 開啟 Google sign-in provider。
2. 把 Cloudflare 上的域名加到 Firebase console → Authentication → Settings → **Authorized domains**。
3. 部署後若遇到 `auth/unauthorized-domain` 錯誤，就是域名沒加進去，加完即通（有 Cloudflare Pages + SvelteKit 的實作教學驗證過此流程）。

**唯一要注意的坑**：`signInWithRedirect` 在阻擋第三方儲存的瀏覽器上，非 Firebase Hosting 會有問題。Firebase 官方文件建議非 Firebase hosting 改用 `signInWithPopup`，其餘程式碼不變。

## 三、免費 AI 額度比較

| | Cloudflare Workers AI | Gemini Developer API（經 Firebase AI Logic） |
|---|---|---|
| 免費額度 | 約 10,000 requests / 天 | 免費 input + output tokens（限速 RPM/TPM/RPD） |
| 模型等級 | Llama 3.1 8B 等較小開源模型 | Gemini 3 等旗艦模型 |
| 綁卡需求 | 不用 | 不用（Spark 免費 plan 即可） |
| 代價 | — | 免費版的 prompt 內容會被 Google 拿去改善產品 |
| 超量後 | $0.011 / 1,000 neurons | 切付費 tier（Blaze） |

Firebase AI Logic 的 SDK 本身免費：走 Gemini Developer API 後端就直接吃免費額度；只有走 Vertex AI 後端才需要 Blaze 付費方案。

**混合架構選項**：站放 Cloudflare，用 Cloudflare Worker 當後端去呼叫 Gemini API，key 藏在 Worker 環境變數不暴露給瀏覽器 —— 兩邊免費額度都吃得到。但對 JOVICHEER 而言沒必要（見第五節）。

## 四、為什麼當初用 Cloudflare 的理由已經不存在

當初選 Cloudflare 是為了教廈門的朋友部署（Firebase / Google 體系在中國有問題）。但：

- Cloudflare 免費版在中國**一樣不能用**，中國節點需要企業版 + ICP 備案。
- JOVICHEER 面向美國 / 全世界，放棄中國市場 —— 中國可達性不再是選型因素。

## 五、選型分析：為什麼 JOVICHEER 適合 Firebase

按重要性排序：

1. **肌肉記憶**：20+ 個 Firebase 專案經驗。一人公司最大的瓶頸是開發速度，不是 infra 單價，用熟的工具就是最好的工具。
2. **All-in-one**：Auth、Firestore、Hosting、Functions、AI Logic、Crashlytics 全在同一個 console、同一個 SDK。一個人維護，這是決定性的；Cloudflare 要自己把 Workers、D1、R2、Access 拼起來。
3. **Auth 免費額度大**：email / Google 等登入免費到 50K MAU，前期碰不到天花板（電話簡訊驗證另計費）。
4. **AI 免費額度給的是旗艦模型**，Cloudflare 免費版給的是小模型。

## 六、Scale 的誠實比較

Cloudflare 在 scale 的「單價」上確實比較漂亮：R2 零 egress 費用、D1 讀取便宜、Workers $5/月含 1,000 萬次請求。

Firebase 的 scale 痛點是 Firestore 按次計費：讀 $0.03 / 10 萬次、寫 $0.09 / 10 萬次。有實測估算 150K MAU 的 SaaS 在 Firebase 上約 $145/月，其中 Firestore 讀取是大頭。

但關鍵判斷：**$145/月 是有 150K 月活之後才要煩惱的事**，那時已有營收。一人公司的死因永遠是做太慢、做不出來，不是 infra 貴了幾十塊。先求活下來、跑出 PMF，真的貴到痛再把熱路徑搬去 Cloudflare —— 不要預先優化。

## 七、建議

- **JOVICHEER 用 Firebase 開局**：Hosting + Auth（Google 登入）+ Firestore + Firebase AI Logic（Gemini），全吃 Google 體系。
- **Cloudflare 保留為逃生門**：將來流量大到 Firestore / 頻寬費用有感時，再把熱路徑（靜態資源、快取、邊緣運算）搬過去。
- **不為中國市場做任何架構妥協**。

## 備註

- 數字來源為 2026-09 查證的官方文件與第三方實測；Firebase 2026-09-01 起 Remote Config 改為用量計費（每天 10 萬次 fetch 免費），JOVICHEER 若用 Remote Config 做 feature flag 需注意此變化。
- 本報告為選型建議，實際計費以各平台官方 pricing page 為準。

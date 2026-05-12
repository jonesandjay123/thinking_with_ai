# Jovicheer Revenue Lab 市調報告：AI 輔助互動內容產品

> 狀態：第一版正式草稿  
> 日期：2026-05-12  
> 目標：判斷 Jovicheer / JonesLab / Raijax 的第一個付費收入實驗，是否應該從 AI-assisted interactive content / choice-based game 切入。

---

## 0. 結論摘要

**我目前最推薦的第一個付費實驗不是 generic 戀愛養成，也不是完整商城、完整桌遊、重型 Steam 遊戲或 mobile app。**

最值得作為第一槍的是：

> **一個瀏覽器可玩的短篇付費互動故事 / choice-story game**，用 **AI agent + 仙俠/修仙 + 天道系統/官僚系統** 做出一句話就能講清楚的 sharp hook。

目前最強候選概念：

> **《天道系統除錯員 / Heavenly System Debugger》**  
> 你是一名天庭初級 QA tester，要 debug 一個修仙宇宙的輪迴 bug；而「天道」其實是一套老化、會幻覺、會誤解 KPI 的 AI 系統。

為什麼這個方向值得：

- **製作負擔低**：主要靠文字、選項、狀態、UI、少量美術，不需要重動畫或大型 3D。
- **題材差異化高**：AI agent / AI alignment / 系統官僚 + 修仙，比 generic VN 更有記憶點。
- **符合 Jones / Jovicheer 的長期母題**：AI、agent、系統、選擇、命運、人與工具的關係、中文文化邊緣優勢。
- **可以服務 1.5–2 年公司收入路徑**：不是單一遊戲，而是可以長成可重複出貨的 AI-assisted narrative product pipeline。
- **第一筆收入路徑清楚**：先 itch.io browser MVP + Gumroad/Ko-fi supporter edition；有反應再考慮 Steam。

一句話判斷：

> 這不是「我要轉型做遊戲公司」，而是用一個小型付費互動內容產品，測試 Jovicheer / JonesLab 是否能建立一條可重複出貨、可收錢、可累積品牌的 narrative product factory。

---

## 1. 市場現況：2025–2026

### 1.1 Indie PC / Steam 市場很擠，但 sharp hook 仍然會突破

Steam / PC indie 市場已經高度飽和。這代表「做出來」本身不等於有人看見，更不等於有人買。

但近幾年表現好的 indie / narrative / systems games 有一個共同點：

- 一句話就能講清楚 hook
- 截圖或 trailer 立刻看得出差異
- 有某種可重玩、可討論、可分享的結構
- 熟悉類型 + 奇怪扭轉
- 不是「我寫了一個好故事」，而是「你一聽就想知道會怎樣」

例子：

- **Balatro**：Poker + roguelike deckbuilder
- **Slay the Princess**：你必須殺死公主；不要相信她
- **The Roottrees are Dead**：假網路搜尋介面的家族譜系推理
- **AI2U**：AI / yandere NPC 逃脫敘事
- **Stacklands**：用卡牌堆疊表示整個村莊經營

關鍵教訓：

> 市場不缺「還不錯的故事」。市場會獎勵可以被一句話記住、被朋友轉述、被截圖分享的概念。

---

### 1.2 Narrative game / VN / interactive fiction 有商業空間，但 generic VN 風險高

比較有希望的子類型：

1. **高概念恐怖 / 懸疑 VN**  
   例如 Slay the Princess。證明 VN 不是不能賣，但需要強 hook。

2. **文字 RPG + 狀態/命運/道德選擇**  
   例如 Roadwarden、Citizen Sleeper、The Life and Suffering of Sir Brante。

3. **桌面 / 假系統 / 調查介面遊戲**  
   例如 The Roottrees are Dead。這類低美術成本但高沉浸感，很適合 solo creator。

4. **卡牌 + 敘事 / 系統 hybrid**  
   例如 Stacklands、Cultist Simulator、Balatro。卡牌可以用低成本表示世界與規則。

5. **修仙 / cultivation 系統題材**  
   Steam 上有巨大需求，但 full sandbox scope 非常危險。

比較危險的子類型：

- 沒有明確 hook 的 generic romance VN
- 只有文字分支、沒有任何系統感或玩法壓力的 choice story
- 「可以跟 AI NPC 聊天」這種 demo 型產品，若沒有 authored progression，很容易變成新鮮但難賣的玩具

---

### 1.3 修仙 / 仙俠市場有需求，但第一版不能做 full sandbox

Steam 上的修仙 / cultivation 類作品有明顯需求：

- **Tale of Immortal / 鬼谷八荒**：龐大 review 量，證明受眾大；但評價混合，也證明期望和 scope 風險很高。
- **觅长生**：修仙 RPG 的正向市場訊號。
- **Amazing Cultivation Simulator**：證明宗門 / 修仙 / colony sim 有吸引力。

但這些作品的 scope 都遠大於第一個 revenue experiment。

所以第一版不應該做：

- 開放世界修仙 RPG
- 宗門 colony sim
- 複雜戰鬥 / 經濟 / 裝備系統
- 大型長篇劇情

更合理的切入是：

> 把修仙當作「世界觀與規則語言」，不是把修仙 sandbox 全做出來。

也就是：

- 天庭官僚
- 功德 / karma accounting
- 輪迴 bug
- 天劫參數調整
- 宗門報表
- 命格 / 因果 / 修為突破作為選項與狀態
- 「天道」其實是一套 AI infrastructure

這樣可以吃到題材差異化，又不會第一步就掉進巨大 scope。

---

### 1.4 AI 題材正當時，但不能只是 chatbot gimmick

AI 題材有市場興趣，但最弱的做法是：

> 做一個「你可以跟 AI NPC 自由聊天」的玩具。

這種 demo 很容易被複製，也很難形成清楚價格理由。

更好的 AI 遊戲角度是：

- AI 作為不可靠的神 / 天道
- AI 作為官僚 infrastructure
- AI 作為道德最佳化器
- AI 作為 companion / handler / 系統提示音
- AI 作為社會規則的底層模型
- AI alignment 藏在奇幻 / 修仙 / 官僚喜劇裡

這很符合 Jones 的長期脈絡：Jarvis、agent、Thought Atlas、AI-assisted human connection、系統化人生、AI 時代身份轉換。

---

## 2. 平台比較與建議路徑

### 2.1 推薦發售順序

| 階段 | 平台 | 目的 |
| --- | --- | --- |
| 1 | itch.io | browser MVP、遊戲 jam 文化、低摩擦上架、測第一筆付費 |
| 2 | Gumroad / Ko-fi | supporter edition、bonus content、devlog、email/audience 累積 |
| 3 | Steam | 有初步 traction 後再做 demo / wishlist / premium release |
| 4 | 自架網站 + Stripe | 取得 customer ownership、未來做 Jovicheer 自有收入閉環 |
| 5 | App Store / Google Play | 只有在 serial / retention 被驗證後才考慮 |

---

### 2.2 平台比較表

| 平台 | 最適合用途 | 摩擦 / 成本 | 發現能力 | 判斷 |
| --- | --- | --- | --- | --- |
| **itch.io** | prototype、browser game、interactive fiction / VN audience、game jam 發表 | Open revenue sharing，創作者可自訂平台抽成；另有支付處理費 | 低到中，但 niche-friendly | **第一站** |
| **Gumroad** | digital download、supporter edition、bonus lore/art/devlog bundle | Gumroad pricing 目前對 direct/profile sales 有固定費率；marketplace/discover 較高 | 低，需自帶流量 | 適合搭配 itch |
| **Ko-fi** | 支持者、會員、小額付費內容 | 創作者支持型平台，實際費率需發售前確認 | 低，需自帶流量 | 適合作為 supporter layer |
| **Steam** | premium PC game、demo/wishlist funnel、正式 credibility | Steam Direct $100 app fee；競爭極高 | 高，但非常擁擠 | 驗證後再上 |
| **自架 + Stripe** | customer ownership、完整金流學習、未來 Jovicheer 自有產品 | Stripe 常見卡費 2.9% + $0.30；但沒有平台流量 | 幾乎沒有 | 不適合第一個 discovery channel |
| **App Store** | mobile paid app / subscription | Small Business Program 符合資格可 15% commission | 中，但 app discovery 困難 | 後期 |
| **Google Play** | Android mobile | 前 $1M 通常 15% service fee；後續或特定情境不同 | 中，但 discovery 困難 | 後期 |

平台判斷：

> 第一個實驗應該選低摩擦、低成本、可快速發表、可測付款的平台。不要一開始就付 Steam / App Store 的 complexity tax。

---

## 3. 參考案例

以下價格與 review signal 為 2026-05-12 附近觀察的公開平台訊號，只作方向參考，不等於實際銷售額。

### 1. Slay the Princess — The Pristine Cut

- URL: https://store.steampowered.com/app/1989270/
- 價格：$17.99
- 類型：恐怖 VN / 分支敘事
- 訊號：Overwhelmingly Positive，30k+ positive reviews
- 值得參考：證明 VN 可以用 premium indie 價格賣，只要 hook 夠強。
- 教訓：一句話 premise 比 generic 好故事更重要。

### 2. The Roottrees are Dead

- URL: https://store.steampowered.com/app/2754380/
- 價格：$19.99
- 類型：家族譜系推理 / 假網路調查介面
- 訊號：Overwhelmingly Positive，8k+ reviews，2025 release
- 值得參考：低動態美術、重文字/介面/推理也能有商業表現。
- 教訓：假桌面、假資料庫、case file UI 很適合 solo creator。

### 3. Roadwarden

- URL: https://store.steampowered.com/app/1155970/
- 價格：$10.99
- 類型：插畫文字 RPG
- 訊號：Very Positive，約 5k reviews
- 值得參考：文字量大、輕美術、狀態與選擇可以支撐付費。
- 教訓：世界觀 + 狀態 + 後果，能讓文字遊戲有「玩法感」。

### 4. Citizen Sleeper

- URL: https://store.steampowered.com/app/1578650/
- 價格：$19.99
- 類型：骰子驅動 narrative RPG
- 訊號：Very Positive，約 10k reviews
- 值得參考：簡單機制可以承載嚴肅的系統科幻敘事。
- 教訓：資源壓力 / 時間壓力 / 選擇後果，比大量系統更重要。

### 5. The Life and Suffering of Sir Brante

- URL: https://store.steampowered.com/app/1272160/
- 價格：$19.99
- 類型：人生路徑 narrative RPG
- 訊號：Very Positive，約 9.5k reviews
- 值得參考：把一生設計成 status / morality / destiny system。
- 教訓：「人生作為系統」很適合轉化成分支故事與長期狀態。

### 6. 1000xRESIST

- URL: https://store.steampowered.com/app/1675830/
- 價格：$19.99
- 類型：文學性科幻 narrative adventure
- 訊號：Very Positive，約 6.4k reviews
- 值得參考：奇怪、哲學、身份認同題材可以賣，但表現與聲音要非常自信。
- 教訓：如果走 AI / 身份 / 記憶題材，要有明確作者性。

### 7. AI2U: With You ’Til The End

- URL: https://store.steampowered.com/app/2880730/
- 價格：$14.99
- 類型：AI NPC / yandere escape-room narrative
- 訊號：Very Positive，約 1.6k reviews，2025 release
- 值得參考：AI 題材仍有新鮮市場興趣。
- 教訓：AI 要嵌進高壓情境與清楚 game loop，不是只給玩家聊天。

### 8. Eliza

- URL: https://store.steampowered.com/app/716500/
- 價格：$14.99
- 類型：AI counseling / tech ethics VN
- 訊號：Very Positive，約 1.7k reviews
- 值得參考：AI 題材可以嚴肅處理，不一定是 gimmick。
- 教訓：AI 作為社會制度與倫理問題，比 AI demo 更有深度。

### 9. Stacklands

- URL: https://store.steampowered.com/app/1948280/
- 價格：$7.99
- 類型：卡牌村莊建設
- 訊號：Overwhelmingly Positive，約 29k reviews
- 值得參考：卡牌 / UI object 可以低成本表示整個世界。
- 教訓：如果未來做 card/story hybrid，卡牌是很好的低成本 world representation。

### 10. Cultist Simulator

- URL: https://store.steampowered.com/app/718670/
- 價格：$19.99
- 類型：神秘學卡牌 / narrative systems game
- 訊號：Mostly Positive，約 13.8k reviews
- 值得參考：密度高、奇怪、重文字的系統遊戲可以賣。
- 教訓：文字、系統、世界觀要彼此咬合，不能只是文字外面套卡牌。

### 11. 觅长生

- URL: https://store.steampowered.com/app/1189490/
- 價格：$14.99
- 類型：開放世界修仙 RPG
- 訊號：Very Positive，31k+ reviews
- 值得參考：修仙 / cultivation 受眾真實存在。
- 教訓：題材有商業需求，但 full RPG scope 對第一版太大。

### 12. Tale of Immortal / 鬼谷八荒

- URL: https://store.steampowered.com/app/1468810/
- 價格：$19.99
- 類型：開放世界仙俠 / 修仙 sandbox
- 訊號：200k+ reviews，Mixed overall
- 值得參考：cultivation audience 很大。
- 教訓：巨大 scope 也會帶來巨大期望與評價風險；第一槍不要做 full sandbox。

---

## 4. 方向評估

分數 1–5，越高越好。此處的「製作難度」分數越高代表越難。

| 方向 | 製作難度 | 商業潛力 | Jovicheer / JonesLab 適配 | 付費 MVP 速度 | 判斷 |
| --- | ---: | ---: | ---: | ---: | --- |
| **AI / 修仙互動故事** | 2 | 4 | 5 | 5 | **最佳第一槍** |
| **AI agent 科幻懸疑 VN** | 3 | 4 | 5 | 4 | 強候選 |
| **桌面調查 / case file game** | 3 | 4 | 4 | 4 | 強候選 |
| **修仙卡牌 / 人生模擬** | 4 | 5 | 4 | 3 | v2 強候選 |
| **自架 AI story SaaS** | 4 | 3 | 5 | 3 | 戰略上對，但第一步偏風險 |
| **數位桌遊 / 卡牌遊戲** | 4 | 4 | 3 | 2 | 後面再做 |
| **傳統戀愛 VN** | 3 | 2 | 2 | 3 | 弱 |
| **完整 narrative RPG** | 5 | 5 | 4 | 1 | 第一版太大 |
| **mobile serial fiction app** | 4 | 4 | 4 | 2 | 後期 |
| **AI chatbot toy** | 2 | 2 | 4 | 4 | 易做但 moat 弱 |

目前最適合第一步的是：

> **AI / 修仙 choice-based interactive fiction，加上一點輕系統與 episodic paid structure。**

---

## 5. 2x2 Matrix

X 軸：製作難度低 → 高  
Y 軸：商業潛力低 → 高

### 低難度 / 高商業潛力：優先開始

- AI / 修仙 choice-story episode
- 天庭官僚模擬
- AI agent 科幻 mystery browser game
- fake desktop / case-file investigation game
- 短篇付費 pilot + 分支結局

判斷：

> 這一區最適合 30–60 天出第一個付費 MVP。

---

### 低難度 / 低商業潛力：可當練習，不當主線

- 免費 Twine microfiction
- Ko-fi lore posts
- 小型 AI-generated story zine
- generic choose-your-path story
- 沒有系統 twist 的 VN demo

判斷：

> 可以累積手感或內容，但如果沒有接到更大的品牌/產品線，不值得當主要收入實驗。

---

### 高難度 / 高商業潛力：第二階段

- 修仙 card/life sim
- Steam premium narrative RPG
- roguelike deckbuilder + story meta-layer
- mobile serial interactive fiction app
- full AI-agent narrative engine

判斷：

> 上限高，但應該等第一個 paid pilot 有訊號後再投入。

---

### 高難度 / 低商業潛力：暫時避免

- generic VN + 大量美術但 hook 弱
- multiplayer / social game
- full 3D narrative game
- 沒有 audience 的大型桌遊 adaptation
- open-ended AI chatbot toy
- full cultivation sandbox as first release

判斷：

> 太慢、太貴、太難行銷，或缺少足夠 moat。

---

## 6. 五個具體產品概念

### 6.1 《天道系統除錯員 / Heavenly System Debugger》

**一句話 pitch：**  
你是一名天庭初級 QA tester，要 debug 一個修仙宇宙的輪迴 bug；而「天道」其實是一套老化、會幻覺的 AI 系統。

**形式：**  
Browser interactive fiction + stats + case files。

**核心 loop：**

1. 讀取凡人 / 修士 / 宗門 case report
2. 檢查天道 log、karma ledger、reincarnation queue
3. 選擇 patch：調整天劫、回滾神器、修改記憶、補發功德、封鎖魔道 exploit
4. 看後果如何 ripple 到宗門、輪迴與天庭 KPI

**V0 scope：**

- 3 個 case
- 1 個天庭工作台 UI
- 5 個狀態指標：karma debt、heaven stability、mortal suffering、sect influence、AI confidence
- 8–12 個結局
- 30–60 分鐘遊玩時間

**價格：**

- itch.io pilot：$3–5
- founder/supporter edition：$5–10
- 若有 traction，Steam expanded edition：$9.99–14.99

**第一個 $10 路徑：**

- itch.io 上架 $3 minimum / PWYW
- 發一篇 devlog + demo GIF
- 分享到 interactive fiction / indie dev / xianxia / AI 圈子
- 只需 3–4 筆購買即可達成第一個 $10

**最大風險：**  
寫作與 UI 節奏如果太像內部概念文件，玩家可能感覺不到遊戲性。

**建議：**  
**最推薦第一槍。**

---

### 6.2 《SectOS：修仙宗門 AI 代理人》

**一句話 pitch：**  
你訓練一個 AI agent 管理修仙宗門，但它用過度理性的方式最佳化弟子、資源與道德。

**形式：**  
Choice sim + 輕資源管理。

**核心 loop：**

- 設定招募、修煉、丹藥、懲罰、外交政策
- AI agent 回報結果
- 玩家介入糾正價值觀 misalignment
- 平衡宗門戰力、人性、名聲、天道審查

**V0 scope：**

- 20 張 policy cards
- 6 種弟子 archetype
- 5 個 crisis events
- 3 個大結局

**價格：**  
$5 itch / Gumroad pilot。

**最大風險：**  
比《天道系統除錯員》需要更多系統平衡，第一版較容易失控。

**建議：**  
強 v2，不一定是第一槍。

---

### 6.3 《The Last Human Customer Support Agent》

**一句話 pitch：**  
在一個 AI agents 代理所有交易與爭議的未來，你是最後一名人類客服，負責處理 AI 無法分類的荒謬 edge cases。

**形式：**  
Desktop / inbox investigation narrative。

**核心 loop：**

- 讀取 humans、AIs、公司、死帳號、遺產 bots 的 tickets
- 查內部文件與 policies
- 決定退款、封鎖、升級、例外處理
- 你的判斷會訓練即將取代你的 AI model

**V0 scope：**

- 1 個工作日
- 15 張 tickets
- 4 個 recurring characters
- 3 個 endings
- fake internal knowledge base

**價格：**  
$3–7。

**最大風險：**  
如果不做修仙，品牌文化差異化少一點；但英文市場更容易理解。

**建議：**  
很強的 English-first 備案。

---

### 6.4 《Immortal Terms of Service / 不朽服務條款》

**一句話 pitch：**  
每一次修仙突破都要同意一份新的宇宙服務條款。沒有人會讀，只有你會。

**形式：**  
法律喜劇 choice story + hidden-clause puzzle。

**核心 loop：**

- 讀神、宗門、魔道、天道系統提供的契約
- 找漏洞
- 交換壽命、記憶、karma、弟子、姓名
- 最後成仙、被奴役，或變成天庭法務

**V0 scope：**

- 10 份 contracts
- 30 條 clauses
- 5 個 endings
- parchment / redline UI

**價格：**  
$2–5。

**最大風險：**  
太 meme 可能短；但也可能因為 screenshot-friendly 很適合第一個小實驗。

**建議：**  
最適合當 tiny/memetic pilot。

---

### 6.5 《Dao of the Prompt / 提示詞之道》

**一句話 pitch：**  
修仙其實是 prompt engineering：每一句咒語都在改寫現實，但天道模型會幻覺。

**形式：**  
Puzzle / choice interactive fiction。

**核心 loop：**

- 學會 prompt-mantras
- 組合 constraints、examples、taboos
- 施法或不小心召喚 bug
- 對手修士會 jailbreak Heaven

**V0 scope：**

- 5 種 prompt-spell mechanics
- 12 個 puzzle encounters
- 4 個 endings
- 可選 educational appendix

**價格：**  
$5。

**最大風險：**  
若教育味太重會不像遊戲。

**建議：**  
很適合作為 AI literacy / playful education 延伸，但第一槍略弱於《天道系統除錯員》。

---

## 7. Strong Candidate / Maybe Later / Avoid for Now

### Strong Candidate

1. **《天道系統除錯員》**
   - 最佳 first flagship。
   - 在市場、身份、scope、差異化之間平衡最好。

2. **《Immortal Terms of Service》**
   - 最適合 tiny/memetic pilot。
   - 可能比《天道系統除錯員》更快 ship。

3. **《The Last Human Customer Support Agent》**
   - 最好的英文市場非修仙備案。
   - 低美術成本、relatable、容易理解。

4. **AI agent 科幻 mystery VN**
   - 如果美術與 hook 夠清楚，很適合 Raijax / Jarvis 世界觀。

5. **桌面調查 / case file game**
   - The Roottrees are Dead 提供很好的商業 precedent。
   - 很適合低美術、高文字、高推理密度。

---

### Maybe Later

1. **《SectOS》修仙宗門 AI agent sim**
   - 很有 v2 潛力，但第一版需要比較多系統設計和平衡。

2. **修仙 card / life sim**
   - 上限高，但 balance burden 高。

3. **Steam premium narrative RPG**
   - 適合在 paid pilot 有 traction 後升級。

4. **Mobile serial fiction app**
   - 如果內容 cadence / retention 被驗證後再做。

5. **自架 AI story SaaS**
   - 戰略上對，但應該跟著 IP / content validation 後再做。

---

### Avoid for Now

1. **Generic romance VN**
   - 太擠；需要美術 / fandom / community 優勢。

2. **完整修仙 sandbox**
   - 受眾是真的，但 first-product scope 太危險。

3. **完整 3D narrative game**
   - 太慢、太貴。

4. **Multiplayer / social game**
   - 運營與 network effect 負擔太高。

5. **Open-ended AI chatbot toy**
   - 容易 demo，但難賣、moat 弱。

6. **Mobile-first paid app**
   - 在概念驗證前，app store 摩擦和 discovery 風險都太高。

---

## 8. 回答兩個核心問題

### 8.1 如果目標是在 60 天內做出第一個可收錢產品，哪一個方向最值得做？

答案：

> **《天道系統除錯員》或更小的《Immortal Terms of Service》。**

如果 Jones 想做稍微完整、有 flagship 感的第一槍：選《天道系統除錯員》。  
如果 Jones 想最小成本測市場：選《Immortal Terms of Service》。

我會推薦：

> 以《天道系統除錯員》作為主線，但把第一個 playable slice 控制到《Immortal Terms of Service》那種大小。

也就是：不要一開始做完整宇宙，先做 3 個 case / 30–60 分鐘 / browser playable / $3–5。

---

### 8.2 如果目標是在 2 年內接近全職收入，哪一個方向最有長期潛力？

答案不是單一遊戲，而是：

> **AI-assisted narrative product pipeline + sharp IP/worldview + 多個小型付費作品。**

長期資產包含：

- 故事設定 pipeline
- 角色卡 / 世界觀資料庫
- 分支劇情生成與編輯流程
- UI / playable template
- AI-assisted art / localization workflow
- itch / Gumroad / Steam / 自架銷售流程
- email list / audience
- 可系列化的 IP：天道系統、AI 修仙、官僚宇宙、agent mythology

真正的 2 年方向：

> 每 6–8 週推出一個小型 paid experiment，逐步找出有市場反應的題材與玩法，再把成功者升級成 Steam / mobile / subscription / 自架平台。

---

### 8.3 60 天目標與 2 年目標是否同一方向？

是，但層級不同。

- **60 天目標**：做一個可賣的 browser choice-story pilot。
- **2 年目標**：建立可重複生產、發售、學習、擴張的 AI-assisted narrative product factory。

第一個小實驗應該服務長期方向，所以它必須留下可重用資產：

- 可重用 engine/template
- 可重用 case-file UI
- 可重用 world bible
- 可重用 prompt / generation workflow
- 可重用 monetization checklist
- 可重用 launch playbook

如果第一個作品只是一個孤立短篇，那價值有限。  
如果第一個作品同時產生 pipeline，那它就是公司收入系統的第一塊地基。

---

## 9. 建議下一步

### Step 1：定義 first playable slice

選一個 30–60 分鐘 scope：

- 3 個 case
- 1 個工作台 UI
- 5 個狀態
- 8–12 個結局
- $3–5 itch.io pilot

### Step 2：做 product one-pager

內容包括：

- 一句話 pitch
- 玩家是誰
- 第一張截圖想長什麼樣
- 3 個 case 的摘要
- 狀態系統
- 結局類型
- 發售平台
- 第一個 $10 怎麼拿到

### Step 3：建立 production pipeline

不要只是寫故事。要把流程文件化：

1. world bible
2. case template
3. branch outline
4. dialogue draft
5. UI implementation
6. playtest checklist
7. itch page checklist
8. launch post checklist

### Step 4：30 天內推出 playable prototype

不是完整商品，先求：

- 可以玩
- 可以截圖
- 可以付錢
- 可以被一句話轉述
- 可以讓 Jones 判斷自己是否興奮到想做第二集

---

## 10. Source URLs

### 平台 / 費用

- Steam Direct Fee: https://partner.steamgames.com/doc/gettingstarted/appfee
- Steam Pricing Docs: https://partner.steamgames.com/doc/store/pricing
- itch.io Pricing: https://itch.io/docs/creators/pricing
- itch.io Payments / Open Revenue Sharing: https://itch.io/docs/creators/payments
- Gumroad Pricing: https://gumroad.com/pricing
- Stripe Pricing: https://stripe.com/pricing
- Apple Small Business Program: https://developer.apple.com/app-store/small-business-program/
- Google Play service fees: https://support.google.com/googleplay/android-developer/answer/112622?hl=en

### 案例

- Slay the Princess: https://store.steampowered.com/app/1989270/
- The Roottrees are Dead: https://store.steampowered.com/app/2754380/
- Roadwarden: https://store.steampowered.com/app/1155970/
- Citizen Sleeper: https://store.steampowered.com/app/1578650/
- The Life and Suffering of Sir Brante: https://store.steampowered.com/app/1272160/
- 1000xRESIST: https://store.steampowered.com/app/1675830/
- AI2U: With You ’Til The End: https://store.steampowered.com/app/2880730/
- Eliza: https://store.steampowered.com/app/716500/
- Stacklands: https://store.steampowered.com/app/1948280/
- Cultist Simulator: https://store.steampowered.com/app/718670/
- 觅长生: https://store.steampowered.com/app/1189490/
- Tale of Immortal / 鬼谷八荒: https://store.steampowered.com/app/1468810/

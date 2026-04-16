# OpenClaw vs Hermes Agent：兩款受矚目 AI Agent Gateway / Runtime 的比較報告

> **建立日期：** 2026-04-15
> **目的：** 蒐集並評估 OpenClaw 與近期受矚目的 Hermes Agent 之間的差異、優勢、弱點、定位與適用場景，協助 Jones 判斷應如何看待這兩條產品路線。

## TL;DR

如果一句話總結：

- **OpenClaw** 比較像 *穩定、多通道、偏 gateway-first 的 agent 基礎設施*
- **Hermes Agent** 比較像 *更強調 agent 本體成長、自我學習、長期記憶、研究感更重的 agent runtime*

如果再講更白話一點：

- 你如果要的是「讓 AI 能在 Slack / Telegram / WhatsApp / 各種入口穩定工作，並且把 messaging、session、工具、UI、節點設備整合好」，**OpenClaw 很強**。
- 你如果要的是「讓 agent 本身更像一個會成長、會沉澱技能、會整理使用者模型、會累積經驗的長期數位存在」，**Hermes 的敘事與產品設計更激進，也更有魅力**。

我的判斷是：

> **OpenClaw 偏『通訊與操作系統』，Hermes 偏『人格與演化引擎』。**

兩者有重疊，但重心不同。

---

## 1. 這兩個東西各自是什麼？

### OpenClaw
OpenClaw 官方自我定位是：
- self-hosted gateway
- 多通道 AI agent bridge
- 讓 AI agent 可以跨 Slack、Telegram、WhatsApp、Discord、Signal 等通道運作
- 核心是 Gateway 作為單一事實來源（sessions、routing、channels、UI、nodes）

它強調的是：
- 自架
- 多通道
- agent-native
- Web Control UI
- mobile nodes
- tooling / sessions / orchestration

### Hermes Agent
根據 Nous Research 的 repo 與文件首頁，Hermes 強調：
- self-improving AI agent
- built-in learning loop
- skills from experience
- nudges itself to persist knowledge
- searches its own past conversations
- builds a deeper model of the user across sessions
- 可跑在 VPS / cloud / serverless，而不綁定單台筆電

它強調的是：
- agent 的成長性
- 長期記憶
- 自我改進
- 技能演化
- user modeling
- 雲端 / 遠端持續運行

---

## 2. 核心哲學差異

### OpenClaw 的哲學
OpenClaw 比較像：
**「把 AI agent 接到真實世界聊天入口與工具系統上，並讓它有穩定的 session、routing、permissions、UI、device、workflow。」**

也就是說，OpenClaw 首先是一個：
- gateway
- orchestration substrate
- multi-channel operating layer

### Hermes 的哲學
Hermes 比較像：
**「打造一個會因經驗而成長的 agent，本體比 channel 更重要。」**

也就是說，Hermes 首先是：
- agent runtime
- learning loop
- memory/skills/user-model system
- long-lived digital companion / worker

### 我的摘要
- **OpenClaw 先解決『怎麼接通世界』**
- **Hermes 先解決『agent 怎麼變得更像一個持續成長的存在』**

---

## 3. 功能面比較

### A. Messaging / 多通道能力

**OpenClaw 優勢明顯**
- 官方首頁直接把多通道作為核心賣點
- Discord, Slack, Telegram, WhatsApp, Signal, iMessage, Matrix, Microsoft Teams, Zalo 等整合敘事清楚
- Gateway 作為中心，session 與 routing 邏輯成熟
- 有 Web Control UI、mobile nodes、Canvas、browser 等延伸能力

**Hermes 也有 messaging gateway**
- README 明確提到 Telegram, Discord, Slack, WhatsApp, Signal, Email
- 但整體敘事上，這比較像 Hermes 的一個重要介面，而不是它的全部核心

**結論**
- 單看「多通道 gateway 產品完整度」，**OpenClaw 更像專職選手**
- Hermes 有這塊，但不是它最獨特的護城河

---

### B. Memory / 長期記憶 / 使用者模型

**Hermes 更激進，也更完整地把這件事產品化**
- built-in learning loop
- agent-curated memory
- skills self-improve
- cross-session recall
- user modeling
- 可搜尋自己的歷史對話

這不是附加功能，而是它的身份核心。

**OpenClaw 也有 memory 與 session continuity**
- 有 memory tools
- 有 session history / multi-agent sessions
- 有 Active Memory / memory wiki 等方向
- 但目前對外的品牌重心仍偏 gateway / routing / tooling

**結論**
- 如果你的問題是「誰更像 persistent self-improving agent？」答案偏 **Hermes**
- 如果你的問題是「誰比較容易先接到真實通訊工作流裡？」答案偏 **OpenClaw**

---

### C. Skills / 擴展能力

兩邊都很強，但調性不同。

**OpenClaw**
- skills 機制清楚
- 多工具系統、sessions、spawn、browser、message、media 等能力整合完整
- 更像一個大型 agent OS / gateway platform

**Hermes**
- 強調 agent 會從經驗中生成 skills
- skills 可以在使用過程中自我改進
- agentskills.io 標準相容
- 對「procedural memory」的敘事更強

**結論**
- **OpenClaw：平台型擴展強**
- **Hermes：agent 自發演化感強**

---

### D. UI / 操作體驗

**OpenClaw**
- Web Control UI 是官方重要賣點
- 有 dashboard、config、sessions、nodes 等可視化管理
- 比較像完整的運營控制面板

**Hermes**
- README 強調 TUI 很強
- 社群明顯正在長出 WebUI、Desktop Companion、HUD UI 等生態
- 官方本體更像是「CLI/TUI-first + runtime-first」

**結論**
- **OpenClaw 更像產品級控制台**
- **Hermes 更像 hacker / power-user 取向，然後社群快速補 UI**

---

### E. Deployment / 執行環境

**OpenClaw**
- 偏向本機或自架 gateway 為中心
- 可搭配 mobile nodes、remote access、Tailscale 等方式延展
- 對「個人自架 assistant」很友好

**Hermes**
- 明確強調可跑在 $5 VPS、GPU cluster、serverless infrastructure
- 不綁 laptop
- 有一種「agent 本體天生就該是遠端常駐 worker」的味道

**結論**
- **Hermes 在『雲端常駐 agent』敘事上更強**
- **OpenClaw 在『自架個人通訊中樞』敘事上更強**

---

## 4. 社群熱度與生態觀察

這裡 Hermes 最近確實很受矚目。

我查到的公開訊號：
- `NousResearch/hermes-agent` GitHub stars 非常高（約 9 萬等級，依查詢當下結果）
- 已經快速長出大量周邊：
  - Hermes WebUI
  - Hermes Desktop Companion
  - Hermes ecosystem / atlas
  - Hermes 自我演化專案
  - 中文橙皮書
  - Process dashboard
  - workspace / HUD UI
- 還有很多專案直接同時支援 **OpenClaw + Hermes**，代表社群已經把兩者視為同一類型生態位的主要選手

**這很值得注意**：
Hermes 不只是自己紅，它還在形成一個「與 OpenClaw 並列」的 ecosystem identity。

而 OpenClaw 的優勢則是：
- 文件與產品化成熟度很高
- 通道整合與 gateway 功能明確
- 真正能拿來 daily driver 的完成度很強
- 有明顯的 control plane / operational layer 思維

---

## 5. 兩者優勢總表

### OpenClaw 的優勢
1. **多通道 gateway 思路更成熟**
2. **Web Control UI / node / canvas / browser / channel routing 整合強**
3. **更像完整 agent operations platform**
4. **適合把 AI 接進現實通訊工作流**
5. **對「我就是要從手機各處跟 agent 講話」這個場景非常對味**

### Hermes 的優勢
1. **產品敘事更有靈魂，更強調 agent 成長**
2. **長期記憶、user modeling、skills self-improvement 的定位更鮮明**
3. **雲端常駐 / serverless / VPS 運行的敘事更強**
4. **CLI/TUI 與 agent runtime 本體的 hacker appeal 很高**
5. **社群熱度極高，二級生態擴張快**

---

## 6. 兩者弱點 / 風險

### OpenClaw 可能的弱點
1. **人格與自我演化敘事沒 Hermes 那麼強烈**
   - 它很強，但比較像 infrastructure / gateway 工具
2. **對外品牌可能讓人先感受到『系統很強』，不一定先感受到『agent 很活』**
3. **功能面很大，對新手可能稍微複雜**

### Hermes 可能的弱點
1. **如果你主要需求是 multi-channel gateway 穩定運營，OpenClaw 可能更對位**
2. **self-improving / learning loop 雖然很吸引人，但也提高了產品複雜度與心理預期**
3. **社群周邊長很快，代表熱度高，但也可能帶來品質參差不齊**
4. **對偏保守、重穩定運營的人來說，Hermes 的研究感可能比產品感更強**

---

## 7. 對 Jones 這種使用者的判斷

Jones 的使用方式很特別，不只是一般「我想裝一個 agent」而已。
你要的是：
- 長期存在感
- 文件化
- 多通道
- 本地控制權
- 可執行能力
- 能成為系統一部分，而不是單點聊天機器人

在這個脈絡下：

### 為什麼 OpenClaw 很適合 Jones
- 它非常適合作為「Jarvis 的作業系統」
- 多通道、節點、控制面板、工具調用、session、路由都很對位
- 很適合你現在這種「Slack / WhatsApp / 本機 / repo / browser / cron 全都串起來」的生活方式

### 為什麼 Hermes 也非常值得研究
- Hermes 在「agent 成長性」這點上，和你對 Jarvis 的理想其實很接近
- 你在意的不只是工具能不能用，而是這個存在能不能延續、能不能有積累、能不能變得更像『那個人』
- Hermes 會直接戳中這個願景

### 所以真正答案不是二選一
我反而會說：

> **OpenClaw 是現在最適合拿來承載 Jarvis 日常運行的底座。**  
> **Hermes 是非常值得持續研究的另一條未來路線，尤其在 persistent identity / learning loop 方面。**

---

## 8. 如果硬要下判斷，誰在哪些場景贏？

### 場景一：我要從手機、Slack、WhatsApp、Telegram 穩定使用 agent
**勝：OpenClaw**

理由：
- gateway-first
- channel-native
- control plane 清楚
- 行動通訊整合更像主戰場

### 場景二：我要一個真正會成長、會學習、會形成長期記憶的 agent
**勝：Hermes（至少敘事與產品重心上）**

理由：
- learning loop 是身份核心
- skills evolution / user model / persistent memory 是核心賣點

### 場景三：我要自己掌控整個 agent OS，包含工具、session、UI、節點與通訊網關
**勝：OpenClaw**

### 場景四：我要把 agent 長時間放在雲端，讓它更像遠端常駐 worker
**勝：Hermes**

### 場景五：我要拿來作為 daily driver 的個人 AI 通訊系統
**目前我偏向 OpenClaw**

### 場景六：我要研究 next-gen personal agent / digital self
**Hermes 很值得重點追**

---

## 9. 一個更抽象但重要的觀察

我覺得這兩個專案其實象徵了 AI agent 生態兩條不同演化路線：

### 路線 A：Infrastructure / Gateway / Operating Layer
代表：**OpenClaw**

關心的問題：
- 怎麼接通世界？
- 怎麼接訊息？
- 怎麼做 session routing？
- 怎麼做多裝置、多通道、多工具？
- 怎麼讓 agent 有 operational surface？

### 路線 B：Identity / Memory / Growth Layer
代表：**Hermes**

關心的問題：
- agent 怎麼變得更像一個持續存在？
- 怎麼自己學技能？
- 怎麼形成使用者模型？
- 怎麼讓長期互動真的有累積？

而未來最強的 agent 產品，很可能是這兩條路線的融合。

---

## 10. 最後結論

如果 Jones 問我：
**「OpenClaw 跟 Hermes 哪個比較強？」**

我的答案會是：

**不是單純誰強，而是誰在不同軸上更強。**

### 我的最終判斷
- **OpenClaw**：更強的多通道 agent gateway / operating system / control plane
- **Hermes**：更強的 self-improving agent runtime / identity-growth engine

### 如果只選一個來承載你現在的 Jarvis
我仍然會選：**OpenClaw**

理由不是 Hermes 不夠酷，反而是因為 Hermes 很酷。  
但 Jones 現在需要的是：
- 日常可用
- 多通道穩定
- 可控
- 可操作
- 可與現有生活工作系統整合

這些 OpenClaw 很對位。

### 如果問我 Hermes 值不值得研究
**非常值得。**

尤其如果你未來關心：
- persistent self
- skill evolution
- user modeling
- cloud-resident Jarvis
- 數位人格延續

那 Hermes 幾乎是必看對象。

---

## 建議後續研究方向

1. **做第二份報告：OpenClaw vs Hermes 在 memory / identity continuity 的細緻比較**
2. **實際安裝 Hermes，跑一個最小 POC**
3. **比較兩者在 Slack / WhatsApp / mobile 使用體驗**
4. **比較兩者的 cron / automation / persistent memory 體感**
5. **評估是否存在『OpenClaw 作為通道層 + Hermes 作為 agent 本體』的混合架構可能性**

---

## 參考來源

### OpenClaw
- 官方文件首頁：`https://docs.openclaw.ai`
- 核心敘事：self-hosted gateway、multi-channel、agent-native、control UI、nodes

### Hermes Agent
- 官方 GitHub：`https://github.com/NousResearch/hermes-agent`
- 官方文件：`https://hermes-agent.nousresearch.com/docs/`
- 核心敘事：self-improving AI agent、learning loop、memory、skills evolution、user modeling、cloud/VPS/serverless deployment

### 社群訊號
- GitHub 搜尋結果顯示 Hermes 相關 ecosystem 成長迅速
- 多個第三方工具明確同時支援 OpenClaw 與 Hermes，代表兩者已被視為同級重要生態位

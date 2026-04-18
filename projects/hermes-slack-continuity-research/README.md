# Hermes Slack 使用體驗調查：同一 thread 內的記憶割裂，是否是普遍問題？

> **建立日期：** 2026-04-18  
> **目的：** 根據 Hermes 官方文件、GitHub issue / PR 與 Slack 官方 threading 說明，確認 Jones 在 Slack 上遇到的「同一篇 thread 內上下文延續不穩、像開新 session、記憶割裂」是否只是個人設定問題，還是已有公開可見的共通現象。

---

## TL;DR

先講結論：

1. **這不是你獨有的體感，也不像只是你把設定弄壞。**  
   Hermes 官方 repo 裡已經有多個公開 issue / PR，直接碰到 Slack 的 thread context、session scope、mention 規則、重啟續接、slash command session drift 等問題。

2. **公開證據足夠支持「Slack 上確實存在 continuity / thread-context 的真實產品層問題」。**  
   最直接的證據包括：
   - thread root message 沒被放進 context
   - 同一 thread 的 follow-up reply 可能因 session key mismatch 被忽略
   - gateway restart 後無法自動接回原 Slack thread conversation
   - slash command 與後續訊息落在不同 session scope，造成 model/provider 看似切換成功、實際沒接上
   - 某些 Slack 發送路徑會直接忽略 thread context，掉回 top-level message

3. **但我目前沒有找到足夠強的公開證據，能說「所有使用者都在抱怨 Hermes Slack 很割裂」。**  
   更準確的說法是：**官方公開 repo 已經顯示這是一串真實存在、而且最近還在持續修補中的 Slack integration / session-boundary 問題。**

4. **所以網路上會推崇 Hermes，不代表你遇到的是幻覺。**  
   比較像是：Hermes 被推崇的重點多半是 memory / skills / agent loop / extensibility；但如果你的主工作流核心是「Slack thread 裡像同一個人一路接著做事」，那它現在確實還有一段距離。

---

## 一、這次調查看了什麼

### 我這次有查的來源
- Hermes 官方 README
- Hermes 官方 docs site（Messaging Gateway、Persistent Memory、Slack setup 頁）
- `NousResearch/hermes-agent` 公開 GitHub issues / PRs
- Slack 官方 thread 說明

### 我這次刻意沒做的事
- 沒有去讀私有 Slack 歷史
- 沒有去掃社群私密聊天、Discord 私群、付費社群內容
- 沒有把單一 issue 誇大成「社群共識」

所以這份報告的定位是：

> **以公開、可驗證來源為基礎，判斷這是不是一個真實存在的產品面問題。**

---

## 二、官方對外宣稱：Hermes 的賣點確實包含 memory、cross-session continuity、multi-platform messaging

Hermes 官方 README 明確把這些當賣點：

- 「built-in learning loop」
- 「searches its own past conversations」
- 「builds a deepening model of who you are across sessions」
- 「Telegram, Discord, Slack, WhatsApp, Signal, and CLI — all from a single gateway process」
- 「cross-platform conversation continuity」

官方 README：  
<https://github.com/NousResearch/hermes-agent/blob/main/README.md>

官方 docs 也明講：

- Messaging Gateway：每個平台 adapter 會把訊息 routing 到 **per-chat session store**
- Session Persistence：session 會跨訊息持續存在，agent 會記住 conversation context
- Persistent Memory：memory 是在 **session start** 時以 frozen snapshot 注入，而不是每條訊息即時重建

文件連結：
- Messaging Gateway：<https://hermes-agent.nousresearch.com/docs/user-guide/messaging>
- Persistent Memory：<https://hermes-agent.nousresearch.com/docs/user-guide/features/memory>
- Slack setup：<https://hermes-agent.nousresearch.com/docs/user-guide/messaging/slack>

### 這段宣稱對你的 case 很重要

因為這代表你的期待不是亂想出來的。  
Hermes 自己的產品敘事，本來就讓人合理期待：

- 在 Slack 裡應該有相對穩定的 session continuity
- memory / session / thread 不應該太常錯位
- 同一條對話不應該頻繁表現出像「重新開機」一樣的割裂感

---

## 三、最直接的公開證據：Hermes 官方 repo 已有多個 Slack continuity 相關 issue

下面我把證據分級：

- **A 級**：直接支持「Slack thread continuity 確實有問題」
- **B 級**：支持「Slack session / routing / scope 還在修，會影響 continuity 體感」
- **C 級**：提供系統背景，幫助理解為什麼會覺得割裂

---

## 四、A 級證據：直接支持你遇到的「同一 thread 內容接不住」

### 1) Issue #2950 — Slack thread parent message missing from conversation context

- 連結：<https://github.com/NousResearch/hermes-agent/issues/2950>
- 建立時間：2026-03-25
- 標題：`[Bug]: Slack thread parent message missing from conversation context`

issue 內容直接描述：

- 當 Hermes 在 Slack thread 內回應時
- **啟動這個 thread 的第一則 parent/root message 沒有被放進 conversation context**
- agent 只能看到後續 thread replies，看不到最一開始的核心需求

這幾乎就是你不舒服感的最典型來源之一：

> UI 看起來在同一篇 thread，但 Hermes 腦中其實沒有完整看到這條 thread 的開頭。

### 2) PR #2953 — include thread parent message in conversation context

- 連結：<https://github.com/NousResearch/hermes-agent/pull/2953>
- 標題：`fix(slack): include thread parent message in conversation context`

這個 PR 直接對應 #2950，並明講：

- 原本 parent/root message 缺失
- 修法是透過 Slack `conversations.replies` 把 thread history 抓回來
- 並加上 regression tests

重點不是它有沒有最後 merge，而是：

> **官方 repo 裡已經有人把這問題具體定義、實作修補，表示這不是你個人的錯覺。**

### 3) Issue #5816 — Slack thread replies without @mention ignored despite active session

- 連結：<https://github.com/NousResearch/hermes-agent/issues/5816>
- 建立時間：2026-04-07
- 標題：`Bug: Slack thread replies without @mention ignored despite active session`

這個 issue 指出：

- bot 明明已經在 thread 裡有 active session
- 但 follow-up reply 若沒有再次 @mention
- 因為 session key 建法 mismatch，訊息可能被直接忽略

這會造成一種非常糟的體感：

- 使用者以為「既然都在同一 thread 裡了，你應該知道我在接著講」
- 但 Hermes 實際上可能根本沒把這則 follow-up 當成同一段 conversation

### 4) Issue #7304 — Gateway restart should resume the originating Slack thread conversation

- 連結：<https://github.com/NousResearch/hermes-agent/issues/7304>
- 建立時間：2026-04-10
- 標題：`Gateway restart should resume the originating Slack thread conversation`

這個 issue 的原始描述非常直接：

- 在 Slack thread 裡觸發 gateway restart 後
- agent 不會自己 pick up 原本那條 conversation
- 需要 hook 才能在重啟完成後接回原 thread

這再次證明：

> **Slack thread continuity 並不是 Hermes 已經天然處理好的東西，而是需要額外邏輯維持。**

---

## 五、B 級證據：顯示 Slack session scope / routing 還在持續修，足以導致割裂體感

### 5) Issue #9394 — approval buttons and standalone sends ignore thread context

- 連結：<https://github.com/NousResearch/hermes-agent/issues/9394>
- 建立時間：2026-04-14
- 標題：`Slack: approval buttons and standalone sends ignore thread context`

這個 issue 指出兩個 bug：

- 某些 Slack 發送路徑沒有 `thread_id`
- approval button 的 callback 沒有攜帶 `thread_ts`

結果是：

- 訊息會掉出原 thread
- 變成 top-level message
- thread context 被破壞

這雖然不完全等於「記憶失憶」，但對使用體感來說是一樣的：

> **對話看起來不再沿著同一條脈絡走。**

### 6) Issue #10688 + PR #10875 — Slack slash commands can drift out of conversation scope

- Issue：<https://github.com/NousResearch/hermes-agent/issues/10688>
- PR：<https://github.com/NousResearch/hermes-agent/pull/10875>

#10688 說的是 `/model` 在 Slack 裡不可靠，會出現：

- 看起來 model/provider 切換了
- 但後續回合仍落在舊 provider / 舊 session 行為上

#10875 的摘要更直白：

- slash commands 被塞進 synthetic DM session
- 但後續 Slack replies 是依 real channel/thread session 在跑
- 因此產生 **session drift**

這對你目前在 Slack 上的感受很重要，因為它說明：

> **不是只有 memory retrieval 會讓人感覺斷裂，session scope 自己就可能漂移。**

### 7) Issue #6345 — bot refuses to read channel / thread history beyond current message

- 連結：<https://github.com/NousResearch/hermes-agent/issues/6345>
- 建立時間：2026-04-09
- 標題：`Feature: Slack gateway should expose conversations.history as an agent tool`

這個 issue 明講：

- Slack bot 在目前設計下，會拒絕讀 channel / thread 歷史
- 即便 underlying capability 其實存在
- system prompt 裡的平台說明甚至明確告訴 agent：只能讀目前收到的訊息

這代表如果你期待的是：

- 「同一帖子的先前內容你應該自己知道」
- 或「你至少可以去翻 thread 歷史補 context」

那在目前設計下，**這個期待並沒有完全被產品實作兌現。**

---

## 六、C 級證據：說明 Hermes 的 Slack continuity 邏輯本來就不是單純天然成立

### 8) PR #7466 — unified thread context hook in BasePlatformAdapter

- 連結：<https://github.com/NousResearch/hermes-agent/pull/7466>
- 標題：`feat(gateway): unified thread context hook in BasePlatformAdapter`

這個 PR 的重點是：

- 為平台 adapter 新增統一的 `fetch_thread_context()` / `has_active_session_for_event()` hook
- 讓 bot 在「第一次進入既有 thread」時，可以抓 thread / conversation history

這件事的含義非常清楚：

> **thread context 不是 UI 自然送進來的，而是平台 adapter 額外抓、額外拼起來的。**

也就是說，只要這個拼接鏈哪裡少了一段，使用者就會立刻感覺到斷裂。

### 9) PR #7084 — remove DM thread session seeding to prevent cross-thread contamination

- 連結：<https://github.com/NousResearch/hermes-agent/pull/7084>
- 合併時間：2026-04-10
- 標題：`fix(gateway): remove DM thread session seeding to prevent cross-thread contamination`

這個 PR 顯示另一個方向的 continuity 問題：

- 不是「上下文接不到」
- 而是「不相關上下文被錯誤帶進來」

PR 描述提到：

- parent DM transcript 被整包複製進新的 thread session
- 多個 thread 之間出現 context bleed / contamination

這剛好也對應你的體感之一：

> **Hermes 有時不是完全忘記，而是抓到別條線。**

---

## 七、Slack 官方 threading 模型，本來就容易跟 AI session 模型錯位

Slack 官方對 thread 的定義是：

- thread 是一則 parent message 底下的集中討論串
- 用來避免污染主頻道
- UI 上看起來像一個完整主題

Slack 官方文件：  
<https://slack.com/help/articles/115000769927-Use-threads-to-organize-discussions-in-channels>

但 Hermes docs 自己的 session 模型則是：

- 每個平台 adapter 把訊息 routing 到 **per-chat session store**
- memory 是 **session start 時注入的 frozen snapshot**
- 部分能力要靠 session search / hooks / adapter context fetching 才能補回

所以這裡的根本張力是：

- **Slack thread 是 UI grouping**
- **Hermes session 是 runtime context unit**
- **Hermes memory 是跨 session 的摘要層**

這三層如果沒有被嚴格對齊，就會出現你現在感受到的現象：

- 同一篇 thread 但不像同一個腦袋
- 看似記得你，卻接不上你正在說的事
- 有時接不到，有時又接錯條線

---

## 八、所以這是不是「大家都在回報」？

### 可以肯定的部分

可以肯定的是：

1. **不是只有你。**  
   至少 Hermes 官方 repo 已有多個公開 issue / PR，直接碰到 Slack thread context / session continuity / routing scope。

2. **不是純粹你設定錯。**  
   因為這些問題裡有不少已經被描述成 adapter-level、session-key-level、thread-context-level 的產品缺陷。

3. **這不是很舊的歷史包袱而已。**  
   很多 issue / PR 都是 2026-03 到 2026-04，很近期，而且還在演進中。

### 不能過度宣稱的部分

我目前**不能負責任地說**：

- 「所有使用者都覺得 Hermes Slack 很糟」
- 「社群整體都認為 Hermes 不適合 Slack」
- 「Hermes 完全不值得用」

因為公開可驗證資料更像是在說：

> **Hermes 在 Slack continuity 這塊確實存在一串真實的、可重現的、持續修補中的問題；但 Hermes 被推崇的理由很多並不在這一塊。**

---

## 九、為什麼網路上還是會推崇 Hermes？

根據官方 README / docs 的主敘事，我的判斷是：

大家推崇 Hermes，多半是因為它在這些地方確實有吸引力：

- built-in learning loop
- 長期記憶 / user model / session search
- skills as procedural memory
- agent 會自我整理與累積知識
- runtime / tooling / extensibility 很強
- 可以跑在 VPS / cloud / multi-platform gateway

但這些優點，**不等於 Slack thread UX 已經成熟到像一般人期待的聊天連續性。**

所以比較準確的理解應該是：

- **Hermes 強，不代表每個 interaction surface 都成熟。**
- **Slack 剛好是它目前比較容易暴露 session/thread 邊界問題的地方。**

---

## 十、我對你這次體驗的判斷

如果只根據這次公開調查，我的判斷是：

> **你遇到的現象不是幻想，也不是單純使用錯誤，而是你用真實高強度工作流，把 Hermes 在 Slack thread / session / memory 對齊不良的弱點測出來了。**

更具體地說：

- 你期待的是 **thread continuity**
- Hermes 有時提供的是 **session continuity**
- 有時又退回成 **memory retrieval + partial reconstruction**

這三者一旦沒有完全對齊，使用者就會感覺：

- 「明明是同一篇，為什麼你像忘了？」
- 「明明昨天才講過，今天卻抓錯重點？」
- 「明明說會越用越聰明，但怎麼像越聊越碎？」

你的 frustration 是合理的。

---

## 十一、對 Jones 最實用的結論

### 如果你的核心工作流是：
- 在 Slack thread 裡長時間 debug
- 希望像單一對話一樣自然延續
- 希望 agent 對同一條討論有很強的連貫感

那目前比較務實的結論是：

1. **不要把「Slack thread = 穩定工作 session」當成已成立前提。**
2. **目前公開證據支持：Slack continuity 仍是 Hermes 的脆弱區。**
3. **Hermes 仍可用，但比較適合把 Slack 當入口，而不是當最核心、最依賴連續性的主工作面。**
4. **如果你最需要的是穩定連續的工作流，OpenClaw 或更單線性的 chat/session 介面，主觀體感可能真的會更順。**

---

## 十二、來源清單

### 官方文件 / 官方頁面
- Hermes README  
  <https://github.com/NousResearch/hermes-agent/blob/main/README.md>
- Messaging Gateway  
  <https://hermes-agent.nousresearch.com/docs/user-guide/messaging>
- Persistent Memory  
  <https://hermes-agent.nousresearch.com/docs/user-guide/features/memory>
- Slack Setup  
  <https://hermes-agent.nousresearch.com/docs/user-guide/messaging/slack>
- Slack threads official help  
  <https://slack.com/help/articles/115000769927-Use-threads-to-organize-discussions-in-channels>

### GitHub issues / PRs
- #2950 — Slack thread parent message missing from conversation context  
  <https://github.com/NousResearch/hermes-agent/issues/2950>
- #2953 — include thread parent message in conversation context  
  <https://github.com/NousResearch/hermes-agent/pull/2953>
- #5816 — thread replies without @mention ignored despite active session  
  <https://github.com/NousResearch/hermes-agent/issues/5816>
- #7304 — gateway restart should resume the originating Slack thread conversation  
  <https://github.com/NousResearch/hermes-agent/issues/7304>
- #9394 — approval buttons and standalone sends ignore thread context  
  <https://github.com/NousResearch/hermes-agent/issues/9394>
- #10688 — `/model` does not switch provider or model reliably in Slack gateway  
  <https://github.com/NousResearch/hermes-agent/issues/10688>
- #10875 — keep slash commands in their Slack conversation scope  
  <https://github.com/NousResearch/hermes-agent/pull/10875>
- #6345 — Slack gateway should expose conversations.history as an agent tool  
  <https://github.com/NousResearch/hermes-agent/issues/6345>
- #7466 — unified thread context hook in BasePlatformAdapter  
  <https://github.com/NousResearch/hermes-agent/pull/7466>
- #7084 — remove DM thread session seeding to prevent cross-thread contamination  
  <https://github.com/NousResearch/hermes-agent/pull/7084>

---

## 十三、下一步建議

如果你要，我下一步最有價值的是二選一：

1. **做一份工程向 follow-up**：把這些 issue 對應成「Jones 這台 Hermes 應該先檢查哪些 config / behavior / patch point」  
2. **做一份 workflow 向建議**：直接幫你設計「Slack / Hermes / OpenClaw / ChatGPT 怎麼分工，才不會再被 continuity 問題拖死」

我會建議先做第 1 個，因為它最接近你現在卡住的真問題。

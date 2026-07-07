# Claude Fable 5：Pro plan 是否仍可使用、7/7 計費變更調查

日期：2026-07-07  
查詢時間：美東 16:55 後  
問題：Jones 記得 Anthropic 曾說 7/7 起 Fable 5 會改成額外收費，但 7/7 當天 Claude 介面看起來沒有明顯變化。要確認 Pro plan 現在到底還能不能用 Fable 5，以及什麼時候會產生額外費用。

## 一句話結論

截至本次查詢，Claude Fable 5 仍可在 Pro / Max / Team / Enterprise 等符合資格的訂閱方案中使用，而且 Anthropic Help Center 現在明確寫著 promotional access 期間是 2026-07-01 到 2026-07-12 23:59:59 PT。

所以 Jones 今天 7/7 在介面上沒看到大變化，合理。原本「7/7 之後改 usage credits」這個印象不是空穴來風，而是來自 Anthropic 6/30 redeploy blog 的原始說法；但 Anthropic 後來的 Help Center 專門頁已把 promotional period 寫到 7/12。

務實判斷：

1. Pro plan 現在仍可用 Fable 5。
2. 在 promotional period 內，最多可用 50% weekly subscription limit 在 Fable 5 上，無額外費用。
3. 用完 Fable 5 的 50% weekly limit 後，若繼續用 Fable 5，會改用 usage credits，也就是額外按 standard API rates 計費。
4. 若不想額外付費，用到 limit 後改用其他模型即可。
5. Usage credits 官方說會有明確提示 / confirmation；不是在完全無警告下默默收費。

## 目前官方資訊

### 1. Anthropic 的 Claude Fable 5 產品頁

Anthropic 的 Fable 5 產品頁顯示：

- 2026-07-01：Claude Fable 5 access restored / rolling out。
- Fable 5 available to Pro, Max, Team, and Enterprise users。
- Developers 可透過 Claude Platform、marketplaces、AWS、Google Cloud、Microsoft Foundry 使用。
- API 價格是 $10 / 1M input tokens、$50 / 1M output tokens。
- Fable 5 在 cyber / biology safeguard 觸發時，很多 query 會 fallback 到 Opus 4.8；被 reroute 時不會收 Fable prices。
- Using Fable requires 30-day data retention for safety monitoring。

這頁沒有說 Pro plan 已經不能用 Fable 5。相反，它明確把 Pro 列在可用方案中，並連到 Help Center promotional access 頁。

### 2. Help Center：Claude Fable 5 promotional access

這是本次最關鍵的官方頁。

Anthropic Help Center 說：

- For a limited time, users can use Claude Fable 5 at no extra cost as part of subscription plan。
- During the promotional period, up to 50% of weekly subscription limits can be used on Fable 5 at no extra cost。
- After that, users can keep using Fable 5 with usage credits, or switch to another model and keep working within remaining limits。
- Nothing to claim or activate。
- Fable 5 draws from regular weekly usage limit and uses it faster than other Claude models。
- Promotional period starts July 1, 2026 and ends July 12, 2026 at 11:59:59 PM PT。

這一頁直接回答 Jones 的問題：今天 7/7，Pro plan 仍應該可以用 Fable 5；如果還在 weekly limit / Fable 5 子限制內，不需要額外付費。

### 3. Usage credits 官方說明

Anthropic 的 usage credits 頁說：

- Usage credits 讓 Pro、Max 5x、Max 20x 使用者在達到 included usage limits 後，可以改成 pay-as-you-go 繼續用 Claude。
- Usage credits 是按 standard API rates 計費。
- 可在 Settings > Usage 開啟 / 關閉。
- 會有 spending controls。
- 官方 FAQ 說 approaching / reaching limit 時會有 clear notification，並確認你會用 usage credits 繼續。

因此「不敢亂用」的安全策略可以改成：

- 到 Settings > Usage 確認 usage credits 是否啟用。
- 若不想有任何額外花費，把 usage credits 關掉。
- 若 usage credits 關掉，用完 included limit 後就只能等 reset / 換模型，而不是被額外收費。

### 4. Release notes

Claude Help Center release notes 只寫：

- 2026-06-12：Fable 5 / Mythos 5 access suspended。
- 2026-07-01：Fable 5 / Mythos 5 access restored，詳情見 Anthropic statement。

Release notes 沒有把 7/7 或 7/12 的計費細節寫在主頁；真正細節在 promotional access Help Center 頁。

### 5. API pricing docs

Claude API pricing docs 顯示 Fable 5 的 standard API rates：

- Base input：$10 / MTok
- 5m cache writes：$12.50 / MTok
- 1h cache writes：$20 / MTok
- Cache hits / refreshes：$1 / MTok
- Output：$50 / MTok

這些價格主要影響：

- Claude API
- Claude Platform / AWS / Bedrock / Google Cloud / Microsoft Foundry
- subscription plan 用完 included limit 後的 usage credits

它不等於「你只要在 Pro plan 介面點 Fable 5 就立刻按 token 收費」。在 promotional access 還有效且未超過 Fable 5 子限制時，官方說是 no extra cost。

## 7/7 說法從哪裡來？

Jones 的印象是合理的。Anthropic 6/30 的官方 redeploy blog 原文寫：

- Fable 5 will be available starting July 1 globally。
- For Pro, Max, Team, and select Enterprise plans, Fable 5 will be included for up to 50% of weekly usage limits through July 7。
- After that it will be available via usage credits。

也就是說，原公告確實把「included through July 7」當作關鍵日期。

但後來 Anthropic Help Center 的專門頁把 promotional period 寫成 July 1 到 July 12 23:59:59 PT。這通常代表兩種可能：

1. Anthropic 延長 promotional period，但沒有同步改舊 blog 文案。
2. Blog 的「through July 7」是初始過渡說法，Help Center 後續補充了更正式的當前政策。

在產品使用判斷上，Help Center 專門頁比舊 blog 段落更應該當作現行操作依據。它寫得更細，也直接回答 billing / usage limit / usage credits 的問題。

## 7/7 當天到底會不會被額外收費？

我的判斷：正常情況下不會，只要你還在 promotional access 的 Fable 5 子限制內。

會進入額外收費的情境是：

1. 你已經用 Fable 5 用到最多 50% weekly subscription limit。
2. 你仍然選擇繼續用 Fable 5。
3. Usage credits 已啟用。
4. Claude 介面提示後，你確認繼續。

如果 usage credits 沒啟用，或你在 limit 後改用其他模型，理論上就不會進入 Fable 5 的額外計費。

## Pro plan 目前可不可以用 Fable 5？

可以。官方頁明確列出 Pro users 可用 Fable 5 promotional access。

但要注意幾個限制：

- 不是整個 Pro weekly limit 都能拿去用 Fable 5；最多是 50% weekly limit。
- Fable 5 使用量會消耗得比其他 Claude models 快。
- 如果你已經先用其他模型消耗了很多 weekly limit，剩下可用的 Fable 5 空間也會變少。
- Fable 5 的 cyber / bio safeguard 可能把某些對話 fallback 到 Opus 4.8。
- 某些企業或 managed settings 可能限制 Claude Code 裡的 model availability。

## 為什麼介面看起來沒變？

可能原因：

1. Anthropic 已把 no-extra-cost promotional access 延到 7/12，所以 7/7 當天不該看到硬切換。
2. 如果你還沒碰到 Fable 5 的 50% weekly limit，就不會看到 usage credits flow。
3. Usage credits 是 limit 後才出現的延伸用量機制，不是 model picker 上一定會出現的大 banner。
4. 如果 usage credits 沒開，你可能只會在用完 limit 後被限制或被引導切模型，而不是先看到收費變化。

## 社群與新聞訊號

英文新聞搜尋在 7/7 前後仍可看到「Fable 5 subscription included period ends / shifts to usage credits」這類標題，這些多半沿用 6/30 blog 的 July 7 說法。這解釋了為什麼使用者會以為今天就開始全部改 pay-as-you-go。

但我沒有找到比 Anthropic Help Center 更可靠、且可直接驗證「7/12 promotional access」的論壇討論。Bing / Google 的 Reddit、HN、Anthropic community 搜尋結果污染嚴重，很多被同名遊戲 Fable 或其他網站混入；因此報告不把論壇當主要依據。

目前最可靠的層級排序是：

1. Anthropic Help Center promotional access：現行計費與使用判斷依據。
2. Anthropic Fable product page / pricing docs / release notes：可用性與定價背景。
3. Anthropic 6/30 redeploy blog：解釋 7/7 印象來源，但可能已被 Help Center 更新覆蓋。
4. 新聞與論壇：反映市場困惑，但不適合當最終判斷。

## Jones 現在該怎麼用

### 如果你只是怕「一點 Fable 5 就被額外收費」

目前看來不用怕。只要還在 Fable 5 的 included promotional quota 內，不會額外收費。

### 如果你想完全避免額外費用

建議：

1. 打開 Claude 的 Settings > Usage。
2. 確認 usage credits 是否啟用。
3. 如果不想任何 pay-as-you-go，把 usage credits 關掉。
4. 用 Fable 5 時注意 usage meter；接近 limit 後改用 Opus / Sonnet / 其他模型。

### 如果你願意少量付費換不中斷

可以開 usage credits，但要設 spend limit。Fable 5 standard rate 很高，$10 input / $50 output per MTok；長 context、Claude Code、Cowork、Research mode、Projects/files 都可能消耗很快。

### 對 Jarvis / repo work 的實務建議

- 不要把 Fable 5 當低成本隨便問模型。
- 適合用在高價值、長鏈路、真的需要 frontier capability 的任務。
- 平常探索、草稿、一般 coding 可以先用 Opus / Sonnet。
- 若用 Fable 5 做 Claude Code / Cowork 長任務，先確認 usage credits / spend limit，並盡量把任務切小。

## 持續追蹤清單

- 2026-07-12 23:59:59 PT 後，Help Center 是否再次更新 promotional period。
- Claude model picker 是否仍顯示 Fable 5。
- Settings > Usage 是否出現 Fable 5 specific meter / usage credits prompt。
- Anthropic 是否把 6/30 redeploy blog 的 July 7 文字改掉。
- Pro plan 使用者是否在英文社群回報 7/12 後的實際收費流程。

## 來源

官方來源：

- Anthropic, “Claude Fable.” https://www.anthropic.com/claude/fable
- Anthropic Help Center, “Claude Fable 5 promotional access.” https://support.claude.com/en/articles/15424964-claude-fable-5-promotional-access
- Anthropic Help Center, “Manage usage credits for paid Claude plans.” https://support.claude.com/en/articles/12429409-manage-usage-credits-for-paid-claude-plans
- Anthropic Help Center, “Why Claude switched models in your conversation with Fable 5.” https://support.claude.com/en/articles/15363606-why-claude-switched-models-in-your-conversation-with-fable-5
- Anthropic Help Center, “Data retention practices for Mythos-class models.” https://support.claude.com/en/articles/15425996-data-retention-practices-for-mythos-class-models
- Anthropic Help Center, “Release notes.” https://support.claude.com/en/articles/12138966-release-notes
- Anthropic, “Redeploying Claude Fable 5.” https://www.anthropic.com/news/redeploying-fable-5
- Anthropic, “Statement on the US government directive to suspend access to Fable 5 and Mythos 5.” https://www.anthropic.com/news/fable-mythos-access
- Anthropic Docs, “Models overview.” https://platform.claude.com/docs/en/about-claude/models/overview
- Anthropic Docs, “Pricing.” https://platform.claude.com/docs/en/about-claude/pricing


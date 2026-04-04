# Claude Max 停止涵蓋 OpenClaw 用量：社群反應與應對方案

> **事件日期：** 2026-04-03（公告）→ 2026-04-04 12pm PT 起生效
> **報告產出：** 2026-04-04
> **資料來源：** Hacker News、Reddit（間接）、Twitter/X（間接）

---

## 一、事件摘要

### Anthropic 官方通知內容

2026/04/03，Anthropic 向 Claude Pro/Max/Team 訂閱用戶發送郵件，宣布：

> Starting April 4 at 12pm PT / 8pm BST, you'll no longer be able to use your Claude subscription limits for third-party harnesses including OpenClaw. You can still use them with your Claude account, but they will require **extra usage**, a pay-as-you-go option billed separately from your subscription.

**關鍵變更：**
- ✅ Claude 訂閱仍涵蓋：Claude.ai、Claude Code、Claude Cowork
- ❌ 不再涵蓋：OpenClaw 及所有第三方 harness（先從 OpenClaw 開始，之後擴展到其他工具）
- 💰 第三方工具改為 **extra usage**（pay-as-you-go），與訂閱費分開計費
- 🎁 過渡措施：一次性 credit（等同月費金額），需在 4/17 前兌換
- 📦 預購折扣：批量購買 extra usage 最高可享 30% 折扣
- 💳 可申請退訂退款

### Anthropic 的理由

> "These tools put an **outsized strain** on our systems. Capacity is a resource we manage carefully and we need to **prioritize our customers using our core products**."

---

## 二、社群反應（情緒與觀點）

### A. 憤怒與不滿陣營（多數）

#### 1. 「我付的是 token，不是介面」
**HN 用戶 eagleinparadise：**
> "Anthropic measures your usage based on token consumption. We are paying for a certain amount of token consumption. Why then, is this an outsized strain on your system? It's like buying gasoline from Shell, and then Shell's TOS forcing you to use that gas in a Hummer."

**HN 用戶 stavros：**
> "Anthropic does not let you 'refill however many times you want', they have limits. That's what 'limits' in your account is. It would be like the restaurant saying 'you can buy the 2-liter soda pack' and then getting all uppity when you bring your own 2L hydroflask in."

#### 2. 「這是 bait-and-switch」
**HN 用戶 fc417fc802：**
> "Had they very loudly advertised the subscription as limited to their harness up front with a note about maximum token use people presumably wouldn't feel cheated. You don't get to sell a subscription described primarily as being for some quantity of X and then change the terms every time people find creative ways to use the stream of X."

#### 3. 「這是在推銷自家產品，打壓第三方」
**HN 用戶 goosejuice：**
> "This is about Anthropic subsidizing their own tools to keep people on their platform. OpenClaw is just a good cover story for that. You can maximize plans just as easily w/ /loop."

**HN 用戶 rvz（引用「賭場」比喻）：**
> "The Anthropic casino wants you to continue gambling tokens at their casino only on their machines (Claude Code)... But you cannot repurpose your subscription on other slot machines."

#### 4. 「先讓人上癮，再加價」
**HN 用戶 mememememememo：**
> "Drug dealer got them hooked, now time to charge by the ounce."

#### 5. 「和 OpenAI 沒兩樣了」
**HN 用戶 windexh8er：**
> "I'm not a fan of what they're doing and will likely drop them and go out of my way to find a different option and move away from their lock-in strategy. They're really no different than OpenAI at this point (for the worst)."

---

### B. 理解/支持陣營（少數但有理據）

#### 1. 「這是產能問題，不是價格問題」
**HN 用戶 nl（最深入的分析）：**
> "I suspect people are misdiagnosing the root cause. I think Anthropic is **capacity constrained** so is having to make choices about the customers they want to serve. We know Anthropic weren't as aggressive as OpenAI in signing huge capacity deals... Claude Code usage is growing very fast... Anthropic has suffered from brown-outs. They are choosing which customers to service rather than raising prices."

#### 2. 「OpenClaw 用量確實是 rate limit 惡化的原因」
**HN 用戶 Fogest：**
> "I am pretty sure Claude Code limits were being hit so fast recently because there was an increasing amount of OpenClaw style usage. I bet OpenClaw was the reason for the rate limiting being so bad lately."

#### 3. 「你付的是介面，不是 token」
**HN 用戶 danpalmer：**
> "You don't pay for capacity, you pay for an interface. Paying for capacity is what API keys are for."

**HN 用戶 verdverm：**
> "Those are subsidized tokens because you are also using their product. They have a per-token payment option where you can use any tool you like."

#### 4. 「ToS 早就說了」
**HN 用戶 Jimmc414：**
> "I'm not sure why people expect Anthropic to subsidize tokens through OpenClaw when it's specifically forbidden in the ToS."

**HN 用戶 jasonlotito：**
> "Yes, this was made clear a while back and should not be a surprise. You can use your Claude Code subscription with third-party tools, but you have to use the Claude Code harness. OpenClaw could use the Claude Code harness, but they don't."

---

### C. 技術問題與疑慮

**HN 用戶 Wowfunhappy：**
> "Claude is a UNIX command line tool with an SDK. Yes there's an interactive mode, but it can be invoked as a normal utility too, and piped to other tools and so on. I don't understand the difference between a 'third party harness' and a shell script. **How are they even detecting OpenClaw?**"

**HN 用戶 jatora：**
> "If I have my own local platform similar in nature to openclaw, and am leveraging claude -p through my subscription, is this now against ToS? Or is this just a ban specific to certain services?"

---

### D. 前情：Anthropic vs 第三方工具

此事件並非突然發生。2026 年 3 月，Anthropic 已對 **OpenCode** 採取法律行動，要求其停止使用內部 Claude Code API：

**HN 用戶 malisper 的解析：**
> "Anthropic has two different products: the Claude API (usage-based pricing) and Claude Code (monthly subscription, much cheaper). When it comes to third party products such as OpenClaw and OpenCode, Anthropic has made it clear those products should be using the Claude API and not the internal Claude Code APIs."

---

## 三、應對方案

### 方案 1：繼續用 OpenClaw + Claude Extra Usage（Pay-as-you-go）

| 項目 | 說明 |
|------|------|
| **做法** | 在 Claude 帳號設定開啟 extra usage，OpenClaw 繼續使用 Claude 模型 |
| **優點** | 零遷移成本、維持原有工作流程、一次性 credit 可抵一部分費用 |
| **缺點** | 成本大幅上升、需監控用量避免帳單暴增 |
| **預估成本** | 見下方「成本估算」段落 |
| **適合誰** | 不想改變任何工作流程、使用量較低的用戶 |

**省錢技巧：**
- 🎁 先兌換一次性 credit（4/17 截止）
- 📦 批量預購 extra usage 享最高 30% 折扣

---

### 方案 2：切換到 Claude Code 替代 OpenClaw

| 項目 | 說明 |
|------|------|
| **做法** | 放棄 OpenClaw，完全使用 Anthropic 官方的 Claude Code CLI |
| **優點** | 仍在訂閱涵蓋範圍內（$20 Pro / $100-$200 Max）、官方支持、不會被限制 |
| **缺點** | 功能不如 OpenClaw 全面（缺少多頻道整合、記憶系統、排程等 agent 功能）、需重新適應工作流程 |
| **預估成本** | $0 額外成本（已含在訂閱中） |
| **適合誰** | 主要用 Claude 寫程式的開發者、不需要 OpenClaw 的 agent 框架功能 |

**注意：** HN 用戶 jasonlotito 指出，技術上第三方工具*可以*透過 Claude Code harness 來呼叫，但 OpenClaw 選擇不這麼做。

---

### 方案 3：切換 OpenClaw 底層模型到其他 Provider

| Provider | 優點 | 缺點 | 預估成本 | 適合誰 |
|----------|------|------|----------|--------|
| **Gemini (Google)** | 成本低、速度快、大 context window | 推理品質略遜 Claude、coding 能力差異 | Gemini 2.5 Pro: $1.25-2.5/MTok in, $10/MTok out | 成本敏感用戶、大量文件處理 |
| **OpenAI GPT-4o / o3** | 生態系最大、ChatGPT Pro 有 API 存取 | 部分任務不如 Claude、o3 貴 | GPT-4o: ~$2.5/MTok in, $10/MTok out | 已有 OpenAI 訂閱的用戶 |
| **DeepSeek** | 極低成本、推理能力強 | 中國服務可能有合規疑慮、穩定性 | ~$0.14/MTok in, $2.19/MTok out | 極度成本敏感、不介意中國服務 |
| **本地模型 (Ollama/LM Studio)** | 零 API 成本、完全私密 | 品質顯著下降、需要強力硬體 | 硬體一次性投入 + 電費 | 隱私優先、有 GPU 資源的用戶 |
| **OpenRouter** | 一個 API 用多模型、靈活切換 | 加成本手續費、延遲稍高 | 各模型定價 + ~10-20% markup | 想靈活切換模型的用戶 |

**OpenClaw 已原生支持的 provider：** Claude, OpenAI, Gemini, DeepSeek, OpenRouter 等（來源：KatClaw 描述）

---

### 方案 4：切換到其他 Agent 平台

| 平台 | 優點 | 缺點 | 預估成本 |
|------|------|------|----------|
| **Claude Code（官方）** | 免費（含在訂閱中）、官方維護 | 功能有限、非 agent 平台 | $0（含在訂閱中）|
| **Codex (OpenAI)** | 原生支持 OpenAI 模型 | 生態不同、需重新學習 | API 費用 |
| **OpenCode** | 類似 OpenClaw 的開源替代 | 同樣面臨 Anthropic 限制、已被法律警告 | API 費用 |
| **自建 agent 框架** | 完全控制、不受第三方限制 | 開發維護成本極高 | 開發時間 + API 費用 |

---

### 方案 5：創意混合方案

#### 5a. 模型分流策略
- **核心任務用 Claude Code**（含在訂閱中）
- **OpenClaw 次要任務用便宜模型**（Gemini Flash、Haiku 等）
- 預估成本：$100 訂閱 + $20-50/月 便宜模型 API

#### 5b. HexaClaw 一站式方案
有用戶在 HN 推廣 **HexaClaw**（基於 OpenClaw 的托管雲端層），$19.99/月包含 12+ LLM（Claude、GPT-4、Gemini 等）。但可能也受此政策影響。

#### 5c. 等待 OpenClaw 官方回應
OpenClaw 可能會：
- 改用 Claude Code harness 接入（技術上可行，HN 用戶 jasonlotito 提到）
- 新增 pay-as-you-go Claude 整合選項
- 強化其他模型支持作為替代

---

## 四、成本估算

### 繼續用 OpenClaw + Claude Pay-as-you-go 的額外成本

根據 Anthropic 最新 API 定價（2026-04）：

| 模型 | Input | Output | 說明 |
|------|-------|--------|------|
| **Opus 4.6** | $5/MTok | $25/MTok | 最強模型，agent 與 coding 最佳 |
| **Sonnet 4.6** | $3/MTok | $15/MTok | 性價比最佳 |
| **Haiku 4.5** | $1/MTok | $5/MTok | 最便宜最快 |

### 使用量估算

以一般 OpenClaw 使用者（中度使用）為例：

| 使用程度 | 每日互動 | 估計 token 用量/月 | Sonnet 4.6 月費 | Opus 4.6 月費 |
|----------|----------|-------------------|-----------------|---------------|
| 輕度 | 5-10 次 | ~5M input + 2M output | ~$45 | ~$75 |
| 中度 | 20-30 次 | ~15M input + 5M output | ~$120 | ~$200 |
| 重度 | 50+ 次 | ~40M input + 15M output | ~$345 | ~$575 |
| 極重度（自動化 agent） | 持續 | ~100M+ input + 30M+ output | ~$750+ | ~$1,250+ |

**結論：** 
- 原本 Claude Max 5x = $100/月、20x = $200/月
- 改為 pay-as-you-go 後，**中度使用者大約需要額外 $100-200/月**
- 重度使用者可能 **額外 $300-500+/月**
- 等同 HN 用戶所說：「成本一夜翻倍」

### 省錢計算

- 一次性 credit：$100-$200（等同月費）
- 批量預購 30% 折扣：中度使用者可從 ~$120 降至 ~$84/月
- **最佳短期策略：** 兌換 credit + 批量預購 + 盡快遷移部分用量到更便宜的模型

---

## 五、OpenClaw 官方回應

截至報告時間（2026-04-04 凌晨），**尚未發現 OpenClaw 官方公開回應**。OpenClaw 官方文件站（docs.openclaw.ai）返回 404，Discord 社群動態無法通過 web_fetch 取得。

**預期可能的回應方向：**
1. 支持透過 Claude API key 直接使用（pay-as-you-go）
2. 改用 Claude Code harness 作為後端接入方式
3. 強化 Gemini、OpenAI 等替代模型支援
4. 推出模型切換指南和成本優化建議

---

## 六、總結與建議

### 短期（本週）
1. ✅ **立即兌換一次性 credit**（4/17 截止）
2. ✅ 評估自己的實際 token 用量（`claude.ai/settings/usage`）
3. ✅ 設定 extra usage 上限，避免帳單暴增

### 中期（本月）
4. 🔄 測試 OpenClaw 搭配其他模型（Gemini、OpenAI）的表現
5. 🔄 將非關鍵任務遷移到便宜模型
6. 🔄 考慮批量預購 extra usage（享 30% 折扣）

### 長期（持續觀察）
7. 👀 等待 OpenClaw 官方回應和技術調整
8. 👀 觀察 Anthropic 是否調整政策
9. 👀 評估是否完全遷移到 Claude Code 或其他平台

---

## 七、參考來源

1. **HN: Tell HN: Anthropic no longer allowing Claude Code subscriptions to use OpenClaw** - https://news.ycombinator.com/item?id=47633396 （主討論串，多數意見來源）
2. **HN: Anthropic just cut off Claude subscriptions for OpenClaw** - https://news.ycombinator.com/item?id=47633862
3. **HN: Ask HN: Anthropic changing billing for third-party harnesses for Teams Accounts?** - qdot76367 發文（Teams 版本通知）
4. **HN: Anthropic takes legal action against OpenCode** - https://news.ycombinator.com/item?id=47444748 （前情：3月法律行動）
5. **Boris Cherny (Anthropic) Twitter** - https://x.com/bcherny/status/2040206443094446558 （credit 兌換資訊）
6. **Claude 定價頁面** - https://claude.com/pricing
7. **先前討論（1個月前）** - https://news.ycombinator.com/item?id=47069299 （655 points, 793 comments）

---

*本報告基於 2026-04-04 凌晨可取得的公開資訊。由於事件剛發生不到 24 小時，社群反應和官方回應仍在快速演變中。建議持續追蹤。*

# 本地 LLM 回應中斷問題：診斷與解決方案

> **2026-04-11** | Jones 在 RTX 5080 16GB 上跑 Gemma 4 26B（LM Studio）遇到的問題
> **症狀：** 回應吐到一半就停住，不繼續了。對話累積幾輪後發生。
> **目標：** 找到讓本地 LLM 可以持續回應的方法，不要有割裂感

---

## TL;DR

**你遇到的不是 bug，是 VRAM 被 KV cache 吃光了。**

Gemma 4 26B 的 Q4 量化後模型權重佔 ~15GB。你的 5080 有 16GB VRAM（實際可用 ~15GB，顯示輸出佔 ~1GB）。**剩下給 KV cache（對話記憶）的空間幾乎是零。**

KV cache 隨每一個 token（你說的 + 模型回的）線性增長。當它漲到把 VRAM 吃光，模型就停了——不是「不想回」，是「記憶體裡放不下下一個字了」。

**最快的修復（30 秒）：** 在 LM Studio 把 Context Overflow Policy 改成 `rollingWindow`。

**最根本的修復：** 手動把 context length 設小一點（8192 或 16384），加上 KV cache 量化。

---

## 一、問題診斷：為什麼以前不會，現在會？

### 以前沒遇到的可能原因

1. **以前跑的模型更小**（7B / 13B），模型本身佔的 VRAM 少，剩更多給 KV cache
2. **以前的 context window 設定比較保守**（8K / 16K），不會自動開到 128K
3. **Gemma 4 26B 雖然是 MoE，但所有 expert 權重都要載入 VRAM**（只在計算時選擇性激活，但存儲是全量）

### 這次的數學

```
RTX 5080 總 VRAM:           16 GB
- 顯示輸出佔用:             ~1 GB
- 可用 VRAM:                ~15 GB
- Gemma 4 26B Q4_K_M 權重:  ~15 GB
- 剩餘給 KV cache:          ~0 GB  ← 問題在這
```

**KV cache 一個 token 在 Gemma 4 26B 上大約佔 0.5-1 MB。** 如果你只剩 500MB，大約只能撐 500-1000 個 token 的對話（含你的輸入 + 模型的輸出）。幾輪問答就爆了。

### 為什麼 31B 更慘

31B 是 dense 模型（非 MoE），Q4 量化後 ~18GB，超過你的 VRAM 上限。LM Studio 會把一部分 offload 到 CPU RAM，推論速度暴跌（所以你感覺「一個字等超過一秒」）。

---

## 二、即時修復：LM Studio 設定調整（30 秒搞定）

### 修復 1：改 Context Overflow Policy 為 `rollingWindow`

**這是最快解決「吐到一半停住」的方法。**

**步驟：**

1. LM Studio 左側 → 「My Models」
2. 找到你的 Gemma 4 26B 模型 → 點齒輪 ⚙️ 圖示
3. 切到「Inference」分頁
4. 找到 `Context Overflow Policy`
5. 改成 **`rollingWindow`**

**效果：** 當 KV cache 快滿時，LM Studio 會**自動丟掉最舊的對話記錄**，騰出空間讓模型繼續生成。你不會再遇到「吐到一半停住」，但代價是**前面的對話會被遺忘**。

**三種 Policy 比較：**

| Policy | 行為 | 適合誰 |
|---|---|---|
| **stopAtLimit**（預設） | Context 滿了就停 → **你遇到的就是這個** | 需要完整歷史但接受手動新開 chat |
| **rollingWindow** ⭐ | 自動丟舊訊息，繼續生成 | **聊天情境首選**，接受忘掉舊對話 |
| **truncateMiddle** | 保留開頭（system prompt）和結尾，砍中間 | 特殊用途，容易出 bug，**不推薦** |

### 修復 2：手動降低 Context Length

**步驟：**

1. 同樣在 Gemma 4 26B 的 ⚙️ 設定
2. 找到 `Context Length`（可能在 Model Loader 區）
3. 從預設值（可能是 32768 或更高）降到 **8192** 或 **16384**

**為什麼這樣做：** 降低 context length 直接限制 KV cache 的最大 VRAM 佔用。設成 8192 tokens，KV cache 最多佔 4-8 GB，你的 VRAM 就有足夠空間。

**代價：** 能記住的對話更短。8192 tokens 大約等於 6000 字中文 / 3000 字英文——大概 3-5 輪深入對話。

### 修復 3：KV Cache 量化

**如果你用 Ollama 而非 LM Studio：**

```bash
export OLLAMA_KV_CACHE_TYPE=q8_0   # 8-bit KV cache（省一半）
# 或
export OLLAMA_KV_CACHE_TYPE=q4_0   # 4-bit KV cache（省四分之三）
```

**LM Studio 有沒有類似設定？** 在進階設定裡找 `kv_cache_type` 或類似選項。**q8_0 幾乎不影響品質但 VRAM 省一半——這是你最該先試的。**

**效果：** 原本 KV cache 的 VRAM 佔用砍半或砍四分之三，等於同樣 VRAM 可以撐更長的對話。

### 組合拳：三個一起設

```
Context Length:         16384
Context Overflow:       rollingWindow
KV Cache Type:          q8_0（如果有這個選項）
```

**這樣設完，你應該能聊 10-20 輪不會卡住。** 如果再加上 KV cache q8_0，可能可以撐到 30 輪。

---

## 三、方案 A — 願意放棄前後 context，直接換 session

如果你對「忘掉前面的對話」沒有意見，以下是最乾淨的做法：

### A1. rollingWindow（自動，推薦）

上面已經講了。模型會自動丟舊的，繼續回答。你感覺不到切換，但前面的話它已經不記得了。

**體驗：** 像在跟一個「短期記憶很短的人」聊天。最近幾輪它都記得，再前面的就忘了。

### A2. 手動新開 Chat（最簡單）

當你感覺模型開始「答得怪怪的」或「變慢了」，直接新開一個 chat session。

**體驗：** 最乾淨。每次開新 chat 就是 100% fresh。

### A3. 自動偵測 + reset 腳本

如果你用 LM Studio 的 API 模式（server），可以寫一個腳本：

```python
# 偽代碼
if response.usage.total_tokens > THRESHOLD:
    # 存一份當前對話的 1-sentence summary
    summary = call_llm("Summarize this conversation in 2 sentences: ...")
    # 新開 session，把 summary 當 system prompt
    new_session(system_prompt=f"Previous context: {summary}")
```

**體驗：** 自動切換，但保留一句話摘要讓新 session 知道前面大概聊了什麼。

---

## 四、方案 B — 盡可能保留前後文

如果你希望對話盡量長、模型盡量記得前面的內容：

### B1. 用更小的模型省 VRAM 給 KV cache

**核心邏輯：** 模型權重越小，剩越多 VRAM 給 KV cache，可以聊越久。

| 模型 | 權重 VRAM | 剩餘給 KV cache | 大約可用 context |
|---|---|---|---|
| Gemma 4 26B Q4 | ~15 GB | ~0 GB | 幾乎不能聊 |
| Gemma 4 E4B Q4 | ~2.5 GB | ~12.5 GB | **超長**（64K+ tokens） |
| Qwen 2.5 14B Q4 | ~8 GB | ~7 GB | 中長（16K-32K） |
| Llama 3.3 8B Q4 | ~5 GB | ~10 GB | 長（32K-64K） |

**結論：** 如果你需要長對話，**Gemma 4 E4B 比 26B 更適合**。品質差一截，但不會卡住。

或者用 **Qwen 2.5 14B**——品質和 context 長度的平衡最好。

### B2. Progressive Summarization（漸進式壓縮）

**概念：** 當對話到 70-80% context 容量時，讓模型自己把前半段對話壓成摘要，釋放空間。

```
第 1-10 輪：正常對話（完整保留）
第 11 輪（觸發壓縮）：
  → 把第 1-5 輪壓成 3 句摘要
  → 新的 context = [摘要] + [第 6-11 輪完整對話]
  → 空間釋放，繼續聊
第 20 輪（再次觸發）：
  → 把第 1-10 輪的摘要 + 第 6-10 輪壓成更短的摘要
  → 新的 context = [更短的摘要] + [第 11-20 輪完整對話]
```

**這就是 Claude 的 `/compact` 指令在做的事**——你在 Claude Code 裡用過。

**本地怎麼做：**
- LM Studio 本身**沒有**內建這個功能
- 需要自己寫腳本（用 LM Studio API）或用 **Letta**（前身 MemGPT）這類框架
- Letta 可以接 Ollama / LM Studio 後端，幫你管理「working memory → short-term → long-term」三層記憶

### B3. Letta（前身 MemGPT）— 最接近「無限對話」的本地方案

[Letta](https://github.com/letta-ai/letta) 是一個開源框架，專門解決「LLM context window 不夠長」的問題。它的做法是：

| 記憶層 | 說明 |
|---|---|
| **Working Memory** | 當前對話的 active context（最新幾輪） |
| **Short-term Memory** | 這次 session 的歷史摘要 |
| **Long-term Memory** | 跨 session 的持久記憶（存在 SQLite / PostgreSQL） |

模型在 working memory 裡對話，當空間不夠時，自動把舊的壓成 short-term 摘要，超長期的壓進 long-term。**使用者感覺不到記憶管理在發生**。

**適合你嗎？** 如果你願意花 1-2 小時設定 Letta + Ollama，它能讓你的 Gemma 4 E4B 或 26B 跑出「好像沒有 context 限制」的體驗。但設定複雜度比直接用 LM Studio 高。

### B4. 混合策略：本地快問快答 + Claude 做深度長對話

最務實的做法：

- **本地 Gemma 4 26B**：短對話、快速問答、coding 輔助（1-5 輪解決的事）
- **Claude Max（你已經有）**：需要長對話、深度推理、保留大量 context 的事

本地模型天生不擅長長對話（VRAM 是硬限制），Claude 的 1M context 才是長對話的正確工具。不要把本地 LLM 當 Claude 用——它們的定位是「快速、免費、簡短」。

---

## 五、給 Jones 的具體建議（按優先順序）

### 今天立刻做（5 分鐘）

1. **LM Studio → 你的 Gemma 4 26B 設定 → Context Overflow Policy 改成 `rollingWindow`**
2. **Context Length 降到 `8192`**
3. 重新載入模型

這兩步做完，「吐到一半停住」的問題立刻消失。

### 這週試試看（30 分鐘）

4. **找 KV Cache 量化選項**，設成 q8_0（如果 LM Studio 有的話；如果沒有考慮用 Ollama）
5. **試一下 Gemma 4 E4B**——你會發現它雖然笨一點，但可以聊非常長
6. **試一下 Qwen 2.5 14B Q4**——社群公認 16GB VRAM 上 context 和品質平衡最好的選擇

### 之後有興趣再做

7. **裝 Letta / MemGPT** 玩看看——配合 Ollama 做三層記憶管理
8. **寫一個 progressive summarization 腳本**——70% 容量時自動壓縮前半段

### 心態調整

**本地 LLM 的定位不是「取代 Claude」，是「免費、快速、短對話」。** 如果你需要跟 AI 做 30 輪深度討論，那正確的工具是 Claude Max（你已經有的），不是本地模型硬撐。本地模型的甜蜜點是：3-10 輪的快問快答、coding 輔助、分類、抽取——這些短任務它很強而且零成本。

---

## 六、VRAM 佔用速查表

| 配置 | 模型權重 | KV cache 可用 | 大約可聊幾輪 |
|---|---|---|---|
| 26B Q4 + context 8K | 15 GB | 0-0.5 GB | **3-5 輪（很短）** |
| 26B Q4 + context 8K + KV q8_0 | 15 GB | 0-1 GB | 5-10 輪 |
| E4B Q4 + context 32K | 2.5 GB | 12.5 GB | **50+ 輪（超長）** |
| E4B Q4 + context 64K | 2.5 GB | 12.5 GB | 30+ 輪 |
| Qwen 14B Q4 + context 16K | 8 GB | 7 GB | 20-30 輪 |
| Llama 8B Q4 + context 32K | 5 GB | 10 GB | 30-50 輪 |

**結論：如果你要長對話，選 E4B 或 14B 級模型。26B 在 16GB VRAM 上天生不適合長對話。**

---

## 七、參考來源

- [LM Studio — How to Increase Context Length](https://localllm.in/blog/lm-studio-increase-context-length)
- [LM Studio — Context Overflow Policy Bug](https://github.com/lmstudio-ai/lmstudio-bug-tracker/issues/1620)
- [LM Studio — Context Overflow API Config](https://github.com/lmstudio-ai/lmstudio-bug-tracker/issues/532)
- [Gemma 4 Memory Requirements Wiki](https://www.gemma4.wiki/requirements/gemma-4-memory-requirements)
- [Gemma 4 VRAM Guide](https://gemma4guide.com/guides/gemma4-vram-requirements)
- [Gemma 4 Config Guide](https://gemma4guide.com/guides/gemma4-config-guide)
- [Ollama VRAM Requirements 2026](https://localllm.in/blog/ollama-vram-requirements-for-local-llms)
- [LM Studio VRAM Requirements](https://localllm.in/blog/lm-studio-vram-requirements-for-local-llms)
- [Overcoming Output Token Limits (Medium)](https://medium.com/@gopidurgaprasad762/overcoming-output-token-limits-a-smarter-way-to-generate-long-llm-responses-efe297857a76)
- [Progressive Summarization with LLM (Medium)](https://medium.com/@auslei/progressive-summarisation-using-llm-with-limited-context-window-e160f5041316)
- [Context Window Overflow Strategies (Redis)](https://redis.io/blog/context-window-overflow/)
- [Top Techniques to Manage Context Length (Agenta)](https://agenta.ai/blog/top-6-techniques-to-manage-context-length-in-llms)
- [Letta / MemGPT GitHub](https://github.com/letta-ai/letta)
- [VladimirGav Gemma4-26b-16GB-VRAM Ollama Model](https://ollama.com/VladimirGav/gemma4-26b-16GB-VRAM)

---

## 修訂紀錄

- **2026-04-11 v1** — 初版。由 Jarvis（Dispatch 研究）產出。涵蓋診斷、即時修復（rollingWindow + context 降低 + KV cache 量化）、方案 A（放棄 context）、方案 B（保留 context）、VRAM 速查表

---

*本報告基於 2026-04-11 的 LM Studio / Ollama / Gemma 4 版本。軟體更新可能改變設定位置或增加新功能。*

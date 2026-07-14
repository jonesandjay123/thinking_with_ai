# MiniCPM-1B 是新的開源王者嗎？——2026-07-14 開放權重模型比較

> 結論先講：*若你聽到的是「MiniCPM-1B」，大概率指的是 2024 的舊模型，或是把名稱簡化了。現在真正被討論的新品是 **MiniCPM5-1B**（2026-05）。它很可能是目前最強的 **1B 級本地／端側文字模型之一**，但不是所有尺寸、所有任務的「開源總冠軍」。*

MiniCPM5-1B 的價值不是取代 4B、7B、24B 或 MoE 模型，而是把「能做工具調用、程式、推理與長上下文」的下限壓到約 1B。若產品需要在手機、Mac、低成本常駐服務或離線 agent 上運行，它非常值得優先測；若目標是最高品質的一般工作助手，Qwen3 4B／30B-A3B、Mistral Small 3.1 24B 等仍在不同的能力級距。

## 先校正名稱與「開源」的意思

- **MiniCPM-1B**：OpenBMB 在 2024 年推出的早期 1B 系列型號，不能直接拿來代表今天的最新能力。
- **MiniCPM5-1B**：OpenBMB 於 2026-05-19 發布的現行 1B 旗艦；1.081B 總參數、24 層、原生 131K context、同一權重支援 Think / No Think，並提供 GGUF 與 MLX 4-bit 格式。
- 本文將「開源」精確寫為 **open weights（可下載權重）**。大多數主流模型並未完整公開訓練資料、訓練程式與全流程可重現條件；授權也不同。MiniCPM、Qwen3、Mistral Small 3.1 是 Apache-2.0；Phi-4-mini 為 MIT；Gemma 與 Llama 屬自訂社群／使用條款，商業合規不能一概而論。

## MiniCPM5-1B 到底強在哪裡？

OpenBMB 將它定位為端側與資源受限情境的 dense 1B Transformer。官方比較的同量級對手包含 LFM2.5-1.2B-Thinking、Qwen3-0.6B/think、Qwen3.5-0.8B/think，並主張它在該集合達到 1B 級 SOTA，優勢集中在 tool use、code 與較難的推理題。

它的幾個實際訊號：

- **不是只有聊天的小模型**：同一 checkpoint 可開關思考模式，目標包含 local assistant、coding agent、tool-use workflow。
- **長 context 在 1B 級很少見**：原生 131K，但不代表 1B 在 131K 的真實檢索與多步推理會等同大型模型；KV cache、吞吐與實際任務需另測。
- **部署摩擦低**：官方提供 BF16、GGUF、MLX 4-bit 及主流推理／微調框架的 cookbook。對 Apple Silicon 與 `llama.cpp` / Ollama / LM Studio 的本地試驗特別友善。
- **訓練策略值得注意**：官方公開描述包含 SFT、RL、on-policy distillation（OPD）；其主張 RL+OPD 在指定數學、程式、指令遵循任務的平均分提高 16 點，並降低過長輸出比例。這是廠商自評，應視為選型假設，不是跨實作的最終結論。

## 同場比較：不要用一張排行榜混掉不同級距

|模型|定位／適合什麼|關鍵規格與能力|主要代價／提醒|
|---|---|---|---|
|**MiniCPM5-1B**|端側、離線助手、輕量工具 agent、低成本常駐分類／草稿|1.081B、131K、Think/No Think、GGUF/MLX；1B 級工具／程式／推理主張強|小模型仍有知識、穩定性、複雜規劃天花板；官方「SOTA」只適用 1B 級比較集合|
|**Qwen3-4B**|最值得先裝的通用本地文字模型；中英多語、程式、一般 agent|4B dense、32K 原生（YaRN 可到 131K）、Think/No Think、100+ 語言；Apache-2.0|比 1B 多出明顯記憶與延遲成本；長 context 延伸不等於原生等效|
|**Phi-4-mini-instruct**|需要較小但重視指令遵循、長文件與 reasoning 的文字模型|約 3.8B、128K、MIT；以合成資料＋過濾公開網站訓練|不是視覺模型；中文、工具協定與真實 agent 體驗應自己測|
|**Gemma 3 4B IT**|本地多模態：圖片理解、文件／圖表、長 context、多語|文字＋圖片輸入、128K、140+ 語言；4B 是可落地的 vision 選擇|Gemma license 不是 Apache/MIT；存取與商業使用先過條款|
|**DeepSeek-R1-Distill-Qwen-7B**|把數學／程式推理當主要 KPI，願意接受較慢較長回答|7B distilled reasoning；官方 AIME 2024 pass@1 為 55.5、MATH-500 為 92.8（其指定設定）|較舊的一代；長 CoT 會拖慢互動、也未必是穩定 agent 的最佳預設|
|**Qwen3-30B-A3B**|單機／小型伺服器的高品質通用 agent 與推理，想要 MoE 效率|30.5B 總參數、每 token 3.3B activated、128 experts、32K 原生（YaRN 131K）；Apache-2.0|**3.3B activated 不等於只需 3.3B 的記憶體**：權重與 MoE routing 仍要容納 30B 級模型；量化後才是務實本地選項|
|**Mistral Small 3.1 24B**|高品質自架全能模型：文字＋視覺、function calling、長文件、專業微調|24B、vision、128K、Apache-2.0；官方稱量化後可在 32GB RAM MacBook 或單張 4090 運行|BF16/FP16 約需 55GB GPU RAM；不是端側模型，應當服務端部署|

## 哪一個是「王者」？正確答案是分賽道

### 1. 端側／手機／Mac 常駐：MiniCPM5-1B 有資格叫 1B 王者

若限制是「模型必須真的小、可離線、可量化、又不能只會閒聊」，MiniCPM5-1B 是目前非常有說服力的選擇。它把 reasoning 與 tool-use 變成 1B 級的可用能力，對本地 UI agent、個人知識庫第一層、分類／萃取、低風險草稿尤其有價值。

但不要期待它單獨承擔：高可靠研究、跨檔案規劃、嚴格程式 agent、自主長任務。較合理的架構是 **MiniCPM 做 first-pass / router / offline fallback，難題升級到 4B+ 或雲端模型**。

### 2. 一般本地文字助手：Qwen3-4B 是更穩妥的預設

若硬體容許 4B，Qwen3-4B 通常是比 1B 更安全的起點：有思考切換、32K 原生 context、多語與 agent 導向設計，Apache-2.0 也降低整合阻力。它不是各基準絕對第一，但「能力、部署、授權、社群工具」的平衡很好。

### 3. 本地視覺／文件理解：Gemma 3 4B；追求高品質則 Mistral Small 3.1

Gemma 3 4B 的關鍵不是只比文字分數，而是 4B 級就提供圖片輸入與 128K context；很適合 screenshot、表格、文件、相簿型工作流。若有 32GB Mac 或 4090 級資源，Mistral Small 3.1 24B 是更完整的私有部署候選：vision、function calling、JSON、128K 與 Apache-2.0。

### 4. 高難推理：不要忽略 DeepSeek-R1 distill，但別把它當唯一助手

R1 distill 仍是小中型模型推理能力的重要基準，特別是數學與競賽程式。但它是為長推理策略優化的模型：成本、延遲、語氣與工具流程的可控性，要以實際 agent 任務驗證。它適合「需要深想時的 expert」，不一定適合每一輪都開的 default chat model。

## 給 Jones 的具體選型建議

1. **想探索 local Jarvis / Voice Lite 的第一層**：先跑 MiniCPM5-1B MLX 4-bit。用它做 intent router、短回答、離線 fallback、資料萃取；把它的失敗樣本收集成升級路由規則。
2. **要一個真正能做事的本地 default**：同時測 Qwen3-4B。測試集至少含繁中對話、英文程式、工具呼叫 JSON、100K 以下長文本摘要、拒絕／不確定性表達。
3. **要讀圖或 UI screenshot**：加測 Gemma 3 4B；若對視覺／長文件品質有明顯要求，再上 Mistral Small 3.1 24B。
4. **不要只看廠商雷達圖**：用同一套 prompt、相同量化、同一 runtime、固定輸出 token budget 比較。小模型常因長思考 token 把「分數高」換成「體感慢」。
5. **不要把 MoE 的 activated params 當 VRAM 需求**：Qwen3-30B-A3B 的每 token 計算近 3.3B，但載入和記憶體規畫仍是 30B 級問題。

## 建議的最小驗證實驗

在同一台 Mac / 同一套 runtime，對 MiniCPM5-1B 4-bit、Qwen3-4B 4-bit、Gemma 3 4B 4-bit 做 30 題小型 eval：

- 8 題繁中事實整理與不確定性處理
- 6 題英文程式修正／單元測試設計
- 6 題 tool-call JSON schema 遵循
- 5 題長文摘要與資料擷取
- 5 題 screenshots／圖表（Gemma 3 專項）

記錄：首 token 延遲、tokens/s、總耗時、JSON 有效率、人工 1–5 分、幻覺／違反指令次數、記憶體峰值。這比跨廠商、不同 prompt、不同 sampling 的 benchmark 排名更能回答「誰是你的王者」。

## 來源與查核範圍

查核時間：2026-07-14。本文優先使用模型作者／官方模型卡；其中 benchmark 多為模型方公布，未當作獨立第三方排名。

1. [OpenBMB MiniCPM GitHub](https://github.com/OpenBMB/MiniCPM) — MiniCPM5-1B 發布日、1B 級定位、系列歷史、Apache-2.0 LICENSE。
2. [MiniCPM5-1B model card](https://huggingface.co/openbmb/MiniCPM5-1B) — 1.081B、131K、Think/No Think、格式、訓練與官方同級比較。
3. [Qwen3-4B model card](https://huggingface.co/Qwen/Qwen3-4B) — 4B、context、雙模式思考、語言與部署資訊。
4. [Qwen3-30B-A3B model card](https://huggingface.co/Qwen/Qwen3-30B-A3B) — 30.5B total / 3.3B activated、MoE、context。
5. [Gemma 3 4B IT model card](https://huggingface.co/google/gemma-3-4b-it) — 多模態、128K、140+ 語言與授權連結。
6. [Phi-4-mini-instruct model card](https://huggingface.co/microsoft/Phi-4-mini-instruct) — 128K 與 MIT license。
7. [DeepSeek-R1 distill model card](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-7B) — distill 方法、系列尺寸、官方 benchmark。
8. [Mistral Small 3.1 24B model card](https://huggingface.co/mistralai/Mistral-Small-3.1-24B-Instruct-2503) — 24B、vision、128K、Apache-2.0、部署記憶體與官方 benchmark。

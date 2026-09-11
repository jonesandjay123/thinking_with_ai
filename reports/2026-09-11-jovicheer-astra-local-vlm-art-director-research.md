# Jovicheer World Astra：RTX 5080 16GB 本地 VLM Scene Art Director 選型

> 研究日期／來源存取日：2026-09-11
> 目標：Windows + NVIDIA RTX 5080 16GB，在本地以開放權重模型協助 Three.js 世界的 Art Pass；不以 coding benchmark 作主排名，而以多圖視覺理解、空間／視角推理、具體美術建議、約束遵守與迭代品質為主。

## Executive Summary

**可以，而且已經有實用價值；但它是「有眼睛、能持續給可驗證 Art Pass 的副導演」，不是可獨立設計可玩世界的總監。**

RTX 5080 16GB 足以在本地跑 7–9B 視覺語言模型（VLM）的 Q4/Q5/GGUF 或 4-bit/AWQ 版本，並以 2–4 張 Jovicheer screenshot + 一張 2D canonical map + 一張 mood reference 做比較、指出高影響問題、產生帶 zone / parameter 的修改清單，再對修改後圖片作第二輪批評。

不能期待它可靠地：

- 從單張圖準確重建 frozen topology、elevation 或可走性；
- 自動理解未提供的 Three.js state；
- 一次產出可直接合併、完全無誤的 scene edits；
- 把一張情緒參考圖直接變成完整可玩的城堡／地形。

最正確的定位是：**人類維護 canonical 2D model、frozen paths/stations/landmarks/elevation；VLM 只提出受約束的 art-tool calls，renderer 與人類驗收每輪結果。**

### 明確推薦

1. **#1 Qwen3-VL-8B-Instruct，Q4/GGUF 或 4-bit 版本**：16GB 最平衡的主力；適合多圖比較、中文/英文指令、結構化 Art Pass JSON 與後續工具迴圈。
2. **#2 MiniCPM-V 4.5，Q4/4-bit**：高頻 screenshot reviewer；小、快、多圖／影片取向強，適合每輪 render 後快速復盤。
3. **#3 GLM-4.1V-9B-Thinking，Q4/4-bit**：較慢的「深度 review」備機；用於視線、遮擋、空間關係與高影響排序的第二意見。

若願意用較慢速度換取更強的長篇 planning／tool-call JSON，再測 **Gemma 4 12B Unified Q4**；它接近 16GB 邊界，必須留意 image tokens、context、KV cache 與 runtime overhead，不能把「權重檔剛好塞下」誤認為舒適可用。

---

## 1. 16GB 的真實限制

VLM 的顯存不是只有 parameter × quantization bits。實際預算還包含：vision encoder / projector、KV cache、image tokens、多圖、context、CUDA/runtime workspace 和 Windows 顯存碎片。

| 模式 | 16GB 上的判斷 |
|---|---|
| **完全 GPU resident** | 7–9B 的 Q4/Q5/GGUF 或 4-bit 通常合理；保留約 2–4GB 給 vision + KV + overhead，且限制 context/圖片數量。 |
| **大部分 GPU + 少量 RAM offload** | 10–14B Q4 可嘗試；可用但首 token 與多輪 latency 變差，不適合每次 UI 小改都呼叫。 |
| **理論可跑但不建議** | 20B+ 或 MoE 大模型靠大量 offload；可做離線一次性評審，通常不符合「反覆 screenshot → critique」工作流。 |

**實務預設**：每回合先限 2–4 張 1280px 以內的圖、8k–16k context、輸出上限 800–1200 tokens。多圖或高解析不是免費：它會吃 image tokens、KV cache 並降低速度。

---

## 2. Top 5：最值得在 RTX 5080 16GB 測試的本地 VLM

> VRAM 為本次任務的保守工程估算（含合理 runtime 餘裕），不是供應商保證；先以 benchmark 實測。`強／中／弱`是本任務的相對判讀，不是通用 benchmark 名次。

| Rank | 模型／主要來源 | 16GB 建議與 VRAM 判讀 | 多圖／空間／Art critique | Windows runtime 與結論 |
|---|---|---|---|---|
| **1** | [Qwen3-VL-8B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-8B-Instruct) | Q4_K_M / 4-bit；約 7–10GB weights + vision/runtime，保留 4GB 左右作 KV；2–4 張圖可合理運作 | 多圖：強；空間：強；能把 map + view A/B + reference 做有條件比較；tool JSON：強 | **主力**。LM Studio/llama.cpp 若模型格式與 vision projector 正確，或 Transformers；先測 schema adherence。license 依 model card。 |
| **2** | [MiniCPM-V](https://github.com/OpenBMB/MiniCPM-V)（4.5 family） | Q4/4-bit，通常是本榜最寬鬆；約 6–10GB 實用預算 | 多圖：強；視覺比較／video：強；空間：中強；藝術判斷需用 rubric 固定 | **高速 reviewer**。適合每輪 screenshot 後檢查「是否真的改善」。 |
| **3** | [GLM-4.1V-9B-Thinking](https://huggingface.co/zai-org/GLM-4.1V-9B-Thinking) | Q4/4-bit，約 10–14GB 工作負荷；圖多時需減 context | 多圖：中強；空間／遮擋／推理：強；tool planning：強 | **深度第二意見**。較慢，放在 Art Pass 1→2 的關鍵 review，不做每次滑桿調整。 |
| **4** | [InternVL3.5-8B](https://huggingface.co/OpenGVLab/InternVL3_5-8B) | 4-bit/AWQ；約 9–13GB，圖片解析與 dynamic tiling 要保守 | 多圖／高解析：強；GUI/細節觀察：強；空間：中強 | **高解析比較備機**。較適合看小 landmark、UI/debug overlay 和細節遮擋。 |
| **5** | [Gemma 3 12B IT](https://huggingface.co/google/gemma-3-12b-it)（若當前 runtime/模型卡提供 vision 變體則依該變體） | Q4，約 12–16GB；常需少量 CPU offload 或縮小 context | 美術語言／長篇 brief：中強；多圖能力依實際 vision runtime；空間：中 | **較慢 director/planner 實驗**。不要作第一隻下載，也不要假定所有 GGUF build 都完整支援其 vision path。 |

### 不列為主榜的家族

- **Pixtral 12B**：視覺能力很有價值，但 16GB 多圖和 context 餘裕偏緊；可用量化／offload 做對照，不當明天主力。
- **Kimi/Moonshot**：雲端 vision 產品很強，但未找到應被當成「16GB 本地、開放權重 vision 主力」的等價選擇；不要混淆 API 與本地部署。
- **DeepSeek**：本地可用的小型 visual/distill 路線可試，但本次未找到在多圖 scene critique 上比前三更有明確優勢、且 16GB 更舒服的候選。

---

## 3. 這些模型對 Jovicheer 能做到什麼？

### 已可實用的任務

- 比較 scene screenshot 與 moonlake/mood reference 的 fog、moon intensity、cold/warm palette、atmospheric perspective、water roughness、植被密度與 silhouette。
- 比較 view A / B，指出某 landmark 在哪個視角丟失、路徑導引被哪一區遮擋、castle reveal 太早／太晚或 lake focal point 被破壞。
- 根據 2D canonical map 的標註，把問題限制成「zone、視線、非拓撲」語言，而非臆測路徑改造。
- 將建議輸出為 schema：affected zone、problem、desired result、specific tool call、expected effect、risk。
- 對 before/after 進行 verification：哪些改善成功、哪些無效、是否引入 landmark readability/navigation risk。

### 仍不可靠的任務

- 只從 screenshot 推出完整 3D world coordinates 或 collision/navigation correctness。
- 在沒有 semantic zone map / camera metadata 時，精確說出物件 ID、fog range 或 terrain parameter。
- 自主決定改動 frozen landmarks／stations／elevation；這必須在 tool schema 層禁止。

**結論**：它能做出很有用的 Art Pass assistant；成功的關鍵不是換更大模型，而是把 canonical constraints、camera metadata、zone names 和可用工具明確餵給它。

---

## 4. Scene Art Director Test（明天可跑）

### 固定輸入

每隻模型都給相同資料，並限制圖數與解析度：

1. `art-pass-1-view-a.png`：玩家進場／路徑主要視角。
2. `art-pass-1-view-b.png`：lake/castle reveal 或反向視角。
3. `moonlake-reference.jpg`：目標氛圍 reference。
4. `canonical-map.png`：只標 zone、frozen paths、stations、landmarks、elevation bands。
5. 一段不可違反約束：`Do not alter frozen topology, paths, stations, landmarks, or elevation.`
6. 可用 tool manifest（例如 fog, moon, vegetation, sightline, water, palette）。

### Prompt

```text
You are the Jovicheer World Astra Scene Art Director.
Compare the current scene views against the mood reference and canonical map.
Identify the five highest-impact visual problems. Do not alter frozen topology,
paths, stations, landmarks, or elevation.

Return JSON only. For each recommendation specify:
- affectedZone
- visualProblem
- desiredResult
- concreteToolCall
- expectedVisualEffect
- navigationOrReadabilityRisk
- confidence
Rank by visual impact.
```

第二輪餵 `art-pass-2-view-a/b`：

```text
Compare Art Pass 2 against Art Pass 1 and the reference. Which proposed
improvements actually succeeded? Which did not? Identify regressions and return
the next three tool calls. Do not invent changes that are not visible.
```

### 100 分評分規則

| 維度 | 分數 | 觀察點 |
|---|---:|---|
| Reference matching | 15 | 是否正確抓到 lighting/fog/palette/silhouette 的真正差異 |
| Visual perception | 15 | 是否看見遮擋、water、植被、landmark、readability 問題 |
| Spatial + multi-view understanding | 15 | 是否能跨 A/B/map 一致定位，而非把不同視角當不同世界 |
| Artistic judgment | 15 | 是否抓到最高槓桿的五項，而非列瑣碎事項 |
| Specificity / executable calls | 15 | 是否產生具體、可映射到 tool 的參數變更 |
| Constraint preservation | 10 | 是否不碰 frozen topology/path/station/landmark/elevation |
| Before/after verification | 10 | 是否辨識真改善、失敗和 regression |
| Hallucination control | 5 | 是否承認看不見／不確定，而非編造物件或修改 |

**通過門檻**：75+ 才可接進半自動 Art Pass；85+ 且 Constraint preservation ≥9/10、Hallucination ≥4/5，才考慮允許 limited tool calls；低於 75 只當人類 brainstorming reviewer。

---

## 5. screenshot → critique → tool calls → rerender 的自主迴圈

技術上現在已可行，但應採**受限自治**：

```text
screenshots + canonical map + tool manifest
  → VLM JSON critique
  → schema validator + policy gate
  → allowlisted tool calls（最多 2–3 個、參數範圍受限）
  → Three.js rerender（固定 camera A/B）
  → VLM before/after review
  → human approve / rollback
```

### 必要護欄

- 將 frozen topology、path、station、landmark、elevation 寫進 immutable policy，不讓模型以 text 規避。
- 每個 tool 有 zone allowlist、數值 min/max、每回合變動上限；例如 `setFogDensity` 每次最多 ±0.015。
- `clearSightline` 不接受任意文字座標，只接受 canonical map 定義的 corridor ID。
- 固定 camera、seed（若適用）、render resolution，才可以比較 before/after。
- VLM 給的是 proposal，不直接執行；schema validation、renderer screenshot 與 human approval 是閉環的一部分。

**最有希望帶 tool use 的模型**：Qwen3-VL-8B（主）、GLM-4.1V-9B-Thinking（複雜 review）、Gemma 12B（慢速 planner）。MiniCPM-V 更適合作快速視覺 verifier。

---

## 6. 非 LLM 3D／scene 支線（與 VLM Art Director 分開）

| Rank | 工具／模型 | 16GB 可行性 | 最適合 Jovicheer | 不要期待 |
|---|---|---|---|---|
| **1** | [Hunyuan3D](https://github.com/Tencent-Hunyuan/Hunyuan3D-2) | 單物件 image-to-3D 有合理 16GB 路徑；依版本／texture stage 實測 | 石頭、斷牆、拱門、hero prop、ruins、樹根石雕 | 一張圖直接做可玩城堡／正確 modular topology |
| **2** | [Stable Fast 3D](https://github.com/Stability-AI/stable-fast-3d) | 單張圖快速 mesh baseline，16GB 合理 | 快速 GLB 草稿、reference→asset 方向判斷 | 乾淨 UV/PBR/遊戲級 LOD 不需後製 |
| **3** | [Depth Anything V2](https://github.com/DepthAnything/Depth-Anything-V2) + [SAM 2](https://github.com/facebookresearch/sam2) | 顯存友善、很適合本地工具鏈 | depth/normal/segmentation、terrain height/reference mask、asset isolation | 真實 metric terrain 或自動世界 layout |
| **4** | [TRELLIS](https://github.com/microsoft/TRELLIS) | 16GB 多半需 low-VRAM/offload；選擇性試 | 高品質單件 asset research baseline | 明天第一個安裝／大量產出 |
| **5** | 多視角／relight/texture pipeline | 依模型不同，16GB 可做小批次 | texture reference、PBR mood、multi-view consistency 輔助 | 自動保留 scale、collision、navigation |

**建議工作流**：VLM 把 reference 拆成資產 brief → 2D 先產生乾淨單件 reference → Hunyuan3D/SF3D 產 hero asset → Depth/SAM 做 mask、terrain projection → Blender 做 scale、retopo、UV、collision、LOD → 再接 Three.js。這條路對單件石頭、樹、ruins、castle ornament 很實用；完整世界仍應由既有 map-driven layer 主導。

---

## 7. 明天的第一套 Windows 環境

### Runtime 排名

1. **LM Studio（第一天）**：Windows 原生、模型搜尋／下載與多圖聊天最短；用 GGUF Q4_K_M 做 VLM proof-of-value。先確認該模型包含正確 vision projector/processor，不能只下載文字 GGUF。若當日 Qwen3-VL 的 GGUF/vision runtime 尚未完整可用，直接以 **Qwen2.5-VL-7B-Instruct Q4_K_M** 做第一個可重複的 benchmark，不為追最新名稱犧牲多圖功能。
2. **Ollama（第二步）**：適合把已通過 benchmark 的模型提供成本機 API，接到 Jovicheer tool-loop；vision support 與 model tag 要以當前 Ollama library/model card 為準。
3. **llama.cpp（長期可控）**：最適合精確控制 GGUF、GPU layers、context、offload 和 local HTTP server；vision model 需成對使用正確 mmproj/processor。
4. **Transformers + bitsandbytes/Flash Attention（模型官方路徑）**：當最新 VLM 尚未被 LM Studio/Ollama 完整支援時用；安裝與 VRAM tuning 較複雜，但不該為了簡單犧牲視覺能力。
5. **vLLM/SGLang**：Windows 原生第一天不推薦；較適合 WSL2/Linux、多人／高吞吐服務，單卡 16GB desktop 是不必要的複雜度。

### 只下載三隻的實驗清單

1. **Qwen3-VL-8B-Instruct Q4（第一隻）**：主 benchmark；平衡多圖、中文、空間與 structured output，最可能成為長期主力。
2. **MiniCPM-V 4.5 Q4（第二隻）**：測多圖和 after-pass verifier；若它在 2–4 圖速度和一致性明顯更好，就做高頻 reviewer。
3. **GLM-4.1V-9B-Thinking Q4（第三隻）**：測「少量但深度」評審；用在 castle reveal、sightline、lake focal point 等高影響決策。

每隻先下載量化檔約 **5–8GB**（視 GGUF/4-bit 格式而異），另保留模型 cache、vision projector 與至少 8GB 系統 RAM 餘裕；不要預留到剛好 16GB。

---

## Sources

### 模型／runtime 官方來源

1. [Qwen3-VL-8B-Instruct model card](https://huggingface.co/Qwen/Qwen3-VL-8B-Instruct)（2026-09-11）
2. [MiniCPM-V official repository](https://github.com/OpenBMB/MiniCPM-V)（2026-09-11）
3. [GLM-4.1V-9B-Thinking model card](https://huggingface.co/zai-org/GLM-4.1V-9B-Thinking)（2026-09-11）
4. [InternVL3.5-8B model card](https://huggingface.co/OpenGVLab/InternVL3_5-8B)（2026-09-11）
5. [Gemma 3 12B IT model card](https://huggingface.co/google/gemma-3-12b-it)（2026-09-11）
6. [llama.cpp](https://github.com/ggerganov/llama.cpp)（2026-09-11）
7. [LM Studio vision documentation](https://lmstudio.ai/docs/app/basics/vision)（2026-09-11）
8. [Ollama vision documentation](https://docs.ollama.com/capabilities/vision)（2026-09-11）

### 非 LLM 3D／scene 來源

9. [Hunyuan3D 2](https://github.com/Tencent-Hunyuan/Hunyuan3D-2)（2026-09-11）
10. [Stable Fast 3D](https://github.com/Stability-AI/stable-fast-3d)（2026-09-11）
11. [Depth Anything V2](https://github.com/DepthAnything/Depth-Anything-V2)（2026-09-11）
12. [Segment Anything 2](https://github.com/facebookresearch/sam2)（2026-09-11）
13. [TRELLIS](https://github.com/microsoft/TRELLIS)（2026-09-11）

> 注意：模型 card、量化供應者、runtime 對 vision projector、多圖、Blackwell CUDA 支援的狀態變化很快。下載前應以同一模型的官方 model card 和實際 runtime release notes 驗證；本報告將「單卡 16GB 的可用性」視為需要以 Jovicheer benchmark 證明的工程假設，而非僅看參數量。

# 影片換臉與身份保留影片技術地圖（2026-09-07）

> 研究問題：2021 年的 DeepFake / FaceSwap 工具，到了 2026 年有哪些明顯的技術演進？有哪些仍在維護、可實際評估的開源工具與 repo？
>
> 使用邊界：本報告只討論已取得肖像使用同意、角色創作、影視後製、教育研究與內部原型。不要用於冒名、詐騙、未經同意的真人肖像、私密影像、未成年人或誤導性傳播。公開發布合成真人媒體時，應明確標示 AI／合成處理。

## 一句話結論

**有，而且技術已經不是單純「把 A 的臉逐幀貼到 B 的影片」而已。**

2021 年常見的 FaceSwap／DeepFaceLab／roop 類路線，核心是偵測臉部、抽取 identity embedding、對齊、替換、遮罩融合與修復。它對「已有一段影片，想把授權臉替換成角色臉」仍然非常實用。

2024–2026 的重要進展是第二條路線：**身份保留的生成式影片（identity-preserving video generation）**。它以一張或多張參考人像，配合動作影片、姿態、語音、mask 或文字提示，重新生成整個人物鏡頭；因此可同時處理表情、頭部姿態、局部光照、衣物、背景甚至全身動作。代價是可控性、算力、時間與授權盤點也更複雜，而且它不等於逐幀精準的傳統換臉。

如果今天要開始評估：

1. **既有影片的離線換臉／後製**：先看 **FaceFusion**。
2. **單張角色圖要自然跟隨頭部表情動作**：先看 **LivePortrait**。
3. **研究「保留同一人物身份、但用文字生成新短片」**：看 **ConsisID**。
4. **全身姿態驅動**：看 **MimicMotion**；需要較強 NVIDIA GPU 與更多測試。
5. **想整合生成工作流，而不是獨立 app**：看 **ComfyUI-ReActor**，但要特別審核底層模型權利。

---

## 1. 技術演進：從逐幀貼臉到身份條件化生成

| 類型 | 典型輸入 | 典型輸出 | 優勢 | 主要弱點 |
|---|---|---|---|---|
| 傳統任意換臉 | source 臉圖 + target 影片 | target 影片中的臉被替換 | 對既有鏡頭直接、可離線、可批次 | 側臉、遮擋、快速運動、低畫質與劇烈光線常造成閃爍或邊界問題 |
| 肖像驅動動畫 | source 人像 + driving video | source 肖像跟隨頭部姿態與表情 | 快、自然、控制清楚 | 對全身／大幅肩部動作不理想；不是任意 target-video 的精準換臉 |
| 姿態驅動人物影片 | 一張人物圖 + pose sequence | 角色做出全身動作 | 可處理全身動作與角色化 | 手、衣物、遮擋、長時間一致性仍是難題；算力較高 |
| 身份保留 T2V / I2V | 參考臉圖 + prompt（可加首幀） | 新生成影片且維持身份 | 能改場景、鏡頭語言與動作，不只是替換臉 | 生成不可完全預測；身份、動作、畫質、速度之間需取捨 |
| 工作流節點整合 | 圖片／影片／生成圖工作流 | 可串接換臉、修復、遮罩、放大 | 最彈性，適合技術型創作 | 安裝、模型版本、VRAM、授權相依性最複雜 |

### 這個差別為什麼重要？

傳統換臉是在**保留原始鏡頭**的條件下替換臉部；生成式方法則可能**重建整個鏡頭**。前者通常更適合後製與角色替身，後者更適合從已同意的人像建立新角色片段、風格化短片與可控動畫。

因此，2026 年的合理問題不再是「哪個工具最像 DeepFake？」而是：**你希望保留原影片的哪些元素，又願意讓模型重新生成哪些元素？**

---

## 2. 目前最值得看的開源／本機工具

### A. FaceFusion：當前最直接的本機影片換臉候選

- **定位**：本機 GUI / CLI 的 face-manipulation 平台；有影像與影片流程、批次與 headless job 工作流。
- **為什麼值得看**：仍在活躍維護。近期 release 記錄包含新的 swapper、FFmpeg 影片管理、AV1、HDR 色彩處理、CoreML 與 CUDA provider 改善、macOS QuickTime 相容性與 VRAM 修正。
- **適合**：已取得授權的既有短片，要進行離線角色化、替身臉後製或內部概念測試。
- **不保證的事**：極端側臉、髮絲／手部遮臉、快速運動、低清素材與光線突變時，仍可能有身份漂移、膚色不連續、臉部邊界或時間閃爍。高品質取決於素材、偵測／遮罩、swap model、enhancer 與輸出編碼整體。
- **硬體判斷**：提供 CUDA 與 CoreML 等 execution provider；CPU 雖可能可用，但影片吞吐量通常不適合實務。Apple Silicon 值得 benchmark，但不要先假設它能取代 NVIDIA GPU 的吞吐量。
- **授權提醒**：主 repo 授權為 OpenRAIL-AS；下載的模型與第三方 processor 可能有額外條款，商用前應逐項盤點。
- **來源**：[repo](https://github.com/facefusion/facefusion)｜[releases](https://github.com/facefusion/facefusion/releases)｜[license](https://github.com/facefusion/facefusion/blob/master/LICENSE.md)

### B. ComfyUI-ReActor：適合整合式影像／影片工作流

- **定位**：ComfyUI custom node，可把 face swap、face model、遮罩、修復與放大接入既有的節點圖。
- **近期性**：目前 repo 的 changelog 仍有核心更新，例如不再強制依賴 InsightFace 的 ReActor Core、Face Similarity、HyperSwap / ReSwapper 支援、臉部遮罩與針對臉部的 restore 流程。
- **適合**：已在使用 ComfyUI，且想將換臉與生成、去背、修復、超解析、影片幀序列工作流串起來的人。
- **限制**：它是「工作流底座」而非一鍵產品；可用性高度取決於 ComfyUI、ONNX Runtime、顯卡、模型與節點版本的組合。影片穩定性仍需要遮罩、修復與輸出設定一起測。
- **授權陷阱**：即使節點 repo 可用，所選 swap model、InsightFace 元件與下載權重未必可以商用；必須拆開審核。
- **來源**：[Gourieff/ComfyUI-ReActor](https://github.com/Gourieff/ComfyUI-ReActor)

### C. Rope：偏 GUI 的本機交換工具

- **定位**：以 InsightFace `inswapper_128` 為核心、偏 GUI 操作的專案；提供批次處理、遮罩、色彩匹配、likeness / fidelity 控制與桌面 capture 模式。
- **適合**：希望研究傳統換臉 pipeline 的可調參數與互動式 UI。
- **限制**：本質仍是傳統 face-swap；應把它視為本機實驗候選，而不是宣稱能解決所有長影片一致性問題。
- **來源**：[Hillobar/Rope](https://github.com/Hillobar/Rope)

### D. LivePortrait：目前最重要的「肖像動畫」替代路線

- **定位**：將來源肖像的外觀身份，與 driving video 的頭部姿態／表情結合；也支援 source video 的 v2v 編輯、姿態編輯與局部控制。
- **近期性**：2024 釋出後持續有功能更新；官方在 2025 年仍更新動物模型，並列出其已被多個短影音與創作平台採用。
- **優點**：比完整 diffusion 影片生成更即時、更可控，適合角色頭像、表情驅動短片與已同意的 avatar 素材。官方也提供 Apple Silicon 的人類模式支援。
- **限制**：不是全身生成器，也不是任意影片中的完整精準換臉。官方建議 driving video 聚焦頭部、第一幀接近正面中性臉、減少大幅肩部動作；違反這些條件時品質會明顯下降。Apple Silicon 人類模式可跑，但官方估計可能比 RTX 4090 慢約 20 倍。
- **授權提醒**：程式碼是 MIT，但 repo 特別提示其中 InsightFace detection models 是非商業研究用途；商用需要替換或取得適當授權。
- **來源**：[repo](https://github.com/KlingAIResearch/LivePortrait)｜[論文](https://arxiv.org/abs/2407.03168)

---

## 3. 生成式身份保留：研究前沿已經走到哪裡？

### A. ConsisID（CVPR 2025 Highlight）：參考臉 + prompt 生成一致人物影片

- **定位**：identity-preserving text-to-video（IPT2V）研究模型；不必對每個人另行微調，即可從參考人臉與文字提示生成保持身份特徵的影片。
- **技術重點**：以頻率分解的方式，把人臉全域特徵與細節特徵注入 Diffusion Transformer，目標是在生成影片中減少 identity drift。
- **近期性／可用性**：CVPR 2025 Highlight；官方 code / weights 已釋出，且 Hugging Face Diffusers 已提供 pipeline 文件。
- **現實限制**：這是研究級路線而非輕量 app。官方 repo 指出，720×480、49 frames（約 6 秒、8 FPS）的預設推論在無 offload 下最高約需 44GB VRAM；可透過 CPU offload、slicing、tiling 降低顯存，但會更慢且可能影響品質。
- **適合**：研究「把有權使用的人像轉為不同場景、服裝、鏡頭語言的新短片」，而不是要求原始 target video 每個像素都不動的換臉。
- **來源**：[論文](https://arxiv.org/abs/2411.17440)｜[官方 repo](https://github.com/PKU-YuanGroup/ConsisID)｜[Diffusers 文件](https://huggingface.co/docs/diffusers/main/en/using-diffusers/consisid)

### B. MimicMotion（ICML 2025）：全身姿態驅動的身份保留人物影片

- **定位**：單張人物圖片加上 pose sequence，產生全身人物動作影片。
- **能力**：官方 1.1 checkpoint 將最大輸出從 16 增至 72 frames，主打姿態置信度導引與時間一致性。
- **成本**：官方文件以 NVIDIA GPU 為主要環境；72-frame 版本約需 16GB VRAM，示例顯示 4090 生成 35 秒內容仍要約 20 分鐘，因此較適合離線生成，不是即時換臉。
- **限制**：複雜動作、手部、衣物、遮擋與輸入照片缺少的身體資訊仍可能出錯。身份一致性與動作遵從通常要取捨。
- **來源**：[repo](https://github.com/Tencent/MimicMotion)｜[論文](https://arxiv.org/abs/2406.19680)

### C. VACE / Wan 系列：把換臉放進「影片編輯」而非單一模型

- **定位**：近代影片 foundation-model 路線的價值在於多條件控制：文字、參考圖、首尾幀、影片、遮罩、姿態或局部編輯可被合併到同一生成流程。
- **意義**：若目標不是「只把臉貼上去」，而是要修正臉、換服裝、控制動作、改背景、維持片段連續性，這類 video-to-video / reference-to-video 模型通常比傳統 swap 更有上限。
- **成本／風險**：不同 checkpoint 的授權、硬體需求與模型品質差距很大；需要用自己的、已同意的測試素材建立 benchmark，而非單看示範影片。
- **來源**：[Wan-Video / Wan2.1](https://github.com/Wan-Video/Wan2.1)｜[VACE](https://github.com/ali-vilab/VACE)

### D. InfiniteYou：雖然是圖像而非影片，仍是重要「身份底圖」元件

- **定位**：ICCV 2025 Highlight 的 identity-preserved image generation，建立在 FLUX 的 DiT 上；可在角色創作前先生成更一致的角色 key art，再交給 LivePortrait 或 image-to-video 工作流。
- **為何列入**：生成影片品質高度依賴參考人像品質。先做出高品質、可控、視覺一致的角色 reference，往往比直接把一張隨手自拍丟進影片模型更穩。
- **硬體／授權**：完整 bf16 最高約 43GB VRAM；offload + 8-bit quantization 可降至約 16GB。程式碼 Apache-2.0，但模型為 CC BY-NC 4.0，且底層模型／臉部模型也必須遵循各自條款；不適合直接假設可商用。
- **來源**：[repo](https://github.com/bytedance/InfiniteYou)｜[論文](https://arxiv.org/abs/2503.16418)

---

## 4. 舊工具不是不能用，但不該當作新專案預設

| 工具 | 現況 | 2026 判斷 |
|---|---|---|
| DeepFaceLab | 歷史上影響力很大 | 適合理解早期訓練式 deepfake 流程；不是本報告建議的新專案預設 |
| roop | 作者停止開發並封存 repo | **不推薦新部署**；作為歷史或概念參考即可 |
| DeepFaceLive | 2024-11 已封存、唯讀 | 即時 webcam／直播換臉的歷史選項；缺少後續相容性與安全維護，不宜做新依賴 |
| SimSwap | 重要學術基線，公開更新主要落在 2023 前後 | 值得讀論文與做研究比較，但技術棧偏舊 |

- roop 的作者明確說明，因重新評估此類軟體的二階影響，已停止開發並封存 repo：[s0md3v/roop](https://github.com/s0md3v/roop)。
- DeepFaceLive 已標示 archived / read-only：[iperov/DeepFaceLive](https://github.com/iperov/DeepFaceLive)。
- SimSwap 論文與 code：[arXiv](https://arxiv.org/abs/2106.06340)｜[repo](https://github.com/neuralchen/SimSwap)。

---

## 5. 授權不是附註，而是選型條件

這個領域最常見的誤會是：**GitHub repo 開源，不代表你下載的權重、臉部辨識元件、資料集和輸出用途都可自由商用。**

特別要注意：

1. **InsightFace code 與模型的權利不同。** 官方說明指出：其訓練資料與預訓練模型僅供非商業研究用途；`inswapper` 系列需要另行取得商業授權。許多上層換臉工具直接或間接依賴它，因此不能只看 wrapper repo 的 license。
2. **一個 pipeline 通常有多層依賴。** 主程式、swap model、偵測／對齊模型、face restoration、base video model、LoRA、輸出素材與雲端服務條款都要單獨審核。
3. **「本地跑」不是法律豁免。** 它只代表沒有雲端平台即時把關；肖像同意、人格權、著作權、商標、誤導性廣告與平台政策仍然存在。
4. **要發布就做來源與揭露管理。** 保留同意紀錄、來源素材版本、模型／權重版本與輸出標示；對真人合成內容加上清楚、可見的 AI 處理揭露。

來源：[InsightFace 授權說明](https://github.com/deepinsight/insightface)｜[FaceFusion license](https://github.com/facefusion/facefusion/blob/master/LICENSE.md)

---

## 6. 依用途的選型建議

### 情境 1：已拍好的短片，要把「已同意」的人換成角色／演員替身

**首選：FaceFusion。**

理由：它直接處理既有影片，適合先做 source/target 對照測試、評估不同臉部角度、光線、遮擋與輸出編碼對品質的影響。若要多步修復、遮罩與生成背景，再考慮接進 ComfyUI-ReActor 工作流。

### 情境 2：只需要角色人像有自然表情、轉頭、微動作

**首選：LivePortrait。**

理由：這不是硬把臉貼到複雜影片上，而是從一張授權角色圖生成受 driving motion 控制的肖像動畫；速度、自然度與可控性通常更符合 avatar／短片開場需求。

### 情境 3：要讓同一個人物在新場景中做新動作，而非保留原鏡頭

**先研究：ConsisID；再視控制需求評估 Wan/VACE 路線。**

理由：這是身份保留影片生成的核心問題。要接受它是研究與生成工作流，而不是一鍵、完全可預測的換臉。

### 情境 4：全身跳舞、走路或姿態重演

**先研究：MimicMotion。**

理由：它明確處理 pose-conditioned human motion。但先以小片段和已取得同意的測試素材驗證，因為全身一致性、手部和衣物是最容易失敗的地方。

### 情境 5：要做長期、可商業化的角色／persona 資產

**不要從「誰能最快換臉」選工具，而要從可追溯性選。**

建議把人物設定為自有或明確授權的 synthetic identity，保留角色來源圖、模型權重與生成紀錄；優先選擇能清楚追蹤模型／素材權利的路線。這比依賴真人臉或來源不明的「無審查」服務更能長期經營。

---

## 7. 建議的安全評估方式（不使用真人第三方臉）

在決定採用前，做一個小型、可重複的 benchmark：

- **素材**：自己、明確書面同意的演員，或完全合成的角色；至少準備正臉、側臉、不同光線與含遮擋的短片。
- **指標**：身份相似度、時間閃爍、邊緣／膚色融合、口型／表情、運動一致性、輸出速度、VRAM、輸出檔案品質。
- **比較方式**：每個工具固定同一批測試素材與輸出規格；不要只看各 repo 的精選 demo。
- **記錄**：工具 commit/version、模型與權重版本、授權、硬體、參數概要、結果與失敗案例。

這樣才能回答對自己最有用的問題：*哪一條路線在我的硬體、我的鏡頭條件、我的授權邊界下真的能用？*

---

## 延伸來源（優先一手）

1. [FaceFusion repo](https://github.com/facefusion/facefusion) 與 [release notes](https://github.com/facefusion/facefusion/releases)
2. [ComfyUI-ReActor](https://github.com/Gourieff/ComfyUI-ReActor)
3. [Rope](https://github.com/Hillobar/Rope)
4. [LivePortrait repo](https://github.com/KlingAIResearch/LivePortrait)；[論文](https://arxiv.org/abs/2407.03168)
5. [ConsisID repo](https://github.com/PKU-YuanGroup/ConsisID)；[CVPR 2025 論文](https://arxiv.org/abs/2411.17440)；[Diffusers implementation](https://huggingface.co/docs/diffusers/main/en/using-diffusers/consisid)
6. [MimicMotion repo](https://github.com/Tencent/MimicMotion)；[論文](https://arxiv.org/abs/2406.19680)
7. [Wan2.1](https://github.com/Wan-Video/Wan2.1)；[VACE](https://github.com/ali-vilab/VACE)
8. [InfiniteYou](https://github.com/bytedance/InfiniteYou)；[論文](https://arxiv.org/abs/2503.16418)
9. [InsightFace](https://github.com/deepinsight/insightface)
10. [roop（已封存）](https://github.com/s0md3v/roop)；[DeepFaceLive（已封存）](https://github.com/iperov/DeepFaceLive)；[SimSwap](https://github.com/neuralchen/SimSwap)

---

*研究日期：2026-09-07。模型能力、release、權重授權與雲端服務政策變化很快；實作或商用前請重新核對官方 repo、模型卡與當前條款。*

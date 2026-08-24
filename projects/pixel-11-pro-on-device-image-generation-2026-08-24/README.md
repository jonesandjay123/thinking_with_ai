# Pixel 11 Pro 本地圖片生成：JoviYarn 即時故事插圖可行性調查

> 查詢日期：2026-08-24（America/New_York）  
> 對象：Pixel 11 Pro（題設：Tensor G6、16 GB RAM）、Android、JoviYarn。  
> 範圍：完全 on-device、可在完成模型 provisioning 後飛航模式運作的「文字／Story State → 小插圖」。不把 cloud image API 當作核心方案。  
> 證據標記：**[官方確認]**、**[論文／原始碼]**、**[實機證據]**、**[推論]**、**[未知]**。  

## Executive Summary

**最短答案：對「任意文字 prompt → 新的一張 256–512px raster 圖」而言，Pixel 11 Pro 平均 1–2 秒的結果是 `Maybe`，不是現在可以承諾的 `Yes`。**

理由不是 Tensor G6 不夠快，而是截至查詢日沒有一條同時滿足「可取得權重、Android/Tensor runtime 已驗證、商用授權清楚、Pixel 11 實測、含冷暖機與連續壓力的 1–2 秒」的完整證據鏈。最接近的研究證據是 SnapFusion 宣稱手機 <2 秒，以及 MobileDiffusion 宣稱「instant」mobile text-to-image；但前者的公開 repo 不是可直接 Android shipping 的完整權重/runtime bundle，後者沒有公開可直接部署的 Android/Tensor 交付物。**這些是研究上令人振奮的 proxy，不是 Pixel 11 benchmark。**

對 JoviYarn，我的產品判斷很明確：

1. **現在就 prototype：procedural scene card / pixel-art assets composition。**Gemini Nano 已可在本機把 Story State 轉成嚴格 scene JSON；Android Canvas/Compose/SVG 或預製 asset layer 幾十毫秒即可出圖、可離線、風格一致、幾乎不影響故事 loop。這才是真正能保證「故事世界同步長出來」的方案。
2. **以 diffusion 當可替換的 enhancement worker，不當核心。**先做一個 benchmark app，跑 MediaPipe Image Generator（SD 1.5）、`stable-diffusion.cpp`/NCNN，以及可取得的 few-step 權重；只有量到 P50/P95 才能決定是否進產品。
3. **不要把 Google runtime 和 Google image model 混為一談。**Google 有 LiteRT、MediaPipe、GPU/OpenCL/Vulkan 路徑等 runtime/工具；目前沒有一個像 AICore Gemini Nano 一樣、公開讓第三方 Android App 直接呼叫的 Google-managed 本地 text-to-image model。Nano Banana、Imagen、Gemini image generation 是 cloud 產品/API，不能作為 offline contract。
4. **pixel art prompt 本身不會讓 diffusion 幾乎免費。**若仍餵給同一個 512² SD UNet，即使最後畫風是 8-bit，每一步的主要運算量大致不變。真正省時間的是低解析、少步數、模型本身蒸餾／縮小，或根本不做 diffusion。
5. **買 Pixel 11 Pro 的價值仍很高，但不是因為已證明它能 1 秒生圖。**它是良好的 Android/Tensor 實驗機，尤其可讓 Nano 做 scene planning；圖像生成部分必須以實測取代推測。

### 建議的 JoviYarn 兩層體驗

```text
STT → Story State → Gemini Nano（scene JSON；已有本地路徑）
                         │
              立即：asset / SVG / Canvas renderer  ── 20–150 ms
                         │
              非阻塞：optional local diffusion     ── unknown；可能數秒
                         │
              失敗／熱／busy：保留立即 scene card，不影響敘事
```

這個設計不是退而求其次；對故事產品而言，快速、一致、可理解的視覺語言通常比每次隨機生成一張不穩定的小圖更有價值。

---

## 1. 硬體與 Android AI stack：能用什麼、不代表有什麼模型

題設中的 Pixel 11 Pro 是 Tensor G6、16 GB RAM。前一份 Pixel 11 調查已記錄 Google 的 consumer 說法是搭載最新 Gemini Nano；Jones 亦已在實機跑通本地 Nano 和繁中離線語音。這些事實對「Nano 做文字 scene planning」很重要，**但不構成** Tensor G6 有公開 image diffusion SDK 或 Google 下放 image model 的證據。

| 層 | Google／Android 是否提供 | 對自帶 image model 的意義 | 這不是什麼 |
|---|---|---|---|
| **LiteRT**（原 TensorFlow Lite） | **[官方確認]** Android runtime、CPU/GPU/accelerator delegation 與 model tooling | 可部署自己轉換／量化的模型；模型檔、記憶體、速度、授權由 app 負責 | Google 替你供應的文生圖模型 |
| **MediaPipe Tasks** | **[官方確認]** Android task framework；含 Image Generator task | 有現成 Java/Kotlin API 和 SD 1.5 conversion path | 長期維護、快速、Pixel 11 optimized 的保證 |
| **GPU / Vulkan / OpenCL** | **[官方確認]** Android 有 Vulkan；MediaPipe Image Generator 明列 Android 12+ 的 OpenCL native library manifest entries | 有可能讓 image pipeline 避開純 CPU；每條 delegate 必須逐機驗證 | Tensor TPU/NPU 一定會被 diffusion 使用 |
| **NNAPI** | **[官方確認]** Android 已將 NNAPI 標為 deprecated，鼓勵改用 LiteRT | 不應以新 NNAPI integration 作為 2026 新 PoC 主線 | 可移植、穩定的 generative acceleration contract |
| **Google Tensor SDK** | **[未知／勿假設]** 本次查核沒有找到面向第三方 Android App、可直接把任意 diffusion graph dispatch 至 Tensor G6 TPU 的公開 SDK contract | 把 LiteRT delegate profiling 做成真實驗證 | public NPU API 或 Google 提供的 image generator |
| **AICore + Gemini Nano** | **[官方確認（文字/ML Kit GenAI 路徑）**] | 很適合輸出 `SceneSpec`、tag、palette、asset selection、SVG DSL | 可呼叫的 local raster image-generation API |

### NNAPI 現況

Android 官方在 NNAPI 文件直接標示 deprecated，並推薦 LiteRT（含 Google Play services runtime 選項）。因此本報告不建議新 app 把「NNAPI support」視為成敗核心；應讓 candidate 用自己的 LiteRT/GPU/Vulkan/NCNN backend，並在 Pixel 11 實測實際 delegate 是否啟用。

### 三件常被混淆的事

1. **runtime/acceleration stack**：LiteRT、MediaPipe、Vulkan、OpenCL。它們是跑模型的道路。
2. **Google-managed model**：Gemini Nano（本案可供文字／multimodal API 的 system model）。截至本報告，沒有同等公開的本地 image model API。
3. **自帶 open-weight model**：SD 1.5、SD-Turbo、Tiny-SD、LCM 等。它們的權重、轉換、下載、license、更新、RAM、熱與 safety 都是 JoviYarn 的責任。

---

## 2. Google 官方本地 image generation 現況

### 2.1 結論：有 runtime，沒有可依賴的 Google 下放 image model

**[官方確認]** MediaPipe Android 的 Image Generator task 仍在文件中，能根據文字做 diffusion text-to-image、支援 LoRA 和 face/edge/depth condition plugins。但頁首明示：*task still available, but is no longer actively maintained*。它要求開發者下載與轉換 **Stable Diffusion v1.5 EMA-only** 相容 foundation model；production 時模型太大、不應 bundle 進 APK、建議 runtime download。這是很重要的原文級證據：

> **Google 提供 runtime / acceleration stack，但實際 image model 需要 developer 自行帶入。**

該 task 的 Java 原始碼還把 output 常數寫為 `512 × 512`；文件將 20 iterations 列為合理起點。這不是 256/384 dynamic fast-thumbnail API，也不是一個 Pixel 11 專用新模型。

### 2.2 MediaPipe Image Generator 的產品風險

- **[官方確認]** task 不再 actively maintained；不能把它當長期唯一 production backend。
- **[官方確認]** foundation model 需自行轉換，且須符合 SD 1.5 EMA-only 格式；外部模型授權由開發者負責。
- **[官方確認]** 使用受到 Generative AI Prohibited Use Policy 約束；模型本身另有 license。
- **[推論]** 20-step SD 1.5 的 512² 工作量與「每 1–2 秒一張」需求方向相反；就算 GPU 可跑，也會是 RAM/thermal 的重負載。沒有 Pixel 11 官方 latency 數據，不能填一個漂亮數字。

### 2.3 Nano Banana / Imagen / Gemini Image

**[官方確認／產品邊界]** 它們是 Google 的雲端 Gemini/Imagen image generation product surfaces，不是 Android AICore/ML Kit 提供給任意 app 的 offline text-to-image runtime。本研究沒有找到可讓 Android app 在飛航模式呼叫的公開 API、device matrix、model package 或 SDK sample。故：

- 可作為日後 **explicit opt-in cloud high-quality** 層的研究候選；
- **不可**作為本報告「本地 1–2 秒」的證據，也不可把 Pixel 的 consumer image feature 當 third-party API。

### 2.4 LiteRT、MediaPipe 與 Tensor SDK 的正確定位

LiteRT 可以承載/執行符合其 backend 的模型；它不會自動把 PyTorch diffusion pipeline 變快或自動吃到 Tensor TPU。MediaPipe 提供較完整的舊 SD 1.5 task，但已 deprecated from active maintenance。若有新的 Google AI Edge experimental sample，除非同時有公開文件、可取得 model、Android implementation 及 support statement，應視為實驗材料而非產品依賴；本次查核未找到可取代上述 task 的正式 Google on-device image generation API。

---

## 3. 候選技術比較（刻意把 unknown 留為 unknown）

下表的「速度」僅記錄來源明確說過的結果。除非欄位明示 Pixel/Tensor 實測，**絕不外推成 Pixel 11 Pro 成績**。model size 是權重／工作集的近似級別，不把 APK、tokenizer、VAE、runtime 和峰值 activation 混在一起。

| Model / approach | 模型大小 | runtime | 真實 mobile evidence | resolution / latency evidence | acceleration | Android maturity | license / 商用判斷 | JoviYarn suitability |
|---|---:|---|---|---|---|---|---|---|
| **MediaPipe Image Generator + SD 1.5** | 約 GB 級完整 pipeline；官方稱不宜 APK bundle | MediaPipe Tasks / OpenCL | **Android 官方 sample**，無 Pixel 11 benchmark | 固定 512²；官方建議起點 20 steps；時間 unknown | OpenCL path；實際 GPU unknown | 有 API、但 task 已不再 active maintenance | MediaPipe Apache-2.0；SD1.5 為 OpenRAIL-M | **Benchmark baseline**，不宜承諾即時 |
| **SnapFusion** | 論文稱高效 UNet；公開 shipping asset size unknown | 論文 PyTorch research code | **論文稱 mobile <2s**；未找到 Pixel/Tensor Android app | 512²、8 steps，論文的 <2s claim | paper mobile optimization | 無 turnkey Android packaging | repo 未宣告 SPDX；模型／資料權利需另查 | 研究上很強；交付風險高 |
| **MobileDiffusion** | paper model；公開下載/shipping size unknown | research implementation | **論文**：主張 instant mobile generation；本輪未驗證可引用的 device/time table | device/time/解析度 public evidence：unknown；不是 Android | backend details for shipping: unknown | 無 public Android/Tensor kit | 研究結果不等於可 redistribute license | 最佳「速度上限」研究方向；不可直接整合 |
| **SD-Turbo** | SD family，約 GB 級 FP weights，量化後仍需實測 | diffusers / custom C++ / conversion | 沒找到可信 Pixel/Tensor benchmark | 1–4 step 方法；桌面 demo 很快，**不代表手機** | backend-dependent | 無官方 Android product path | Stability AI Community License；先法律審查，非 Apache/MIT | 值得探索、不是本期保證 |
| **SD 1.5 + LCM-LoRA** | base SD1.5 + 小 LoRA；base 仍 GB 級 | diffusers / custom conversion | 論文少步有效；未找到 Pixel benchmark | LCM 2–4 step 論文；手機 latency unknown | backend-dependent | Android 需自己移植 | base OpenRAIL-M；LCM LoRA OpenRAIL++ | 品質/速度折衷 research candidate |
| **Tiny-SD** | 較小 UNet，但完整 text encoder/VAE 仍有成本 | diffusers/custom | 未找到可信 Android/Tensor data | model card examples；mobile latency unknown | backend-dependent | 無正式 Android runtime bundle | CreativeML OpenRAIL-M | 低記憶體方向；仍須 converter + bench |
| **stable-diffusion.cpp + SD/LCM/Turbo weights** | 取決於 GGUF/量化與模型 | C/C++ JNI；可探索 Vulkan 等 | open-source project 有跨平台/mobile intent；**未驗證 Pixel 11** | latency device/model dependent; unknown | CPU/Vulkan 等由 build/device 決定 | 比 Python 容易嵌入，但需 NDK engineering | code MIT；權重另計 | **Candidate B**：可控、適合 benchmark |
| **Stable-Diffusion-NCNN** | 同上 | ncnn / Vulkan | repo 明確支援 Android；無 Pixel 11 public number | unknown | Vulkan | native integration 工作量高 | code BSD-3-Clause；權重另計 | **Candidate C**：可先跑通，速度未知 |
| **AI + asset composition** | 依 asset pack；可從 MB 級開始 | Compose/Canvas/SVG | Android renderer 是成熟路徑 | 約 20–150 ms 為工程目標，須測 | GPU UI render | 很成熟 | 自製／取得 permissive asset 最可控 | **最適合核心產品** |
| **procedural pixel art / SVG** | 幾乎零模型；templates/assets | Canvas/Compose/SVG/AGSL | 不需 ML benchmark | 通常 frame-level 至數百 ms；內容複雜度決定 | GPU UI render/CPU | 很成熟 | 自有 code/asset 可 MIT/Apache | **最符合 2 秒必出圖** |

### 重要：表內 1–2 秒 evidence 的可信層級

- **SnapFusion**：論文標題即為 *Text-to-Image Diffusion Model on Mobile Devices within Two Seconds*；摘要說其 8 denoising steps 在 MS-COCO 指標優於 SD1.5 的 50 steps。這是優秀的研究證據，並不等於「任何 Android、尤其 Tensor G6、都小於 2 秒」。
- **MobileDiffusion**：論文公開摘要主張 instant mobile generation；但本輪沒有取得可核驗的 device/latency table 或 Android/Tensor delivery artifact。它可說明「物理上有機會」，不能拿來採購或排期保證。
- **SD-Turbo / LCM**：one-/few-step 是把迭代數降下來的關鍵；但 text encoder、UNet、VAE、memory bandwidth、quantization 和 backend compile 都仍會主導端上 E2E latency。A100/desktop demo 必須標作 desktop evidence。

---

## 4. 真正的 mobile evidence 與缺口

### 已找到的強證據

1. **[官方確認] MediaPipe Android sample**：Image Generator 有 Android sample/API，且公開要求 OpenCL native libs 和 SD 1.5 model conversion。它證明「Android 可以在本機跑 diffusion pipeline」，不證明速度。
2. **[論文] SnapFusion**：公開論文的目標/claim 是 mobile device <2s，架構/step distillation 是合理方向；公開 GitHub repo 存在，但沒有可直接 shipping 的 Android Kotlin reference、也沒有 Tensor benchmark。 
3. **[論文] MobileDiffusion**：研究展示 512² fast on mobile 的可能性；Android/Tensor deployment artifact 未見公開可驗證版本。
4. **[原始碼] stable-diffusion.cpp / Stable-Diffusion-NCNN**：兩者是可審查的 native code base；前者 MIT、後者 BSD-3-Clause。它們可做 Android NDK/Vulkan research，但 repo license 不能覆蓋任何下載的 Stable Diffusion weights。

### 缺口（不要用 marketing 補）

- **Pixel 11 Pro / Tensor G6 的 public diffusion benchmark：unknown。**本次沒有找到可信、可重現的 SD-Turbo/SnapFusion/LCM Pixel 11 數字。
- **Tensor TPU/NPU 執行上述 diffusion 的 public app contract：unknown。**LiteRT/MediaPipe 的可用性不等於 particular graph 會映射到 TPU。
- **background generation：不可假定。**Nano AICore 有 foreground constraint；自帶 native inference 雖可由 app control，但 Android 的 background execution、thermal throttling、memory pressure 與 process death 都要以 UX/WorkManager/foreground policy 設計，不能在故事進行中悄悄燒 30 秒 GPU。
- **「Pixel art 速度」benchmark：unknown。**大多公開結果量 photorealistic 512²，不是 JoviYarn 的小插圖 workload。

---

## 5. 降畫質、pixel art 與 latency：什麼真的會變快？

### 四種看似相似、實則不同的路徑

| 路徑 | 計算量會大幅下降嗎？ | 為何 | 對 JoviYarn 的判斷 |
|---|---|---|---|
| 1. 用大型 SD prompt 寫「pixel art, 8-bit」 | **通常不會** | 模型仍跑同一個 UNet、同樣解析度、同樣步數；只是輸出分布不同 | 不可把它當 latency 策略 |
| 2. 小型/蒸餾模型、低 resolution、1–4 steps | **可能會** | 直接減少 network FLOPs、latent spatial size、denoise iterations | 要 benchmark；品質/提示遵從會掉 |
| 3. 先 diffusion，最後 nearest-neighbor/palette quantize | **幾乎不會縮短生成** | 後處理快，主要 diffusion 已完成 | 可統一風格，但不是加速核心 |
| 4. asset/SVG/procedural renderer | **會大幅下降** | 沒有 latent denoising；是 deterministic draw/composition | 本案最好的即時核心 |

### 解析度的真實價值

對 latent diffusion 而言，從 512² 降到 256² 通常能大幅減少 spatial compute（理想情況近四分之一），但不是完整 E2E 必然 4×：text encoder、VAE、memory copy、scheduler 和 UI 仍有固定成本，且有些模型／task 強制 512²。MediaPipe Image Generator 的現行原始碼固定 output 512²，所以它正好不適合拿來驗證「256² 1 秒 story thumbnail」這條假設。

### 低畫質不是低品質

故事插圖的價值是「當下故事狀態有一個可辨識的舞台」：地下室、舊收音機、手電筒光、神祕氣氛。對此，固定 art direction、有限 palette、可重複的 object grammar 和從 state 穩定衍生的構圖，往往比泛用 diffusion 的寫實細節更能讓世界持續存在。

---

## 6. 非 diffusion 替代方案：更接近產品需求

### A. Nano + 素材拼裝（首選）

先準備一個可控 asset catalog，例如：

- 20–50 個背景（basement、forest、castle、sea、room）；
- silhouette character layer、道具（radio、umbrella、key）、light/weather/mood overlays；
- 8/16-bit tiles、palette、frame、caption badge；
- 每個 asset 有大小、anchor、z-index、mood、allowed style 的 metadata。

Gemini Nano 輸出受 schema 約束的資料，而不是 SVG/程式任意碼：

```json
{
  "background": "basement_dark",
  "subject": "old_radio",
  "subjectPosition": "lower_right",
  "lighting": "flashlight_cone",
  "mood": "mysterious",
  "palette": "desaturated_blue",
  "caption": "牆角的收音機仍亮著。"
}
```

renderer 只接受 allowlist enum，將 2–6 層圖片／vector 合成。**[推論，但工程上高信心]**：這可在數十到數百 ms 內產生可用圖，RAM/thermal 遠低於 diffusion，且能在 Nano busy 時用 deterministic rules 產生同一份 JSON 的 fallback。

優點：速度、可預測、畫風一致、可控 safety、可下載 asset pack、可完整 offline。缺點：視覺組合有限、需要 art pack 設計、不是無限創意。

### B. Procedural illustration / SVG scene card（同樣強烈推薦）

將 `SceneSpec` map 到幾何 primitive：背景矩形/gradient、牆磚、收音機圓角矩形與 knob、cone light、noise/dither、月光／雨線。使用 Compose Canvas、Android Canvas、SVG renderer 或 AGSL shader；輸出可以是畫面本身或 snapshot bitmap。

這比讓 LLM 直接輸出任意 SVG 更安全：LLM 只選有限 schema；renderer 才擁有視覺 grammar。好處是：

- 2 秒 deadline 幾乎可保證；
- 可將故事人物／物件維持 identity（同一 `objectId` 同一 seed/shape variant）；
- 不需 model download，也沒有 diffusion content risk；
- 失敗模式容易診斷（缺 asset → placeholder），不會生出無關圖。

### C. Hybrid：instant local + user-demand cloud

產品可先顯示 local scene card；只有使用者點「畫成正式插圖」且同意網路時，才以 cloud model 作高品質版本。這將 latency、成本、privacy 和服務不可用性從主敘事 loop 分離。Cloud 不該默默 upload 故事內容；需要明確 consent、隱私說明和取消機制。

---

## 7. JoviYarn 架構選項與操作約束

### 建議架構：視覺是可取消、可降級的 sidecar

```text
local STT → append transcript → canonical Story State
                                │
                      Nano SceneSpec patch (short, debounced)
                                │
                  ┌─────────────┴─────────────┐
                  │                           │
              immediate renderer         optional generator queue
            assets / SVG / Canvas       diffusion only when idle/allowed
                  │                           │
              always show card         replace only if succeeds
```

規則：

- 不在每個 partial transcript 觸發。只在 final chunk、scene change、停頓或使用者選「visualize」時觸發。
- single-flight：新 scene 來了就取消/丟棄舊 diffusion job 的結果，避免舊故事畫面晚到。
- 視覺 worker 可被 memory/thermal/battery policy 關閉；主故事、STT、Nano suggestions 不受影響。
- 先用 256/384 scene card 當產品輸出；512² 若要測，獨立列為 benchmark，不要讓它拖住交互。
- 保存 `SceneSpec + seed + asset version`，而不是只存 bitmap；這讓畫面可重現並支持未來以更好 backend rerender。

### 「地下室與收音機」應怎麼呈現？

1. Nano 從 story state 選 `basement_dark + old_radio + flashlight_cone + mysterious`。
2. 150 ms 內 compositing 層顯示限色 pixel-art card（牆角、收音機、光束）。
3. 若使用者在 Wi-Fi/充電／明確點選高品質，才送入 chosen local diffusion 或 cloud enhancement。
4. 如果 local diffusion 超過 2 秒、被取消、OOM 或熱節流，scene card 已完成，不顯示錯誤打斷故事。

---

## 8. Top 3 Pixel 11 Pro 實驗候選

### Candidate A — 最可能達成「1–2 秒體感」：Nano SceneSpec + asset/SVG renderer

- **model/runtime**：ML Kit GenAI Prompt API（既有 Nano path）+ Jetpack Compose Canvas / SVG / 自製 assets。
- **repository**：不依賴 image model repo；renderer 應隨 JoviYarn source 管理。Nano 入口見 [ML Kit GenAI samples](https://github.com/googlesamples/mlkit/tree/master/android/genai)。
- **模型大小/RAM**：無 image model；asset pack 可從數 MB 起。Nano 是 system-managed，app 不包模型。
- **預估速度**：render 20–150 ms 是需驗證的工程目標；端到端主要取決於 Nano SceneSpec，不能把既有 Nano latency 自動算進來。
- **品質**：stylized、可讀、風格最穩；不是自由 photorealistic generation。
- **主要風險**：asset coverage 和 prompt→schema quality；需規劃 visual grammar。
- **建議測試**：256²、384²、512² 三種 snapshot；先 30 個固定 story fixtures；測 scene consistency/asset miss rate。
- **是否值得 benchmark app**：**是，第一優先。**它會直接決定產品方向。

### Candidate B — 品質／速度最平衡的真正 generative 實驗：`stable-diffusion.cpp` + SD 1.5 LCM / SD-Turbo（法律核准後）

- **model/runtime**：[stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp)（MIT code）+ 可合法取得的 SD 1.5 base 與 LCM-LoRA，或 SD-Turbo；Android 以 NDK/JNI，探索 Vulkan backend。
- **模型大小/RAM**：unknown，取決於 quantization、VAE、text encoder、backend；預期仍是數百 MB 至數 GB download／高峰值 RAM，不能拿 16 GB 直接當可用額度。
- **預估速度**：**unknown on Pixel 11**。2–4 steps、256/384² 是合理 bench grid，不是承諾。
- **品質**：比素材組合自由；pixel-art prompt 可作 style，但要另作 palette postprocess。
- **主要風險**：Android build/ABI、Vulkan driver compatibility、OOM/thermal、權重授權、沒有 Tensor-specific optimization。
- **建議測試**：256²、384²；LCM 2/4 steps；Turbo 1/2/4 steps；固定 10 prompts × warm/cold。
- **是否值得 benchmark app**：**是，第二優先。**先做最小 native harness，不進 JoviYarn UI。

### Candidate C — 最容易先跑 Android proof：MediaPipe Image Generator + converted SD 1.5

- **model/runtime**：[MediaPipe Image Generator Android](https://developers.google.com/edge/mediapipe/solutions/vision/image_generator/android)，`com.google.mediapipe:tasks-vision-image-generator`。
- **repository**：[MediaPipe](https://github.com/google-ai-edge/mediapipe)（Apache-2.0 code；model license separate）。
- **模型大小/RAM**：官方明說 foundation model 太大，不適合 APK bundle；exact converted package/RAM unknown。
- **預估速度**：unknown；現有 task 固定 512² 且 docs 建議 20 iterations，故本報告預期它更像 correctness/compatibility baseline，不像 1–2 秒 winner。
- **品質**：成熟 SD1.5 baseline，可加 LoRA/condition image。
- **主要風險**：task no longer actively maintained、轉檔流程、512² fixed、OpenCL path device variance、commercial weight restrictions。
- **建議測試**：512²、4/8/20 steps（若品質/後端允許）；只量測，不先做產品依賴。
- **是否值得 benchmark app**：**是，第三優先**，因為最快確認 Android compatibility，但別因 sample 能跑就認定產品可行。

> 未入選但值得追蹤：SnapFusion 與 MobileDiffusion。前者是最直接的 mobile-diffusion research result，後者是極具吸引力的 latency proxy；兩者都缺足夠的 public Android/Tensor shipping path，故不應讓工程排程依賴它們。

---

## 9. 最小 Benchmark App 設計（提案，不在本輪實作）

### 目標

建立能比較「真實 E2E latency、穩定性、熱與功耗」的 Android app，不測一次漂亮數字便宣布成功。

### 固定輸入

- prompt A：`pixel art, dark basement, old radio in the corner, flashlight cone, mysterious, limited blue palette`
- prompt B：角色 + 道具 + 場景，測 prompt adherence。
- prompt C：繁中同義 prompt，測 tokenizer/model language effect。
- fixed seeds；各 candidate 記錄 model revision、checksum、runtime/delegate、steps、CFG、scheduler、resolution、quantization。

### 每個 configuration 的流程

1. 開機/force-stop 後測 **cold init**（model load、first generation）。
2. warm-up 一次不計入主要統計。
3. 連續生成 10 次，記錄 `TTFT/first preview`（若有）、E2E done、mean、median、P95、success/error/OOM/cancel。
4. resolution grid：256²、384²、512²；step grid：1/2/4/8（MediaPipe 另按它實際允許值）。
5. 量 Java/native heap、RSS/PSS（可取時）、storage、GPU/CPU utilization（可取時）、battery level/current/temperature、thermal status、skin/battery temperature、是否降頻。
6. 20–30 分鐘 loop：前景故事 app 模擬每 30–60 秒 scene request；不要只做 back-to-back synthetic burn。記錄裝置可用性與主 UI jank。
7. 先在 online 完成 model download；之後 **airplane mode + app restart** 重跑。另分開測「全新/模型未下載的 airplane mode」失敗路徑。
8. background/foreground：不假定 background 可跑。測 Home、screen-off、notification interruption、low-battery；記錄 OS/process 行為，並以產品 policy 而非強行繞過。

### 成功門檻（建議）

| 目的 | 可接受門檻 |
|---|---|
| instant scene card | P95 < 500 ms（不含 Nano 的資料已可得時）；0 crash；視覺可讀 |
| optional local diffusion | warm median < 2 s 才進入「可考慮自動顯示」；否則只 user-triggered |
| sustained story session | 30 分鐘沒有 persistent OOM/crash/主 UI 明顯 jank；熱/電量有可接受曲線 |
| offline claim | 已 provisioned restart + airplane mode 10/10 成功；首次未下載情境必須誠實顯示 setup |
| license gate | weights、code、asset、redistribution 和 app store distribution 全部有書面可追溯答案 |

影像品質不應只靠 FID/CLIP：請讓 2–3 位人針對「是否看得出場景、關鍵物件、氣氛、是否和故事 state 一致」做盲評，並保存生成圖與 config。

---

## 10. License / distribution 風險清單

| 項目 | code license | weight/license 情況 | 對公開 app 的意思 |
|---|---|---|---|
| MediaPipe | Apache-2.0 | SD 1.5 model 為 CreativeML OpenRAIL-M | runtime permissive 不等於權重 permissive；需遵守 model use restrictions/notice |
| stable-diffusion.cpp | MIT | 外接權重另計 | 可商用程式碼不讓你自動有權 redistribute weights |
| Stable-Diffusion-NCNN | BSD-3-Clause | 外接權重另計 | 同上；注意 BSD notice |
| SD-Turbo | runtime implementation varies | Stability AI Community License / model card 為準 | **不可預設 Apache/MIT**；商用門檻、禁止用途、redistribution 必須法務/授權審查 |
| Tiny-SD | implementation varies | model card 為 CreativeML OpenRAIL-M | 不是 permissive weight；要保存 model card/revision |
| SnapFusion | public repo 未見 SPDX declaration | research code/weights/distribution terms需逐項確認 | **不能因 GitHub 可 clone 就當可商用 bundle** |
| MobileDiffusion | public deployable artifact unknown | unknown | 不可列入 shipping dependency |
| 自製 assets/SVG | 可自選 MIT/Apache 或 proprietary | 最可控 | 建議 asset provenance manifest 和 attribution 檔 |

除 license 外，還有：模型 safety/prohibited-use 條款、使用者故事可能含敏感內容、模型下載 CDN/隱私、export controls、app store model disclosure 和 model security updates。若未來公開發布，應把 model checksum、source URL、license text、attribution、download consent 和 rollback strategy 一起放進 release process。

---

## 11. 最終建議

### 1. Pixel 11 Pro 平均 1–2 秒本地生成簡單圖片，realistically achievable？

**答案：Maybe。**

- **可能做到**：特化/蒸餾/few-step image model，在理想 warm、短 prompt、低 resolution、特定 backend 下；研究已證明此級別不是物理不可能。
- **尚未證明**：Pixel 11 Pro/Tensor G6 對任何可公開 shipping 的 candidate 之 P50/P95；更沒有電池、thermal、連續故事 session 證據。
- **可以肯定做到的使用者體感**：若將「簡單圖片」定義為 story scene card/pixel-art composition/procedural image，而非每張全新 diffusion raster，則 **Yes**，而且更適合產品。

### 2. 若要追求 full image generation，最可能路徑？

先以 **`stable-diffusion.cpp` + 合法 SD1.5 LCM/SD-Turbo** 做可控 native benchmark；同時以 **MediaPipe Image Generator** 建 Android correctess baseline。前者較有機會切到低步數／低解析，後者最快驗證官方 Android task 能否在本機跑。兩者皆不應在測出數據前宣傳 1–2 秒。

### 3. 真正瓶頸是什麼？

不是單一 NPU 峰值。是「可用模型 + Android backend + mobile memory + model loading + VAE/text encoder + few-step quality + Tensor driver mapping + thermal/sustained workload + redistribution license」的乘積。任何一項 unknown 都足以令 demo 變成不可發布產品。

### 4. 現在應該做什麼？

**立即做 procedural/asset composition prototype；並平行做三個 benchmark candidate。**不要等更好的模型才讓故事世界有視覺，也不要把 core UX 綁在 cloud。待 2026 的 Pixel 11 實機 P95/thermal/license 都過門檻後，再把 local diffusion 升級為 optional enhancement；需要高品質時，採明確使用者同意的 cloud hybrid。

---

## Sources（優先一手來源）

### Google / Android official

1. [MediaPipe Image Generator for Android（deprecated notice、SD1.5 conversion、OpenCL、512²/iterations/API）](https://developers.google.com/edge/mediapipe/solutions/vision/image_generator/android)（查詢：2026-08-24）
2. [MediaPipe ImageGenerator Java source（`GENERATED_IMAGE_WIDTH/HEIGHT = 512`）](https://github.com/google-ai-edge/mediapipe/blob/master/mediapipe/tasks/java/com/google/mediapipe/tasks/vision/imagegenerator/ImageGenerator.java)
3. [MediaPipe repository（Apache-2.0）](https://github.com/google-ai-edge/mediapipe)
4. [LiteRT overview](https://developers.google.com/edge/litert)
5. [LiteRT on Android](https://developers.google.com/edge/litert/android)
6. [Android NNAPI documentation（deprecated）](https://developer.android.com/ndk/guides/neuralnetworks)
7. [ML Kit GenAI Android samples](https://github.com/googlesamples/mlkit/tree/master/android/genai)
8. [ML Kit GenAI overview（AICore / Gemini Nano device/API boundary）](https://developers.google.com/ml-kit/genai)

### Models, papers, and source repositories

9. [SnapFusion paper, arXiv:2306.00980](https://arxiv.org/abs/2306.00980) — mobile <2s claim 是論文作者的實驗主張，非 Pixel evidence。
10. [SnapFusion project repository](https://github.com/snap-research/SnapFusion) — check its actual code/weights terms before any use.
11. [MobileDiffusion: Instant Text-to-Image Generation on Mobile Devices（Springer DOI）](https://doi.org/10.1007/978-3-031-73033-7_13) — 不把其特定手機成績外推至 Tensor；本輪未找到可公開下載的 Android/Tensor shipping artifact。
12. [Latent Consistency Models, arXiv:2310.04378](https://arxiv.org/abs/2310.04378)
13. [SD-Turbo model card](https://huggingface.co/stabilityai/sd-turbo) — 必讀 `LICENSE.md`，勿只看 repo code license。
14. [Stable Diffusion v1.5 model card](https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-v1-5)
15. [Tiny-SD model card](https://huggingface.co/segmind/tiny-sd)
16. [LCM-LoRA SD1.5 model card](https://huggingface.co/latent-consistency/lcm-lora-sdv1-5)
17. [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp)（MIT code; model terms separate）
18. [Stable-Diffusion-NCNN](https://github.com/EdVince/Stable-Diffusion-NCNN)（BSD-3-Clause code; model terms separate）

### Related repository context

19. [Pixel 11 Pro AI capability map（本 repo，Nano/STT 已實機確認背景）](../pixel-11-pro-ai-capability-map-2026-08-23/)
20. [Pixel 11 on-device AI for JoviYarn（本 repo，AICore/Prompt API/限制調查）](../pixel-11-on-device-ai-for-joviyarn-2026-08-22/)

---

## Concise decision record

| 問題 | 決策 |
|---|---|
| 主故事 loop 要不要等 local diffusion？ | **不要。**先 shipping-quality scene card renderer。 |
| 要不要今天做 full diffusion product implementation？ | **不要。**只做 isolated benchmark，先得出 Pixel P50/P95/thermal/license。 |
| Google Nano 能不能直接生本地 raster 圖？ | **本次查核未找到公開 third-party API；不要假設。** |
| 最值得拿 Pixel 11 Pro 實測的三項？ | A: Nano→SceneSpec→renderer；B: stable-diffusion.cpp+few-step；C: MediaPipe SD1.5 baseline。 |
| 今日最合理的產品路線？ | **local instant composition + optional local diffusion + explicit cloud high-quality hybrid。** |

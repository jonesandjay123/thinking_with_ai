# Strict Local Editing / Immutable Anchors Research

日期：2026-06-21  
主題：在 `[A][X][B]` 工作圖中，只允許 X 被生成或修改，A/B 必須完全不可動。

## TL;DR

這次需求要重新定義成工程硬規格，不是一般「局部修圖」：

```text
A = immutable left anchor
B = immutable right anchor
X = only editable/generated region

Required:
pixel_diff(A_out, A_original) = 0
pixel_diff(B_out, B_original) = 0
```

在這個標準下，最務實的第一步不是追求全自動，而是把 **Photoshop manual baseline + repo hard composite** 定成第一版 workflow。

原因：

- 新場景不是高頻批量生成，而是像手機遊戲新活動 / 新季節場景更新：一次只做一兩個，但每次都是大工程。
- Human-in-the-loop 可以接受，甚至合理。
- Photoshop Generative Fill 目前仍是成熟、可用、可人工精準選區的 baseline；缺點是 Adobe subscription / generative credits 成本，以及 GUI 流程不適合直接批量化。
- 不管用 Photoshop、Firefly、Midjourney、OpenAI、FLUX、ComfyUI，最後都不應該盲信整張 raster。最硬的工程保證是：**A/B 永遠從原圖取回，AI 只提供 X layer / X patch。**

建議優先順序：

1. **Photoshop manual baseline + repo composite**  
   人工框選 X，用 Generative Fill 產生 X，最後由 repo 腳本把 X 合回原始 `[A][B]`，驗證 A/B 0 diff。
2. **Adobe Firefly API / Photoshop API semi-automation**  
   若 Photoshop manual 成功，再研究如何用 Adobe Firefly Services / Photoshop API 把 PSD/layer/mask/composite 流程半自動化。
3. **ComfyUI / Diffusers / FLUX Fill engineering replacement**  
   之後若要降低人力，再用 mask-based inpainting backend 取代 Photoshop 的 X 生成，但接受標準仍然是 A/B 0 diff。

Midjourney Vary Region / Editor 可以作為人工創意工具，但不是第一 baseline。OpenAI Image Edit API 可試，但官方文件明確說 GPT Image mask 是 prompt-based guidance，不適合當 strict selected-region guarantee。

## 需求重述

我們不是在找「最好看的生圖模型」，也不是在找「大致保留原圖」的 image-to-image。

真正問題是：

```text
Given: [A][X][B]
Only X may change.
A and B are immutable anchors.
```

這是一個 scene insertion / event update pipeline 的問題。每次新增新場景，例如聖誕節、節慶活動、新世界觀，不只是換圖，而可能包含：

- 專屬 item
- 3D 建模
- 後台程式
- 場景資料
- 整套導入與驗收

所以這不是「每天批量生成 30 張圖」的問題。只要能可靠地建立一個新場景，即使用人工修圖也可接受。

本案成功標準：

1. X 視覺上自然、可用。
2. A/B 不只是看起來一樣，而是實際像素不變。
3. 導入 pipeline 時不需要重新修舊邊界。
4. 最終輸出可以被無腦接回既有系統。

## 概念邊界

### 普通 image edit / image-to-image

不符合本案核心需求。這類模型會把整張圖重新融合，即使看起來接近，也可能導致 A/B 像素漂移。

### Selected-region editing / Vary Region

使用者在 GUI 中選區，讓模型只改那一塊。這類工具對人工流程很有價值：

- Photoshop Generative Fill
- Adobe Firefly Web Generative Fill
- Midjourney Vary Region / Editor

但 GUI selected-region 不等於工程上可以直接信任整張輸出。若要進 pipeline，仍要抽出 X，並把 A/B 從原始圖重組回來。

### Mask-based inpainting

程式化版本的 selected-region editing。輸入 image + mask + prompt，由模型補 mask 區。

代表：

- Adobe Firefly API
- FLUX Fill
- ComfyUI / Diffusers inpainting
- Stability AI Inpaint
- OpenAI Image Edit API

它們是未來工程化候選，但本案仍要把 final composite 和 pixel-diff verification 放在模型後面。

### Immutable anchors

這是本案最重要的工程語言：

```text
A/B are not "background context".
A/B are immutable anchors.
```

所以任何工具只要不能保證 A/B 不動，就只能被當成 X generator，不能被當成 final image generator。

## 最新候選工具評估

| 方案 | 最新可用狀態 | 適合人工 | 適合工程化 | 是否可作第一 baseline | 對本案判斷 |
| --- | --- | --- | --- | --- | --- |
| Photoshop Generative Fill | 官方仍支援 selection + non-destructive Generative Fill；可選 Firefly/partner models；使用 generative credits | 很高 | 中低 | 是 | 第一 baseline |
| Adobe Firefly Web | 支援 brush selection / Generative Fill；同 Adobe credits 生態 | 高 | 低 | 可作人工備援 | 人工 baseline 2 |
| Adobe Firefly API | 官方 mask 文件明確：black protected, white exposed | 低 | 高 | 第二階段 | 最值得半自動化 |
| Photoshop API / Firefly Services | 有 Photoshop API、Smart Objects、PSD/layer automation | 低 | 中高 | 第二階段 | 適合後續把 Photoshop workflow 產品化 |
| Midjourney Vary Region / Editor | Vary Region / Editor 仍存在；官方說可選區修改且不改 rest | 高 | 低 | 否 | 創意工具，不是 strict pipeline |
| OpenAI Image Edit API | 支援 mask，但官方說 GPT Image mask 是 prompt-based guidance，不精準保證 | 中 | 高 | 否 | 只能當 X candidate generator |
| FLUX.1 Fill | BFL 官方支援 mask / alpha；black preserve, white inpaint | 中 | 高 | 第三階段 | 高品質 X 生成候選 |
| ComfyUI / Diffusers | 成熟 mask inpainting 工作台 / Python pipeline | 中 | 高 | 第三階段 | 工程替代路線 |
| Stability AI Inpaint | API 支援 mask 或 alpha channel | 低 | 高 | 第三階段 | 可作 API 比較 |
| Runway | 目前官方重點偏 video inpainting / video API | 中 | 中 | 否 | 不適合本案第一線 |

## 方案 1：Photoshop manual baseline

### 最新資料

Adobe Photoshop 官方文件顯示 Generative Fill 的流程仍是：先用 selection tool 定義要編修的區域，再使用 Generative Fill 以 prompt 修改；文件也說這是 non-destructive workflow，會建立 generative layer。來源：<https://helpx.adobe.com/photoshop/desktop/create-open-import-images/create-images/edit-images-with-generative-fill.html>

Adobe 產品頁也將 Photoshop Generative Fill 描述為 Firefly-powered、可用 prompt 加入/移除內容、non-destructive 的工具。來源：<https://www.adobe.com/products/photoshop/generative-fill.html>

Adobe generative credits FAQ 說 generative credits 像 token，可用於 Photoshop、Illustrator、Lightroom、Firefly 等 Creative Cloud 產品中的 high-quality image/vector/video/audio outputs。來源：<https://helpx.adobe.com/creative-cloud/apps/generative-ai/generative-credits-faq.html>

### 對本案的判斷

Photoshop 現在應該升級成 **第一 baseline**，不是備案。

理由：

- Jones 已親眼知道兩年前 Photoshop 就能做到選區式生成，這不是假設。
- 最新官方文件仍顯示它支援 selection-based Generative Fill。
- 它是人工可控度最高的 route：人可以精準框 X、看 variation、反覆調 prompt。
- 本案不是高頻批量生圖，人工成本可以接受。
- 只要最後 A/B 從原圖回貼，Photoshop raster 是否微動 A/B 就不再是致命風險。

### 建議 Photoshop baseline workflow

```text
Input:
  original_work_canvas.png  # [A][X placeholder][B]
  x-mask.png                # only X = white / selected

Photoshop:
  1. Open original_work_canvas.png.
  2. Duplicate base layer; lock/hide original base as immutable reference.
  3. Create a precise X selection. Do not include A/B pixels unless they are intentionally part of the editable transition band.
  4. Run Generative Fill on X only.
  5. Pick a usable variation.
  6. Keep generated result as a separate layer / generative layer / masked layer.
  7. Export either:
     a. full PSD with layers, or
     b. generated-X-only PNG with alpha, or
     c. full preview PNG plus X mask.

Repo:
  8. final = original A/B + generated X through hard mask.
  9. verify pixel_diff outside X hard mask = 0.
  10. Only final-composited PNG is accepted into pipeline.
```

最重要的操作原則：

```text
Do not trust full Photoshop export as final.
Trust original A/B + generated X composite.
```

如果 Photoshop 只能輸出整張圖，也可以接受；repo 端再用 mask 只取 X，把 A/B 丟掉，回到原圖 A/B。

### Photoshop 的限制

- 需要 Adobe subscription / generative credits。
- GUI 人工操作不適合大量批次。
- 不能直接把「Photoshop 看起來不動 A/B」當成 pixel guarantee，仍要 repo diff。

但這些限制都可以接受，因為本案不是批量生產，而是重大場景更新。

## 方案 2：Adobe Firefly Web / Firefly API / Photoshop API

### 最新資料

Adobe Firefly Web 的 Generative Fill 仍支援 upload image、brush over edit area、以 prompt fill。來源：<https://helpx.adobe.com/firefly/web/edit-images/prompt-to-edit/generative-fill.html>

Adobe Firefly API mask 文件明確說，mask 是 applied to an image 的 grayscale overlay；black areas are protected，white areas are exposed；requested edits apply only to exposed regions。來源：<https://developer.adobe.com/firefly-services/docs/firefly-api/guides/concepts/masking/>

Adobe Firefly Services 文件說 Firefly Services 包含 Firefly APIs、Lightroom APIs、Photoshop APIs、Content Tagging APIs，結合 Photoshop/Lightroom 等 creative tools 與 Generative Fill、Text to Image 等 AI/ML features。來源：<https://developer.adobe.com/firefly-services/docs/guides/>

Photoshop API 文件顯示它提供 programmable layer automation、Smart Objects、Remove Background、Product Crop 等 endpoint；Smart Objects API 可在 PSD 中替換 embedded smart object 並維持 bounds。來源：<https://developer.adobe.com/firefly-services/docs/photoshop/>、<https://developer.adobe.com/firefly-services/docs/photoshop/guides/smart-objects-and-the-api/>

### 對本案的判斷

Adobe API 生態是 Photoshop baseline 成功後的第一個工程化方向。

可能路線：

1. Firefly API 直接做 mask-based generative fill。
2. Photoshop API / PSD layer workflow 管理 base layer、X layer、Smart Object、export。
3. 本地 repo 腳本做 final composite + pixel diff。

這條路最符合「human-in-the-loop 但可逐步工程化」：

- 前期：Photoshop 人工做 X。
- 中期：Photoshop/Firefly API 協助建立 PSD/layer/export。
- 後期：repo 自動 composite / diff / artifact management。

注意：即使 Firefly mask 文件語義很強，本案仍應把 A/B 0 diff 當 repo postprocess 的責任，而不是 API 的承諾。

## 方案 3：Midjourney Vary Region / Editor

### 最新資料

Midjourney Vary Region 官方文件說，Vary Region 是 Discord 中的 feature，可選擇 Midjourney image 的部分並修改，而不改變 rest of the image。來源：<https://docs.midjourney.com/hc/en-us/articles/32794723105549-Vary-Region>

Midjourney Editor 官方文件顯示 web Editor 可用 Smart Select 建立 selection mask，並可 erase selected area；Editor 也支援 export / download。來源：<https://docs.midjourney.com/hc/en-us/articles/32764383466893-Editor>

Midjourney modifying creations 文件也說 Vary Region/Erase 適合 editing specific parts of your image。來源：<https://docs.midjourney.com/hc/en-us/articles/33329329805581-Modifying-Your-Creations>

Midjourney 2026 version docs 顯示 V8.1 已於 2026-06-10 成為 default version。來源：<https://docs.midjourney.com/hc/en-us/articles/32199405667853-Version>

### 對本案的判斷

Midjourney 確實仍有 Jones 記憶中的 selected-region 能力，而且已演進到 web Editor / Smart Select。

但它不適合當第一 baseline：

- 沒有穩定正式 API 可直接接 repo pipeline。
- 它是創作工具，不是 layer/composite/pixel-diff workflow。
- 即使 UI 說不改 rest，我們仍無法把整張 export 當 A/B immutable guarantee。
- 外部圖片編輯和 Midjourney 版本/帳號/權限會影響可用性。

可用定位：

- 人工探索 X 的視覺方向。
- 生成 concept reference。
- 不是 final production workflow。

若用 Midjourney，也應該只取 X，A/B 仍由原圖 composite 回來。

## 方案 4：OpenAI Image Edit API

### 最新資料

OpenAI image generation guide 顯示可以提供 mask 指出要編修的部分；但文件明確說 GPT Image 的 masking 是 prompt-based，模型會把 mask 當 guidance，但可能不完全精準遵守 mask shape。來源：<https://developers.openai.com/api/docs/guides/image-generation>

OpenAI Image Edit API reference 提供 `POST /images/edits`，支援 image edit、mask、output format 等參數。來源：<https://developers.openai.com/api/reference/resources/images/methods/edit/>

### 對本案的判斷

OpenAI 不適合當 strict local editing baseline。

原因不是它不能編修圖片，而是它的 mask 不是硬幾何約束。官方已經明說 mask 是 prompt guidance。因此：

- 可以用 OpenAI 產生 X candidate。
- 不可把 OpenAI 輸出的整張圖直接當 final。
- 一律要用 original A/B + X patch 做 composite。

OpenAI 的價值：

- API 接入快。
- prompt understanding 強。
- 可作為快速外部候選。

但如果本案核心是 A/B immutable，它只能排在 Photoshop/Adobe/ComfyUI 之後。

## 方案 5：FLUX.1 Fill

### 最新資料

Black Forest Labs FLUX.1 Fill 官方文件說，FLUX.1 Fill 是 text-driven inpainting/outpainting，可用於 edit image regions 或 extend borders；支援 separate mask 或 alpha channel。使用 mask 時 black areas preserve original，white areas get inpainted。來源：<https://docs.bfl.ml/flux_1_fill>

Hugging Face 的 `FLUX.1-Fill-dev` model card 說它是 12B rectified flow transformer，可根據文字描述填補 existing images 的區域，並提供 Diffusers usage。來源：<https://huggingface.co/black-forest-labs/FLUX.1-Fill-dev>

ComfyUI 官方也有 Flux.1 fill dev inpainting / outpainting workflow。來源：<https://docs.comfy.org/tutorials/flux/flux-1-fill-dev>

### 對本案的判斷

FLUX Fill 是最值得在工程化階段驗證的高品質 X generator。

優點：

- 官方就是 inpainting/outpainting 模型。
- mask / alpha 語義清楚。
- 對自然過渡、光線、材質可能比傳統 SDXL inpaint 更強。

限制：

- 本地 dev model 重，需要工作流配置與 VRAM 考量。
- ComfyUI workflow 仍可能遇到版本/節點問題，需驗證。
- 即使官方寫 preserve，本案仍要 final hard composite。

最佳定位：

```text
Photoshop baseline 成功後，
用 FLUX Fill 嘗試替代 Photoshop 的 X generation，
但保留同一套 repo composite + diff harness。
```

## 方案 6：ComfyUI / Diffusers / Stable Diffusion inpainting

### 最新資料

Hugging Face Diffusers inpainting guide 說 inpainting 需要 base image 和 mask，使用 sketch/mask 定義要填補的區域。來源：<https://huggingface.co/docs/diffusers/en/using-diffusers/inpaint>

Stable Diffusion inpaint pipeline docs 說 inpainting 可透過 mask + text prompt 編修 image specific parts。來源：<https://huggingface.co/docs/diffusers/api/pipelines/stable_diffusion/inpaint>

ComfyUI 官方 inpainting workflow 文件介紹 mask editor、`VAE Encoder (for Inpainting)` node。來源：<https://docs.comfy.org/tutorials/basic/inpaint>

ComfyUI Inpaint Crop and Stitch nodes 可在 sampling 前 crop masked area，sampling 後 stitch 回原圖。來源：<https://github.com/lquesada/ComfyUI-Inpaint-CropAndStitch>

ComfyUI ImageCompositeMasked node 可用 mask 將 source composited 到 destination。來源：<https://comfyui-wiki.com/en/comfyui-nodes/image/image-composite-masked>

### 對本案的判斷

這是未來最可控的 engineering route，但不是第一個該做的 human baseline。

優點：

- 可本地跑。
- 可 script。
- 可生成 deterministic mask。
- 可接 pixel diff verification。

缺點：

- X 視覺品質、模型選擇、workflow tuning 都需要時間。
- 如果還沒有驗證 Photoshop baseline，太早投入 ComfyUI 可能會把問題又變成「找最強模型」。

建議：

1. 先用 Photoshop 證明 X 的人工視覺可行。
2. 同步寫 repo composite/diff harness。
3. 再用 ComfyUI/Diffusers 替換 X generator。

## 方案 7：Stability AI / Runway / 其他

### Stability AI

Stability AI API reference 顯示 inpaint mask 可由 separate mask 或 image alpha channel 提供。來源：<https://platform.stability.ai/docs/api-reference>

可作為 API candidate，但在本案優先級低於 Adobe/Flux/ComfyUI。

### Runway

Runway 官方 inpainting help 目前描述的是 video tool，用於移除影片中的 unwanted objects。Runway API 最新重點也偏 video generation / video editing，例如 2026-06 的 Aleph 2.0。來源：<https://help.runwayml.com/hc/en-us/articles/19155664495379-Inpainting>、<https://docs.dev.runwayml.com/>

不適合當 static `[A][X][B]` image pipeline 的第一線。

## 最務實路線

### Phase 1：Photoshop manual baseline

目標：用最少抽象討論，直接確認「人工 strict selected-region editing」可行。

步驟：

1. 準備一張真實 `[A][X][B]` 工作圖。
2. 在 Photoshop 中鎖定/保留 original base layer。
3. 人工精準選取 X。
4. Generative Fill X。
5. 產出 X layer / X patch。
6. 用 repo 腳本把 X 合回 original A/B。
7. 驗證 outside-X pixel diff = 0。

成功後，這就是第一版可用 workflow。

### Phase 2：Repo composite / diff harness

不管 X 來自 Photoshop、Midjourney、OpenAI、Firefly、FLUX，repo 都要有同一個驗收器：

```text
inputs:
  original.png
  x-mask-hard.png
  generated-x.png or candidate-full-output.png

outputs:
  final.png
  diff-report.json

acceptance:
  outside_x_changed_pixels == 0
```

這個 harness 是整條流水線的核心，不是模型本身。

### Phase 3：半自動化 / 程式化替代

當 Photoshop baseline 成功後，再看哪些步驟值得自動化：

- Firefly API：直接 mask-based generated X。
- Photoshop API：PSD/layer/smart object/export automation。
- FLUX Fill：高品質 X generator。
- ComfyUI/Diffusers：本地可控 X generator。

## 最終結論

如果本案目標是「A/B 完全不可動」，我現在的排序是：

1. **Photoshop manual baseline + repo hard composite**  
   最符合真實需求。人工可接受，選區可控，立刻可試。
2. **Adobe Firefly API / Photoshop API**  
   最自然的第二步，因為它延續 Photoshop/Adobe 工作流，又能逐步 API 化。
3. **FLUX Fill / ComfyUI / Diffusers**  
   最適合未來工程替代，但不要在 Photoshop baseline 前把問題複雜化。

明確不建議：

- 不要把 OpenAI image edit 的整張輸出當 final，因為 mask 是 prompt guidance。
- 不要把 Midjourney 當 production pipeline，除非只是拿來做 X visual exploration。
- 不要再用「大致保留」當評估語言。

本案真正規格：

```text
Only X may change.
A/B are immutable anchors.
Human-in-the-loop is acceptable.
The final artifact must be composited so A/B come from the original image.
```

## Sources

- Adobe Photoshop Generative Fill Help: <https://helpx.adobe.com/photoshop/desktop/create-open-import-images/create-images/edit-images-with-generative-fill.html>
- Adobe Photoshop Generative Fill product page: <https://www.adobe.com/products/photoshop/generative-fill.html>
- Adobe Generative Credits FAQ: <https://helpx.adobe.com/creative-cloud/apps/generative-ai/generative-credits-faq.html>
- Adobe Firefly Web Generative Fill: <https://helpx.adobe.com/firefly/web/edit-images/prompt-to-edit/generative-fill.html>
- Adobe Firefly API Masking: <https://developer.adobe.com/firefly-services/docs/firefly-api/guides/concepts/masking/>
- Adobe Firefly Services docs: <https://developer.adobe.com/firefly-services/docs/guides/>
- Adobe Photoshop API overview: <https://developer.adobe.com/firefly-services/docs/photoshop/>
- Adobe Photoshop API Smart Objects guide: <https://developer.adobe.com/firefly-services/docs/photoshop/guides/smart-objects-and-the-api/>
- Midjourney Vary Region: <https://docs.midjourney.com/hc/en-us/articles/32794723105549-Vary-Region>
- Midjourney Editor: <https://docs.midjourney.com/hc/en-us/articles/32764383466893-Editor>
- Midjourney Modifying Your Creations: <https://docs.midjourney.com/hc/en-us/articles/33329329805581-Modifying-Your-Creations>
- Midjourney Version docs: <https://docs.midjourney.com/hc/en-us/articles/32199405667853-Version>
- OpenAI Image Generation Guide: <https://developers.openai.com/api/docs/guides/image-generation>
- OpenAI Image Edit API Reference: <https://developers.openai.com/api/reference/resources/images/methods/edit/>
- Black Forest Labs FLUX.1 Fill docs: <https://docs.bfl.ml/flux_1_fill>
- Hugging Face FLUX.1-Fill-dev: <https://huggingface.co/black-forest-labs/FLUX.1-Fill-dev>
- ComfyUI Flux Fill workflow: <https://docs.comfy.org/tutorials/flux/flux-1-fill-dev>
- Hugging Face Diffusers Inpainting guide: <https://huggingface.co/docs/diffusers/en/using-diffusers/inpaint>
- Hugging Face Stable Diffusion Inpaint pipeline docs: <https://huggingface.co/docs/diffusers/api/pipelines/stable_diffusion/inpaint>
- ComfyUI Inpainting workflow: <https://docs.comfy.org/tutorials/basic/inpaint>
- ComfyUI Inpaint Crop and Stitch: <https://github.com/lquesada/ComfyUI-Inpaint-CropAndStitch>
- ComfyUI ImageCompositeMasked: <https://comfyui-wiki.com/en/comfyui-nodes/image/image-composite-masked>
- Stability AI API reference: <https://platform.stability.ai/docs/api-reference>
- Runway Inpainting help: <https://help.runwayml.com/hc/en-us/articles/19155664495379-Inpainting>
- Runway API docs: <https://docs.dev.runwayml.com/>

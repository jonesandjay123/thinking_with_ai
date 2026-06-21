# Selected-Region Image Editing / Mask Inpainting Research

日期：2026-06-21  
主題：哪些現成工具、模型與 API 能做到「只修改合成圖中間 X 區域，左右 A/B 區域完全不動」？

## TL;DR

一句話結論：這件事不是新問題，成熟方向叫做 selected-region editing / mask-based inpainting / generative fill；但「模型支援 mask」不等於「輸出保證 mask 外 pixel-perfect 不變」。本案的需求不是「盡量不動 A/B」，而是 **A/B outside hard mask 必須完全不動，pixel diff 必須是 0**。如果 `[A][X][B]` 之後還要接回原 pipeline，最穩的做法是兩段式：

```text
原圖 + mask -> 讓模型只負責產生 X 的候選內容
模型輸出 + 原圖 + hard mask -> deterministic composite 回原圖
```

也就是：用 inpainting model 生成 X，但最後用程式把 A/B 從原圖硬貼回去。這不是「保險」而是本案的必要條件：final output 的 A/B 區域必須由原圖 deterministic composite 回來。

最值得優先驗證的三個方案：

1. **ComfyUI / Diffusers + SDXL 或 FLUX Fill inpainting + final hard composite**  
   最適合工程 pipeline。mask 可控、可本地跑、可批次化、可插入自動驗證。缺點是 workflow 要自己搭，畫質要選模型調。
2. **Adobe Firefly API / Photoshop Generative Fill 作為人工和商業可用 baseline**  
   Adobe 的 GUI 和 API 都明確支援 selection / mask / generative fill，Photoshop 可以做到人工選區生成，適合快速驗證「人類修圖效果上限」。實務成本是 Adobe 服務 / generative credits；Photoshop GUI 不適合全自動 pipeline，Firefly API 則需要 Adobe 開發者流程與費用評估。
3. **OpenAI Image Edit API 作為 API baseline，但不能當 strict mask guarantee**  
   OpenAI 官方支援 mask editing，但官方文件也明說 GPT Image 的 mask 是 prompt-based guidance，可能不完全精準遵守 mask 邊界。因此它適合快速 API 實驗，不適合作為唯一 pixel-preserve 保證；若用它，也要 final composite。

Midjourney Vary Region / Editor 確實支援框選重畫，適合手動創作，不適合工程化與嚴格 pixel preserve。Runway 更偏影片/創意工具，不是這個 `[A][X][B]` image pipeline 的第一優先。

## 問題定義

我們的工作圖長得像：

```text
[ A 區域 ][ X 區域 ][ B 區域 ]
```

需求：

- A/B 是既有圖像內容，必須完全保留。
- X 是中間要重新生成、補圖或修改的區域。
- 輸出要能回到原 pipeline 裡銜接，所以不能只要求「看起來差不多」。
- 成功標準是 mask 外 pixel-preserve：A/B outside hard mask 的 pixel diff 必須是 0。

這裡要找的不是一般 image-to-image，也不是整張圖重畫，而是：

- 局部重生成
- 區域式編修
- selected-region editing
- inpainting with explicit mask / selection
- 只允許 X 變，A/B 完全不變

## 概念釐清

### 1. 普通 image edit / image-to-image

輸入一張圖與 prompt，模型產生一張新圖。它可能保留構圖、風格或主體，但通常是整張重新融合。這類方法不適合我們的 strict A/B preserve 需求，除非最後另做 hard composite。

典型風險：

- A/B 會被微調顏色、細節、壓縮紋理。
- 邊界位置可能漂移。
- 對人眼看起來差不多，但 pixel diff 很大。

### 2. Selected-region editing / Vary Region

使用者在 GUI 裡框選或塗抹某區域，模型只針對該區域重新生成。Photoshop Generative Fill、Adobe Firefly Generative Fill、Midjourney Vary Region / Editor 都屬於這類。

它適合人工修圖，但不等於工程上可重複、可驗證、可 pixel-perfect。若最後要進 pipeline，仍要把輸出經過 hard composite / pixel diff 驗證。

### 3. Mask-based inpainting

輸入原圖、mask、prompt。mask 通常定義「哪裡可改、哪裡不可改」：

- Stable Diffusion / Diffusers 常見慣例：white = inpaint，black = keep。
- Adobe Firefly API 文件：black areas protected，white areas exposed。
- FLUX Fill 文件：black areas preserved，white areas inpainted。
- OpenAI image edit 使用 alpha mask；透明區域代表可編修區域。

這是工程化最適合的抽象，因為 mask 可以由程式生成。

### 4. Pixel-preserve outside mask

這是最容易被混淆的重點。

模型層級的「preserve」通常表示模型會嘗試不修改或少修改 mask 外內容，但不能直接等同 byte-for-byte / pixel-for-pixel guarantee。原因包括：

- VAE encode/decode 可能改變整張圖的像素。
- latent sampling 可能影響邊界外。
- mask blur / grow 會讓保護區和生成區有過渡帶。
- API 可能輸出整張合成後的新圖，而不是只輸出 X patch。
- JPEG/WebP output 可能讓整張圖產生壓縮差異。

所以工程上要區分兩件事：

```text
模型支援 mask editing != mask 外像素必然完全不變
```

因為 A/B 必須完全保留，本案標準方法是：

1. 用 soft mask 給模型，讓 X 邊界生成自然。
2. 從模型輸出取出 X 或 soft-transition band。
3. 用 hard mask / alpha composite 把 X 貼回原圖，A/B 一律取自 original。
4. 對 A/B 區域跑 pixel diff，只有 0 diff 才算通過。

## 候選方案總覽

| 方案 | 支援選區 / mask | mask 外是否嚴格不改 | 工程化 / API | 手動使用 | 初步判斷 |
| --- | --- | --- | --- | --- | --- |
| Photoshop Generative Fill | 是，selection tools | 原圖以 generative layer 方式保留，但匯出結果仍要驗證 | 弱，主要 GUI；可用 Photoshop/Firefly services 另走 API | 很強 | 最佳人工 baseline |
| Adobe Firefly Web Generative Fill | 是，brush selection | 官方描述為修改特定區域，但工程仍需驗證 | 中，Web GUI；Firefly API 另有 mask | 強 | 人工快速驗證 |
| Adobe Firefly API | 是，grayscale mask | 文件語義最接近 protected / exposed | 強，REST API | 弱 | 值得測的商業 API |
| Midjourney Vary Region / Editor | 是，框選/Erase | 文件說不改 rest，但不是工程保證 | 弱，無正式通用 API | 強 | 創作工具，不是 pipeline 主線 |
| OpenAI Image Edit API | 是，mask object / alpha mask | 官方明說 mask 是 prompt guidance，可能不完全精準 | 強，API 友善 | 中 | API baseline，但必須 hard composite |
| Stable Diffusion / Diffusers Inpainting | 是，white fill / black keep | 模型層面不保證 pixel exact；可程式 hard composite | 強，本地 / Python | 中 | 最適合自建 pipeline baseline |
| ComfyUI Inpainting | 是，mask editor / VAE Encoder for Inpainting | 視 workflow；可用 composite/crop-stitch 保證 | 強，本地 graph / API 可自動化 | 中 | 最適合實驗工作台 |
| FLUX.1 Fill [pro/dev] | 是，mask 或 alpha | 文件宣稱 black preserve / white inpaint；仍建議 hard composite | 強，BFL API / Diffusers / ComfyUI | 中 | 高品質候選 |
| Stability AI Inpaint | 是，API endpoint / mask | 應視輸出與格式驗證 | 強，API | 弱 | 可列入商業 API candidate |
| Runway | 有 inpainting，但重點偏 video / creative tools | 不適合作為本案 strict image baseline | 中，API 偏影片生成 | 強 | 暫不優先 |
| IOPaint / LaMa / MAT 類開源 inpaint | 是，mask | 可本地 composite | 中高 | 中 | 適合 object removal，不一定適合創意 X 生成 |

## 方案細節

### 1. Photoshop Generative Fill / Generative Expand

Adobe Photoshop 官方文件將 Generative Fill 描述為在 Photoshop 中用文字 prompt 非破壞式修改圖片，操作流程是先用 selection tool 定義要編修的區域，再選 Generative Fill，輸入 prompt 或留空讓 Photoshop 根據周圍內容填補。來源：<https://helpx.adobe.com/photoshop/desktop/create-open-import-images/create-images/edit-images-with-generative-fill.html>

Adobe FAQ 也說，每次 Generate 會建立新的 generative layer，不會直接破壞原圖。來源：<https://helpx.adobe.com/in/photoshop/desktop/generative-ai/frequently-asked-questions-about-generative-ai-features.html>

判斷：

- 這就是 Jones 直覺中最接近的成熟 GUI 方案。
- 它非常適合人工選 X、快速修、挑 variation。
- Jones 已確認 Photoshop 這條路實務上可行；要納入的限制是 Adobe 服務訂閱與 generative credits / 點數成本。
- 對「工程 pipeline」不是第一選擇，因為 GUI 操作和 PSD/generative layer 不等於 headless batch pipeline。
- 但它很適合當 human baseline：先看 Photoshop 人工修圖能不能達到想要的 X transition。

對 `[A][X][B]`：

- 手動：高優先。
- 自動：除非接 Photoshop/Firefly services，否則不建議。
- pixel preserve：PSD 裡原圖 layer 可保留；但最終進 pipeline 的 PNG / raster 輸出仍應對 A/B 跑 diff。若 A/B 不是 0 diff，就要以原圖 A/B hard composite 修正。

### 2. Adobe Firefly Web / Firefly API

Firefly Web 的 Generative Fill 文件說它結合 generative AI 與 manual selection controls，可修改 image specific areas；操作上可 upload image，使用 brush paint over 要編修的區域，再用 prompt fill。來源：<https://helpx.adobe.com/firefly/web/edit-images/prompt-to-edit/generative-fill.html>

Firefly API 的 masking 文件更接近工程需求：mask 是 applied to an image 的 grayscale overlay，用於決定哪些部分 unaffected、哪些開放給 edits；black areas are protected，white areas are exposed。來源：<https://developer.adobe.com/firefly-services/docs/firefly-api/guides/concepts/masking/>

判斷：

- Firefly API 是 Photoshop 之外，Adobe 路線裡最值得工程化驗證的部分。
- 它的 mask 語義清楚，適合 `[A][X][B]` pipeline。
- 但仍要實測輸出是否 byte-identical；即使文件說 black protected，本案仍要求 final hard composite。
- 商業可用性、版權/Content Credentials、生態整合是 Adobe 的優勢。
- 現實限制是 Adobe API 權限、服務費用、generative credits / usage metering。

對 `[A][X][B]`：

- 手動：Firefly Web / Photoshop 都很強。
- 自動：Firefly API 值得試。
- pixel preserve：文件語義強，但仍要驗證；工程上仍要求 hard composite。

### 3. Midjourney Vary Region / Editor

Midjourney Vary Region 官方文件說，它讓使用者選擇 Midjourney image 的一部分並修改，rest of image 不改；流程包含 rectangular/freehand selection、可配 Remix Mode 修改 prompt。來源：<https://docs.midjourney.com/hc/en-us/articles/32794723105549-Vary-Region>

Midjourney Editor 文件說 web Editor 可用 Remix、inpainting/Vary Region、Pan、Zoom Out，也可在 Edit tab upload external images 使用更 advanced Editor tools。來源：<https://docs.midjourney.com/hc/en-us/articles/32764383466893-Editor>

判斷：

- Jones 對 Midjourney 的記憶是對的：它確實有 Vary Region / Editor。
- 它很適合視覺創作中「局部重畫」。
- 但它不是工程 pipeline 第一優先：沒有穩定正式 API、版本/解析度限制、輸出不是為 pixel diff 和 deterministic composite 設計。
- 官方文件也提醒 selection size 會影響結果，選太多可能讓想保留的 original parts 被 blending/replacing。

對 `[A][X][B]`：

- 手動探索：可試。
- 工程化：不建議當主線。
- pixel preserve：不可視為保證；只能視為 GUI selected-region edit。

### 4. OpenAI Image Edit API

OpenAI Image generation guide 支援 image edit with mask：可以提供 mask 指出要編修的部分。重點是，官方文件明確說 GPT Image 的 masking 是 prompt-based，模型會把 mask 當 guidance，但 may not follow exact shape with complete precision。來源：<https://developers.openai.com/api/docs/guides/image-generation>

OpenAI Image Edit API reference 也列出 `POST /images/edits`，支援 GPT Image models 和 `dall-e-2`，有 `images`、`prompt`、`mask`、`input_fidelity`、`quality`、`output_format` 等參數。來源：<https://developers.openai.com/api/reference/resources/images/methods/edit/>

判斷：

- OpenAI 是非常方便的 API baseline。
- 它支援 mask，但官方自己已經把「不精準遵守 mask」講清楚了。
- 所以它不適合被當作 strict local edit guarantee。
- 如果要用在工程 pipeline，應該採用：

```text
OpenAI edit with mask -> get candidate -> crop X / composite X back to original -> verify A/B pixel diff
```

對 `[A][X][B]`：

- 手動/半自動：中高。
- API：高。
- strict pixel preserve：低，除非外加 hard composite。

### 5. Stable Diffusion / Hugging Face Diffusers Inpainting

Hugging Face Diffusers 官方 inpainting 文件說，inpainting 會替換或編修 image specific areas，依靠 mask 決定哪些區域要 fill；white pixels 代表 inpaint area，black pixels 代表 keep area。來源：<https://huggingface.co/docs/diffusers/en/using-diffusers/inpaint>

Stable Diffusion inpainting pipeline 也明確是透過 mask + text prompt 編修圖片特定部分。來源：<https://huggingface.co/docs/diffusers/api/pipelines/stable_diffusion/inpaint>

判斷：

- 這是最標準、最可程式化的開源路線。
- 能直接用 Python 生成 mask、跑 model、保存中間產物、做 pixel diff。
- 可選 SD 1.5 inpaint、SDXL inpaint、Kandinsky inpaint、FLUX Fill pipeline 等。
- 缺點是模型輸出品質與 prompt following 需要調，且非 mask 區域仍不能只靠模型信任。

對 `[A][X][B]`：

- 工程化：很高。
- 可控性：很高。
- pixel preserve：靠 final composite 可做到。
- 建議：第一個本地 baseline。

### 6. ComfyUI Inpainting Workflow

ComfyUI 官方 inpainting workflow 文件介紹了 inpainting、mask editor、`VAE Encoder (for Inpainting)` node。它說 inpainting 的場景就是整體滿意但有局部元素要修；用 mask 告訴 AI 哪些區域要調整。文件也說 `mask` 是 VAE Encoder for Inpainting 的輸入，`grow_mask_by` 可擴大 mask 以避免硬邊。來源：<https://docs.comfy.org/tutorials/basic/inpaint>

判斷：

- ComfyUI 是最適合實驗的視覺工作台，尤其我們之前已經在 `pano-loop-lab` 類問題裡研究過 mask / crop / stitch。
- 它可以組合 SDXL inpaint、FLUX Fill、ControlNet、IPAdapter、crop-and-stitch、ImageCompositeMasked。
- 對 strict A/B preserve，ComfyUI workflow 裡要特別加入 final hard composite 或 crop/stitch nodes，而不是只看 Save Image 的輸出。

對 `[A][X][B]` 建議 workflow：

```text
1. Load original composite image
2. Load hard mask: X = white, A/B = black
3. Create soft/grown mask for model generation
4. Inpaint X using SDXL/FLUX Fill
5. Composite generated X back onto original with hard mask
6. Save final PNG
7. Run pixel diff on A/B outside hard mask
```

這是目前最接近 production pipeline 的方案。

### 7. FLUX.1 Fill [pro/dev]

Black Forest Labs 官方文件說 FLUX.1 Fill 是 text-driven inpainting/outpainting，用來 edit image regions 或 extend borders。它支援 separate mask 或 alpha channel：separate mask 方式是 image + black/white mask，black preserve original，white inpaint；alpha channel 方式則 transparent areas inpaint、opaque areas preserved。來源：<https://docs.bfl.ml/flux_1_fill>

BFL 也在 FLUX.1 Tools 介紹中說 FLUX.1 Fill 是 inpainting/outpainting model，可用 text description 與 binary mask 編輯、擴展真實或生成圖片。來源：<https://bfl.ai/blog/24-11-21-tools>

ComfyUI 官方也有 Flux.1 fill dev inpainting/outpainting workflow。來源：<https://docs.comfy.org/tutorials/flux/flux-1-fill-dev>

判斷：

- FLUX Fill 是高品質候選，尤其適合 X 需要自然銜接、光線/材質/世界一致性時。
- 它比一般 SD inpaint 可能更會理解 prompt 和保持周圍 context。
- 但模型較重，API 或本地配置成本較高。
- 即使官方寫 black preserved，本案工程上仍要求 final hard composite。

對 `[A][X][B]`：

- 品質候選：很高。
- 工程化：BFL API 高；ComfyUI/Diffusers 中高。
- strict pixel preserve：仍用 hard composite。
- 建議排在 ComfyUI/SDXL baseline 之後驗證。

### 8. Stability AI Inpainting

Stability AI developer platform 搜尋結果顯示其 REST API 有 `post /v2beta/stable-image/edit/inpaint` endpoint；官方 pricing 也列出 Inpaint：use a mask or alpha channel to replace anything in an image。來源：<https://platform.stability.ai/docs/api-reference>、<https://platform.stability.ai/pricing>

判斷：

- Stability 是可程式化商業 API 候選。
- 它符合 mask-based inpainting 類別。
- 但本輪沒有比 OpenAI / Firefly / FLUX Fill 更值得優先，除非我們想比較不同 commercial API 的穩定性、速度、價格。

對 `[A][X][B]`：

- API 化：高。
- 優先級：第二梯隊。
- strict preserve：仍需 hard composite 和 pixel diff。

### 9. Runway

Runway 官方 help center 的 inpainting 主要描述為 video tool，用於自動 remove unwanted objects throughout a clip。Runway API 文件重點是把 video generation / editing models 接到 app/product 中。來源：<https://help.runwayml.com/hc/en-us/articles/19155664495379-Inpainting>、<https://docs.dev.runwayml.com/>

判斷：

- Runway 是強創意工具，但本案是 static image `[A][X][B]` mask pipeline。
- 它不是第一優先，除非問題轉成影片/動態物件移除。

### 10. IOPaint / LaMa / MAT 類 open-source inpaint

IOPaint 是開源 inpainting tool，支援 LaMa、MAT、Stable Diffusion 等模型，定位包含 remove unwanted object / erase and replace。來源：<https://github.com/Sanster/IOPaint>

判斷：

- 適合 object removal / cleanup / batch erase。
- 如果 X 只是要 remove/fill background，它可能很有效。
- 如果 X 要根據 prompt 生成複雜新內容，還是 SDXL/FLUX/OpenAI/Firefly 類更合適。

## 哪些是真的「嚴格局部編修」？

要分三個層級：

### Level 1：GUI selected-region edit

代表：

- Photoshop Generative Fill
- Firefly Web Generative Fill
- Midjourney Vary Region / Editor

使用者體感上是「只改選區」，但對工程 pipeline 來說通常不能直接宣稱 pixel-perfect。適合人工修圖，不適合作為唯一自動保證。

### Level 2：API / model 支援 mask

代表：

- Adobe Firefly API
- OpenAI Image Edit API
- Stability AI Inpaint
- FLUX Fill API
- Diffusers / Stable Diffusion inpaint
- ComfyUI inpaint workflows

這類有明確 mask input，適合程式化。可是不同平台對 mask 的解釋和遵守程度不同。OpenAI 官方明確說 GPT Image mask 是 prompt-based guidance，可能不完全精準；FLUX/Firefly 文件的 protected/preserved 語義更強，但仍要實測。

### Level 3：工程上真正 pixel-preserve

代表做法不是單一模型，而是 pipeline：

```text
candidate = inpaint(original, soft_mask, prompt)
final = composite(original, candidate, hard_mask)
assert pixel_diff(final, original, outside_hard_mask) == 0
```

只有這層能對 A/B 做真正保證。

## 工程用途建議

如果我們要做 repo pipeline，不建議把「模型輸出整張圖」當最終答案。建議標準流程：

```text
inputs/
  original.png
  mask-hard.png      # X=white, A/B=black
  mask-soft.png      # X expanded/blurred for nicer boundaries

generation/
  run inpaint model with original + mask-soft + prompt
  output candidate.png

postprocess/
  final.png = original outside mask-hard + candidate inside mask-hard
  diff-outside-mask.json = pixel diff report
```

驗收指標：

- A/B outside hard mask pixel diff = 0。
- X 區域視覺自然。
- X 與 A/B 邊界沒有硬邊。
- 如果需要 soft transition，可以讓 hard mask 包含一小段 transition band，但要明確定義哪些像素允許改。
- 輸出格式優先 PNG，避免 JPEG 壓縮造成 A/B diff。

這裡的重點是：strictness 來自 postprocess/composite，不來自模型承諾。

## 人工 vs API / 自動化分類

### 最適合人工手動

1. Photoshop Generative Fill  
   人工 selection + generative layer + variation selection 最成熟。
2. Adobe Firefly Web Generative Fill  
   快速 brush selection，適合 proof-of-concept。
3. Midjourney Vary Region / Editor  
   創作上方便，但不適合要求 pixel-diff 或批次流程。

### 最適合程式化 / pipeline

1. ComfyUI inpainting workflow + final composite  
   適合視覺化調參與本地自動化。
2. Diffusers / Python inpainting + final composite  
   最適合進 repo pipeline、CI-like diff validation。
3. FLUX Fill API / BFL API  
   高品質商業 API candidate。
4. Adobe Firefly API  
   商業安全與 mask 語義明確，值得比較。
5. OpenAI Image Edit API  
   開發體驗好，但 mask precision 官方保留，需要外部 composite。

## 建議優先驗證的 3 個方案

### Priority 1：ComfyUI / Diffusers + SDXL inpaint baseline + hard composite

理由：

- 最快驗證工程核心：mask、composite、pixel diff。
- 成本低、可本地、可重複。
- 不被任何 SaaS UI 限制。
- 能清楚拆出「模型生成 X」和「A/B 保留」兩個問題。

第一個實驗不追求最美，只驗證：

```text
outside hard mask 是否 0 pixel diff
X 是否真的被改
X/A/B 邊界是否可接受
```

### Priority 2：FLUX.1 Fill [pro/dev] + same hard composite harness

理由：

- 它是專門 inpainting/outpainting 的高品質模型。
- 官方支援 mask / alpha channel，語義直接對應 X。
- 若 SDXL baseline 能跑通但 X 品質不足，Flux Fill 是最自然的下一步。

注意：

- 本地 dev 版可能較重，先用裁切 X band 或 API。
- 仍要用同一套 hard composite + pixel diff harness，不要信任模型輸出整張圖。

### Priority 3：Adobe Firefly API 或 OpenAI Image Edit API 作為 commercial API baseline

理由：

- 這兩個都是現成可用 API。
- Firefly mask 語義更接近 protected/exposed；OpenAI API 接入和 prompt flexibility 很方便。
- 可以比較 SaaS API 在 X 生成品質、速度、成本、mask 遵守程度上的差異。

排序：

- 如果重視 mask 語義與商業素材安全：先 Firefly API。
- 如果重視快速接 API、prompt 能力、和現有 OpenAI stack：先 OpenAI，但必須接受 mask 不是 strict guarantee。

## 為什麼之前會鬼打牆

因為很多「image edit」工具看起來像局部編修，但底層其實是在做整張圖的重融或整張輸出：

- image-to-image 會重畫整張，只是保留視覺相似。
- selected-region GUI 讓人感覺只改選區，但輸出仍可能是整張新 raster。
- mask-based inpainting 有 mask，但 VAE / latent / output encoding 仍可能讓 mask 外有差異。
- 有些模型把 mask 當 prompt guidance，不是硬約束。

所以如果我們把「模型有 mask」誤認成「A/B 一定 pixel-preserve」，就會反覆遇到：看起來差不多，但接回 pipeline 時邊界、色彩、像素都漂了。

真正要解法是承認模型只負責生成候選內容；工程流程負責把不該動的像素硬保住。

## 明確結論

如果目標是「只改 X、不動 A/B」，下一步最值得做的是：

1. 建一個小型 mask inpaint harness：
   - input：`original.png`、`mask-hard.png`、`mask-soft.png`、prompt。
   - output：`candidate.png`、`final-composited.png`、`pixel-diff-outside-mask.json`。
2. 第一輪用 ComfyUI 或 Diffusers SDXL inpaint 跑通，不追求美，只追求流程正確。
3. 第二輪把模型換成 FLUX Fill 或 Firefly/OpenAI API，比較 X 區域品質。
4. 永遠用 hard composite 保證 A/B，並用 pixel diff 驗證。

最短推薦路線：

```text
Diffusers/ComfyUI SDXL inpaint smoke test
  -> final hard composite
  -> outside-mask pixel diff
  -> if pipeline passes, test FLUX Fill for quality
  -> compare Firefly/OpenAI API as external baselines
```

Photoshop / Midjourney 可以作為人工靈感和品質 baseline，但不應該作為工程 pipeline 的核心。

## Sources

- Adobe Photoshop Generative Fill: <https://helpx.adobe.com/photoshop/desktop/create-open-import-images/create-images/edit-images-with-generative-fill.html>
- Adobe Photoshop Generative AI FAQ: <https://helpx.adobe.com/in/photoshop/desktop/generative-ai/frequently-asked-questions-about-generative-ai-features.html>
- Adobe Firefly Web Generative Fill: <https://helpx.adobe.com/firefly/web/edit-images/prompt-to-edit/generative-fill.html>
- Adobe Firefly API masking: <https://developer.adobe.com/firefly-services/docs/firefly-api/guides/concepts/masking/>
- Midjourney Vary Region: <https://docs.midjourney.com/hc/en-us/articles/32794723105549-Vary-Region>
- Midjourney Editor: <https://docs.midjourney.com/hc/en-us/articles/32764383466893-Editor>
- OpenAI Image Generation Guide / mask editing: <https://developers.openai.com/api/docs/guides/image-generation>
- OpenAI Image Edit API reference: <https://developers.openai.com/api/reference/resources/images/methods/edit/>
- Hugging Face Diffusers inpainting guide: <https://huggingface.co/docs/diffusers/en/using-diffusers/inpaint>
- Hugging Face Stable Diffusion inpaint pipeline docs: <https://huggingface.co/docs/diffusers/api/pipelines/stable_diffusion/inpaint>
- ComfyUI inpainting workflow: <https://docs.comfy.org/tutorials/basic/inpaint>
- Black Forest Labs FLUX.1 Fill docs: <https://docs.bfl.ml/flux_1_fill>
- Black Forest Labs FLUX.1 Tools announcement: <https://bfl.ai/blog/24-11-21-tools>
- ComfyUI Flux.1 fill dev workflow: <https://docs.comfy.org/tutorials/flux/flux-1-fill-dev>
- Stability AI API reference: <https://platform.stability.ai/docs/api-reference>
- Stability AI pricing / Inpaint listing: <https://platform.stability.ai/pricing>
- Runway Inpainting help: <https://help.runwayml.com/hc/en-us/articles/19155664495379-Inpainting>
- Runway API docs: <https://docs.dev.runwayml.com/>
- IOPaint GitHub: <https://github.com/Sanster/IOPaint>

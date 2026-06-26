# A/B Image Transition Adapter Research

日期：2026-06-19  
主題：世界上是否已有模型、repo、paper 或 ComfyUI workflow，專門或接近解決「給定兩張圖 A/B，生成中間橋接圖 / transition adapter」的問題？

## TL;DR

一句話回答：沒有找到成熟、現成、直接對位 `pano-loop-lab` 的「A.right + B.left -> fixed-width panoramic adapter」工具或 repo；最接近的是三類相鄰解：diffusion image morphing / interpolation、dual-reference / IPAdapter / reference-guided inpaint、以及 anchor-preserved crop-and-stitch inpaint。真正可落地的下一步不是把其中一類單獨當答案，而是組成 pipeline：先用 image interpolation / Higgsfield / Flux 產生 concept transition，再用 ComfyUI inpaint / Flux Fill / IPAdapter 把 concept 壓進中央 mask，最後用 crop-and-stitch / hard composite 保證 A/B anchors pixel exact。

最值得馬上試：

1. `concept-guided anchor-preserved inpaint`：用 A/B edge crops + concept image + mask inpaint，而不是只靠文字 prompt。
2. `DiffMorpher / FreeMorph / diffusion interpolation` 當 concept generator：產生 dawn->dusk 的語意過渡參考圖，但不當 final adapter。
3. `dual-reference IPAdapter / Flux Fill / Flux Redux 類 reference-guided fill`：讓模型同時看 A 與 B 的視覺語彙，再在中央 band 生成。

暫時不建議把 video frame interpolation、panorama stitching、Schrödinger bridge / diffusion bridge 當主線。它們名字像，但 task objective 不同。

## 我們的任務定義

`pano-loop-lab` 要解的不是一般 inpainting、video morph、frame interpolation，也不是傳統 panorama stitching。任務比較像：

```text
Input:
  scene A image
  scene B image
  A 的右邊緣 crop
  B 的左邊緣 crop
  固定 adapter 寬度

Output:
  一張固定寬度的 adapter / bridge / transition image

Requirements:
  adapter 左邊能自然接回 A 的右邊
  adapter 右邊能自然接回 B 的左邊
  中央要形成可信的 transition world
  最好能 repeatable，用於未來任意 scene pair
```

目前狀態：

1. Higgsfield whole-frame image-to-image：單張漂亮，但左右 anchor 被重畫，接回原 plate 時 `blend=0` 明顯失敗。
2. ComfyUI SDXL inpaint：anchor preserve 可以做到 pixel exact，但 official SDXL inpaint 的中央 transition world 品質不足；多輪 prompt / mask / soft composite 微調仍不自然。

本案成功標準：

1. 是否能吃兩張 input images。
2. 是否能保留 A.right / B.left anchor。
3. 是否能生成固定寬度 adapter。
4. 是否能產生自然 transition world。
5. 是否能在 ComfyUI / local RTX 5080 16GB PC 上實作。
6. 是否可重複用於 future scene pairs。
7. 是否比目前 SDXL inpaint route 更有突破性。

## 最接近的已知方法分類表

| 類型 | 代表 paper/repo/workflow | 接近程度 | 可用性 | 風險 | 是否建議嘗試 |
| --- | --- | --- | --- | --- | --- |
| Diffusion image morphing / interpolation | DiffMorpher、FreeMorph、Interpolating between Images with Diffusion Models、AID | 中。真的吃兩張圖，會生成中間態 | 有 paper/repo，但多偏 research / sequence | 產出是時間中間幀，不是固定寬度 spatial adapter；不保證左右 anchor pixel exact | 建議當 concept generator，不當 final |
| ComfyUI image morph / frame interpolation | ComfyUI frame interpolation、WAS Create Morph Image、AnimateDiff / Steerable Motion workflows | 低到中。能做 A->B 過渡動畫 | ComfyUI 生態可用 | 目標是 temporal smoothness，不是左右接縫 weld | 僅可參考，不當主線 |
| Dual-reference / IPAdapter / reference-guided inpaint | IP-Adapter、ComfyUI_IPAdapter_plus、Flux Fill + reference / Redux 類 workflow | 中高。能讓模型看 A/B 或 concept image | ComfyUI 可實作，需要 custom nodes | Reference guidance 不等於 boundary constraint；仍需 hard composite | 最值得工程化嘗試 |
| Inpainting / outpainting from both sides | Flux Fill、SDXL inpaint、Fooocus patch、Crop & Stitch | 高。工程上可構造 A/B anchors + middle mask | ComfyUI 支援成熟度最高 | 本身不自動理解「A->B bridge」，需要 prompt/reference/control | 最實用骨架 |
| Panorama / equirectangular / matte painting | PanoDiffusion、panorama image inpainting、sd-webui-panorama-tools、DiT360 | 中。處理 panorama / outpainting / seam | 多為 research 或 A1111 extension | 多假設單一 panorama 或 360 場景，不是兩個 AI worlds 的 adapter | 可借鑑，不是直接解 |
| Game tileset / transition tiles | tileset generation、GB Studio transition tips、game background workflows | 低到中。概念上接近 tile adjacency | 多偏像素 / tilemap / manual alignment | 不處理 high-res panoramic diffusion bridge | 可借評估思路 |
| Schrödinger bridge / diffusion bridge | I2SB、Diffusion Schrödinger Bridge Matching | 名字像，實質低 | Research repo | 通常是 distribution-to-distribution restoration / translation，不是 A/B boundary adapter | 暫緩 |

## 直接相關候選

### 1. DiffMorpher

來源資訊：DiffMorpher 是 CVPR 2024 image morphing 工作。project page 明確說「給定兩張 input images，可以生成 smooth and natural transition video between them」。方法上，先對兩張圖分別 fit LoRA，再插值 LoRA parameters 與 latent noises，並做 attention interpolation / injection。來源：<https://kevin-thu.github.io/DiffMorpher_page/>、<https://github.com/Kevin-thu/DiffMorpher>、<https://arxiv.org/abs/2312.07409>

對本案的符合度：

- 吃兩張 input images：是。
- 保留 A.right / B.left anchor：否。它保留的是首尾 frame 概念，不是 spatial boundary pixels。
- 固定寬度 adapter：否。它生成的是 morph sequence / intermediate image。
- 自然 transition world：可能很有幫助，尤其作為 concept reference。
- ComfyUI / local 5080：不是原生 ComfyUI；GitHub code 可跑，但需要 per-pair LoRA fitting，速度和流程重。
- 重複用於 future scene pairs：理論可，但每對圖都要 fit / invert，成本高。

推論：DiffMorpher 是「最像我們問題的 research direction」，但不是 final adapter generator。它最有價值的用法是產生 dawn-valley -> dusk-ridge 的語意中間態或多個 transition concepts，選一張最合理的 concept image，再餵回 anchor-preserved inpaint pipeline。

建議實驗：

```text
A full plate / A edge crop + B full plate / B edge crop
  -> DiffMorpher / FreeMorph generate 3-7 morph frames
  -> select middle concept frame
  -> crop/resize concept into adapter canvas center
  -> ComfyUI inpaint/refine only central band
  -> hard composite anchors
```

### 2. Interpolating between Images with Diffusion Models

來源資訊：Clinton Wang 與 Polina Golland 的工作提出 zero-shot controlled interpolation between two input images，project page 說可在 diverse styles、layouts、subjects 間生成 controllable / creative interpolations；arXiv 摘要說這是目前 deployed image generation pipelines 缺少的 feature，方法使用 latent diffusion、textual inversion、interpolated text embeddings、可選 pose guidance 與 CLIP candidate selection。來源：<https://clintonjwang.github.io/interpolation>、<https://arxiv.org/abs/2307.12560>、<https://openreview.net/forum?id=L2D9Gybx0P>

對本案的符合度：

- 吃兩張 input images：是。
- 保留 endpoints / boundaries：只保留作為插值端點，不保證 adapter 左右像素可接回原圖。
- 固定寬度 adapter：否。
- 自然 transition world：可能有用。
- ComfyUI / local：不是直接 workflow；偏 research pipeline。

推論：這類方法可以回答「中間世界長什麼樣」，但不能回答「adapter 左右邊緣如何 weld」。因此可作為 concept prior，不應替代 mask inpaint。

### 3. FreeMorph

來源資訊：FreeMorph 宣稱是 tuning-free generalized image morphing with diffusion，能處理 semantics / layout 不同的 inputs，避免 DiffMorpher per-instance fitting 的時間成本。來源：<https://github.com/yukangcao/FreeMorph>、<https://yukangcao.github.io/FreeMorph/>

對本案的符合度：

- 吃兩張 input images：是。
- 保留 anchors：否。
- 固定寬度 adapter：否。
- 實用性：若安裝順利，比 DiffMorpher 更適合作為快速 concept generator。

推論：FreeMorph 比 DiffMorpher 更適合先試，因為 tuning-free 對 repeated future scene pairs 更友善。但它仍是 morphing，不是 spatial bridge welding。

### 4. Attention Interpolation via Diffusion / AID

來源資訊：AID 是 NeurIPS 2024 training-free method，用 text-to-image diffusion 在不同 conditions 間生成 interpolation；PAID 加 prompt guidance。來源：<https://github.com/QY-H00/attention-interpolation-diffusion>

對本案的符合度：

- 對 prompt / condition interpolation 有幫助。
- 不直接吃 A.right/B.left 作為 hard boundaries。
- 不直接產生固定 spatial adapter。

推論：AID 更適合「prompt / semantic condition interpolation」而不是本案的 pixel weld。可以參考 attention interpolation 的想法，但不建議作為下一步工程主線。

## 可改造候選

### 1. Dual-reference / IPAdapter-guided inpaint

來源資訊：IP-Adapter 是 Tencent AI Lab 的 image prompt adapter，目標是讓 pretrained text-to-image diffusion model 具備 image prompt capability；它使用 decoupled cross-attention，將 text features 與 image features 分開，並可與 text prompt、ControlNet 等工具一起使用。來源：<https://ip-adapter.github.io/>、<https://github.com/tencent-ailab/IP-Adapter>

ComfyUI_IPAdapter_plus 是 ComfyUI 的 IPAdapter reference implementation，README 描述 IPAdapter 能把 reference image 的 subject 或 style 轉移到 generation，類似 1-image LoRA。來源：<https://github.com/cubiq/ComfyUI_IPAdapter_plus>

對本案的符合度：

- 吃兩張 input images：可透過 multiple reference / multiple IPAdapter / masked conditioning 近似做到。
- 保留 anchors：不保證，必須靠 final composite。
- 固定寬度 adapter：可在我們的 canvas/mask workflow 裡做到。
- 自然 transition world：比純文字 prompt 更有機會，因為模型能看到 A/B 的實際視覺語彙。
- ComfyUI / 5080：可行，但需要 custom nodes 和 careful weighting。

推論：這是本案最實用的新方向之一。因為目前 SDXL official inpaint 的問題不是 anchor，而是「中間不知道該如何自然 bridge dawn->dusk」。讓模型同時看到 A、B，或看到 Higgsfield c08 concept，再在 masked center 裡生成，比繼續改 prompt / mask blur 更有突破性。

建議兩種用法：

1. A/B dual reference：
   - IPAdapter left/reference weight 偏向 dawn-valley。
   - IPAdapter right/reference weight 偏向 dusk-ridge。
   - 用 mask 或 regional conditioning 讓左半 transition 更像 A，右半更像 B。
2. Concept reference：
   - 用 Higgsfield c08/c04 或 DiffMorpher middle frame 當 concept image。
   - IPAdapter / Flux Redux 只提供中央 world 的 style/structure。
   - 最終仍只 composite 中央白區。

### 2. Flux Fill / Flux Redux / reference-guided Fill

來源資訊：Black Forest Labs 將 FLUX.1 Fill 定義為 inpainting / outpainting model，可基於 text description 與 binary mask 編輯或擴展影像。ComfyUI 官方也有 Flux Fill workflow。來源：<https://bfl.ai/blog/24-11-21-tools>、<https://huggingface.co/black-forest-labs/FLUX.1-Fill-dev>、<https://docs.comfy.org/tutorials/flux/flux-1-fill-dev>

另外，Flux Fill + reference image / Flux Redux 類 workflow 在 ComfyUI 社群教學中被用於 reference-guided inpainting。來源：<https://learn.thinkdiffusion.com/the-inpainting-revolution-how-reference-images-with-flux-fill-and-flux-redux-are-changing-the-game/>

對本案的符合度：

- 吃 A/B 兩張圖：Flux Fill 本身吃 init image + mask + prompt；若要吃兩張 reference，需要搭配 Redux / IPAdapter / custom workflow。
- 保留 anchors：仍需 hard composite。
- 固定寬度 adapter：可由 canvas/mask 控制。
- 自然 transition world：高潛力。
- 5080 16GB：可嘗試 crop band / quantized / offload，不能一開始 full native。

推論：Flux Fill 是「生成中央 transition」的高品質候選，但不是「A/B adapter task」的完整解法。它要跟 A/B reference conditioning、crop-and-stitch、hard composite 合用。

### 3. Crop-and-stitch / anchor-preserved inpaint

來源資訊：ComfyUI Crop & Stitch nodes 可在 sampling 前裁切 masked area，sampling 後 stitch 回原圖，不修改 unmasked part，甚至不讓 unmasked part 經過 VAE encode/decode。來源：<https://github.com/comfyorg/comfyui-crop-and-stitch>

對本案的符合度：

- 吃兩張 input images：它不是 model，不負責語意 bridge。
- 保留 anchors：非常符合。
- 固定寬度 adapter：符合。
- 自然 transition world：無法單獨解。

推論：這是必備工程骨架。任何 morphing / IPAdapter / Flux concept 最後都應該回到 crop-and-stitch 或 hard composite，否則 anchor preserve 仍不可信。

## 名字像但不適用

### 1. Video frame interpolation

來源資訊：ComfyUI 官方 frame interpolation workflow 說明其用途是生成 existing video frames 中間的 frames，以提高 frame rate、smooth motion、改善 temporal consistency。來源：<https://docs.comfy.org/tutorials/utility/frame-interpolation>

對本案的問題：

- frame interpolation 的 endpoints 是時間上的前後影格。
- 它假設兩張圖是同一段 motion 的相鄰或近似相鄰 frame。
- 它產出的中間 frame 不會具有「左邊接 A、右邊接 B」的空間結構。

推論：video frame interpolation 可以啟發「中間態」概念，但不能直接轉成 spatial adapter。把 frame interpolation 的中間 frame 拿來當 concept reference 可以試；把它當 final adapter 不合理。

### 2. Simple image morph / crossfade nodes

來源資訊：RunComfy 的 WAS `Create Morph Image` node 被描述為 blend two images、產生 smooth transition effect，特別適合 animated GIFs / visual transitions。來源：<https://www.runcomfy.com/comfyui-nodes/was-node-suite-comfyui/Create-Morph-Image>

對本案的問題：

- 這類 node 通常是 pixel/latent blend 或 animation morph。
- 不會生成新的可信地形 world。
- 不保證 A.right/B.left weld。

推論：可以用來做快速 diagnostic 或 low-tech concept，但不會解決 dawn-valley -> dusk-ridge 的高品質 transition。

### 3. Panorama stitching

來源資訊：傳統 panorama / stitching 類工具通常假設多張圖是同一真實場景，有可匹配 features，可估 homography / seam / blender；先前 source scout 也已看過 OpenStitching / OpenPano 類 repo。這與本案「兩個 AI 世界」不同。

對本案的問題：

- A 和 B 不是同一相機掃過同一場景。
- 沒有 reliable features 可對齊。
- stitching 主要解 seam placement / blending，不負責生成新 transition world。

推論：可借 seam finder / blender / quality metric 的概念，不是 generator。

### 4. Schrödinger bridge / diffusion bridge

來源資訊：I2SB（Image-to-Image Schrödinger Bridge）是 conditional diffusion / image restoration / image-to-image translation 方向，project page 說 diffusion bridges 特別適用於 image restoration，因為 degraded images 是重建 clean images 的 structurally informative priors；GitHub 也描述它在 image restoration tasks 上有效。來源：<https://i2sb.github.io/>、<https://github.com/NVlabs/I2SB>、<https://proceedings.mlr.press/v202/liu23ai.html>

對本案的問題：

- 它的「bridge」是 probability distributions / restoration process 的 bridge。
- 不是產品型 pipeline：給 A.right、B.left、固定寬度，輸出 adapter。
- 不直接保證 endpoints pixel exact。

推論：這方向理論上有趣，但不該進下一輪實驗。除非未來要做自訓模型，否則暫緩。

## Panorama / game background / matte painting

### Panorama diffusion / inpainting

來源資訊：

- `panorama_image_inpainting` 是 360-degree panoramic image inpainting method，使用 equirectangular -> cube map 以降低 distortion，並考慮 cube map 六面相關性。來源：<https://github.com/swhan0329/panorama_image_inpainting>
- `sd-webui-panorama-tools` 是 A1111 panorama editing extension，提供 equirectangular panorama editing/outpainting 工具。來源：<https://github.com/Flyguygx/sd-webui-panorama-tools>
- PanoDiffusion 是 360-degree indoor RGB-D panorama outpainting model，用 latent diffusion 從 narrow FoV 生成完整 360 panorama。來源：<https://arxiv.org/html/2307.03177v5>
- DiT360 / PanFusion 等研究處理 text-to-360 panorama generation。來源：<https://github.com/Insta360-Research-Team/DiT360>、<https://arxiv.org/html/2404.07949v1>

對本案的符合度：

- 這些方向處理 panorama distortion / 360 consistency，很接近 `panoramic` 字面。
- 但它們通常處理單一 panorama 的 outpainting / completion / 360 generation，不是兩個 scene plates 的 fixed-width transition adapter。
- 如果 `pano-loop-lab` 未來改成 equirectangular 360 world，這些很值得回來看；目前不是最短路徑。

### Game background / tileset transition

來源資訊：GB Studio transition tip 類文章談到背景 transition / alignment 是 trial-and-error 調整；tileset AI generator / Stable Diffusion tileset workflow 多處理 tile grid、seamless assets、pixel/top-down tiles。來源：<https://gbstudiocentral.com/tips/create-an-almost-seamless-transition/>、<https://www.boristhebrave.com/2025/02/04/generating-tilesets-with-stable-diffusion/>

對本案的符合度：

- 概念上相近：相鄰 tile / background 必須接得上。
- 技術上不同：通常是 pixel art / grid tile / procedural adjacency，不是 high-res panoramic matte painting。

推論：可借「adjacency constraint / transition tile」的測試思路，但沒有找到可直接拿來生成 Jcompany panoramic adapter 的成熟 repo。

## ComfyUI 可嘗試 workflows

### Workflow A：Concept-guided anchor-preserved inpaint

這是最值得馬上試的實用方向。

```text
Inputs:
  adapter-work-canvas.png
  adapter-mask.png
  A.right crop
  B.left crop
  optional concept image: Higgsfield c08/c04 or DiffMorpher middle frame

ComfyUI:
  load work canvas + hard mask
  crop masked band with left/right context
  encode concept/reference via IPAdapter or Flux Redux
  run SDXL/Flux Fill/Fooocus inpaint on central white band
  stitch/composite back
  verify anchor pixel diff = 0
```

優點：

- 保留目前已證明有效的 anchor-preserved engineering。
- 加入 concept/reference 來解決「中央世界不知道畫什麼」。
- 可在 5080 16GB 上先用 SDXL / cropped band 跑。

風險：

- IPAdapter / reference strength 太高可能把 anchor 邊界或 concept artifacts 帶進去。
- 需要區分 generation soft mask 與 final hard mask。

### Workflow B：DiffMorpher / FreeMorph concept generator -> ComfyUI weld

```text
A full / A edge + B full / B edge
  -> DiffMorpher or FreeMorph generate transition sequence
  -> pick 1-3 frames as concept
  -> use concept as visual guide in ComfyUI inpaint
  -> final hard composite
```

優點：

- 專門解「A 到 B 中間語意長什麼樣」。
- 可提供 prompt 很難描述的地形/光線過渡。

缺點：

- 安裝/運行可能慢。
- 不保證 spatial edge compatibility。
- 不應直接拿 middle frame 當 adapter，除非重新 weld。

### Workflow C：Flux Fill + dual reference / Redux 高品質候選

```text
crop adapter central band + context
  -> Flux Fill masked generation
  -> optional reference image conditioning
  -> hard composite anchors
  -> renderer blend=0 / blend=16 compare
```

優點：

- 最高品質潛力。
- 比 official SDXL inpaint 更可能生成自然地形與光線 transition。

缺點：

- 16GB VRAM 需要 crop / quantization / offload。
- ComfyUI workflow 變數多。
- 仍不是專門的 A/B adapter generator。

## 對 pano-loop-lab 的建議

### Option A：先做 concept-guided inpaint

下一步最實際。用目前已存在的 `adapter-work-canvas.png` / `adapter-mask.png`，但新增 concept/reference input：

- concept source 1：Higgsfield c08/c04，因為它漂亮但 anchor fail。
- concept source 2：DiffMorpher / FreeMorph middle frame。
- concept source 3：手工 rough value map / structure sketch。

目標：讓 central transition 有更強的「世界設計」訊號，同時用 hard composite 保留 anchors。

### Option B：用 image morphing 當 research spike

做一個很小的 spike，不要先整合主流程：

- 跑 DiffMorpher 或 FreeMorph，輸入 dawn-valley / dusk-ridge。
- 只看中間 frame 是否提供比 prompt 更好的 transition concept。
- 不做 pixel weld 驗收。

若中間 frame 有價值，再把它納入 Option A。

### Option C：Flux Fill production candidate

當 Option A 用 SDXL / Fooocus patch 仍不自然時，再用 Flux Fill。

- 先 cropped band，不 full native。
- 若 VRAM 不穩，走 FP8 / GGUF / offload。
- 必須保留 hard composite + renderer weld test。

## 最終建議

### 1. 最值得馬上試

`concept-guided anchor-preserved inpaint`。

具體做法：不要只丟 prompt 給 official SDXL inpaint。把 A/B edges、Higgsfield concept、或 morphing-generated concept 變成 reference/control，讓模型知道中間世界應該怎麼過渡；最後用 crop-and-stitch / ImageCompositeMasked 保證 anchors。

### 2. 第二順位

`DiffMorpher / FreeMorph as concept generator`。

這類 research 最接近「兩張圖生成中間態」，但不是 spatial adapter。值得用 1-2 小時 spike 看它能不能產生有用的 dawn->dusk middle concept。

### 3. 值得研究但暫緩

`Flux Fill + dual-reference / Redux`。

它很可能是高品質 production candidate，但應等 concept-guided SDXL / Fooocus route 跑出對照後再投入，避免把 VRAM / workflow / model quality 問題混在一起。

### 4. 不建議

- 直接把 video frame interpolation 當 adapter generator。
- 直接把 DiffMorpher middle frame 當 final adapter。
- 只做 latent/pixel blend / morph。
- 把 panorama stitching 當 generator。
- 把 I2SB / Schrödinger bridge 當近期工程路線。

## 來源與判斷標註

### 來源讀到的資料

- DiffMorpher：給定兩張 input images，可生成 smooth and natural transition video；方法包含 fitting two LoRAs、interpolating LoRA parameters / latent noises、attention interpolation / injection。  
  <https://kevin-thu.github.io/DiffMorpher_page/>  
  <https://github.com/Kevin-thu/DiffMorpher>  
  <https://arxiv.org/abs/2312.07409>
- Interpolating between Images with Diffusion Models：zero-shot controlled interpolation between two input images，使用 latent diffusion、textual inversion、text embedding interpolation、可選 pose guidance / CLIP selection。  
  <https://clintonjwang.github.io/interpolation>  
  <https://arxiv.org/abs/2307.12560>  
  <https://openreview.net/forum?id=L2D9Gybx0P>
- FreeMorph：tuning-free generalized image morphing with diffusion，可處理不同 semantics / layouts。  
  <https://github.com/yukangcao/FreeMorph>  
  <https://yukangcao.github.io/FreeMorph/>
- AID / Attention Interpolation via Diffusion：training-free condition interpolation for text-to-image diffusion。  
  <https://github.com/QY-H00/attention-interpolation-diffusion>
- ComfyUI frame interpolation：官方文件將它定位為 video frame interpolation，用於提高 frame rate、smooth motion、temporal consistency。  
  <https://docs.comfy.org/tutorials/utility/frame-interpolation>
- WAS Create Morph Image：blend two images，常用於 animated GIFs / transition effect。  
  <https://www.runcomfy.com/comfyui-nodes/was-node-suite-comfyui/Create-Morph-Image>
- IP-Adapter：有效、輕量的 image prompt adapter，可與 text prompt / ControlNet 等工具合用。  
  <https://ip-adapter.github.io/>  
  <https://github.com/tencent-ailab/IP-Adapter>
- ComfyUI_IPAdapter_plus：ComfyUI reference implementation，將 reference image 的 subject/style 轉移到 generation。  
  <https://github.com/cubiq/ComfyUI_IPAdapter_plus>
- FLUX.1 Fill：BFL 官方 inpainting / outpainting model；ComfyUI 有 Flux Fill workflow。  
  <https://bfl.ai/blog/24-11-21-tools>  
  <https://huggingface.co/black-forest-labs/FLUX.1-Fill-dev>  
  <https://docs.comfy.org/tutorials/flux/flux-1-fill-dev>
- Flux Fill + reference image / Flux Redux 社群 workflow。  
  <https://learn.thinkdiffusion.com/the-inpainting-revolution-how-reference-images-with-flux-fill-and-flux-redux-are-changing-the-game/>
- ComfyUI Crop & Stitch：裁切 masked area sampling，再 stitch 回原圖，避免 unmasked part 被 VAE 改動。  
  <https://github.com/comfyorg/comfyui-crop-and-stitch>
- Panorama image inpainting / panorama tools / panorama diffusion：處理 360 panorama inpainting/outpainting/generation，但不是 A/B fixed adapter。  
  <https://github.com/swhan0329/panorama_image_inpainting>  
  <https://github.com/Flyguygx/sd-webui-panorama-tools>  
  <https://arxiv.org/html/2307.03177v5>  
  <https://github.com/Insta360-Research-Team/DiT360>  
  <https://arxiv.org/html/2404.07949v1>
- Game background / tileset adjacency references。  
  <https://gbstudiocentral.com/tips/create-an-almost-seamless-transition/>  
  <https://www.boristhebrave.com/2025/02/04/generating-tilesets-with-stable-diffusion/>
- I2SB / Schrödinger bridge：image restoration / image-to-image translation 方向，不是 product pipeline for A/B spatial adapter。  
  <https://i2sb.github.io/>  
  <https://github.com/NVlabs/I2SB>  
  <https://proceedings.mlr.press/v202/liu23ai.html>

### 本報告針對 pano-loop-lab 的推論

- 沒有找到直接成熟解法，尤其沒有找到「給定 A.right + B.left + fixed width -> output adapter with pixel-exact endpoints」的現成 repo / ComfyUI workflow。
- Image morphing / interpolation 是最像的研究方向，但產出是 temporal / semantic middle state，不是 spatial weld。
- Dual-reference / IPAdapter / Flux reference-guided fill 是最值得工程化的突破口，因為它補上 SDXL inpaint 缺少的 visual reference。
- Crop-and-stitch / hard composite 仍是不可替代的 anchor 保護層。
- 下一輪不應再只是改 mask width / prompt，而應加入 concept/reference conditioning。


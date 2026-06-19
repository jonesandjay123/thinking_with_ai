# ComfyUI Panorama Mask-Inpaint Model Research

日期：2026-06-19  
主題：在 ComfyUI 裡，哪一類 inpaint / fill 模型最適合解決 `pano-loop-lab` 的 anchor-preserved panoramic adapter 問題？

## TL;DR

一句話建議：第一輪用 `diffusers/stable-diffusion-xl-1.0-inpainting-0.1` 做 baseline / smoke test，但不要只靠模型保留 anchor；必須用「生成 soft mask + 最終 hard-mask composite / crop-and-stitch」保證左右 anchor 像素級不變。第二輪改試 RealVisXL / JuggernautXL 類寫實 SDXL checkpoint 加 Fooocus inpaint patch 或 inpaint workflow。Flux.1 Fill 值得作為第二輪後段或第三輪高品質候選，但在 RTX 5080 16GB 上應先用裁切 band / quantized 或 offload workflow，不建議第一輪就把它當唯一主線。

明確排名：

1. 第一輪 smoke test：SDXL official inpaint + hard composite verification。
2. 第二輪 quality candidate：JuggernautXL / RealVisXL / DreamShaperXL 類 checkpoint + Fooocus inpaint patch 或 Crop & Stitch workflow。
3. 高品質 / production candidate：Flux.1 Fill [dev]，尤其用於中央 transition band 的高品質地形與光線銜接。
4. 暫時不建議：Higgsfield whole-frame image-to-image 當最終 adapter、純 prompt img2img 不做 mask、只靠 CSS `blend=16` 掩蓋結構錯誤、在沒有 final composite 的情況下相信任何 inpaint 模型會「自然保留」黑區。

## 我們的 use case

`pano-loop-lab` 的核心不是一般修圖，也不是人物補洞，而是替兩張既有 panoramic scene plates 生成中間 adapter：

```text
dawn-valley -> adapter -> dusk-ridge
```

已知條件：

- `loop renderer / selector` 已可用。
- Higgsfield whole-frame image-to-image 可以生成漂亮 adapter。
- 但 Higgsfield 沒有 mask inpaint，左右 anchor 會被整張重畫。
- 結果是單張漂亮，但 `blend=0` butt-join 接縫仍明顯。
- 下一步要改用 ComfyUI mask inpaint：
  - `init image = adapter-work-canvas.png`
  - `mask = adapter-mask.png`
  - white = regenerate
  - black = preserve
  - 只重生中央 transition band
  - 左右 anchor 最後要像素級保留

本案成功標準不是「圖看起來漂亮」而已，而是：

1. 左右 anchor 黑區像素級保留。
2. 中央 transition 自然。
3. `dawn-valley right edge | adapter left edge` 的 butt-join 接縫明顯改善。
4. `adapter right edge | dusk-ridge left edge` 的 butt-join 接縫明顯改善。
5. `blend=0` 要比 Higgsfield c08 / c04 好。
6. `blend=16` 可以更自然，但不能只靠 CSS 掩蓋結構錯誤。

重要推論：任何 diffusion inpaint 都可能在 VAE encode/decode、latent sampling、mask blur、resize、stitching 時讓非 mask 區域產生微小像素差。因此本案要把「模型有 mask inpaint 能力」和「最終 anchor pixel exact」視為兩件事。前者靠 inpaint/fill 模型；後者靠 deterministic composite 或 crop-and-stitch。

## 模型類型比較表

| 類型 | 代表模型 / 節點 | 優點 | 缺點 | 是否適合本案 | 建議用途 |
| --- | --- | --- | --- | --- | --- |
| SDXL official inpaint | `diffusers/stable-diffusion-xl-1.0-inpainting-0.1` | 標準、可預期、真正 mask inpaint UNet；ComfyUI 有基礎 workflow；16GB VRAM 較可控 | 畫質與 prompt 美術感可能不如社群 checkpoint / Flux；1024 訓練尺度，超寬 panoramic 要裁切或縮放 | 適合。尤其適合先證明 mask / composite / weld 評估流程 | Round 1 baseline / smoke test |
| 寫實 SDXL checkpoint + inpaint workflow | RealVisXL、JuggernautXL、DreamShaperXL、Realism Engine XL 等 | 寫實 / cinematic / landscape 品質通常更好；可配 Fooocus patch 或 inpaint conditioning | 有些不是原生 inpaint；workflow 差異大；不加 final composite 仍可能動到 anchor | 適合第二輪。對 matte painting / terrain texture 可能比官方 SDXL inpaint 更漂亮 | Round 2 quality refinement |
| Fooocus inpaint patch / ComfyUI inpaint nodes | `Acly/comfyui-inpaint-nodes`、Fooocus inpaint patch | 可把一般 SDXL checkpoint 轉成 inpaint model；適合沿用喜歡的 checkpoint 風格 | 需要 custom nodes / patch model；distilled checkpoint 不適用；workflow 需驗證 | 很適合第二輪，特別是想用 JuggernautXL / RealVisXL 的風格 | Round 2 candidate |
| Crop & Stitch / ImageCompositeMasked | `comfyui-crop-and-stitch`、`ImageCompositeMasked` | 不是模型，但對本案最重要：可只處理 masked area，最後 stitch/composite 回原圖，避免非 mask 區域被 VAE 改動 | 需要 workflow discipline；mask blur 與 final hard mask 要分清楚 | 必要。這是 anchor pixel exact 的保險，不是可選項 | 每一輪都要用 |
| Flux.1 Fill / Flux inpaint | `FLUX.1 Fill [dev]`、ComfyUI Flux Fill workflow | BFL 官方定位就是 inpainting/outpainting；prompt following 與自然銜接潛力高 | 12B、模型大、下載需同意 license；16GB VRAM 要小心；ComfyUI workflow / issues 仍比 SDXL 複雜 | 值得，但不建議第一輪唯一主線 | Round 2 late / Round 3 production candidate |
| ControlNet / Tile / Depth / Canny | SDXL ControlNet Tile/Depth/Canny、Flux Depth/Canny | 可幫地形、遠山、地平線、邊緣結構一致；Tile/Tiled VAE 可協助大圖 | ControlNet 太強會卡死 transition；Canny 可能把舊接縫也固化 | 有幫助，但應第二輪後才開 | Round 2/3 refinement |

## SDXL official inpaint 評估

### 來源讀到的事實

Hugging Face 的 `diffusers/stable-diffusion-xl-1.0-inpainting-0.1` model card 說明它是 SDXL inpainting 模型，具備使用 mask 進行 inpainting 的額外能力；它由 `stable-diffusion-xl-base-1.0` 初始化，訓練解析度是 `1024x1024`，inpainting UNet 額外有 5 個 input channels，包含 encoded masked image 與 mask 本身。來源：<https://huggingface.co/diffusers/stable-diffusion-xl-1.0-inpainting-0.1>

ComfyUI 官方 inpainting 文件將 inpainting workflow 定義為修改影像中特定區域，並介紹 `VAE Encoder (for Inpainting)`。來源：<https://docs.comfy.org/tutorials/basic/inpaint>

社群教學也顯示 SDXL official inpaint 在 ComfyUI 中可直接下載 `diffusion_pytorch_model.fp16.safetensors`，約 5.14GB，放入 ComfyUI model folder 使用。來源：<https://mybyways.com/blog/using-the-sdxl-inpainting-01-model>、<https://sbcode.net/genai/sdxl-inpainting/>

### 針對本案的判斷

官方 SDXL inpaint 是合理第一步，原因不是它一定最好看，而是它最適合把未知數拆開：

- 它是真正支援 mask inpaint 的 baseline。
- 它比 Flux Fill 小、workflow 更標準，適合 RTX 5080 16GB 先跑通。
- 如果它都無法改善 `blend=0` butt-join，問題可能在 mask / canvas / anchor geometry，而不只是模型品質。
- 它的弱點是自然景觀、遠山、霧、光線 transition 的美術品質可能不如 JuggernautXL / RealVisXL / Flux Fill。

因此現在 Codex 正在用 `diffusers/stable-diffusion-xl-1.0-inpainting-0.1` 是合理 baseline / smoke test 選擇。它的優勢是穩定、標準、可預期，而不是最終品質必然最好。

本案第一輪不應該問「SDXL official inpaint 是否能直接產出最美 adapter」，而應該問：

- mask 白區是否真的被重生？
- mask 黑區經過 final hard composite 後是否 byte/pixel exact？
- 用相同 renderer 檢查 `blend=0`，兩側接縫是否比 Higgsfield whole-frame 好？

如果第一輪通過 weld test，再換更高品質模型才有意義。

## Realistic SDXL checkpoints + inpaint workflow 評估

### 來源讀到的事實

RealVisXL 是 SDXL family 的 photorealistic fine-tuned / merged checkpoint，目標是高品質寫實生成，資料頁描述它適合 realistic images、varied settings，也提到它由 SDXL base 及 DreamShaper XL、Juggernaut XL 等模型特徵融合而來。來源：<https://openlaboratory.com/models/realvis-xl/>

社群討論中，有使用者提到 RealVisXL 有 inpaint version，也有人提到 JuggernautXL inpaint；同一串也有觀點認為 SDXL official inpaint 比單純 base inpaint 好，但「still not great」，也有人指出對 SDXL 而言可以透過 Fooocus / InvokeAI / ComfyUI 等 inpaint 方法讓非 inpaint checkpoint 工作。來源：<https://www.reddit.com/r/StableDiffusion/comments/1b79jbu/so_does_no_one_use_specific_inpainting_models/>

ComfyUI inpaint nodes 專案說明 Fooocus inpaint patch 可套用在 SDXL checkpoint 上，將其轉成 inpaint model，用於填補與擴展影像區域；但也提醒要用 regular checkpoint，distilled merges 如 Turbo / Lightning / Hyper 不適用。來源：<https://github.com/Acly/comfyui-inpaint-nodes>

### 針對本案的判斷

這類模型很可能是第二輪品質提升的主力。原因：

- 本案中央 transition 是 landscape / matte painting，寫實 SDXL checkpoint 通常比官方 base inpaint 更會處理材質、光線、霧、遠山。
- 如果 anchor 已由 final composite 保證，模型只需要負責「白區生成」與「接近邊界的自然過渡」，不需要負責保護黑區。
- Fooocus inpaint patch 讓一般 SDXL checkpoint 更容易用於 inpaint，而不是被迫只用 official SDXL inpaint。

風險：

- 這類 checkpoint 的社群版本、inpaint 版本、patch workflow 差異很大，不如 official SDXL baseline 可重現。
- 有些模型偏人像或商業攝影，不一定比官方模型更會做 panoramic matte painting。
- 如果 mask 太窄、沒有足夠 context，模型會生成局部漂亮但全景不連貫的 band。
- 如果 final output 沒有做 hard composite，仍可能因 VAE 或 stitch blur 造成 anchor 變化。

建議優先試：

1. JuggernautXL 或 RealVisXL regular checkpoint + Fooocus inpaint patch。
2. Realism Engine XL / DreamShaperXL 作為風格替代。
3. 若找到可信的 dedicated JuggernautXL inpaint / RealVisXL inpaint checkpoint，可作為同輪 A/B。

## Fooocus inpaint patch / ComfyUI inpaint nodes 評估

### 來源讀到的事實

`Acly/comfyui-inpaint-nodes` 提供 Fooocus inpaint model 相關 nodes。README 說它是小而彈性的 patch，可套用在 SDXL checkpoint 上，把 checkpoint 轉成 inpaint model，用於 seamless fill / expand。它也提供 `VAE Encode & Inpaint Conditioning`，讓 inpaint model 能結合 masked area 既有內容。來源：<https://github.com/Acly/comfyui-inpaint-nodes>

### 針對本案的判斷

Fooocus patch 對 `pano-loop-lab` 很有價值，因為本案要的不是修掉物件，而是讓中央 band 變成符合左右 plates 的景觀過渡。這需要美術風格與地形生成能力。官方 SDXL inpaint 是標準 baseline；Fooocus patch 則讓我們把更會畫景觀的 checkpoint 變成 inpaint candidate。

但它不是第一輪 smoke test 的最佳起點，因為它引入 custom node、patch model、checkpoint 相容性等額外變數。建議在 Round 1 的 mask/composite/renderer 評估可靠之後再上。

## Flux Fill / Flux inpaint 評估

### 來源讀到的事實

Black Forest Labs 官方介紹 FLUX.1 Tools 時，將 `FLUX.1 Fill` 定義為 inpainting / outpainting model，能基於 text description 與 binary mask 編輯與擴展真實或生成影像；同篇也提到 FLUX.1 Depth / Canny 可用 depth map / canny edges 做結構引導。來源：<https://bfl.ai/blog/24-11-21-tools>

Hugging Face 的 `black-forest-labs/FLUX.1-Fill-dev` model card 說它是 12B rectified flow transformer，可根據文字描述填補既有影像中的區域；key features 包含高品質、prompt following、完成 source image structure。它的檔案下載需要同意 license / usage agreement，Diffusers 使用 `FluxFillPipeline`，範例中有 `image`、`mask_image`、`height`、`width`、`guidance_scale`、`num_inference_steps` 等參數。來源：<https://huggingface.co/black-forest-labs/FLUX.1-Fill-dev>

ComfyUI 官方文件已有 `Flux.1 fill dev` inpainting / outpainting workflow，列出需要 `flux1-fill-dev.safetensors`、`clip_l.safetensors`、`t5xxl_fp16.safetensors`、`ae.safetensors`，並提示要先在 Hugging Face 同意模型條款。來源：<https://docs.comfy.org/tutorials/flux/flux-1-fill-dev>

同時，ComfyUI GitHub 上存在 Flux Fill 相關問題回報：有使用者在 2026-06 回報最新 ComfyUI 中 Flux Fill mask 被忽略，輸出與輸入相同；也有舊 issue 回報 Flux Fill 結果模糊、低對比。這不能證明 Flux Fill 不可用，但說明 workflow / 版本 / 節點組合仍有踩坑風險。來源：<https://github.com/Comfy-Org/ComfyUI/discussions/14467>、<https://github.com/Comfy-Org/ComfyUI/issues/6765>

### 針對本案的判斷

Flux Fill 值得用，而且可能是最接近本案 production candidate 的模型類型。原因：

- 它原生定位就是 masked fill / outpaint，不是把一般 txt2img checkpoint 硬改成 inpaint。
- 本案中央 band 需要自然地形、霧、遠山、光線 transition；Flux 系列的 prompt following 和整體一致性可能比 SDXL official inpaint 更好。
- BFL 官方也有 Depth / Canny tools，未來若要加結構 guidance，Flux 生態可延伸。

但 Flux Fill 不適合作為第一輪唯一候選：

- full model 是 12B，ComfyUI 官方 workflow 需要大模型 + T5 XXL + VAE；16GB VRAM 可以嘗試，但比 SDXL 更容易碰到 offload / quantization / speed 問題。
- 本案 native `3168x1344` 尺寸對任何 latent diffusion 都偏大；Flux 更應先裁切 central band 或用 crop-and-stitch。
- ComfyUI support 已有官方 workflow，但 issues 顯示版本與 workflow 仍可能出現 mask 行為異常或品質坑。
- Flux Fill [dev] license / access 需要同意條款，不像一般 SDXL checkpoint 那麼直接。

結論：Flux Fill 值得作為第二輪或第三輪候選。具體而言，先用 SDXL official inpaint 跑通 weld test；再用 JuggernautXL / RealVisXL + Fooocus patch 做品質 A/B；如果仍卡在中央 transition 不自然或光線/地形理解不足，再上 Flux Fill 做高品質候選。

## ControlNet / Tile / Depth 是否有必要

### 來源讀到的事實

BFL 官方把 Flux Depth / Canny 定義為結構引導工具：Depth 基於 depth map，Canny 基於 canny edges。來源：<https://bfl.ai/blog/24-11-21-tools>

ComfyUI-TiledDiffusion 專案說明它支援大型影像生成與 upscaling，在有限 VRAM 下使用 Tiled Diffusion / Tiled VAE，支援 SDXL、FLUX、ControlNet，並支援 ultra-large image generation。來源：<https://github.com/shiimizu/ComfyUI-TiledDiffusion>

`comfyui-crop-and-stitch` 說明其 Inpaint Crop / Stitch nodes 可在 sampling 前裁切 masked area，sampling 後 stitch 回原圖，不修改 unmasked part，甚至不讓 unmasked part 經過 VAE encode/decode。來源：<https://github.com/comfyorg/comfyui-crop-and-stitch>

`ImageCompositeMasked` 的 node documentation 說明它可用 mask 將 source image composited 到 destination image，mask 決定 source 的哪些部分被貼上。來源：<https://comfyui-wiki.com/en/comfyui-nodes/image/image-composite-masked>

### 針對本案的判斷

ControlNet / Tile / Depth 不是第一輪必要條件，但第二輪後很可能有用。

建議順序：

1. 先不用 ControlNet，驗證 mask / composite / join metrics。
2. 若中央地形方向亂、遠山高度跳動，再加 Depth / Canny guidance。
3. 若細節不足或 native resolution 太大，再加 Tiled VAE / Crop & Stitch / tiled upscale。
4. Tile ControlNet 可用於 texture/detail refinement，但不應讓它主導構圖。

注意：Canny / Depth 的來源圖要小心。本案不是要複製 Higgsfield c08/c04 的錯誤接縫，也不是要把 dawn/dusk edge 硬拉成一條折線。控制圖應該來自 adapter work canvas 的合理 structure sketch / value map，而不是直接把既有 bad seam 固化。

## 對 RTX 5080 16GB 的可行性

### SDXL official inpaint

可行性高。`fp16` inpaint UNet 約 5GB 等級，ComfyUI SDXL inpaint workflow 在 16GB VRAM 上應該比 Flux 容易。真正要避免的是直接在 full `3168x1344` 上全圖 latent inpaint，因為超寬圖的 latent 面積與 VAE memory 會增加。

建議第一輪尺寸：

- 先以 downscaled canvas 或 central band crop 開始。
- 目標可以是 `1536x640`、`1792x768`、`2048x864` 這類接近原比例但仍可控的尺寸。
- 若用 Crop & Stitch，可強制 crop 到 SDXL 友善尺寸，例如 `1536x768` 或 `1024x768`，再 stitch 回原 canvas。

### SDXL checkpoint + Fooocus patch

可行性中高。checkpoint 通常 6GB 左右，16GB VRAM 有機會跑；但 custom node、patch、VAE、ControlNet 會提高記憶體需求。第二輪建議仍採 central band crop，而不是 full native。

### Flux Fill

可行性中等，值得但要保守。官方 / model card 顯示 Flux Fill 是 12B 模型，ComfyUI workflow 需要 diffusion model、T5 XXL、CLIP-L、VAE；另有 GitHub issue 提到實際使用的 `flux1-fill-dev.safetensors` 約 23.2GB。這不等於 16GB VRAM 不能跑，因為可用 CPU offload、FP8 / GGUF / quantized variants、裁切 band 等方式，但速度、穩定性與 workflow 複雜度會明顯高於 SDXL。

因此 RTX 5080 16GB 上的 Flux Fill 建議：

- 不直接 full native `3168x1344`。
- 先 crop central transition band。
- 若 full precision 不穩，嘗試 FP8 / GGUF / quantized community workflow。
- 必須保留 final hard composite，不能因 Flux 高品質就放棄 anchor verification。

## 建議 workflow

### 共通原則：兩層 mask

本案最重要的工程原則是分離兩種 mask：

1. `generation_mask_soft`：給模型用，可 grow / blur，讓邊界過渡自然。
2. `final_composite_mask_hard`：給最後合成用，黑區必須完全保留原始 anchor，不能 blur 到 anchor pixel。

流程上應該是：

```text
adapter-work-canvas.png
  -> inpaint/fill white transition band
  -> generated candidate
  -> ImageCompositeMasked / Inpaint Stitch 回 original canvas
  -> hard-verify left/right anchor pixels identical
  -> run loop renderer at blend=0 and blend=16
```

如果 `generation_mask_soft` 需要讓模型看見一點 anchor context，可以在 sampling crop 中包含 anchor context；但最終輸出仍必須用 `final_composite_mask_hard` 把黑區換回原始 pixels。

### Round 1 — baseline weld test

目標：不是追求最美，而是證明 mask inpaint 能比 Higgsfield whole-frame 改善 `blend=0` butt-join，且 anchor pixel exact。

- 模型：`diffusers/stable-diffusion-xl-1.0-inpainting-0.1`
- Workflow：
  - ComfyUI official SDXL inpaint / VAE Encode for Inpainting
  - 或搭配 `comfyui-crop-and-stitch`
  - 最後一定用 `ImageCompositeMasked` 或 Inpaint Stitch 回原圖
- 尺寸：
  - 不建議第一輪直接 full `3168x1344`
  - 優先：central band crop / downscaled full canvas
  - 候選：`1536x640`、`1792x768`、`2048x864`
- Denoise：
  - 若中央 band 需要完全重建：`0.85-1.0`
  - 若 work canvas 已有 rough transition 可保留：`0.65-0.85`
- Mask blur：
  - generation soft mask：`16-32px` 起跳，依 seam 寬度調
  - final composite hard mask：`0px` 或只在白區內部 feather，不能侵入 anchor 黑區
- Steps：`25-35`
- CFG：`5-7`
- Sampler：先用穩定的 DPM++ 2M / SDE Karras 類；不要第一輪同時開太多 sampler 變數
- ControlNet：關
- 驗收：
  - anchor pixel diff = 0
  - `blend=0` seam snapshots / selector 分數比 Higgsfield c08 / c04 好
  - 若圖較醜但 seam 明顯改善，Round 1 仍算成功

### Round 2 — quality refinement

目標：在 Round 1 weld 可行後，提高中央 transition 的景觀 / matte painting 品質。

- 模型候選：
  - JuggernautXL regular + Fooocus inpaint patch
  - RealVisXL regular + Fooocus inpaint patch
  - DreamShaperXL / Realism Engine XL 作為替代
  - 若找到可靠 dedicated inpaint checkpoint，可加入 A/B
- Workflow：
  - `Acly/comfyui-inpaint-nodes` Fooocus inpaint
  - `comfyui-crop-and-stitch` crop central band / stitch back
  - final hard composite verification
- 尺寸：
  - central band crop forced to SDXL-friendly size
  - 可試 `2048x864` 或 native crop，但不要一開始全圖
- Denoise：
  - `0.75-0.95`
  - 如果邊界破壞太大，降低 denoise 或縮小 soft mask overlap
- Mask blur：
  - soft mask `24-48px`
  - hard composite 保留 anchor
- ControlNet：
  - 第一組 A/B 先關
  - 若遠山 / 地平線漂移，再加 Depth 或 Canny
  - ControlNet strength 低到中，避免把 bad seam 固化
- 目標：
  - 中央 transition 更自然
  - dawn/dusk 兩側光線與地形可讀
  - `blend=0` 接縫不因美術提升而退步

### Round 3 — native resolution / production candidate

目標：產出可進 production candidate 的 adapter，不只低解析驗證。

路線 A：SDXL / Fooocus patch production

- 用 Crop & Stitch 在 native canvas 上只裁中央 transition band。
- Crop 區域包含足夠左右 context，但 final hard composite 保證 anchor。
- 若 crop 太大，先 downscale crop 到模型友善尺寸，生成後 stitch 回 native。
- 用 Tiled VAE / TiledDiffusion 降低 VRAM 壓力。
- 可做 2-pass：
  1. structure pass：低步數、低變數，產生自然地形。
  2. detail pass：低 denoise refinement，只作用在中央白區。

路線 B：Flux Fill production candidate

- 使用 ComfyUI Flux.1 Fill workflow。
- 先裁切 central transition band，不直接 full `3168x1344`。
- 若 full precision VRAM 不穩，試 FP8 / GGUF / CPU offload。
- 保留 final hard composite，不讓 Flux output 直接成為 final output。
- 若 Flux Fill 生成的中央 band 品質明顯優於 SDXL，但邊界稍不穩，可混合策略：
  - Flux 生成中央 70-80% band
  - SDXL / Crop & Stitch 或 deterministic blend 處理靠 anchor 的 10-15% 過渡

路線 C：low-res generate + native anchor composite

- 先低解析生成 adapter candidate。
- 將 generated central band upscale / color match 到 native。
- 只把中央 white region composite 回 native work canvas。
- 最後以 pixel diff 驗證 anchors。

Native resolution 的硬規則：

- 不管用哪個模型，最後輸出都要跑 anchor diff。
- 不允許任何 non-mask anchor pixel 因 VAE decode、resize、blur、color correction 變動。
- `blend=16` 只能作為附加展示，不能替代 `blend=0` weld test。

## 最終建議

### 1. 第一輪 smoke test

用 `diffusers/stable-diffusion-xl-1.0-inpainting-0.1`。

理由：它是標準、真正 mask-aware 的 SDXL inpaint baseline，ComfyUI 支援成熟，16GB VRAM 比 Flux 更可控。這一輪要驗證的是 workflow：mask 是否正確、anchor 是否 pixel exact、`blend=0` 是否改善。

### 2. 第二輪 quality candidate

用 JuggernautXL / RealVisXL / DreamShaperXL 類寫實 SDXL checkpoint + Fooocus inpaint patch / Crop & Stitch。

理由：本案中央 band 是 landscape / matte painting transition，社群寫實 checkpoint 很可能比 official SDXL inpaint 更會處理光線、材質、遠山與霧。但它們要在 Round 1 workflow 穩定後再引入。

### 3. 高品質 / production candidate

Flux.1 Fill [dev]。

理由：Flux Fill 是官方定位的 inpaint / outpaint fill model，對自然銜接、prompt following、source image structure completion 很有潛力。它值得做第二輪後段或第三輪候選，尤其當 SDXL 中央 transition 不夠自然時。

但不要第一輪就壓全部賭注在 Flux，因為 16GB VRAM、模型下載/授權、ComfyUI workflow、版本相容與速度都是額外變數。

### 4. 暫時不建議

- 只用 Higgsfield whole-frame adapter 當最終方案：會重畫 anchor，與本案成功標準衝突。
- 只用一般 img2img checkpoint，不做 inpaint conditioning / mask：無法控制 anchor。
- 只靠 mask blur 或 CSS `blend=16`：可能讓視覺變柔，但無法保證 `blend=0` 結構接縫。
- 沒有 final hard composite 的任何 workflow：就算模型理論上 preserve mask，也不能保證 pixel exact。
- 第一輪就加入 ControlNet / Flux / upscale / 多 sampler 大矩陣：會讓失敗原因難以定位。

## 來源與判斷標註

### 來源讀到的資料

- SDXL official inpaint model card：模型是 mask inpaint，基於 SDXL base，1024 訓練，inpaint UNet 有額外 mask / masked-image channels。  
  <https://huggingface.co/diffusers/stable-diffusion-xl-1.0-inpainting-0.1>
- ComfyUI official inpainting workflow：介紹 inpainting workflow、mask editor、`VAE Encoder (for Inpainting)`。  
  <https://docs.comfy.org/tutorials/basic/inpaint>
- SDXL inpaint ComfyUI 安裝 / workflow 參考：`fp16` inpaint safetensors 約 5.14GB，放入 ComfyUI models 使用。  
  <https://mybyways.com/blog/using-the-sdxl-inpainting-01-model>  
  <https://sbcode.net/genai/sdxl-inpainting/>
- RealVisXL model report：RealVisXL 是 SDXL photorealistic fine-tuned / merged model，適合 realistic images / varied settings，與 DreamShaperXL、JuggernautXL 等相關。  
  <https://openlaboratory.com/models/realvis-xl/>
- SDXL inpaint 社群討論：有人使用 RealVisXL / JuggernautXL inpaint；也有人認為 SDXL official inpaint 比 base inpaint 好但仍不是完美。  
  <https://www.reddit.com/r/StableDiffusion/comments/1b79jbu/so_does_no_one_use_specific_inpainting_models/>
- Fooocus inpaint nodes：可將 SDXL checkpoint 轉成 inpaint model；distilled checkpoint 不適用；提供 inpaint conditioning 相關 node。  
  <https://github.com/Acly/comfyui-inpaint-nodes>
- BFL Flux Tools：`FLUX.1 Fill` 是 inpainting/outpainting model，使用 text description 與 binary mask；Flux Depth / Canny 提供結構 guidance。  
  <https://bfl.ai/blog/24-11-21-tools>
- FLUX.1 Fill [dev] model card：12B rectified flow transformer，可填補 image areas；需要同意 license；Diffusers 用 `FluxFillPipeline`。  
  <https://huggingface.co/black-forest-labs/FLUX.1-Fill-dev>
- ComfyUI Flux.1 Fill dev workflow：官方 ComfyUI 文件已有 Flux Fill inpainting / outpainting workflow，列出 model / text encoder / VAE 檔案位置。  
  <https://docs.comfy.org/tutorials/flux/flux-1-fill-dev>
- ComfyUI Flux Fill issues：有 mask ignored、品質模糊/低對比等問題回報，表示 workflow / 版本仍需驗證。  
  <https://github.com/Comfy-Org/ComfyUI/discussions/14467>  
  <https://github.com/Comfy-Org/ComfyUI/issues/6765>
- ComfyUI-TiledDiffusion：支援 limited VRAM 的 large image / upscaling、Tiled VAE、SDXL / FLUX / ControlNet。  
  <https://github.com/shiimizu/ComfyUI-TiledDiffusion>
- ComfyUI Crop & Stitch：可裁切 masked area sampling，再 stitch 回原圖，不修改 unmasked part，甚至不經過 VAE encode/decode。  
  <https://github.com/comfyorg/comfyui-crop-and-stitch>
- ImageCompositeMasked：可用 mask 將 source image composite 到 destination image，mask 決定 source 哪些部分被貼上。  
  <https://comfyui-wiki.com/en/comfyui-nodes/image/image-composite-masked>

### 本報告針對 pano-loop-lab 的推論

- `anchor pixel exact` 不能只交給 diffusion model；必須在最後用 hard composite / crop-and-stitch 驗證。
- 第一輪最重要是 weld test，而不是畫面美術分數。
- SDXL official inpaint 是合理 baseline，但不是最終品質上限。
- 寫實 SDXL checkpoint + Fooocus patch 可能是最好的第二輪性價比。
- Flux Fill 很值得做，但因 VRAM / workflow / license / issue 風險，應放第二輪後段或第三輪。
- ControlNet Depth / Canny / Tile 應等 baseline 通過後再加，否則容易把壞接縫或錯誤結構固化。


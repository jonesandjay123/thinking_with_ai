# Fashion Design Clothing Simulation Tools Research

日期：2026-06-19  
主題：有沒有適合服裝設計師使用、能輔助模擬服裝的工具 / Repo / 平台？如果自己做小 app，可以從哪些技術核心切入？

## TL;DR

有，而且這個領域已經很成熟，但要分清楚三種完全不同的工具：

1. **專業服裝設計 / 打版 / 3D 試衣模擬平台**：CLO 3D、Browzwear VStitcher、Style3D、Marvelous Designer、Optitex、Gerber AccuMark。這是服裝設計師最接近「真的拿來工作」的方向。
2. **開源 / 可改造 Repo**：Seamly2D、FreeSewing、GarmentCode、Costumy、Garment-Pattern-Generator。這類很適合研究、教學、做小 app 原型，但不像 CLO 那樣開箱即用。
3. **AI virtual try-on / 生成式試穿**：CatVTON、IDM-VTON、OOTDiffusion、Outfit Anyone。這類適合快速做「把衣服穿到模特身上」的展示 app，但它不是嚴格的打版或布料物理模擬。

給櫻井妹妹的最實際建議：

- 如果她是想找「服裝設計師真的會用的軟體」：先看 **CLO 3D**，再比較 **Browzwear / Style3D / Marvelous Designer**。
- 如果她想找「開源 Repo，可以拿來做自己的小工具」：先看 **FreeSewing + Seamly2D + GarmentCode**。
- 如果今天要做一個小 web/mobile app：不要一開始做完整 3D 布料物理引擎；先做 **款式靈感板 + 量身參數 + 2D pattern / AI try-on preview + tech pack/export**，更容易做出可用 demo。

## 需求拆解：她說的「模擬服裝介面」可能有四種意思

服裝設計工具其實分成幾個層級：

| 層級 | 使用者想做什麼 | 代表工具 / 技術 | 適合程度 |
| --- | --- | --- | --- |
| 2D 款式 / 打版 | 畫 pattern、改尺寸、輸出紙樣 | Seamly2D、Optitex、AccuMark、FreeSewing | 很適合服裝設計師 |
| 3D 布料模擬 | 把 2D pattern 縫到 avatar 上，看垂墜、鬆量、皺摺 | CLO、Browzwear、Style3D、Marvelous Designer | 最接近「模擬服裝」 |
| AI 試穿預覽 | 給一張人像 + 一張衣服圖，生成穿上去的效果 | CatVTON、IDM-VTON、OOTDiffusion | 適合展示 / 電商 / 原型，不是精準打版 |
| 作品集 / moodboard / tech pack | 管理靈感、色卡、布料、款式規格 | Figma、Notion、自製 app、PLM 類工具 | 適合做小 app MVP |

所以如果櫻井妹妹想要的是「像服裝設計師工作台」：CLO / Browzwear / Style3D 是主線。  
如果她想要的是「AI 幫我看這件衣服穿起來怎樣」：CatVTON / IDM-VTON / OOTDiffusion 是主線。  
如果她想要的是「我想自己做一個小 app」：FreeSewing / Seamly2D / AI try-on API 是比較實際的起點。

## 現成主流平台

### 1. CLO 3D

來源資訊：CLO 官方將它定位為 3D fashion design software，可建立 virtual、true-to-life garment visualization，並使用 simulation technologies。來源：<https://www.clo3d.com/en/>

適合：

- 服裝設計師、打版師、品牌開發。
- 2D pattern + 3D avatar + fabric simulation。
- 做作品集、digital sample、virtual fitting。

優點：

- 服裝設計圈最常被提到的主流工具之一。
- 工作流接近真實製衣：pattern、縫線、布料、avatar、試穿。
- 對學生作品集很有價值。

缺點：

- 商業軟體，要學習成本。
- 如果只是做簡單 prototype，會比 AI try-on 重很多。

結論：如果櫻井妹妹要認真學「數位服裝設計」，CLO 3D 是第一順位。

### 2. Browzwear / VStitcher

來源資訊：Browzwear 官方說它是 digital apparel design & development software，協助 fashion companies speed up time to market、improve fit、create design/sell/make workflow。來源：<https://browzwear.com/>

適合：

- 品牌、企業、供應鏈協作。
- 2D/3D digital sample、fit review、merchandising。

優點：

- 偏企業與供應鏈，真實服裝產業採用度高。
- Vogue 也提到 Browzwear 被用於推動 fashion brands 的 3D design adoption。來源：<https://www.vogue.com/article/fashion-brands-embrace-3d-design>

缺點：

- 對個人 / 學生可能比 CLO 更 enterprise。
- 不一定是最容易入門。

結論：適合了解產業級工具，但初學或小 app 概念不一定從它開始。

### 3. Style3D

來源資訊：Style3D 官方說它提供 AI and 3D fashion design workflow，包含 design、development、fabric、collaboration 等 end-to-end fashion workflow；Style3D Studio 則定位為 3D fashion design software with high-quality simulation。來源：<https://www.style3d.com/>、<https://studio.style3d.com/>

適合：

- 想看 AI + 3D fashion 最新產品化方向。
- 設計、布料數位化、供應鏈協作。

優點：

- 很積極把 AI 和 3D fashion workflow 結合。
- 有從設計靈感、3D 模擬、布料、製造協作一路串起來的產品敘事。

缺點：

- 開放程度、價格、教育版可得性需要個別查。
- 若只是做小 app，直接整合 Style3D 不一定容易。

結論：值得當「未來產品方向」參考，尤其是 AI inspiration + 3D simulation + collaboration。

### 4. Marvelous Designer

來源資訊：Marvelous Designer 官方 trial page 將它稱為 digital cloth simulation tool，主打 pattern-based modeling and cloth simulation。來源：<https://www.marvelousdesigner.com/product/trial>

適合：

- 3D artist、遊戲 / 影視角色服裝、digital fashion。
- 需要漂亮、可信的布料垂墜與皺摺。

優點：

- 在 3D / game / character art 圈很常用。
- 做概念服裝、角色衣服、虛擬時裝非常強。

缺點：

- 相比 CLO，Marvelous 更偏 3D content creation；若目標是實體製衣 / 工業打版，CLO/Optitex/AccuMark 可能更貼。

結論：如果櫻井妹妹偏創作、角色、作品集、虛擬服裝，Marvelous 很適合；如果偏現實服裝製作，先看 CLO。

### 5. Optitex / Gerber AccuMark

來源資訊：Optitex 官方說它的 Pattern Design Software 結合 2D CAD design 與 3D virtual prototyping；Gerber AccuMark 官方說它是 2D/3D digital patternmaking software，協助 fashion companies develop collections with fit。來源：<https://optitex.com/products/2d-and-3d-cad-software/>、<https://www.lectra.com/en/fashion/products/gerber-accumark-fashion>

適合：

- 產業級 patternmaking / grading / production。
- 技術設計、打版、製造端。

優點：

- 專業工業流程更完整。
- 與量產、版型、放碼、規格流程更接近。

缺點：

- 對學生 / 小 app 來說太重。
- 學習門檻和企業採購成本較高。

結論：可當產業標竿，不建議小 app 從這種級別開始。

## 開源 / GitHub Repo 候選

### 1. Seamly2D

來源資訊：Seamly 官方說 Seamly2D 是 free and open source fashion design software；GitHub README 說 Seamly2D 是 open source patternmaking software，GPLv3+，可在 Windows / macOS / Linux 使用，也支援 multi-size measurement files / individual measurement files。來源：<https://seamly.io/>、<https://github.com/fashionfreedom/seamly2d>

適合：

- 2D pattern drafting。
- 自訂尺寸、紙樣、打版教學。
- 想做「服裝設計 app」的 pattern 核心參考。

優點：

- 真開源。
- 接近服裝設計師實際需要的 pattern workflow。
- 適合學習 pattern / measurement / drafting concepts。

缺點：

- 不是現代 web UI。
- 不含完整高品質 3D simulation。

結論：若要做「服裝設計師工具」而不是純 AI 圖片 app，Seamly2D 是最值得研究的開源專案之一。

### 2. FreeSewing

來源資訊：FreeSewing 官方說它是 open source software to generate bespoke sewing patterns；developer docs 說它是 open source JavaScript library for parametric sewing patterns / on-demand garment manufacturing。來源：<https://freesewing.eu/>、<https://freesewing.dev/>、<https://github.com/freesewing>

適合：

- web app。
- 使用者輸入身體尺寸，生成客製化 sewing pattern。
- 快速做 demo：量身表單 -> pattern SVG/PDF。

優點：

- JavaScript / web-friendly。
- parametric pattern 很適合小 app。
- 對教學、個人化服裝、slow fashion 很適合。

缺點：

- 主要是 pattern generation，不是 3D cloth simulation。
- 款式種類與設計自由度受 pattern library 限制。

結論：如果今天要做一個網頁小 app，FreeSewing 是最實際的開源核心。

### 3. GarmentCode

來源資訊：GarmentCode 是 modular programming framework for designing parametric sewing patterns；論文摘要說它是 garment modeling DSL，讓設計者用 hierarchical、component-oriented programming 設計 sewing patterns，並能 retarget 到不同 body shapes。來源：<https://github.com/maria-korosteleva/GarmentCode>、<https://korosteleva.com/publication/garmentcode/>、<https://arxiv.org/abs/2306.03642>

適合：

- 研究型 prototype。
- 用程式化參數生成 garment components。
- 未來做 AI-to-pattern / design-to-pattern 的 backbone。

優點：

- 很接近「把服裝設計變成可編程元件」。
- 對自製 app 的 architecture 很有啟發。

缺點：

- 不像 CLO / Seamly 那樣可直接給設計師用。
- 需要懂 Python / geometry / simulation pipeline。

結論：不是給初學設計師直接用的 app，但非常值得工程端研究。

### 4. Costumy

來源資訊：Costumy README 說它是 open source prototype，可將 2D clothing patterns 變成 3D garments，並包含 mesh measurement、使用 FreeSewing 生成 patterns 等功能。來源：<https://github.com/cdrinmatane/Costumy>

適合：

- 研究「2D pattern -> 3D garment」的小 app 原型。
- 看 open-source 系統怎麼串 FreeSewing / 3D mesh / avatar fitting。

優點：

- 比 Seamly / FreeSewing 更接近「2D 到 3D」。
- 對自製 prototype 很有參考價值。

缺點：

- prototype 性質，不是成熟商業工具。
- 需要評估維護狀態。

結論：如果想做自己的 demo，可以參考它的方向，但不要期待直接 production-ready。

### 5. Garment-Pattern-Generator / GarmentCodeData

來源資訊：Garment-Pattern-Generator README 說它生成了 19 garment types、20,000+ garment samples 的 3D garment + sewing pattern dataset；docs 提供 Maya UI 來檢視 parametrized garment template、測試參數、simulation / rendering setup。來源：<https://github.com/maria-korosteleva/Garment-Pattern-Generator>

適合：

- 研究資料集、訓練模型、生成 3D garment samples。
- 不是一般設計師直接用的工具。

結論：適合 AI / research，不適合第一個小 app。

## AI Virtual Try-On / 生成式試穿

### 1. CatVTON

來源資訊：CatVTON GitHub 說它是 ICLR 2025 virtual try-on diffusion model，特點是 lightweight network、parameter-efficient training，以及 simplified inference；project page 說它只需要 garment reference、target person image 和 mask，不需要 pose estimation / human parsing / text input，並聲稱 1024x768 inference 小於 8GB VRAM。來源：<https://github.com/Zheng-Chong/CatVTON>、<https://zheng-chong.github.io/CatVTON/>

適合：

- 做 AI 試穿 demo。
- 使用者上傳人物照 + 衣服圖 -> 看穿上效果。

優點：

- 推論需求相對友善。
- 較適合小 app prototype。

缺點：

- 不是 pattern design。
- 無法告訴你版型、縫線、布料物理是否可製作。

結論：如果小 app 目標是「給設計師/顧客快速看效果」，CatVTON 很值得試。

### 2. IDM-VTON

來源資訊：IDM-VTON official repo 說它是 Improving Diffusion Models for Authentic Virtual Try-on in the Wild 的官方實作；project page 說它提升 image fidelity 與 detail preservation。來源：<https://github.com/yisol/IDM-VTON>、<https://idm-vton.github.io/>

另有 ComfyUI adaptation：`ComfyUI-IDM-VTON` README 提到目前實作需要至少 16GB VRAM。來源：<https://github.com/TemryL/ComfyUI-IDM-VTON>

適合：

- 高品質 virtual try-on。
- ComfyUI workflow 實驗。

優點：

- 效果在社群中常被拿來做高品質 try-on demo。
- 有 Hugging Face / ComfyUI 生態。

缺點：

- 16GB VRAM 起跳，對一般筆電/手機不友善。
- 仍是視覺試穿，不是嚴格服裝設計工具。

結論：適合接 RTX 5080 / ComfyUI 做 demo，不適合直接手機端本地跑。

### 3. OOTDiffusion

來源資訊：OOTDiffusion GitHub 是 AAAI 2025 paper 官方實作，主題是 Outfitting Fusion based Latent Diffusion for Controllable Virtual Try-on；Hugging Face paper page 摘要說它用 outfitting UNet 生成 high-fidelity and controllable virtual try-on images while preserving garment details。來源：<https://github.com/levihsu/OOTDiffusion>、<https://huggingface.co/papers/2403.01779>

適合：

- 研究 controllable virtual try-on。
- 做視覺展示 / 穿搭生成。

缺點：

- 實際部署複雜。
- 不提供 pattern / 3D garment construction。

結論：可以列入 AI try-on 候選，但小 app MVP 我會先看 CatVTON 或 API 化服務。

## 如果今天要自己做一個小 app：最實際的方向

### MVP A：服裝設計 moodboard + AI 試穿

這是最容易做出「哇，有用」的版本。

功能：

- 上傳設計草圖 / 衣服圖 / 參考圖。
- 上傳模特照或選 avatar。
- 用 CatVTON / IDM-VTON / 商用 API 做 virtual try-on。
- 儲存不同版本：布料、顏色、袖長、領型。
- 生成簡單設計卡：正面圖、prompt、布料 notes、色票、尺寸 notes。

技術核心：

- 前端：Next.js / React Native / Expo。
- 後端：Python FastAPI。
- AI：CatVTON / IDM-VTON / Replicate / Hugging Face / ComfyUI API。
- 儲存：Firebase / Supabase。

優點：

- 最快做出可展示成果。
- 適合教學、作品集、創作流程。

限制：

- 不是真正打版，不保證可製作。

### MVP B：量身參數 -> 2D pattern generator

這比較像真正的服裝設計工具。

功能：

- 輸入身體尺寸：胸圍、腰圍、臀圍、肩寬、袖長等。
- 選基礎款式：T-shirt、shirt、skirt、pants。
- 用 FreeSewing 生成 SVG/PDF pattern。
- 使用者可調整 ease、length、fit。
- 匯出 PDF / SVG / DXF-like assets。

技術核心：

- FreeSewing JS library。
- SVG editor / Canvas editor。
- Measurement schema。
- Pattern versioning。

優點：

- 比 AI try-on 更接近服裝製作。
- web app 可行度高。

限制：

- 3D 模擬仍缺。
- 設計自由度受 pattern template 限制。

### MVP C：2D pattern editor + lightweight 3D preview

這是最有野心但難度最高的版本。

功能：

- 2D pattern pieces。
- 縫線 pairing。
- Avatar / mannequin。
- 3D drape preview。
- fabric presets。

技術核心：

- 2D：SVG / Canvas / Paper.js / Fabric.js。
- 3D：Three.js / Babylon.js。
- 物理：Ammo.js / Rapier / WebGPU cloth simulation；或後端 Blender / Maya / custom simulator。
- 開源參考：Costumy、GarmentCode、Garment-Pattern-Generator。

風險：

- 真實布料 simulation 非常難，尤其要穩定又好用。
- 若只是課程或小 app，不建議第一版就做。

建議：可以先做「假的 3D preview」：用 avatar + AI render / try-on image 取代物理 simulation，等需求確定再做真 3D。

## 給櫻井妹妹的建議路線

如果她問「我可以用什麼現成工具？」：

1. 先試 CLO 3D 或 Marvelous Designer。
2. 如果她想學實體服裝/打版：CLO 優先。
3. 如果她偏 3D 角色/虛擬服裝：Marvelous Designer 優先。
4. 如果她想找免費開源 pattern 工具：Seamly2D / FreeSewing。
5. 如果她想做 AI 試穿：CatVTON / IDM-VTON / OOTDiffusion。

如果她問「我們能不能自己做？」：

可以，但要選對切入口。

最推薦的小 app 方向：

```text
Design Board + Measurement + AI Try-On + Pattern Export
```

第一版不要追求完整 CLO 替代品，而是做一個服裝設計學生會真的想用的小工具：

- 靈感圖 / 草圖收集。
- 款式卡管理。
- 色票 / 布料 notes。
- AI try-on preview。
- FreeSewing pattern export。
- 最後輸出一頁 tech pack / presentation board。

這樣既有視覺效果，也有設計 workflow，不會卡死在布料物理 simulation。

## 最終推薦

### 1. 直接現成工具

- **CLO 3D**：服裝設計師主線，最值得先試。
- **Marvelous Designer**：虛擬服裝 / 3D 角色服裝非常強。
- **Style3D**：看 AI + 3D fashion workflow 的未來方向。
- **Browzwear / Optitex / Gerber AccuMark**：產業級，適合了解但不一定適合學生入門。

### 2. 開源 Repo / 可做小 app 的核心

- **FreeSewing**：最適合 web app 的 parametric pattern core。
- **Seamly2D**：最成熟的開源 2D pattern drafting 工具。
- **GarmentCode**：最值得工程端研究的 programmable garment pattern DSL。
- **Costumy**：2D pattern -> 3D garment prototype 參考。

### 3. AI / 視覺展示

- **CatVTON**：最適合小 app prototype 的 lightweight virtual try-on。
- **IDM-VTON**：高品質，但需要較強 GPU。
- **OOTDiffusion**：值得研究，但部署成本較高。

### 4. 不建議第一版做

- 自己從零做完整布料物理 simulation。
- 自己做 CLO / Marvelous 替代品。
- 只做 AI 圖片生成卻沒有 pattern / measurement / design workflow。
- 手機端本地跑大型 try-on model。

## 來源

- CLO 3D official：3D fashion design software / true-to-life garment visualization。  
  <https://www.clo3d.com/en/>
- Browzwear official：digital apparel design and development platform。  
  <https://browzwear.com/>
- Vogue：fashion brands adopt 3D design / Browzwear and digital product workflows。  
  <https://www.vogue.com/article/fashion-brands-embrace-3d-design>
- Style3D official / Studio：AI + 3D fashion design workflow / high-quality simulation。  
  <https://www.style3d.com/>  
  <https://studio.style3d.com/>
- Marvelous Designer trial：digital cloth simulation / pattern-based modeling and cloth simulation。  
  <https://www.marvelousdesigner.com/product/trial>
- Optitex 2D/3D CAD：2D CAD design + 3D virtual prototyping。  
  <https://optitex.com/products/2d-and-3d-cad-software/>
- Gerber AccuMark / Lectra：2D/3D digital patternmaking。  
  <https://www.lectra.com/en/fashion/products/gerber-accumark-fashion>
- Seamly / Seamly2D：open-source fashion design / patternmaking software。  
  <https://seamly.io/>  
  <https://github.com/fashionfreedom/seamly2d>
- FreeSewing：open-source JavaScript library for parametric sewing patterns。  
  <https://freesewing.eu/>  
  <https://freesewing.dev/>  
  <https://github.com/freesewing>
- GarmentCode：programming framework / DSL for parametric sewing patterns。  
  <https://github.com/maria-korosteleva/GarmentCode>  
  <https://korosteleva.com/publication/garmentcode/>  
  <https://arxiv.org/abs/2306.03642>
- Costumy：open-source prototype turning 2D clothing patterns into 3D garments。  
  <https://github.com/cdrinmatane/Costumy>
- Garment-Pattern-Generator：3D garments with sewing patterns dataset / generator。  
  <https://github.com/maria-korosteleva/Garment-Pattern-Generator>
- CatVTON：ICLR 2025 lightweight virtual try-on diffusion model。  
  <https://github.com/Zheng-Chong/CatVTON>  
  <https://zheng-chong.github.io/CatVTON/>
- IDM-VTON：authentic virtual try-on in the wild。  
  <https://github.com/yisol/IDM-VTON>  
  <https://idm-vton.github.io/>  
  <https://github.com/TemryL/ComfyUI-IDM-VTON>
- OOTDiffusion：outfitting fusion based latent diffusion for controllable virtual try-on。  
  <https://github.com/levihsu/OOTDiffusion>  
  <https://huggingface.co/papers/2403.01779>


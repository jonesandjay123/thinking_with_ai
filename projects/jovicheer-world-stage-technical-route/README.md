# Jovicheer World Stage 技術路線調查報告

> 撰寫時間：2026-06-12  
> 對象：Jovicheer 固定鏡頭 / 導演式鏡頭 cinematic web world  
> 核心邊界：Jovicheer 是祈願絲帶世界，不是遊戲；沒有角色、沒有自由移動、沒有自由旋轉鏡頭，不做 full open world。

## 0. Context

Jovicheer 想做的是一個在 web 上運行、能和 React / HTML UI / wish flow 整合的「祈願絲帶世界舞台」。它需要讓使用者感覺自己看見一個高質感、會呼吸的世界，但技術目標不是重建一套完整 3A engine。

這份報告刻意避開：

- full open-world game
- free camera 3D exploration
- character controller / walking character
- multiplayer
- real cloth simulation as mandatory requirement
- heavy Unity WebGL game embedded as whole website
- overly complex physics

真正目標是：

> 固定鏡頭的高質感場景 + 可互動的祈願物件 + 少量導演式鏡頭運動 + HTML / React wish flow。

## 1. Executive Recommendation

主推薦：**Route D — Hybrid Cinematic Web Diorama**。

更精確地說，Jovicheer 最適合走：

> **D as production pipeline, B1 as browser runtime, C as authoring support.**

也就是：

- 用 **React + R3F / Three.js** 做瀏覽器 runtime。
- 用 **AI image / matte painting / Blender / Unreal** 產出背景、遠景、光影、render layers、depth maps、局部 GLB props。
- 在 browser 裡只做最值得即時互動的東西：ribbon、wind particles、mist、light motes、clickable anchors、hidden wishes、opening ritual、staged dolly-in / pan。
- UI 保持 HTML / React overlay，不把表單、文字、wish flow 硬塞進 3D canvas。

我不建議 Jovicheer 第一階段走純 B1，也不建議走純 C。

純 B1 的問題是：agent 很容易做出能跑的 Three.js scene，但若背景、光影、美術方向、材質、霧、鏡頭節奏沒有被強力約束，成品會快速滑向「便宜 3D demo」。R3F 很適合實作互動層，但不應承擔整個高質感世界的美術來源。

純 C 的問題是：Blender / Unity / Unreal 很適合做漂亮場景和鏡頭，但若最後變成整個 Unity / Unreal WebGL runtime 嵌進網站，會太重、web UI 整合差、迭代慢，也和 Jovicheer 的「祈願流程 / web product」本質不合。

Route D 的優勢是它承認 illusion 比 simulation 重要：遠景和大氣感用預渲染 / layered assets 鎖住品質；互動只保留最有情緒價值的部分。這最接近「3A game menu / cinematic scene 的 illusion」，而不是「真的做 3A game」。

## 2. Route Comparison Table

| Route | Visual Ceiling | Speed to Prototype | Engineering Difficulty | Asset Difficulty | Cheap Demo Risk | Mobile Feasibility | Agent Assist Level | Recommendation |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| B1 — R3F / Three.js runtime with image/layer background | 中高。若有好背景 plate、layered planes、mist、ribbon shader，可很美；若全靠 primitive 3D，天花板低 | 快。1-2 週可做可點擊固定鏡頭 prototype | 中。R3F、camera、raycast、particles、responsive、performance 都可控 | 中。需要背景圖、透明層、ribbon texture、少量 props | 高。最容易變「漂浮幾何 + bloom + 粒子」demo | 高。可做 aggressive fallback | 高。AI agent 很擅長寫 runtime、互動、state、shader 初版 | 適合做 runtime，但不該獨自承擔完整 production pipeline |
| C — External 3D authoring pipeline | 最高。Blender / Unreal 可做真正 cinematic 光影、霧、景深、材質 | 中慢。漂亮場景需要美術迭代 | 低到高不等。若只輸出 render layers，web 工程低；若上 Unity/Unreal WebGL，工程高 | 高。場景、美術、材質、燈光、render passes 都是主成本 | 中。工具本身不 cheap，但匯到 web 後可能融合差 | 中。預渲染很可行；完整 engine runtime 不理想 | 中。AI 可 script 場景、整理 assets、批量渲染；審美仍需人控 | 適合作為 authoring / render pipeline，不建議作為整站 web runtime |
| D — Hybrid Cinematic Web Diorama | 很高。用預渲染鎖住大氣，runtime 補互動生命感 | 中快。第一個場景 2-3 週可驗證 | 中。需要前端 + asset pipeline 的清楚分工 | 中高。比純 B1 更需要素材，但比完整 3D 世界輕 | 中低。只要背景品質好，cheap demo 風險大幅下降 | 高。可把重效果預渲染，手機只跑少量 WebGL | 高。AI 可完成 60-80% runtime / tooling / pipeline glue | **主推薦** |

## 3. Route B1 — R3F / Three.js Runtime With High-Quality Image/Layer Background

### 技術可行性

B1 很可行，而且是 Jovicheer web runtime 的核心。Three.js 是 browser 3D 的成熟基礎；R3F 是 Three.js 的 React renderer，適合把 3D scene 做成 React component，和網站 state、routing、HTML overlay 整合。R3F 官方 performance 文件也明確提醒：不要在 `useFrame` 或 fast events 裡反覆 `setState`，應直接 mutate 物件或用 delta / spring 類方式處理連續動畫。這對 Jovicheer 很重要，因為 ribbon、particles、wind 都是 per-frame 動畫。

適合 B1 的 scene structure：

```text
WorldStage
├── HTML / React overlay
│   ├── wish input
│   ├── anchor label
│   ├── ritual CTA
│   └── accessibility / fallback content
├── R3F Canvas
│   ├── fixed PerspectiveCamera
│   ├── background plate plane
│   ├── far / mid / near image layers
│   ├── depth-map displaced background plane or parallax planes
│   ├── 3D ribbon strips
│   ├── small GLB anchor props
│   ├── mist / light mote particles
│   └── invisible hit targets for anchors
└── CSS / DOM atmospheric layers
    ├── noise
    ├── vignette
    └── subtle overlay gradients
```

### 成品效果可能長什麼樣

好的 B1 成品不是「一堆 3D 物件在空間裡轉」，而是像一張會呼吸的電影分鏡：

- 遠景是一張高品質 matte painting 或 Blender/AI render。
- 幾層透明 foreground / midground image planes 有很小的 parallax。
- ribbon 是真 3D flat strip，會被風輕推，不是 tube。
- 祈願 anchor 是少量可點擊 3D prop 或 image cutout。
- 點擊 anchor 時 camera 只做 3-8% 的 dolly-in / pan / focus shift。
- 粒子、mist、light motes 只在空間感上補一點生命，不搶主視覺。

### 是否容易做出 premium cinematic 感

可以，但條件是：**不要讓 runtime 生成所有視覺品質**。

Premium 感主要來自：

- 強構圖的背景 plate
- 統一的光源方向
- 景深 / haze / foreground occlusion
- ribbon 的材質和厚薄控制
- 低頻、克制的運動
- UI 和 world 的節奏協調

R3F 負責讓它「活」，但不應負責從零變美。

### 是否容易變 cheap 3D demo

非常容易。典型翻車樣子：

- 背景是 gradient sky，前面放幾個 torus / sphere / tube。
- 全場 bloom 開太重，所有東西都像塑膠。
- ribbon 用 `TubeGeometry`，看起來像蛇或水管。
- particle 太多、太均勻，像 template。
- 相機自由 orbit，立刻變成 model viewer，不像導演式場景。
- 光影不一致：背景是黃昏，3D 物件卻像白棚商品照。

### 參考案例 / demo / library / tutorial / repo

- [R3F examples](https://r3f.docs.pmnd.rs/getting-started/examples)：值得看 component 化、pointer interaction、3D card / shopping demo 的結構；不要照抄 showcase 風格。
- [R3F performance pitfalls](https://r3f.docs.pmnd.rs/advanced/pitfalls)：必讀。Jovicheer 的風、ribbon、particles 不應每幀觸發 React state 更新。
- [R3F scaling performance](https://r3f.docs.pmnd.rs/advanced/scaling-performance)：可參考 on-demand rendering、instancing、LOD、texture 管理等策略。
- [Drei showcase / helpers](https://storybook.js.org/showcase/pmndrs-drei/)：Drei 是 R3F 常用 helper 集合；Jovicheer 可用 `Html`、`Environment`、`Billboard`、`Float`、`Cloud` 類工具，但要小心 helper demo 味。
- [gltfjsx](https://github.com/pmndrs/gltfjsx)：把 GLB 轉成 R3F component，適合 anchor props / small scene props。
- [Theatre.js with R3F](https://www.theatrejs.com/docs/0.5/getting-started/with-react-three-fiber)：適合導演式 camera / light / prop keyframe；若要調鏡頭節奏，比手寫很多 magic numbers 更像 production workflow。
- [GSAP ScrollTrigger docs](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)：適合 scroll-linked 或 staged entrance；但 Jovicheer prototype 先用 click-to-focus，不必一開始做 heavy scroll cinematic。
- [Codrops Animated 3D Ribbons with Three.js](https://tympanus.net/codrops/2021/11/29/animated-3d-ribbons-with-three-js/)：值得看 ribbon geometry / motion 的視覺方向；不要照抄過度抽象、科技感強的 ribbon。
- [Codrops depth-map / WebGPU scanning effect](https://tympanus.net/codrops/2025/03/31/webgpu-scanning-effect-with-depth-maps/)：值得參考「深度圖 + shader 讓 2D image 有空間感」；Jovicheer 不應照抄 scanning / sci-fi 效果。

### 每個場景搭建時間成本

若走 B1：

- simple scene：3-5 天。可用 AI 生成背景 + R3F planes + 1 條 ribbon + particles。
- polished scene：2-4 週。需要背景 plate、分層、光影匹配、ribbon material、mobile fallback。
- highly cinematic scene：4-8 週。需要外部 authoring、render passes、細緻 camera、sound / timing / postprocessing。

### AI coding agent 是否能大部分完成

能完成 runtime 的大部分：

- R3F scene scaffold
- layer planes
- camera transition
- anchor hit target
- GSAP / react-spring / Theatre integration
- particle system 初版
- ribbon geometry 初版
- responsive / fallback / performance guard
- build / deploy

但 AI agent 不能保證：

- 背景構圖好看
- ribbon 像布而不是蛇
- 光影融合
- 品牌氣質準確
- 場景節奏克制

所以需要 Jones / GPT / art direction 介入視覺判斷。

### 主要坑點

- R3F scene 太「前端 demo」。
- 使用太多 postprocessing 掩蓋美術不足。
- texture 太大導致手機卡。
- 不限制 DPR，Retina 手機 GPU 壓力爆。
- 每幀更新 React state。
- 3D object 和背景 plate perspective 不一致。
- click hit target 太小或被 HTML overlay 擋住。

## 4. Route C — External 3D Authoring Pipeline

### Blender 適合做什麼

Blender 最適合 Jovicheer 的角色是：

- 固定鏡頭場景 blocking
- 低成本建構 props / ribbons / anchors
- 材質與燈光 preview
- Cycles / Eevee render plate
- render layers / transparent foreground layers
- depth map / mist pass / shadow pass
- texture baking
- GLB / glTF props export
- Python scripting 批量輸出 camera / layer / pass

Blender 的 glTF exporter 是官方 manual 支援的路線，適合把模型、材質、部分動畫輸出成 web-friendly `.glb` / `.gltf`。Blender Python API 也能 script camera、render、scene settings，這讓 AI agent 可以協助重複性工作。

Blender 的優點：

- 免費、可 script、可控、適合 agent 協作。
- 和 web 的 GLB pipeline 直接。
- 對 fixed-camera diorama 很適合。
- 可做 baked lighting，讓 browser 只吃貼圖成本。

Blender 的缺點：

- 要漂亮仍需要美術眼睛。
- 霧、半透明、volume 在 web 端不一定能直接保真。
- Cycles render 迭代可能慢。
- 3D 新手容易在 scale、origin、UV、texture baking 卡住。

### Unity 適合做什麼

Unity 對 Jovicheer 比較適合做：

- 快速使用 Unity Asset Store 場景包
- Timeline / Cinemachine 做 camera previs
- 輸出 reference video loop / animation timing
- 如果未來有更 interactive 的 ritual flow，可做互動 prototype

Unity 的 Cinemachine 是控制 Unity camera 的套件，Timeline + Cinemachine 可建立切鏡、dolly、blend 等 cinematic sequence。這對「導演式固定鏡頭」概念很吻合。

但我不建議第一階段把 Unity WebGL build 當網站 runtime。原因：

- bundle heavy
- React / HTML UI 整合麻煩
- mobile browser 風險高
- update / deploy / debugging 都比普通 web stack 重
- Jovicheer 沒有角色控制和大型互動，不值得背 Unity runtime 成本

### Unreal Engine 適合做什麼

Unreal 最適合做：

- 最高視覺上限的 cinematic previs
- Lumen / Nanite / volumetric fog / lighting lookdev
- Sequencer camera animation
- Movie Render Queue 輸出高品質 image sequence / video / render passes
- Quixel / Fab assets 組大場景

Unreal Sequencer / Movie Render Pipeline 是做 cinematic render 的強項。若 Jovicheer 要一張非常漂亮的「世界舞台主視覺」或 trailer-like background，Unreal 很有吸引力。

但 Unreal 不適合：

- 作為 Jovicheer web runtime
- 小團隊頻繁改 web product flow
- 每個 wish interaction 都在 Unreal 裡做
- 需要 SEO / HTML / React form 的網站核心體驗

### 哪個最適合固定鏡頭、漂亮光影、祈願世界場景

如果只選一個外部 authoring 工具：

> **Blender 最適合第一階段 production pipeline。Unreal 適合高階 lookdev / cinematic reference。Unity 目前不是主線。**

理由：

- Blender 和 web 的 GLB / layer render / depth map / transparent PNG pipeline 最順。
- Blender 可被 AI agent / Python script 協助，且不需要引入 game engine project complexity。
- Unreal 視覺天花板更高，但 project setup、asset 管理、render pipeline、授權/資源流程較重；適合在確定 visual target 後做 hero shot，不適合第一個 prototype 的主要依賴。
- Unity 的強項偏互動遊戲 / camera tool / asset store；Jovicheer 第一階段不需要 Unity runtime。

### 是否能用 AI agent / MCP / scripting 協助搭場景

可以，但要分清楚「可協助」和「可替代美術」。

AI agent 可以：

- 寫 Blender Python 建 camera、lights、planes、import assets。
- 批量設定 render resolution / transparent background / depth pass。
- 把 GLB assets 優化、命名、放到 web repo。
- 生成 R3F loader / material assignment code。
- 產出 scene checklist。
- 幫忙跑 image compression。

AI agent 不可靠的地方：

- 最終構圖
- 「像祈願世界」的 taste
- asset 是否俗氣
- 霧氣是否剛好
- ribbon 動態是否有布感

### 現成 asset library

- [Poly Haven](https://polyhaven.com/)：CC0 HDRIs、textures、models；很適合拿 HDRI / PBR 材質 /自然 props。License 很乾淨，官方標示 CC0。
- [Sketchfab](https://sketchfab.com/features/free-3d-models)：大量 free / Creative Commons / store models；適合找 anchor props，但要嚴格檢查 license、polycount、texture quality、attribution。
- [BlenderKit / Blendkit](https://www.blenderkit.com/)：Blender 內建式 asset library，含 models、materials、HDRIs、scenes；適合快速搭 scene，但要注意免費/付費與 license。
- [Fab / Quixel Megascans](https://quixel.com/megascans/free)：高品質 scan assets；2025 後免費政策變動，需要逐項確認是否 free / acquired / license。適合 Unreal lookdev 或 Blender 場景，但不要假設全部免費。
- Unity Asset Store / Unreal Marketplace / Fab：適合買現成自然場景、神社/寺院/花園/幻想 props，但要檢查 web polycount、授權、材質格式。
- Mixamo：基本無關。它主打 3D characters、rigging、character animations；Jovicheer 沒有人物角色，所以除非未來做「布條骨架動畫」旁路，不應成為主要工具。

### 每個場景搭建時間成本

若走 C 作 authoring：

- simple scene：1-2 週。用 assets + camera + render plate。
- polished scene：3-6 週。需要 moodboard、asset curation、lighting、render layers、web integration。
- highly cinematic scene：6-12 週。需要 lookdev、render passes、camera choreography、multiple iterations。

### 匯出到 web 的常見方式

- video loop：最穩，互動最低；適合 hero ambience / loading / fallback。
- image sequence：品質高、可控制，但檔案量大；適合短 ritual animation。
- layered renders：背景 / midground / foreground / mist / light / shadow 分層，web 端做 parallax。
- depth map：用 shader 做 2.5D displacement / parallax。
- GLB / glTF props：少量 anchor / ribbon holder / stone / bell / wish object 進 R3F。
- transparent PNG / WebP foreground layers：樹枝、絲帶、霧、門框、祈願物件輪廓。
- reference animation：先在 Blender/Unreal 做 camera motion，再把 keyframes 手工或 script 轉給 R3F / Theatre.js。

### web 整合成本

最低：video / image layers + HTML overlay。  
中等：layered planes + depth map + R3F anchors。  
最高：完整 GLB scene + realtime lights + postprocessing + runtime particles。

Jovicheer 第一版應該選中間：預渲染遠景 + R3F 局部即時互動。

## 5. Route D — Hybrid Cinematic Web Diorama

### 這是否是最適合 Jovicheer 的 production pipeline

是。Route D 最符合 Jovicheer 的真實需求。

Jovicheer 不需要做「可探索的 3D 世界」。它需要的是：

- 一個觀眾願意停留的祈願場景
- 可被點擊的 anchor / wish object
- ribbon / 風 / 霧 / 光粒讓世界像活著
- staged camera motion 讓互動有儀式感
- HTML / React wish flow 能清楚、可靠、可維護

Hybrid diorama 正好把「視覺品質」和「互動產品」分開：

- 視覺品質由預渲染 / 美術資產 / authoring tool 鎖住。
- 互動產品由 web runtime 做。

這比純 Three.js 場景更漂亮，比 Unity/Unreal WebGL 整站更輕。

### 如何分工：哪些該預渲染，哪些該即時 3D

應該預渲染 / 預製：

- sky / far background / large landscape
- 大面積光影
- 遠景建築 / 山 / 樹 / 願望牆輪廓
- 主要霧氣低頻層
- 大面積 shadow / AO / baked lighting
- 複雜材質、細節 props 的 beauty render
- optional depth map
- loading / ritual cutaway 的短 video loop

應該在 browser 即時做：

- 1-3 條主要 ribbon 的微動
- light motes / small particles
- foreground mist drift
- clickable anchor hit targets
- hidden wish reveal
- staged camera dolly-in / pan
- hover / focus / active state
- HTML overlay / wish form / accessibility copy

不要即時做：

- 真正 cloth simulation 作為第一階段必要條件
- volumetric fog raymarching
- full realtime GI
- hundreds of unique physics ribbons
- fully navigable 3D world
- in-canvas UI form

### 第一個 prototype 應該怎麼做

第一個 prototype 名稱可叫：

> **Jovicheer World Stage P0 — One Anchor, One Ribbon, One Wish**

目標不是做完整世界，而是驗證：

1. 固定鏡頭世界是否有 premium 感。
2. ribbon 是否不像蛇、不像塑膠管。
3. click anchor -> camera staged movement -> hidden wish reveal 是否有情緒。
4. HTML wish flow 是否能自然疊在 cinematic world 上。
5. mobile 是否可接受。

詳細定義見第 8 節。

### 每個場景搭建時間成本

Route D 粗估：

- simple scene：1 週左右。
- polished scene：2-4 週。
- highly cinematic scene：5-10 週。

和純 C 相比，D 的高階場景成本較低，因為不需要整個場景都 interactive；和純 B1 相比，D 的品質更穩，因為遠景不用 runtime 從零 fake。

### AI agent 可以幫到哪裡

AI agent 在 D 路線價值很高：

- 把 visual spec 轉成 R3F scene scaffold。
- 寫 `RibbonMesh` / `WindField` / `AnchorHotspot` / `CameraDirector` components。
- 寫 Theatre.js / GSAP timeline。
- 寫 asset manifest / layer alignment config。
- 寫 image compression pipeline。
- 寫 mobile fallback。
- 寫 Playwright screenshot tests。
- 幫整理 assets source/license。
- 幫寫 Blender Python export scripts。

但最終 still needs human taste：

- 哪張背景有 Jovicheer 氣質
- 哪個 anchor 有祈願感
- 風的速度是否太焦躁
- ribbon 的曲線是否像有重量
- UI 是否破壞沉浸

### 素材來源與 asset library

建議來源順序：

1. AI image / matte painting：快速定義 mood、構圖、世界觀。
2. Blender：拆 layer、補 props、做 depth / foreground / shadow pass。
3. Poly Haven：HDRI / natural materials。
4. Sketchfab / BlenderKit：小型 anchor props，需 license audit。
5. Fab / Quixel：高品質自然 assets，尤其若用 Unreal 做 lookdev。
6. 自製 ribbon texture：不要用隨機 cloth texture；要符合 Jovicheer 色彩和厚薄。

## 6. 技術名詞 / 工具適配判斷

### Web runtime

#### Three.js

適合。Three.js 是 Jovicheer runtime 的底層 3D library。它適合 scene、camera、mesh、shader、particles、GLB props、raycasting。若未來需要更細 shader 控制，可以直接用 Three.js primitives。

注意：直接 raw Three.js 和 React app 整合時，要自己管理 lifecycle、resize、dispose、state bridge。

#### React Three Fiber / R3F

非常適合。Jovicheer 是 React / HTML UI + 3D world 的混合產品，R3F 的 component model 比 raw Three.js 更好維護。它也適合 AI agent 生成結構化 component。

注意：避免把 fast animation 綁到 React state；遵守 R3F performance pitfalls。

#### @react-three/drei

適合，但要克制。Drei 提供 `Html`、`Environment`、`Float`、`Billboard`、`Sparkles`、`Cloud`、`useGLTF` 等 helper，能加速 prototype。

風險：Drei demo 味很強；若照搬 `Sparkles` / `Float` 預設參數，容易像 template。

#### GSAP

適合 camera / object staged timeline，尤其 click-to-focus、ritual sequence、UI entrance。ScrollTrigger 也成熟，但 Jovicheer 第一版不必過度 scroll-driven。

#### Framer Motion

適合 HTML / React overlay，不適合作為 3D scene 主動畫系統。用它處理 wish panel、button、modal、text transition 很好。

#### WebGL

現在的實際 production target。Three.js / R3F 的主要穩定 runtime 仍是 WebGL。Jovicheer 第一版應以 WebGL + graceful fallback 為主。

#### WebGPU

暫不作為第一版主線。Three.js 的 `WebGPURenderer` 已有官方文件，會在支援時使用 WebGPU backend，否則 fallback 到 WebGL 2 backend。但 WebGPU / TSL 生態仍比 WebGL 更動態，Drei / R3F / postprocessing 相容性也要小心。

建議：prototype 可研究 WebGPU depth-map / particles，但 production 第一版不要依賴 WebGPU。

#### PixiJS

可作為 2D layered diorama alternative，但不是主推薦。PixiJS 是高效 2D WebGL / WebGPU renderer，很適合 2D particles、parallax、sprite layers。如果 Jovicheer 完全不想碰 3D props，PixiJS 很合理。

但 Jovicheer 需要少量 3D ribbon / anchors / camera depth illusion，R3F 更合適。PixiJS 可作為 pure 2D fallback 或 loading/ritual animation layer。

#### Canvas 2D

適合非常輕量的 fog/noise/particle fallback，不適合作為主要 world runtime。Canvas 2D 可畫 drifting mist、grain、light motes，但 depth、3D ribbon、GLB props 會很難。

#### Lottie / Rive

適合 UI micro-interaction，不適合 world stage 主場景。

Lottie 適合 After Effects exported JSON，做 icon、ritual mark、small looping ornament。Rive 適合 interactive vector state machine，例如 wish seal、button ritual、small symbolic animation。

它們不適合承擔大面積 cinematic world，因為 canvas/vector animation 和 3D depth / lighting / particles 的融合有限。

#### Spline

適合快速 visual prototype / pitch，不建議作為 production runtime 主線。Spline 的 browser 3D design、collaboration、web export 很方便，能快速做概念展示。

風險：

- 產物容易有 Spline 風格。
- performance / asset control / shader control 不如自管 R3F pipeline。
- 深度整合 wish flow、custom ribbon shader、asset optimization 時可能受限。

建議：可用 Spline 做 mood prototype，不作為 Jovicheer world stage 的主架構。

#### Theatre.js

很適合 Jovicheer 的「導演式鏡頭」。Theatre.js 官方提供 Three.js / R3F integration，可用視覺化 keyframe 控制 camera、lights、material colors、object transforms。

建議：P0 可以先用 GSAP / react-spring；P1 若需要反覆調鏡頭節奏，導入 Theatre.js。

#### react-spring / drei animation helpers

適合自然微動、hover / focus transition、click-to-focus 的 spring feel。對 ribbon 主動態可用自訂 shader / `useFrame`，對 camera / anchor transition 可用 spring。

## 7. Ribbon / Fabric / Wind Implementation

Jovicheer 的 ribbon 是核心，必須特別小心。最重要原則：

> 不要把 ribbon 做成 tube；不要把 motion 做成蛇；不要把材質做成塑膠。

### 方法比較

#### Curve-based ribbon mesh

推薦。用 curve 產生中心線，再沿著中心線建立 flat strip vertices。每個 segment 有左右兩個 vertex，寬度沿 normal / binormal 展開，再用 UV 貼 ribbon texture。

優點：

- 真正像絲帶：有寬度、正反面、扭轉。
- 可控制厚薄、邊緣、透明、顏色。
- 可做 wind deformation。

風險：

- Frenet frame / normal 方向處理不好會翻面、扭爛。
- segment 太少會像折線；太多會耗。
- 曲線過度 S 型會像蛇。

#### CatmullRomCurve3

適合當 ribbon centerline。Three.js 的 `CatmullRomCurve3` 可用少量 control points 產生柔順 curve。

建議：

- control points 不要過度蜿蜒。
- amplitude 小、frequency 低。
- 用固定 anchor + slack sag，讓它有重力感。

警告：若整條 curve 持續左右擺，使用者會讀成 snake。

#### TubeGeometry

不推薦作為 Jovicheer ribbon 主體。`TubeGeometry` 很容易變成 rope、hose、snake、plastic tube。除非要做繩子、桿子、光線軌跡，否則不要用來代表布條。

#### Flat strip mesh / custom ribbon geometry

主推薦。自己建立 `BufferGeometry`：

- positions：每個 path sample 左右兩點
- uvs：長度方向 0-1，寬度方向 0-1
- indices：每段兩個 triangles
- normals：可簡化或用 double-sided material
- material：thin fabric texture + alpha / roughness / subtle normal

這最符合 Jovicheer。

#### Vertex shader sine deformation

推薦作為輕量風感。用 vertex shader 對 ribbon segment 做低頻位移：

- 依 segment progress 加 phase
- 依 anchor distance 控制 amplitude
- 用 noise / sine 混合
- wind direction 是抽象場，不是物理

警告：

- sine 太規則會像海帶或蛇。
- amplitude 太大會像塑膠旗。
- 所有 ribbon 同步會像 screensaver。

#### Bone-chain ribbon animation

適合 hero ribbon 或 opening ritual。把 ribbon 分成骨架 chain，在 Blender 做 animation 或 browser 做 bone transforms。

優點：

- 可 art-direct，控制美感。
- 比 cloth sim 穩。

缺點：

- setup 比 shader deformation 多。
- 多條 ribbon 成本較高。

#### Spline-driven ribbon

推薦。把 ribbon 的 path / width / twist / phase 參數化，可由 data config 控制。這適合 AI agent 生成不同 ribbon variants，也適合 staged reveal。

#### Instanced ribbon segments

適合大量遠景小 ribbon。用 instancing 畫很多短 ribbon / wish strips，每個 instance 有 position、rotation、phase、color。遠景可以假；近景 hero ribbon 仍用 custom mesh。

#### GPU particles

適合 light motes、petals、dust、wind indicators，不適合直接代表 ribbon。GPU particles 可增加風場可見性，但不要把 ribbon 拆成粒子流，會失去祈願絲帶的物質感。

#### Wind field abstraction

強烈推薦。不要做真物理，做一個簡單可 art-direct 的 wind field：

```text
wind = baseDirection
     + lowFrequencyNoise(position, time)
     + anchorInfluence(anchor)
     + ritualBurst(clickTime)
```

這個 wind field 同時驅動：

- ribbon vertex displacement
- particles drift
- mist plane offset
- petals / paper strips
- light motes

這樣整個世界會像被同一陣風推動，而不是每個效果各動各的。

### 外觀警告

會像 snake 的條件：

- ribbon 中心線左右擺幅太大
- 頭尾都自由移動
- motion 是整條 forward wave
- 沒有固定 anchor / sag / fabric constraint

會像 rope 的條件：

- 用 TubeGeometry
- ribbon 太窄
- 沒有面寬和正反面
- 材質沒有布的 roughness / translucency

會像 plastic tube 的條件：

- specular 太高
- clearcoat / metallic 誤用
- bloom 太重
- 形狀圓而不是扁

會像 cheap cloth 的條件：

- texture resolution 低
- alpha edge 鋸齒
- bend 沒有重力
- 動畫頻率太高
- ribbon 在風中像旗子硬抖

會像 flat sticker 的條件：

- 完全沒有 twist
- 沒有光照或陰影變化
- 永遠面向 camera billboard
- 和背景沒有 occlusion / depth relationship

## 8. Suggested First Prototype

### Prototype 名稱

**Jovicheer World Stage P0 — One Anchor, One Ribbon, One Wish**

### 必須包含

- fixed camera
- no character
- one scene only
- high-quality background / layers strategy
- ribbon strategy
- wind / particles strategy
- anchor interaction
- staged depth movement or click-to-focus
- HTML UI overlay

### 場景設定

一個祈願庭院 / 霧光平台 / 遠方願望牆，只做一個 camera angle。畫面中有：

- 遠景：soft luminous background plate
- 中景：一個 anchor object，例如祈願木架、石環、光柱、懸掛物
- 前景：一條主要 ribbon 從 anchor 延伸出來
- 空氣：mist + small light motes
- UI：右下或底部 HTML wish panel

### Background / layers strategy

P0 建議：

- 先用 AI image 或 matte painting 做 16:9 / 21:9 background plate。
- 用手工或 AI 分層成 background / midground / foreground transparent PNG/WebP。
- 產一張 depth map，先不一定上 shader displacement，但保留。
- R3F 中用 3-5 個 planes 排出 layered diorama。
- CSS 再疊 subtle noise / vignette / light wash。

### Ribbon strategy

P0 只做一條 hero ribbon：

- custom flat ribbon geometry
- CatmullRom centerline
- 16-32 segments
- double-sided fabric material
- subtle alpha edge / texture
- vertex shader 或 CPU update 做低頻 wind
- one end visually anchored，另一端可輕微浮動

不做：

- real cloth simulation
- multiple interacting ribbons
- collision
- knot tying

### Wind / particles strategy

P0 做三種很克制的 motion：

- ribbon low-frequency sway
- light motes slow drift
- mist planes subtle offset / opacity pulse

所有 motion 用同一個 `WindField` config：

```text
direction: left-up
baseSpeed: very slow
gustInterval: rare
ritualBurst: triggered on anchor click
```

### Anchor interaction

Anchor 是一個 visible object + invisible hit target。互動：

1. hover / focus：anchor 有一點 light response。
2. click：camera 0.8-1.2 秒微推近或橫向 pan。
3. ribbon 稍微拉開 / 發光。
4. hidden wish / anchor inscription 出現。
5. HTML overlay 顯示 wish input 或 wish reveal。

### Staged depth movement / click-to-focus

不要自由 orbit。只做 director-authored states：

- `overview`
- `anchorFocus`
- `wishReveal`

Camera 轉場：

- position 只移動 3-8%
- target / lookAt 微調
- FOV 可微收
- foreground layer parallax 稍微加強

### HTML UI overlay

HTML / React overlay 負責：

- wish input
- submit / ritual CTA
- hidden wish text
- accessibility labels
- loading / fallback

不要把表單放進 3D canvas。3D canvas 是 atmosphere + interaction stage；產品 flow 是 HTML。

### What to fake

- 大場景 depth：用 layers / depth map fake。
- 霧：用 transparent planes / CSS / texture fake。
- 光影：用 background plate / baked shadows fake。
- cloth physics：用 spline + shader fake。
- camera depth：用 small dolly + parallax fake。

### What to build for real

- R3F scene component
- custom ribbon mesh
- anchor click hit testing
- camera staged movement
- wind field abstraction
- particle / mist layer
- HTML overlay integration
- mobile fallback
- screenshot/performance checks

### What to defer

- 多場景系統
- scene editor
- real cloth sim
- Unreal/Unity runtime
- WebGPU dependency
- complex postprocessing
- user-generated ribbon persistence
- shareable wish world map

## 9. Per-scene Cost Estimate

### One simple scene

人工時間：3-7 天。

AI agent 可協助比例：70-85%。

素材準備時間：0.5-2 天。

調光 / 調材質 / 調動畫時間：1-2 天。

內容：

- 1 張 AI / matte background
- 2-4 層 parallax planes
- 1 條 ribbon
- 1 個 clickable anchor
- 基本 particles / mist
- 1 個 HTML wish panel

主要風險：

- 背景太 AI / 太 generic。
- ribbon 動態 cheap。
- anchor 沒有祈願感。
- mobile 上視覺層糊或卡。

### One polished scene

人工時間：2-4 週。

AI agent 可協助比例：55-75%。

素材準備時間：4-8 天。

調光 / 調材質 / 調動畫時間：5-10 天。

內容：

- moodboard + visual target
- AI image / Blender layered render
- foreground / midground / depth map
- 1-3 條 ribbons
- 2-4 anchors / wish objects
- tuned camera states
- refined mist / light motes
- responsive desktop/mobile composition
- performance budget

主要風險：

- 背景和 3D props 光影不融合。
- asset pipeline 反覆重切 layer。
- 動畫過多，失去安靜祈願感。
- 每次改背景都要重對齊 planes / hit targets。

### One highly cinematic scene

人工時間：5-10 週。

AI agent 可協助比例：40-65%。

素材準備時間：2-4 週。

調光 / 調材質 / 調動畫時間：2-4 週。

內容：

- Blender / Unreal lookdev
- 多個 render passes
- hero ribbon art-directed animation
- opening ritual
- sound / timing / cinematic camera
- high-quality fallback video
- multiple device-specific compositions
- asset compression / QA

主要風險：

- 視覺 ambition 超過 production capacity。
- render iteration 太慢。
- runtime / authoring 工具之間來回成本高。
- web 上看起來不如 Blender / Unreal render。
- 團隊把精力花在「做更像遊戲」而不是「做更有儀式感」。

## 10. Tooling Recommendation

### Recommended stack

Runtime：**React + R3F + Drei + Three.js**

Animation：

- P0：GSAP 或 react-spring
- P1：Theatre.js for camera / light / prop sequence
- HTML overlay：Framer Motion

Asset authoring：

- P0：AI image + manual layer separation + light Blender pass
- P1：Blender 作主要 authoring / render layer / GLB export
- P2：Unreal only for high-end cinematic lookdev / hero render if needed

Asset format：

- background / layer：WebP / AVIF
- transparent foreground：PNG or WebP with alpha
- short ambience：MP4 / WebM video loop only when helpful
- 3D props：GLB / glTF
- depth：grayscale PNG/WebP
- metadata：JSON scene manifest

UI：**HTML / React overlay**

Deployment：normal web stack，如 Vite / Next / Astro + static hosting / Vercel / Firebase Hosting。

### 為什麼

這套工具鏈把責任切得乾淨：

- React 管產品 UI。
- R3F 管 3D stage。
- Drei 加速常見 3D helpers。
- GSAP / Theatre.js 管導演式時間軸。
- Blender / AI image 管美術品質。
- GLB / WebP / PNG / video 都是普通 web assets，部署和 cache 都簡單。

最重要的是：這套不會把 Jovicheer 綁死在 Unity/Unreal WebGL，也不會讓純 Three.js 從零承擔所有美術。

## 11. Reference Examples

### Web cinematic 3D sites

#### Bruno Simon portfolio

URL: https://bruno-simon.com/

值得參考：

- WebGL 可以做到非常完整、可記憶的世界感。
- 物件、場景、互動、音效、UI 能形成品牌 personality。
- Three.js 可以支撐高品質 creative site。

和 Jovicheer 的關係：

- 證明 browser 3D 能有作品級質感。
- 也證明 strong art direction 比技術本身更重要。

不應照抄：

- 它是可駕駛 / 可探索 / game-like portfolio，和 Jovicheer 的固定鏡頭路線相反。
- Jovicheer 不需要車、不需要物理、不需要開放地圖。

#### Awwwards Three.js collections

URL: https://www.awwwards.com/awwwards/collections/three-js/

值得參考：

- 看 creative web 的視覺語言、camera、transition、product-quality polish。
- 能建立「premium web 3D」的品味樣本。

和 Jovicheer 的關係：

- 可作 visual benchmark。

不應照抄：

- 很多 Awwwards site 追求 novelty / shock / scroll spectacle，Jovicheer 需要安靜、祈願、儀式感。

### Three.js / R3F diorama demos

#### R3F official examples

URL: https://r3f.docs.pmnd.rs/getting-started/examples

值得參考：

- R3F scene component pattern。
- pointer interaction。
- small 3D experiences how to structure。

和 Jovicheer 的關係：

- 可作 P0 scaffold 參考。

不應照抄：

- demo visual style。Jovicheer 需要自有 art direction。

#### Baked lighting in R3F

URL: https://tchayen.github.io/posts/baked-lighting-in-r3f

值得參考：

- 用 Blender baked lighting 讓 web scene 接近 offline render quality。
- 對 fixed-camera diorama 特別適合。

和 Jovicheer 的關係：

- 如果要做 GLB anchor / small props，baked light 是避免 cheap realtime light 的重要方法。

不應照抄：

- 不要為了 baked scene 變成整個 3D room exploration。

### Interactive matte painting / parallax examples

#### Codrops WebGPU scanning effect with depth maps

URL: https://tympanus.net/codrops/2025/03/31/webgpu-scanning-effect-with-depth-maps/

值得參考：

- depth map 能讓 2D image 產生空間感。
- 2.5D + shader 可以比全 3D 更有效率。

和 Jovicheer 的關係：

- 適合 background plate + depth map + subtle parallax。

不應照抄：

- scanning light / sci-fi shader 不是 Jovicheer 氣質。
- WebGPU 不應作為第一版 production dependency。

#### Codrops scroll-reactive 3D gallery

URL: https://tympanus.net/codrops/2026/03/09/building-a-scroll-reactive-3d-gallery-with-three-js-velocity-and-mood-based-backgrounds/

值得參考：

- depth-layered images、palette-driven backgrounds、motion based on scroll velocity。

和 Jovicheer 的關係：

- 可參考 layered images + subtle motion。

不應照抄：

- Jovicheer 第一版不需要 gallery 或強 scroll mechanic。

### Ribbon / cloth / wind examples

#### Codrops Animated 3D Ribbons with Three.js

URL: https://tympanus.net/codrops/2021/11/29/animated-3d-ribbons-with-three-js/

值得參考：

- ribbon 可以用 geometry trick 做出漂亮 3D 動勢。
- 可以研究 mesh / shader / motion 的方向。

和 Jovicheer 的關係：

- 直接關係到祈願絲帶。

不應照抄：

- 範例偏抽象科技感；Jovicheer ribbon 要更柔、更薄、更有布感和祈願感。

#### Three.js cloth examples / forum discussions

URL: https://threejs.org/examples/  
URL: https://discourse.threejs.org/t/cloth-flag-simulation/25426

值得參考：

- 真 cloth simulation 的成本與複雜度。
- 可看 flag / cloth motion 的基本概念。

和 Jovicheer 的關係：

- 幫助理解為什麼 P0 不應把 real cloth sim 作 mandatory requirement。

不應照抄：

- 官方 cloth/flag demo 不是 production-ready ribbon art direction；也不適合直接塞進 wish world。

### Product-quality WebGL examples / references

#### Three.js Journey / Bruno Simon ecosystem

URL: https://threejs-journey.com/

值得參考：

- Three.js learning / production vocabulary。
- creative developer 如何系統性處理 scene、camera、materials、performance。

和 Jovicheer 的關係：

- 可作 Jones / agent collaboration 的 3D language foundation。

不應照抄：

- 不要把學習路線誤解成「要先學完整 3D game」。

#### Theatre.js

URL: https://www.theatrejs.com/

值得參考：

- 專門為 browser animation / cinematic sequence 提供 timeline UI。
- 可 keyframe camera、lights、material colors、object transforms。

和 Jovicheer 的關係：

- 很適合 staged camera movement / ritual sequence。

不應照抄：

- 不要一開始把所有微動都丟進 timeline；wind/ribbon continuous motion 仍適合 procedural。

## 12. Risk / Pitfall List

### Three.js 場景像 cheap demo

原因：

- runtime primitive 太多
- 沒有高品質背景 / light direction
- postprocessing 取代 art direction
- demo helper 直接套用

規避：

- 先定 background plate / mood。
- 限制 3D 物件數量。
- 每個場景先做 still frame 美術驗收，再加互動。
- 建立 visual QA：截圖看是否像 product shot，而不是 coding demo。

### Ribbon 像蛇或塑膠管

原因：

- 用 TubeGeometry。
- 中心線擺幅太大。
- motion 週期太規律。
- 材質太亮太厚。

規避：

- 用 custom flat strip mesh。
- 一端固定，保留重力 sag。
- 低頻、低幅度、非同步 wind。
- rough fabric material，避免 high specular。

### 背景和 3D 物件光影不融合

原因：

- 背景光源方向與 3D light 不一致。
- 3D props 沒有 color grading。
- 沒有 shadow / occlusion / haze。

規避：

- 背景 plate 產出時記錄 light direction。
- R3F 使用 matching directional / hemisphere / environment。
- 小 props 做 baked lighting 或 shadow plane。
- 用 fog / color grading 把元素黏在一起。

### mobile performance 差

原因：

- DPR 無限制。
- texture 太大。
- particles 太多。
- shadows / postprocessing 太重。
- 每幀 React state。

規避：

- DPR clamp，例如 max 1.5 或 mobile 1.0。
- texture budget：background WebP/AVIF，GLB texture 壓縮。
- mobile 減少 particles、關閉 shadow、簡化 fog。
- R3F performance rules。
- 提供 static/video fallback。

### asset pipeline 太重

原因：

- 每次都回 Blender / Unreal 全量重渲染。
- layers 沒命名規則。
- 沒有 scene manifest。

規避：

- 建立 `scene.json`：camera、layers、anchors、asset paths、depth scale。
- P0 只做一個場景。
- 先用 flat image layers 驗證，再升級 authoring。
- render layer naming convention。

### Unity/Unreal WebGL runtime 太笨重

原因：

- 把 cinematic tool 當網站 runtime。

規避：

- Unity/Unreal 只做 authoring / render / reference。
- Web runtime 仍是 React + R3F。
- 除非未來產品變成真正 3D app，否則不走 engine WebGL build。

### AI agent 能寫 code 但不能保證美術審美

原因：

- agent 擅長把 spec 變成 code，不擅長 final taste。

規避：

- 每輪交付 screenshot / short capture。
- 明確 visual adjectives + negative examples。
- Jones / GPT 做 art direction review。
- 建立「cheap demo red flags」檢查清單。

### 場景每次都要重渲染導致迭代慢

原因：

- 過早走完整 C pipeline。

規避：

- P0 用 AI image + layers。
- P1 才引入 Blender layers。
- Unreal 僅在 hero lookdev 需要時使用。

### 互動太像遊戲但又不好玩

原因：

- 加了自由 camera、drag、探索，但沒有 game loop。

規避：

- 堅持 fixed camera / staged camera。
- 互動是 ritual，不是 navigation。
- 點擊後只 reveal / focus / submit wish，不做漫遊。

### UI 被 3D 搶走

原因：

- canvas 全屏但沒有產品 flow hierarchy。

規避：

- HTML overlay 是一等公民。
- 3D stage 服務 wish flow，不取代 wish flow。
- mobile 優先檢查文字可讀、CTA 可點。

## 13. Suggested Production Plan

### Phase 0 — Visual Target Spike

時間：2-4 天。

產出：

- 6-12 張 Jovicheer world stage mood frames。
- 選 1 張作 P0 背景 plate。
- 定義 color palette、light direction、ribbon material references。

### Phase 1 — P0 Browser Prototype

時間：1-2 週。

產出：

- React + R3F single scene。
- layered background planes。
- one custom ribbon mesh。
- one anchor click interaction。
- camera `overview -> anchorFocus`。
- HTML wish overlay。
- desktop/mobile screenshots。

### Phase 2 — Polished Scene

時間：2-4 週。

產出：

- Blender-assisted layers / depth map。
- refined ribbon shader。
- tuned mist / particles。
- 2-3 anchor states。
- fallback mode。
- production-ready asset compression。

### Phase 3 — Repeatable Scene Pipeline

時間：2-3 週。

產出：

- `scene.json` schema。
- asset folder convention。
- Blender export script。
- R3F `WorldStageScene` loader。
- QA checklist。

## 14. Sources

Core web runtime:

- Three.js WebGPURenderer docs: https://threejs.org/docs/pages/WebGPURenderer.html
- R3F examples: https://r3f.docs.pmnd.rs/getting-started/examples
- R3F performance pitfalls: https://r3f.docs.pmnd.rs/advanced/pitfalls
- R3F scaling performance: https://r3f.docs.pmnd.rs/advanced/scaling-performance
- Drei showcase: https://storybook.js.org/showcase/pmndrs-drei/
- gltfjsx: https://github.com/pmndrs/gltfjsx
- GSAP ScrollTrigger docs: https://gsap.com/docs/v3/Plugins/ScrollTrigger/
- Theatre.js R3F docs: https://www.theatrejs.com/docs/0.5/getting-started/with-react-three-fiber
- Theatre.js Three.js docs: https://www.theatrejs.com/docs/0.5/getting-started/with-three-js
- PixiJS intro: https://pixijs.com/8.x/guides/getting-started/intro
- Lottie web: https://github.com/airbnb/lottie-web
- Rive runtimes: https://rive.app/runtimes
- Spline: https://spline.design/

3D / cinematic authoring:

- Blender glTF 2.0 manual: https://docs.blender.org/manual/en/latest/addons/import_export/scene_gltf2.html
- Blender Python API render operators: https://docs.blender.org/api/current/bpy.ops.render.html
- Unity Cinemachine docs: https://docs.unity3d.com/Packages/com.unity.cinemachine@latest/
- Unity Cinemachine + Timeline docs: https://docs.unity3d.com/Packages/com.unity.cinemachine%402.2/manual/CinemachineTimeline.html
- Unreal Movie Render Pipeline: https://dev.epicgames.com/documentation/unreal-engine/movie-render-pipeline-in-unreal-engine
- Unreal high-quality Movie Render Queue guide: https://dev.epicgames.com/documentation/unreal-engine/rendering-high-quality-frames-with-movie-render-queue-in-unreal-engine

Assets:

- Poly Haven: https://polyhaven.com/
- Poly Haven license: https://polyhaven.com/license
- Sketchfab free models: https://sketchfab.com/features/free-3d-models
- BlenderKit / Blendkit: https://www.blenderkit.com/
- Quixel / Megascans transition: https://quixel.com/megascans/free
- Mixamo: https://www.mixamo.com/
- Meshy: https://www.meshy.ai/
- Tripo: https://www.tripo3d.ai/

Examples / references:

- Bruno Simon portfolio: https://bruno-simon.com/
- Three.js Journey: https://threejs-journey.com/
- Awwwards Three.js collection: https://www.awwwards.com/awwwards/collections/three-js/
- Codrops React Three Fiber demos: https://tympanus.net/codrops/hub/tag/react-three-fiber/
- Codrops Animated 3D Ribbons with Three.js: https://tympanus.net/codrops/2021/11/29/animated-3d-ribbons-with-three-js/
- Codrops WebGPU depth-map scanning effect: https://tympanus.net/codrops/2025/03/31/webgpu-scanning-effect-with-depth-maps/
- Codrops scroll-reactive 3D gallery: https://tympanus.net/codrops/2026/03/09/building-a-scroll-reactive-3d-gallery-with-three-js-velocity-and-mood-based-backgrounds/
- Baked lighting in R3F: https://tchayen.github.io/posts/baked-lighting-in-r3f
- Three.js cloth / flag discussion: https://discourse.threejs.org/t/cloth-flag-simulation/25426

## 15. Final Answer

Recommended route:

> **Route D — Hybrid Cinematic Web Diorama**，並採取 staged combination：D 作 production pipeline，B1 作 browser runtime，C 作 authoring / render support。

Recommended first prototype:

> **One Anchor, One Ribbon, One Wish**：單一固定鏡頭場景、一張高品質 background plate、3-5 層 parallax planes、一條 custom flat ribbon、一個 clickable anchor、一次 staged dolly-in / pan、一個 HTML wish overlay。

Recommended runtime:

> **React + R3F + Drei + Three.js**。動畫 P0 用 GSAP / react-spring；P1 若需要導演式 keyframe 調整，加入 Theatre.js。HTML UI 用 React / Framer Motion overlay。

Recommended asset pipeline:

> P0 用 AI image / matte painting + layer separation；P1 用 Blender 做 render layers、depth map、GLB props、baked lighting；Unreal 只在需要高階 cinematic lookdev / hero render 時使用，不作 web runtime。

What to build now:

- R3F fixed-camera stage
- layered background planes
- custom flat ribbon mesh
- shared wind field
- light motes / mist
- anchor hit target
- staged camera focus
- HTML wish overlay
- mobile fallback and screenshots

What to fake now:

- 大場景 depth
- 遠景光影
- 霧和空氣深度
- cloth physics
- cinematic camera depth

What to defer:

- real cloth simulation
- multi-scene world system
- WebGPU dependency
- Unity/Unreal WebGL runtime
- full 3D scene editor
- multiplayer / avatars / character controller
- complex physics

What not to do:

- 不要做 free-camera 3D exploration。
- 不要做 walking character。
- 不要把整個網站包成 Unity / Unreal WebGL game。
- 不要把 ribbon 做成 TubeGeometry。
- 不要用 bloom / particles 掩蓋美術不足。
- 不要讓互動變成「像遊戲但不好玩」。


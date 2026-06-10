# Agentic Coding 時代的 Three.js 入門報告

Date: 2026-06-09  
Question: 如果 Jones 想為 Jovicheer、ObserverJ、Raijax 等未來網站補一點 Three.js / 3D web 基礎，在幾乎不需要自己逐行手打 code、可以大量使用 agentic coding 的現在，最適合怎麼學？

## Executive Summary

Jones 不需要走「從零手刻 WebGL / 背完整 Three.js API」那條路。比較適合的定位是：

> 成為一個懂 3D web 基本語言、能判斷 agent 產物好壞、能把視覺意圖轉成可執行 spec 的 creative technical director。

也就是說，學習目標不是「每一行 Three.js 都自己寫」，而是要能看懂：

- 這個 scene 裡有哪些物件、相機、光、材質、動畫與互動
- agent 產出的 code 是否有 resize、cleanup、mobile、performance、fallback
- 3D 模型為什麼匯入後變黑、變灰、太大、太慢、位置不對
- 哪些效果適合 Three.js，哪些其實用 CSS / video / image sequence 就好
- 什麼時候用 raw Three.js，什麼時候用 React Three Fiber / Drei

我的建議是：先用 raw Three.js 建立 mental model，再用 React Three Fiber 做真正會進站的 prototype。學習成果應該是一組「可複用的網站 3D pattern」，而不是一堆散掉的教學練習。

## 為什麼仍然值得學 Three.js 基礎

Three.js 官方 manual 把它定位成讓 3D content 更容易上網頁的 library；它幫你包掉 WebGL 低階細節，例如 scene、lights、shadows、materials、textures、3D math 等原本要自己寫的東西。官方 fundamentals 也明確說，典型 app 由 `renderer`、`scene`、`camera`、scene graph、mesh、geometry、material、texture、light 組成。

在 agentic coding 時代，這些概念更重要，不是更不重要。因為 agent 可以很快吐出一個 rotating cube，但如果你不懂這些詞，你就無法判斷它為什麼：

- 在手機上爆 GPU
- scroll 時掉幀
- 相機角度看不到主物件
- lighting 讓模型失去質感
- SSR / hydration 在 Next 或 Astro 裡出問題
- 只在 agent 預覽環境能跑，放進真網站就壞

## 你應該具備的知識地圖

### 1. Web / JavaScript / React 基礎

不需要刷演算法，但要看得懂 agent 寫出的基本結構：

- module import / export
- async loading
- event listener
- React component / props / state / refs
- effect cleanup
- TypeScript 型別錯誤大概在吵什麼
- Vite / Next / Astro 的 client-only component 概念

Three.js Journey 的 prerequisite 也明講：它是 beginner-friendly，但仍需要 JavaScript basics，例如 variables、array、objects、loops、functions、events。這點在 agentic coding 下仍成立，只是你不必靠手打語法練熟，而是靠 review、修改、追錯來建立感覺。

### 2. 3D 場景的基本語言

最低限度要懂：

- `Scene`：3D 世界的 root
- `Camera`：觀察世界的視角，尤其 `PerspectiveCamera`
- `Renderer`：把 scene + camera 畫到 canvas
- `Mesh`：geometry + material + transform
- `Geometry`：形狀或 vertex data
- `Material`：表面怎麼被畫出來
- `Texture`：貼圖、normal map、roughness map 等
- `Light`：ambient、hemisphere、directional、point、spot
- `Object3D / Group`：parent-child hierarchy
- transform：position / rotation / scale
- animation loop：`requestAnimationFrame` 或 R3F 的 `useFrame`

這些不是為了考試，而是為了跟 agent 溝通。例如你可以說：「把 camera 拉遠一點，主體維持在 z=0；不要改 mesh size，改 camera framing。」

### 3. 燈光、材質、色彩與質感

Three.js 最常見的新手挫折不是「跑不動」，而是「模型怎麼這麼醜」。你要懂：

- `MeshBasicMaterial` 不吃光，適合 flat/iconic 效果
- `MeshStandardMaterial` / `MeshPhysicalMaterial` 才接近 PBR 工作流
- ambient / hemisphere 只能補暗部，不會給明確方向感
- directional light 類似太陽，適合大部分 hero scene
- shadow 很貴，尤其 point light shadow 可能要從六個方向重畫 scene
- environment map / HDRI 往往比亂加一堆燈更重要
- color management / tone mapping 會影響模型看起來是否「灰、髒、失真」

對 Jovicheer / Jyn Null / ObserverJ 這種品牌視覺，材質與光比 geometry 更關鍵。先學會控制 mood，比先學 shader 更實用。

### 4. glTF / GLB 與 Blender 基礎

未來真的做網站，八成會用現成或自製模型。Three.js manual 說 glTF 是為 transmission / display 設計的格式，相比 OBJ / DAE 更適合網頁顯示，資料更小，也更接近 ready-to-render。

要懂的不是完整 Blender，而是 3D asset pipeline：

- `.glb` / `.gltf` 差異
- 模型大小、原點、pivot、座標軸
- UV unwrap / baked texture 的概念
- Draco / Meshopt compression
- KTX2 / texture compression
- 低面數與 LOD
- 匯出時保留材質、動畫、camera 的限制
- 為什麼 Blender 裡漂亮，進 browser 後不一定漂亮

社群討論裡也常提醒：AI 足以幫你把模型放到網站上，但如果要「看起來跟 Blender 一樣」，燈光、UV、baking、材質與 texture pipeline 仍需要人懂一點。

### 5. React Three Fiber / Drei

如果未來網站多半是 React / Next / Astro + React islands，那我建議不要只停在 raw Three.js。React Three Fiber 官方定義很直接：它是 Three.js 的 React renderer，讓 scene 能以 reusable components 的方式宣告，並參與 React ecosystem。

你應該知道：

- `<Canvas>` 是 R3F 的入口
- `<mesh>` 對應 `new THREE.Mesh()`
- `useFrame` 是 animation loop
- pointer events / hover / click 可以走 React-like pattern
- Drei 提供大量常用 helper，例如 `OrbitControls`、`Environment`、`Html`、`Text`、`ScrollControls`、`PresentationControls`
- `gltfjsx` 可以把 GLTF 轉成可複用的 R3F JSX component

R3F 官方也提醒版本配對：`@react-three/fiber@8` 對 React 18，`@react-three/fiber@9` 對 React 19。這在 agentic coding 時代很重要，因為 agent 常會混用舊教學版本。

### 6. 互動、scroll 與網站整合

Jovicheer / ObserverJ / Raijax 的 3D 用途大概不是純遊戲，而是品牌 hero、互動展示、思想圖譜、agent space、產品/作品宇宙。因此要懂：

- Orbit / drag / pointer hover
- raycaster / pointer events
- HTML overlay 與 3D 物件對齊
- scroll-driven animation
- route transition / page transition
- responsive canvas
- mobile fallback
- `prefers-reduced-motion`
- accessibility：3D 只是 enhancement，不應讓主要內容消失

Codrops / Awwwards 類案例可以拿來看視覺語言，但不要一開始就追求整站沉浸式 WebGL。先學「一個段落裡有一個有用的 3D moment」比較實際。

### 7. 效能、debug 與生產化

這是 agent 最容易亂來、但你最需要有判斷力的地方。

最低限度要懂：

- draw calls：很多 mesh / material 會慢
- geometry / material / texture 要 reuse
- instancing / merging geometry
- texture size 不要亂上 4K / 8K
- shadow map 越大越慢
- mobile devicePixelRatio 不能無腦拉滿
- animation loop 不要每幀 set React state
- unmount 時要 dispose GPU resource
- Chrome devtools / performance / FPS / memory 基本查看

Three.js manual 的 responsive 文章特別提醒，高 DPI 手機若照 devicePixelRatio 全解析度渲染，可能造成大量 GPU 負擔；R3F docs 也有 performance pitfalls，提醒不要在 `useFrame` 或 fast events 裡用 React state 追每幀變化，而應該直接 mutate 或用 delta。

## Raw Three.js vs React Three Fiber：我的建議

### 學習順序

先 raw Three.js，後 R3F。

原因：raw Three.js 能讓你直接看懂 scene / camera / renderer / mesh / material / light 的本體。R3F 很適合做網站，但如果一開始只學 R3F，很容易以為 `<mesh>` 是 React 魔法，而不知道底下其實是 Three.js object。

### 實戰順序

真正放進未來網站時，優先 R3F + Drei。

原因：Jovicheer / ObserverJ / Raijax 更像 React/Astro/Next 網站專案，不是獨立 3D engine。R3F 的 component model、hooks、Drei helpers、gltfjsx workflow，比 raw Three 更適合和現代前端整合。

### 例外

如果只是做一個很小、完全獨立、不需要 React state 的 canvas background，raw Three.js 仍然可以更簡潔。  
如果要做高壓效能、遊戲、複雜 shader pipeline，可能要更靠近 raw Three.js / WebGL mental model。

## 最適合 Jones 的 6 週入門安排

這不是學生式課表，而是「用 agentic coding 反向學會判斷」的安排。每天不需要很久，重點是每週產出一個可看的小物。

### Week 1：看懂一個 3D scene

目標：能說出 scene 裡每個東西在幹嘛。

任務：

- 跑一個 raw Three.js Vite template
- 做 rotating cube
- 改 camera position / fov / near / far
- 換 `MeshBasicMaterial`、`MeshPhongMaterial`、`MeshStandardMaterial`
- 加 directional light / hemisphere light
- 加 OrbitControls
- 做 resize helper

Agentic prompt:

```text
請用 Vite + TypeScript + Three.js 做一個最小 rotating cube demo。
限制：不要加 framework；請把 scene/camera/renderer/mesh/material/light/resize/render loop 分成清楚區塊，並在 code comment 解釋每個區塊。
完成後列出我應該理解的 10 個 Three.js 名詞。
```

### Week 2：模型與材質

目標：知道模型匯入後為什麼會醜、黑、太大、太慢。

任務：

- 用 `GLTFLoader` 載入一個 `.glb`
- 調 scale / position / rotation
- 加 environment map
- 比較不同 light setup
- 看模型 triangle count / texture size
- 試一次 glTF compression 概念

Agentic prompt:

```text
請做一個 GLB model viewer：OrbitControls、environment light、loading state、mobile responsive。
請另外寫一段 README，解釋如果模型匯入後變黑、變灰、太大、太慢，應該依序檢查哪些原因。
```

### Week 3：React Three Fiber / Drei

目標：能把 3D scene 變成網站 component。

任務：

- 建 R3F `<Canvas>`
- 用 `<mesh>` 重做 Week 1
- 使用 `useFrame`
- 加 Drei `OrbitControls` / `Environment` / `Html`
- 用 `gltfjsx` 把模型轉成 component
- 做 hover / click interaction

Agentic prompt:

```text
請把 Week 2 的 GLB viewer 改成 React Three Fiber + Drei。
限制：使用 @react-three/fiber 和 @react-three/drei；模型 component 用 gltfjsx 風格；不要在 useFrame 裡 setState；請標出哪些地方是 Three.js 原生概念，哪些是 R3F abstraction。
```

### Week 4：網站效果 pattern

目標：產出未來網站會真的用到的 3D moment。

任務三選一：

- Jovicheer：一個柔和發光、可隨 scroll 慢慢轉動的 brand object
- ObserverJ：一個思想星圖 / node constellation，可 hover 顯示概念
- Raijax：一個互動小場景，滑鼠移動時 agent/object 有反應

Agentic prompt:

```text
請做一個適合放在網站 hero 的 Three.js/R3F 3D moment。
需求：不是全站遊戲；必須有 mobile fallback；支援 prefers-reduced-motion；主要文字不能被 canvas 擋住；請提供 performance checklist。
```

### Week 5：效能與生產檢查

目標：能審查 agent 產物。

任務：

- 加 FPS / stats
- 檢查 draw calls
- 限制 pixel ratio
- 測 desktop / mobile viewport
- 移除不必要 shadows
- 測 unmount / route switch 是否有 cleanup
- 用 Playwright 截圖驗證 canvas 非空白

Agentic prompt:

```text
請審查這個 R3F component 的生產風險：mobile GPU、draw calls、texture size、shadow cost、cleanup、SSR/client boundary、accessibility fallback。
請用 checklist 回覆，並提出最小修改 diff。
```

### Week 6：整理成自己的 starter kit

目標：不是只學一次，而是建立 Jones 自己的 3D website starter。

內容：

- `ThreeHeroCanvas`
- `ModelViewer`
- `ConstellationGraph`
- `ResponsiveCanvas`
- `useReducedMotionFallback`
- `performance-checklist.md`
- `agent-prompts.md`

這個 starter kit 之後可以餵給 Codex / Claude / OpenClaw，讓 agent 每次做 3D feature 都沿用你的標準。

## 最值得先做的四個練習

### 練習 1：Jovicheer Brand Object

一個簡單 3D ribbon / orb / soft geometry，不要過度炫技。重點是光、材質、動態節奏。

學到：

- material
- lighting
- camera framing
- responsive hero
- motion restraint

### 練習 2：ObserverJ Thought Constellation

把幾個概念 node 放在 3D 空間中，hover 顯示文字，click 可以切換 focus。

學到：

- points / spheres
- labels / Html overlay
- raycast / pointer events
- camera movement
- data-driven scene

### 練習 3：Raijax Agent Playground

一個小型互動場景：agent / avatar / object 會跟 cursor 或 user input 互動。

學到：

- input mapping
- animation loop
- state boundary
- simple physics 或 easing
- game-like interaction

### 練習 4：Jyn Null Mood Scene

把一句短文做成抽象 3D 場景，例如「happiness is more like a muscle」的柔和有機形狀。

學到：

- shape / mood translation
- postprocessing
- screenshot/export
- social visual pipeline

## Agentic Coding 工作流

### 1. 先寫視覺 brief，不要先叫 agent 寫 code

好的 brief 包含：

- 用途：hero / background / model viewer / graph / game toy
- mood：calm / cyber-noir / playful / editorial
- 互動：hover / drag / scroll / no interaction
- performance budget：mobile 必須順、低端機可 fallback
- assets：是否用 GLB、texture、HDRI
- 禁止事項：不要全頁擋文字、不要無限加 shadow、不要把 pixel ratio 拉滿

### 2. 要求 agent 分階段交付

不要一次叫它「做一個很酷的 3D 網站」。改成：

1. 先做 static scene
2. 再加 camera / controls
3. 再加 material / light
4. 再加 interaction
5. 再加 responsive / fallback
6. 最後做 performance pass

### 3. 每次都要求它解釋 scene graph

讓 agent 回答：

- scene 裡有幾個 mesh？
- 每個 mesh 用什麼 geometry / material？
- 有哪些 light？哪些 cast shadow？
- animation loop 每幀做什麼？
- resize 怎麼處理？
- unmount 時釋放哪些 resource？

如果 agent 答不出來，代表它自己也可能只是拼貼。

### 4. 用檢查表審查，不靠感覺

每個 3D component 進 repo 前，至少檢查：

- desktop / mobile screenshot
- canvas 非空白
- 沒有遮住主要文字
- mobile FPS 可接受
- reduced motion fallback
- no hydration error
- no console error
- assets 有合理大小
- route 離開後沒有 memory leak

### 5. 把成功 pattern 存成自己的 prompt library

每次做出可用的 3D pattern，就保存：

- 原始 brief
- agent prompt
- final component
- 截圖
- performance notes
- 哪些坑被修過

長期來看，這會比「看完課程」更值錢。

## 推薦資源

### 必讀 / 官方

- [Three.js Manual](https://threejs.org/manual/)  
  先讀 fundamentals、responsive、lights、shadows、textures、load glTF、cleanup、optimization。

- [Three.js Docs](https://threejs.org/docs/)  
  查 API 用，不建議從頭硬讀。

- [Three.js Examples](https://threejs.org/examples/)  
  當靈感庫與 agent reference。看到喜歡的效果，叫 agent 拆解它用到哪些 module。

- [React Three Fiber Docs](https://r3f.docs.pmnd.rs/getting-started/introduction)  
  未來如果網站走 React / Next / Astro islands，這是主戰場。

- [Drei](https://github.com/pmndrs/drei)  
  R3F helper collection。很多常用網站效果不用自己從零寫。

- [gltfjsx](https://github.com/pmndrs/gltfjsx) / [gltf.pmnd.rs](https://gltf.pmnd.rs/)  
  把 GLTF 轉成 R3F component，對網站整合非常實用。

### 系統課程

- [Three.js Journey](https://threejs-journey.com/)  
  目前仍是社群最常被推薦的完整課。官方頁面標示 93 小時、從 beginner 到 advanced；它也明確列出需要 JavaScript basics，但不要求很強的數學。適合 Jones 不是因為要從頭看完，而是可以當「結構化 reference library」。

- [Discover three.js](https://discoverthreejs.com/)  
  免費、偏書籍式的完整入門。適合補 mental model。

### 靈感與案例

- [Codrops Three.js tutorials](https://tympanus.net/codrops/tag/three-js/)  
  適合看 scroll、gallery、shader、portfolio 類網站效果。不要一開始照抄整套，先拆 pattern。

- [Awwwards Three.js examples](https://www.awwwards.com/websites/three-js/)  
  適合建立審美參考，但也要警覺：很多作品很炫，未必適合你的品牌或 mobile performance。

- [Class Central 2026 Three.js courses guide](https://www.classcentral.com/report/best-three-js-courses/)  
  可快速掃免費 / 付費課程地圖。它也提醒官方 docs 對新手可能完整但 overwhelm，因此 project-based course 比較容易起步。

### 社群建議

- [Reddit: total beginner learning Three.js](https://www.reddit.com/r/threejs/comments/1lp37aq/the_best_way_to_learn_threejs_as_a_total_beginner/)  
  代表性建議是：先補 web / JS 基礎，再進 Three.js Journey；不要一開始被 R3F 分散，先理解 Three.js 本身。也有人提醒，如果只是把模型放上網站，AI 已經能做不少，但模型材質、光照、baking、Blender 到 browser 的落差仍是難點。

- [Three.js forum roadmap](https://discourse.threejs.org/t/three-js-roadmap/75313)  
  社群常見方向是先做小 scene，再逐步加 camera、lighting、controls、best practices，而不是一開始追求大而全。

## 不建議一開始碰太深的東西

先不要急著碰：

- raw WebGL
- shader 從零寫
- GPGPU
- WebGPU
- physics-heavy game
- multiplayer 3D
- full-screen immersive site
- complex skeletal animation
- 自製 3D model pipeline

這些都可以晚點碰。現在最有價值的是「網站裡可控、可部署、可維護的 3D moment」。

## 給 Jovicheer / ObserverJ / Raijax 的實際建議

### Jovicheer

方向：品牌入口、作品宇宙、柔和但有生命感的 3D hero。

最適合學：

- lighting / material
- scroll subtle motion
- responsive hero
- reduced motion fallback
- model / abstract geometry

不要做成：一進站就重度 3D loading game。

### ObserverJ

方向：思想觀察、文章、Thought Atlas 外部出口。

最適合學：

- constellation / node graph
- labels / HTML overlay
- hover / focus interaction
- data-driven scene
- camera transition

不要做成：看起來像 crypto dashboard 或太科幻而壓過文字。

### Raijax

方向：AI agent / human interaction / playful experiments。

最適合學：

- interactive object
- cursor response
- simple state machine
- game-like affordances
- 3D as interaction grammar

不要做成：只有裝飾性 3D，卻沒有互動意義。

## 最終建議

Jones 的最佳路線不是「我要學會 Three.js 才能做 3D 網站」，而是：

> 我先學會 3D web 的基本語言，讓 agent 幫我快速做 prototype；  
> 我再用審美、產品感、效能檢查與品牌判斷，把 prototype 收斂成可用作品。

如果只選一條最小路線：

1. 用 Three.js manual 做 3 個 raw demo
2. 用 R3F + Drei 做 3 個網站 pattern
3. 用 glTF / Blender asset 做 1 個 model viewer
4. 做 1 個 Jovicheer / ObserverJ / Raijax 的真實小 prototype
5. 把過程整理成自己的 `3d-web-starter` 與 agent prompt library

這樣學到的不是語法，而是未來每次要做 3D 網站時都能重複使用的判斷力。


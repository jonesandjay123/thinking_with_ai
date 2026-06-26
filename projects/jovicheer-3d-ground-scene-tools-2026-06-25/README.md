# Jovicheer 3D Asset Lab：參照圖到 3D 地景工具調查報告

> 撰寫時間：2026-06-25  
> 對象：`3d-asset-lab` 下一階段地面 / 場景實驗  
> 核心問題：已經有 2D 地面設計想法與成功轉成 3D 的木樁後，下一步如何把「地面」變成可控、可驗收、可交給 web runtime 使用的 3D / 2.5D scene piece？

## 0. Executive Summary

我的建議不是「停止 `3d-asset-lab`，先去學完 Blender / Unity / Unreal 再回來」，也不是「全部交給 AI 2D→3D 一鍵生成」。

更好的路線是：

> Jones 這個週末先摸 Blender 的最小必要流程；同時把 `3d-asset-lab` 當成 AI 2D→3D、Blender、專用地形工具、外部素材庫的驗收儀表板。

原因是：木樁這類 hero prop 是 bounded object，一張圖轉成一個 3D 物件，只要外形、比例、材質大致成立，就能先用。地面不是單一物件，而是整個舞台座標系。它要回答高度、厚度、坡度、路徑、落地點、前中後景、camera framing、runtime 檔案大小、未來第二個物件怎麼擺等問題。

所以本階段不該追求「哪個工具最神」，而該建立一個比較流程：

1. 同一張 2D 地面參照圖，丟給 AI 2D→3D 工具。
2. 同一張參照圖，用 Blender 做 heightfield / blockout terrain slab。
3. 同一張參照圖，用 `3d-asset-lab` 的 procedural terrain baseline 做 browser-native 對照。
4. 全部放進 `3d-asset-lab` Stage Sandbox，讓木樁站上去，用同一套標準評分。

第一階段最值得投入的是 Blender，不是因為 AI 工具沒用，而是因為 Blender 會讓我們看懂 AI 產物壞在哪裡、該怎麼修、怎麼清成 runtime-safe GLB。

## 1. 為什麼地面不能直接照木樁的成功經驗做？

木樁成功很重要，代表「AI image-to-3D 產 hero prop」這條路可以繼續推。但地面是不同問題。

### 木樁是 bounded object

木樁的輸入與輸出關係相對簡單：

- 一張圖
- 一個物件
- 有中心、有高度、有邊界
- 背面不完美也可以接受
- 遠看成立就能先放進場景

這類物件很適合用 Meshy、Tripo、Rodin、Higgsfield 這類工具快速出候選版。

### 地面是 spatial system

地面需要回答的是：

- 哪裡高、哪裡低？
- 哪裡是路、哪裡是草、哪裡是石頭？
- 木樁的 `x / y / z` 座標在哪裡？
- 木樁站上去會不會浮空或陷進去？
- 地面有沒有厚度？
- camera 固定時好看，換一點角度會不會破？
- 檔案大小、面數、貼圖大小能不能進 web runtime？
- 之後第二個、第三個 prop 要怎麼放？
- 要改一條路或一個坡，是局部修改，還是整包重生？

這也是為什麼很多 AI 直接 2D→3D 的地面結果會變成「漂亮但不可控的浮雕 / blob」。它看起來像場景，但不一定是一塊可以放物件、可以量測、可以封裝的 terrain slab。

## 2. 工具類別地圖

這件事可以分成五類工具，不要混成一個問題。

### A. AI 參照圖 / 文字轉 3D 物件工具

適合用來產：

- 木樁、石碑、燈、符號物
- 單棵樹、石頭、小橋、拱門
- foreground cutout 或 scenic silhouette 的 3D 候選
- 小型地形片段的 baseline 測試

不適合直接信任為：

- 完整可走地面
- 大範圍 terrain
- 有穩定座標系的場景
- 需要局部可編輯的路徑 / 山坡 / 草地

代表工具：

- [Meshy](https://www.meshy.ai/)：主打 text / image to 3D models，並提到內建後處理、poly budget、mesh 修復、Blender / Unity / Unreal / Godot plugin 與 API。適合列為第一批 prop 生成工具。
- [Tripo](https://www.tripo3d.ai/)：主打 text / image to 3D model。適合快速產出多個候選物件，但要用 `3d-asset-lab` 檢查 mesh / texture / scale。
- [Hyper3D Rodin](https://hyper3d.ai/)：主打 image / text to 3D，並提供 split、partial edit、3D ControlNet、low-poly、GLB / FBX / OBJ / STL / USDZ 等輸出。因為它強調控制與局部修改，值得優先試。
- Higgsfield：目前 `3d-asset-lab` 已經有實驗脈絡。木樁方向可繼續；地面則應當作 baseline，不要預設會是最終地形。

### B. Blender / DCC 清理與地形 slab 管線

Blender 是下一步最值得摸的工具。

不是因為 Jones 要變成 3D artist，而是因為我們需要懂：

- unit / scale
- origin / pivot
- camera framing
- mesh cleanup
- UV / texture
- displacement / height map
- decimate 降面數
- export GLB
- 把木樁放到地面上測比例與落地

[Blender](https://www.blender.org/features/) 本身是完整 3D creation suite，也有 Python API 可以自動化。官方 glTF 文件也明確說 glTF 是為 web / native app 傳輸與載入設計的格式，Blender 的 glTF exporter 支援 meshes、PBR materials、textures、cameras、lights、animations 等。這和 `jovicheer-world-stage` 未來的 web / Three.js runtime 很匹配。

本階段 Blender 的任務不是做完整世界，而是做乾淨小零件：

```text
2D reference
→ rough height map / path mask / material mask
→ Blender terrain slab
→ 放入木樁測比例與落地
→ export GLB
→ 進 3d-asset-lab 驗收
```

### C. 專用地形工具

這類工具不一定要立刻買 / 深學，但很值得知道它們在解什麼問題。

- [Gaea](https://quadspinner.com/)：專業 procedural terrain design tool，主打 non-destructive、erosion、生態與 terrain build / bake。它甚至明確說自己不使用 AI / ML。適合未來要做更自然的山谷、山脊、侵蝕地形、height map / mask pipeline。
- [World Creator](https://www.world-creator.com/)：realtime terrain generator，主打 GPU 加速、procedural + manual sculpting、non-destructive workflow、scatter 3D objects。適合快速探索比較完整的 landscape / foliage / scatter 場景。

這類工具的價值不是「一鍵生成 Jovicheer 場景」，而是：

- 產 height map
- 產 slope / erosion / flow / material masks
- 產 terrain texture
- 產地形設計參考
- 幫 Blender / Three.js pipeline 提供更乾淨的 source data

### D. Unity / Unreal 場景搭建工具

Unity / Unreal 很值得看，但不要第一天就把它們當最終 runtime。

原因是 `jovicheer-world-stage` 的合理 runtime 仍然偏 web / Three.js / React。Unity / Unreal 比較適合扮演「理解場景搭建與 authoring」的老師，而不是直接變成網站本體。

Unity 的 Terrain 官方文件提到，Unity editor 內建 Terrain features，可以建立 terrain tiles、調高度與外觀、加入樹與草，並支援 heightmap import / export。這很適合學「遊戲引擎怎麼看地形」。

Unreal 的 Landscape system 官方文件則把它定位成一套建立 expansive outdoor environments 的工具集合，可以 sculpt Landscape Actors、建立材質並 paint 到 landscape 上。這很適合理解高品質 outdoor scene 的邏輯。

它們適合用來：

- 學 terrain / foliage / lighting / camera 的概念
- 用 asset packs 快速組 reference scene
- 產 reference screenshots / videos
- 看清楚遊戲引擎如何把地面拆成 height、material、scatter、lighting

暫時不建議：

- 直接把 Unity / Unreal WebGL build 當 Jovicheer 網站 runtime
- 為了做一塊小地面而掉進完整 engine 生態
- 把 engine 裡的巨型場景包原封不動搬進 web

### E. 外部素材庫與材質工具

Jovicheer 不需要每顆石頭、每株草都自己生成。素材庫應該成為 kitbash 來源。

可優先看：

- [Poly Haven](https://polyhaven.com/)：public 3D asset library，提供 HDRI、textures、models。適合找材質、HDRI、少量自然物件。
- [Quaternius](https://quaternius.com/)：大量 free game assets，包含 stylized nature、medieval village、props、modular kits。風格偏遊戲、低模、可控，適合 prototype。
- [Kenney](https://kenney.nl/assets)：大量 game assets。若走 stylized / low-poly blockout 很方便。
- Sketchfab：下載型 3D model 來源很多，但 license、面數、材質品質落差大，使用前要逐一檢查。
- Fab / Megascans：品質高，但容易太寫實、太重，並且需要注意授權與 web runtime 成本。

材質工具：

- [Adobe Substance 3D Sampler](https://www.adobe.com/products/substance3d/apps/sampler.html)：可以從影像產生 albedo、roughness、normal、displacement 等 maps，也能處理 tiling、shadow removal、surface details。適合把 2D 地面圖拆成可用材質 maps。
- Material Maker：開源 procedural PBR material tool，可作為低成本材質實驗。

## 3. 建議優先測試清單

### 第一梯隊：這週末就能試

1. Blender
   - 目標：做一塊簡單 terrain slab，放上木樁，匯出 GLB。
   - 學習範圍：scale、origin、camera、plane subdivision、displace、texture、decimate、GLB export。

2. Meshy
   - 目標：用同一張地面 concept 圖測 direct image-to-3D ground；另外再測 1-2 個小 prop。
   - 驗收：面數、檔案大小、pivot、木樁是否能站上去。

3. Hyper3D Rodin
   - 目標：測同一張地面 concept 圖，以及 1 個 prop。特別看 partial edit / low-poly / control 能不能救地面。
   - 驗收：是否比 Meshy 更可控，是否能輸出乾淨 GLB。

4. `3d-asset-lab` procedural terrain baseline
   - 目標：用 repo 內現有 procedural terrain 方向做對照組。
   - 驗收：固定鏡頭下木樁落地、前中後景、效能與可控性。

### 第二梯隊：下週再看

1. Tripo
   - 適合補 prop generation comparison。

2. Substance 3D Sampler / Material Maker
   - 適合把地面圖變成材質 maps，而不是直接變 3D mesh。

3. Poly Haven / Quaternius / Kenney
   - 適合補石頭、草、樹、木頭、小道具。

### 第三梯隊：有明確需求再深入

1. Gaea
   - 當我們要做更自然的山谷 / 山坡 / erosion terrain。

2. World Creator
   - 當我們要快速探索完整 landscape + scatter。

3. Unity / Unreal
   - 當 Jones 想理解場景搭建流程，或要用 asset pack 做 reference scene / previs。

## 4. 評分標準：不要只看截圖

每個工具輸出的東西，都要進 `3d-asset-lab` 用同一套標準看。

### 必看指標

- 檔案大小：GLB / texture 是否太重？
- 面數：triangles 是否可接受？是否能 decimate？
- bounds：地面範圍是否合理？有沒有巨大的不可見空間？
- pivot / origin：是否在可預期位置？
- scale：木樁放上去比例是否可信？
- thickness：地面是否有可接受的厚度，還是薄紙 / 厚 blob？
- landing：木樁是否自然落地，不浮空、不陷入、不斜倒？
- camera：固定鏡頭下是否有前中後景？
- editability：能不能局部改路徑 / 高度 / 材質？
- runtime fit：Three.js / web 是否能順利載入？
- license：是否能用於未來產品 / 商業用途？

### 直接淘汰條件

- 看起來像場景，但無法判斷地面高度。
- 木樁放上去永遠浮空或陷入。
- mesh 是不可整理的高面數 blob。
- texture 全 baked 成單張透視圖，換角度就破。
- 檔案太大，只能當截圖參考。
- 無法穩定輸出 GLB / glTF / OBJ / FBX。

## 5. 週末實作流程

### Friday：建立 Blender 最小手感

目標不是做美，而是知道每個東西在 3D 空間裡怎麼被控制。

任務：

- 安裝 / 打開 Blender。
- 匯入目前成功的木樁 GLB。
- 建一個 plane 當地面。
- 調 scale，讓木樁大小看起來合理。
- 把 camera 固定成接近 Jovicheer stage 的角度。
- 匯出一個最簡單的 `post-on-plane.glb`。
- 丟回 `3d-asset-lab` 看是否能載入與驗收。

學習點：

- object mode / edit mode
- transform：move / rotate / scale
- origin / pivot
- camera framing
- GLB export

### Saturday：做第一塊 heightfield terrain slab

目標是讓地面不只是平面貼圖，而是有基本起伏。

任務：

- 用一張 2D 地面 concept 圖當 reference。
- 手工或用 AI 輔助生成：
  - height map
  - path mask
  - grass / dirt / stone material mask
- 在 Blender 用 subdivided plane + displace modifier 做 terrain。
- 加一張簡單 top texture。
- 放上木樁。
- 降面數 / 清 origin / export GLB。
- 放進 `3d-asset-lab` terrain-slab experiment。

驗收：

- 木樁站得住。
- 地面有一點坡度與厚度。
- 固定鏡頭下看起來像可用舞台。
- GLB 不爆量。

### Sunday：AI direct 2D→3D 同場比較

目標是確認 AI 工具直接做地面的上限，而不是盲信或盲拒。

任務：

- 用同一張地面 concept 圖丟 Meshy / Rodin / Higgsfield。
- 每個工具至少輸出一個 GLB / OBJ。
- 全部進 Blender 快速檢查 scale / origin / mesh。
- 進 `3d-asset-lab` Stage Sandbox。
- 用同一個木樁 placement 測落地。

輸出：

```text
ground-ai-meshy-001
ground-ai-rodin-001
ground-ai-higgsfield-001
ground-blender-heightfield-001
ground-procedural-baseline-001
```

最後用一頁 notes 記：

- 哪個最漂亮？
- 哪個最可控？
- 哪個最容易修？
- 哪個最適合當 runtime asset？
- 哪個只適合當 reference？

## 6. `3d-asset-lab` 應該新增的實驗 contract

這個 repo 接下來最有價值的不是多塞很多模型，而是定義一個可重複驗收格式。

建議每個 ground experiment 至少包含：

```text
terrain-experiment/
├── source-reference.png
├── notes.md
├── terrain.glb
├── preview.png
├── maps/
│   ├── height.png
│   ├── albedo.png
│   ├── path-mask.png
│   └── scatter-mask.png
└── placement.json
```

`placement.json` 可以先很簡單：

```json
{
  "regionId": "valley-path-001",
  "placements": [
    {
      "propId": "wooden-post-001",
      "localPosition": [1.2, 0.18, -0.7],
      "rotationY": -0.35,
      "scale": 0.85,
      "depthBand": "midground"
    }
  ]
}
```

這個 contract 比「多找一個 AI 工具」更重要。因為一旦 contract 穩了，任何工具輸出的東西都能被比較。

## 7. 我對每類工具的判斷

### AI 2D→3D 工具

值得測，而且應該持續測。它們會越來越好。

但現在最可靠的用法是：

- 產 hero prop
- 產小型 scenic object
- 產地面候選 reference
- 產可被 Blender cleanup 的粗 mesh

不建議把它們當成唯一地面生產線。

### Blender

最值得這週末投入。

Jones 不需要先變成 Blender expert，只要會把地面 / 木樁 / camera / export 這幾件事串起來，就會立刻提高整個 pipeline 的判斷力。

### Gaea / World Creator

很有潛力，但先不用急。等我們已經知道 Jovicheer 的 terrain slab contract，再看它們能不能更有效地產 height map、mask、texture、scatter。

### Unity / Unreal

適合學場景搭建概念與找現成模組，不適合一開始就變成 Jovicheer web runtime。

我會把它們當成「場景 authoring 參考工具」：

- 看 terrain / foliage / lighting / camera 怎麼被專業工具組織。
- 需要時用 asset pack 快速組 reference。
- 產截圖 / video / layout idea。

但最終仍然回到 Blender / GLB / `3d-asset-lab` / Three.js contract。

## 8. 最終建議

短期不要問「哪個工具可以一鍵把 2D 場景變成 3D」。更好的問題是：

> 哪個工具能提供一段可被驗收、可被清理、可被放進 web runtime 的 scene piece？

因此下一步建議是：

1. Jones 週五 / 週末摸 Blender 最小流程。
2. Jarvis 在 `3d-asset-lab` 裡把 ground experiment 的驗收格式補清楚。
3. 同一張地面參照圖跑三條路：AI direct、Blender heightfield、procedural baseline。
4. 用木樁落地效果當第一個真實驗收點。

這樣 `3d-asset-lab` 就不是暫停，而是從「看 3D asset」升級成「把 2D concept 變成 runtime-safe scene pieces」的實驗台。

## 9. 參考來源

- [Meshy](https://www.meshy.ai/)
- [Tripo](https://www.tripo3d.ai/)
- [Hyper3D Rodin](https://hyper3d.ai/)
- [Blender Features](https://www.blender.org/features/)
- [Blender glTF 2.0 exporter manual](https://docs.blender.org/manual/en/latest/addons/import_export/scene_gltf2.html)
- [Gaea / QuadSpinner](https://quadspinner.com/)
- [World Creator](https://www.world-creator.com/)
- [Unity Terrain manual](https://docs.unity3d.com/Manual/script-Terrain.html)
- [Unreal Engine Landscape documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/landscape-outdoor-terrain-in-unreal-engine)
- [Poly Haven](https://polyhaven.com/)
- [Quaternius](https://quaternius.com/)
- [Kenney Assets](https://kenney.nl/assets)
- [Adobe Substance 3D Sampler](https://www.adobe.com/products/substance3d/apps/sampler.html)

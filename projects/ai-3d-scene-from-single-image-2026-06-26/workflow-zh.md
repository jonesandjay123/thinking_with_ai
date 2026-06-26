# 用 AI 從單張圖建立 3D 場景：影片步驟重點

來源影片：<https://www.youtube.com/watch?v=fxMUIRxWQU0>

影片主題是使用 Marble AI，把單張參考圖、文字提示、影片或 panorama 轉成可走動的 3D 空間。它不是傳統手工建模流程，而是以 Gaussian Splatting 為主，把一張圖擴展成可環視、可移動、可匯出的場景。對 stage scene 類型專案來說，這條路線最有價值的地方是：可以快速生成「有整體氛圍和空間感的環境底稿」，再把真正需要互動、碰撞、角色落腳、可控動畫的物件交給 runtime 或 Blender 另外處理。

## 核心判斷

這個方案最適合拿來做：

- 場景背景、沉浸式環境、環形世界、房間或廣場類空間。
- cinematic preview、概念驗證、landing scene 的氛圍打樣。
- 需要快速從 2D concept art 轉成可走動 3D 空間的探索階段。
- 和現有 3D props 組合：AI 生成大環境，人手或既有 pipeline 放置主角、道具、入口、互動物。

它不適合單獨承擔：

- 高精度 gameplay collision。
- 需要乾淨拓撲、可編輯 UV、可精準動畫的 production mesh。
- 玩家會非常貼近檢查的 foreground hero asset。

最好的用法不是「把整個世界一次生成完」，而是把它當成 3D 場景的快速空間底板。

## 影片中的完整流程

### 1. 準備單張參考圖

影片示範先準備幾張不同風格的 image prompt，例如王座大廳、指揮中心、溫泉浴場、動畫風房間。這些圖可以是 AI 生成圖、概念圖、截圖風格參考或手繪風格圖。

實作建議：

- 先用 image model 做一張「空間感明確」的圖，而不是只畫漂亮主物件。
- 鏡頭最好有地面、牆面、天花板或遠景深度，方便 AI 推測空間。
- 如果之後要放 3D 主物件，參考圖裡可以先不要放太強的主物件，或之後用 inpainting 移除。

### 2. 在 Marble AI 建立 world

Marble AI 可以吃 image、video、panorama，也可以使用 3D scene 作為 reference。影片主要示範 image input：

1. 新增 image。
2. 可加額外文字指令。
3. 開啟 advanced editing。
4. 按 create。
5. 等待約數分鐘，先生成 panorama。

advanced editing 的價值是，它會先產生 panorama，讓你在真正生成 3D world 前先修圖。這一步很重要，因為之後 splat world 會繼承這張 panorama 的錯誤。

### 3. 先修 panorama，再生成 3D world

影片裡的例子是王座大廳。作者想之後自己放一個 3D 王座模型，所以先在 Marble AI 裡輸入：

```text
remove the throne, make hall smaller
```

Marble AI 會用 AI inpainting 把王座移除，並修改大廳尺度。作者也示範繼續用文字修改燈光，例如讓畫面更暗。確認 panorama 結果後，再按 create world。

實作建議：

- 把「主角 / 可互動物 / 精準模型」從 AI 場景底圖中移除。
- 讓 AI 保留大環境：牆、地、遠景、窗、天空、霧、光源、氛圍。
- 在生成 world 前先修掉明顯錯誤，因為 world 生成後再修成本更高。

### 4. 檢查生成後的可走動空間

生成 world 後，可以在 Marble AI 裡直接移動視角。影片示範按 shift 可以加速移動。作者強調這不是完整傳統 3D 世界，而是一個可走動的 3D space；它的優勢是空間感、反射、氛圍和視覺完整度很好。

需要注意：

- 離初始視角越遠，splat 品質會下降。
- 遠距離看很漂亮，貼近看可能出現 splat 破碎或形狀怪異。
- 它適合作為環境與視覺背景，不應直接等同 production-ready mesh。

### 5. 也可以用 3D scene 當 reference

影片另一條路線是使用 3D scene input。作者先建立簡單幾何或擺放物件，Marble AI 會讀取空間約束或 depth map，再生成符合這個空間結構的 panorama。

這對我們的啟發很重要：如果單張圖不夠可控，可以先用極簡 blockout 建立空間骨架，再讓 AI 補上風格、材質和氛圍。

可能流程：

1. 在 Blender / Three.js / Spline / 任意簡單 3D 工具做 low-poly blockout。
2. 輸出 reference image 或 3D scene。
3. 丟進 Marble AI 生成 panorama/world。
4. 用生成結果反向當作 background world 或概念稿。

### 6. 匯出格式

影片列出幾個重要 export options：

- Panorama：可當 environment texture 或 2D 背景資產。
- Splat / PLY：可匯入 Blender、Unreal Engine 或支援 Gaussian Splatting 的工具。
- Collider mesh / GLB collider：可給 Unreal Engine 類工具做可走動碰撞空間。
- High quality mesh：把環境 bake 成 mesh，耗 credits，等待可能到一小時，但可以在不支援 splat 的 3D 軟體中使用。
- Screenshot / video：可直接生成 demo、預告或 cinematic proof。

對 stage scene pipeline 的建議：

- 第一優先：匯出 panorama 和 splat。
- 第二優先：如需要角色落地或碰撞，另外匯出 collider mesh。
- 第三優先：只有在 splat 不方便 runtime 使用時，才測 high quality mesh。

### 7. 在 Marble 裡做影片 preview

影片示範 Marble AI 內建 camera keyframe：

1. 選定起始 camera view。
2. 加 keyframe。
3. 移動或旋轉視角。
4. 再加 keyframe。
5. preview。
6. 匯出影片。

也可以付 credits 做 AI enhance / upscale。這對我們的用途不是核心 production 路線，但很適合快速做概念展示。

### 8. Studio Mode 組合多個 worlds

影片示範進入 Studio Mode 後，可以：

1. 打開一個 world。
2. 用 brush 選取某些 splat 區域。
3. 按 backspace 移除不需要的 splat。
4. import 另一個 world 或 splat。
5. 把多個場景片段拼在一起。

作者認為這對完全不同概念的拼接不一定合理，但對同一概念下不同場景片段的組合有用。

實作啟發：

- 可以把 AI 生成的「地面 / 背景 / 牆面 / 遠景」拆成幾塊。
- 只保留好看的局部，刪掉錯誤局部。
- 把多個生成結果拼成更完整的 stage environment。

### 9. 匯入 Blender

影片示範兩個 Blender 路線：

- Gaussian Splatting Blender add-on。
- Kirin Engine。

基本步驟：

1. 從 Marble AI 匯出 splats PLY。
2. 在 Blender 安裝 Gaussian Splatting add-on。
3. 使用 import Gaussian Splatting 匯入 PLY。
4. 切到 viewport shading 或 render mode 才能正確看見效果。
5. 如使用 Kirin Engine，匯入後可能需要旋轉 90 或 180 度。
6. 開啟 camera update / render splats 等設定。

作者提醒 Blender 裡要調整參數才能有好品質；Unreal Engine 可能是更理想的 splat preview / real-time 呈現工具。

### 10. 匯入 Unreal Engine

影片認為 Unreal Engine 是最簡單、效果最好的目的地之一。splat 在 Unreal Engine 裡可以很漂亮地即時渲染，也可以搭配 collider mesh 變成可走動空間。

如果 stage runtime 最後不走 Unreal，仍可把 Unreal 當作參考：

- 驗證 splat 在 real-time engine 裡的視覺表現。
- 評估 collision mesh 是否足夠支援角色走動。
- 用它快速判斷 Marble 輸出的場景是否值得進一步移植。

## 建議的最小實驗 SOP

### A. 概念圖生成

先生成 3 到 5 張 stage environment reference。每張圖都要符合：

- 有明確地面。
- 有可站立的中心區域。
- 有背景深度。
- 主物件不要太多，避免之後難清。

### B. Marble AI world generation

每張圖都跑：

1. image input。
2. advanced editing。
3. 先看 panorama。
4. 用 inpainting 清掉不該 baked-in 的主物件。
5. 調整光線和尺度。
6. create world。

### C. 匯出三種 baseline

每個成功 world 至少輸出：

- panorama。
- splat PLY。
- collider mesh / GLB collider，如果有提供。

如果某個 world 特別好，再測 high quality mesh。

### D. 在 3D 工具裡驗證

Blender 驗證：

- 能否匯入 PLY。
- 軸向是否正確。
- 地面位置是否可對齊。
- 視角是否能放進現有 stage camera。
- 是否能加入獨立 3D props。

Runtime 驗證：

- splat 是否能在 target runtime 中穩定顯示。
- 如果 runtime 不支援 splat，是否可用 panorama + mesh collider / baked mesh 替代。
- foreground props、角色、互動物是否需要和 splat 分層。

### E. 選出 production direction

每個候選場景用這些標準評分：

- 氛圍是否接近目標世界。
- 中心地面是否乾淨。
- camera 移動時破圖是否可接受。
- 是否容易加 props。
- 匯出後是否能進 Blender / runtime。
- credits 與等待時間是否可接受。

## 對目前 3D 場景工作的結論

影片裡的路線很適合作為「AI 快速建立可走動 3D 環境」的探索工具。最便利的解法不是追求一次生成完整可控遊戲場景，而是：

1. 用 AI 圖像或簡單 blockout 定義場景概念。
2. 用 Marble AI 生成 panorama 和 splat world。
3. 用 inpainting 把未來要獨立控制的主物件移除。
4. 匯出 splat / panorama / collider。
5. 在 Blender 或 runtime 中把 AI 環境當作 background world。
6. 再疊加真正可控的 3D props、角色、入口、互動物與碰撞。

這剛好補上 stage scene 裡最難、最花時間的部分：大環境、氣氛、空間感、遠景和環境包覆感。真正需要精準控制的東西，仍然應該留在既有 3D asset pipeline 裡處理。

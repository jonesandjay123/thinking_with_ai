# Jcompany 水平無縫循環背景研究

> 2026-06-08 研究整理。目標是幫 Jcompany 第一版首頁做一個穩定、輕量、溫柔的水平環繞背景：柔和霧氣、奶油金 / 淡粉 / 柔白光源、空氣深度、漂浮 wish ribbons，使用者像站在原地慢慢左右觀看一個會回到原位的空間。

## 結論先講

Jcompany 第一版不建議繼續把「天空色塊、太陽、霧、ribbon placeholders」全部放在一個 4x viewport world 裡一起移動，然後靠複製三份來補 buffer。這種做法只要左邊界和右邊界不是同一張可平鋪材質的自然延續，接縫一定會出現；霧氣和柔光尤其容易露餡，因為人眼對大面積低頻漸層的不連續很敏感。

最穩的第一版架構是：

1. 底色與大光源固定在 viewport，不跟 world 走。
2. 霧氣拆成 2-3 張「水平可重複」的半透明 WebP / AVIF strip，用 `background-repeat: repeat-x` 或 pseudo-elements 做平移。
3. ribbon / 顯眼物件用 world coordinate 管理，但要做 wrap-around duplicate：接近左邊界的物件在右邊也出現副本，反之亦然。
4. 每一層的循環距離不同，且霧氣層低對比、模糊、透明，讓接縫不會在同一時間同一位置疊出來。
5. seam 測試要把相機直接移到 `x = 0` / `x = worldWidth - viewportWidth` 附近看，不要只看正常慢速動畫。

## A. 最適合 Jcompany 第一版的推薦方案

### CSS/DOM + 預製 tileable 霧氣 strips + 固定光源

這是最適合 landing page v1 的方案。技術上很樸素，但最符合「穩定、手機可用、不要太重、不要像遊戲 tech demo」。

建議層級：

```text
hero
├── sky-base: fixed viewport gradient, 不水平移動
├── sun-glow: fixed viewport radial-gradient / blurred div, 不進 loop seam
├── far-fog: repeat-x tileable strip, very slow offset
├── mid-fog: repeat-x tileable strip, medium offset
├── near-haze: repeat-x tileable strip, slightly faster offset + mask
├── ribbon-world: 4x viewport logical world, transform: translateX(...)
└── foreground-soft-vignette: fixed masks / subtle light wash
```

CSS 核心：

```css
.fogLayer {
  position: absolute;
  inset: 0;
  background-image: image-set(
    url("/assets/fog-mid.avif") type("image/avif"),
    url("/assets/fog-mid.webp") type("image/webp")
  );
  background-repeat: repeat-x;
  background-size: 1600px 100%;
  background-position: calc(var(--camera-x) * -0.18) 50%;
  opacity: 0.34;
  filter: blur(10px);
  pointer-events: none;
}

.fogLayer::after {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(
    90deg,
    rgba(255,255,255,0.18),
    transparent 18%,
    transparent 82%,
    rgba(255,255,255,0.18)
  );
  mix-blend-mode: screen;
}
```

重點不是「做一張很大圖」，而是「每張會水平重複的素材本身就沒有左右接縫」。MDN 的 `background-repeat` 支援 `repeat-x`，但它只負責重複，不會替你修圖；若 tile 的左右邊緣不匹配，CSS 只會忠實地把 seam 重複出來。

### 優點

- 性能成本最低：主要是幾張圖片背景和 transform / background-position。
- 最容易做 mobile fallback：少一層霧、降低 blur、停掉自動 drift 即可。
- 視覺最貼近 Jcompany：柔霧、光、空氣感都能靠美術資產和 CSS blend 完成。
- 不需要一開始引入 Three.js / shader pipeline。
- 可把接縫問題收斂到「少數 tileable strips」而不是整個 DOM world。

### 缺點

- 需要做或生成真正 tileable 的霧氣素材。
- 若 ribbon 也要跨邊界，DOM layout 要有 wrap duplicate 邏輯。
- CSS `background-position` 小數位移在某些裝置上可能有微閃，需用整數 pixel 或 transform layer 測試。

## B. 第二適合的備選方案

### Canvas 2D：啟動時預渲染 tileable fog texture，之後只 offset drawImage

如果想降低 DOM 層數，或想做比 CSS 更自然的霧氣擾動，可以用 Canvas 只負責霧氣層。不要每幀重新生成 noise；啟動時在 offscreen canvas 生成 1-2 張低解析 tile，例如 1024x512 或 2048x768，之後每幀只畫兩到三份貼圖。

核心方式：

```js
function drawLoopingTile(ctx, img, offsetX, width, height) {
  const x = ((offsetX % width) + width) % width;
  ctx.drawImage(img, -x, 0, width, height);
  ctx.drawImage(img, width - x, 0, width, height);
  if (x > width - canvas.width) {
    ctx.drawImage(img, width * 2 - x, 0, width, height);
  }
}
```

Canvas 性能原則：

- 啟動時生成 / 載入 tileable fog texture。
- 每幀只做 offset、alpha、drawImage。
- 低解析度 texture 放大，再用 CSS / canvas blur 製造柔度。
- `requestAnimationFrame` 跟著瀏覽器 repaint；背景 tab 通常會暫停，有助於省電。
- mobile 可降到 30fps，或使用 CSS 靜態 fallback。

web.dev 的 Canvas 性能文章建議把重複 primitive 預渲染到 offscreen canvas，再貼回 visible canvas；MDN 也有 Canvas optimization 指南。這正好符合霧氣背景：生成成本高，但視覺結果可重複使用。

### 優點

- 比純 CSS 更容易做程序化霧氣、低頻變化、alpha compositing。
- 可以把霧氣渲染收斂到一個 canvas，DOM 更乾淨。
- 預渲染後每幀成本可控。

### 缺點

- 比 CSS 更需要測 mobile GPU / CPU。
- Canvas 內元素不可被 CSS 逐層調整，設計迭代較慢。
- 若 resize / DPR 處理不好，會糊或耗記憶體。

## C. 不建議現在採用的方案

### 1. 一張超寬 panorama 扛全部背景

可做，但不適合當唯一方案。8:1 或 4:1 的大圖很容易大、難 responsive，而且只要左右邊界不是嚴格無縫，seam 仍然存在。它適合做遠景氛圍，不適合同時承擔霧、光、ribbon、互動視差。

### 2. Heavy WebGL volumetric fog / particle field

Jcompany 第一版不需要真正 3D 霧體。大量 particles、raymarching volumetric fog、複雜 postprocessing 都會讓手機和普通筆電變成風險點，而且視覺容易偏科幻遊戲。

### 3. 純 SVG/CSS procedural 大世界

SVG `feTurbulence` 有 `stitchTiles="stitch"`，可用來做可拼接 noise；但把所有霧、雲、光源、物件都用 SVG filter / DOM procedural 組起來，debug 成本可能比做 2-3 張霧氣 texture 更高。適合做局部 noise overlay，不適合第一版承擔整個世界。

## D. 技術路線比較

### Seamless tile texture

做一張或多張可水平平鋪的霧 / 雲 / haze strip，然後 repeat-x。

優點：
- 最穩、最省、最容易驗收。
- seam 問題可以在素材階段解決。
- 可以用 Photoshop / AI / procedural 工具做。

缺點：
- 重複模式可能被看出來，需要多層不同尺寸、速度、透明度打散。
- 素材需要手工驗收。

### 超寬 panorama

生成 4:1 / 8:1 的大背景，把左邊和右邊修到可以接。

優點：
- 美術控制完整，整體氛圍容易統一。
- 可作為遠景底圖。

缺點：
- 大圖載入慢；手機上高 DPR 會更貴。
- 只要邊界色溫、雲形、光源不一致，接縫很明顯。
- 不適合把強光源放在 seam 附近。

### SVG / CSS procedural layers

用 radial-gradient、linear-gradient、SVG `feTurbulence`、mask、blend mode 組合。

優點：
- 不依賴大圖，調色快。
- `feTurbulence stitchTiles="stitch"` 對 tileable noise 有幫助。
- 可做細微 grain / haze。

缺點：
- 大面積 SVG filter 在手機上可能昂貴。
- 低頻雲霧形狀不一定自然。
- 跨瀏覽器視覺差異要測。

### Canvas tileable noise

在 Canvas 生成或載入 tileable fog，再用 offset 繪製。

優點：
- 可做自然噪聲和動態。
- 可預渲染，避免每幀重算。

缺點：
- 需要手寫 render loop、DPR、resize、fallback。
- 對 landing page v1 來說比 CSS 多一層風險。

### Photoshop / AI 無縫接圖

用 Offset filter / Generative Fill / Content-Aware Fill 修 seam。

優點：
- 最適合有明確視覺方向的背景資產。
- 能把 AI 生成圖轉成可用 web texture。

缺點：
- 需要人工反覆 tile test。
- AI 容易在 seam 產生局部合理但整體不連續的形狀。

### Mirrored / blended edges

把邊緣鏡射、疊加、羽化、crossfade。

優點：
- 快速救 seam。
- 對抽象霧氣有效。

缺點：
- 可能出現鏡像對稱感。
- 對有方向性的雲、光、物件不自然。

### Multiple parallax layers with different loop lengths

每層有不同 tile width / speed / opacity。

優點：
- 降低「所有 seam 同時出現」的感覺。
- 也能增加空氣深度。

缺點：
- 每一層本身仍要 tileable；不同 loop length 不是 seam 修復，只是 seam 分散。

### Noise texture offset animation

對 tileable noise 做 background-position / texture offset。

優點：
- 很適合霧、柔光 breakup、微動。
- 成本低。

缺點：
- 不能直接承擔具象雲形或 ribbon。

### Shader / WebGL periodic noise

用 periodic noise 或把 x 座標映射成圓周，產生左右相接的 noise。

優點：
- 理論上最乾淨，能真正做到週期性。
- Three.js 可用 `RepeatWrapping` 和 `texture.offset` 做低成本平移。

缺點：
- 第一版維護成本較高。
- 容易走向 tech-demo 視覺，偏離溫柔 shrine landing。

### CSS background-repeat + background-position animation

最傳統的無縫水平背景做法：tileable image + repeat-x + position。

優點：
- 寫法簡單、性能好、可 progressive enhancement。
- 適合 Jcompany v1 的霧氣 strips。

缺點：
- 前提是圖片本身 tileable。
- `background-size` 不要隨 viewport 任意拉伸到破壞 tile 對齊。

## E. 避免接縫的具體做法

### 視覺 / 美術規則

- 左右邊界的平均色、亮度、霧密度要接近。
- 強辨識物件不要放 seam：太陽、清楚雲朵、最大 ribbon、明顯光束都要避開。
- 霧氣邊界不要用硬 alpha；使用 feathered mask。
- 最底層天空不要跟著 world 移動；用固定 viewport gradient 避免背景色塊 seam。
- 用多層低對比霧藏 seam，而不是一層高對比雲。

### DOM / layout 規則

- `worldWidth` 固定，例如 `max(4096px, 4 * viewportWidth)`，並用同一套 modulo 邏輯。
- 每個會 wrap 的物件都以 world coordinate 存在：`x ∈ [0, worldWidth)`。
- render 時若物件接近邊界，畫 duplicate：
  - `x < viewportWidth` 時，同時畫 `x + worldWidth`
  - `x > worldWidth - viewportWidth` 時，同時畫 `x - worldWidth`
- seam zone 內不要放唯一性物件；若必須放，就讓它跨邊界有副本。
- transform world 時用 `translate3d`，但不要把固定 sun / base sky 放進這個 world。

### Layer / animation 規則

- 每層 loop width 不同，例如 1536、2048、3072 px。
- 每層速度不同，但要很慢：霧氣 0.02-0.12x camera speed；ribbon 0.6-1.0x。
- 在 seam 附近疊一層固定 screen haze 或 vignette，讓邊界過渡更軟。
- 可用 `mask-image: linear-gradient(...)` 或 pseudo-element 做局部 feather。MDN 說 `mask-image` 可用 CSS gradient 作為遮罩；但要注意舊裝置支援，必要時用 overlay gradient fallback。

### Noise 規則

- 用週期性 / tileable noise，不要拿普通 Perlin noise 直接裁一段。
- SVG `feTurbulence` 可用 `stitchTiles="stitch"` 幫 Perlin noise 在 tile border 銜接。
- Shader noise 可以把 x 映射到圓周座標，Red Blob Games 在 wraparound map 說明了 east-west edge 對應到 cylinder 的做法；這個概念可套到水平霧氣。
- Ronja 的 tiling noise 教學也把核心問題講得很清楚：一般 noise 無限延展，但 baked texture 需要在固定距離後重複。

## F. AI / Photoshop 製作超大背景或 tileable strip SOP

### 推薦輸出規格

第一版不要先做 8:1 巨圖扛全部。建議分成：

- `sky-base`: CSS gradient，不用圖片。
- `fog-far`: 1536x768 或 2048x1024，透明 WebP / AVIF。
- `fog-mid`: 2048x1024，透明 WebP / AVIF。
- `haze-near`: 1536x768，透明 WebP / AVIF。
- optional `panorama-far`: 3072x1024 或 4096x1280，低細節、低對比，只做遠景氛圍。

如果真的要 panorama：

- 4:1 比 8:1 更務實；8:1 容易大且手機浪費。
- 左右 15-20% 留作 seam repair zone。
- 不要讓太陽或最亮光源跨左右邊界。
- 生成時就要求「horizontal seamless panorama / tileable left and right edges」，但不要相信生成結果，必須 Offset 測試。

### Photoshop SOP

1. 建立目標尺寸，例如 2048x1024 fog strip。
2. 讓圖層保持可編輯：背景色、霧、光、grain 分層。
3. `Filter > Other > Offset`，Horizontal 設寬度一半，例如 1024；Vertical 設 0；Undefined Areas 用 `Wrap Around`。Adobe 文件說 Offset filter 的 Wrap Around 會用 opposite edge 的內容填補未定義區域，這正是把左右 seam 移到畫面中央的方法。
4. 中央 seam 出現後，用 soft brush、clone stamp、healing、Content-Aware Fill、Generative Fill 修掉 seam。
5. 修 seam 時選取範圍要 feather，且選區稍微延伸進要複製的區域；Adobe Content-Aware Fill 文件也建議選區延伸到要取樣/複製的區域，效果更自然。
6. 再 Offset 回來，檢查左右邊緣。
7. 建立 3x1 tile test document，把同一張圖橫向排三份；縮放到手機寬度、桌面寬度都看一次。
8. 壓縮成 AVIF / WebP，保留 PNG master。web.dev 建議用 WebP / AVIF 降低下載量，並用 `<picture>` 或 build pipeline fallback。

### AI outpainting / inpainting SOP

1. 先生成一張符合 Jcompany 氛圍的寬圖，但 prompt 避免具象雲朵和強物件：
   - "soft cream gold and pale pink atmospheric mist, gentle shrine-like light, abstract fog layers, no hard horizon, no text, no buildings, no sci-fi"
2. 把左 20% 複製到右邊參考，或用 Offset 把 seam 移到中央。
3. 對中央 seam 做 inpainting / Generative Fill，prompt 保持簡短：
   - "soft continuous mist and warm diffuse light, seamless transition, no objects"
4. 回到 tile test，不要只看單張。
5. 若出現 AI 新增具象物件，刪掉重修；Jcompany 背景應該讓 ribbon 是唯一明顯語彙。

## G. 如果繼續用 CSS/SVG/DOM，建議新架構

### 1. 把 background world 和 object world 分開

不要讓太陽、底色、霧、ribbon 全部同一個 `transform: translateX(...)`。改成：

- `AtmosphereBackdrop`: viewport-fixed，包含 sky gradient、sun glow、vignette。
- `LoopingFogLayer`: repeat-x texture layers，使用 camera value 做 offset。
- `RibbonWorld`: 只管理 ribbon / shrine objects / near decorative elements，做 modulo 和 duplicate。

### 2. 霧氣做成 tileable strip，而不是 DOM blob 集合

霧氣如果是很多大 SVG blobs，接縫處要讓每個 blob 左右相接會很痛苦。更穩的做法是把霧氣烘成 strip：

```css
.fogFar {
  background-image: url("/fog/far.webp");
  background-repeat: repeat-x;
  background-size: 2048px 100%;
  background-position-x: calc(var(--camera-x) * -0.06);
}
```

如果要用 SVG：

- 用 `<pattern>` 或 `feTurbulence stitchTiles="stitch"` 做小面積 overlay。
- 不要把大型低頻雲霧全壓在 SVG filter。

### 3. 世界寬度 4x viewport 的自然連接方式

如果仍要 4x viewport world：

- world 本身視為圓環：所有 `x` 都 modulo `worldWidth`。
- 左右邊界不是特殊終點，而是同一空間的相鄰點。
- 光源和大型背景不要放進 world；world 只放可 duplicate 的物件。
- 任何跨 seam 的元素都要畫兩份。
- seam test mode：開發時加一個 slider 或鍵盤快捷鍵，把 camera 直接跳到 `0`, `worldWidth - viewportWidth`, `worldWidth - 200`。

## H. WebGL / Three.js 輕量做法

如果第二版想升級 Three.js，建議只做這些：

- 一個 full-screen orthographic scene。
- 2-4 個 transparent planes，貼 tileable fog textures。
- texture 設定 `wrapS = THREE.RepeatWrapping`，`wrapT` 視素材決定。
- 每幀只改 `texture.offset.x`，不要移動大量 mesh。
- 可用 `MirroredRepeatWrapping` 做某些抽象層，但要注意鏡像感。
- 不做 volumetric fog、不做大量 particles、不做 raymarching。

Three.js manual 說 texture 預設不 repeat，需要設定 `wrapS` / `wrapT`；也提醒 texture 記憶體成本是未壓縮像素尺寸，不是下載檔案大小。所以不要因為 AVIF 很小就載入 8192x4096 大圖，GPU 記憶體仍然會爆。

## I. 可直接給 Codex 的實作 prompt

```text
請重構 Jcompany 首頁背景的水平循環架構，目標是消除霧氣 / 光源 / 背景色塊 seam，保持手機和普通筆電穩定。

請先閱讀現有首頁背景實作，不要大改 unrelated UI。新的架構請採用：

1. 把背景拆成三類：
   - viewport-fixed sky/light layer：奶油金、淡粉、柔白 gradient / radial glow，不跟 camera/world 位移。
   - repeat-x fog texture layers：2-3 層 tileable fog strips，用 CSS background-repeat: repeat-x 和 background-position-x / CSS variable 跟 camera 輕微位移。
   - ribbon/object world：只放 ribbons / 近景物件，用 logical worldWidth 和 modulo wrap。

2. 不要再讓太陽、大面積底色、霧氣色塊跟 4x viewport world 一起移動。

3. 實作 wrap-around duplicate：
   - 每個 world object 有 x in [0, worldWidth)。
   - render 時若 object 接近左邊界，也 render x + worldWidth 的 duplicate。
   - 若接近右邊界，也 render x - worldWidth 的 duplicate。

4. 霧氣層先用 placeholder assets 或 generated CSS/SVG data URI，但 API 要設計成可替換成 /assets/fog-far.webp, /assets/fog-mid.webp, /assets/haze-near.webp。

5. 每層 fog 使用不同 tile size、opacity、blur、parallax factor，例如 1536/2048/3072px，避免 seam 同時疊在一起。

6. 加 dev-only seam test controls 或 constants，可以把 camera 跳到 0、worldWidth - viewportWidth、worldWidth - 200 檢查接縫。

7. 加 reduced-motion / mobile fallback：
   - prefers-reduced-motion 時停止自動 drift。
   - mobile 減少 blur 和 fog layer 數量。

8. 驗收：
   - desktop 和 mobile viewport 截圖。
   - camera 在 loop seam 附近不應看到背景色塊、霧氣、光源硬接縫。
   - 不引入 heavy WebGL / particles。
```

## J. 實作 checklist

- [ ] 把 sky base 改成 fixed viewport gradient。
- [ ] 把 sun glow 從 world 中移出，改成 fixed radial light。
- [ ] 準備 2-3 張 tileable fog strip；先可用臨時低對比 procedural asset。
- [ ] fog layers 使用 `repeat-x`，不要複製三份 DOM world 來補霧。
- [ ] ribbon world 加 modulo + duplicate render。
- [ ] seam zone 禁止放強辨識 placeholder。
- [ ] 加 seam test mode。
- [ ] 測 `prefers-reduced-motion`。
- [ ] 測 iPhone 寬度、普通 laptop 寬度、wide desktop。
- [ ] 壓縮素材成 AVIF / WebP，保留 source master。

## K. 有用資料與來源

- [MDN: background-repeat](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-repeat) - CSS 水平 repeat-x / repeat 規則。重點：repeat 只負責重複，素材不 seamless 就會重複 seam。
- [MDN: mask-image](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/mask-image) - 可用 gradient / image 做遮罩與 feather，但舊裝置支援要測。
- [MDN: SVG stitchTiles](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Attribute/stitchTiles) - `feTurbulence` 的 Perlin noise tile 邊界銜接。
- [Three.js manual: Textures](https://threejs.org/manual/en/textures.html) - `RepeatWrapping`、`texture.offset`、texture memory 成本。
- [web.dev: Image performance](https://web.dev/learn/performance/image-performance) - WebP / AVIF、`<picture>` fallback、壓縮策略。
- [web.dev: Improving HTML5 Canvas performance](https://web.dev/articles/canvas-performance) - offscreen canvas 預渲染，適合霧氣 texture。
- [MDN: requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame) - render loop 與背景 tab 暫停行為。
- [MDN: Optimizing canvas](https://developer.mozilla.org/docs/Web/API/Canvas_API/Tutorial/Optimizing_canvas) - Canvas 效能優化。
- [Red Blob Games: Making maps with noise functions](https://www.redblobgames.com/maps/terrain-from-noise/) - wraparound noise / cylinder mapping 概念。
- [Ronja's Tutorials: Tiling Noise](https://www.ronja-tutorials.com/post/029-tiling-noise/) - tileable / periodic noise 實作概念。
- [Adobe: Apply specific filters in Photoshop](https://helpx.adobe.com/ph_fil/photoshop/using/applying-specific-filters.html) - Offset filter 的 Wrap Around 行為。
- [Adobe: Content-Aware Fill](https://helpx.adobe.com/photoshop/desktop/apply-painting-techniques/fill-objects-selections-layers/content-aware-fills.html) - Content-Aware Fill、Pattern Fill、選區延伸與自然修補。

## 我的取捨

Jcompany 的背景不是要炫技，而是要讓願望、ribbon、祈願感站在前面。第一版應該把工程風險壓低：固定底光 + tileable fog + ribbon wrap。等視覺和互動方向穩了，再考慮 Canvas 或 Three.js 做更自然的霧氣微動。

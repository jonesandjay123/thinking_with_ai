# Trip Planner 地圖 API 選型研究（2026-04-24）

## 結論先講

如果目標是替 `trip-planner` 加上「同一天行程在小地圖上顯示景點分布」這個功能，我建議分兩階段：

1. **MVP：Leaflet + OpenStreetMap tile**
   - 不需要 API key。
   - 不需要帳單。
   - 可以直接顯示 marker、fit bounds、popup、簡單連線。
   - 適合現在的私人/小圈子旅行工具。

2. **下一階段：Geoapify 或 MapTiler**
   - 如果需要「景點搜尋 → 自動抓經緯度」、「更穩定 tile quota」、「未來公開多人使用」，再接商用 location API。
   - Geoapify 比較適合把 map tiles、geocoding、routing 一包處理。
   - MapTiler 比較適合漂亮、穩定、好控的地圖底圖。

不建議第一版就用 Google Maps 或 Mapbox，除非我們確定要走產品化、願意處理 billing / API key / quota / 地圖供應商鎖定。

---

## 需求拆解

Jones 想要的功能可以拆成三類：

### 1. 地圖顯示 / Map rendering

在 modal 或 day column 裡顯示互動地圖。

需要：
- map library
- map tile provider
- marker / popup
- fit bounds
- mobile-friendly UI

### 2. 景點定位 / Geocoding

把卡片上的景點名稱或地址轉成經緯度。

可以先手動填：

```js
location: {
  lat: 35.7148,
  lng: 139.7967,
  address: "2 Chome-3-1 Asakusa, Taito City, Tokyo",
  source: "manual"
}
```

之後再做自動搜尋 / geocoding。

### 3. 路線 / Routing

把同一天多個點依排程順序連線，甚至算步行 / 大眾交通 / 開車時間。

這是進階功能。第一版可以只做：
- marker 編號
- 直線 polyline
- 不算真實路線

---

## 推薦方案

## Option A — Leaflet + OpenStreetMap tile（推薦 MVP）

### 適合用途

- 顯示當天所有景點 marker。
- marker popup 顯示 card title / time zone / area。
- 自動縮放到所有 marker。
- 照行程順序標號。
- 用直線連起來看分布。

### 技術

```bash
npm install leaflet react-leaflet
```

React component roughly：

```jsx
<MapContainer bounds={bounds}>
  <TileLayer
    attribution='&copy; OpenStreetMap contributors'
    url="https://tile.openstreetmap.org/{z}/{x}/{y}.png"
  />
  {cardsWithLocation.map((card, index) => (
    <Marker key={card.id} position={[card.location.lat, card.location.lng]}>
      <Popup>{index + 1}. {card.title}</Popup>
    </Marker>
  ))}
</MapContainer>
```

### 成本 / 免費額度

- Leaflet 本身是 open-source JS library。
- OpenStreetMap data 是 open data，但 `tile.openstreetmap.org` tile server 不是無限商用 CDN。
- OSM tile usage policy 要求：
  - 顯示 attribution。
  - 使用正確 tile URL。
  - 不 bulk download / scrape tiles。
  - 不做 prefetch。
  - 遵守 cache headers。
  - 網頁應送出 valid Referer。

### 風險

- OSM public tile server 沒有 SLA。
- 若未來公開化、多使用者、高流量，應改用 MapTiler / Stadia / Geoapify / 自架 tile server。
- 不提供 geocoding；只是底圖。

### 評價

**最適合現在。**

原因：trip-planner 目前是私人/小圈子工具，需求主要是「看分布」而不是商業級地圖平台。MVP 不需要增加 API key 管理與 billing 風險。

---

## Option B — MapLibre GL JS + provider tiles

### 適合用途

- 比 Leaflet 更現代的 vector map rendering。
- 未來想要漂亮 style、GPU 加速、vector tiles、pitch/rotate/3D-ish map。

### 成本 / 免費額度

- MapLibre GL JS 是 open-source library。
- 仍然需要 tile provider：MapTiler、Stadia Maps、Mapbox-compatible tiles、自架 tiles 等。

### 優點

- 現代 vector tile experience。
- 比 Leaflet 更適合未來產品化與 custom style。
- 不被 Mapbox GL JS license 綁住。

### 缺點

- 第一版複雜度比 Leaflet 高。
- 如果只是 marker + popup，Leaflet 更省。

### 評價

適合第二階段，不是今天下午 MVP 的最佳解。

---

## Option C — MapTiler Cloud

### 適合用途

- 穩定 tile provider。
- 好看的底圖 style。
- 可做 geocoding / static maps / maps API。
- 搭配 Leaflet 或 MapLibre 都可以。

### 免費額度（官方頁面 2026-04-24 查詢）

Free plan：
- $0
- non-commercial / personal use
- 5,000 API sessions / month
- 100,000 API requests / month
- Vector & XYZ raster tiles
- 需要 MapTiler logo / attribution

Flex plan：
- $25/month
- commercial use
- 25,000 API sessions / month
- 500,000 API requests / month
- extra requests $0.10 / 1,000

### 優點

- 免費額度對私人 trip-planner 很夠。
- 文件清楚。
- 可用公開前端 key，但要保護 key / domain restriction。
- 地圖樣式比 OSM default tile 好看。

### 缺點

- Free plan 是 personal / non-commercial。未來公開商用要升級。
- 需要 API key。
- 仍要看 session/request 計費模型。

### 評價

如果想讓地圖更穩、更漂亮，我會把 MapTiler 視為 **MVP 後第一個升級選項**。

---

## Option D — Geoapify Location Platform

### 適合用途

- 希望一站處理：maps、geocoding、places、routing、isoline。
- 想要比較慷慨的免費額度。
- 未來需要「輸入景點名稱 → 自動抓 coordinates」。

### 免費額度（官方頁面 2026-04-24 查詢）

Free plan / pricing details：
- No credit card required。
- Limited commercial use。
- Up to 5 requests/second。
- 3,000 credits/day，約 90,000 credits/month。
- Simple API requests such as geocoding、places、routing 通常 1 request = 1 credit。
- Map tile：1 tile request = 0.25 credits；官方估 interactive map session 約 50 tiles。

Paid plans from pricing page：
- $59/month 起，commercial use，higher rate limits。

### 優點

- 免費額度很適合小型 prototype。
- geocoding + routing + map tiles 都有。
- 商業使用限制比純 OSM public endpoint 更清楚。

### 缺點

- 需要 API key。
- 免費方案是否足夠取決於 map tile usage；如果每次打開地圖都消耗 tiles，要控制載入頻率。
- 需要遵守 Geoapify attribution。

### 評價

**如果要自動 geocoding，我會優先考慮 Geoapify。**

對 trip-planner 很實際：
- card modal 裡提供「搜尋地址 / 景點」按鈕。
- 後端 callable function 呼叫 Geoapify Geocoding API。
- 把結果寫入 `card.location`。

---

## Option E — Stadia Maps

### 適合用途

- 需要 OSM-derived tiles + geocoding/routing APIs。
- 想要 domain-based authentication，避免前端暴露 API key。

### 免費額度（官方頁面 2026-04-24 查詢）

Free plan：
- $0/month
- 200,000 credits/month
- Standard basemaps
- Basic APIs only
- Commercial use not allowed

Starter：
- $20/month
- 1,000,000 credits/month
- commercial use allowed
- standard basemaps + static maps + geocoding APIs

Authentication：
- localhost local dev can work without auth, subject to strict limits。
- production websites 建議 domain-based authentication。

### 優點

- Free plan 額度看起來大。
- Domain-based auth 對 browser app 友善。
- 有 routing / geocoding / map APIs。

### 缺點

- Free plan 不允許 commercial use。
- Credit schedule 要仔細看，不同 API 消耗不同。
- 對我們而言，比 Geoapify / MapTiler 稍微不直覺。

### 評價

可以列為備選。若很重視「不在前端放 API key」，Stadia 值得再深入。

---

## Option F — LocationIQ

### 適合用途

- Geocoding / routing / street & static maps。
- 免費方案相對簡單。

### 免費額度（官方頁面 2026-04-24 查詢）

Free plan：
- $0
- 5,000 requests/day
- 2 requests/second
- 60 requests/minute
- Geocoding APIs、Routing APIs、Street & Static Maps
- Limited commercial use，需要顯著 link attribution：`Search by LocationIQ.com`

Paid plans：
- Maps Lite $45/month：10,000 map views/day
- Developer $100/month：25,000 requests/day

### 優點

- 免費 geocoding quota 對 prototype 很夠。
- 文件和 pricing 比較容易理解。

### 缺點

- 地圖 rendering / tiles 跟 UI 整合需要看細節。
- 品牌/attribution 要求比較明顯。
- 對 trip-planner 不是最自然的一站式選擇。

### 評價

適合做 geocoding fallback；不是首選 map rendering provider。

---

## Option G — Mapbox

### 適合用途

- 產品化地圖體驗。
- 高品質 vector map style。
- Directions / routing / search / geocoding 都成熟。
- 未來若要做漂亮互動 map 或商業化產品，可考慮。

### 免費額度 / pricing 摘要（官方頁面 2026-04-24 查詢）

Pricing page 中可見：
- Directions API：up to 100,000 monthly requests free tier（頁面列出 0-100,000）。
- Navigation SDK 有 MAU / trip-based free tier。
- 其他 Maps/Search/Geocoding 需依 SKU 查表；pricing page 很長且分產品計價。

Docs：
- Geocoding API supports forward/reverse geocoding。
- Directions API supports driving、driving-traffic、walking、cycling，up to 25 coordinates per request。

### 優點

- 成熟、漂亮、文件完整。
- Geocoding / Directions 品質好。
- Routing profiles 適合旅行規劃。

### 缺點

- 需要 access token。
- Pricing 複雜，需要開 billing、設 quota/alerts。
- 對目前私人 app 有點重。
- Vendor lock-in 較明顯。

### 評價

不建議第一版用。等 trip-planner 要走公開產品化、有 traffic、有 billing 管理，再考慮。

---

## Option H — Google Maps Platform

### 適合用途

- 最強 Places / POI / 地址資料。
- 日本景點與商家資料通常品質很好。
- 若未來要做「搜尋店名」、「營業時間」、「評分」、「Google Maps 深連結」，Google 很強。

### 免費額度 / pricing 摘要（官方頁面 2026-04-24 查詢）

Google Maps Platform pricing page 顯示：
- Dynamic Maps：10,000 monthly free events，之後約 $7 / 1,000 events 起。
- Static Maps：10,000 monthly free events，之後約 $2 / 1,000 events 起。
- Map Tiles API 2D：100,000 monthly free events，之後約 $0.60 / 1,000 events 起。
- Routes Essentials：10,000 monthly free events，之後約 $5 / 1,000 events 起。
- Geocoding API page 顯示 pay-as-you-go、需 billing/API key/OAuth，並有 3,000 queries per minute limit。

注意：Google 舊的每月 $200 credit 文件已標示到 2025-02-28；現在應以新的 SKU free usage cap 為準。

### 優點

- Places / geocoding / POI 品質最佳之一。
- 日本商家資料、地址、營業時間很強。
- 使用者熟悉 Google Maps。

### 缺點

- 需要 billing。
- 成本與 SKU 複雜。
- API key 保護、quota、意外帳單都要處理。
- Google Maps Platform 服務條款通常對資料快取、混用地圖底圖有更多限制；要仔細遵守。

### 評價

對「旅行規劃產品」長期很有價值，但不適合作為第一個 marker MVP。比較適合第二或第三階段接 Places / opening hours / Google Maps link。

---

## Nominatim / OSM Geocoding 注意

OSM 的 Nominatim public endpoint 可做 geocoding，但不適合作為 app 內大量或自動化 geocoding 服務。

官方 usage policy 重點：
- 絕對最大 1 request/second。
- 必須提供 valid Referer 或 User-Agent。
- 必須 clear attribution。
- Bulk geocoding discouraged，結果要 cache。
- 禁止 autocomplete search。
- 服務容量有限，政策可能變更，可能撤回存取。

結論：
- 可以手動少量查一兩個點。
- 不應直接放進公開 client-side app 做 autocomplete。
- 若要 geocoding，應用 Geoapify / LocationIQ / MapTiler / Google / Mapbox，或 server-side proxy + cache。

---

## 對 trip-planner 的建議架構

## MVP v1：手動 location + day map modal

### Data model

在 card object 加 optional location：

```js
card = {
  id,
  title,
  subtitle,
  zone,
  duration,
  area,
  note,
  tags,
  comments,
  location: {
    lat: 35.7148,
    lng: 139.7967,
    address: "2 Chome-3-1 Asakusa, Taito City, Tokyo",
    placeName: "Senso-ji",
    source: "manual",
    updatedAt: "2026-04-24T...Z"
  }
}
```

### UI

- `CardModal` 新增：
  - latitude
  - longitude
  - address / place name
- `DayColumn` header 新增：
  - `🗺️` 或 `地圖` 按鈕
- 新增 `DayMapModal.jsx`：
  - 顯示當天所有 scheduled cards with location。
  - marker 用行程順序編號。
  - popup 顯示 title、zone、area、note。
  - 沒 location 的卡列在底部。
- 手機版：
  - Day pager 當天標題旁放小 `🗺️` icon。
  - map modal 高度用 `70vh` 或 fullscreen bottom sheet。

### API / dependency

- `leaflet`
- `react-leaflet`
- 不接 geocoding API。
- tile 先用 OSM public tile，但保留 env config：

```js
VITE_MAP_TILE_URL=https://tile.openstreetmap.org/{z}/{x}/{y}.png
VITE_MAP_ATTRIBUTION=&copy; OpenStreetMap contributors
```

### 工期估計

- 45–90 分鐘：MVP 能 build。
- 2–3 小時：加上好看的 marker number、mobile modal、README、幾張東京景點手動座標。

---

## MVP v1.5：半自動 geocoding（建議 Geoapify）

### Flow

1. 使用者在 CardModal 裡按 `搜尋座標`。
2. 前端呼叫 Firebase callable function：`geocodeCardLocation`。
3. Function 使用 Geoapify / MapTiler / LocationIQ API key 查詢。
4. 回傳候選結果讓使用者選。
5. 寫入 card.location。

### 為什麼 server-side function？

- API key 不暴露在前端。
- 可以 cache query → coordinates。
- 可以 rate limit。
- 可以統一 attribution / source。
- 未來可換 provider。

### Cache model

```text
geocodingCache/{normalizedQuery}
```

```js
{
  query: "Sensoji Tokyo",
  provider: "geoapify",
  results: [...],
  createdAt,
  updatedAt
}
```

---

## MVP v2：路線與順序檢查

先不要做真正 directions API。可以先做：

- 依 active plan 當天順序畫 marker number。
- 用直線 polyline 連接。
- 計算 haversine 直線距離。
- 如果相鄰兩點距離超過某門檻，提醒：
  - `⚠️ 這兩站距離較遠，可能需要重新排序。`

等這個有用，再接 routing API。

### Routing provider 建議

- 私人 prototype：Geoapify Routing 或 LocationIQ Routing。
- 更成熟產品化：Mapbox Directions。
- Google Routes：若要最高品質與 Google ecosystem，但 billing 複雜。

---

## Provider 比較表

| Provider | 第一版地圖顯示 | Geocoding | Routing | 免費/低成本 | API key | 適合 trip-planner 嗎 |
|---|---:|---:|---:|---:|---:|---|
| Leaflet + OSM tile | ✅ | ❌ | ❌ | ✅ | ❌ | **最適合 MVP** |
| MapLibre + provider | ✅ | depends | depends | ✅/depends | depends | 適合後續產品化 |
| MapTiler | ✅ | ✅ | limited/depends | ✅ Free 5k sessions / 100k requests | ✅ | 很適合漂亮穩定底圖 |
| Geoapify | ✅ | ✅ | ✅ | ✅ 3k credits/day | ✅ | **最適合 geocoding MVP** |
| Stadia Maps | ✅ | ✅ | ✅ | ✅ 200k credits/month, non-commercial | optional/domain | 備選 |
| LocationIQ | ✅ | ✅ | ✅ | ✅ 5k requests/day | ✅ | Geocoding fallback |
| Mapbox | ✅ | ✅ | ✅ | ✅ but pricing complex | ✅ | 產品化後考慮 |
| Google Maps | ✅ | ✅✅ | ✅ | 有 SKU free caps，但 billing 複雜 | ✅ | Places/POI 最強，後期考慮 |

---

## 最終建議

### 今天如果要做功能

做 **Leaflet MVP**：

1. Install `leaflet react-leaflet`。
2. Card schema 加 `location` optional 欄位。
3. CardModal 加 lat/lng/address 欄位。
4. DayColumn 加 `🗺️` button。
5. DayMapModal 顯示 marker / numbered popup / missing-location list。
6. README 記錄 future provider strategy。

這版不需要 API key，風險最低，而且可以立刻驗證「地圖對旅行規劃是不是真的有用」。

### 下一步再做

如果 Jones 覺得 map modal 有用，再做 **Geoapify geocoding**：

- Firebase callable function。
- Server-side API key。
- cache。
- card modal search UX。

### 不建議現在做

- 不要第一版就接 Google Maps。
- 不要直接用 Nominatim 做 autocomplete。
- 不要一開始就做真實 routing / route optimization。

---

## Sources checked

- Leaflet official site — https://leafletjs.com/
- OpenStreetMap copyright / attribution — https://www.openstreetmap.org/copyright
- OpenStreetMap tile usage policy — https://operations.osmfoundation.org/policies/tiles/
- OpenStreetMap Nominatim policy — https://operations.osmfoundation.org/policies/nominatim/
- MapLibre — https://maplibre.org/
- MapTiler API docs — https://docs.maptiler.com/cloud/api/
- MapTiler pricing — https://www.maptiler.com/cloud/pricing/
- Geoapify pricing — https://www.geoapify.com/pricing/
- Geoapify pricing details — https://www.geoapify.com/pricing-details/
- Stadia Maps authentication — https://docs.stadiamaps.com/authentication/
- Stadia Maps pricing — https://stadiamaps.com/pricing/
- LocationIQ pricing — https://locationiq.com/pricing
- Mapbox pricing — https://www.mapbox.com/pricing
- Mapbox Geocoding API docs — https://docs.mapbox.com/api/search/geocoding/
- Mapbox Directions API docs — https://docs.mapbox.com/api/navigation/directions/
- Google Maps Platform pricing — https://developers.google.com/maps/billing-and-pricing/pricing
- Google Geocoding billing docs — https://developers.google.com/maps/documentation/geocoding/usage-and-billing

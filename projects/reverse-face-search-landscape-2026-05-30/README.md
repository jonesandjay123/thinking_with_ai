# 以照片找人的網站與開源替代研究（2026-05-30）

> 研究問題：PimEyes、Social Catfish、Lenso.ai、FaceCheck.ID 這類「透過照片找人」服務以前有較多免費入口，現在多半變成付費或 freemium。坊間是否還有同類型網站，或可自架/開源 repo 可用？

## 快速結論

1. **真正能「全網找同一張臉」的服務，幾乎都已商業化。**  
   免費入口通常只給預覽、少量 credits、或不給來源連結；要看完整 source link、深度掃描、持續監控，大多需要付費。

2. **一般反向圖片搜尋仍免費，但不是臉部辨識。**  
   Google Lens、TinEye、Yandex Images、Bing Visual Search 這類工具適合找「同一張圖或相似圖片」；遇到裁切、濾鏡、不同自拍照、不同角度時，命中率通常不如專門的 face search。

3. **開源 repo 可以做「自己的資料庫裡找臉」，不能直接取代 PimEyes。**  
   CompreFace、DeepFace、InsightFace、face_recognition、FAISS 等工具可以建立 face embedding、比對相似臉、做 1:N search；但它們沒有 PimEyes / FaceCheck 那種已爬好的公開網路人臉索引。

4. **如果要自架，真正難的不是模型，而是資料來源與合規。**  
   技術上可以做：爬圖/匯入資料 → 偵測人臉 → 產生 embedding → 存進向量資料庫 → 查詢相似臉。難點在公開網路爬取、平台 ToS、個資/生物特徵法規、誤判風險、以及被用於騷擾或 doxxing 的風險。

5. **實務建議：免費工具先做粗查；真的要查臉，選一次性付費比訂閱安全。**  
   若只是防詐/查盜圖，先跑 Google Lens + TinEye + Yandex + Search4Faces；若仍需要 face-level matching，再考慮 FaceCheck、Lenso、ProFaceFinder、CatfishLens 這類付費或 freemium 工具。

## 服務版圖

### A. 專門的 reverse face search / people-by-photo 服務

| 服務 | 現況 | 費用/限制 | 評估 |
|---|---|---|---|
| [PimEyes](https://pimeyes.com/) | 老牌臉部搜尋，偏「公開網頁上的人臉」 | 多數實用功能需付費 | 強，但隱私爭議高；適合查自己照片是否外流，不建議拿來追蹤他人 |
| [Lenso.ai](https://lenso.ai/en) | AI reverse image search，含 People / Face Search、duplicates、places | 官方標示可免費試用；Starter/Professional/Developer 訂閱解鎖結果、Research Mode、API | 目前仍是較成熟的 PimEyes 類替代；Developer plan 提供 Face Search API |
| [FaceCheck.ID](https://facecheck.id/) | 臉部搜尋 + 高風險來源提示 | 1 search = 3 credits；信用包從 $19 crypto 起，完整連結需 credits | 入口相對直接，但 crypto 付款、refund 條款與資料來源透明度要小心 |
| [Social Catfish](https://socialcatfish.com/) | 身分查核 / reverse image / data broker 型服務 | 偏訂閱/付費解鎖 | 比較像「人肉資料聚合」而不只是圖片搜尋；口碑兩極 |
| [CatfishLens](https://catfishlens.com/) | Catfish / dating safety 導向的 reverse face search | 2 credits 約 $6.95、5 credits 約 $9.95；credit 不需訂閱 | 一次性查詢友善；可信度仍需實測 |
| [ProFaceFinder](https://profacefinder.com/pricing/) | Face search / social & dating source links | 一次性：$7.95/2 searches、$11.95/7 searches | 價格比訂閱型友善；適合偶爾查 |
| [FaceSearch.net](https://facesearch.net/) | 宣稱免費起步、跨社群/約會網站搜尋 | 免費方案 + credit packs / $29 monthly Pro | 新服務，宣稱很強但需保守看待；可作候選但要實測 |
| [Face Finder](https://www.facefinder.id/en) | 宣稱 50M+ indexed faces、免費掃描、付費看完整結果 | 官方 FAQ 說完整來源需一次性付費，起價 $5.99 | 類似新興 freemium；資料來源與準確率需驗證 |
| [Search4Faces](https://search4faces.com/) | 俄文系統，資料庫偏 VK、OK、TikTok、Clubhouse、名人 | 可搜尋特定資料庫 | 對俄語圈/東歐社群較有用；不一定適合美國/台灣/日本場景 |

### B. 一般反向圖片搜尋（免費優先）

| 工具 | 強項 | 弱點 |
|---|---|---|
| [Google Lens](https://support.google.com/websearch/answer/1325808) | 免費、入口最方便、可找相似圖片/物件/網站 | 對「同一個人但不同照片」不穩；近年更偏商品/物件理解 |
| [TinEye](https://tineye.com/) | 老牌 reverse image search；官方表示搜尋私密、不保存上傳圖，索引超過 70B images | 偏 exact / near-duplicate；臉部辨識不是主軸 |
| [Yandex Images](https://browser.yandex.kz/help/en/search-and-browse/search-image) | 相似圖片搜尋常有不同結果池 | 地域/語言偏差，穩定性與隱私考量需自評 |
| Bing Visual Search | 免費、可補 Google/TinEye 沒找到的結果 | 臉部搜尋能力不是主軸 |

## 開源與自架方案

### 可以直接研究的 repo / 工具

| Repo / 工具 | 用途 | 授權/狀態 | 適合場景 |
|---|---|---|---|
| [exadel-inc/CompreFace](https://github.com/exadel-inc/CompreFace) | Docker 化、REST API 形式的人臉辨識系統 | Apache 2.0；約 8k stars；最近更新偏少（2024-10） | 想快速自架 API、做已知人臉庫辨識 |
| [serengil/deepface](https://github.com/serengil/deepface) | Python face recognition / verification / attribute analysis，封裝多種模型 | MIT；約 22.8k stars；仍活躍 | 快速 prototype、比對兩張臉、做 embedding |
| [deepinsight/insightface](https://github.com/deepinsight/insightface) | SOTA 2D/3D face analysis、ArcFace 等 | code MIT，但官方模型/訓練資料常見非商用限制；約 28.8k stars | 技術力最強的一類；商用需特別看模型授權 |
| [ageitgey/face_recognition](https://github.com/ageitgey/face_recognition) | dlib-based Python/CLI，簡單易用 | MIT；約 56k stars；近年較少更新 | 小型本地照片庫、demo、教育用途 |
| [facebookresearch/faiss](https://github.com/facebookresearch/faiss) | 向量相似搜尋引擎 | MIT；約 40k stars；活躍 | 把 face embeddings 做成可查詢索引 |
| [cocoindex-io/cocoindex](https://github.com/cocoindex-io/cocoindex) + [Photo Search Faces example](https://cocoindex.io/docs-v0/examples/photo_search) | pipeline：偵測/裁臉/embedding/向量庫 | Apache 2.0；約 10k stars；活躍 | 想做「Google Photos 風格」自己的照片庫搜尋 |
| [yakhyo/face-reidentification](https://github.com/yakhyo/face-reidentification) | SCRFD + ArcFace + FAISS + ONNX Runtime | 授權需確認；小型 repo | 可讀作 face search reference implementation |
| [WISE](https://gitlab.com/vgg/wise/wise) | Oxford VGG 的開源 multimodal search engine，支援 face-based search | 開源；2026 paper | 適合大型影音/圖片 archive 的搜尋，不是全網找人服務 |

### 自架架構草圖

```text
資料來源（自己的照片庫 / 合法取得的公開資料）
  ↓
圖片匯入與去重
  ↓
人臉偵測（SCRFD / RetinaFace / dlib CNN）
  ↓
人臉裁切與品質過濾
  ↓
embedding 產生（ArcFace / InsightFace / DeepFace / face_recognition）
  ↓
向量索引（FAISS / pgvector / Milvus / Qdrant / Weaviate）
  ↓
查詢：上傳照片 → 產生 embedding → top-k 相似臉 → 回傳來源圖片/頁面
```

這樣可以做出「在我自己的資料庫裡找同一個人」；但要做「在整個公開網路找某個人」，還需要大型 crawler、索引更新、來源頁保存、反濫用與 opt-out 流程。這些才是商業服務的核心成本。

## 為什麼以前免費，後來集體收費？

1. **算力與索引成本高。** 臉部搜尋不是只查文字，需要持續爬圖、裁臉、算 embedding、維護向量索引，還要處理大量重複與低品質圖片。

2. **來源連結才是價值。** 很多服務可以免費給你「有結果」的預覽，但真正值錢的是 source URL、完整相似圖、持續監控、匯出報告。

3. **法律與風險成本上升。** 人臉是 biometric data；錯誤匹配可能造成誤認、騷擾、歧視、錯誤背景調查等問題。FaceCheck 自己也在條款中明示不得用於 credit、employment、insurance、tenant screening 等決策。

4. **被濫用的風險太高。** 找失聯親友、防詐、查盜圖是正當用途；但同一技術也可用於 stalking、doxxing、騷擾、追蹤性工作者或未成年者。服務商因此更傾向付費牆、帳號制、opt-out、限制搜尋量。

## 實務查找流程（低風險版）

1. **先確認用途是否正當。** 最安全的用途是查自己的照片是否被盜用、驗證疑似詐騙帳號、或在對方同意下做身分確認。
2. **先跑免費 general reverse image search。** Google Lens → TinEye → Yandex → Bing，各跑一次，因為索引池不同。
3. **如果是俄語圈/公開社群頭像，可試 Search4Faces。**
4. **若仍要 face-level search，選一次性 credit 服務。** ProFaceFinder / CatfishLens / Face Finder 這類比長期訂閱安全，避免忘記取消。
5. **不要把單一 match 當事實。** 臉部搜尋結果只能當線索；要交叉驗證來源頁、時間、帳號歷史、文字內容、互動脈絡。

## 給 Jones 的判斷

如果目標是「偶爾查疑似盜圖/詐騙」，不值得自架。直接用免費工具粗查，再買少量一次性 credits 會比較省時間。

如果目標是「做一個產品/agent 能管理自己的照片庫、影片庫、創作素材庫，搜尋某個人是否出現在自己的 archive 裡」，開源路線很可行，推薦起手式：

- 快速 prototype：`deepface` 或 `face_recognition`
- 系統化 API：`CompreFace`
- 高階模型：`InsightFace`（先確認模型授權）
- 大量索引：`FAISS` / `pgvector` / `Milvus`
- pipeline 參考：`CocoIndex Photo Search (Faces)`

真正不建議的是「自己爬全網建立陌生人人臉搜尋引擎」。技術上可以，但法律、平台 ToS、倫理與濫用風險都太重，不適合作為個人專案起點。

## 來源

- Google Search Help: [Search with an image on Google](https://support.google.com/websearch/answer/1325808)
- TinEye: [Reverse Image Search](https://tineye.com/), [How do I search on TinEye?](https://help.tineye.com/article/242-how-do-i-search-on-tineye)
- Yandex Browser Help: [Image search](https://browser.yandex.kz/help/en/search-and-browse/search-image)
- Lenso.ai: [AI Reverse Image Search with Facial Recognition](https://lenso.ai/en), [Pricing](https://lenso.ai/en/pricing)
- FaceCheck.ID: [Buy Credits](https://facecheck.id/en/buy), [Face Search API](https://facecheck.id/en/Face-Search/API)
- CatfishLens: [Home](https://catfishlens.com/), [Pricing](https://catfishlens.com/pricing/)
- ProFaceFinder: [Pricing Plans](https://profacefinder.com/pricing/)
- FaceSearch.net: [Reverse Face Search Engine](https://facesearch.net/)
- Face Finder: [Face ID Search Engine](https://www.facefinder.id/en)
- Search4Faces: [Home](https://search4faces.com/)
- GitHub: [CompreFace](https://github.com/exadel-inc/CompreFace), [DeepFace](https://github.com/serengil/deepface), [InsightFace](https://github.com/deepinsight/insightface), [face_recognition](https://github.com/ageitgey/face_recognition), [FAISS](https://github.com/facebookresearch/faiss), [CocoIndex](https://github.com/cocoindex-io/cocoindex), [face-reidentification](https://github.com/yakhyo/face-reidentification)
- CocoIndex docs: [Photo Search (Faces)](https://cocoindex.io/docs-v0/examples/photo_search)
- arXiv: [WISE: A Multimodal Search Engine for Visual Scenes, Audio, Objects, Faces, Speech, and Metadata](https://arxiv.org/abs/2602.12819)
- NIST: [Facial Recognition Technology: Ensuring Commercial Transparency & Accuracy](https://www.nist.gov/node/1605531)
- ACLU: [Police Say a Simple Warning Will Prevent Face Recognition Wrongful Arrests. That's Just Not True.](https://www.aclu.org/news/privacy-technology/police-say-a-simple-warning-will-prevent-face-recognition-wrongful-arrests-thats-just-not-true)

## 查證日期

2026-05-30 EDT。網站價格、免費額度、可用性會變動；實際付款前應重新確認官方 pricing / terms。

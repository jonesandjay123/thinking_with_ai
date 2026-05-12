# EasterEgg Vision Scout 技術調查：快速掃描影片與關鍵幀模型

> 日期：2026-05-12  
> 狀態：第一版技術調查  
> 範圍：fast video scanning、keyframe selection、shot boundary detection、frame embedding、OCR、VLM rerank、弱標註與小模型訓練  
> 目的：站在現有工具、論文與預訓練模型上，避免從零無腦搭建。

---

## 0. 結論先講

EasterEgg Vision Scout 不應該第一步就從零訓練一個「會看影片」的大模型。

更聰明的路線是：

> **先用現成 video structure / embedding / OCR / VLM stack，做出一個可解釋、可評估、可累積資料的 frame scout pipeline；等有自己的 domain dataset 後，再訓練小型 useful-frame classifier 或 ranker。**

推薦第一版架構：

```text
影片
  ↓
ffmpeg / PySceneDetect / TransNetV2 切 shot + 抽候選 frames
  ↓
去黑畫面、去模糊、去重
  ↓
OpenCLIP / SigLIP / DINOv2 產生 frame embeddings
  ↓
PaddleOCR 偵測招牌、菜單、門牌、街景文字
  ↓
FAISS / clustering 做去重與 diversity selection
  ↓
Qwen2.5-VL / LLaVA-OneVision / MiniCPM-V 只看 top-K 候選
  ↓
輸出 top frames + timestamp + score breakdown + OCR text + evidence
```

最短 MVP stack：

- `ffmpeg`
- `PySceneDetect`
- `OpenCLIP` 或 `SigLIP`
- `PaddleOCR`
- `FAISS`
- `Qwen2.5-VL` 或其他 VLM 作最後判讀

暫時不要先做：

- end-to-end video summarization model
- reinforcement learning active viewer
- 自訓大型 VLM
- SAM/GroundingDINO 全量掃所有 frames

---

## 1. EasterEgg Vision Scout 的任務本質

這個專案不是一般影片摘要，也不是完整影片理解。它更像：

> 從大量低價值 frames 中，快速找出可能包含「可用線索」的少數 frames。

對 VlogMap / 美食旅行影片來說，可用線索包含：

- 店門口
- 招牌
- 菜單
- 價格表
- 食物特寫
- 街道 / 地理線索
- 門牌 / 地址 / 電話
- 地標
- 海報 / 營業時間
- 影片中一閃而過但對定位有用的文字

因此它和傳統 video summarization 不完全一樣。

傳統摘要問的是：

> 哪些片段最能代表整支影片？

EasterEgg Vision Scout 問的是：

> 哪些 frames 最可能包含後續 OCR / VLM / Maps 驗證需要的證據？

這表示第一版可以不追求「懂整支影片」，只要能把候選 frames 篩得比 uniform sampling 更有效率，就已經有價值。

---

## 2. 第一層：影片切分與候選 frame 抽取

### 2.1 ffmpeg

來源：<https://ffmpeg.org/>

用途：

- 每秒抽 1–2 張 baseline frames
- 每 N 秒抽一張
- 用 scene change filter 抽畫面變化大的 frame

適合用途：

- 建 baseline
- 快速處理本地影片
- 不依賴重模型

優點：

- 穩定、快、跨平台
- 容易整合 pipeline
- 可以先快速建立資料集

限制：

- 不懂語意
- scene-change threshold 需要調
- 容易漏掉同一 shot 裡短暫出現的招牌 / 菜單

建議用法：

- 做 uniform sampling baseline
- 做快速候選 frames 擷取
- 後續再用 CLIP/OCR 重排

---

### 2.2 PySceneDetect

來源：<https://github.com/Breakthrough/PySceneDetect>  
文件：<https://www.scenedetect.com/docs/>

用途：

- 用 OpenCV 做 scene cut / transition detection
- 把影片切成 shots
- 每個 shot 存 representative images

官方 README 提到可以：

- `scenedetect -i video.mp4 split-video`
- `scenedetect -i video.mp4 save-images`
- Python API：`detect('my_video.mp4', ContentDetector())`

適合用途：

- 第一版 shot-based candidate extraction
- 每個 shot 抽 start / middle / end frames
- 減少連續 frames 重複

優點：

- 工具成熟
- Python / CLI 都好接
- 比 uniform sampling 更有結構
- 對 MVP 很夠用

限制：

- 基於畫面變化，對手持 vlog、慢轉場、快速 pan 可能不穩
- 不懂語意，不知道哪個 shot 有招牌

建議：

> MVP 先用 PySceneDetect。若遇到轉場偵測不穩，再加入 TransNetV2。

---

### 2.3 TransNetV2

論文：<https://arxiv.org/abs/2008.04838>  
程式碼：<https://github.com/soCzech/TransNetV2>

用途：

- 神經網路式 shot boundary detection
- 偵測 cut / gradual transition
- 官方 repo 有 pretrained model，可直接 inference

訓練方式：

- 3D convolution architecture
- 使用影片與 shot transition labels
- 也使用 synthetic transitions 增強訓練

適合用途：

- 比 PySceneDetect 更穩的 shot boundary detector
- vlog / 街景 / 手持影片中分段
- 第一層壓縮影片長度

優點：

- 有預訓練模型，不必自己訓練
- 對 shot transition 是專門模型
- 可避免每秒抽大量 frames

限制：

- 只解決「影片結構」，不解決「哪幀有用」
- 訓練資料很大，官方也提示訓練資料可能是數十到數百 GB 等級

建議：

> 若 PySceneDetect 對 YouTube 美食 / 旅行 vlog 不夠穩，第二步換 TransNetV2。不要一開始自己訓練 shot detector。

---

## 3. 第二層：frame embeddings / 語意檢索 / 去重

### 3.1 CLIP / OpenCLIP

OpenAI CLIP 論文：<https://arxiv.org/abs/2103.00020>  
OpenAI CLIP：<https://github.com/openai/CLIP>  
OpenCLIP：<https://github.com/mlfoundations/open_clip>

用途：

- image-text contrastive embedding
- 用自然語言 prompt 對 frame 做 zero-shot ranking
- 不需要先標資料就能搜尋「像不像某種畫面」

可用 prompts：

```text
restaurant storefront
restaurant sign
menu board
food dish close-up
street sign
business hours sign
map
receipt
price list
shop entrance
Japanese restaurant exterior
Taiwanese street food stall
```

訓練方式：

- 大量 image-text pairs 對比學習
- 若要微調，需要 image-text pairs 或 frame labels

適合用途：

- 第一版 semantic scoring
- top-K frame ranking
- frame index / text query retrieval

優點：

- 最容易接
- zero-shot 可用
- 不用先有資料集

限制：

- 對小字、招牌內容本身不如 OCR
- 對文化語境、店名、中文/日文招牌需要搭配 OCR/VLM
- prompt tuning 會很重要

建議：

> 先用 OpenCLIP ViT-B/32 或 ViT-L/14 做第一版；後續與 SigLIP 對比。

---

### 3.2 SigLIP / SigLIP2

Hugging Face 文件：<https://huggingface.co/docs/transformers/en/model_doc/siglip>  
論文：<https://arxiv.org/abs/2303.15343>  
Google big_vision：<https://github.com/google-research/big_vision>

用途：

- CLIP-like image-text model
- 使用 sigmoid loss，不依賴 batch 內全 pair softmax
- 可做 zero-shot image classification / retrieval

適合用途：

- 替代 CLIP 做 frame-text retrieval
- 對候選 frames 做 prompt similarity score
- 建立 frame embedding index

優點：

- Hugging Face Transformers 可直接載
- 通常 retrieval / embedding 表現強
- 對小 batch / scaling 有設計優勢

限制：

- 還是不能取代 OCR
- prompt 設計與 domain 測試仍必要

建議：

> MVP 可以先 CLIP，v1 加 SigLIP A/B test。若 SigLIP 對招牌、菜單、街景 prompt 更穩，就改用 SigLIP。

---

### 3.3 DINOv2

論文：<https://arxiv.org/abs/2304.07193>  
程式碼：<https://github.com/facebookresearch/dinov2>

用途：

- self-supervised image features
- 不靠文字，產生強視覺表徵
- 適合 clustering、near-duplicate removal、novelty detection

Meta repo 說 DINOv2 models 是在 142M images 無標註資料上預訓練，可直接用於多種 vision tasks。

適合用途：

- 同一 shot 內去重
- 找視覺上不一樣的 frames
- 每 shot 選代表 frame
- novelty score

優點：

- 視覺特徵強
- 不受 prompt 限制
- 對 clustering / diversity 特別好

限制：

- 沒有文字對齊能力
- 不能直接問「這是不是菜單」

建議：

> DINOv2 不取代 CLIP/SigLIP，而是輔助去重與 diversity selection。

---

### 3.4 FAISS

來源：<https://github.com/facebookresearch/faiss>

用途：

- dense vector similarity search
- clustering / nearest neighbor / cosine similarity
- 可支援大量 vectors，含 GPU 加速

適合用途：

- 儲存 frame embeddings
- 找相似 frames 去重
- 對 text query 找相似 frames
- 做 cluster representative selection

建議：

> 每個 frame 存 CLIP/SigLIP/DINOv2 embeddings；用 FAISS 做 similarity search 與去重，避免 top-K 全是同一段連續畫面。

---

## 4. 第三層：OCR / 招牌 / 菜單 / 地理文字

### 4.1 PaddleOCR

來源：<https://github.com/PaddlePaddle/PaddleOCR>

用途：

- OCR detection + recognition
- 支援 100+ languages
- 支援 scene text、document parsing、structured output

官方 repo 強調：

- PP-OCRv5 支援多語言混合文本，包括中文、英文、日文等
- 支援 scene text spotting
- 可輸出 JSON / Markdown 等結構化結果

適合用途：

- 招牌文字
- 菜單文字
- 門牌 / 地址 / 電話
- 營業時間
- 街牌 / 地標文字
- 日文 / 中文 / 英文混合場景

優點：

- 對這個專案非常關鍵
- 比把所有 frames 丟 VLM 便宜
- OCR text 可直接送 LLM 做店名候選與 Maps 查詢

限制：

- 模糊、反光、傾斜、遮擋時會錯
- 需要保存 confidence 與 bbox，不要盲信文字

建議：

> PaddleOCR 應是 MVP 必備。EasterEgg Vision Scout 的很多價值會來自 OCR yield。

---

### 4.2 EasyOCR / MMOCR

EasyOCR：<https://github.com/JaidedAI/EasyOCR>  
MMOCR：<https://github.com/open-mmlab/mmocr>

用途：

- EasyOCR：上手快，Python-friendly
- MMOCR：OpenMMLab OCR toolbox，研究/訓練彈性較高

建議：

- MVP 優先 PaddleOCR
- EasyOCR 可當快速 fallback
- MMOCR 適合未來想更深入訓練 OCR 或比較模型時使用

---

### 4.3 EAST / CRAFT

EAST 論文：<https://arxiv.org/abs/1704.03155>  
CRAFT 論文：<https://arxiv.org/abs/1904.01941>

用途：

- EAST：快速 scene text detector
- CRAFT：character region awareness，適合任意方向、彎曲、變形文字

適合用途：

- 只先判斷「這張 frame 是否有文字」
- 對招牌、街景文字做 text-density score
- 作為 PaddleOCR 前置或 fallback

建議：

> 第一版不一定需要單獨接 EAST/CRAFT；若 PaddleOCR 太慢，可以用 text detector 先 filter。

---

## 5. 第四層：open-set detection / region-level evidence

### 5.1 Ultralytics YOLO

來源：<https://github.com/ultralytics/ultralytics>

用途：

- object detection
- segmentation
- classification
- tracking
- pose estimation

適合用途：

- 偵測人、車、食物類物件、交通標誌等
- 對 food / street / storefront 做輔助特徵
- 未來自訓特定類別 detector

優點：

- 很成熟
- inference 快
- 訓練流程完整

限制：

- 預訓練 COCO 類別不一定有「菜單」、「招牌」、「店門口」這些 domain 類別
- 商業使用需留意 license / enterprise terms

建議：

> YOLO 適合 v2 後做自訓小 detector，不是第一版必要核心。

---

### 5.2 GroundingDINO

論文：<https://arxiv.org/abs/2303.05499>  
程式碼：<https://github.com/IDEA-Research/GroundingDINO>

用途：

- open-set object detection
- 用文字 prompt 找物件區域
- 例如：`restaurant sign`、`menu`、`storefront`、`food`、`street sign`

適合用途：

- 不想先標資料時，用文字 prompt 找候選 boxes
- 產生 weak labels
- 幫 Label Studio 預標註
- crop 出 sign/menu region 再 OCR/VLM

優點：

- open vocabulary
- 很適合弱標註資料循環

限制：

- 比 CLIP/OCR 重
- 不適合掃所有 frames
- 可能需要 threshold tuning

建議：

> GroundingDINO 不要全量掃影片；用在 top frames 或 active learning 預標註最合理。

---

### 5.3 SAM / SAM2

SAM：<https://github.com/facebookresearch/segment-anything>  
SAM2：<https://github.com/facebookresearch/segment-anything-2>

用途：

- promptable segmentation
- SAM2 支援 video segmentation 與 streaming memory

適合用途：

- 把 GroundingDINO 找到的 sign/menu/storefront region 切出來
- 對候選物件 crop 做更乾淨 OCR
- 未來要追蹤影片中的同一物件時使用

限制：

- 第一版 frame ranking 不需要 segmentation
- 加 SAM 會增加複雜度

建議：

> MVP 不接 SAM。等需要 region-level OCR 或 tracking 再加。

---

## 6. 第五層：VLM / Video-language model 作候選驗證

### 6.1 Qwen2.5-VL

技術報告：<https://arxiv.org/abs/2502.13923>  
Repo：<https://github.com/QwenLM/Qwen2.5-VL>

用途：

- image/video understanding
- OCR / spatial reasoning / visual agent tasks
- 可判讀複雜畫面、文字、介面與細節

適合用途：

- top-K frames 最終判讀
- 問：「這張是否包含店名、地址、菜單、招牌？」
- 多 frame / short clip reasoning

建議：

> 用 Qwen2.5-VL 或同級 VLM 做最後 rerank，不要全片掃。

---

### 6.2 Video-LLaVA

論文：<https://arxiv.org/abs/2311.10122>  
程式碼：<https://github.com/PKU-YuanGroup/Video-LLaVA>

用途：

- image/video representation 對齊到 LLM
- video QA / caption

適合用途：

- 對候選 clip 做問答
- 判斷一段 shot 是否有視覺線索

限制：

- 比 frame embedding 重
- 應作二階段驗證器

---

### 6.3 LLaVA-OneVision

論文：<https://arxiv.org/abs/2408.03326>

用途：

- single-image、multi-image、video 共用模型
- 適合把一個 shot 的多張 frames 一起看

適合用途：

- 多 frame 問答
- 判斷同一 shot 裡是否出現連續線索
- 對比不同 frame 找最有資訊的一張

---

### 6.4 MiniCPM-V / MiniCPM-o

MiniCPM-V 論文：<https://arxiv.org/abs/2408.01800>  
Repo：<https://github.com/OpenBMB/MiniCPM-V>

用途：

- 小型高效多模態模型
- 支援 image/video/text，部分版本偏 edge / on-device 場景

適合用途：

- 本地長時間跑的低成本 VLM reranker
- 如果不想每次用雲端 API，可以測 MiniCPM-V 類模型

---

### 6.5 VideoMAE / InternVideo

VideoMAE 論文：<https://arxiv.org/abs/2203.12602>  
VideoMAE repo：<https://github.com/MCG-NJU/VideoMAE>  
InternVideo 論文：<https://arxiv.org/abs/2212.03191>  
InternVideo repo：<https://github.com/OpenGVLab/InternVideo>

用途：

- VideoMAE：masked autoencoder for video，學 video representation
- InternVideo：general video foundation model，結合 masked video modeling 與 video-language contrastive learning

適合用途：

- 需要 temporal understanding 時的 video feature extractor
- 對候選 clips 做更高階理解
- 後續研究，不是第一版必要

建議：

> EasterEgg v0 不需要 VideoMAE/InternVideo；等 frame-level pipeline 證明價值後，再評估是否要 clip-level model。

---

## 7. Adaptive / active frame sampling

### 7.1 AdaFrame

論文：<https://arxiv.org/abs/1811.12432>

用途：

- Adaptive Frame Selection for Fast Video Recognition
- 用 LSTM + global memory 決定下一張 frame 要看哪裡
- reward 平衡 recognition accuracy 與 frame cost

訓練方式：

- policy gradient
- 需要 video-level labels，例如 ActivityNet / FCVID 類資料

對 EasterEgg 的啟發：

不必完整複刻 AdaFrame，但可以借它的精神：

> 先低 FPS 掃描；遇到高價值或高不確定區段，再加密取樣。

可用簡化版 active sampling 規則：

- CLIP/SigLIP prompt score 高 → 該 shot 前後 ±N 秒加密取樣
- OCR text density 高 → 加密取樣
- DINOv2 novelty 高 → 加密取樣
- VLM 不確定但可能有線索 → 加密取樣
- shot transition 附近 → 額外取 start/mid/end

建議：

> 先做 heuristic active sampling，不要一開始 RL。等累積 labeled data 後，再訓練 ranker / policy。

---

## 8. 可參考 datasets

### 8.1 Video summarization / keyframe selection

#### TVSum / TVSum50

來源：<https://github.com/yalesong/tvsum>

內容：

- 50 支 YouTube 多類型影片
- 每 2 秒 shot 有 20 位 crowd worker 的重要度分數
- 經典 video summarization benchmark

用途：

- 學習 shot-level importance prediction
- 評估 summarization / keyshot selection 方法

對 EasterEgg 的限制：

- 它標的是「一般重要性」，不是「招牌 / 菜單 / 地理線索」
- 可當評估框架參考，不應作主要訓練資料

#### SumMe

來源：<https://gyglim.github.io/me/>

內容：

- 經典使用者影片摘要資料集
- 規模小、主觀性高

用途：

- summarization baseline
- 不適合直接代表 EasterEgg domain

---

### 8.2 Instructional / food / how-to videos

#### YouCook2

來源：<http://youcook2.eecs.umich.edu/>

內容：

- cooking procedure videos
- temporal annotations

用途：

- food-related video understanding
- procedure segmentation

限制：

- 偏 cooking，不等於 restaurant vlog
- 可用於 food scene / dish frame 類輔助

#### HowTo100M

來源：<https://www.di.ens.fr/willow/research/howto100m/>

內容：

- 1.2M YouTube instructional videos
- 136M captioned clips

用途：

- 大規模弱監督 video-text pretraining

限制：

- 下載與授權麻煩
- YouTube availability 會變動
- 對 MVP 太大

#### ActivityNet Captions

來源：<https://cs.stanford.edu/people/ranjaykrishna/densevid/>

用途：

- dense video captions
- temporal event localization / captioning

#### MSR-VTT

來源：<https://www.microsoft.com/en-us/research/publication/msr-vtt-a-large-video-description-dataset-for-bridging-video-and-language/>

用途：

- video-text retrieval benchmark
- 可參考 video-language retrieval 評估方式

---

### 8.3 OCR / scene text

#### TextOCR

來源：<https://textvqa.org/textocr/>

用途：

- scene text OCR dataset
- 可測自然場景文字偵測與辨識

#### ICDAR Robust Reading

來源：<https://rrc.cvc.uab.es/>

用途：

- scene text benchmark 系列
- OCR / text detection evaluation

---

### 8.4 Food / Places / Street scene

#### Food-101

來源：<https://data.vision.ee.ethz.ch/cvl/datasets_extra/food-101/>

用途：

- food classification
- 可作食物畫面 classifier 參考

#### Places365

來源：<http://places2.csail.mit.edu/>

用途：

- scene classification
- restaurant、street、market、kitchen、indoor/outdoor 等場景分類

#### Mapillary Vistas

來源：<https://www.mapillary.com/dataset/vistas>

用途：

- street-level scene understanding
- street sign / road / storefront context

#### Open Images

來源：<https://storage.googleapis.com/openimages/web/index.html>

用途：

- large-scale object detection / classification
- 可找部分 sign / food / street 相關類別

---

## 9. 弱標註與 active learning 資料循環

第一版不用先人工標幾萬張。更有效的是建立資料飛輪：

### 9.1 初始 weak labels

用現有工具產生弱標註：

- CLIP/SigLIP prompt score：`storefront`、`menu`、`sign`、`food`、`street sign`
- PaddleOCR：是否有文字、文字密度、OCR confidence
- DINOv2：novelty / cluster representative
- PySceneDetect/TransNetV2：shot id / shot boundary
- GroundingDINO：是否偵測到 `menu`、`restaurant sign`、`storefront`

初始 labels：

```text
useful_frame
has_sign
has_menu
has_food
has_storefront
has_geo_text
has_street_scene
duplicate_or_low_value
```

---

### 9.2 人工快速審核

建議工具：Label Studio  
來源：<https://github.com/HumanSignal/label-studio>

做法：

1. 每支影片輸出 top 100 candidate frames。
2. 人工標 useful / not useful / type。
3. 每輪只標 500–2,000 frames。
4. 保存 frame、timestamp、shot id、scores、OCR text、source video metadata。

---

### 9.3 訓練小模型

資料累積後，不是訓練大 VLM，而是先訓練小模型：

- Logistic regression
- LightGBM / XGBoost ranker
- 小型 MLP classifier
- image embedding + metadata features → usefulness score

Features：

- CLIP/SigLIP prompt scores
- DINOv2 embedding / novelty
- OCR text density / confidence
- blur score
- brightness / contrast
- shot position
- detector hits
- scene classifier scores

目標：

- useful / not useful
- frame type multi-label
- ranking score

這會比從零訓練 VLM 快很多，而且可解釋。

---

## 10. 評估方法：必須打敗 uniform sampling

EasterEgg Vision Scout 的第一個成功標準不是「看起來很 AI」，而是：

> 在同樣 top-K frame budget 下，比 uniform sampling 找到更多有用線索。

### 10.1 Baselines

- Uniform sampling：每 1 秒 / 2 秒 / 5 秒抽 frame
- Scene-change sampling：ffmpeg scene filter
- PySceneDetect 每 shot 抽 1 張

### 10.2 指標

#### Useful@K

Top K frames 中，有多少人工標為 useful。

#### Recall@K

影片中所有 useful moments，有多少被 top K 覆蓋。

#### OCR yield

每 K 張 frames 產出的有效 OCR 字數 / 店名 / 地址 / 菜單項 / 電話 / 營業時間數量。

#### Geo clue hit rate

街牌、地標、地址、門牌、店名、電話、語言文字等出現率。

#### Diversity

Top K 是否過度集中在同一 shot / 同一視覺 cluster。

#### Cost reduction

達到同樣 recall 所需 frames 數：

```text
uniform sampling 需要 300 frames
scout pipeline 只需 50 frames
→ 6x frame budget reduction
```

---

## 11. 推薦 MVP 實作方案

### v0：不訓練，先整合現成能力

目標：證明 pipeline 比 uniform sampling 更有效。

功能：

1. 用 ffmpeg 抽 baseline frames。
2. 用 PySceneDetect 切 shots。
3. 每 shot 抽 start/mid/end frames。
4. 過濾黑畫面、模糊畫面、低對比畫面。
5. 用 OpenCLIP 或 SigLIP 對 prompts 打分。
6. 用 PaddleOCR 產生文字與 text-density score。
7. 用 FAISS / pHash 去重。
8. 輸出 top 30 / 50 frames + JSON report。

輸出 JSON 欄位：

```json
{
  "video_id": "...",
  "frame_path": "...",
  "timestamp_sec": 123.4,
  "shot_id": 12,
  "scores": {
    "clip_storefront": 0.83,
    "clip_menu": 0.61,
    "ocr_density": 0.77,
    "novelty": 0.45,
    "blur_quality": 0.91,
    "final_score": 0.82
  },
  "ocr_text": "...",
  "matched_prompts": ["restaurant sign", "menu board"],
  "notes": "candidate storefront/menu frame"
}
```

---

### v1：調權重與加入 scene classifier

新增：

- prompt set tuning
- Places365 scene classifier
- per-shot diversity constraint
- OCR weighting
- face/person penalty，避免自拍畫面佔滿 top-K

---

### v2：active learning + 小模型

新增：

- Label Studio 標註
- train useful-frame classifier
- LightGBM / logistic regression ranker
- error analysis
- uncertainty sampling

---

### v3：region-level evidence

新增：

- GroundingDINO 找 menu/sign/storefront boxes
- crop-level PaddleOCR
- SAM/SAM2 optional segmentation
- VLM final verification

---

### v4：adaptive scanning policy

新增：

- 低 FPS 全片掃描
- 高分 / 高不確定 / 高 OCR / 高 novelty 區段加密取樣
- 可借 AdaFrame 思路，但先用 heuristic，不急著 RL

---

## 12. 開發順序建議

### 第一週：local prototype

- 建 Python package / scripts
- 支援本地影片路徑
- ffmpeg + PySceneDetect 抽 frames
- 產生 `frames/` 與 `metadata.json`

### 第二週：semantic/OCR scoring

- 接 OpenCLIP / SigLIP
- 接 PaddleOCR
- 寫 prompt config
- 輸出 top frames report

### 第三週：evaluation set

- 選 10–20 支美食 / 旅行 / 探店影片
- 人工標 top-K 有用性
- 比較 uniform sampling vs scout pipeline

### 第四週：ranking + repo report

- 調 scoring weights
- 加 FAISS / pHash 去重
- 輸出 HTML report
- 決定是否進入 v2 小模型訓練

---

## 13. 技術判斷

### Strong candidate：現在就該做

- PySceneDetect / TransNetV2 shot boundary
- OpenCLIP / SigLIP prompt scoring
- PaddleOCR
- FAISS 去重與 retrieval
- top-K VLM reranking
- Label Studio 小量人工標註

### Maybe later：有價值，但不是第一版

- GroundingDINO + SAM region pipeline
- Places365 / Food-101 classifier
- VideoMAE / InternVideo clip-level features
- LLaVA-OneVision multi-frame reasoning
- LightGBM / MLP useful-frame ranker

### Avoid for now：先不要碰

- 從零訓練 video foundation model
- 一開始做 reinforcement learning viewer
- 全量 frames 丟 VLM
- 大規模下載 YouTube dataset
- 未驗證前就做 public product / downloader

---

## 14. 最重要的判斷

Jones 的直覺是對的：這件事應該站在巨人的肩膀上，不該從零做一個不如現成模型的粗糙系統。

但「站在巨人的肩膀上」不是直接找一個神奇模型完成全部，而是組合現有強項：

- shot detector 擅長切影片結構
- CLIP/SigLIP 擅長語意篩選
- DINOv2 擅長視覺去重與 novelty
- OCR 擅長文字線索
- VLM 擅長少量候選的最終理解
- 小型 ranker 擅長把 domain-specific signals 融合成自己的 scout model

EasterEgg Vision Scout 真正的技術資產不是第一天就訓練出的模型，而是：

> 一個能持續從真實影片中產生候選、標註、評估、訓練、改進的資料與評估循環。

第一個可交付目標應該是：

> 用現成 stack 在 20–50 支影片上證明：同樣 top-K frames，Scout 找到的招牌、菜單、店名與地理線索，比 uniform sampling 明顯更多。

只要這件事成立，後面再訓練自己的 domain-specific frame scout model 就不是空想，而是自然下一步。

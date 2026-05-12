# EasterEgg Vision Scout

> 狀態：概念與技術計畫  
> 日期：2026-05-12

## 一句話

EasterEgg Vision Scout 是一個本地 computer vision prototype，用來從美食 / 旅行 / 探店影片中快速篩選「值得細看」的關鍵幀，例如店門口、招牌、菜單、食物特寫與地理線索。

## 背景

VlogMap 類產品如果只靠現成 AI API 拼接，容易變成大家都能做的應用。真正有技術資產價值的部分，是訓練自己的 domain-specific frame scout model。

它不需要完整理解整支影片，只要先回答：

> 這一幀是否可能包含有用線索？

如果能把 30 分鐘影片中 95–99% 無用畫面篩掉，只留下少量高價值 frames，後續 OCR / VLM / LLM / Maps 搜尋就會便宜且穩定很多。

## 第一版不使用 RL 的原因

這個任務看起來像 agent 快速掃影片，但第一版不適合直接用 reinforcement learning。

原因：

- reward 難設計
- 需要大量資料與互動環境
- 訓練不穩定
- 還沒有 baseline 與標註資料
- 用 supervised learning 已能快速驗證價值

第一版應該先用：

- frame extraction
- weak labeling
- CLIP / SigLIP embedding
- useful frame binary classifier
- OCR / VLM reranking

未來如果有足夠資料，再考慮 active perception / RL policy。

## 任務定義

輸入：影片 frames  
輸出：每一幀的 useful score 與類型標籤

初始標籤：

- 無關 / 主持人 / 過場
- 店門口 / 招牌
- 菜單 / 價格表
- 食物特寫
- 街景 / 地理線索
- 其他重要線索

第一版可以先做 binary：

- useful frame
- not useful frame

## 技術路線

完整第一版技術調查見：[`technical-research-fast-video-scanning-2026-05-12.md`](./technical-research-fast-video-scanning-2026-05-12.md)

建議先站在現有模型與工具上：

1. 從 YouTube / 本地影片抽 frames。
2. 用 PySceneDetect / TransNetV2 切 shots。
3. 用 OpenCLIP / SigLIP / DINOv2 產生 image embeddings。
4. 用 PaddleOCR 找招牌、菜單、門牌、街景文字。
5. 用 FAISS / pHash 去重與 diversity selection。
6. 用 Qwen2.5-VL / LLaVA-OneVision / MiniCPM-V 只判讀 top-K 候選。
7. 用現成模型產生 weak labels。
8. 人工快速校正少量標籤。
9. 再訓練 logistic regression、LightGBM 或小型 MLP classifier / ranker。
10. 接到 VlogMap / 地點抽取 pipeline。

## 評估方式

要證明它比 uniform sampling 更有用。

評估指標：

- Top 50 frames 中有幾張包含招牌 / 菜單 / 店門口？
- 相比每 10 秒抽一張，命中率提升多少？
- 是否能降低 VLM/OCR 需要看的 frame 數？
- 每支影片處理時間是否可接受？
- 對 Joeman / 台灣美食 / 日本 vlog 是否都能工作？

## 與 VlogMap 的關係

EasterEgg Vision Scout 是 VlogMap 的視覺前處理模組。

VlogMap pipeline：

1. YouTube URL
2. metadata / audio / transcript
3. transcript 店家候選
4. EasterEgg Vision Scout 篩選視覺關鍵幀
5. OCR / VLM 補強店名、招牌、菜單證據
6. Google Maps / Places 驗證
7. HTML / JSON 店家報告

## 未來 RL 路線

當資料足夠後，可以訓練 active sampling policy：

- 每次只能看少量 frames
- 模型決定下一個時間點要看哪裡
- reward 是找到有用線索並降低觀看成本

但這是第二或第三階段，不是第一版。

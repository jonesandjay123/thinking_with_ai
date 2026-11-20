# DIY × AI 機械手臂套件與黑五購買調查

- **查核日期：2026-11-19，美東晚間。** 黑色星期五為 11/27，Cyber Monday 為 11/30。
- **需求：** 可買到的桌上型機械手臂／套件，能自行 coding，讓 AI 透過 Python、API 或機器人學習框架控制；以美國購買為優先。
- **範圍：** 官方商品、活動、公開商品 JSON、開源硬體與 LeRobot 文件。不是實機評測，也不是銷量排行榜；「值得關注」不代表已證明最暢銷。
- **價格皆為美元，未含稅、運費與可能關稅；未加入購物車或測試結帳，未下單。**
- 本文是一般 DIY／AI 入門採購更新；另保留 [11/18 報告](2026-11-18-black-friday-robot-arm-research.md)，該文偏向精密觸控筆／手機操作。舊文的價格、折扣碼及庫存，不能直接當作今日有效。

## 一、先講結論

**有，而且不必買到數千美元。** 最值得先研究的三條路線：

1. **想做「教它幾次，讓它自己學」：SO-101 預組裝 leader＋follower。** 本次重查 Seeed 標價 **$284.05，原價 $299，約 5% off**，包含雙臂與一顆 USB 相機。它的價值是開源結構和 LeRobot 工作流，而不只是「能接 AI」。
2. **想先讓 Jarvis 用程式指揮一隻手：Waveshare RoArm-M2／M3。** 官方提供 JSON／HTTP／USB 控制介面，容易做成 AI tools。M2 系列頁 **$179.99–299.99**，M3 系列頁 **$239.99–339.99**；這是版本區間，不是已確認黑五降價。
3. **想要 DYNAMIXEL 生態與後續維修：ROBOTIS OMX-AI。** 雙臂 **$343.85**、相機另購；官方強調 dataset collection、teleoperation、imitation learning。但其 Physical AI 容器**不支援 Mac**，不能把整套 Linux 教學當作 Mac 開箱即用。

若目的是桌上夾積木、整理輕物、相機辨識後抓取，我會優先 SO-101 或 RoArm-M3。若目的是精準觸控筆按手機，則應另外驗證末端精度、剛性與標定，不因為便宜或「AI 相容」就假設可用。

## 二、你說的「AI 操控」其實有三種

| 路線 | 運作方式 | 合適情境 | 要買什麼 |
|---|---|---|---|
| A：LLM 呼叫工具 | 語言模型決定任務，Python／控制器執行受限制的動作 | 「移到 A」「夾起積木」「畫一條線」 | 一隻有公開 API 的手臂即可；不一定要 leader |
| B：視覺閉環 | 相機看物體，程式轉換座標、移動、再拍照確認 | 顏色分類、桌上取放、確認夾取失敗 | 手臂＋固定相機＋標定板／工作區 |
| C：模仿學習／VLA | 人操作 leader，follower 跟隨並錄資料，再訓練 policy | 示範後學抓取、固定場景重複任務 | 通常 leader＋follower＋相機；另有訓練算力需求 |

**「可以用 Python 控制」「可接 ChatGPT」「LeRobot 支援」「模型已會做你的任務」是四件不同的事。** 相容某個框架不代表下載任意預訓練模型就能直接操控這個機型。機構、動作空間、相機配置、資料與 policy 都需對齊。

## 三、購買選項比較

| 產品／通路 | 今日可讀標價 | 套裝性質 | 控制／AI 路徑 | 黑五證據 | 主要注意事項 |
|---|---:|---|---|---|---|
| [Seeed SO-101 Assembled Kit Pro](https://www.seeedstudio.com/SO-101-Assembled-Kit-Pro-p-6691.html) | **$284.05**，原 $299 | 預組裝 leader＋follower、相機 | LeRobot；上游 Feetech 生態 | 商品頁確認 5% 現價折扣；結束日未在本頁確認 | 電源清單只明列線材，需確認變壓器本體；庫存與美國落地成本未確認 |
| [Waveshare RoArm-M2](https://www.waveshare.com/roarm-m2-s.htm) | **$179.99–299.99** | M2-S／M2-Pro，不同版本 | JSON over HTTP／USB／UART、ROS 2 | 未確認黑五折扣 | 4-DOF，末端姿態限制較多；不是雙臂模仿學習整套價 |
| [Waveshare RoArm-M3](https://www.waveshare.com/roarm-m3.htm) | **$239.99–339.99** | M3-S／M3-Pro，不同版本 | API、ROS 2；廠商宣稱 LeRobot 相容 | 未確認黑五折扣 | 5＋1 DOF；LeRobot 驅動與訓練版本需按廠商教學核對 |
| [ROBOTIS OMX-AI US](https://robotis.us/products/omx-ai-us) | **$343.85** | OMX-L＋OMX-F、需組裝，相機不含 | 官方 Physical AI 工作流 | 未找到本次讀取頁面的折扣證據 | 商品 JSON available=true，但頁面另有交期提示；不可當成立即出貨保證 |
| [Hiwonder ArmPi mini](https://www.hiwonder.com/products/armpi-mini) | 無 Pi **$129.99**；含 Pi 5 2GB **$279.99**／4GB **$339.99**／8GB **$439.99** | 5DOF 視覺教育套件，依 Pi 選項不同 | Python、OpenCV、Pi 端程式 | 分類頁顯示 $129.99／$149.99；黑五有另外滿額活動 | **$129.99 不含 Pi**，不是完整可獨立運作價；已有 Mac 也不等於可直接取代套件的 Pi |
| [Hiwonder MaxArm](https://www.hiwonder.com/products/maxarm) | Starter **$199.99**；Standard **$249.99**；Advanced **$299.99** | ESP32 教育／開發套件 | Python／Arduino、逆運動學 | 未確認這些版本額外降價；可關注全站滿額活動 | 先比各包裝的末端、相機與配件，不以名稱推定全部都含 |
| [Hiwonder SO-ARM101](https://www.hiwonder.com/products/lerobot-so-101) | DIY **$269.99**；Starter **$389.99**；Standard **$419.99**；Advanced **$459.99** | 同名產品有不同套裝 | 廠商提供 LeRobot／VLA 路線 | 活動配置列 $439.99／原 $459.99，但未與商品 variant 對上 | 不可拿 DIY 價和 Seeed 預組裝雙臂價直接比較；伺服與驅動須核對 |
| [Elephant Robotics myCobot 280](https://shop.elephantrobotics.com/collections/mycobot-280) | Uno R4 **$499**／原 $599；M5 **$649** | 預組裝教育／開發機械臂，末端配件另查 | pymycobot；自建 AI tool adapter | Americas 官方有 BFCM 滿額碼，但活動狀態顯示矛盾 | 不保證含夾爪；Americas 活動頁 Uno R4 寫預購、11 月底到貨 |

### 為何我最看好 SO-101

不是因為它的硬體最精密，而是其開發路徑比較透明：

- [LeRobot 官方 SO-101 文件](https://huggingface.co/docs/lerobot/so101)有馬達配置、Mac／Linux USB port 範例、設定 ID、校正，以及 follower／leader 流程。
- [上游 SO-ARM100／SO-101 repo](https://github.com/TheRobotStudio/SO-ARM100)公開列印檔、零件表與供應商，包含 Seeed、Waveshare、PartaBot、ForgeMotion Labs 等；這是開源生態採用證據，不是銷量證明。
- 官方 LeRobot 範例直接使用 `SO101Follower` 和 `SO101FollowerConfig`；後續接 Python tool 層不必從未知串口協定開始。
- 注意它是 **5 個手臂關節＋夾爪**，部分商家叫 6DOF，不能據此以為它等同六軸工業機械臂外加夾爪。

### Seeed 預組裝套件清單核對

本次商品正文列：黑色 leader ×1、白色 follower ×1、彈性夾爪件、32×32mm USB 相機 ×1、伺服驅動板 ×2、5V／12V 電源線、USB-C 線 ×2、DC 尾線、螺絲起子、桌夾 ×4、滑鼠墊。

**待確認：** 清單寫的是 power supply cable，不足以保證 AC/DC 變壓器也含在內。商品比較表寫舊參考價 $299，實際購買區為 $284.05，應以購買區與結帳為準；商品規格的馬達數量欄也有不一致文字，不以該單一欄位推導 BOM。

## 四、2026 黑五：哪些是真的，哪些還不能算成交價？

### 1. Seeed：今日確認商品降價，不採用未重驗優惠碼

- SO-101 預組裝套件 **$299 → $284.05**，省 **$14.95／5%**。[商品來源](https://www.seeedstudio.com/SO-101-Assembled-Kit-Pro-p-6691.html)
- 運費與 duties 顯示結帳時計算。
- 11/18 舊報告提過 `W1KY16FE` 減 $30、其他全站滿額規則及訓練 credits；**本次沒有重新證實適用資格與疊加，也未測試結帳，故不把 $254.05 當作今日可得價。**
- 僅憑商品降價，不能保證它是全年最低價。

### 2. Hiwonder：官方黑五活動配置已可讀，便宜單臂未必達門檻

[官方黑五頁](https://www.hiwonder.com/pages/black-friday)寫「Up to $300 OFF」「Stackable with select sale items」。其頁內公開 JSON 配置列：

| 訂單門檻 | 公布折抵 |
|---:|---:|
| $399 | $20 |
| $899 | $70 |
| $1,599 | $160 |
| $2,499 | $300 |

- 配置 `endAt=2026-12-01T16:00:00+08:00`，換算為 **12/1 00:00 PST＝12/1 03:00 EST**，與頁面「Ends December 1, 00:00 PST」一致。
- 因為動態商品卡依賴 JavaScript，純文字抽取顯示部分 placeholder；本報告另外讀了官方頁面的 `black-friday-config`，不是從優惠聚合網站抄數字。
- **有公布滿額活動，不代表每個 SKU 都適用或可以任意疊碼。** 配置 couponCode 為空；沒有自行杜撰折扣碼。
- 活動內 SO-ARM101 為 **$439.99／原 $459.99**；但商品 JSON 的 Advanced 仍為 $459.99，其餘版本也不同。因此 $439.99 只記為活動展示價，**尚未確認對應 variant 與結帳價格**。
- 以 $199.99 買一隻 MaxArm，並不會因「最高省 $300」就自動得到 $20 或更多折扣；不建議為滿額湊不需要的配件。

### 3. Elephant Robotics：優惠碼已公布，但頁面狀態不一致

[Americas 官方活動頁](https://americas.shop.elephantrobotics.com/pages/black-friday-cyber-monday-sale)公布至 **2026-12-03**：

| 官網門檻用語 | 優惠碼 | 折抵 |
|---|---|---:|
| orders over $299 | `BFCM20` | $20 |
| orders over $799 | `BFCM30` | $30 |
| orders over $1,299 | `BFCM100` | $100 |
| orders over $2,499 | `BFCM300` | $300 |

但同一頁也抽出「This offer has ended」和零倒數，並寫部分折扣結帳自動套用，與 2026 有效日期並列。**可能涉及前端計時／呈現問題，但未經瀏覽器與結帳確認，不判定碼已生效或已失效。**

- Uno R4 現價 $499；若可疊 `BFCM20`，算術結果是 **$479**，這只是**條件式試算**。
- Americas 活動頁另列 Pi 版 **$759／原 $799**、Jetson Nano 版 **$809／原 $849**；全球店分類頁仍顯示 $799／$849。**不同區域店不可混為同一報價。**
- Uno R4 文字寫「Pre-order」「Expected in Stock by Late November」，不應因有 Add to cart 按鈕就說是美國倉現貨。
- 想夾東西要算夾爪；全球分類頁 adaptive／parallel gripper 各 $129.99、pen holder $69.99，買裸臂不一定能立即抓取。

### 4. Waveshare／ROBOTIS：目前僅確認標價，沒有證明額外黑五折扣

「未找到折扣證據」**不等於廠商不會辦、也不等於已確認沒有活動**。如果很在意買最低價，可在 11/26–27 再查；但不要把無折扣的合理選型排除，只追折數。

ROBOTIS OMX-AI 商品 HTML 含通用的「Sale Sold out」文字，也有 Add to cart；本次加讀 [官方商品 JSON](https://robotis.us/products/omx-ai-us.js)，`available=true`、variant price=34385（美分）。這支持可銷售狀態，但頁面還有「LEAD TIME REQUIRED / Estimated Mid-November」，所以真實交期仍需確認。

## 五、純 DIY 到底能多便宜？

上游 [SO-ARM100／101 零件表](https://github.com/TheRobotStudio/SO-ARM100)目前列：

- 單 follower：**$121.94** 參考 BOM。
- leader＋follower：**$229.88** 參考 BOM。

這些是 repo 裡按供應商單價計算的零件表，**不是今日含運可結帳的商品價**；還需考慮列印材料／工時、相機、不同地區運費、零件補貨與調校。leader 馬達的齒輪比配置不全相同，不能隨便買 12 顆同型伺服就假設等同完整 SO-101。

已有 3D 印表機、想學馬達 ID／接線／機構的人可以 DIY；如果主要目標是 AI coding，預組裝雙臂 $284.05 與參考雙臂 BOM 的差額約 $54.17，還附相機，省下的工時可能更值得。

## 六、Mac mini M4 16GB 怎麼搭配？

依本次對話已確認的 Mac mini M4／16GB 規格，**先把它當控制主機，不必先添購 Jetson 或大顯卡**。

```text
Jones 的文字／語音指令
    ↓
雲端模型，或適合 M4 的小型本地模型
    ↓
Jarvis 工具層：讀相機、選擇動作、檢查結果
    ↓
本機 Python bridge：座標轉換、限速、範圍檢查
    ↓
USB／HTTP → 手臂控制器 → 馬達
    ↑
相機與關節狀態回饋
```

- **第一階段：** Python API＋USB 相機＋固定工作區，讓模型呼叫少量工具，例如 `get_robot_state()`、`move_to_named_pose()`、`open_gripper()`、`stop()`。這不用在本地跑大型 VLA。
- **第二階段：** 加入相機標定與物體位置估計，做固定區域的輕物取放；模型負責任務規劃，不負責直接產生高速伺服迴路。
- **第三階段：** 若真的需要「示範後學會」，再接 leader、錄製資料、測 ACT／其他相容 policy。Mac 的控制／錄製可行，不代表所有訓練程式和大型 VLA 都能在 MPS 上順跑；依模型支援選 Linux／CUDA 或租雲端訓練。
- **ROBOTIS 特別注意：** 官方商品頁直接寫 Physical AI container 不相容 Mac。若選它，需先釐清原生通訊與實驗要放在哪個主機，不把有 Python 就視為整套相容。
- **Waveshare 特別注意：** HTTP／JSON 適合做跨平台 bridge，但這是依介面作出的架構判斷，非本報告已完成 Mac 實機驗證。

## 七、第一個可完成的使用案例

建議題目：**「把固定托盤裡的紅色積木放進 A 區」**。

1. 固定底座與相機，在桌面定義小工作區，先手動確認每個命名姿態。
2. 寫 Python 動作：回原點、接近、夾取、抬高、移到 A、放開；先不接 LLM。
3. 接相機，判斷積木是否在預定格位、抓取後是否仍留在原位。
4. 把已驗證動作包成 AI tools，模型只能挑選有限動作與允許的目標區。
5. 每次記錄成功／失敗；先以固定格位達到穩定，再增加隨機位置或模仿學習。

必要控制邊界很簡單：低速起步、限制工作區／負載、保留實體斷電或停止方式，失去回饋便停止。不要讓語言模型繞過 bridge 直接發送任意馬達命令。

## 八、我會怎麼選與抓總預算

以下是**規劃預算，不是套裝報價**，稅／運／關稅另計。

| 目標 | 建議選擇 | 第一階段規劃預算 | 理由 |
|---|---|---:|---|
| 最快進入 LeRobot 與模仿學習 | Seeed SO-101 預組裝雙臂 | **$350–450** | 手臂 $284.05；餘額留給電源確認、治具、相機支架與備件 |
| 先讓 AI 經 API 操控單臂 | RoArm-M3 | **$300–500** | 系列 $239.99–339.99；另留相機／固定與連接配件預算 |
| 更低成本的固定取放／程式控制 | RoArm-M2 或 MaxArm Starter | **$250–350** | 先控制複雜度；接受姿態／套裝功能限制 |
| Pi＋OpenCV 教學整合 | ArmPi mini 含 Pi 版本 | **$330–500** | 避免買到 $129.99 無 Pi 版本後才發現缺主機 |
| 偏 DYNAMIXEL、dataset 工作流 | OMX-AI | **$420–550** | 雙臂 $343.85；相機、固定與配件另外算，先解決 Mac 容器限制 |
| 偏既有 SDK 與成熟配件 | myCobot 280 | **$650–900** | 裸臂 $499 起，夾爪、相機、底座可能另買；本次不保證 LeRobot 開箱即用 |

**針對「DIY＋AI coding」：我會先選 SO-101 預組裝，或 RoArm-M3。** 如果你最想做的是讓 Jarvis 有一隻可呼叫工具的手，RoArm 的 HTTP／JSON 很直接；如果你想親手教動作、研究 physical AI，SO-101 的 leader＋follower 更對題。先別為了「有 AI」花錢買內建 Jetson，任務與軟體相容性比板子算力重要。

## 九、下單前只核對這幾件事

- SKU 是單 follower、雙臂、馬達包、列印框架，還是完整預組裝套件？相同產品名不能代表相同內容。
- 電源變壓器、USB 資料線、桌夾、夾爪、相機、Pi／Jetson 是否實際包含？
- 美國出貨倉、送紐約的稅運關稅與出貨日；有貨、可預購、需等交期分開確認。
- 套件採哪家伺服？LeRobot 教學與驅動是否對得上該 SKU／韌體？
- 黑五碼是否真的套用、能否與標價疊加？只有結帳前的訂單摘要才是可核對總價。
- 若是手機點按或細小目標：先談末端精度驗收，不把馬達 encoder 解析度當作整臂定位精度。

## 十、來源與查核限制

### 本次直接讀取的主要來源

1. [Seeed SO-101 預組裝套件](https://www.seeedstudio.com/SO-101-Assembled-Kit-Pro-p-6691.html)：價格、比較表與包裝清單；一般 HTTP 抓取遇到 403，改用 Web Fetch 取得正文與 $284.05 標價。
2. [LeRobot SO-101 官方文件](https://huggingface.co/docs/lerobot/so101)：安裝、馬達、Mac USB、校正與 Python 類別。
3. [TheRobotStudio 上游 repo](https://github.com/TheRobotStudio/SO-ARM100)：開源設計、供應商與參考 BOM；本次另讀 raw README。
4. [RoArm-M2](https://www.waveshare.com/roarm-m2-s.htm)、[RoArm-M3](https://www.waveshare.com/roarm-m3.htm)：系列價格、DOF、控制介面與廠商相容性宣稱。
5. [ROBOTIS OMX-AI](https://robotis.us/products/omx-ai-us)、[商品 JSON](https://robotis.us/products/omx-ai-us.js)：價格、可銷售狀態、交期提示、相機不含、Mac 容器限制。
6. [Hiwonder 黑五頁](https://www.hiwonder.com/pages/black-friday)：含公開 JSON 活動配置、期限、門檻與活動展示價。
7. [Hiwonder 手臂分類](https://www.hiwonder.com/collections/robotic-arm)、[ArmPi mini](https://www.hiwonder.com/products/armpi-mini)、[MaxArm](https://www.hiwonder.com/products/maxarm)、[SO-101](https://www.hiwonder.com/products/lerobot-so-101)：本次另讀各商品 `.js` 的 variant 價格與 available 欄位。
8. [Elephant Robotics 全球分類](https://shop.elephantrobotics.com/collections/mycobot-280)、[全球黑五頁](https://shop.elephantrobotics.com/pages/black-friday-cyber-monday-sales)、[Americas 黑五頁](https://americas.shop.elephantrobotics.com/pages/black-friday-cyber-monday-sale)：區域售價、優惠碼、末端配件、預購與矛盾倒數。

### 限制

- 搜尋引擎結果受到反自動化頁面限制；本報告主要依上述官方一手頁面與開源文件交叉查核，**不是全網優惠窮盡搜尋**。
- 未進行實機測試、瀏覽器結帳或地址報價。沒有把官方 demo 當作平均成功率，也沒有把 available=true 當成明天就能送達。
- 沒有重新證實舊報告所有型號、精度、優惠碼與第三方評測；相關舊結論不自動帶入本報告。
- 活動標價與商品 JSON 衝突時，並列揭露，不任挑比較低的當作成交價。

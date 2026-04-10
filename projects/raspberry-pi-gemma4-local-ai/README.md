# Raspberry Pi 5 + Gemma 4 E2B：本地 AI DIY 專案點子

> 2026-04-07 | 由 Jarvis（Dispatch + Mac Code Task 協作）產出
> 靈感來源：Zero to MVP 的 YouTube 教學影片《Gemma 4 on Raspberry Pi 5: A Surprisingly Usable Local AI Setup》（2026-04-05，159K 觀看）

---

## TL;DR

一部 $80 的 Raspberry Pi 5（8GB RAM）可以跑 Google Gemma 4 E2B 模型做本地 AI，不需要雲端、不需要 GPU、不需要 API key。速度不是給互動用的（約人類舒適閱讀速度），但對「非即時的腳本 / 自動化任務」完全可用。這份文件是一個**週末 DIY 專案的完整規劃**，給 Jones 當科技宅玩具入門。

---

## 一、靈感來源影片

- **標題：** Gemma 4 on Raspberry Pi 5: A Surprisingly Usable Local AI Setup
- **頻道：** Zero to MVP（Nick，20 年軟體開發資歷）
- **URL：** https://youtu.be/kZhAj8--t8w
- **發布：** 2026-04-05
- **長度：** 約 9 分鐘
- **觀看：** 159,527
- **讚數：** 3,807

**影片做了什麼（一句話）：** 把 Google Gemma 4 E2B 跑在 Raspberry Pi 5（8GB）上，用 LM Studio CLI 起 OpenAI 相容的 API server，透過 socat 做區網 port forwarding，最後接到 Zed Editor 做 AI coding 測試。

---

## 二、為什麼這個專案對 Jones 特別值得做

1. **你已經對 Gemma 4 很熟**——我們前幾天做過 Gemma 4 全版本評估、Mac Mini 實測失敗、OpenClaw 復活計畫。這個專案剛好是「把知識落地成一個小硬體」
2. **AI 機器人 DIY 的第一步**——你想做 LLM 加 Raspberry Pi 機器車，這個影片就是**腦袋的部分**跑通的證明。硬體 + 感測器 + 輪子之後再加上去
3. **完整可重現**——硬體成本低（$100 以內）、週末可完成、步驟公開、工具全開源
4. **跟 media-index 有延伸關係**——Pi 5 可以跑 Whisper + Gemma 4 E2B 做邊緣影音解析節點（輕量版）
5. **紐約可取得**——所有硬體 Amazon / Adafruit / Micro Center 都能隔天到

---

## 三、硬體採購清單（紐約視角）

| 項目 | 規格 | 預估價（USD） | 去哪買 |
|---|---|---|---|
| **Raspberry Pi 5** | 8GB RAM 版 | $80 | Adafruit、Amazon、Micro Center NYC |
| **官方電源** | 27W USB-C PD | $12 | 官方配件才穩，不要省 |
| **Active Cooler** | 官方散熱風扇 | $5 | 跑 LLM 一定要散熱，不然會降頻 |
| **NVMe SSD（256GB+）** | M.2 2230 或 2242 | $25-40 | WD、Samsung、Kingston 都行 |
| **M.2 HAT+** | 官方 NVMe HAT | $12 | Pi 5 接 SSD 的官方擴充板 |
| **microSD 卡** | 32GB Class A2 | $8 | 開機用，SSD 存模型 |
| **外殼** | 散熱穿孔版 | $10-25 | Argon One、Flirc、Pimoroni |
| **HDMI 線 / 鍵鼠** | 首次設定用 | $0-20 | 之後 SSH 接入就不需要 |
| **總計** | — | **~$150-200** | — |

### 紐約實體店推薦

- **Micro Center Brooklyn**（布魯克林） — 最齊全，有時候有 Pi 5 套件組合包打折
- **Adafruit NYC**（曼哈頓）— Pi 官方生態 + sensor + HAT 超齊
- **B&H Photo Video** — 也有 Pi 和周邊
- **Amazon Prime** — 隔天到最方便

### 可以省的 vs 不能省的

- **✅ 不能省：** 8GB 版本（不是 4GB）、NVMe SSD、Active Cooler、27W 官方電源
- **⚠️ 可以省：** 外殼、HDMI 線（如果你有現成的）
- **❌ 千萬別省：** 電源不要用第三方廉價款，Pi 5 對電源很挑

---

## 四、軟體 Stack（影片作者的配方）

```
Raspberry Pi OS (lite，無 GUI) 或 Ubuntu Server 24.04 aarch64
  ↓
LM Studio CLI（headless 版）
  ↓
Gemma 4 E2B 模型（約 4.5GB，Q4 量化）
  ↓
LM Studio API Server（port 4000，OpenAI 相容）
  ↓
socat（把 localhost:4000 → 0.0.0.0:4001 讓區網能連）
  ↓
Zed Editor（或任何 OpenAI 相容 client）從 Jones 的 Mac 連入
```

### 額外好用工具

- **tmux** — 讓 LM Studio server 在背景跑，SSH 斷線不中斷
- **htop** — 監控 CPU / RAM 使用率
- **iperf3** — 測區網頻寬（如果 Pi 是 WiFi 接入）

---

## 五、實作步驟（週末時間軸）

### 🕐 第一天早上（2-3 小時）：硬體組裝 + 作業系統

1. 把 M.2 HAT+ 裝在 Pi 5 上，接 NVMe SSD
2. 裝 Active Cooler 和外殼
3. microSD 燒 Raspberry Pi OS Lite（用 Raspberry Pi Imager）
4. 首次開機，設定 SSH、網路、改 hostname（例如 `jarvis-edge`）
5. 從 Mac SSH 連入，之後都不用 HDMI
6. **把 rootfs 搬到 NVMe SSD**（這一步讓整機大幅加速，也把模型存到 SSD 而不是 microSD）

### 🕑 第一天下午（1-2 小時）：LM Studio + Gemma 4

1. `curl -fsSL https://lmstudio.ai/install.sh | sh` 或官方 CLI script
2. `lms ls` 確認裝好
3. 修改模型儲存路徑到 SSD：`lms settings set models-path /mnt/ssd/models`
4. `lms get gemma4:e2b`（或對應的 HuggingFace repo）
5. `lms load gemma4:e2b` 載入模型進記憶體
6. `lms server start --port 4000` 起 API server
7. `curl http://localhost:4000/v1/chat/completions -d '{"model":"gemma4:e2b","messages":[{"role":"user","content":"Hello"}]}'` 測試

### 🕒 第一天晚上（30 分鐘）：區網暴露 + Mac 連入

1. 安裝 socat：`sudo apt install socat`
2. 跑 port forwarding：`socat TCP-LISTEN:4001,fork,reuseaddr TCP:localhost:4000`
3. 從 Mac 測試：`curl http://jarvis-edge.local:4001/v1/models`
4. 在 Zed Editor 或任何 OpenAI 相容 client 加 custom endpoint
5. 讓 tmux 保持 socat 常駐

### 🕓 第二天：玩法探索

以上影片的內容到這邊就結束了。下面是 **Jones 可以自己延伸的方向**。

---

## 六、⭐ Jones 的延伸方向（影片沒講，我幫你想的）

### 方向 A：Jarvis 輕量邊緣節點

目前 Jarvis 主機是 Mac Mini M4 16GB，但我們試過跑 Gemma 4 會被記憶檔案擠爆 VRAM。**Pi 5 可以當「輕量任務專用節點」**，只跑簡單的分類、Tag、摘要，把 Mac Mini 解放出來。

**典型用例：**
- Mac Mini 收到一封 email → HTTP call Pi 5 → Pi 5 用 Gemma 4 E2B 分類「重要 / 不重要 / 促銷」→ 回傳 Mac Mini
- 這樣 Pi 5 只做「一次性分類」，不累積 context，完美符合無狀態 worker 模式
- 功耗約 10-15W，24/7 一個月電費不到 $2

### 方向 B：media-index 的邊緣 POC 預熱

你現在 media-index 主力想放在 5080 Jovix 跑 Whisper + Gemma 4 26B。但 Pi 5 可以先做一個**最小可行版**：

- Pi 5 + Gemma 4 E2B 處理短影片（1-3 分鐘）
- Whisper tiny 做中文 transcript（Pi 5 可以跑 tiny 或 base 模型）
- Gemma 4 E2B 抽實體
- 結果寫成 JSON

**價值：** 在 Jovix 還沒買 / 還沒設好之前，這是一個**$150 的 media-index 概念驗證機**。做得通就知道真的要上 5080 做大版本；做不通就知道哪裡會卡，不用先燒錢。

### 方向 C：AI 機器車的「腦」

Pi 5 接上 motor driver board（如 Waveshare Build HAT）、攝影機、超音波感測器 → 你就有一台 **LLM 驅動的機器車**。

**基礎玩法：**
- 攝影機拍畫面 → Gemma 4 E2B 的 vision 分析「前面是什麼」
- 超音波偵測距離 → Gemma 4 判斷要左轉右轉
- 使用者語音指令（Whisper tiny）→ Gemma 4 決定動作 → 發馬達指令

**進階玩法：**
- 接 ESP32-CAM 當副眼睛
- LEGO Technic 車身 + Pi 5 當腦
- 跑 SLAM 做室內地圖

這個完全符合你「想 DIY AI 機器車」的夢想，而且 $200 就能開始玩。

### 方向 D：便攜式 AI 終端

- Pi 5 + 小螢幕（7 吋 touchscreen）+ 電池包 → 手持本地 AI 終端
- 完全離線的 ChatGPT 替代品
- 路上靈感來了用語音問問題
- 隱私 100%，不靠網路
- **防災情境**可用（斷網時還有 AI 助手）

---

## 七、學習路徑（如果 Jones 想從零開始）

如果你完全沒碰過 Raspberry Pi，這是建議的入門順序：

### 第一週：基礎熟悉
- [ ] 開箱 + 裝 Raspberry Pi OS
- [ ] SSH 連入，學基本 Linux 指令（ls / cd / nano / sudo）
- [ ] 學 tmux 基礎（screen split、detach、attach）
- [ ] 跑一個 Hello World Python 腳本

### 第二週：影片內容重現
- [ ] 跟著 Zero to MVP 影片完整跑一遍
- [ ] 裝 LM Studio + Gemma 4 E2B
- [ ] Zed Editor 連上 Pi → 問 Gemma 一個 Python 問題
- [ ] 記錄下速度體感（第一個 token 要多久？每秒幾 token？）

### 第三週：延伸到方向 A / B
- [ ] 挑一個延伸方向開始做
- [ ] 最簡單的是方向 A（email 分類器）：寫個 Python 腳本，讀 Gmail → call Pi → 回傳分類
- [ ] 中難度是方向 B（media-index 邊緣版）：先跑 Whisper tiny 轉一段 YouTube 音訊

### 第四週：加感測器（如果要走機器人路線）
- [ ] 學 GPIO 基礎（LED 閃爍、按鈕讀取）
- [ ] 加 motor driver board 和馬達
- [ ] 第一個「Gemma 4 決定左右轉」的 demo

### 推薦學習資源
- [Raspberry Pi 官方 Getting Started](https://www.raspberrypi.com/documentation/computers/getting-started.html)
- [LM Studio 官方 docs](https://lmstudio.ai/docs)
- [Adafruit Learn System](https://learn.adafruit.com/)（感測器 / HAT 教學）
- [Zero to MVP YouTube 頻道](https://www.youtube.com/@zerotomvp) — 這個影片的頻道，有其他相關教學

---

## 八、誠實的預期管理

別期待太高的地方：

1. **速度不是給你用來「聊天」的**——影片作者實測：Python 函式問題約 6 分鐘、三個 web app ideas 約 5 分鐘。這個速度給「腳本 / 自動化」是可以的，給「即時問答」會讓你想捶桌子。
2. **E2B 的推理深度有限**——4.5B 參數模型，複雜 coding / 長鏈邏輯不如 Claude Opus / GPT-5。它適合做分類、抽取、簡單摘要、短對話。
3. **中文能力是個未知數**——E2B 中文 benchmark 不多，實際好不好用要自己測。如果不夠好，考慮用英文 prompt 或接 Claude API 做翻譯層。
4. **Pi 5 再怎麼樣還是 Pi**——不能期待它像 5080 那樣跑 26B 模型。它是「入門級、可重現、好玩」的定位，不是最強。

但也別太悲觀：

1. **$150 的門票買一個本地 AI 節點**，這在 2023 年是不可想像的
2. **Apache 2.0 授權**，你可以拿來做任何商業實驗
3. **完全離線**，隱私 100%
4. **學 Pi 5 + Linux + Python 的基礎**會打開一整個 maker 世界

---

## 九、跟 Jones 其他專案的連結

這個專案跟你手上其他東西可以串起來：

- **Jarvis 復活計畫** — Pi 5 是一個可以 7/24 跑、不搶 Mac Mini 記憶體的輕量節點
- **media-index** — Pi 5 可以先做小範圍 POC，證明概念可行
- **Voice-first media interface**（你之前的發想）— Pi 5 + 麥克風 + Whisper 就是最小可行版
- **AI 機器車夢想** — Pi 5 是最經典的「機器人腦」選擇
- **Gemma 4 研究報告** — 這個專案是那份研究的實戰延伸

---

## 十、下一步建議

**如果 Jones 要動手，我建議：**

1. **這週末去 Micro Center Brooklyn**（或 Amazon 下單）買硬體
2. **下週一晚上**把作業系統和 LM Studio 裝起來
3. **下週三**跑通影片內容，測 Gemma 4 E2B 的第一個問題
4. **下週末**挑一個延伸方向（A / B / C / D）開始玩

**如果現在只想先研究：**

- 看原影片：https://youtu.be/kZhAj8--t8w
- 逛 Raspberry Pi 5 官方頁面熟悉規格
- 看 LM Studio CLI 文件
- 等心情來了再買硬體

---

## 十一、關於這份文件

- **作者：** Jarvis（Dispatch 研究 + Mac Code Task 落檔）
- **來源：** Zero to MVP 的 YouTube 影片 + Jarvis 基於 Jones 脈絡的延伸規劃
- **狀態：** 初版。Jones 實際開始做之後，可以把學習筆記、踩坑紀錄、效能實測追加進來
- **這是 living document** — 隨專案進度更新

---

*本報告由 Jarvis（Dispatch 研究 + Mac Code Task 落檔）產出。2026-04-07。*

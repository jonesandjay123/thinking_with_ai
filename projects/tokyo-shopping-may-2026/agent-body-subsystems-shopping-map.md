# 零件對應架構圖，給 agent-body 計畫

> 2026-04-19 | 買東西時，直接想它是在補哪個 subsystem

---

## 為什麼要這樣看

如果只是看到有趣零件就買，很容易買回一堆酷東西，但不一定能組成系統。

你現在更需要的是：

*每買一樣東西，都知道它是在補 agent-body 的哪一塊身體。*

---

## agent-body 的幾個核心 subsystem

我會先把你的機器人助手拆成這幾層。

1. **Brain interface**
2. **Body controller**
3. **Perception**
4. **Expression / feedback**
5. **Locomotion**
6. **Power**
7. **Structure / assembly**
8. **Personality shell**

---

## 1. Brain interface

### 這層是什麼
讓 PC 上的大腦可以呼叫身體。

### 你現在的狀態
已經有概念雛形了，透過 Wi-Fi / HTTP 去控制 body。

### 逛街時應該找什麼
- Wi-Fi 友善的控制板
- 容易接 API / serial / network 的模組
- 有標準通訊方式的開發板

### 對應可買品
- ESP32
- M5Stack
- 有網路能力的 controller

---

## 2. Body controller

### 這層是什麼
真正接感測器、螢幕、馬達，負責即時控制的那顆板子。

### 逛街時應該找什麼
- 主控板
- 擴充板
- GPIO 資源夠用的模組
- 小而好裝的板子

### 對應可買品
- ESP32 Dev Board
- M5Stack Core / Atom / Stick
- 擴充底座 / hub / unit

---

## 3. Perception

### 這層是什麼
機器人怎麼感知世界。

### 可拆成幾種感官
- 距離
- 姿態
- 聲音
- 視覺
- 光線 / 接近 / 觸碰

### 對應可買品
- 超聲波距離感測器
- ToF 感測器
- IMU
- 麥克風模組
- Camera module
- IR / proximity sensor
- Button / touch sensor

### 為什麼重要
如果你未來想讓它不只是被動擺動，而是真的像助手，perception 一定要慢慢補起來。

---

## 4. Expression / feedback

### 這層是什麼
機器人怎麼把自己的狀態表達出來。

### 你現在已經有的
- OLED
- Servo 動作

### 對應可買品
- OLED / 小螢幕
- RGB LED / LED ring
- 喇叭模組
- 小型 servo
- 可動配件

### 這層的意義
這層會讓它不只是工具，而是開始有 presence。

---

## 5. Locomotion

### 這層是什麼
機器人怎麼移動。

### 你未來最可能的方向
車輪底盤版。

### 對應可買品
- TT gear motor
- 輪胎
- 底盤
- 馬達驅動板
- 編碼器
- 萬向輪

### 這層的核心問題
- 走得動嗎？
- 轉得穩嗎？
- 能不能控制方向？
- 重量和供電扛不扛得住？

---

## 6. Power

### 這層是什麼
整個 body 的電力系統。

### 對應可買品
- 電池盒
- 電源模組
- buck converter
- 開關模組
- 供電分配配件

### 為什麼重要
做到會動的小車後，供電很快會變成真正的瓶頸。

---

## 7. Structure / assembly

### 這層是什麼
所有東西怎麼固定、怎麼維修、怎麼拆。

### 對應可買品
- 螺絲組
- 黃銅熱熔 inserts
- 墊片
- 磁鐵
- 接頭
- 配線材料
- 支架
- 軸承

### 這層常被低估
但它決定你做出來的是「像 prototype」還是「像一坨線」。

---

## 8. Personality shell

### 這層是什麼
機器人的外觀、角色感、可愛度、存在感。

### 你其實已經在做了
招財貓手臂就是超明顯的例子。

### 對應可買品
- 可愛外殼靈感
- LEGO / 模型做結構參考
- 可動裝飾件
- 小螢幕、燈光、表情件

### 為什麼它不是可有可無
因為對 personal robot assistant 來說，personality 會直接影響你願不願意長期跟它互動。

---

## 逛街時怎麼用這份

每看到一個東西，就問：

- 它是在補哪個 subsystem？
- 這個 subsystem 現在是不是我的短板？
- 它能不能和我現有系統接起來？

### 如果答案是這種
- 補短板
- 很快可接
- 通用性高

那就很值得買。

### 如果答案是這種
- 很酷但不知道裝哪
- 和現有架構難整合
- 規格很特別，可能變孤兒

那就先不要急著買。

---

## 以你現在的階段，我建議優先補哪幾層

### 第一優先
1. Perception
2. Locomotion
3. Structure / assembly

### 第二優先
4. Power
5. Expression / feedback

### 第三優先
6. Personality shell
7. 更多進階 brain interface 花樣

原因很簡單，因為你現在最缺的是：
- 讓身體看見 / 聽見 / 感知
- 讓身體開始移動
- 讓整套東西裝得更像系統

---

## 一句話總結

你這次在東京買東西時，最好的思考方式不是：

*這個零件酷不酷？*

而是：

*這個零件是在幫我的 agent-body 長出哪個器官？*

# 東京採購指南，給 agent-body 計畫

> 2026-04-19 | 由 Jarvis 依據 `agent-body` 專案現況補寫
> 
> 目標：讓 Jones 在 2026/05/01-05/10 東京行期間，能用「做自己的機器人助手」這個視角去逛店、看零件、挑道具，而不是只靠現場感覺亂買。

---

## TL;DR

如果這趟東京行要順便為 `agent-body` 補貨，我建議把優先級想成這樣：

### 第一優先，最值得買
- **ESP32 / M5Stack 生態模組**，因為跟你現在的專案最直接連得上
- **感測器與輸入模組**，例如超聲波、麥克風、IMU、ToF 距離感測
- **輪子底盤相關零件**，例如 TT 馬達、輪組、馬達驅動板、萬向輪、電池盒
- **機構材料與工具小物**，例如杜邦線、接頭、螺絲組、軸承、熱縮套、麵包板、電源模組

### 可先記住的品牌 / 官方站
- **M5Stack**: <https://shop.m5stack.com/>
- **Switch Science**: <https://www.switch-science.com/>
- **秋月電子通商**: <https://akizukidenshi.com/>
- **千石電商**: <https://www.sengoku.co.jp/>
- **Marutsu**: <https://www.marutsu.co.jp/>
- **LEGO Technic**: <https://www.lego.com/ja-jp/themes/technic>
- **Tamiya**: <https://www.tamiya.com/japan/>
- **KONDO**: <https://kondo-robot.com/>
- **Futaba**: <https://www.futabausa.com/>

### 第二優先，很適合東京逛的
- **高品質日本伺服馬達**，像 Futaba、KONDO 這類
- **3D 列印輔助零件**，例如黃銅熱熔銅柱、螺母嵌件、磁鐵、快拆結構件
- **機器人 kit / 教育平台**，拿來參考架構，不一定是為了直接用

### 第三優先，可看可不買
- LEGO、模型、可動結構玩具，主要拿來找 **機構靈感**
- 完整機器人套件，除非你真的想買來拆研究，否則先不用急著重押

---

## 我對你這個計畫現在的理解

你不是要做一台一開始就很完整的 humanoid 機器人。

你現在比較像是在走這條路：

```text
本地 LLM / PC 當大腦
        ↓
ESP32 / M5Stack 當 body controller
        ↓
感測器 / 螢幕 / 馬達 / 麥克風 / 輪子底盤
        ↓
做出一台可被 AI 呼叫、可與環境互動的機器人助手
```

所以這趟採購的核心，不是買「最炫的成品」，而是買：

1. **現在馬上能接到 agent-body v1/v2 的零件**
2. **能幫你理解 robot body 架構的模組**
3. **將來車輪底盤版可以沿用的基礎件**

---

## 建議採購框架，不要亂買

我建議你用 5 個籃子來想。

### 1. 控制核心類
你已經在用 ESP32，所以這一類是最穩的延伸。

優先看：
- **ESP32 開發板備品**
- **M5Stack Core / Atom / Stick 系列**
- **有螢幕或內建 IMU 的 ESP32 模組**
- **適合做小車/邊緣控制的控制板**

可參考產品 / 系列：
- **M5Stack Core2**: <https://shop.m5stack.com/products/m5stack-core2-esp32-iot-development-kit-v1-3>
- **M5StickS3**: <https://shop.m5stack.com/products/m5sticks3-esp32s3-mini-iot-dev-kit>
- **ATOM Matrix**: <https://shop.m5stack.com/products/atom-matrix-esp32-dev-kit-v1-1>
- **StackChan**（超值得看，因為它很接近 embodied AI 桌上小機器人概念）: <https://shop.m5stack.com/products/stackchan-kawaii-co-created-open-source-ai-desktop-robot>

你在店裡可以找的關鍵詞：
- ESP32
- M5Stack
- Dev Kit
- Core
- Atom
- Stick
- Unit

### 2. 感測器類
你已經提到超聲波了，這條線很對。接下來你其實在補的是「身體的感官」。

優先看：
- **超聲波距離感測器**，避障入門
- **ToF 距離感測器**，比超聲波更精準、更小型
- **IMU 姿態感測器**，讓你知道車體傾斜、加速度、方向
- **麥克風模組**，之後做聲控或喚醒
- **攝影機模組**，未來做視覺
- **人體接近 / 紅外線 / 光感測**

可參考產品 / 系列：
- **M5Stack ToF Unit (VL53L0X)**: <https://shop.m5stack.com/products/chain-tof-vl53l0>
- **Switch Science 感測器生態入口**: <https://www.switch-science.com/>
- **秋月電子新商品頁**，很適合現場比對型號: <https://akizukidenshi.com/catalog/e/enewall_dT/>

店內關鍵詞：
- ultrasonic
- distance sensor
- ToF
- IMU
- gyro
- microphone
- camera module
- IR sensor

### 3. 動力與移動類
如果你未來要做「車輪底」機器人，這一類其實超重要。

優先看：
- **小型 DC 馬達 + 輪胎組**
- **TT motor gear box** 這種常見入門車輪馬達
- **馬達驅動板**，例如 L298N、TB6612FNG 這類
- **萬向輪 / caster wheel**
- **編碼器馬達**，未來做比較準的移動控制
- **電池盒 / 供電模組 / 降壓模組**

關鍵詞：
- motor driver
- gear motor
- wheel
- chassis
- encoder
- battery holder
- buck converter

### 4. 結構與裝配類
這一類很不 flashy，但超實用，尤其你又有 3D 印表機。

優先看：
- **螺絲 / 螺母 / 墊片 assortments**
- **黃銅熱熔螺母 inserts**，3D 列印件超好用
- **磁鐵**，做可拆式外殼很好用
- **軸承 / 小轉軸 / 聯軸器**
- **杜邦線 / JST 接頭 / 轉接頭**
- **熱縮套管 / 束線帶 / 麵包板 / 洞洞板**

這類東西很容易被忽略，但常常是最提升 DIY 品質的。

### 5. 靈感參考類
這一類未必直接接電，但可以讓你想出 body 的形式。

可以看：
- **LEGO Technic**，看齒輪、轉向、連桿、底盤
- **可動模型 / 組裝模型**，看關節與外殼拆分
- **教育型機器人套件**，看模組化設計
- **小型 RC 車**，看底盤與輪系配置

---

## 東京適合去哪裡逛

## 1. 秋葉原，主戰場
如果你這趟只能選一個地方為 `agent-body` 補貨，那就是秋葉原。

先記幾個站：
- 秋月電子通商: <https://akizukidenshi.com/>
- 千石電商: <https://www.sengoku.co.jp/>
- Marutsu: <https://www.marutsu.co.jp/>
- Switch Science: <https://www.switch-science.com/>

### 最值得逛的店型

#### 秋月電子通商（Akizuki Denshi）
很像 DIY 電子人的寶庫。
適合買：
- 感測器
- 電源模組
- 各種基礎零件
- ESP32 周邊
- 接頭、線材、小型模組

參考：
- <https://akizukidenshi.com/>

#### 千石電商（Sengoku Denshi）
品項很雜很全，很適合「我已經有概念，想現場比比看」的逛法。
適合買：
- 開發板
- 線材
- 感測器
- 電子工具
- 小型馬達與配件

參考：
- <https://www.sengoku.co.jp/>

#### マルツ秋葉原本店（Marutsu）
相對更整齊、更像工程型商店。
適合買：
- Arduino / ESP32 / Raspberry Pi 類
- 開發工具
- 模組與測試用品

參考：
- <https://www.marutsu.co.jp/>

#### Switch Science 相關商品
如果現場看到 M5Stack、Seeed、micro:bit、各種 maker 生態，很值得留意。

參考：
- <https://www.switch-science.com/>
- <https://shop.m5stack.com/>

### 秋葉原採購重點
我會建議你在秋葉原把目標放在：
- 能讓 `agent-body` 下一版前進的核心模組
- 很煩但很常缺的接線/裝配件
- 一兩個能擴展你想像力的感測器

不要第一天就衝太多「大套件」。
先買可快速上手的東西，回去比較容易串進現有系統。

---

## 2. Yodobashi Akiba，適合做總覽與比價
Yodobashi 比較不是深度零件街，但很適合你：
- 先建立「市場上有哪些類型」的概念
- 看完整機器人玩具、工具、相機、麥克風、3D 列印周邊
- 順便看 LEGO、教育型機器人、工具收納

參考：
- <https://www.yodobashi.com/store/0018/>

適合看：
- 麥克風、錄音設備
- 小型工具
- 相機與視覺相關設備
- LEGO / 組裝模型 / 教育玩具

---

## 3. Tokyu Hands / Hands，適合補結構與工具靈感
這裡不一定有你要的電子零件，但非常適合補：
- 手工具
- 收納盒
- 膠材
- 切割工具
- DIY 小五金
- 機構靈感

如果你已經開始做 body 外殼、車體、安裝架，這類店會很有幫助。

參考：
- <https://info.hands.net/en/list/tokyo/>

---

## 4. LEGO Store / LEGO 區，拿機構靈感
我覺得你想到 LEGO 很合理，而且不是幼稚，是很對。

尤其是：
- **LEGO Technic**
- 有齒輪、轉向、懸吊、連桿機構的系列
- 履帶、工程車、越野車那一類

它對你最有價值的點不是「買來當玩具」，而是：
- 理解模組化結構
- 理解輪系與傳動
- 理解外殼如何不妨礙維修
- 理解 body shell 和 mechanical frame 怎麼分離

參考：
- LEGO Japan: <https://www.lego.com/ja-jp>
- LEGO Technic: <https://www.lego.com/ja-jp/themes/technic>

可優先看這類：
- **Jeep Wrangler Rubicon SUV**: <https://www.lego.com/ja-jp/product/jeep-wrangler-rubicon-suv-42227>
- **Bugatti Chiron Pur Sport**: <https://www.lego.com/ja-jp/product/bugatti-chiron-pur-sport-hypercar-42222>
- **Volvo EC500 Hybrid Excavator**: <https://www.lego.com/ja-jp/product/volvo-ec500-hybrid-excavator-42215>

如果你真的想買，優先看：
- 越野車
- 工程車
- 有轉向與齒輪傳動展示的款

---

## 5. 模型店 / RC 店，拿車體與外觀靈感
雖然你未必需要買成品，但這類店很適合看：
- 小車底盤比例
- 輪距、離地高、輪胎尺寸
- 車殼怎麼固定
- 伺服轉向與輪系配置

如果你看到 Tamiya、RC car、mini 4WD、機構展示品，也值得觀察。

Tamiya 參考：
- <https://www.tamiya.com/japan/>

---

## 我實際會建議你買什麼

## A. 最實際的 starter shopping list
這份是「買回來就有很高機率接進專案」的版本。

### 控制與模組
- 1 到 2 塊備用 **ESP32 開發板**
- 1 套 **M5Stack** 主控或小型模組
- 1 個 **攝影機模組**（如果看到合適的 ESP32 camera / 小型 camera unit）
- 1 到 2 個 **麥克風模組**

### 感測器
- 1 到 2 種 **距離感測器**
  - 超聲波
  - ToF
- 1 個 **IMU**
- 1 個 **紅外或接近感測器**

### 移動系統
- 一組 **小型輪車底盤零件**
- 2 個或 4 個 **齒輪馬達**
- 1 個 **馬達驅動板**
- 若有看到適合的，買 **萬向輪**

### 供電與裝配
- **降壓模組 / buck converter**
- **電池盒 / 開關模組**
- **JST 接頭**
- **杜邦線 assortments**
- **熱熔銅柱 / inserts**
- **小螺絲組**

---

## B. 如果你想往「可愛機器人助手」方向靠
這類東西也值得看：
- 小型圓形螢幕或可愛 OLED 模組
- RGB LED / light ring
- 小喇叭模組
- 麥克風模組
- 小型可動耳朵/手臂用 servo
- 磁吸式外殼固定結構

因為你這個計畫不只是功能機器，也有一點 personality / presence 的味道。
這很像你會做的方向。

---

## C. 如果你要為「車輪底盤版」提早布局
那就特別關注：
- 馬達驅動
- 電池供電穩定
- 底盤結構
- 重心配置
- 感測器安裝位置
- 線材整理

這代表你在逛的時候，不只是問「這模組酷不酷」，而是問：

- 它裝哪裡？
- 線怎麼走？
- 重量會不會太重？
- 供電怎麼分？
- 維修時能不能拆？

這種思考方式會讓你買得很準。

---

## 幾個值得先看品牌 / 產品方向

## 1. M5Stack
對你這種想把 AI、IoT、實體互動串起來的人，很值得看。

參考：
- <https://shop.m5stack.com/>
- <https://docs.m5stack.com/>

可留意方向：
- Core 系列
- Atom 系列
- Stick 系列
- 距離感測 Unit
- Servo / Motor Unit
- Camera Unit

## 2. Seeed Studio / XIAO
如果現場有看到，日本 maker 店常會有。

參考：
- <https://www.seeedstudio.com/xiao-series-page>

## 3. Futaba / KONDO 伺服
如果你想從玩具級 servo 往更穩定一點的世界看，這兩個方向很值得做功課。

參考：
- Futaba: <https://www.futabausa.com/>
- KONDO: <https://kondo-robot.com/>

## 4. LEGO Technic
拿來看機構非常好。

參考：
- <https://www.lego.com/ja-jp/themes/technic>

## 5. Tamiya
如果你想從車體、底盤、輪組、 RC 結構拿靈感，這個品牌很適合。

參考：
- <https://www.tamiya.com/japan/>

---

## 我建議你在東京的逛法

## 方案 A，最實用的一天
### 秋葉原 agent-body day
上午：
- 先逛 **秋月電子 / 千石 / Marutsu**
- 只鎖定電子零件、感測器、開發板、供電、線材

下午：
- 去 **Yodobashi Akiba**
- 補看麥克風、相機、工具、LEGO、教育機器人

晚上：
- 回飯店整理戰利品
- 手機記錄每個零件是拿來幹嘛的

## 方案 B，靈感加購版
- 秋葉原買核心零件
- 另一天去 **LEGO / 模型 / Hands** 看機構與外觀靈感

這樣會比全部塞同一天更舒服。

---

## 逛的時候怎麼判斷要不要買

你可以快速問自己 5 個問題：

1. **這東西能不能在 agent-body 下一版 30 天內用到？**
2. **它是在補能力缺口，還是只是覺得酷？**
3. **美國是不是也很容易買到？如果是，就不一定要在東京扛。**
4. **它是通用零件，還是會變成孤兒零件？**
5. **它能幫我學到一個新的 body subsystem 嗎？**

如果有 3 題以上都是好答案，就值得買。

---

## 對 LEGO 這件事，我的看法

我覺得 LEGO 很值得，但用途要想對。

### 值得買的情況
- 你想研究底盤、輪系、外殼、轉向結構
- 你想用它當 prototype thinking tool
- 你想快速用現成機構測一個想法

### 不值得買的情況
- 你把它當成電子控制主線
- 你期待它直接變成機器人本體主架構

所以我的建議是：

*把 LEGO 當「機構靈感與快速原型語言」，不要把它當最終 robot platform。*

---

## 我替你排的採購優先級

### S 級，看到合適就買
- ESP32 / M5Stack 模組
- 距離感測器（尤其 ToF / 超聲波）
- 馬達驅動板
- 輪車底盤零件
- 結構裝配小物

### A 級，有順路就買
- 麥克風模組
- Camera module
- IMU
- 高品質 servo
- 3D 列印友善的固定件

### B 級，偏靈感型
- LEGO Technic
- Tamiya / RC 底盤參考品
- 教育型 robot kit
- 可動模型

---

## 我對這趟旅行的總結建議

對，我真的覺得「做出一台屬於自己的車輪底機器人助手」不是夢想了。

而且你現在的位置很棒，因為你不是在空想，你已經開始建立：
- body controller 概念
- 感測與輸出概念
- 機構與供電概念
- 實際上手經驗

這趟東京如果逛對，你拿回來的不只是零件，還會是：
- 對機器人身體怎麼拆系統的直覺
- 對哪些模組值得長期押注的概念
- 對未來 body v2 / v3 的具體畫面

所以我建議你把這趟當成：

*東京版 field research + shopping mission for agent-body.*

---

## 下次可擴充的內容

如果你要，我下一版可以再幫你補這幾種其中一種：

1. **店家地圖版**，直接列秋葉原值得逛的順路動線
2. **採購清單版**，做成 checkbox shopping list
3. **預算版**，分成小買、中買、放手買三個等級
4. **零件對照版**，把每種零件對應到 `agent-body` 未來哪個子系統

---

*附註：本文件重點是幫 Jones 用 agent-body / personal robot assistant 的視角逛東京，不追求完全價格最優，而追求「買回來真的有助於專案演進」。*
# 零度解說：Pearl（PRL）顯卡挖礦教程

Source: https://youtu.be/Ke8LEe6x9zA?si=GwlhX75QOVcuztMq

Video ID: `Ke8LEe6x9zA`

Channel: 零度解說

Title: 顯卡挖礦又回來了？爆火的 Pearl（PRL）珍珠幣，單卡到集群全教程！（2026最新版）| 零度解說

Upload date: 2026-06-03

Duration: 13:53

Extraction note: YouTube did not expose manual or automatic caption tracks for this video. This note is based on local `whisper-cpp` transcription with `ggml-base`, then manually summarized. The transcript has obvious recognition errors, so exact command names, URLs, pool names, prices, and hashrates should be verified from the original video and linked resources before use.

Important caveat: This is a content summary for later AI discussion, not mining, trading, security, tax, or investment advice. The video discusses crypto mining software, wallet custody, exchange/liquidity, and disabling security tools; all of those carry operational and financial risk.

Follow-up note:

- [Jovix RTX 5080 PRL mining and alternatives evaluation](jovix-rtx-5080-followup.md)

## Why This Video Matters

這支影片不是單純「幣圈消息」，而是一個完整的 PRL/Pearl 顯卡挖礦上手流程：先說收益誘因，再示範錢包、礦池、Windows 單卡挖礦、HiveOS 叢集挖礦、查收益、提幣/變現、以及不同顯卡的功耗與算力參考。

值得後續討論的點在於：它把「閒置 GPU 算力」包裝成 AI + 區塊鏈收益機會，但影片中的核心判斷都高度依賴即時幣價、礦池穩定性、電費、硬體折舊、安全風險和流動性。對 AI 討論來說，這是一個很好的案例：如何拆解高收益教學影片背後的真實風險模型。

## Executive Summary

影片主張 Pearl / PRL 是近期礦工圈很火的 GPU 挖礦項目，敘事上不是傳統純 hash 的 PoW，而是把閒置 GPU 算力導向 AI 推理/算力市場。創作者先展示自己的多張 GPU 測試收益，再示範一般人如何從單張 NVIDIA 顯卡開始挖，最後延伸到多卡機群管理。

流程大致分成三段。第一段是建立 PRL 錢包，保存助記詞，拿到挖礦收款地址。第二段是 Windows 單機挖礦：選礦池/礦工軟體，修改設定檔中的錢包地址、礦池節點和 worker 名稱，啟動後看功耗、溫度和算力。第三段是 HiveOS 叢集挖礦：註冊 HiveOS、加入錢包、建立 flight sheet、設定自訂 miner/pool、製作 HiveOS USB 開機碟，讓礦機上線管理多張卡。

影片最後展示收益查詢、提現門檻和變現路徑，聲稱可把 PRL 兌成 USDT 或美元，並列出不同顯卡的大致功耗/算力。創作者強調他自己的電費近乎免費，所以收益看起來很漂亮；這也是整支影片最重要的限制條件。

## Operating Notes

- 影片開頭聲稱：35 張 RTX 3070 一天約 70 美元左右，單張 RTX 4090 一天約 9 美元左右；後段又用 35 張 3070 加 1 張 4090、電費為 0 的假設試算一天約 80 美元。這些都應視為影片當下聲稱，不應直接外推。
- PRL/Pearl 被描述為 AI + blockchain 項目，透過 GPU 挖礦執行 AI 推理或算力相關工作，而非單純傳統 hash 計算。
- 錢包建立流程重點是保存 12 個助記詞。影片提醒不要截圖或存成容易外洩的檔案，忘記/丟失助記詞就找不回資產。
- 單卡挖礦示範偏向 Windows + NVIDIA 顯卡，影片說 20 / 30 / 40 / 50 系列 N 卡都可嘗試。
- 礦池選擇上，影片比較多個選項，提到有的抽水約 1%，有的約 3%，並推薦使用他實測較穩定、手續費約 1% 的礦池/礦工方案。
- 設定檔主要改三類資訊：自己的 PRL 錢包地址、離自己最近的礦池節點（例如歐洲、加拿大、新加坡）、worker 名稱。
- 影片提到挖礦軟體可能被防毒軟體誤殺，並示範放行/關閉防護。這是明顯的安全風險點，不能只按教學操作。
- 挖礦啟動後，創作者觀察功耗、算力、溫度和是否有 accepted share，以確認是否正常工作。
- 查收益時，影片示範把錢包地址貼到礦池/查詢頁面，看目前算力、挖到的 PRL、worker 狀態和付款紀錄。
- 影片提到最低提現門檻約 5 PRL；示範收到約 5.1 PRL 後，等待錢包同步區塊資料才顯示餘額。
- 多卡叢集部分推薦 HiveOS：建立帳號、添加 wallet、建立 flight sheet、選 PRL、選錢包、設定 pool/miner 參數，再把 HiveOS image 燒到 USB 讓礦機開機。
- HiveOS 製作 USB 的坑：影片提醒不要直接把下載包寫入 USB，應先解壓出 `.img` 檔，再用寫盤工具燒錄。
- 影片後段示範 8 張 GPU 都在線，總算力約 266，已挖到一些 PRL，但尚未達最低提現門檻。
- 變現路徑被描述為把 PRL 在出售/交易平台兌換成 USDT 或美元；影片當下提到價格約 0.73 美元/PRL。這個價格高度易變，必須即時查證。
- 顯卡參考：RTX 3060 功耗約 108W；RTX 3070 算力約 58、功耗約 168W；RTX 4090 算力可到 200 多。這些數字取決於超頻、驅動、礦工版本、電源效率和環境溫度。

## Risk Model

這支影片真正需要拆的是「毛收益」和「淨收益」。毛收益只看幣價與挖到的數量；淨收益要扣電費、平台抽水、硬體折舊、故障率、散熱、噪音、稅務、交易滑價、提現手續費，以及幣價下跌或礦池停止支援的風險。

安全面也不能忽略。任何要求下載礦工、修改設定、放行防毒、連到礦池、管理私鑰/助記詞的流程，都有被植入惡意程式、夾帶抽水、私鑰外洩或下載假版本的可能。影片是教學，不是供應鏈安全審計。

投資面最大問題是「收益窗口可能很短」。若 PRL 因話題爆紅吸引大量 GPU 算力，難度、獎勵、幣價、流動性都可能快速變化。影片發布於 2026-06-03，任何具體收益/價格在幾天後都可能失真。

## AI Discussion Prompts

- PRL/Pearl 的官方白皮書或技術文件是否真的能支持「AI 推理 + 區塊鏈算力市場」這個敘事？
- 影片中的收益數字，如果用紐約/美國住宅電價、商業電價、或太陽能邊際電力分別計算，淨收益會變成多少？
- RTX 3060 / 3070 / 4090 的 ROI 應該怎麼算？要把二手殘值、風扇損耗、電源效率和散熱成本放進模型。
- PRL 的流動性在哪裡？日成交量、買賣價差、提現限制、交易所風險如何？
- 下載礦工與關閉防毒的安全風險如何控管？能否用隔離機、乾淨 OS、只讀錢包地址、硬體錢包或 sandbox 降低風險？
- 這類「AI + mining」項目是真需求驅動，還是短期幣價/敘事驅動？該怎麼判斷？
- 如果只是想利用閒置 GPU，跟出租算力、跑本地模型服務、做渲染/AI 批次任務相比，PRL 挖礦是否更合理？
- 這支影片有沒有典型高收益教學影片的紅旗：強調當下收益、弱化電費與折舊、依賴特定下載連結、避談長期流動性？

## Bottom Line

影片提供的是「如何開始挖 PRL」的實作路線圖，但真正值得保留下來的是分析框架：不要被單日收益吸走注意力，要回到電費、幣價、難度、硬體、資安、流動性和稅務。若後續要跟 AI 深聊，建議先讓 AI 幫你把影片中的每個收益聲稱轉成可調參數的 ROI 表，再逐項查證 PRL 官方資料與市場資料。

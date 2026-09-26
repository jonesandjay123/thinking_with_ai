# 擬真 3D 人體建模引擎：開源 Repo、論文與 AI Agent 實戰評估

- 研究日期：2026-09-25（美東）
- 證據基礎：搜尋索引摘要與官方專案／repo 頁面；凡未在官方頁面直接確認的事實，以下標註「待驗證」。GitHub star 數本次未能取得，一律不引用。

## TL;DR（先給結論）

1. **「人體建模引擎」確實存在，而且是學界標準**：SMPL-X（CVPR 2019）是事實上的參數化人體標準——身體＋手部關節＋臉部表情，一組參數就能驅動 10,475 個頂點、54 個關節。Python 用 `smplx` 套件載入，Blender 有 Meshcapade 的外掛可以直接拉進場景。
2. **但這些引擎只管「形狀和骨架」，不管「長得像不像真人」**：皮膚、頭髮、衣服、毛孔細節是完全不同的技術層，要靠擴散模型貼圖、3D 高斯潑濺（3DGS）、神經渲染去補。
3. **AI agent 驅動已經有具體證據**：ChatPose（CVPR 2024）把 SMPL 姿態做成 LLM 的 token，LLM 可以直接用文字生成 SMPL 參數；BlenderMCP 讓 LLM agent 直接在 Blender 裡建模、調材質、跑 Python。
4. **誠實答案**：今天 AI agent 可以串出一條「原型管線」（SMPL-X 身體 → 文字生成動作 → Blender 腳本 → 擴散貼圖 → 3DGS 渲染），做出看起來很像的靜幀；但**沒有任何方法能從一段文字一次產出「遊戲可用的、真正照片級、身份一致」的人體**——正確的臉和手、髮絲級頭髮、皮膚微結構、分層衣服、乾淨的拓樸／UV／LOD、極端姿態下不崩的綁定，這整套目前拼不起來。
5. **隱藏限制是授權**：SMPL、SMPL-X、STAR、SUPR、MANO 的模型檔案都要註冊、限非商業學術研究用；商用要另外談授權。寫 code 的 loader 和模型資產是兩回事。

## 一、參數化人體模型：真正的「引擎」

這些就是你要找的「3D 引擎的人體建模引擎」——不是渲染器，而是用少量參數（體型＋姿態）精確控制人體網格的數學模型。

### SMPL
- 論文：Loper 等，"SMPL: A Skinned Multi-Person Linear Model"，ACM TOG / SIGGRAPH Asia 2015
- 模型官網：https://smpl.is.tue.mpg.de/
- 載入器：https://github.com/vchoutas/smplx（SMPL／SMPL-X／SMPL+H 共用）
- 授權：模型檔案限非商業學術研究、需註冊；商用另談（待驗證最新條款）

### SMPL+H（身體＋手）
- 身體接上 MANO 關節手；源自 Romero 等 "Embodied Hands"，SIGGRAPH Asia 2017
- 用同一個 `smplx` 載入；模型下載：https://mano.is.tue.mpg.de/
- 授權：同 SMPL 的非商業學術條款

### SMPL-X（目前的主力標準）
- 論文：Pavlakos 等，"Expressive Body Capture: 3D Hands, Face, and Body from a Single Image"，CVPR 2019
- 統一模型：身體＋關節手＋表情臉；約 10,475 頂點、54 關節
- Repo：https://github.com/vchoutas/smplx（`pip install smplx[all]`）
- 授權：限非商業科學研究（確切條款待驗證）

### STAR
- 論文：Osman 等，"STAR: A Sparse Trained Articulated Human Body Regressor"，ECCV 2020
- 稀疏／局部姿態修正，定位為 SMPL 後繼；號稱 14,000 受試者、完整 300 主成分體型空間
- Repo：https://github.com/ahmedosman/STAR
- 授權：repo 標示 "PS:License 1.0"，商業限制待驗證

### GHUM / GHUML
- 論文：Xu 等，"GHUM & GHUML: Generative 3D Human Shape and Articulated Pose Models"，CVPR 2020
- 身體＋精細手＋臉部表情；GHUM 10,168 頂點、GHUML 3,194；以 60,000+ 組態訓練的非線性形變管線
- 論文：https://openaccess.thecvf.com/content_CVPR_2020/html/Xu_GHUM__GHUML_Generative_3D_Human_Shape_and_Articulated_Pose_CVPR_2020_paper.html
- 程式碼應在 Google Research monorepo，確切位置待驗證；散佈疑似需申請、限研究（待驗證）

### SUPR
- 論文：Osman 等，"SUPR: A Sparse Unified Part-Based Human Representation"，**ECCV 2022**（注意：常被誤寫為 2023）
- 整具身體聯合訓練、可拆分成頭／手／腳子模型；腳趾、接觸條件腳部形變；基於 SMPL-X 拓樸
- Repo：https://github.com/ahmedosman/SUPR；論文：https://arxiv.org/pdf/2210.13861
- 授權：疑似 MPI 非商業條款（待驗證）

### MANO（手部）
- 論文：Romero 等，"Embodied Hands"，ACM TOG / SIGGRAPH Asia 2017
- 參數化關節手部形狀與姿態
- 實作：https://github.com/otaheri/mano；PyTorch 層：https://github.com/hassony2/manopth
- 授權：非商業科學研究

### FLAME（臉部／頭部）
- 論文：Li 等，"Learning a Model of Facial Shape and Expression from 4D Scans"，ACM TOG / SIGGRAPH Asia 2017
- 參數化頭部：形狀、表情、下顎、脖子；已被整合進 SMPL-X
- 範例 repo：https://github.com/soccergame/flame-fitting；官方 repo 與現行授權待驗證

**授權重點（重要）**：寬鬆授權的通常只是「載入程式碼」，「模型檔案」是另外管制的（註冊＋非商業學術研究）。任何商業或產品用途都需要另外授權。

## 二、開源工具鏈

| 工具 | 連結 | 授權 | 維護狀況 |
|---|---|---|---|
| `smplx` Python 套件 | https://github.com/vchoutas/smplx | 非商業科學研究 | 未完整審計（待驗證） |
| Meshcapade Blender 外掛 | https://github.com/Meshcapade/SMPL_blender_addon | 程式碼 GPL-3.0；模型檔沿用 SMPL 學術／商業條款 | 研究日前約 191 天有更新（待驗證） |
| EasyMocap（浙大） | https://github.com/zju3dv/EasyMocap | 教育／研究／非營利；商用需許可（待驗證官方 LICENSE） | 快速入門文件在研究日前約 10 天有編輯 |
| MMHuman3D | https://github.com/open-mmlab/mmhuman3d | 程式碼 Apache-2.0；人體模型條款另計 | — |
| MMPose | https://github.com/open-mmlab/mmpose | Apache-2.0 | — |
| XRMoCap | https://github.com/openxrlab/xrmocap | Apache-2.0 | — |
| SMPLify-X | https://github.com/vchoutas/smplify-x | 非商業研究 | — |
| human_body_prior | https://github.com/nghorbani/human_body_prior | 待驗證 | — |

- **EasyMocap**：無標記單目／多視角動捕、相機校準、關鍵點三角測量、SMPL／SMPL-X／MANO 擬合。https://github.com/zju3dv/EasyMocap
- **MakeHuman／MPFB**：MakeHuman 社群仍在活躍開發，目前主力是 **MPFB（MakeHuman Plugin for Blender）**：v2.0.13（2025-12-25，OpenPose 功能）、v2.0.12（2025-10-18）、v2.0.11（2025-09-22，幾何節點頭髮、Mixamo 綁定）；2026 年 2 月新增 viseme／臉部單元／頭髮編輯器資源包。官網稱約 86k 下載。來源：https://github.com/makehumancommunity/makehuman-static-website。寫實程度：美術／遊戲級基礎網格，不是照片級皮膚頭髮。授權待驗證。
- **MB-Lab（ManuelBastioniLAB 分支）**：原版 2018 年停止，社群分支延續；官方連結、授權、現況皆待驗證。

## 三、文字生成 3D 人體與高斯人體

### 擴散模型文字生成 Avatar
- **AvatarCraft**（ICCV 2023）：文字引導神經隱式 Avatar，參數化形狀＋姿態控制，可動畫化。論文：https://arxiv.org/pdf/2303.17606；Repo：https://github.com/songrise/AvatarCraft
- **DreamAvatar**（arXiv 2023）：文字＋SMPL 參數引導，在正則與擺姿空間優化 NeRF。論文：https://arxiv.org/abs/2304.00916；Repo：https://github.com/yukangcao/dreamavatar（發表 venue 待驗證）
- **TADA**（3DV 2024）：文字生成整體可動畫網格＋貼圖，可用自然語言編輯。論文：https://arxiv.org/abs/2308.10899；Repo：https://github.com/TingtingLiao/TADA
- **AvatarVerse**（AAAI 2024）：DensePose 條件擴散，漸進式高解析合成。論文：https://arxiv.org/abs/2308.03610；Repo：https://github.com/bytedance/AvatarVerse
- **DreamHuman**（NeurIPS 2023）：擴散＋神經場＋人體統計先驗的文字可動 Avatar。論文：https://arxiv.org/abs/2306.09329；官方程式碼未找到（待驗證）
- **CHUPA**（arXiv 2023）：前後雙法線圖擴散「雕刻」SMPL-X 先驗網格，補身體／臉部細節。論文：https://arxiv.org/pdf/2305.11870；專案頁：https://snuvclab.github.io/chupa/（venue 待驗證）
- **DreamWaltz-G**（arXiv 2024，TPAMI 2025）：骨架引導分數蒸餾，高斯／網格／隱式混合表示，文字建 Avatar＋全身動畫。論文：https://arxiv.org/abs/2409.17145；Repo：https://github.com/Yukun-Huang/DreamWaltz-G

### 高斯潑濺／神經 Avatar
- **HumanGaussian**（CVPR 2024 Highlight）：結構感知 RGB／深度 SDS、自適應高斯增減。論文：https://arxiv.org/abs/2311.17061；官方 repo 待驗證（搜尋多為分支）
- **GAvatar**（CVPR 2024）：姿態驅動高斯基元＋神經隱式屬性＋SDF 網格提取，號稱 1K 解析度 100 fps。論文：https://arxiv.org/abs/2312.11461；公開程式碼未確認（待驗證）
- **GaussianAvatar**（CVPR 2024）：**單支影片**重建可動 3D 高斯真人（是重建特定真人，不是文字生成）。論文：https://arxiv.org/abs/2312.02134；repo 待驗證
- **Animatable Gaussians**（CVPR 2024）：RGB 影片生成姿態相關前後高斯圖、服裝自適應模板、動態衣物外觀。論文：https://arxiv.org/pdf/2311.16096.pdf；repo 待驗證
- **CharacterGen**（ACM TOG／SIGGRAPH 2024）：單張圖 → 姿態正則化貼圖角色網格，可直接綁定動畫。論文：https://arxiv.org/pdf/2402.17214.pdf；Repo：https://github.com/zjp-shadow/CharacterGen（注意：資料集多為風格化／VRM／動漫，非寫實真人）
- **TeCH**（3DV 2024）：單張圖 → 精細貼圖穿衣人體網格；每人約需 V100 3 小時、需 Replicate API token。Repo：https://github.com/huangyangyi/TeCH
- **TaoAvatar**（CVPR 2025 Highlight）：全身說話 3DGS Avatar，蒸餾到可即時／手機跑，號稱 Vision Pro 上 90 fps。論文：https://arxiv.org/abs/2503.17032v2
- **AniCrafter**（ACM MM 2026）：單圖 3DGS Avatar（經 LHM https://github.com/aigc3d/LHM）→ SMPL-X 動作序列驅動 → 與背景影片合成 → 影片擴散精修。Repo：https://github.com/MyNiuuu/AniCrafter（很新，動手前請直接驗證）

### 2025–2026 新進展（保守看待）
- **PSHuman**（CVPR 2025）：單圖多視角擴散＋重網格的照片級重建；repo 未收錄（待驗證）
- **HumanDreamer-X**（2025）：單圖 3DGS 重建＋修復；僅有二手來源（待驗證）
- **Mon3tr**（arXiv 2026-01）：預建 3DGS Avatar 由單目動作／臉部驅動；僅二手來源（待驗證）
- **SplatShot**（arXiv 2026-05）：單張照片臉部 Avatar，擴散＋3DGS 回饋迴路；僅臉部非全身

**關鍵區分**：重建類（GaussianAvatar、TeCH、PSHuman、單影片 3DGS）從真人照片／影片出發，寫實度最高，但需要拍攝對象且每人要訓練；文字生成類（TADA、AvatarVerse、DreamAvatar、HumanGaussian）有創造力，但身份一致性、解剖正確性、遊戲可用度都較弱；高斯／NeRF 輸出渲染很漂亮，但不是自動可編輯的生產級網格。

## 四、遊戲引擎 NPC 方案（簡述）

- **MetaHuman（Epic）**：專有高寫實數位人創建／綁定生態系，整合於 Unreal Engine；UE 5.6 起創建工具進引擎，2025 年報導稱授權擴及到其他 DCC／引擎。**不開源**。官方文件與現行 EULA 本次未直接讀取（待驗證）。
- **Ready Player Me**：原跨遊戲 Avatar SaaS／SDK（專有）。有第三方 2026-08-31 報導稱其公開平台於 2026-01-31 在 Netflix 收購後下線——**未經官方證實**，引用前請向官方查證。
- **Avaturn**：專有自拍轉 3D Avatar 平台／SDK；風格比 MetaHuman 更卡通；不開源。

## 五、AI Agent 如何驅動這些引擎

### 文字 → 姿態／動作
- **ChatPose**（CVPR 2024）：多模態 LLM 把 SMPL 姿態當成獨立的 signal token，專用投影層把語言 embedding 轉成 SMPL 姿態參數；可從文字和圖片生成 3D 姿態。這是目前 **LLM→文字→SMPL 參數最直接的證據**。專案：https://yfeng95.github.io/ChatPose/；論文：https://openaccess.thecvf.com/content/CVPR2024/html/Feng_ChatPose_Chatting_about_3D_Human_Pose_CVPR_2024_paper.html；Repo：https://github.com/yfeng95/PoseGPT（基於 LLaVA＋LISA；授權待驗證）
- **PoseScript**（ECCV 2022）：自然語言 → 3D 姿態檢索與生成；PoseFix 做文字引導姿態修正。Repo：https://github.com/naver/posescript（CC BY-NC-SA 4.0）
- **HumanTOMATO**（ICML 2024）：文字對齊全身動作生成（含身體／手／臉）。論文：https://arxiv.org/abs/2310.12978；Repo：https://github.com/IDEA-Research/HumanTOMATO
- **MotionGPT**（AAAI 2024）：文字＋姿態 token 化進共享離散詞表，LoRA 微調 0.4% 參數；文字↔動作雙向。論文：https://arxiv.org/abs/2306.10900；專案：https://qiqiapink.github.io/MotionGPT/；官方 repo 未找到，社群管線：https://github.com/zeyuling/motius（待驗證）
- **T2M-GPT**（CVPR 2023）：VQ-VAE 離散動作 token＋自回歸 GPT；HumanML3D 上 FID 0.116。論文：https://arxiv.org/pdf/2301.06052；Repo：https://github.com/Mael-zys/T2M-GPT

### 反向：照片／影片 → SMPL（已成熟）
- **HMR 2.0／4D-Humans**（ICCV 2023）：全 Transformer 單圖 SMPL 姿態形狀恢復＋多人 3D 追蹤。論文：https://arxiv.org/abs/2305.20091v3；Repo：https://github.com/shubham-goel/4D-Humans；專案：https://shubham-goel.github.io/4dhumans/
- **PyMAF-X**：身體／手／臉分部 PyMAF 自適應整合成 SMPL-X 參數（55 關節），像素級網格對齊回饋。論文：https://arxiv.org/pdf/2207.06400；專案：https://hongwenzhang.github.io/pymaf（venue 與官方 repo 待驗證）

### Agent 直接驅動 DCC
- **BlenderMCP**：https://github.com/ahujasid/blender-mcp；官網 https://blendermcp.org/。經 Model Context Protocol 把 Blender 接上 Claude：提示詞輔助建模、場景創建、物件操作、材質、場景檢查、Blender 內任意 Python 執行、視窗截圖、Sketchfab／Poly Haven 資源搜尋、Hunyuan3D 與 Hyper3D Rodin 3D 生成支援。注意：它是**通用場景建模工具**，可以寫角色工作流腳本（如 SMPL 匯入／動畫），但本身不生成寫實人體。社群變體：https://github.com/carlosh7/blender-mcp。
- 對話式 Avatar 控制範例：https://github.com/asanchezyali/talking-avatar-with-ai——GPT 輸出文字＋命名表情／動畫，ElevenLabs 配音、Rhubarb 生成 viseme；驅動的是**現成** Avatar，不是從零創建。

## 六、誠實評估：AI Agent 今天能做到多逼真？

**今天做得到的**：AI agent 已經可以串出一條端到端原型管線——選 SMPL-X／SUPR 體型 → 用文字生成動作（T2M-GPT／MotionGPT／HumanTOMATO）或用語言推理姿態（ChatPose）→ 經 MCP 驅動 Blender → 套擴散生成的貼圖／法線 → 用 NeRF／3DGS 渲染。反向（照片→SMPL）已經很可靠（HMR 2.0、PyMAF-X）。特定真人重建（單圖／單影片 → 3DGS Avatar）是目前寫實度最高的一條路。

**今天做不到的**：沒有任何方法能從一段不受限的文字，一次產出「遊戲可用的、真正照片級、身份一致」的人體，同時滿足：解剖正確的臉和手、髮絲級頭髮、皮膚微結構與次表面散射行為、物理分層服裝、時間穩定的皺褶、乾淨拓樸／UV／LOD／碰撞體、極端姿態下不崩的綁定。

**為什麼**：這個領域是 specialists 的拼圖——參數模型給控制但不給外觀；擴散模型給外觀但解剖／身份一致性弱；3DGS 給渲染品質但不是可編輯的生產拓樸；MetaHuman 給生產整合但專有且不是自主文字生成。

**目前最接近的實作架構**：
1. 持授權的 SMPL-X／SUPR（或生產級綁定）作為可控身體先驗；
2. 文字／圖像擴散做多視角一致的外觀與法線／細節圖；
3. 臉、頭髮、服裝各自獨立模組；
4. 高斯或神經殘差層補姿態相關的高頻外觀；
5. 網格提取／重拓樸、UV／PBR 烘焙、綁定／LOD 驗證，進遊戲引擎。
原則：參數化身體永遠是動畫／控制骨架；3DGS／神經場當作外觀／細節層，不要指望它們取代生產拓樸。

## 七、給你的實作建議（RTX 5080 本機路線）

以你家 Windows＋RTX 5080（16GB VRAM）為前提，最務實的起手式：

1. `pip install smplx[all]`＋申請 SMPL-X 模型檔（非商業研究授權），先在 Python 裡玩轉體型／姿態參數，理解這套「引擎」的 API。
2. 裝 Meshcapade Blender 外掛或 MPFB，把 SMPL-X 丟進 Blender，看網格長什麼樣。
3. 接 BlenderMCP，讓 LLM agent 直接在 Blender 裡下指令（擺姿態、換體型、套材質），驗證 agent 驅動的可行性。
4. 外觀層再疊：先用 Stable Diffusion／FLUX 類模型生貼圖，最後才碰 3DGS（HumanGaussian 類 repo 吃 VRAM 兇，16GB 要調小解析度或分塊）。
5. 先別想一次到位做 NPC：第一階段目標訂為「agent 能用文字叫出正確姿態的 SMPL-X 人體並在 Blender 渲染」，這步通了，後面全是加法。

## 未驗證／待確認事項

- 所有 repo 的 GitHub star 數（本次未取得，報告內不引用任何數字）。
- 確切授權條文：FLAME、GHUM、STAR（PS:License 1.0 細節）、PyMAF-X、ChatPose／PoseGPT、MotionGPT、TeCH、TaoAvatar、MakeHuman／MPFB、MB-Lab。
- 官方 repo 待確認：HumanGaussian、GaussianAvatar、Animatable Gaussians、4D-Humans（官方 vs 分支）、PyMAF-X（找不到 repo）、MotionGPT（找不到官方 repo）、DreamHuman（找不到程式碼）、GAvatar（找不到公開程式碼）。
- 發表 venue 待確認：DreamAvatar、CHUPA、PyMAF-X。
- Ready Player Me 下線：僅第三方 2026-08-31 報導，未經官方證實。
- MetaHuman：官方 Epic 文件與現行 EULA 本次未直接讀取。
- 2025–2026 新項目（Mon3tr、HumanDreamer-X、PSHuman repo、AniCrafter、SplatShot）多為二手／早期來源，動手前請直接驗證。

## 資料來源

### 論文與專案頁
- SMPL：https://smpl.is.tue.mpg.de/
- STAR（MPI）：https://is.mpg.de/en/code/star-a-sparse-trained-articulated-human-body-regressor
- GHUM（CVPR 2020）：https://openaccess.thecvf.com/content_CVPR_2020/html/Xu_GHUM__GHUML_Generative_3D_Human_Shape_and_Articulated_Pose_CVPR_2020_paper.html
- SUPR：https://arxiv.org/pdf/2210.13861
- MANO：https://mano.is.tue.mpg.de/
- ChatPose：https://yfeng95.github.io/ChatPose/；https://is.mpg.de/ps/publications/chatpose；https://openaccess.thecvf.com/content/CVPR2024/html/Feng_ChatPose_Chatting_about_3D_Human_Pose_CVPR_2024_paper.html
- PyMAF-X：https://hongwenzhang.github.io/pymaf；https://arxiv.org/pdf/2207.06400
- MotionGPT：https://arxiv.org/abs/2306.10900；https://qiqiapink.github.io/MotionGPT/
- T2M-GPT：https://arxiv.org/pdf/2301.06052
- HMR 2.0：https://arxiv.org/abs/2305.20091v3；https://shubham-goel.github.io/4dhumans/
- HumanTOMATO：https://arxiv.org/abs/2310.12978
- CHUPA：https://snuvclab.github.io/chupa/；https://arxiv.org/pdf/2305.11870
- AvatarCraft：https://arxiv.org/pdf/2303.17606
- DreamAvatar：https://arxiv.org/abs/2304.00916
- TADA：https://arxiv.org/abs/2308.10899
- AvatarVerse：https://arxiv.org/abs/2308.03610
- DreamHuman：https://arxiv.org/abs/2306.09329
- DreamWaltz-G：https://arxiv.org/abs/2409.17145
- HumanGaussian：https://arxiv.org/abs/2311.17061
- GAvatar：https://arxiv.org/abs/2312.11461
- GaussianAvatar：https://arxiv.org/abs/2312.02134
- Animatable Gaussians：https://arxiv.org/pdf/2311.16096.pdf
- CharacterGen：https://arxiv.org/pdf/2402.17214.pdf
- TaoAvatar：https://arxiv.org/abs/2503.17032v2
- Capture, Canonicalize, Splat：https://arxiv.org/abs/2510.14081v3
- BlenderMCP：https://blendermcp.org/

### Repo（官方為主）
- https://github.com/vchoutas/smplx；https://github.com/ahmedosman/STAR；https://github.com/ahmedosman/SUPR
- https://github.com/otaheri/mano；https://github.com/hassony2/manopth；https://github.com/soccergame/flame-fitting
- https://github.com/Meshcapade/SMPL_blender_addon；https://github.com/zju3dv/EasyMocap
- https://github.com/open-mmlab/mmhuman3d；https://github.com/open-mmlab/mmpose；https://github.com/openxrlab/xrmocap
- https://github.com/vchoutas/smplify-x；https://github.com/nghorbani/human_body_prior
- https://github.com/makehumancommunity/makehuman-plugin-for-blender
- https://github.com/yfeng95/PoseGPT；https://github.com/naver/posescript；https://github.com/IDEA-Research/HumanTOMATO；https://github.com/Mael-zys/T2M-GPT；https://github.com/shubham-goel/4D-Humans
- https://github.com/ahujasid/blender-mcp
- https://github.com/songrise/AvatarCraft；https://github.com/yukangcao/dreamavatar；https://github.com/TingtingLiao/TADA；https://github.com/bytedance/AvatarVerse；https://github.com/Yukun-Huang/DreamWaltz-G；https://github.com/huangyangyi/TeCH；https://github.com/zjp-shadow/CharacterGen；https://github.com/MyNiuuu/AniCrafter

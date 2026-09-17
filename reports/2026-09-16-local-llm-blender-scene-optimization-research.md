# 在 RTX 5080（16GB VRAM）Windows 電腦上跑本地小模型驅動 Blender 做 3D 場景最佳化 —— 技術調查報告

- 調查日期：2026-09-16
- 情境：Windows 電腦，RTX 5080 16GB 顯卡。已有 3D 場景雛形，想讓本地小模型幫忙打磨、最佳化，甚至從頭建出來。做法可能是 Blender MCP，也可能是模型直接生成 bpy（Blender Python）腳本。
- 評估重點：本地模型的可行性、tool calling 可靠度、bpy 生成品質、真實實作案例。

---

## 一、Blender MCP 生態現況

### 1.1 ahujasid/blender-mcp（最主流）
- **原理**：Blender 端裝 addon，在 `localhost:9876` 開 TCP socket server；MCP server（`uvx blender-mcp`）把 Blender 操作包裝成 MCP tools；模型經由 MCP client（Claude Desktop、Cursor、VS Code、OpenCode 等）呼叫 tools。
- **工具**：場景/物件資訊查詢、建立/刪除/修改物件、材質控制、**執行任意 Python（`execute_code`）**、Poly Haven / Sketchfab 資產搜尋下載、Hyper3D Rodin / Hunyuan3D 生成式 3D。
- **關鍵特性：model-agnostic**。blender-mcp 不綁模型，模型由 MCP client 決定。只要 client 支援 MCP + tool calling，就能接本地模型。LM Studio 原生支援 MCP；Continue、OpenCode 等也支援本地模型 + MCP。

### 1.2 專為本地模型設計的專案
- **dhakalnirajan/blender-open-mcp**（最貼近本需求）：「Open Models MCP for Blender Using Ollama」，provider-agnostic 後端 —— Ollama、LM Studio、llama.cpp server、任何 OpenAI-compatible endpoint，可在 **runtime 用 MCP tool 切換 backend**，自帶 CLI client。工具含 `blender_execute_code`、`blender_render_image`、PolyHaven 工具。
- **marcelepsworks-arch/mcp-blender**：本地 LLM 導向，提供 LM Studio / Ollama 的 MCP client config 範本。
- **carlosh7/blender-mcp（ultra）**：239 個 tools，支援 LM Studio、Ollama，gallery 展示 agent 全自動建出的場景（含 PBR 材質、Geometry Nodes、剛體模擬）。
- **pranav-deshmukh/blender-mcp-demo**：曾上 HN Show HN，自然語言建村莊場景（「5 間茅屋圍著營火、河、橋、樹」）。
- **mik1703/blender-mcp-quality**：針對 Blender 5.1 的 production-grade skill，專門修正 AI 常犯的錯誤（見第四節）。

### 1.3 小結
Blender MCP 生態已成熟且 model-agnostic，**本地模型接得上**，關鍵在於：（1）選一個支援本地模型 + MCP 的 client（LM Studio、Continue、blender-open-mcp 自帶 client）；（2）模型本身的 tool calling 能力（見第三節）。

---

## 二、16GB VRAM 能跑多大的模型（實測數字）

以下為 2026 年針對 RTX 5080 的實測/估算（Ollama / llama.cpp，GGUF Q4_K_M，decode 速度，單用戶）：

| 模型 | 量化 | VRAM 佔用 | 速度 | 備註 |
|---|---|---|---|---|
| Llama 3.2 3B | Q4_K_M | ~2.5 GB | 160–200 t/s | 極快但能力弱 |
| Qwen3 8B | Q4_K_M | ~6.0 GB | 90–120 t/s | 寬裕 |
| Mistral Nemo 12B | Q4_K_M | ~8.0 GB | 72–88 t/s | 寬裕 |
| Qwen2.5 14B | Q4_K_M | ~10.7 GB | 62–72 t/s | **甜蜜點** |
| Qwen3 14B | Q4_K_M | ~12.0 GB | 61–70 t/s | **甜蜜點** |
| Qwen3 14B | Q5_K_M | ~12.9 GB | 55–65 t/s | 品質更好，仍可 |
| Qwen3 14B | Q6_K | ~14.5 GB | 42–52 t/s | 緊繃，context 要縮小 |
| Gemma 3 27B | Q3_K_M | ~14.3 GB | 25–35 t/s | 能跑但量化損失大 |
| 32B dense | Q4_K_M | >16 GB | ~6 t/s（CPU offload） | **不實用** |

**結論**：
- **14B Q4_K_M 是 16GB 的甜蜜點**：完整載入 GPU，還留 2–4GB 給 KV cache（16K context 約需 2GB）。
- 32B dense 塞不進 16GB，offload 到 CPU 後只剩 ~6 t/s，agent 迴圈慢到不可用。
- 注意：Windows 顯示輸出會吃掉約 1GB VRAM，實際可用約 14–15GB，規劃時要打折。

---

## 三、關鍵能力：tool calling / function calling

MCP 完全依賴模型的工具呼叫能力。

### 3.1 評測數據
- **Berkeley Function-Calling Leaderboard（BFCL）**：Qwen2.5-32B 在 BFCL v3 達 **82.6%**，接近 GPT-4o/Claude 3.5 水準。
- **關鍵門檻**：**7B 以下的模型 tool-use 準確率斷崖式下跌**，無法可靠遵循 function calling 格式；**32B 以上達 80%+**。14B 落在中間 —— 可用，但需要工程手段輔助。
- Hammer2.1-7B（專為 tool use 微調的 7B 模型）在小尺寸中表現突出，說明**專門微調比單純放大參數更有效**。
- 小模型 tool-calling 研究（2026）：頂尖小模型勝出的關鍵是知道何時**不**呼叫工具（restraint）；Qwen3 的 4B 在 restraint 上表現好，1.7B 反而有過度自信亂呼叫的問題。

### 3.2 候選模型的 tool calling 體質
- **Qwen3 系列（8B/14B）**：原生支援 tool calling，Apache 2.0，Ollama/LM Studio/llama.cpp 都有 template 支援。社群共識的本地 agent 首選。
- **Qwen2.5-Coder（14B）**：程式碼專精 + tool calling 可用；32B 塞不進 16GB，14B 是實際選項。
- **Phi-4（14B）**：推理強，Ollama 官方支援 tools，適合需要空間推理的場景描述理解。
- **Mistral Nemo 12B / Ministral 14B**：官方支援 function calling，12B 檔位 VRAM 寬裕。
- **Llama 3.x**：Ollama 支援 tools，但 8B 的 tool calling 可靠度不如 Qwen 同級。
- **Gemma 3**：27B 需 Q3 量化才塞得進，實測曾出現過度呼叫問題，不列首選。

### 3.3 實務要點
- Ollama 的 tool calling 依賴模型的 template 支援 —— 選 Ollama library 上標註 tools 支援的模型（如 `qwen3:14b`）。
- **Code-first agent 是另一條路**：讓模型直接寫 Python（含 bpy）再執行，繞過 JSON tool calling。實測顯示 code generation 對小模型是比 structured JSON 更強的能力。這正是 `blender_execute_code` 工具的價值所在。

---

## 四、程式碼生成能力：bpy 腳本的品質與坑

### 4.1 已知的系統性問題
1. **API 幻覺（版本差異）**：模型常混用不同 Blender 版本的 API。例如不存在的 enum、已被移除的函式、Blender 5.1 改用 layered/slotted Actions 但模型還在用 legacy 路徑。
2. **Context 陷阱**：某些 `bpy.ops` 只能在特定 editor context 下執行，從 script 呼叫會 poll 失敗。
3. **Stale data**：改完 `location` 後直接讀 `matrix_world` 會拿到舊值，必須呼叫 `view_layer.update()` —— 模型經常忘記。
4. **失控的參數**：如 UV Sphere 開 128 segments、物件浮空或沉到地面下 —— 缺乏空間驗證。

### 4.2 解法（比換模型更有效）
1. **在 system prompt 註明 Blender 版本** —— 最便宜、效益最高。
2. **給模型 introspection 工具**（如 `introspect_rna`、`list_operators`），讓它查 live API 而不是憑記憶。
3. **Render 回饋迴圈**：渲染後把圖丟回給模型做自我修正。
4. **Schema 拒絕時列出合法值**，讓模型從錯誤中恢復。
5. **小步迭代**：一次只做一個修改，每步驗證。

### 4.3 小模型的實際表現
- 14B 級（Qwen3-14B、Qwen2.5-Coder-14B）寫**短的、單一目的的 bpy 片段**（建基本體、調材質、擺位置）成功率不錯；**長腳本一次到位**的失敗率高，需要拆步 + 錯誤回饋迴圈。
- 8B 以下寫 bpy 基本不可靠，會頻繁幻覺 API。

---

## 五、真實實作案例

| 案例 | 架構 | 說明 |
|---|---|---|
| **dhakalnirajan/blender-open-mcp** | Ollama/LM Studio/llama.cpp → FastMCP → TCP → Blender addon | 專為本地模型設計，可 runtime 切換 backend。本需求最直接的起點。 |
| **carlosh7/blender-mcp** | 239 tools 的 MCP server → 任意 MCP client | Gallery 展示 agent 全自動建出的 PBR 材質球、Geometry Nodes 地形、剛體模擬場景。 |
| **pranav-deshmukh/blender-mcp-demo** | Node.js MCP server + bpy | HN Show HN：單句自然語言生成村莊場景。 |
| **mackson/blender-mcp** | 17 個高階 tools（含 `proc` 程序化生成、`opt` 減面、`preview` 快速渲染） | 把常用 3D 操作封裝成高階工具，降低對模型 bpy 能力的依賴 —— 小模型場景的關鍵設計模式。 |

**誠實的提醒**：目前公開案例中效果驚豔的 demo 多半用的是雲端大模型（Claude/GPT）；**本地小模型 + Blender 的完整成功案例在公開社群還很少**，因為 7B–14B 的 tool calling 與 bpy 可靠度需要上述工程手段補強。這不代表做不到，而是「模型選擇 + 工具設計 + 回饋迴圈」三者缺一不可。

---

## 六、實務建議：起手配置

### 建議配置 Top 3

**🥇 Top 1：平衡首選 —— Qwen3-14B Q4_K_M + Ollama + blender-mcp（經 LM Studio 或 Continue 接 MCP）**
- 14B Q4 約 12GB VRAM，16GB 卡上全 GPU 載入還有餘裕開 16K context，速度 60–70 t/s，agent 迴圈體感流暢。
- Qwen3 是目前本地 agent 能力的共識首選，tool calling + 中文都強。
- 接線：Blender 開 addon（port 9876）→ `uvx blender-mcp` 起 MCP server → LM Studio（原生 MCP 支援）載入 `qwen3:14b`。

**🥈 Top 2：程式碼生成導向 —— Qwen2.5-Coder-14B Q4_K_M + llama.cpp server + blender-open-mcp**
- Coder 系列寫 bpy 片段更穩；blender-open-mcp 專為本地模型設計，可 runtime 切換 backend，自帶 client CLI 除錯方便。
- 適合「模型主要靠 `blender_execute_code` 寫 Python」的 workflow（code-first 比 JSON tool calling 更適合小模型）。

**🥉 Top 3：推理/空間理解導向 —— Phi-4 14B Q4_K_M + LM Studio + mackson/blender-mcp（高階工具封裝）**
- Phi-4 推理強，適合理解複雜的場景描述與空間關係；高階 tools 把 bpy 細節封裝掉，降低對模型 API 記憶的依賴。
- 備選：Mistral Nemo 12B（VRAM 更寬裕，~8GB）。

### 期望值設定

**小模型做得到**：
- 雛形打磨：調整物件位置/旋轉/縮放、換材質參數、加減基本體、打燈光、設相機、批量整理。
- 在**高階工具封裝**下（一個 tool = 一個完整操作），14B 的多步任務可靠度大幅提升。
- 配合 render 回饋迴圈做迭代修正。

**小模型做不到（或很吃力）**：
- 一次到位的複雜場景藝術指導（如「做一個有故事感的賽博龐克街道」）：空間規劃與審美仍是大模型的領域。
- 精密的 bpy 操作（Geometry Nodes、rigging、複雜 modifier 堆疊）：幻覺率高，需要人工接手或拆成極小步。
- 長 horizon 任務（20+ 步工具呼叫）：錯誤會累積，建議每 3–5 步做一次 render 檢查點。

### 通用起手 checklist
1. Windows 裝 Ollama（或 LM Studio），`ollama pull qwen3:14b`。
2. Blender（建議 4.2+ LTS，API 較穩定）裝 blender-mcp addon，啟動 server。
3. MCP client（LM Studio / Continue）接上 `uvx blender-mcp`。
4. System prompt 第一行寫死 Blender 版本 + 常用 gotcha（`view_layer.update()`、能用 data API 就不用 `bpy.ops`）。
5. 先從「查詢場景 → 小修改 → 渲染確認」的迴圈開始，逐步放開權限。

## 備註
- VRAM 數字為 2026 年 RTX 5080 實測/估算；Windows 顯示輸出約佔 1GB，實際可用 14–15GB。
- 本地小模型 + Blender 的公開成功案例仍少，建議以 Top 1 配置先跑通最小迴圈再擴大。

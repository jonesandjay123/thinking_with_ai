# 本地模型 × Blender MCP：公開案例、硬體揭露與可落地工作流調查

> 研究日期／來源存取日：2026-09-14
> 問題：是否已有開發者以**本地／開放權重模型**，透過 MCP（Model Context Protocol）控制 Blender 開發 3D 場景？他們採用哪些模型、GPU、runtime 與做法？

## 先講結論

**有，而且從「本地模型 → MCP client/agent host → Blender MCP server → Blender Python」這條技術鏈來看，已經可做出多 agent、場景生成、量測、render、再修正的迭代工作流。**

但若問題是「能不能從公開網路得到大量、可信、附 GPU/VRAM 的真人成功案例」，答案是：**非常有限。**多數公開內容只揭露 Blender MCP repo、Ollama/local endpoint 或模型名其中一部分；完整同時揭露 **模型版本＋量化＋runtime＋GPU/VRAM＋Blender版本＋可檢查產物** 的案例仍罕見。

最強的公開實作證據不是一張炫圖，而是 Atlas / DayDreams 的 multi-agent 工程回報：本地 Ollama 與 headless Blender-MCP bridge 驅動 **8 個隔離 Blender session**，每個 agent 可 `execute_code`、量測實際 scene bounds、render-to-file；不過該案例**沒有公開模型名稱、GPU 或 VRAM**，只能證明系統形狀可行，不能證明某張顯卡的表現。

### 關鍵判讀

- **Blender MCP server 不等於本地模型。**它只將 MCP tool call 轉成 Blender add-on 裡的 `bpy` / Python 操作。
- **Ollama、LM Studio、llama.cpp 多半是 inference runtime，不是完整 MCP agent host。**要有一層 client/agent：把 tool schema 交給模型、解析 tool call、真的執行 MCP、把結果回灌模型。
- 小模型可以做「明確規格 → 可驗收 Blender Python」；困難在多輪 planning、工具選擇、觀察 render、修正錯誤與維持 scene constraint，而非單一 `bpy.ops.mesh.primitive_*` 指令。
- 目前社群資料支持的最佳策略是：**本地模型先產生受版控的 `bpy` 腳本、headless Blender 執行與 render 驗收；MCP 用在互動探索、scene inspection 和快速修正。**

---

## 1. 何謂「本地模型做 Blender MCP」？

典型資料流如下：

```text
Local model（Qwen / Llama / DeepSeek / Mistral 等）
  ↓ 由 agent host 提供 function/tool calling
MCP client / orchestration（例如 Atlas、Goose 或自建 client）
  ↓ stdio / HTTP MCP
Blender MCP server
  ↓ localhost socket
Blender add-on
  ↓ bpy Python
scene / render / geometry stats
  ↑ tool result / screenshot / file path
模型繼續規劃與修正
```

主要公開 server [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp) 的 README 明載：Blender add-on 建 socket server，MCP Python server 轉送工具；可建立／修改／刪除物件、改材質、檢視場景，以及在 Blender 執行任意 Python。它也明示可接不同 MCP client，而不是綁定一個雲端模型。

這代表「本地」至少分三層，報告不混為一談：

| 類別 | 意義 | 可否視為本地 Blender agent |
|---|---|---|
| 本地模型 + 本地 inference | 模型 weights 和 token generation 都在自己機器 | 是，最嚴格定義 |
| 雲端模型 + 本機 Blender MCP | 模型雲端、Blender/Python 本機 | 不算本地模型，但常見 workflow |
| 本地 MCP server + 未揭露模型 | MCP/Blender 在本機，模型端不明 | 僅能證明整合層，不能推論本地 LLM 成功 |

---

## 2. 公開案例矩陣

> 證據分級：**A**＝公開可檢查的實作／產物與具體執行敘述；**B**＝可檢查 PR/issue 的工程回報；**C**＝README、proposal 或使用者討論；**D**＝僅一般相容性宣稱。沒有硬體資料時，一律寫「未公開」，不以推測補齊。

| 日期 | 案例與來源 | 本地模型 / runtime | GPU / VRAM / OS | 實際 Blender 工作 | 證據與限制 |
|---|---|---|---|---|---|
| 2026-07-28 | [Atlas issue #851：DayDreams v4 multi-agent Blender MCP pool](https://github.com/thekaveh/atlas/issues/851) | **Ollama**；模型名未公開。Atlas managed localhost Blender-MCP bridge | GPU/VRAM/OS 未公開；每個孤立 Blender process 約 **1–2GB RAM**（不是 VRAM） | 8 位 agent 分別建 artifact；`execute_code`、取得真實 scene bounds、每 session render-to-file、看結果再修 | **A-/B+**：具體工程回報與架構細節，但 issue 不是受控 benchmark；無模型與 GPU 資料 |
| 2026-07-28 | [Atlas issue #759：managed headless Blender + MCP bridge](https://github.com/thekaveh/atlas/issues/759) | 本地 managed bridge；可與 local agent host 串接 | 未公開 | Headless Blender lifecycle、readiness、health、MCP connection，作為 agent 可控制的 host Blender | **B**：證明可運行的 infrastructure 方向；不是模型品質案例 |
| 2026-07-28 | [Atlas issue #830：Blender 5.1+ / Blender Foundation MCP successor wiring](https://github.com/thekaveh/atlas/issues/830) | 本地 bridge / MCP source 的工程規劃 | 未公開 | 將新版 Blender/MCP 接入 managed bridge | **C**：設計與路線，不可當成已完成的本地模型實測 |
| 2026-08-19 | [K-AIX Studio PR #3](https://github.com/ubuntuokos/K-AIX-Studio/pull/3) | Ollama + Goose + local tool stack，含 Blender/Blender MCP reference stack | GPU inventory/doctor 有設計，但貼文未提供實測 GPU/VRAM | Loopback-only local AI stack、Blender MCP 與其他創作工具的整合設定 | **B/C**：可檢查 stack 與設定，但 PR 為 draft，未提供具體 scene 成果或模型名 |
| 2026-09-14 查核 | [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp) | 模型不綁定；可由任一 MCP client 接入。README 亦列 Codex、Claude、Cursor、VS Code、OpenCode 等 client setup | 本地 Blender 支援 Windows/macOS/Linux；GPU 無關於 server 本身 | object/material/scene inspect/Python execution；可接 Poly Haven、Sketchfab、Poly Pizza、Hunyuan3D 等資產工作流 | **D（相容性）**：主流 server 的能力文件，不等於本地模型已成功完成某場景 |
| 2026-05-19 | [blender-mcp issue #251](https://github.com/ahujasid/blender-mcp/issues/251) | 未揭露 local model；Claude Code 經 MCP 的對照案例 | Linux；GPU 未公開 | Headless `-b` timer 阻塞導致 MCP request timeout；改同步背景執行後 `get_scene_info` 成功，並回報 960×540/32 samples Cycles render 約 3.6s | **C**：極有價值的 headless 失敗／修復經驗，但不是 local LLM 或 GPU benchmark |
| 2026-08-20 | [blender-mcp issue #328](https://github.com/ahujasid/blender-mcp/issues/328) | Codex MCP client 對照案例（非 local 模型） | 未公開 | `get_scene_info` 因 server schema 對 telemetry 欄位的要求而先失敗 | **C**：證明 tool schema 相容性是瓶頸，不可歸咎於模型能力 |
| 2026-08-29 | [blender-mcp issue #339](https://github.com/ahujasid/blender-mcp/issues/339) | 未揭露模型；Claude Desktop | Windows 11 + Blender 5.1.1；GPU 未公開 | addon 顯示 connected 但 command 未執行，約四分鐘 timeout | **C**：提示 version/PATH/port/addon 整合成本；不證明 local 方案較差 |

### 最值得重視的案例：Atlas / DayDreams

[issue #851](https://github.com/thekaveh/atlas/issues/851) 直接說明了最接近「本地 agent 團隊做 3D 場景」的工作形狀：每個 artifact agent 有自己的 Blender + MCP，依序建構、render、觀看、修正。作者明確寫出其 consumer 目前自行以 `subprocess.Popen` 啟動每個 free port 的 Blender launcher，並驗證：

- 8 個 independent session；
- `execute_code` 可用；
- live measure 回傳真實 scene bounds；
- 每個 session 可 render-to-file；
- 若 multi-agent 共用一個可變 Blender canvas，會造成 cross-agent interference；
- process 被殺後，orphan Blender 會留在背景並各占約 1–2GB 系統 RAM。

這對想做大型場景很重要：**第一個擴充瓶頸不一定是 LLM VRAM，而是 Blender process、render、資產載入、port/session isolation 和清理。**

---

## 3. 使用者實際採取的做法（可抽象的模式）

### 模式 A：單一互動式 Blender（最適合起步）

```text
Ollama / LM Studio / llama.cpp serve local model
  → 支援 MCP 的 agent host
  → blender-mcp
  → 已開啟的 Blender GUI
```

適用：請 agent 「建立 3 個 modular rock」、「調整材質」、「查 scene objects」、「render 一張 preview」；人盯著 viewport，確認後再繼續。

優點：低架構成本、能用 scene inspection 讓模型修正。
限制：模型若 tool calling 不穩，會卡在格式、漏參數或對 tool result 的誤解；任意 Python 權限高。

### 模式 B：headless、腳本優先（最適合可靠產物）

```text
local model → 產生 / 修改 versioned task.py
  → blender -b base.blend --python task.py --python-exit-code 1
  → render / .blend / .glb / geometry validation
  → result 回給模型
```

適用：procedural assets、批次變體、夜間 render、CI、可重跑 `.glb` export。

這不是「不用 MCP」：MCP 可以用於 inspection/brainstorming，而真正確認的 recipe 固化為 headless `bpy` script。issue #251 的經驗也顯示，不要假設 GUI-oriented add-on timer 在 `blender -b` 會自然工作。

### 模式 C：多 agent、每物件一個 Blender instance（進階）

Atlas/DayDreams 的做法是每 agent / artifact 都有自己的 process/port。適用於村落裡多個互不相依的 house、rock cluster、prop 或 vegetation variation。

必須額外建立：port allocator、readiness/health checks、process cleanup、asset namespace、對輸出 GLB 的統一驗收。否則 orphan process 和共享 scene 汙染會快速吞掉主機資源。

---

## 4. 本地 runtime 和模型：什麼能、什麼不能？

| 層 | Ollama | LM Studio | llama.cpp | 實用判讀 |
|---|---|---|---|---|
| 本地推理 API | 有 | 有 | 有 | 都可 serve 量化開放權重模型 |
| 自己就是 MCP agent | 通常否 | 通常否 | 通常否 | 需要 MCP client / orchestration 層 |
| tool calling 成敗 | 取決於模型模板、host 與 schema | 同左 | 同左 | runtime 名字不是品質排名 |
| 適合 Blender 起步 | 可作本機 model backend | GUI 方便做模型/上下文原型 | 便於精控量化/顯存/HTTP | 選你最容易固定版本、記錄 logs 的一個 |

### 本地模型選擇原則

公開案例不足以宣稱某一模型是「Blender MCP 冠軍」。比較合理的 shortlist 是選擇 **已知能穩定 function calling、可輸出長 Python、量化後仍有足夠 context** 的 coding/instruct 模型，再用同一個 Blender fixture 實測。

可先比較：

- **Qwen Coder / Qwen Instruct family**：通常有較完整 tool/function calling 與程式輸出工作流；依當期 model card 選可放入顯存的尺寸。
- **Llama Instruct family**：成熟 GGUF/Ollama/llama.cpp 生態；工具呼叫品質要以所用模板與版本實測。
- **DeepSeek Coder / distilled coding variants**：對長 `bpy` 腳本值得對照，但模型／量化越大，VRAM、context 和 tool latency 越容易成為互動瓶頸。
- **Mistral / Ministral 類 instruct**：小型 agent prototype 可測，但大場景長迴圈需實測其 tool schema 遵守率。

不要只看「會寫 Python」。Blender MCP 至少要測：**tool-selection、JSON/schema adherence、讀 scene 回傳資料、保留 denylist objects、失敗後自我修復、能否由 render/geometry stats 做第二輪修正。**

---

## 5. GPU / VRAM：公開資料說了什麼、沒有說什麼

### 已確認

- Atlas 案例公開的是 **每一 Blender process 1–2GB RAM** 的代價，不是 GPU VRAM。
- `blender-mcp` server/add-on 本身並沒有列出 LLM GPU 最低需求；它是 socket/Python bridge。
- 多數貼文沒有揭露 model quantization、GPU/VRAM、CUDA、Blender render engine 或同時 render + infer 的顯存壓力。

### 不能從資料推出的事

- 不能因為某個案例用 Ollama，就假設 8GB/12GB/16GB VRAM 都能順跑相同工作；模型名和 quantization 未公開。
- 不能把 Blender 的 1–2GB **系統 RAM** 誤算為 GPU requirement。
- 不能把「模型能 tool call」等同於模型能看 render image；純文字 MCP 回傳的 agent 需要額外接 screenshot/VLM 或人類 review。

### 務實的本機配置建議（工程推論，不是公開案例 benchmark）

| 目標 | 模型層建議 | Blender 層建議 | 註記 |
|---|---|---|---|
| 最小 prototype | 7–8B 量化 coding/instruct model | Eevee preview、單一 Blender GUI instance | 用 structured scene summary + render image 交給人或 VLM；先證明 tool loop |
| 個人 production assistant | 12–16GB VRAM 或更高，選 7–14B Q4/Q5；保留 context/KV 餘裕 | 不要讓同一張 GPU 同時高 samples Cycles render + 大模型推理 | 可將 render 和 inference 排程化或分 GPU |
| 多 agent asset pipeline | 多 GPU 或嚴格排隊；模型服務要支持並發策略 | 每 artifact isolate process；限制並行 render | Atlas 案例已證明 process lifecycle 比模型本身更早爆炸 |

這些是保守工程建議；不是「社群已證明 RTX 某型號必然可用」的陳述。

---

## 6. 你可以直接重跑的 Blender MCP benchmark

不要只看 Reddit/YouTube 成功片段。建立固定 fixture，讓不同 local model + runtime 可比較。

### 固定環境記錄

每一 run 記下：model ID、quantization、runtime/version、GPU/VRAM、RAM、OS、Blender version、MCP server commit、agent host、context、temperature、prompt、seed（若適用）。

### 三項任務

1. **Procedural prop**：生成 lantern / arch / rock cluster，要求 collection/object 命名、modifier policy、PBR material、GLB export。
2. **Scene composition**：在固定 `.blend` 新增 8–12 物件，依 camera framing、material palette、hierarchy 與 denylist 完成。
3. **Repair loop**：提供 bad normals、重複 material、過高 polycount 的 scene；只可改白名單 objects，輸出 before/after render 與 geometry stats。

### 100 分 gate

| 指標 | 分數 | 通過證據 |
|---|---:|---|
| 真正執行與產物 | 25 | exit code、檔案 mtime/hash、可重開 `.blend/.glb` |
| 幾何/結構正確 | 20 | objects、collection、polycount、modifier、UV/normals 檢查 |
| 視覺結果 | 20 | 固定 camera render + 人工或 VLM review |
| tool schema 與約束遵守 | 15 | 不改 denylist；不碰原始檔；工具呼叫可解析 |
| 可重現性 | 10 | 乾淨 workdir 可重跑 |
| 效率與除錯 | 10 | wall time、tool turns、可用 log、失敗復原 |

推薦採用「**MCP 探索 → 固化 bpy script → headless rebuild → artifact validation**」的雙軌方式；它既利用 MCP 的交互性，又避免把單次對話錯誤當成可交付資產。

---

## 7. 安全與可靠性邊界

`blender-mcp` README 明確指出模型可執行任意 Blender Python。其 Safe Mode 是腳本檢查，**不等同 sandbox**。本地模型不會自動降低這個風險。

最低實務線：

- 每個任務使用原始 `.blend` 的 copy，不能直接覆蓋唯一來源。
- MCP server 僅 bind loopback；不要為了方便暴露 socket 到 LAN/Internet。
- agent 只可寫入 task workspace / output directory；render、import、export 都走 allowlist。
- 對 export 做檔案大小、format、reopen、fixed-camera render、scene diff 驗證。
- 多 agent 必須有每 task 一個 process／port／output namespace，並有 orphan reaper。

---

## 最終判斷

**本地 LLM + Blender MCP 已不是純概念。**公開工程案例已證明：local Ollama 驅動的 agent pipeline 能以 MCP 同時控制多個隔離 Blender instance，產生 artifact、執行程式、量測 scene、render 並迭代。

但「具體模型／GPU」公開樣本仍不足，不能誠實地列出一張可靠的「誰用 RTX 4090 + 某 Q4 模型最強」排行榜。現在最值得做的不是照抄未揭露硬體的貼文，而是先在自己的目標 GPU 上跑固定 3-task benchmark，並把結果連同 Blender/MCP/runtime versions 留檔。

對想建立 3D 場景的人，最成熟的落地形狀是：

> **本地 tool-calling coding model 負責規劃與腳本；Blender MCP 負責互動 inspection；headless `bpy` runner 負責可重跑 build 與驗收；人或 VLM 負責最後的視覺品質判定。**

---

## 來源

### MCP / Blender 原始專案與官方技術文件

1. [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp)（README、能力、架構、safe mode，2026-09-14 查核）
2. [Model Context Protocol — introduction](https://modelcontextprotocol.io/introduction)（2026-09-14 查核）
3. [Blender command-line arguments](https://docs.blender.org/manual/en/latest/advanced/command_line/arguments.html)（2026-09-14 查核）
4. [Blender Python API — use Blender without UI](https://docs.blender.org/api/current/info_tips_and_tricks.html#use-blender-without-its-user-interface)（2026-09-14 查核）

### 公開工程案例／使用回饋

5. [Atlas #851 — managed instance pool / DayDreams v4](https://github.com/thekaveh/atlas/issues/851)（2026-07-28）
6. [Atlas #759 — managed headless Blender + MCP bridge](https://github.com/thekaveh/atlas/issues/759)（2026-07-28 查核）
7. [Atlas #830 — Blender 5.1+ / official MCP bridge successor](https://github.com/thekaveh/atlas/issues/830)（2026-07-28 查核）
8. [K-AIX Studio PR #3 — local AI studio reference stack](https://github.com/ubuntuokos/K-AIX-Studio/pull/3)（2026-08-19）
9. [blender-mcp issue #251 — headless timer timeout and workaround](https://github.com/ahujasid/blender-mcp/issues/251)（2026-05-19）
10. [blender-mcp issue #328 — Codex schema compatibility](https://github.com/ahujasid/blender-mcp/issues/328)（2026-08-20）
11. [blender-mcp issue #339 — Windows timeout report](https://github.com/ahujasid/blender-mcp/issues/339)（2026-08-29）
12. [blender-mcp issue #347 — MCP tool-schema token cost measurement](https://github.com/ahujasid/blender-mcp/issues/347)（2026-09-05）

> 使用提醒：以上 GitHub issue/PR 是公開、可回溯的工程材料，但除了官方文件外均不是供應商保證或受控 benchmark。其模型、硬體、效能若未明列，報告一律標示為「未公開」。

# Codex / GPT-6 Astra 與 Claude Code / Claude Fable 5.1：Blender 3D 建模工作流的公開回饋比較

> 研究日期／來源存取日：2026-09-13
> 範圍：比較兩條 Blender 自動化路徑：**(1) headless Blender Python CLI**，及 **(2) 開啟 Blender MCP 的互動式 addon/server 工作流**。聚焦可查證的公開回饋、限制與可驗收工作流，而非模型廠商行銷。

## 先講結論

1. **題目中的「Codex GPT-6 Astral」不是可確認的官方模型名。**截至研究日，OpenAI 官方文件可確認的是 **GPT-6 Astra**（`gpt-6-astra`）；Codex 是可使用模型與工具的 coding agent／產品，而非「Codex GPT-6 Astral」這個單一官方模型。官方的 `gpt-6-astral` 模型頁不存在。下文將它正名為「**Codex + GPT-6 Astra**」。
2. **「Claude Code Fable 5.1」可理解為 Claude Code 使用 Claude Fable 5.1，但不是一個單一產品名。**Anthropic 的正式模型名為 **Claude Fable 5.1**（`claude-fable-5-1`），Claude Code 可以選用它。
3. **「Blender NCP」未能驗證為通行協定或正式 Blender 專案。**可驗證、且最可能被誤寫的名稱是 **MCP（Model Context Protocol）**。本報告以下比較的是 Blender **MCP**，不把 NCP 當成既有產品能力。
4. 對可靠交付而言，兩個模型／agent 組合的最佳共同路徑都是：

   ```text
   agent 產生受版控 bpy Python 腳本
     → blender --background --python script.py
     → 檢查 exit code、.blend/.glb、render 和檔案新鮮度
   ```

   **headless CLI 是生產與 CI 主線；MCP 是互動探索與人工監看時的加速器。**目前公開資料不足以支持「GPT-6 Astra 明確勝過 Fable 5.1」或相反的 Blender 建模定論。

---

## 1. 名稱與證據邊界

| 原始說法 | 查核結果 | 本報告採用的可驗證說法 |
|---|---|---|
| Codex GPT-6 Astral | **未確認；命名不正確。**OpenAI 模型資料為 GPT-6 Astra，非 Astral；Codex 為產品／agent。 | Codex + GPT-6 Astra |
| Claude Code Fable 5.1 | 可拆成兩項確認：Claude Code 可選用 Claude Fable 5.1。不是一個合併的正式產品名。 | Claude Code + Claude Fable 5.1 |
| Blender NCP | **無法驗證為通行協定／正式 repo。** | Blender MCP（若提問者指的是 MCP） |

### 已確認：官方來源

- [OpenAI GPT-6 Astra model page](https://developers.openai.com/api/docs/models/gpt-6-astra) 與 [OpenAI models overview](https://developers.openai.com/api/docs/models)：列出 **GPT-6 Astra / `gpt-6-astra`**；研究日對 `gpt-6-astral` 的官方 URL 查核為 404。
- [OpenAI Codex](https://openai.com/codex/)：Codex 可在 ChatGPT、IDE 和 terminal 使用，定位於工程任務如開發、重構、測試與 code review。此能力描述**不等於**官方承諾 Blender 原生建模品質。
- [Anthropic models overview](https://platform.claude.com/docs/en/models/overview)：列出 **Claude Fable 5.1 / `claude-fable-5-1`**。
- [Claude Code model configuration](https://code.claude.com/docs/en/model-config)：說明 Claude Code 可以 `fable` alias 或完整模型 ID 選用 Fable 5.1；官方定位為高要求推理、長時程 agentic work。
- [Model Context Protocol introduction](https://modelcontextprotocol.io/introduction)：MCP 是 AI client 連外部工具／工作流的開放協定；不是 NCP。

### 重要不確定性

模型、Codex/Claude Code CLI、MCP server 與 Blender addon 都快速更新。網路上的成功影片、PR 或 issue 往往沒有留下完整 model ID、prompt、版本、OS、GPU、Blender build 或重跑紀錄。因此本報告把來源分為：

- **官方文件**：能確認名稱、API/CLI、機制與宣稱支援範圍。
- **公開 issue / PR / repo**：可觀察真實整合摩擦或產物，但不是受控 benchmark。
- **未能歸因的使用者體驗**：不能用來排序特定模型。

---

## 2. 兩條操作模式

### 模式 A：Headless Blender Python CLI

Blender 官方支援 background mode 與 Python：

```bash
blender --background scene.blend --python task.py --python-exit-code 1
# 簡寫：blender -b scene.blend --python task.py
```

`--background` / `-b` 不開 UI；`--python` 執行 `bpy` 腳本；`--python-exit-code` 可把 Python exception 轉為非零退出碼。

官方依據：

- [Blender command line arguments](https://docs.blender.org/manual/en/latest/advanced/command_line/arguments.html)
- [Use Blender without its user interface](https://docs.blender.org/api/current/info_tips_and_tricks.html#use-blender-without-its-user-interface)

**適合：**可重跑 procedural modeling、asset conversion、批次材質/UV/LOD、render regression、CI、需要 `.blend` / `.glb` 作為交付物的任務。

**核心要求：**agent 產生腳本不是完成。每次必須驗證 process exit code、輸出檔存在且是本輪新產生、開檔／匯出成功，並至少檢查 one-or-more renders 或 geometry stats。

### 模式 B：互動式 Blender MCP / addon server

可驗證的主要公開實作是 [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp)（MIT）。架構為：

```text
Codex 或 Claude Code（MCP client）
  → STDIO MCP server
  → localhost socket
  → 開啟中的 Blender addon
  → bpy 操作 / 場景資訊 / render
```

該 repo 的 README 明列 Claude Code 與 Codex 的 MCP client 設定方式，並支援 scene inspection、object/material 操作和在 Blender 中執行 Python。這是「agent 可連到正在開啟的 Blender」的整合，不是模型直接控制 GUI。

**適合：**人看著 viewport 進行探索、逐輪構圖或材質調整、詢問目前 scene 狀態、把 render/screenshot 回給 agent 評論。

**安全提醒：**此類 addon/server 讓 agent 有機會在 Blender 執行任意 Python。`BLENDER_MCP_SAFE_MODE=1` 是該第三方 repo 的自訂檢查，**不是**完整 sandbox；應使用測試檔、受限 project path、git/LFS 或檔案備份、最小權限與人工核准。

---

## 3. 模式比較：實際工程取捨

| 面向 | Headless Python CLI | 互動式 Blender MCP |
|---|---|---|
| 最佳用途 | 正式交付、CI、批次、可重跑資產 | 探索、即時 scene review、人工協作 |
| 可重現性 | **高**：script + input + exit code + artifacts | 中：受 GUI 當前 scene、socket、addon、模型決策影響 |
| 回饋 | logs、檔案、固定角度 render | 場景檢視、互動回合、viewport/render feedback |
| 系統複雜度 | 較低；主要是 Blender/腳本/檔案 | 較高；加上 client、MCP server、Python/uvx、socket、addon、GUI event loop |
| 自動驗收 | 很適合：return code、asset existence、poly/mesh/renders | 仍需驗收；不要把 tool call success 當成功建模 |
| 失敗隔離 | 容易：每個 job 是獨立 process | 較難：session state、port、addon、互動場景可能殘留 |
| 建議地位 | **production default** | creative/prototyping companion |

---

## 4. 公開使用回饋：可驗證案例與摩擦

> 下列是公開 repo/issue 的可追溯觀察，不是模型廠商測試，也不能外推成所有版本的成功率。

### 4.1 Claude Code 相關

| 來源／日期 | 模式 | 觀察 | 證據強度與限制 |
|---|---|---|---|
| [CardboardVR-FECAP PR #1](https://github.com/DG0809/CardboardVR-FECAP/pull/1)（2026-09-09） | Headless CLI | 公開 `.blend`、`.glb`、預覽與用 Python 重新生成 Blender 路燈的指令。 | **中**：產物可檢查，但標示為 Claude Opus 5 參與，**不是 Fable 5.1**，且不是 benchmark。 |
| [CLI-Anything PR #459](https://github.com/HKUDS/CLI-Anything/pull/459)（2026-09-09） | Headless CLI | 修正一個「只輸出 bpy 腳本和 command 文字、卻回報成功」的 harness；修後才真正啟動 Blender 5.2.1 LTS、檢查 render/檔案/退出碼。 | **強的流程教訓**，非模型排名：script 文本 ≠ 成功。PR 標示 Claude Code/Opus 5，非 Fable 5.1。 |
| [blender-mcp issue #251](https://github.com/ahujasid/blender-mcp/issues/251)（2026-05-19） | MCP + headless | 回報 addon 透過 `bpy.app.timers` 派送 request，在 `-b` 下被長駐主腳本阻塞、全部 request timeout；改成同步背景執行後，Claude Code `get_scene_info` 可回覆，960×540、32 samples Cycles render 報稱約 3.6 秒。 | **中低**：單一回報，不是官方 SLA；但直接證明 GUI-oriented MCP 不應假定能自然跑 headless daemon。 |
| [blender-mcp issue #339](https://github.com/ahujasid/blender-mcp/issues/339)（2026-08-29） | MCP GUI | Windows 11 + Blender 5.1.1 中 addon 顯示 connected，但 command 未執行，tool call 約四分鐘 timeout。 | **中低**：單案；顯示版本/PATH/port/addon 配對成本。 |
| [blender-mcp issue #260](https://github.com/ahujasid/blender-mcp/issues/260)（2026-05-28） | MCP GUI | Apple Silicon + Blender 4.5.3 LTS 下 port server 運行但 Claude Desktop 呼叫 timeout。 | **中低**：非 Claude Code、非 Fable 5.1，但反映同一整合棧的可靠性風險。 |

### 4.2 Codex 相關

| 來源／日期 | 模式 | 觀察 | 證據強度與限制 |
|---|---|---|---|
| [blender-mcp README](https://github.com/ahujasid/blender-mcp#quickstart)（研究日查核） | MCP | 明列可將 Blender MCP server 加到 Codex client。 | **官方 repo 文件（第三方整合）**：確認設定路徑，不代表對任何 Codex model 的品質保證。 |
| [blender-mcp issue #328](https://github.com/ahujasid/blender-mcp/issues/328)（2026-08-20） | MCP | 回報 Codex 雖能啟動標準 STDIO MCP server，但 `get_scene_info` schema 的 required telemetry field 讓 Codex 的無參數呼叫先失敗；當時文件亦不足。 | **中低**：特定第三方 server schema 問題，不等於 Codex 本身不能操作 Blender。 |
| [blender-mcp issue #350](https://github.com/ahujasid/blender-mcp/issues/350)（2026-09-07） | MCP | 使用者要求 Codex guide，表示文件可發現性／配置仍有摩擦。 | **低至中**：需求訊號，不是效能測試。 |
| [blender-mcp issue #347](https://github.com/ahujasid/blender-mcp/issues/347)（2026-09-05） | MCP | 第三方量測稱 MCP tool schema 約 5,462→6,928 tokens（+26.8%）；並討論 Claude Code 的 lazy tool-definition 行為。 | **低至中**：單一 repo 的 token accounting，非通用 benchmark。 |

### 4.3 缺口：不能替模型編造勝負

公開材料可支持「兩種 agent 都能接 Blender MCP，也都能產生/執行 Python-driven workflow」；但**沒有找到可公平比較的、同 prompt、同 Blender 版、同 assets、同工具權限、同 verification criteria 下的 Codex + GPT-6 Astra vs Claude Code + Fable 5.1 Blender benchmark**。

因此以下說法目前都應標為**未證實**：

- GPT-6 Astra 對 hard-surface topology 一定勝過 Fable 5.1；
- Fable 5.1 在 MCP server 互動一定更穩定；
- 任一模型一次 prompt 就可可靠完成 production-ready mesh、UV、PBR、rig、LOD、collision；
- 互動式 addon 成功回覆即代表 `.blend/.glb` 可用。

---

## 5. 對兩個組合的實用判讀

| 組合 | Headless Blender Python CLI | Blender MCP 互動式 | 最適用情境 |
|---|---|---|---|
| **Codex + GPT-6 Astra** | 官方可確認 Codex 能在 terminal／MCP 等工程環境中工作；Blender CLI 提供可驗收的外部執行層。**尚無專屬 Blender quality benchmark 可證勝率。** | blender-mcp 有 Codex 設定路徑，但公開 issue 顯示 schema/config docs 曾造成摩擦。 | 偏工程化 asset pipeline、腳本審查、批次 build；先用 minimal MCP smoke test。 |
| **Claude Code + Claude Fable 5.1** | Fable 5.1 的官方定位適合長時程 agentic work；但公開可檢查 Blender headless 案例多歸因 Opus 5 或未標 model，**不能當成 Fable 5.1 實證**。 | Claude Code 是成熟 MCP client 之一；但同一 blender-mcp stack 有 timeout/版本整合案例。 | 看著 viewport 創作、探索後固化 bpy 腳本；長任務仍要 artifact-based gates。 |

### 推薦選擇

- **你最需要可靠 repeatability/CI**：不要先選模型；先採 headless CLI harness。讓 Codex 與 Claude Code 都跑同一份 fixtures 與驗收，再選勝者。
- **你最需要藝術探索、人類盯著 Blender 持續對話**：用 Blender MCP，但把它放在 disposable copy 的 `.blend`；每個 milestone export/render，完成後再移植為 headless scripts。
- **你需要兩者兼得**：採混合式。MCP 做 scene interrogation / prototype / screenshot review；在「接受此方向」後，agent 產生 versioned `bpy` script，用 headless runner 重新生成並驗證。

---

## 6. 建議的公平測試，而非憑網路聲量選模型

建立三個固定 Blender fixtures，每個 fixture 各跑 3 次，每個組合都用相同 prompt、Blender version、assets、time budget、MCP server version（若適用）。

1. **Procedural hard-surface asset**：建立可重跑的 crate / lantern / arch；要求乾淨命名、modifier policy、UV、GLB export。
2. **Scene composition**：在固定場景新增 8–12 個 objects，符合指定 hierarchy、material palette、camera framing 與 Cycles/Eevee render。
3. **Repair task**：給一個有 broken normals、重複材質、過高 polycount 的 `.blend`，要求只改白名單 objects，保留 animation/camera。

### 評分（100 分）

| 指標 | 分數 | 驗證方法 |
|---|---:|---|
| 產物與執行正確性 | 25 | Blender 真執行；exit code 0；預期 `.blend/.glb/render` 存在且新鮮 |
| 幾何／資料結構 | 20 | object names、collection、polycount、manifold/normals、modifier/UV checks |
| 視覺結果 | 20 | 固定 camera/render 比對與人工 review |
| 指令遵守／安全範圍 | 15 | 不改 denylist objects；不覆寫 source；所有 file path 在 sandbox |
| 可重現性 | 10 | 乾淨 workdir rerun 能重建同等產物 |
| 迭代效率與可診斷性 | 10 | wall time、tool turns、失敗時 log/rollback quality |

### 最低安全與品質 gate

- 不允許 agent 在唯一原始 `.blend` 工作；先複製至 task workspace。
- 禁止任意 shell/file path；CLI runner 只允許指定 project/output 路徑。
- MCP addon 可執行任意 Python；只在受控電腦與受控 Blender project 啟用。
- 以 `--python-exit-code`、hash/mtime、Blender reopen、固定 render camera 作為完成條件。
- Agent 自我聲稱「done」不算通過；須由 runner 驗證 artifacts。

---

## 最終建議

**不要因「GPT-6 Astral」或「Blender NCP」這兩個未精確驗證的名稱，直接做採購或技術結論。**

若現在要投入建置：

1. 用官方 Blender CLI 做小型、受控的 headless verification harness。
2. 同時以 `ahujasid/blender-mcp` 在 disposable scene 做 Claude Code、Codex 的互動 smoke test。
3. 為每次成功互動保留「prompt、模型 ID、server/addon/Blender version、腳本、logs、輸出 artifact、固定角度 render」。
4. 僅在你自己的 fixtures 上得到重複通過的數字後，再決定哪個模型做主力。

這會比蒐集無法重跑的社群成功截圖更可靠，也避免把可用的 Blender automation 誤解成「模型已具備穩定 3D production autonomy」。

---

## 來源索引

1. [OpenAI — GPT-6 Astra](https://developers.openai.com/api/docs/models/gpt-6-astra)（官方，2026-09-13 查核）
2. [OpenAI — Models](https://developers.openai.com/api/docs/models)（官方，2026-09-13 查核）
3. [OpenAI — Codex](https://openai.com/codex/)（官方，2026-09-13 查核）
4. [Anthropic — Models overview](https://platform.claude.com/docs/en/models/overview)（官方，2026-09-13 查核）
5. [Claude Code — Model configuration](https://code.claude.com/docs/en/model-config)（官方，2026-09-13 查核）
6. [Claude Code — Overview](https://code.claude.com/docs/en/overview)（官方，2026-09-13 查核）
7. [Model Context Protocol introduction](https://modelcontextprotocol.io/introduction)（官方，2026-09-13 查核）
8. [Blender manual — command-line arguments](https://docs.blender.org/manual/en/latest/advanced/command_line/arguments.html)（官方，2026-09-13 查核）
9. [Blender Python API — without UI](https://docs.blender.org/api/current/info_tips_and_tricks.html#use-blender-without-its-user-interface)（官方，2026-09-13 查核）
10. [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp)（第三方開源 repo，2026-09-13 查核）
11. [blender-mcp issue #251](https://github.com/ahujasid/blender-mcp/issues/251)（公開使用回饋，2026-05-19）
12. [blender-mcp issue #260](https://github.com/ahujasid/blender-mcp/issues/260)（公開使用回饋，2026-05-28）
13. [blender-mcp issue #328](https://github.com/ahujasid/blender-mcp/issues/328)（公開使用回饋，2026-08-20）
14. [blender-mcp issue #339](https://github.com/ahujasid/blender-mcp/issues/339)（公開使用回饋，2026-08-29）
15. [blender-mcp issue #347](https://github.com/ahujasid/blender-mcp/issues/347)（公開使用回饋，2026-09-05）
16. [blender-mcp issue #350](https://github.com/ahujasid/blender-mcp/issues/350)（公開使用回饋，2026-09-07）
17. [CardboardVR-FECAP PR #1](https://github.com/DG0809/CardboardVR-FECAP/pull/1)（公開產物案例，2026-09-09）
18. [CLI-Anything PR #459](https://github.com/HKUDS/CLI-Anything/pull/459)（公開流程案例，2026-09-09）
19. [gangyusuke/yuske PR #4](https://github.com/gangyusuke/yuske/pull/4)（公開 MCP/headless 案例，2026-08-22；未作主結論依據）

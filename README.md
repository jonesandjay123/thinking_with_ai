# Thinking with AI

一個把 *思考 → 封裝 → 建造 → 累積* 接起來的 repo。

這裡不只是研究資料夾，也不只是靈感倉庫。它是 Jones 與 AI 夥伴（GPT / Jarvis / 其他 agents）共同工作的知識與產出中樞。

## Start here

如果你是未來的 Jones，或是未來的 Jarvis，建議先從這裡進入：

### 1. 核心方法論
- [Thinking → Building → Compounding](./principles/thinking-building-compounding/)
- [Playable Life System](./principles/playable-life-system/)

### 2. 操作流程
- [Conversation → Article Playbook](./playbooks/conversation-to-article/)

### 3. 關鍵決策
- [Repo 2.0 Upgrade Decision Log](./decision-log/2026-04-repo-2.0-upgrade.md)

### 4. 近期代表性專案
- [OpenClaw Slack Thread Continuity Success](./projects/openclaw-slack-thread-continuity-success/)
- [OpenClaw vs Hermes](./projects/openclaw-vs-hermes/)
- [Codex mobile usage check](./projects/codex-mobile-usage-check/)
- [Claude Code vs OpenClaw](./projects/claude-code-vs-openclaw/)
- [Trip Planner](./projects/trip-planner/)

## Repo 的新定位

`thinking_with_ai` 2.0 的核心不是「什麼都丟進 projects/」，而是把不同類型的內容分層管理，避免未來越堆越亂。

### 這個 repo 現在裝什麼？

- **principles/**：核心方法論、長期有效的框架、人生/創作/思考 operating principles
- **playbooks/**：可執行流程、操作手冊、工作流模板
- **projects/**：具體主題研究、專案檔案、比較報告、產品探索
- **decision-log/**：重要決策、方向轉折、為什麼這樣做
- **ideas/**：尚未封裝成專案的點子與待辦
- **archive/**：已失效、已轉向、已封存的舊內容

## 建議的放置規則

### 放進 `principles/` 的內容
適合放「過幾個月回來看仍然有用」的東西，例如：
- Thinking → Building → Compounding
- 個人創作方法論
- 長期工作觀、產品觀、人生觀

### 放進 `playbooks/` 的內容
適合放「下次還要照著做」的流程，例如：
- 怎麼把一段對話封裝成文章
- 怎麼用 Jarvis 協作做研究報告
- 怎麼從洞見變成 repo/project

### 放進 `projects/` 的內容
適合放「有明確主題邊界」的研究與專案，例如：
- OpenClaw vs Hermes
- Codex mobile usage check
- Trip planner
- Claude Code vs OpenClaw

### 放進 `decision-log/` 的內容
適合放「重要但不一定長期抽象化」的決策，例如：
- 為什麼不另外開新 repo，而是升級 thinking_with_ai
- 為什麼某專案轉向
- 某個產品方向被否決的理由

## 目前結構

```text
thinking_with_ai/
├── principles/                 # 長期原則與方法論
│   └── thinking-building-compounding/
├── playbooks/                  # 可執行流程與操作手冊
│   └── conversation-to-article/
├── projects/                   # 研究、報告、產品探索、專案檔案
├── decision-log/               # 關鍵決策與轉折
├── ideas/                      # 尚未封裝的點子與 backlog
└── archive/                    # 已封存歷史內容
```

## 目前最重要的內容

### Principles
- [Thinking → Building → Compounding](./principles/thinking-building-compounding/)
- [Playable Life System](./principles/playable-life-system/)

### Playbooks
- [Conversation → Article](./playbooks/conversation-to-article/)

### Decision logs
- [2026-04 Repo 2.0 Upgrade](./decision-log/2026-04-repo-2.0-upgrade.md)

### Selected Projects
- [OpenClaw Slack Thread Continuity Success](./projects/openclaw-slack-thread-continuity-success/)
- [OpenClaw vs Hermes](./projects/openclaw-vs-hermes/)
- [Codex mobile usage check](./projects/codex-mobile-usage-check/)
- [Claude Code vs OpenClaw](./projects/claude-code-vs-openclaw/)
- [Trip Planner](./projects/trip-planner/)
- [Vibe Coding Course](./projects/vibe-coding-course/)
- [Raijax v2](./projects/raijax-v2/)

## 實務運作原則

1. **不要再把所有東西都丟進 `projects/`**
2. **先分辨內容是 principle、playbook、project 還是 decision**
3. **原始對話可以保留，但最好封裝後再進主結構**
4. **repo 的目標不是囤積，而是讓未來的 Jones 和 Jarvis 找得到、用得上、接得回去**

## 下一步可持續優化

- 補更多 `playbooks/`
- 在 `decision-log/` 累積重要轉折
- 逐步檢查舊 `projects/` 是否有內容其實應該升格到 `principles/` 或 `playbooks/`
- 視需要加 `indexes/` 或 `maps/` 做跨專案導航

---

*Maintained by Jones + Jarvis*

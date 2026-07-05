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

### 3. 對話原文歸檔
- [Rubik's Cube Algorithms → Animal Shogi AI](./conversations/rubiks-cube-algorithms-animal-shogi-ai-2026-04-26/)
- [找工作成為攻擊入口 → Creative Lab 對齊](./conversations/job-scam-security-creative-lab-2026-06-02/)
- [驚嘆號、語氣指紋與個人操作說明書](./conversations/exclamation-mark-writing-style-memory-2026-06-20/)

### 4. 關鍵決策
- [Repo 2.0 Upgrade Decision Log](./decision-log/2026-04-repo-2.0-upgrade.md)

### 5. 近期代表性專案
- [OpenClaw 圖像識別失效調查](./projects/openclaw-image-pipeline-regression-2026-04-29/)
- [OpenClaw Slack Thread Continuity Success](./projects/openclaw-slack-thread-continuity-success/)
- [OpenClaw vs Hermes](./projects/openclaw-vs-hermes/)
- [Codex mobile usage check](./projects/codex-mobile-usage-check/)
- [Claude Code vs OpenClaw](./projects/claude-code-vs-openclaw/)
- [Trip Planner](./projects/trip-planner/)
- [Gemini 語音 → Firestore / Trip Planner](./projects/gemini-voice-firestore-trip-planner/)
- [NotebookLM x Jyn Pipeline Research](./projects/notebooklm-jyn-pipeline-research/)
- [AI 美女短影片生成技術前沿調查](./projects/ai-beauty-short-video-generation-2026-05-24/)
- [Logitech Slim Solar+ K980 電池與充電異常調查](./projects/logitech-slim-solar-k980-battery-issue-2026-06-01/)
- [Jcompany 水平無縫循環背景研究](./projects/jcompany-seamless-looping-background/)
- [Jcompany World Stage 技術路線調查報告](./projects/jcompany-world-stage-technical-route/)
- [Jcompany 3D Asset Lab：參照圖到 3D 地景工具調查報告](./projects/jcompany-3d-ground-scene-tools-2026-06-25/)
- [Decision Field Lab：點子發酵與交匯點資料結構調查](./projects/decision-field-idea-fermentation-2026-06-29/)
- [Android / Wear OS 手錶四年後升級調查：Google Fi + 續航 + 新功能](./projects/android-smartwatch-google-fi-2026/)
- [2024 邁阿密 Bayside Marketplace「外星人事件」調查報告](./projects/miami-bayside-alien-rumor-2024/)
- [Adobe 取消訂閱流程阻滯案例調查](./projects/adobe-cancellation-flow-obstruction-2026-07-04/)

### 6. 近期調查報告
- [LLM API 免費額度與強模型調查](./reports/llm-api-free-tier-research-2026-06-28.md)

## Repo 的新定位

`thinking_with_ai` 2.0 的核心不是「什麼都丟進 projects/」，而是把不同類型的內容分層管理，避免未來越堆越亂。

### 這個 repo 現在裝什麼？

- **principles/**：核心方法論、長期有效的框架、人生/創作/思考 operating principles
- **playbooks/**：可執行流程、操作手冊、工作流模板
- **projects/**：具體主題研究、專案檔案、比較報告、產品探索
- **conversations/**：值得長期保存的 GPT / AI 對話原文、來源與簡短脈絡
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

### 放進 `conversations/` 的內容
適合放「和 GPT / AI 聊完後覺得有長期價值、值得原文保存」的對話，例如：
- 觸發某個未來專案或產品方向的深聊
- 把 Jones 的真實想法、方法論、直覺講清楚的對話
- 之後可能要餵進 Thought Atlas、改寫成文章、或成為 project / principle 種子的材料

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
├── conversations/              # 值得保存的 AI 對話原文
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
- [OpenClaw 圖像識別失效調查](./projects/openclaw-image-pipeline-regression-2026-04-29/)
- [OpenClaw Slack Thread Continuity Success](./projects/openclaw-slack-thread-continuity-success/)
- [OpenClaw vs Hermes](./projects/openclaw-vs-hermes/)
- [Codex mobile usage check](./projects/codex-mobile-usage-check/)
- [Claude Code vs OpenClaw](./projects/claude-code-vs-openclaw/)
- [Trip Planner](./projects/trip-planner/)
- [Gemini 語音 → Firestore / Trip Planner](./projects/gemini-voice-firestore-trip-planner/)
- [NotebookLM x Jyn Pipeline Research](./projects/notebooklm-jyn-pipeline-research/)
- [AI 時代的膝蓋復健與物理治療消費調查](./projects/ai-physical-therapy-self-rehab-2026-05-23/)
- [AI 美女短影片生成技術前沿調查](./projects/ai-beauty-short-video-generation-2026-05-24/)
- [以照片找人的網站與開源替代研究](./projects/reverse-face-search-landscape-2026-05-30/)
- [Logitech Slim Solar+ K980 電池與充電異常調查](./projects/logitech-slim-solar-k980-battery-issue-2026-06-01/)
- [Android / Wear OS 手錶四年後升級調查：Google Fi + 續航 + 新功能](./projects/android-smartwatch-google-fi-2026/)
- [2024 邁阿密 Bayside Marketplace「外星人事件」調查報告](./projects/miami-bayside-alien-rumor-2024/)
- [Jcompany 水平無縫循環背景研究](./projects/jcompany-seamless-looping-background/)
- [Jcompany World Stage 技術路線調查報告](./projects/jcompany-world-stage-technical-route/)
- [Jcompany 3D Asset Lab：參照圖到 3D 地景工具調查報告](./projects/jcompany-3d-ground-scene-tools-2026-06-25/)
- [Decision Field Lab：點子發酵與交匯點資料結構調查](./projects/decision-field-idea-fermentation-2026-06-29/)
- [Adobe 取消訂閱流程阻滯案例調查](./projects/adobe-cancellation-flow-obstruction-2026-07-04/)
- [AI 時代的 LeetCode 面試文化調查](./projects/leetcode-ai-interview-culture-2026-06-03/)
- [Vibe Coding Course](./projects/vibe-coding-course/)
- [Raijax v2](./projects/raijax-v2/)

## 實務運作原則

1. **不要再把所有東西都丟進 `projects/`**
2. **先分辨內容是 principle、playbook、project、conversation 還是 decision**
3. **有長期價值的原始對話先放 `conversations/` 保存，之後可再封裝成 project / principle / playbook**
4. **repo 的目標不是囤積，而是讓未來的 Jones 和 Jarvis 找得到、用得上、接得回去**

## 下一步可持續優化

- 補更多 `playbooks/`
- 在 `decision-log/` 累積重要轉折
- 逐步檢查舊 `projects/` 是否有內容其實應該升格到 `principles/` 或 `playbooks/`
- 視需要加 `indexes/` 或 `maps/` 做跨專案導航

---

*Maintained by Jones + Jarvis*

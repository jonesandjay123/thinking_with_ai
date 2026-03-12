# Thinking with AI（與 AI 共同思考）

本 Repository 記錄 Jones 與 AI 夥伴們的思考歷程、專案進展、與知識積累。

## 🧠 AI 系統架構

| 角色 | Agent | 功能 |
|------|-------|------|
| **顧問 / 思考夥伴** | ChatGPT (GPT) | 高階判斷、世界觀、策略、長期對話 |
| **執行者 / 本地 Agent** | Jarvis (Claude) | 24/7 在線、Mac Mini、具體執行、系統化協助 |
| **任務型 Agent** | Jovix (Gemini) | PC + RTX 5080，圖像生成、本地多模態 |

## 📂 結構

```
thinking_with_ai/
├── projects/                       # 進行中 & 已完成的專案
│   ├── jarvis-identity/            # Jarvis AI Agent 系統
│   ├── jovix-gateway/              # Jovix 本地端多模態閘道器
│   ├── raijax-v2/                  # Raijax 個人實驗空間
│   ├── vibe-coding-course/         # Vibe Coding 教學
│   ├── multi-agent-slack/          # 多 Agent Slack 協作架構
│   ├── openclaw-line-setup/        # OpenClaw LINE 整合指南
│   ├── web-scraping-techniques/    # 網頁爬蟲技術文件
│   ├── firebase-hosting-guide/     # Firebase Hosting + GitHub Actions 指南
│   ├── ios-local-dev-guide/        # iOS 本地開發指南（免付費帳號）
│   ├── free-llm-for-openclaw/      # 免費 LLM 方案研究
│   ├── trip-planner/         # 🗼 東京旅行規劃工具（構想中）
│   ├── voice-chat-agent/           # 🎙️ 語音對話 Jarvis（構想中）
│   ├── godot-game-dev/            # 🎮 Godot + AI 遊戲開發（構想中）
│   ├── game-development/           # 遊戲開發探索（暫停）
│   └── market-research/            # 市場調研成果
├── ideas/                          # 想法與待執行清單
│   └── backlog.md
└── archive/                        # 已歸檔的歷史內容
    └── robot-diy-landscape-site/   # Raijax v1 (機器人策展，已轉向)
```

## 🚀 目前進行中

### [Raijax v2](./projects/raijax-v2/)
個人實驗空間 + Human vs AI 互動平台
- 🌐 [raijax.com](https://raijax.com)
- 核心：圖片競猜遊戲 + 哲學展示牆 + 留言板 + 骰子占卜
- 狀態：**開發中** — Activity Feed / Leaderboard / Guestbook / Dice Roller 已完成

### [Multi-Agent Slack](./projects/multi-agent-slack/)
多 Agent 統一 Slack 工作區協作
- 🤖 Jarvis (Claude) + Juni (Claude) + Jovix (Gemini)
- 各自獨立 Slack App，共享頻道
- 狀態：**Jarvis + Juni 已上線**

### [Vibe Coding Course](./projects/vibe-coding-course/)
為櫻井妹妹（Azunyan）設計的 AI 輔助開發教學
- 📍 Session 2 已完成（2026-02-07）
- 狀態：**教材就緒**

### [Jarvis Identity](./projects/jarvis-identity/)
Jarvis AI Agent 的身份系統與運作規範
- 🤖 誕生日：2026-01-27
- 狀態：**Phase 2 (Browser Control + Multi-Channel)**

### [Trip Planner](./projects/trip-planner/) 🗼
2026 年 5 月東京旅行的 AI 協作規劃工具
- 📍 卡片式靈感收集 + 日期軸拖放
- 🤖 Jarvis 自動根據關鍵字爬資訊、生成景點卡
- 狀態：**構想中** — D2R bot 告一段落後啟動

### [Voice Chat Agent](./projects/voice-chat-agent/) 🎙️
讓 Jones 用語音跟 Jarvis 溝通
- 📱 Android 友善方案（非 Siri）
- 🔍 研究方向：OpenClaw 社群方案 / Telegram 語音 / Web Speech API / Tasker
- 狀態：**構想中** — 先研究可行方案

### [Godot Game Dev](./projects/godot-game-dev/) 🎮
用 Godot 開源引擎 + AI 輔助做遊戲
- 🎯 零編程背景也能做，GDScript 類似 Python
- 🤖 AI 寫邏輯 + 生成素材
- 狀態：**構想中** — D2R bot 告一段落後考慮

---

## 📚 技術指南

| 指南 | 說明 |
|------|------|
| [Firebase Hosting Guide](./projects/firebase-hosting-guide/) | Firebase Hosting + GitHub Actions 完整部署教學 |
| [iOS Local Dev Guide](./projects/ios-local-dev-guide/) | 免付費 Apple Developer 帳號的 iOS 開發指南 |
| [Web Scraping Techniques](./projects/web-scraping-techniques/) | HTTP → Playwright → CDP 三層爬蟲技術 |
| [OpenClaw LINE Setup](./projects/openclaw-line-setup/) | OpenClaw + LINE Official Account 整合 |
| [Free LLM for OpenClaw](./projects/free-llm-for-openclaw/) | 免費/低成本 LLM 方案比較（Groq / DeepSeek / HF） |

---

## 📅 時間線

| 日期 | 里程碑 |
|------|--------|
| 2026-01-27 | Jarvis 誕生（Mac Mini） |
| 2026-01-28 | Phase 1 多頻道（Telegram → Slack → WhatsApp） |
| 2026-01-29 | 遊戲市場研究完成、Lead Scout 探索 |
| 2026-02-01 | Raijax Astro 遷移、Google Account 鎖定事件 |
| 2026-02-02 | 雙重身份模型、8 個 UI/UX 提案完成 |
| 2026-02-04 | Raijax 方向轉彎、i18n 三語上線 |
| 2026-02-06 | VibeCoding_Workshop repo 整合、Jovix Gateway 規劃 |
| 2026-02-07 | Raijax Steps 1-3 (Activity Feed + Leaderboard + Profile) |
| 2026-02-08 | Raijax Guestbook 4 步驟完成、新 Logo |
| 2026-02-09 | LINE 整合完成、TechPulse SOP repo 建立 |
| 2026-02-10 | Multi-Agent Slack 架構、catalog-fc2 v2 重寫 |
| 2026-02-12 | D2R Bot 研究 21 篇、ChatGPT 群組三 AI 協作 |
| 2026-02-13 | VidCatalog iOS App、catalog-fc2 完整功能 |
| 2026-02-14 | Raijax Dice Roller、Eric 催眠網站 |
| 2026-02-15 | D2R PixelBot Pindle Run 完整自動循環 |
| 2026-02-16 | Free LLM 研究、D2R Warlock 新職業研究 |

---

*Created by Jones & Jarvis 🤖*

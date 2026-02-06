# Market Research — Lead Scout 成果

> 工具：Brave Search API + Web Fetch + Cron
> 產出：結構化 leads 資料庫

## $10 最快路徑探索（2026-01-28~29）

### 統計
- **41 leads** 涵蓋 7+ 賽道
- **12 shortlist**（高潛力）
- **88 visited URLs**

### Shortlist 精選

| 平台 | 類型 | 為何入選 |
|------|------|----------|
| **PromptBase** | AI Prompt 市場 | 直銷 0% 手續費，3-7 天可達 $10 |
| **Amazon KDP** | 電子書自出版 | 72hr 上架，35-70% 版稅 |
| **Printify** | POD T-shirt | 免費，配合 AI 圖像設計 |
| **UserTesting** | 產品測試 | $4-10+/測試 |
| **Prolific** | 調查研究 | $6+ 即可提款 |
| **Rev** | 轉錄/字幕 | $0.30-$1.10/分鐘 |

### 賽道覆蓋
- 💰 Side Gig / Micro-task
- 🎨 Creative / Design
- 📝 Content / Writing
- 🔍 Research & Testing
- 🌐 Translation
- 🎙️ Transcription
- 🏷️ Data & AI Training

## Raijax 市場調研（2026-02-01~02）

### 統計
- **69 個結構化發現**
- **5,377 行數據**
- **38 輪 cron 執行**

### 覆蓋平台
| 類型 | 平台 |
|------|------|
| 委託平台 | Fiverr, Upwork, VGen, Skeb, SKIMA |
| 創作者平台 | DeviantArt, ArtStation, Ko-fi, Cara |
| 內容平台 | WEBTOON, Tapas |
| 日本市場 | Coconala, Prompton, Aipictors |
| 社群 | Reddit, Toyhouse |

### 輸出
- `~/Downloads/exports/leads/raijax-scout-2026-02-01.md`

## 工具鏈

| 工具 | 用途 |
|------|------|
| Brave Search API | 安全的網頁搜尋（取代 browser crawling） |
| Web Fetch | 網頁內容擷取 |
| Cron | 定時執行探索任務 |
| Sub-agent | 並行研究 |

### Brave API 設定
- Plan：Free（2,000 requests/月）
- Rate limit：1 req/秒
- Keychain：`brave-api-key`

## 教訓

1. **長時間 cron 任務會撞到 API 用量上限** — 需要 rate limiting 和 backoff
2. **Browser crawling 有帳號鎖定風險** — 改用 API-based 搜尋
3. **結構化 leads 比散落筆記有用** — 建立標準格式

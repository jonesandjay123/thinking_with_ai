# 現代網頁爬蟲技巧筆記

> 2026-02 實戰經驗整理。從最簡單的 HTTP 請求到繞過 Cloudflare 的瀏覽器自動化，記錄一路踩過的坑。

---

## 目錄

1. [爬蟲的三個層次](#爬蟲的三個層次)
2. [為什麼簡單的 HTTP 請求會失敗](#為什麼簡單的-http-請求會失敗)
3. [Playwright 瀏覽器自動化](#playwright-瀏覽器自動化)
4. [CDP（Chrome DevTools Protocol）連接模式](#cdpchrome-devtools-protocol連接模式)
5. [Cookie 與 Session 管理](#cookie-與-session-管理)
6. [Cloudflare 反爬蟲機制與應對](#cloudflare-反爬蟲機制與應對)
7. [實戰架構設計](#實戰架構設計)
8. [常見坑與解法](#常見坑與解法)

---

## 爬蟲的三個層次

### Level 1：純 HTTP 請求（httpx / requests）

最簡單的方式，直接發 HTTP 請求取得 HTML：

```python
import httpx

resp = httpx.get("https://example.com/page/123")
html = resp.text
```

**優點：** 快、輕量、不需要瀏覽器
**限制：** 很多現代網站會直接返回 403 Forbidden

### Level 2：Headless Browser（Playwright / Puppeteer）

啟動一個無頭瀏覽器，像真人一樣瀏覽網頁：

```python
from playwright.async_api import async_playwright

async with async_playwright() as pw:
    browser = await pw.chromium.launch(headless=True)
    page = await browser.new_page()
    await page.goto("https://example.com")
    html = await page.content()
```

**優點：** 能執行 JavaScript、渲染 SPA
**限制：** 反爬蟲系統可以偵測到「自動化瀏覽器」的指紋

### Level 3：CDP 連接已有的瀏覽器

不啟動新瀏覽器，而是連接到一個「已經在跑的、已登入的」Chrome 實例：

```python
browser = await pw.chromium.connect_over_cdp("http://localhost:9222")
context = browser.contexts[0]  # 使用已有的 context（含 cookies）
page = context.pages[0]        # 使用已開啟的分頁
```

**優點：** 完全使用真實瀏覽器環境，cookies/session 都在，反爬蟲幾乎無法區分
**這是我們最終採用的方案。**

---

## 為什麼簡單的 HTTP 請求會失敗

很多網站使用 **Cloudflare** 等 CDN/WAF 服務，會在用戶到達網站前進行檢查：

```
用戶請求 → Cloudflare → 人機驗證 → 通過 → 實際網站
```

純 HTTP 請求被擋的原因：
- **沒有 JavaScript 執行能力** — Cloudflare 的 JS Challenge 無法完成
- **缺少瀏覽器指紋** — User-Agent、TLS 指紋、Canvas 指紋等
- **沒有 Cookie** — Cloudflare 會設置 `cf_clearance` cookie，純請求拿不到

實際測試結果：

```python
# ❌ 直接被 403
resp = httpx.get("https://some-protected-site.com/page/123")
# Status: 403 Forbidden

# ❌ 加 User-Agent 也沒用
headers = {"User-Agent": "Mozilla/5.0 ..."}
resp = httpx.get("https://some-protected-site.com/page/123", headers=headers)
# Status: 403 Forbidden
```

---

## Playwright 瀏覽器自動化

### 基本用法

```python
from playwright.async_api import async_playwright

async def scrape():
    async with async_playwright() as pw:
        browser = await pw.chromium.launch(headless=False)  # 有頭模式
        context = await browser.new_context()
        page = await context.new_page()
        
        await page.goto("https://example.com")
        await page.wait_for_load_state("networkidle")
        
        html = await page.content()
        # 用 BeautifulSoup 解析 HTML...
        
        await browser.close()
```

### 為什麼 Playwright 直接啟動也可能被擋？

因為自動化瀏覽器有「指紋」：

1. **`navigator.webdriver = true`** — Playwright 啟動的瀏覽器會設這個標記
2. **缺少插件** — 真實 Chrome 有各種 extensions，自動化版本沒有
3. **視窗大小異常** — headless 模式的預設解析度
4. **TLS 指紋** — 自動化 Chromium 的 TLS handshake 跟真正的 Chrome 略有不同

反爬蟲服務（Cloudflare、DataDome 等）會檢查這些特徵。

---

## CDP（Chrome DevTools Protocol）連接模式

### 核心概念

Chrome 有一個內建的除錯協議（CDP），允許外部程式控制瀏覽器。關鍵是：**我們可以控制一個「真正的、人工使用的」Chrome 實例**，而不是啟動一個新的自動化瀏覽器。

### 啟動 Chrome 並開啟 CDP

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=~/.my-chrome-profile \
  --no-first-run \
  --no-default-browser-check
```

**重點參數：**

| 參數 | 說明 |
|------|------|
| `--remote-debugging-port=9222` | 開啟 CDP 監聽埠 |
| `--user-data-dir=<path>` | **必須指定！** Chrome 不允許在預設 profile 開 debug port |
| `--no-first-run` | 跳過首次使用導引 |

### ⚠️ `--user-data-dir` 的坑

**Chrome 拒絕在預設 profile 上啟用 `--remote-debugging-port`**，會報錯：

> DevTools remote debugging requires a non-default data directory

解決方案：指定一個獨立的 profile 目錄（例如 `~/.my-chrome-profile/`）。

**好處：**
- 這個獨立 profile 不影響你平常用的 Chrome
- Cookies 在這個 profile 裡持久化，下次開還在
- 登入狀態保持，不用每次重新登入

### 用 Playwright 連接 CDP

```python
from playwright.async_api import async_playwright

async def scrape_via_cdp():
    pw = await async_playwright().start()
    
    # 連接到已在跑的 Chrome
    browser = await pw.chromium.connect_over_cdp("http://localhost:9222")
    
    # 取得現有的 context（包含所有 cookies）
    context = browser.contexts[0]
    
    # 取得已開啟的分頁，或新建一個
    page = context.pages[0] if context.pages else await context.new_page()
    
    # 正常瀏覽 — 此時有完整的 cookies/session
    await page.goto("https://some-protected-site.com/page/123")
    html = await page.content()
    
    # 重要：斷開連接（不是關閉瀏覽器！）
    await browser.close()  # 只是斷開 CDP 連接，Chrome 繼續跑
    await pw.stop()
```

### CDP 連接的生命週期管理

**錯誤做法：** 長時間持有 CDP 連接

```python
# ❌ 不好 — 連接可能因為 timeout 或 Chrome 更新斷掉
class Scraper:
    def __init__(self):
        self.browser = None  # 長時間持有
    
    async def init(self):
        self.browser = await pw.chromium.connect_over_cdp(...)
    
    async def scrape(self, url):
        page = self.browser.contexts[0].pages[0]  # 可能已經斷了
        await page.goto(url)
```

**正確做法：** 每次使用時短暫連接

```python
# ✅ 用 async context manager，每次連接→使用→斷開
from contextlib import asynccontextmanager

@asynccontextmanager
async def cdp_page():
    pw = await async_playwright().start()
    browser = await pw.chromium.connect_over_cdp("http://localhost:9222")
    try:
        context = browser.contexts[0]
        page = context.pages[0] if context.pages else await context.new_page()
        yield page
    finally:
        await browser.close()
        await pw.stop()

# 使用
async def scrape(article_id):
    async with cdp_page() as page:
        await page.goto(f"https://example.com/articles/{article_id}")
        html = await page.content()
        return parse(html)
```

---

## Cookie 與 Session 管理

### Cookie 的角色

網站的登入狀態通常靠 Cookie 維持：

```
登入成功 → 伺服器設置 session cookie → 後續請求帶上 cookie → 伺服器認得你
```

關鍵 Cookie 類型：
- **Session Cookie** — 登入狀態，通常是 `httpOnly`（JavaScript 讀不到）
- **cf_clearance** — Cloudflare 人機驗證通過的證明
- **其他追蹤 Cookie** — 各種分析、偏好設定

### 為什麼 CDP 模式不用管 Cookie？

因為你連接的是一個**已經登入過的** Chrome 實例，所有 Cookie 都已經在那裡了。瀏覽器自動帶上 Cookie，就跟你手動瀏覽一模一樣。

### Playwright Storage State（備用方案）

如果不想用 CDP，可以把登入後的 Cookie 存下來：

```python
# 登入後儲存狀態
await context.storage_state(path="storage_state.json")

# 下次啟動時載入
context = await browser.new_context(storage_state="storage_state.json")
```

**限制：**
- `httpOnly` cookie 可以正確儲存和還原
- 但 Cloudflare 的 `cf_clearance` cookie 綁定瀏覽器指紋，換了瀏覽器可能失效
- 適合不使用 Cloudflare 的網站

---

## Cloudflare 反爬蟲機制與應對

### Cloudflare 的防護層級

1. **JS Challenge** — 要求瀏覽器執行 JavaScript 計算（約 5 秒等待）
2. **Managed Challenge** — 根據 IP/行為決定是否要驗證
3. **CAPTCHA** — 最嚴格，需要人工點選
4. **Bot Management** — 企業級，綜合多種指紋判斷

### 我們的應對策略

| 方法 | 效果 |
|------|------|
| 純 HTTP 請求 | ❌ 100% 被擋 |
| Playwright 新實例 | ❌ 被偵測為自動化 |
| Playwright + stealth plugin | ⚠️ 有時能過，不穩定 |
| CDP 連接已登入的 Chrome | ✅ 穩定通過 |

**為什麼 CDP 能通過？**

因為我們用的是「真正的 Chrome」，不是自動化工具啟動的 Chromium：
- `navigator.webdriver = false`（真正的 Chrome）
- 有真實的瀏覽器指紋
- 已經通過過 Cloudflare 驗證，有 `cf_clearance` cookie
- TLS 指紋完全正常

### 請求頻率控制

即使繞過了反爬蟲，頻率太高還是會觸發封鎖：

```python
import random
import time

def random_delay(min_sec=2.0, max_sec=4.0):
    """模擬人類瀏覽間隔"""
    time.sleep(random.uniform(min_sec, max_sec))
```

建議：
- 每次請求間隔 2-4 秒隨機延遲
- 不要同時爬太多頁面
- 觀察網站的速率限制回應（429 Too Many Requests）

---

## 實戰架構設計

### 三模式爬蟲架構

設計一個能自動降級的爬蟲：

```python
import os

SCRAPER_MODE = os.getenv('SCRAPER_MODE', 'auto')
# auto: 先試 CDP，失敗則 standalone
# cdp: 只用 CDP
# standalone: 只用獨立瀏覽器

@asynccontextmanager
async def get_page():
    if SCRAPER_MODE == 'cdp':
        async with cdp_page() as page:
            yield page
    elif SCRAPER_MODE == 'standalone':
        async with standalone_page() as page:
            yield page
    else:  # auto
        try:
            async with cdp_page() as page:
                yield page
                return
        except Exception:
            print("CDP 連接失敗，改用獨立模式...")
            async with standalone_page() as page:
                yield page
```

### 一鍵啟動腳本

```bash
#!/bin/bash
# start.sh — 啟動 Chrome + 後端服務

CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
PROFILE="$HOME/.my-chrome-profile"
CDP_PORT=9222

# 1. 關閉現有 Chrome
osascript -e 'tell application "Google Chrome" to quit' 2>/dev/null
sleep 2

# 2. 啟動 Chrome（開啟 CDP）
"$CHROME" \
  --remote-debugging-port=$CDP_PORT \
  --user-data-dir="$PROFILE" \
  --no-first-run \
  --no-default-browser-check &

# 3. 等待 CDP 就緒
echo "等待 Chrome CDP..."
for i in {1..30}; do
    curl -s http://localhost:$CDP_PORT/json/version > /dev/null 2>&1 && break
    sleep 1
done

# 4. 確認登入狀態（首次需要手動登入目標網站）
echo "請確認已在 Chrome 中登入目標網站"
read -p "按 Enter 繼續..."

# 5. 啟動後端
SCRAPER_MODE=cdp python app.py
```

### Flask + 爬蟲整合

Flask 是同步框架，但 Playwright 是 async。解決方式：

```python
import asyncio

def run_async(coro):
    """在 Flask 同步環境中執行 async 函式"""
    loop = asyncio.new_event_loop()
    try:
        return loop.run_until_complete(coro)
    finally:
        loop.close()

# Flask route 中使用
@app.route('/api/scrape/<int:id>')
def api_scrape(id):
    result = run_async(scrape_article(id))
    return jsonify(result)
```

**注意：** 每次建立新的 event loop，不要重複使用。Python 3.10+ 的 `asyncio.run()` 在某些環境下重複調用會有問題。

---

## 常見坑與解法

### 1. Chrome 說「正在現有瀏覽器工作階段中開啟」

**原因：** Chrome 已經在跑，新的啟動指令只是在舊的 Chrome 開了新分頁，debug port 沒有生效。

**解法：** 先完全關閉 Chrome，再用 debug 參數重新啟動。

```bash
# macOS
osascript -e 'tell application "Google Chrome" to quit'
sleep 2
# 然後再啟動
```

### 2. `browser.contexts` 是空的

**原因：** CDP 連接成功，但 Chrome 沒有開啟任何視窗。

**解法：** 防禦性程式碼

```python
if not browser.contexts:
    raise Exception("沒有可用的 browser context")
context = browser.contexts[0]
page = context.pages[0] if context.pages else await context.new_page()
```

### 3. Session Cookie 過期

**原因：** 長時間不操作，伺服器端 session 過期。

**解法：** 
- 在 Chrome 裡手動重新登入
- Cookie 會自動更新，下次 CDP 連接就有新的 session

### 4. Pillow 版本與 Python 版本不相容

**情境：** 使用 Pillow 處理爬下來的圖片時報錯。

**解法：** 確認版本對應，例如 Python 3.13 需要 Pillow ≥ 10.4.0。

### 5. SQLite BLOB 與 JSON 序列化衝突

**情境：** 資料庫存了圖片 BLOB，Flask `jsonify()` 無法序列化 bytes。

**解法：** 查詢時明確列出欄位，排除 BLOB：

```python
# ❌ 會把 BLOB 帶進來
cursor.execute("SELECT * FROM items WHERE id = ?", (id,))

# ✅ 只選需要的欄位
cursor.execute("SELECT id, name, created_at FROM items WHERE id = ?", (id,))
```

### 6. asyncio.run() 重複調用問題

**情境：** Python 3.9 環境下，多次 `asyncio.run()` 報 event loop 相關錯誤。

**解法：** 用 `asyncio.new_event_loop()` 手動管理：

```python
def run_async(coro):
    loop = asyncio.new_event_loop()
    try:
        return loop.run_until_complete(coro)
    finally:
        loop.close()
```

---

## 技術選型總結

| 場景 | 推薦方案 |
|------|----------|
| 無反爬蟲的網站 | httpx / requests |
| 有 JS 渲染、無 Cloudflare | Playwright standalone |
| Cloudflare 保護 + 需要登入 | CDP 連接已登入的 Chrome |
| 大量頁面批次爬取 | CDP + 隨機延遲 + 錯誤重試 |
| 需要定期自動爬取 | CDP + 獨立 Chrome profile + 排程腳本 |

**核心心得：** 現代反爬蟲越來越聰明，與其跟它們正面對抗（改 User-Agent、用 stealth plugin），不如直接用「真正的瀏覽器」。CDP 連接模式是目前最務實的解法。

---

## 專屬 AI Agent 的場景：CDP 是最佳解

上面列出的 CDP「缺點」——佔記憶體、無法平行、需手動登入、綁定單機——在 **AI Agent + 專屬主機** 的場景下完全不成立：

| 所謂的「缺點」 | 為什麼不影響 |
|----------------|-------------|
| Chrome 一直跑佔記憶體 | 人類上班也整天開著 Chrome，Agent 代替人類操作，本質一樣 |
| 一次只能做一件事 | 人類也是。Agent 不趕時間，一整晚都可以慢慢跑 |
| 第一次需要手動登入 | 提前規劃好要訪問的網站，登入一次就好，Cookie 持久化 |
| 綁定單機 | Agent 本來就住在這台機器上，單機正是設計目標 |

**結論：** 對於擁有專屬主機的 AI Agent 來說，CDP 幾乎是零成本的最佳方案。不需要複雜的分散式架構，一台機器 + 一個 Chrome + 合理的延遲，就等同於一個人坐在電腦前慢慢瀏覽。

### 操作守則

- **一次一個分頁**，不貪心
- **隨機延遲 2-4 秒**，模擬人類節奏
- **晚上 / 閒置時段跑**，不影響主人使用
- **拋棄式帳號**用於高風險網站，主帳號不碰自動化

---

## ⚠️ 血淚教訓：Google 帳號被鎖事件

> **2026-02-01**，Jarvis（AI Agent）的主要 Google 帳號 `jarvis.jovi.ai@gmail.com` 被 Google 永久鎖定。

### 事件經過

1. Agent 使用 **cron job**（定時排程）自動執行任務
2. 任務涉及 **瀏覽器自動化操作 Google 自家服務**（Gmail 登入頁等）
3. 短時間內（12 小時，38 次執行）高頻觸發
4. Google 偵測到異常行為 → **帳號鎖定，無法恢復**

### 根本原因

- **在 Google 自己的平台上做自動化** — Google 對自家服務的監控極嚴
- **Cron 高頻執行** — 沒有 rate limiting 和 backoff 機制
- **用主要帳號** — 被鎖後損失巨大（Gmail、Drive、所有綁定的服務）

### 教訓（永久適用）

| 規則 | 說明 |
|------|------|
| 🔴 **絕不在 Google 服務上做 browser automation** | Gmail、Google Drive、Google 登入頁都不行 |
| 🔴 **主帳號禁止任何自動化** | 自動化只用拋棄式帳號 |
| 🟡 **Cron 任務要有 rate limiting** | 設定合理間隔，加 backoff 機制 |
| 🟢 **第三方網站風險低很多** | 爬第三方網站，Google 不會管你用 Chrome 做了什麼 |
| 🟢 **最差情況可控** | 第三方網站頂多封你的帳號/IP，不會影響其他服務 |

### 後續：雙重身份模型

事件後建立了身份分離機制：

- **Core Identity**（主帳號）— 只做人工操作，**永不自動化**
- **Bot Identity**（拋棄式帳號）— 專門用於自動化，被鎖了就換一個

這個教訓的核心：**不是爬蟲本身危險，而是在「錯誤的平台」用「錯誤的帳號」做「錯誤的頻率」。**

---

*最後更新：2026-02-09*

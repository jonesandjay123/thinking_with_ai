# SocialPulse 中國／全球雲端部署平台調查

> **研究日期／來源存取日：2026-09-11**
> 對象：廈門的初學者 + 美國 mentor，共同開發 React + TypeScript + Vite 的小型 client-side Web App。
> 目標：手機與電腦、盡量 VPN OFF、GitHub push 自動 build/deploy、低門檻／免費起步，未來可加安全的 serverless + LLM JSON extraction。

## Executive Summary

**今天不要因為 Cloudflare Pages 沒有「正式中國大陸支援」就直接放棄它；也不要把它當成廈門 VPN OFF 的保證。**

SocialPulse 目前是 localStorage 為主的靜態 React App，真正的決策成本很低：部署 10 分鐘、由實際的廈門使用者以她的 Wi-Fi 和手機行動網路測試，會比任何海外網路的泛用說法更有價值。

**今天兩小時 workshop 的推薦：先用她自己的 Cloudflare Pages 帳戶，連她自己的 GitHub repo，部署後立即做 VPN OFF 實測；若兩種她日常網路都能穩定完成首頁、reload、主要互動與手機開啟，就繼續 Cloudflare。** 不需要只因「未正式支援 China」而立即遷移。

但必須設定明確的退出條件：若廈門的 Wi-Fi 或行動網路任一在 VPN OFF 下有無法開啟、頻繁 timeout、明顯慢到無法日常使用，**不要用 `pages.dev` 繼續硬撐**。先改測自訂網域；仍不穩定就把同一 repo 匯入 **Tencent EdgeOne Makers（原 EdgeOne Pages）的 Global excluding Chinese mainland 路線**並再次實測。它有 Git deployment、免費方案、static/Functions 與較貼近中國網路營運的產品面，但在無 ICP 的 global 模式下仍是「需要實測」，不是中國大陸加速保證。

若未來成為面向中國大陸的正式產品，且「VPN OFF 穩定可用」不再只是個人使用：**不要尋找海外免 ICP 的神奇平台；應走中國 Mainland region + ICP + 中國雲／合作夥伴的正式路線。**

---

## 我們真正需要解決的問題

SocialPulse 的前端技術並不構成難題：Cloudflare Pages、Azure Static Web Apps、EdgeOne Makers、Vercel、Netlify、GitHub Pages 都能處理 React + Vite 靜態輸出，也都能由 GitHub push 觸發部署。

難題是跨境網路與合規：

1. 廈門一般使用者的不同 ISP、Wi-Fi 與行動網路，對境外服務的路由、DNS、TLS、CDN domain reputation 和臨時網路策略可能不同。
2. **沒有 ICP 且不使用中國 Mainland acceleration** 時，任何海外平台都不是「官方承諾的中國大陸穩定 SLA」。
3. 「Cloudflare 在中國有 China Network」不等於 Free/Pro 的 Cloudflare Pages 自動走中國網路；「Azure 有 China」也不等於 Global Azure Static Web Apps 會成為中國本地服務。
4. LLM API 不應在 browser 直接呼叫：API key 會被看見、濫用或耗盡免費額度。應由 serverless API 保存 secret、驗證輸入、呼叫 LLM、驗證 JSON 後才回傳 preview。

---

## 證據標示規則

- **[官方]**：供應商文件、定價或產品頁直接陳述。
- **[高可信推論]**：由官方網路／區域架構導出的工程結論。
- **[實測／社群]**：OONI 測量、GitHub issue、V2EX 等可追溯外部觀測；可反映風險，不能取代廈門本人測試。
- **[不可保證]**：跨境連線結果受 ISP、時間、網域、DNS 與政策改變影響。

---

## Cloudflare Pages 現況

### Global Network 與 China Network 不是同一件事

- **[官方]** Cloudflare 文件明說，境外伺服器流量跨越中國網路邊界會面臨 latency 與 reliability 問題；China Network 是在中國大陸、由 JD Cloud 合作營運的獨立網路。
- **[官方]** China Network 是 **Enterprise plan 的獨立訂閱**，每個 apex domain 必須有有效 ICP filing/license，並非 Free/Pro Pages 的自動能力。
- **[官方]** China Network 並非所有 Cloudflare 產品皆可用；功能表應逐一確認。不要把 Cloudflare Pages 當成已在 China Network 受正式支援的預設產品。若真進入 Enterprise + ICP 路線，應和 Cloudflare 確認 entitlement；靜態內容可評估 Workers + Static Assets／回源，而非假設 Pages 直接可用。

### `*.pages.dev`、自訂網域與廈門體驗

- **[高可信推論]** `pages.dev` 是共享平台網域，較難對單一 app 控制 DNS、聲譽與替代路由；它不應是正式入口。
- **[高可信推論]** 自訂網域可消除共享預覽網域依賴、讓 DNS/監測/未來遷移可控，也可能因 domain 路徑差異改善可達性；但**不會讓 Global Pages 自動變成 Cloudflare China Network，也不會提供中國 SLA**。
- **[不可保證]** 因此可以合理用於私人、教學、少量日常使用，前提是實際使用者以日常 ISP 測過並接受偶發風險；不應用作需要保證的中國公開服務入口。
- **[實測原則]** 「有時能開、有時慢、不同 ISP 結果不同」是跨境服務的合理風險模型。必須記錄日期、VPN OFF、Wi-Fi ISP／行動電信商、URL、首頁和 API 結果，而不是只做一次成功截圖。

### Cloudflare Pages / Workers 是否仍適合 SocialPulse？

適合**今天的 workshop**：GitHub 連接、Vite build、preview deployment、custom domain、Pages Functions 都成熟；Free Pages 的 500 builds/month 對 workshop 充裕。未來可用 Pages Functions 或 Workers 作為 API 層。

不適合承諾「廈門長期 VPN OFF 一定穩」：沒有 ICP、Enterprise China Network 或在地基礎設施時，這是海外 best-effort。

---

## 候選平台比較（簡潔版）

| 候選 | 廈門 VPN OFF 的定位 | GitHub → 自動 deploy | 免費／初學者 | Serverless / AI | ICP／主要限制 | 本案判斷 |
|---|---|---|---|---|---|---|
| **Cloudflare Pages + Workers** | 實測優先；Global best-effort，非正式 China Pages | 很好 | 很好 | Pages Functions、Workers AI | China Network 要 Enterprise + ICP；`pages.dev` 不當正式入口 | **今天首測** |
| **Azure Static Web Apps Global** | 境外 Global，不能保證 China connectivity | 很好（GitHub Actions） | Free 可用 | Azure Functions、Azure AI | Global 與 Azure China 分離；仍需廈門實測 | **第二海外對照** |
| **Azure China / 21Vianet** | 中國本地服務 | 需另行確認每項服務／帳戶 | 非 workshop 低摩擦方案 | 中國區 Azure 服務取決於 availability | 獨立營運、帳戶／訂閱與功能差異，通常要 ICP | **正式中國產品候選，不是今日首選** |
| **Tencent EdgeOne Makers** | Global excluding Mainland 仍須實測；中國供應商產品面較合適 | 很好（Git import） | Free、無卡起步 | Edge/Cloud Functions、內建 AI 面向 | Mainland acceleration / 某些正式 domain 規則與 ICP 有關 | **Cloudflare 失敗後的第一 Plan B** |
| **Alibaba Cloud HK + OSS/FC** | 香港部署免 ICP，但跨境仍無 SLA；常是可行折衷 | 需 CI/CLI，較不「一鍵」 | 有新戶活動，長期成本需算 | Function Compute、Model Studio/Qwen | Mainland region/CDN acceleration 通常要 ICP | **需要更多控制時的 Plan B2** |
| **Vercel** | 風險高；OONI 一年 CN 測量 `vercel.app` 102/102 anomaly | 極好 | Hobby 好 | Functions / AI SDK | 不能視為 VPN OFF 入口 | **不推薦本案** |
| **Netlify** | 有部分成功也有 anomaly；不穩 | 很好 | Free 可起步 | Functions | 境外服務，無 China 保證 | **僅備選測試** |
| **GitHub Pages** | `github.io` OONI 中國測量明顯波動 | 很好 | 免費 | 無原生 serverless | 需外接 backend，網域可達性風險 | **baseline，不作產品入口** |

> OONI 一年 CN 測量摘要（2025-09-11～2026-09-11；屬外部網路測量，不是官方可用性承諾）：`vercel.app` 102/102 anomaly；`netlify.app` 23 OK / 8 anomaly；`github.io` 38 OK / 44 anomaly。這足以把 Vercel、Netlify、GitHub Pages 排除於「可保證 VPN OFF」的候選，但不能推論任何未測的 custom domain 一定失敗或一定成功。

---

## 各候選的務實判讀

### A. Cloudflare Pages / Workers

- **部署**：對 Vite 是標準 static build；push build/deploy，preview 很適合 mentor review。
- **免費**：[官方] Free Pages 有 500 builds/month、100 custom domains/project；Pages Functions 計入 Workers quota。
- **後端**：[官方] Pages Functions 在 Workers runtime 執行，能做 auth、form、middleware、server-side API，適合把 API key 放在 secret。
- **中國**：不把 Global Pages 與 China Network 混為一談；China Network 的正式條件對本案個人 workshop 過重。
- **結論**：今日先測；通過廈門雙網 VPN OFF 實測後，可先繼續使用，並每月／每次 ISP 變動後保留輕量健康檢查。

### B. Azure Static Web Apps（Global Azure）

- **[官方]** Global SWA 是針對 static frontend + serverless API 的產品，有 GitHub-native build/deploy workflow；Free plan 有免費 hosting、SSL、custom domain、100 GB/subscription bandwidth，整合 Azure Functions 可取得每月 100 萬免費 executions（實際 Functions 計價／限制要依帳戶確認）。
- **中國**：這是 Global Azure，不是中國境內服務。Microsoft 沒有把 Global `azurestaticapps.net` 宣告為中國大陸可保證服務；因此只可列為海外對照測試，不能從「Microsoft 在中國有 Azure」推出可達性保證。
- **結論**：若 Cloudflare 在她的日常雙網失敗，才部署同 repo 到 Global SWA 作對照；若它成功可作替代，但仍需要自訂網域與持續實測。

### C. Azure China（21Vianet）

- **[官方]** Azure China 是實體與商務上都分離的雲端 instance，由 21Vianet 獨立營運／交易；有 feature parity gap、不同定價與服務 availability。它不是把 Global Azure subscription 切換 region 即可。
- **實務**：帳戶、subscription、portal、付款／採購流程與可用服務需按中國區規則確認；對兩小時 workshop 與海外 mentor 而言明顯增加摩擦。Static Web Apps 是否在所需中國區／帳戶可用，應在 Azure China service availability 中逐項確認，不能預設。
- **結論**：僅在團隊決定正式落地中國、願意面對 ICP／本地帳務與 service-gap 時納入；不作今日 side project 起點。

### D. Alibaba Cloud（Mainland / Hong Kong / Singapore）

- **Mainland**：OSS static hosting、Function Compute、Serverless App Engine 等可組合；在中國 Mainland server 或 Mainland CDN acceleration 上線通常應把 ICP 當成前提，不適合本案「不想辦 ICP」的第一階段。
- **Hong Kong / Singapore**：香港 server 不等於 Mainland server，通常不以 Mainland-hosting ICP 為前提；但對廈門仍是跨境連線，速度與穩定性須實測。香港通常比美西／歐洲更接近，但不是 SLA。
- **GitHub workflow**：可用 GitHub Actions／CLI／SDK 自動化，但比 Pages/SWA/EdgeOne 的 dashboard connect repo 多一層基礎設施設定。
- **AI**：Alibaba Model Studio/Qwen 對中文與中國帳戶生態具吸引力；適合把 LLM 與 hosting 分離使用。
- **結論**：若 EdgeOne Global 也不佳、但仍不辦 ICP，Alibaba HK 是較可控的工程折衷；代價是 workshop 難度和長期費用管理提高。

### E. Tencent Cloud / EdgeOne Makers（原 EdgeOne Pages）

- **[官方]** EdgeOne Makers 支援連接 GitHub/GitLab/Bitbucket、設定 build command/output directory、自動部署 static SPA/full-stack app，並提供 Functions、storage 和 AI/agent 產品面；產品頁宣稱 Free plan 長期可用、無信用卡起步。
- **重要限制**：`global excluding Chinese mainland` 不等於 Mainland acceleration。沒有 ICP 時，應把它視為境外／全球邊緣路線並讓廈門本人測試。供應商系統 preview domain 的大陸使用限制不應做正式入口；用自己的 custom domain，並依 EdgeOne 當前區域與 domain 規則確認是否需要 ICP。
- **結論**：這是 Cloudflare 失敗後最值得在 workshop 內「同 repo 再部署」的 Plan B，因為 Git workflow 最接近、也容易帶初學者操作；仍以 VPN OFF 雙網實測決定。

### F. Vercel

- **Git／開發體驗**：業界最順，React/Vite/Functions/AI SDK 都優秀，Hobby 起步門檻低。
- **中國連通性**：[實測] OONI 中國探針一年對 `vercel.app` 102 次都回報 anomaly，不能把它排為本案 VPN OFF 入口。自訂網域或某個時間能開也不足以推翻這個風險結論。
- **結論**：不推薦給廈門日常使用者做 SocialPulse 主入口。

### G. Netlify

- Git push、Vite、Functions 與 Free plan 都好用。
- [實測] `netlify.app` 有成功測量但也有 anomaly；比 Vercel 不那麼明確地壞，但仍不足以承諾廈門 VPN OFF。
- **結論**：可當額外對照，不應是第一選擇或長期可靠性承諾。

### H. GitHub Pages

- 免費、GitHub workflow 最簡單、對 client-only Vite 很適合。
- 缺少內建 serverless API；LLM 必須外接 backend，不能把 key 放在前端。
- [實測] `github.io` 中國 OONI 測量 OK/anomaly 接近，穩定性風險高。
- **結論**：教學 baseline／文件頁可用，不作 SocialPulse 正式入口。

---

## 中國大陸 VPN OFF 可達性：如何測才有決策價值

不要只測「手機曾經開得出來」。在 workshop 以同一個 **custom domain** 測：

1. 廈門家用／日常 Wi-Fi，VPN OFF：首頁冷開、hard reload、進入主功能、至少連續 3 次。
2. 廈門手機行動網路，VPN OFF：同上。
3. 兩者各記錄 TTFB／主觀開啟時間、是否有 DNS/TLS/error、時間戳與截圖。
4. VPN ON 只作診斷：VPN ON 成功但 OFF 失敗，代表不是 app build 問題，而是入口／跨境可達性問題。
5. 24 小時後重複一次，不要只信一次測試。

**可接受門檻（side project）**：兩種她真正日常使用的網路皆能穩定開啟和互動，且 24 小時後未出現錯誤；可暫時繼續。
**換平台門檻**：任一日常網路 VPN OFF 有重複失敗、明顯卡住或需 VPN 才可用；立即改測 EdgeOne Makers，不為「已經部署過 Cloudflare」付 sunk cost。

---

## ICP 與中國法規：本案應怎麼看

- **中國 Mainland server／Mainland CDN acceleration**：通常需要 ICP filing；使用 China Network、Mainland CDN、Mainland OSS/compute 時，要把 ICP、domain 實名、帳戶與當地合規當設計前提。
- **香港／新加坡／美國等境外 region**：通常不以 Mainland hosting ICP 作為服務開通前提，但仍是跨境網路；不需要 ICP **不等於**保證 VPN OFF 穩定。
- **Custom domain**：不會自動創造 ICP 義務；義務主要取決於是否在中國 Mainland hosting/acceleration。若要把 domain 接入中國 CDN／China Network，ICP 會成為條件。
- **私人 side project**：在尚未證明有中國產品需求前，不值得為兩小時 workshop 辦 ICP。先以海外／香港部署 + 真人實測建立事實；若日後真要向中國公開、商業化、需要穩定性，再規劃中國實體、ICP 與資料／內容合規。

---

## AI / LLM 能力比較

### Cloudflare Workers AI（本案最容易先試）

- **[官方]** Free / Paid Workers 都有每天 **10,000 Neurons** 免費額度，00:00 UTC reset。Free plan 超過後後續操作會 error；Paid plan 超額為 **US$0.011 / 1,000 Neurons**。部分 frontier models 需要付費計費方式。
- **[官方]** Workers AI 支援 JSON Mode、OpenAI-style `response_format` 和 JSON Schema；但文件明示模型無法滿足 schema 時會 error，且 JSON Mode 目前不支援 streaming。因此 API 必須處理 schema failure、重試或讓使用者修正。
- **模型選擇**：若需要中文人際紀錄抽取，優先在真實樣本比較 Qwen family、DeepSeek/Qwen distill 或其他可用的中文導向模型，再以 schema adherence、成本、latency 選擇；不要只憑模型名稱保證中文品質。
- **整合**：Pages Functions/Workers 把 `POST /api/extract` 放在 server side，secret 留在 Worker binding，LLM 回傳 JSON 後以 Zod/JSON Schema 二次驗證，再回傳 preview。不要讓 browser 直接持有 API key。
- **中國風險**：Workers AI request 仍取決於使用者能否到達 Cloudflare Global service；它不應成為未驗證的中國可達性假設。

### Azure AI / Azure Functions

- Azure Static Web Apps 能整合 Azure Functions；Azure AI 則可作後端 provider。適合已經選 Azure、需要 Entra/企業整合的團隊。
- 對本案，Azure AI 並不因為 AI provider 是 Microsoft 就自動解決廈門至 Global SWA 或 Global Azure endpoint 的跨境可達性；仍需測試，且 key 一樣只應放在 Function secret。

### Alibaba Model Studio / Qwen

- 若需要中文資料抽取與中國雲帳戶／帳務的長期相容性，Qwen/Model Studio 是強候選；以 serverless backend 呼叫，要求 JSON output 並驗證 schema。
- Hosting 不必跟 Model Studio 綁在一起：Cloudflare/EdgeOne/Azure 的 serverless endpoint 都可以呼叫合法且可用的 LLM provider。要在廈門實測 API 從 server-side 出網的可達性與帳戶開通條件。

### Tencent AI / Hunyuan

- 若採 Tencent Cloud / EdgeOne 生態，Hunyuan/Tencent AI 可降低供應商帳務與中文產品整合摩擦；仍應先確認個人帳戶、區域、免費額度、structured JSON API 與輸出資料政策。

### 重要架構結論：hosting 和 AI provider 不必綁定

合理的早期架構是：

```text
React / Vite 靜態前端
  → 同源 serverless endpoint（保護 API key、rate limit、驗證 schema）
    → 可替換 LLM provider（Workers AI / Qwen / Azure AI / Hunyuan）
      → structured JSON preview
        → 使用者確認後才寫入 localStorage（未來再換 database）
```

這讓今天先把 hosting 的 VPN OFF 問題測清楚，不會因為 AI provider 更換就重寫 UI。

---

## 成本、Free Tier 與初學者體驗

| 路線 | 起步成本 | 初學者 workflow | 未來成本風險 |
|---|---|---|---|
| Cloudflare Pages | Free；500 builds/month | Dashboard 連 GitHub → build command → deploy | Workers/AI 超額與跨境可用性 |
| Azure SWA Global | Free 有 hosting/SSL/custom domain/100GB | GitHub Actions 自動產生 | Azure Functions、Azure AI、跨境可達性 |
| EdgeOne Makers | 官方稱 Free 長期可用 | Connect repo → build → deploy | 地區／domain／Mainland acceleration 與付費能力需確認 |
| Alibaba HK OSS + FC | 有活動／用量型；不是全程免費保證 | Actions/CLI/OSS+FC 多元件 | egress、CDN、Function/LLM、較多設定 |

對兩小時 workshop，**最便宜的不是最重要；「是否能在她的手機 VPN OFF 打開」才是第一指標。**

---

## 今天兩小時 Workshop 的實際建議

### 0–20 分：建立 owner-first 基礎

1. 由她登入自己的 GitHub repo；mentor 維持 collaborator，不接管 repo。
2. 她建立自己的 Cloudflare account，連接 GitHub。
3. Pages 選 Vite preset（或設定 build `npm run build`、output `dist`）；不要先加 database、auth 或 AI。

### 20–40 分：部署與可遷移入口

4. 讓第一次 production deployment 成功。
5. 若已有自己的 domain，立即加 custom domain；若沒有，先用 `pages.dev` 測試流程，但把它標示為 temporary preview，不當正式網址。
6. 將 build command、output directory、domain/DNS、rollback 方法寫進 repo README 或 workshop note。

### 40–70 分：最重要的廈門 VPN OFF 測試

7. 她在日常 Wi-Fi 和手機行動網路、VPN OFF 測試 cold open、reload、主要互動；mentor 在美國同步測。
8. 對每一網路記錄結果。這比泛泛的「Cloudflare 在中國是否能用」更直接回答 SocialPulse。

### 70–95 分：依測試分支

- **通過**：保持 Cloudflare Pages。不要急著遷移；把 24 小時 re-test 設為上線條件。
- **失敗**：先加／改測自訂網域；若仍失敗，直接把同一 GitHub repo 匯入 EdgeOne Makers Global，使用相同 Vite build，重做雙網 VPN OFF 測試。
- **兩者都失敗**：今天先保留 GitHub repo 與 local dev；不要倉促接入 Azure China 或辦 ICP。下一輪以 Alibaba HK OSS + Function Compute（或確認 Global Azure SWA）做受控對照，並決定是否已達正式中國產品門檻。

### 95–120 分：只在入口已驗證後才加 AI

9. 新增一個最小 serverless `extract` endpoint；使用 test key / test prompt，不在 browser 放 key。
10. 先要求固定 JSON schema，顯示 preview、使用者按確認才寫 localStorage。Workers AI 可當最短 path，但若 Cloudflare 的廈門測試不穩，就先把 endpoint 設計為 provider adapter，暫不綁死 Workers AI。

### 對核心問題的回答

**是，仍應該先 deploy Cloudflare Pages 並直接讓廈門的真實使用者測試。** 在她的日常 Wi-Fi + 行動網路、VPN OFF 都穩定時，這是比「官方不承諾 China」更強的本案證據；可繼續 Cloudflare，但要把它視為小型 side-project 的已驗證 best-effort，而非全中國可靠性保證。

---

## Ranking A：今天 workshop 最適合

1. **Cloudflare Pages + custom domain（先 `pages.dev` 作 preview）**
   最快、免費、GitHub integration 優、Vite 直覺、未來 Functions/Workers AI 路線短。唯一需要立刻驗證的是廈門 VPN OFF。
2. **Tencent EdgeOne Makers Global**
   若 Cloudflare 失敗，Git import／build／Functions/AI 模型接近原工作流，Free 起步且產品面更貼近中國市場；正式入口使用 custom domain，不能把 Global 模式誤稱 Mainland acceleration。
3. **Azure Static Web Apps Global**
   GitHub Actions、Free、Functions 都成熟，適合作為明確對照；但仍為海外服務，並不能因 Microsoft/China 品牌推定中國穩定性。

## Ranking B：櫻井妹妹在廈門長期日常使用最適合（不辦 ICP）

1. **在她本人兩種 VPN OFF 日常網路實測最穩的 Cloudflare Pages 或 EdgeOne Makers custom domain**
   沒有單一供應商可取代本地測試；通過實測者第一。預設先測 Cloudflare，再測 EdgeOne。
2. **Tencent EdgeOne Makers Global + custom domain（若測試表現最佳）**
   是無 ICP 下值得優先驗證的中國供應商全球邊緣方案；仍非 Mainland SLA。
3. **Alibaba Cloud Hong Kong OSS + Function Compute（若能接受較多設定）**
   地理上更近、工程上可控，但不可把香港免 ICP誤解成 Mainand 穩定保證，且初學者部署較複雜。

> Azure Global、Netlify、GitHub Pages可作測試對照；Vercel 因 `vercel.app` OONI 觀測極差，不進前三。

## Ranking C：若未來變成真正產品

1. **中國 Mainland 部署 + ICP + Tencent Cloud 或 Alibaba Cloud（加全球 region／CDN）**
   唯一能把中國 VPN OFF 可靠性、serverless/database/AI、合規與 scaling 正式納入設計的路線；代價是本地帳戶、ICP、運維與成本。
2. **Cloudflare Enterprise China Network + ICP + 全球 Cloudflare 架構**
   對已經深用 Cloudflare 的跨國產品有吸引力；但 Enterprise、ICP、JD Cloud 審核和產品 entitlement 遠超目前 side project 需求。
3. **Global hosting + China-specific mirror / API provider 的雙軌架構**
   未準備 ICP 時可作過渡；要明確告知其中國可達性仍無 SLA，資料同步、身份、支付和合規複雜度會上升。

---

## Recommended Decision Tree

```text
先以她自己的 Cloudflare Pages 帳戶部署同一個 GitHub repo
        |
        v
廈門 Wi-Fi + 行動網路（VPN OFF）都能冷開、reload、互動嗎？
        |
   +----+----+
   |         |
 YES       NO / 不穩
   |         |
   v         v
Cloudflare 保留為     先測 custom domain（不是 pages.dev）
side-project 主入口          |
   |                   +-----+-----+
24 小時後雙網複測        |           |
   |                   穩定        仍不穩
   v                    |           |
加 serverless provider   v           v
adapter；可先接       繼續 CF     匯入同 repo 到 EdgeOne Makers Global
Workers AI                         |
                                  v
                       廈門雙網 VPN OFF 穩嗎？
                           |              |
                         YES             NO
                           |              |
                           v              v
                    EdgeOne 作主入口    暫停把它當日常產品入口；
                                      做 Alibaba HK / Azure Global 受控對照，
                                      並決定是否進入 Mainland + ICP 正式方案
```

---

## 最終建議

> **「如果我們今天晚上只有兩個小時帶櫻井妹妹做 SocialPulse，現在到底應該先用哪個平台、怎麼做，以及什麼條件出現時我們才應該換平台？」**
>
> **先用她自己的 Cloudflare Pages + GitHub repo 部署，盡快加 custom domain，然後用她在廈門真正日常的 Wi-Fi 與行動網路在 VPN OFF 下測試；兩者都穩定就繼續 Cloudflare 並以 serverless provider adapter 逐步加 Workers AI，任一網路重複失敗就先測 custom domain、再無縫把同 repo 改部署到 Tencent EdgeOne Makers Global；只有連這條也無法日常使用時，才停止海外 side-project 方案、評估 Alibaba HK/Azure Global 對照或正式 Mainland + ICP 路線。**

---

## Sources

### 官方來源

1. Cloudflare, [China Network overview](https://developers.cloudflare.com/china-network/)（存取 2026-09-11）：Enterprise standalone subscription、JD Cloud、ICP、產品可用性限制。
2. Cloudflare, [Pages Functions](https://developers.cloudflare.com/pages/functions/)（存取 2026-09-11）：Pages 的 Workers runtime server-side 能力。
3. Cloudflare, [Pages limits](https://developers.cloudflare.com/pages/platform/limits/)（存取 2026-09-11）：Free 500 builds/month、custom domains、Functions 計入 Workers quota。
4. Cloudflare, [Workers AI pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/)（存取 2026-09-11）：10,000 Neurons/day、超額處理與 neuron 定價。
5. Cloudflare, [Workers AI JSON Mode](https://developers.cloudflare.com/workers-ai/features/json-mode/)（存取 2026-09-11）：`response_format`、JSON Schema、失敗行為與 non-streaming 限制。
6. Microsoft, [Azure Static Web Apps pricing](https://azure.microsoft.com/en-us/pricing/details/app-service/static/)（存取 2026-09-11）：Free hosting/SSL/custom domain/100GB、GitHub workflow、Azure Functions。
7. Microsoft, [Azure operated by 21Vianet](https://learn.microsoft.com/en-us/azure/china/overview-operations)（存取 2026-09-11）：中國 Azure 是實體分離、由 21Vianet 營運／交易，有 feature parity gap。
8. Tencent EdgeOne, [EdgeOne Makers product page](https://edgeone.ai/products/pages)（存取 2026-09-11）：Git-based deployment、Functions、Free plan、SPA/full-stack 與 AI 產品面。
9. Tencent EdgeOne, [Import a Git repository](https://pages.edgeone.ai/document/importing-a-git-repository)（存取 2026-09-11）：Git import deployment 文件。
10. Alibaba Cloud OSS, [Static Website Hosting](https://www.alibabacloud.com/help/en/oss/user-guide/static-website-hosting)（存取 2026-09-11）：OSS 靜態網站能力（頁面內容可能依地區／登入顯示）。

### 外部可達性測量（非官方、不可作 SLA）

11. OONI API/Explorer，中國探針一年期 domain measurements（存取 2026-09-11）：
    - [`vercel.app`](https://api.ooni.io/api/v1/measurements?probe_cc=CN&domain=vercel.app&since=2025-09-11&until=2026-09-11)
    - [`netlify.app`](https://api.ooni.io/api/v1/measurements?probe_cc=CN&domain=netlify.app&since=2025-09-11&until=2026-09-11)
    - [`github.io`](https://api.ooni.io/api/v1/measurements?probe_cc=CN&domain=github.io&since=2025-09-11&until=2026-09-11)

*本報告不以 SEO 農場文章作主要依據。中國跨境可達性只能用官方架構文件界定「是否受正式支援」，再以廈門實際使用者的 VPN OFF 雙網測試決定 SocialPulse 的短期平台。*

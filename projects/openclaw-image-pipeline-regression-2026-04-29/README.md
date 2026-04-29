# OpenClaw 2026.4.26 圖像識別忽然失效調查

> 日期：2026-04-29  
> 調查者：Jarvis  
> 主題：OpenClaw 圖片 / 截圖 / LINE 與 Slack 圖像識別在同版 2026.4.26 下突然失效

## 結論摘要

這次不是 LINE 單一 channel 壞掉，也不是 GPT-5.5 本身不能看圖。

目前最可能的根因是：**OpenClaw 2026.4.25 / 2026.4.26 的 shared media image optimization pipeline 發生包裝 / runtime dependency regression，導致 `sharp` 在 Gateway runtime 裡無法被 `media-understanding-core` 正確載入。**

結果是：

1. Slack 與 LINE 圖片檔都可以被 OpenClaw 收到並落地成檔案。
2. 但在送進 vision model 之前，OpenClaw 會先走 `loadWebMedia()` / `optimizeImageToJpeg()`。
3. 這條路徑需要 `sharp` 來讀 metadata、旋轉 EXIF、resize / JPEG optimize。
4. `sharp` 無法在目前 running Gateway process 中解析時，OpenClaw 會把真正錯誤吞掉，最後只回：`Failed to optimize image`。
5. 因為錯誤發生在「送模型前」，所以不論後面是 Codex GPT-5.5、OpenAI、Gemini、NVIDIA、OpenRouter 或其他 VLM，都看不到圖。

也就是說：**壞掉的是 OpenClaw 圖片前處理層，不是單純模型層。**

## Jones 的現象描述

Jones 觀察到：

- 同樣 OpenClaw 版本，看起來是 2026.4.26。
- 昨天圖像識別還很自然、絲滑。
- 今天沒有主動更新，卻突然 LINE 圖片不行。
- Slack 測試一開始看似可讀，但後來發現只是文字截圖被 OCR workaround 救到。
- 真照片仍然無法可靠辨識。

這個直覺是對的：Slack 截圖之所以「看似可讀」，不是 vision pipeline 正常，而是因為該圖是文字截圖，可以用本機 OCR / tesseract 讀出文字。當 Jones 傳一般照片時，Slack 也同樣卡住。

## 本機證據

### 1. OpenClaw 狀態

`openclaw status` 顯示：

```text
OpenClaw 2026.4.26 (be8c246)
Gateway service running pid 217
Model: openai-codex/gpt-5.5
Node: v25.6.0
```

### 2. LINE 圖片其實有收到

LINE session log 中，圖片落地為：

```text
/tmp/openclaw/line-media-1777461173303-bc90f52c-f443-4920-901a-b9e34a852b5c.jpg
```

檔案檢查：

```text
JPEG image data, 1108x1477
128K
```

所以不是 LINE 沒下載圖片。

### 3. Slack 圖片也有收到

Slack 附件落地為：

```text
/Users/jarvis/.openclaw/media/inbound/173f1bef-98cf-4262-bf99-c1a3f6d0d9de.jpg
```

檔案檢查：

```text
JPEG image data, 4032x2268
```

所以也不是 Slack 沒下載圖片。

### 4. image tool 失敗點一致

無論 LINE 圖、Slack 截圖、Slack 真照片，呼叫 `image` tool 都得到：

```text
Failed to optimize image
```

Gateway log 中對應紀錄：

```text
[tools] image failed: Failed to optimize image raw_params={...LINE image...}
[tools] image failed: Failed to optimize image raw_params={...Slack screenshot...}
[tools] image failed: Failed to optimize image raw_params={...Slack photo...}
```

### 5. `read` 圖片暴露更接近根因的錯誤

對圖片使用 `read` 時，OpenClaw 回：

```text
Optional dependency sharp is required for image attachment processing
```

這比 `Failed to optimize image` 更接近真正原因。

### 6. 原始碼路徑確認

本機 OpenClaw 2026.4.26 的相關檔案：

```text
/opt/homebrew/lib/node_modules/openclaw/dist/extensions/media-understanding-core/image-ops.js
/opt/homebrew/lib/node_modules/openclaw/dist/web-media-9UYLzIiN.js
```

`image-ops.js` 中的關鍵行為：

```js
const SHARP_MODULE = "sharp";
...
import(SHARP_MODULE)
...
throw new Error("Optional dependency sharp is required for image attachment processing", { cause: err });
```

`web-media-9UYLzIiN.js` 中的關鍵行為：

```js
for (const side of sides) for (const quality of qualities) try {
  const out = await resizeToJpeg(...)
  ...
} catch {}
...
throw new Error("Failed to optimize image");
```

這代表每一次 resize/optimize 失敗都會被 `catch {}` 吞掉，最後只丟出不透明的 `Failed to optimize image`。

## 為什麼「昨天好好的，今天忽然壞」？

目前最合理推論：**Gateway process / package layout / runtime deps 進入了不一致狀態。**

本機檢查顯示：

- 目前 `/opt/homebrew/lib/node_modules/openclaw/package.json` 已經有 `dependencies.sharp = ^0.34.5`。
- 從新的 shell process 進入 OpenClaw package 後，`import('sharp')` 可以成功。
- 直接從 `media-understanding-core/image-ops.js` 建立 image ops，也可以成功讀出 Slack 真照片 metadata 並 resize：

```text
{ width: 4032, height: 2268 }
53588
```

但 running Gateway 仍回 `Failed to optimize image`。這高度暗示：

- `sharp` 可能是在 Gateway 啟動後才被手動補裝；
- 或 Gateway 仍使用舊的 module resolution / runtime deps staging 狀態；
- 或 2026.4.26 的 packaged runtime-deps staging 沒有正確把 `media-understanding-core` 的 `sharp` 準備好，running process 需要重啟後才會吃到修補後的 node_modules。

重要的是：這不是圖片本身壞，也不是 LINE-only。這是 shared media preprocessing path 壞。

## 網路 / GitHub 類似回報

我查了 `openclaw/openclaw` issue / PR，找到多個高度相符的回報。

### Issue #73224 — 2026.4.26 sharp not installed

- URL: <https://github.com/openclaw/openclaw/issues/73224>
- 標題：`[Bug] 2026.4.26: sharp not installed with media-understanding-core extension — image optimization fails`
- 狀態：Closed as fixed-on-main
- 重點：
  - `media-understanding-core` 在 2026.4.26 新增 / 重構。
  - extension 宣告 `sharp`，但 npm global update 後沒有正確安裝到可解析位置。
  - 任何 image optimization 都可能失敗成 `Failed to optimize image`。
  - workaround 是手動在 OpenClaw install 目錄裝 `sharp@0.34.5`，然後 restart Gateway。

### Issue #73424 — image tool preprocessing pipeline failure

- URL: <https://github.com/openclaw/openclaw/issues/73424>
- 標題：`image tool: 'Failed to optimize image' error in preprocessing pipeline`
- 狀態：Open
- 重點：
  - 圖片在送 VLM 前就死在 preprocessing。
  - direct model API 可以成功，代表模型不是根因。
  - 多個 comment 回報 2026.4.26 / macOS / WhatsApp / Telegram / LINE 類似狀況。

### Issue #73148 — opaque error when sharp missing

- URL: <https://github.com/openclaw/openclaw/issues/73148>
- 標題：`Image tool: opaque "Failed to optimize image" when sharp is not installed`
- 狀態：Open
- 重點：
  - `image-ops.js` 動態 `import("sharp")`。
  - `optimizeImageToJpeg()` 會吞掉底層錯誤，造成 operator 只能看到 `Failed to optimize image`。
  - 建議 fix：把 `sharp` 變成 hard dependency，或至少 surface underlying error。

### Issue #74111 — staged sharp dependency cannot resolve

- URL: <https://github.com/openclaw/openclaw/issues/74111>
- 標題：`BUG: media-understanding-core cannot resolve staged sharp dependency for image processing`
- 狀態：Closed as implemented on current main
- 重點：
  - 即使 staged runtime deps 裡有 sharp，ESM `import("sharp")` 也可能因載入路徑而解析不到。
  - 修法在 main，但不一定已發布到目前 2026.4.26 release。

### PR #74216 — runtime deps for library extensions

- URL: <https://github.com/openclaw/openclaw/pull/74216>
- 標題：`fix(plugins): install runtime deps for library extensions`
- 狀態：Closed（但非本機目前 release 已證實包含）
- 重點：
  - Root cause 寫得非常符合本機狀況：
    - `media-understanding-core` 是 bundled library extension，沒有一般 plugin manifest。
    - runtime deps scanner 之前只看 plugin manifest，忽略 package.json 裡的 `stageRuntimeDependencies: true`。
    - 因此 `sharp` 沒被安裝，導致 image tools / native vision injection 都 `Failed to optimize image`。

### PR #73451 / #74243 — fallback/resilience fixes

- #73451: <https://github.com/openclaw/openclaw/pull/73451>
- #74243: <https://github.com/openclaw/openclaw/pull/74243>
- 重點：
  - 即使 optimizer 壞，也應該能在安全條件下 fallback 到原圖，讓 VLM 至少能收到圖。
  - 這是 user-facing resilience fix，與 `sharp` packaging fix 互補。

## 為什麼我是 Codex GPT-5.5，卻跑去 Gemini 幫忙識別？

這裡要分清楚三層：

### 1. 正常情況：Codex GPT-5.5 可以吃 image input

目前 OpenClaw model config 裡 `openai-codex/gpt-5.5` 有：

```json
"input": ["text", "image"]
```

理論上，如果 OpenClaw 把圖片作為 multimodal attachment 正常餵給目前 session model，GPT-5.5 應該可以直接讀圖。

### 2. 這次壞在送模型之前

錯誤發生在 OpenClaw shared image preprocessing：

```text
image file -> loadWebMedia / optimizeImageToJpeg -> sharp missing / optimization fail -> Failed to optimize image -> model never receives image
```

所以不是「GPT-5.5 不會看圖」，而是圖片根本沒有成功抵達 GPT-5.5。

### 3. Gemini 是我臨時亂找的 fallback，不是正規路徑

當 `image` tool 失敗時，我嘗試過幾種 workaround：

- `read` 圖片
- `sips` 轉圖
- tesseract OCR
- browser/canvas
- Gemini CLI

其中 Gemini CLI 是我試圖繞過 OpenClaw 內建 image tool 的臨時外部眼睛，不代表 OpenClaw 本來應該用 Gemini 來讀圖。這一步其實有點誤導，因為它把「OpenClaw 圖片前處理壞」和「外部 VLM fallback」混在一起了。

更正確的診斷方式應該是：

1. 先確認 image file 是否落地。
2. 測 `loadWebMediaRaw()` / metadata / optimizer。
3. 檢查 `sharp` resolution 與 Gateway runtime deps。
4. 再測 native multimodal attachment 是否能直接抵達目前 GPT-5.5 session。

## 目前建議處置

### 短期修復

由於我已確認新的 shell process 內 `sharp` 可以被載入，短期最可能有效的修復是：

1. 確認沒有正在進行的重要 long-running session。
2. 重啟 OpenClaw Gateway。
3. 用同一張 Slack 真照片、LINE 圖片重新測：
   - `image` tool 是否還是 `Failed to optimize image`
   - `read` 圖片是否還說缺 `sharp`
   - 原生 attachment 是否能直接讓 GPT-5.5 描述照片

注意：重啟前應先跟 Jones 確認，因為重啟可能中斷目前 sessions，而且重啟後不保證原 session 能自動回 LINE 群。

### 中期修復

等 OpenClaw 發布包含以下修復的版本後更新：

- bundled library extension runtime deps 正確 staging（PR #74216 類）
- image optimization 失敗時保留底層錯誤（PR #73217 類）
- 對有效且 under-cap 的圖片，在 optimizer 壞時 fallback 到原圖（PR #74243 類）

### 本機可做的驗證命令

```bash
cd /opt/homebrew/lib/node_modules/openclaw
node -e "import('sharp').then(m=>console.log('sharp ok')).catch(e=>console.error(e))"
```

直接測 `media-understanding-core`：

```bash
cd /opt/homebrew/lib/node_modules/openclaw/dist/extensions/media-understanding-core
node -e "import('./image-ops.js').then(async m=>{ const fs=require('fs'); const ops=m.createMediaAttachmentImageOps({maxInputPixels:50000000}); const buf=fs.readFileSync('/path/to/test.jpg'); console.log(await ops.getImageMetadata(buf)); })"
```

如果 shell 可用但 Gateway 仍失敗，就更支持「running Gateway process 沒吃到修補後 dependency / 需要 restart」這個判斷。

## 最終判斷

這次事件的根因優先排序：

1. **最高可能：OpenClaw 2026.4.26 `media-understanding-core` / `sharp` runtime dependency staging regression，running Gateway 無法解析 `sharp`。**
2. **次高可能：Gateway process 在手動安裝 `sharp` 之前已啟動，尚未重啟，因此仍處於壞狀態。**
3. **較低可能：LINE channel 特有 media ingestion 問題。** Slack 真照片同樣失敗，已降低 LINE-only 的可能性。
4. **較低可能：Codex GPT-5.5 vision 能力問題。** 因為失敗點在 model call 之前。

一句話版本：

> 圖片有進 OpenClaw，但在「送給 GPT-5.5 看」之前，被 OpenClaw 2026.4.26 的 image optimization / `sharp` dependency 問題攔掉了。Slack 截圖能讀只是 OCR 偶然救到文字，不代表 vision pipeline 正常。

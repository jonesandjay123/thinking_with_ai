# Firebase Hosting + GitHub Actions 自動部署完整教學

> **適用對象：** 有基礎程式能力的初學者（大學生程度）  
> **更新日期：** 2026-02-09  
> **Firebase CLI 版本：** 最新版 (13.x+)  

## 📖 目錄

1. [前置準備](#1-前置準備)
2. [Firebase 專案設定](#2-firebase-專案設定)
3. [Firebase CLI 安裝與設定](#3-firebase-cli-安裝與設定)
4. [本地專案設定](#4-本地專案設定)
5. [Firebase Hosting 初始化](#5-firebase-hosting-初始化)
6. [GitHub Actions 設定](#6-github-actions-設定)
7. [完整部署流程測試](#7-完整部署流程測試)
8. [常見問題與踩坑注意事項](#8-常見問題與踩坑注意事項)
9. [進階設定](#9-進階設定)
10. [參考文獻](#10-參考文獻)

---

## 1. 前置準備

### 1.1 需要的帳號和工具

- **Google 帳號**（用於 Firebase）
- **GitHub 帳號**（用於代碼託管）
- **終端機 / 命令列**（macOS Terminal、Windows PowerShell 或 Git Bash）
- **代碼編輯器**（VS Code、Sublime Text 等）

### 1.2 基本知識要求

- 基本的 Git 操作
- HTML/CSS/JavaScript 基礎
- 命令列操作基礎

---

## 2. Firebase 專案設定

### 2.1 建立 Firebase 專案

1. 前往 [Firebase Console](https://console.firebase.google.com/)
2. 點擊 **「新增專案」**
3. 輸入專案名稱（例如：`my-website-project`）
4. 選擇是否啟用 Google Analytics（建議啟用）
5. 點擊 **「建立專案」**

### 2.2 啟用 Firebase Hosting

1. 在 Firebase Console 左側選單點擊 **「Hosting」**
2. 點擊 **「開始使用」**
3. 先暫時跳過設定步驟，我們稍後會用 CLI 來設定

---

## 3. Firebase CLI 安裝與設定

### 3.1 安裝 Firebase CLI

Firebase CLI 提供多種安裝方式，選擇適合你的作業系統：

#### **Windows 用戶**

**方法 1：使用獨立執行檔（推薦給新手）**
```bash
# 下載並執行 Firebase CLI 執行檔
# 前往 https://firebase.tools/bin/win/instant/latest 下載
```

**方法 2：使用 npm（需要先安裝 Node.js）**
```bash
npm install -g firebase-tools
```

#### **macOS 用戶**

**方法 1：自動安裝腳本（推薦）**
```bash
curl -sL https://firebase.tools | bash
```

**方法 2：使用 npm**
```bash
npm install -g firebase-tools
```

#### **Linux 用戶**

**方法 1：自動安裝腳本（推薦）**
```bash
curl -sL https://firebase.tools | bash
```

**方法 2：下載獨立執行檔**
```bash
# 下載 Linux 版本
curl -Lo firebase https://firebase.tools/bin/linux/latest
chmod +x firebase
sudo mv firebase /usr/local/bin/
```

### 3.2 驗證安裝

```bash
firebase --version
```

### 3.3 登入 Firebase

```bash
firebase login
```

這會開啟瀏覽器進行 Google 帳號驗證。

### 3.4 驗證登入狀態

```bash
firebase projects:list
```

應該會顯示你的 Firebase 專案列表。

---

## 4. 本地專案設定

### 4.1 建立專案資料夾

```bash
mkdir my-website
cd my-website
```

### 4.2 初始化 Git 儲存庫

```bash
git init
```

### 4.3 建立基本的網站檔案

建立一個簡單的 `public` 資料夾和檔案：

```bash
mkdir public
```

**建立 `public/index.html`：**
```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的網站</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f0f0f0;
            margin: 0;
            padding: 50px;
        }
        .container {
            background: white;
            padding: 2rem;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #4285f4;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 歡迎來到我的網站</h1>
        <p>這是使用 Firebase Hosting + GitHub Actions 自動部署的網站！</p>
        <p id="deploy-time">部署時間：載入中...</p>
    </div>
    
    <script>
        document.getElementById('deploy-time').textContent = 
            '部署時間：' + new Date().toLocaleString('zh-TW');
    </script>
</body>
</html>
```

**建立 `public/404.html`：**
```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>404 - 找不到頁面</title>
</head>
<body>
    <h1>404 - 找不到頁面</h1>
    <p>抱歉，您尋找的頁面不存在。</p>
</body>
</html>
```

---

## 5. Firebase Hosting 初始化

### 5.1 初始化 Firebase Hosting

在專案根目錄執行：

```bash
firebase init hosting
```

### 5.2 設定選項

按照提示進行設定：

1. **選擇 Firebase 專案：** 選擇你剛才建立的專案
2. **公開目錄 (public directory)：** 輸入 `public`（預設值）
3. **單頁應用程式 (SPA)：** 根據需求選擇，一般靜態網站選 `N`
4. **覆寫現有檔案：** 選擇 `N`（保留我們建立的檔案）

### 5.3 檢查設定檔

初始化完成後，會產生以下檔案：

**`firebase.json`：**
```json
{
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ]
  }
}
```

**`.firebaserc`：**
```json
{
  "projects": {
    "default": "your-project-id"
  }
}
```

### 5.4 測試本地部署

```bash
firebase serve --only hosting
```

打開瀏覽器訪問 `http://localhost:5000` 檢查網站是否正常運作。

### 5.5 第一次手動部署

```bash
firebase deploy --only hosting
```

部署成功後，你會得到兩個網址：
- `https://your-project-id.web.app`
- `https://your-project-id.firebaseapp.com`

---

## 6. GitHub Actions 設定

### 6.1 建立 GitHub 儲存庫

1. 在 GitHub 建立新的儲存庫
2. 將本地代碼推送到 GitHub：

```bash
git add .
git commit -m "初始化專案"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

### 6.2 設定 Firebase 與 GitHub 整合

#### **方法一：使用 Firebase CLI 自動設定（推薦）**

```bash
firebase init hosting:github
```

這個指令會：
1. 建立 Firebase 服務帳號
2. 將服務帳號金鑰加密並上傳到 GitHub Secrets
3. 自動建立 GitHub Actions workflow 檔案

#### **方法二：手動設定**

如果自動設定失敗，可以手動進行：

**步驟 1：取得 Firebase 服務帳號金鑰**
1. 前往 [Google Cloud Console](https://console.cloud.google.com/)
2. 選擇你的 Firebase 專案
3. 左側選單 → IAM 與管理 → 服務帳號
4. 建立服務帳號，命名為 `github-actions`
5. 角色選擇：`Firebase Hosting Admin` 和 `Firebase Admin`
6. 建立金鑰（JSON 格式）並下載

**步驟 2：設定 GitHub Secrets**
1. 前往 GitHub 儲存庫 → Settings → Secrets and variables → Actions
2. 點擊 `New repository secret`
3. 名稱：`FIREBASE_SERVICE_ACCOUNT`
4. 值：貼上剛才下載的 JSON 檔案內容

### 6.3 建立 GitHub Actions Workflow

建立 `.github/workflows/` 資料夾：

```bash
mkdir -p .github/workflows
```

**建立 `.github/workflows/firebase-hosting-merge.yml`：**
```yaml
name: Deploy to Live Channel

on:
  push:
    branches:
      - main
    # 可選：只有特定檔案變更時才觸發
    # paths:
    #   - "public/**"

jobs:
  deploy_live_website:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # 如果有需要建置步驟，在這裡加入
      # - name: Install dependencies
      #   run: npm ci
      # - name: Build project
      #   run: npm run build

      - name: Deploy to Firebase
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: '${{ secrets.GITHUB_TOKEN }}'
          firebaseServiceAccount: '${{ secrets.FIREBASE_SERVICE_ACCOUNT }}'
          channelId: live
          projectId: YOUR_FIREBASE_PROJECT_ID
```

**建立 `.github/workflows/firebase-hosting-pull-request.yml`：**
```yaml
name: Deploy to Preview Channel

on:
  pull_request:
    # 可選：只有特定檔案變更時才觸發
    # paths:
    #   - "public/**"

jobs:
  build_and_preview:
    if: '${{ github.event.pull_request.head.repo.full_name == github.repository }}'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # 如果有需要建置步驟，在這裡加入
      # - name: Install dependencies
      #   run: npm ci
      # - name: Build project
      #   run: npm run build

      - name: Deploy to Preview
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: '${{ secrets.GITHUB_TOKEN }}'
          firebaseServiceAccount: '${{ secrets.FIREBASE_SERVICE_ACCOUNT }}'
          expires: 30d
          projectId: YOUR_FIREBASE_PROJECT_ID
```

**⚠️ 重要：** 將上面 workflow 檔案中的 `YOUR_FIREBASE_PROJECT_ID` 替換為你的實際專案 ID。

---

## 7. 完整部署流程測試

### 7.1 提交 GitHub Actions 設定

```bash
git add .
git commit -m "新增 GitHub Actions 自動部署設定"
git push origin main
```

### 7.2 檢查 GitHub Actions 執行狀況

1. 前往 GitHub 儲存庫
2. 點擊 **Actions** 標籤
3. 查看 workflow 執行結果

### 7.3 測試 Pull Request 預覽

1. 建立新分支：
```bash
git checkout -b feature/update-content
```

2. 修改 `public/index.html`，例如更改標題：
```html
<h1>🚀 更新後的網站標題</h1>
```

3. 提交並推送：
```bash
git add .
git commit -m "更新網站標題"
git push origin feature/update-content
```

4. 在 GitHub 建立 Pull Request
5. 檢查 Actions 是否建立預覽頻道並留言提供預覽網址

### 7.4 合併到主分支

1. 合併 Pull Request
2. 檢查主分支的 Actions 是否自動部署到正式環境

---

## 8. 常見問題與踩坑注意事項

### 8.1 Firebase CLI 相關問題

**問題：`firebase command not found`**
```bash
# 解決方法：確認 PATH 設定
echo $PATH
# 重新安裝 Firebase CLI
npm install -g firebase-tools
```

**問題：權限不足**
```bash
# macOS/Linux 可能需要 sudo
sudo npm install -g firebase-tools
```

**問題：網路連線問題**
```bash
# 中國大陸用戶可能需要設定代理
npm config set registry https://registry.npmmirror.com
```

### 8.2 GitHub Actions 相關問題

**問題：`FIREBASE_SERVICE_ACCOUNT` secret 設定錯誤**
- 確認 JSON 格式正確
- 確認服務帳號有正確權限
- 檢查專案 ID 是否正確

**問題：Workflow 不觸發**
- 檢查分支名稱是否為 `main`
- 確認 workflow 檔案路徑正確：`.github/workflows/`
- 檢查 YAML 語法是否正確

**問題：部署失敗**
```yaml
# 在 workflow 中加入除錯資訊
- name: Debug Firebase Project
  run: |
    echo "Project ID: ${{ secrets.FIREBASE_PROJECT_ID }}"
    firebase projects:list
```

### 8.3 Firebase Hosting 相關問題

**問題：404 錯誤**
- 檢查 `firebase.json` 中的 `public` 路徑
- 確認 `index.html` 存在於正確位置

**問題：快取問題**
```bash
# 強制重新整理瀏覽器
# Ctrl+F5 (Windows) 或 Cmd+Shift+R (macOS)
```

**問題：自訂網域設定**
```json
// firebase.json 中加入
{
  "hosting": {
    "public": "public",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "headers": [
      {
        "source": "**/*.@(js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000"
          }
        ]
      }
    ]
  }
}
```

### 8.4 效能優化建議

**1. 啟用壓縮：**
```json
{
  "hosting": {
    "public": "public",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "headers": [
      {
        "source": "**/*.@(html|js|css)",
        "headers": [
          {
            "key": "Content-Encoding",
            "value": "gzip"
          }
        ]
      }
    ]
  }
}
```

**2. 設定快取策略：**
```json
{
  "hosting": {
    "public": "public",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "headers": [
      {
        "source": "**/*.@(jpg|jpeg|gif|png|webp)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000"
          }
        ]
      }
    ]
  }
}
```

---

## 9. 進階設定

### 9.1 多環境部署

**設定多個 Firebase 專案：**
```bash
firebase use --add
# 選擇 staging 專案並命名為 staging
firebase use --add
# 選擇 production 專案並命名為 production
```

**切換環境：**
```bash
firebase use staging
firebase use production
```

### 9.2 自訂 Firebase Hosting 設定

**完整的 `firebase.json` 範例：**
```json
{
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(eot|otf|ttf|ttc|woff|font.css)",
        "headers": [
          {
            "key": "Access-Control-Allow-Origin",
            "value": "*"
          }
        ]
      },
      {
        "source": "**/*.@(js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000"
          }
        ]
      }
    ],
    "cleanUrls": true,
    "trailingSlash": false
  }
}
```

### 9.3 與 Node.js 專案整合

**如果你的專案需要建置步驟：**

**1. 建立 `package.json`：**
```json
{
  "name": "my-website",
  "version": "1.0.0",
  "scripts": {
    "build": "npm run build:css && npm run build:js",
    "build:css": "postcss src/css/main.css -o public/css/main.css",
    "build:js": "webpack --mode production"
  },
  "devDependencies": {
    "postcss": "^8.4.0",
    "webpack": "^5.0.0"
  }
}
```

**2. 更新 GitHub Actions workflow：**
```yaml
- name: Install dependencies
  run: npm ci

- name: Build project
  run: npm run build

- name: Deploy to Firebase
  uses: FirebaseExtended/action-hosting-deploy@v0
  # ... 其他設定
```

---

## 10. 參考文獻

### 10.1 官方文件
- [Firebase Hosting 官方文檔](https://firebase.google.com/docs/hosting)
- [Firebase CLI 參考手冊](https://firebase.google.com/docs/cli)
- [Firebase GitHub Actions 整合](https://firebase.google.com/docs/hosting/github-integration)
- [GitHub Actions Marketplace - Deploy to Firebase Hosting](https://github.com/marketplace/actions/deploy-to-firebase-hosting)

### 10.2 工具與資源
- [Firebase Console](https://console.firebase.google.com/)
- [GitHub Actions 文檔](https://docs.github.com/en/actions)
- [Firebase Tools (安裝腳本)](https://firebase.tools/)

### 10.3 社群資源
- [Firebase 官方 GitHub](https://github.com/firebase/firebase-tools)
- [Firebase Community Slack](https://firebase.community/)

---

## 🎉 結語

恭喜你完成了 Firebase Hosting + GitHub Actions 自動部署的完整設定！

現在你的網站已經具備：
- ✅ 自動部署：推送到 `main` 分支自動部署到正式環境
- ✅ 預覽功能：Pull Request 自動建立預覽版本
- ✅ 高效能：Firebase Hosting 的全球 CDN
- ✅ 免費額度：適合個人專案和小型網站

### 下一步建議

1. **學習 Firebase 其他功能**：Database、Authentication、Cloud Functions
2. **優化建置流程**：加入 CSS/JS 壓縮、圖片優化
3. **設定自訂網域**：使用你自己的網域名稱
4. **監控與分析**：設定 Google Analytics、Performance Monitoring

### 需要幫助？

如果遇到問題，可以：
1. 檢查本文的「常見問題」章節
2. 查閱官方文檔
3. 在 GitHub Issues 或 Stack Overflow 尋求協助

---

**文檔版本：** v2.0  
**最後更新：** 2026-02-09  
**作者：** Jarvis AI Assistant
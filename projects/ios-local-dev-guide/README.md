# 免費 iOS App 本地開發完整指南

## 概述

這份指南將教你如何在**沒有付費 Apple Developer Program** 的情況下，在本地開發 iOS App 並安裝到自己的 iPhone 上。我們將介紹兩種主流方案，適合有基礎程式能力的初學者（大學生程度）。

> **更新日期**：2026年2月  
> **適用版本**：Xcode 16+、iOS 18+、Flutter 3.24+

## 🎯 兩種開發方案

### 方案 A：Xcode + Swift（原生開發）
- **優點**：原生性能、完整 iOS 功能支援、Apple 官方工具
- **缺點**：只能開發 iOS、學習曲線較陡

### 方案 B：Flutter（跨平台開發）
- **優點**：一次開發支援 iOS/Android、熱重載、豐富生態系
- **缺點**：部分原生功能需要額外配置

---

## 📱 方案 A：Xcode + Swift 原生開發

### 1. 系統需求

- **macOS**：Sonoma 14.0+ (建議 Sequoia 15.0+)
- **Xcode**：16.0+ (支援 iOS 18)
- **iPhone**：iOS 16.0+ (建議 iOS 18+)
- **儲存空間**：至少 15GB 可用空間

### 2. 安裝 Xcode

#### 方法一：App Store 安裝（推薦）
```bash
# 檢查系統版本
sw_vers

# 從 App Store 安裝 Xcode
# 搜尋 "Xcode" 並點擊安裝（約 6-8GB 下載）
```

#### 方法二：開發者網站下載
1. 前往 [developer.apple.com/download](https://developer.apple.com/download)
2. 使用 Apple ID 登入
3. 下載對應的 Xcode 版本

### 3. 設定免費 Apple 開發者帳號

1. **開啟 Xcode**
2. **導航到偏好設定**：`Xcode` → `Settings...` → `Accounts`
3. **新增帳號**：
   - 點擊 `+` 按鈕
   - 選擇 `Apple ID`
   - 輸入你的 Apple ID 和密碼
4. **驗證帳號**：
   - 系統會要求雙重驗證
   - 輸入從其他 Apple 裝置收到的驗證碼

### 4. 建立第一個 Swift 專案

1. **新建專案**：
   ```
   Xcode → File → New → Project
   ```

2. **選擇範本**：
   - 選擇 `iOS` 頁籤
   - 選擇 `App`
   - 點擊 `Next`

3. **專案設定**：
   ```
   Product Name: MyFirstApp
   Team: [你的個人團隊]
   Organization Identifier: com.yourname.myfirstapp
   Bundle Identifier: com.yourname.myfirstapp (自動生成)
   Language: Swift
   Interface: SwiftUI (推薦) 或 Storyboard
   ```

### 5. 設定 Code Signing

這是**最重要的步驟**！

1. **選擇專案設定**：
   - 在左側導航器中點擊專案名稱
   - 選擇 `TARGETS` 下的 App 名稱

2. **Signing & Capabilities 設定**：
   ```
   Team: 選擇你的個人團隊 (Personal Team)
   Bundle Identifier: 確保是唯一的（如：com.yourname.myfirstapp）
   Automatically manage signing: ✅ 勾選
   ```

3. **處理 Bundle Identifier 衝突**：
   - 如果出現紅色錯誤訊息
   - 修改 Bundle Identifier 為獨特名稱
   - 例如：`com.yourname.myfirstapp2026`

### 6. 準備 iPhone 裝置

1. **啟用開發者模式**（iOS 16+）：
   ```
   設定 → 隱私權與安全性 → 開發者模式 → 開啟
   ```

2. **連接 iPhone**：
   - 使用 Lightning/USB-C 線連接
   - 首次連接時選擇「信任此電腦」

3. **在 Xcode 中選擇裝置**：
   - 頂部工具欄選擇你的 iPhone
   - 應該顯示為「[你的iPhone名稱] (iOS版本)」

### 7. 建置並安裝 App

1. **建置專案**：
   ```
   產品選單 → Build (⌘+B)
   ```

2. **安裝到 iPhone**：
   ```
   產品選單 → Run (⌘+R)
   ```

3. **授權 App**（首次安裝）：
   - iPhone 上：`設定` → `一般` → `VPN與裝置管理`
   - 找到你的開發者帳號
   - 點擊「信任」

### 8. 免費版本限制

#### 🚫 主要限制

| 限制項目 | 免費版 | 付費版 ($99/年) |
|---------|--------|-----------------|
| **App 數量** | 最多 3 個 | 無限制 |
| **憑證有效期** | 7 天 | 1 年 |
| **App Store 發布** | ❌ | ✅ |
| **TestFlight** | ❌ | ✅ |
| **Push Notifications** | ❌ | ✅ |
| **In-App Purchase** | ❌ | ✅ |
| **CloudKit** | ❌ | ✅ |

#### 📅 7天重新簽名問題

免費版憑證每 7 天過期一次，App 會無法啟動。

### 9. 延長使用技巧

#### 方法一：定期重新 Build
```bash
# 每週執行一次
# 在 Xcode 中按 ⌘+R 重新安裝
```

#### 方法二：自動化腳本
```bash
#!/bin/bash
# auto-resign.sh
cd /path/to/your/project
xcodebuild -workspace MyApp.xcworkspace \
  -scheme MyApp \
  -configuration Debug \
  -destination 'platform=iOS,name=你的iPhone名稱' \
  -allowProvisioningUpdates
```

#### 方法三：使用 Wireless 安裝
1. **啟用無線調試**：
   - iPhone 連接電腦後
   - `Window` → `Devices and Simulators`
   - 勾選「Connect via network」

2. **之後可無線安裝**：
   - 不需 USB 線
   - 確保在同一 WiFi 網路

---

## 🌐 方案 B：Flutter 跨平台開發

### 1. 安裝 Flutter SDK

#### 方法一：Homebrew 安裝（推薦）
```bash
# 安裝 Homebrew（如果還沒有）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安裝 Flutter
brew install flutter

# 驗證安裝
flutter --version
```

#### 方法二：手動安裝
```bash
# 下載 Flutter SDK
cd ~/Downloads
curl -O https://storage.googleapis.com/flutter_infra_release/releases/stable/macos/flutter_macos_arm64_stable.zip

# 解壓縮
unzip flutter_macos_arm64_stable.zip

# 移動到應用程式目錄
mv flutter ~/Applications/

# 新增到 PATH
echo 'export PATH="$HOME/Applications/flutter/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### 2. 安裝依賴

Flutter iOS 開發仍需要 Xcode：

```bash
# 檢查系統依賴
flutter doctor

# 安裝 CocoaPods
sudo gem install cocoapods

# 接受 Xcode 授權
sudo xcodebuild -license accept
```

### 3. 建立 Flutter 專案

```bash
# 建立新專案
flutter create my_flutter_app

# 進入專案目錄
cd my_flutter_app

# 檢查專案結構
ls -la
```

### 4. 設定 iOS 簽名

1. **開啟 iOS 專案設定**：
   ```bash
   open ios/Runner.xcworkspace
   ```

2. **在 Xcode 中設定簽名**：
   - 選擇 `Runner` target
   - `Signing & Capabilities`
   - 設定與 Swift 專案相同的簽名設定

### 5. 執行在 iPhone 上

```bash
# 列出可用裝置
flutter devices

# 執行在 iPhone 上
flutter run -d "你的iPhone名稱"

# 或者指定 iOS 平台
flutter run --flavor ios
```

### 6. Flutter 特有注意事項

#### 🔧 常見設定問題

1. **Bundle Identifier 設定**：
   ```
   檔案位置：ios/Runner.xcodeproj/project.pbxproj
   或在 Xcode 中直接修改
   ```

2. **最低 iOS 版本設定**：
   ```dart
   // ios/Podfile
   platform :ios, '12.0'  // 根據需要調整
   ```

3. **權限設定**：
   ```xml
   <!-- ios/Runner/Info.plist -->
   <key>NSCameraUsageDescription</key>
   <string>App needs camera access</string>
   ```

#### 📦 常用 Flutter 套件

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^1.1.0           # 網路請求
  shared_preferences: ^2.2.2  # 本地儲存
  camera: ^0.10.5         # 相機功能
  location: ^5.0.3        # GPS 定位
```

#### 🔥 熱重載功能

Flutter 的殺手級功能：
```bash
# App 運行時，修改程式碼後按
r  # 熱重載
R  # 完全重啟
q  # 退出
```

---

## 🆚 兩種方案比較

| 特性 | Xcode + Swift | Flutter |
|------|---------------|---------|
| **開發速度** | 中等 | 快（熱重載） |
| **性能** | 原生最佳 | 接近原生 |
| **學習成本** | 較高 | 中等 |
| **跨平台** | 否 | 是 |
| **社群資源** | 豐富 | 快速成長 |
| **就業機會** | iOS 專門 | 跨平台優勢 |

## 🚀 進階技巧

### 1. 版本控制
```bash
# 初始化 Git
git init
git add .
git commit -m "Initial commit"

# 忽略檔案設定
echo "*.xcodeproj/xcuserdata/" >> .gitignore
echo "build/" >> .gitignore
```

### 2. 測試
```swift
// Swift 單元測試
import XCTest

class MyAppTests: XCTestCase {
    func testExample() {
        XCTAssertEqual(2 + 2, 4)
    }
}
```

```dart
// Flutter 測試
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('Counter value should be incremented', () {
    expect(2 + 2, 4);
  });
}
```

### 3. 除錯技巧

#### Xcode 除錯
```swift
// 使用 print 除錯
print("Debug: \(variableName)")

// 使用中斷點
// 點擊行號左側設定中斷點
```

#### Flutter 除錯
```dart
// 使用 print 除錯
print('Debug: $variableName');

// 使用 debugPrint（在 Release 模式不會輸出）
debugPrint('Debug info');

// 視覺除錯
import 'package:flutter/rendering.dart';
debugPaintSizeEnabled = true;  // 顯示 widget 邊界
```

---

## ⚠️ 常見問題排解

### iOS 簽名問題

**問題**：「Signing certificate is invalid」
```
解決方案：
1. 刪除舊的憑證
2. 重新登入 Apple ID
3. 清理 Xcode 快取：⌘+Shift+K
4. 重新建置專案
```

**問題**：「App installation failed」
```
解決方案：
1. 確認 iPhone 已信任電腦
2. 檢查 USB 線連接
3. 重啟 Xcode 和 iPhone
4. 確認開發者模式已啟用
```

### Flutter 問題

**問題**：「CocoaPods not installed」
```bash
# 重新安裝 CocoaPods
sudo gem uninstall cocoapods
sudo gem install cocoapods
cd ios && pod install
```

**問題**：「Flutter doctor 顯示錯誤」
```bash
# 檢查詳細錯誤
flutter doctor -v

# 常見解決方案
flutter clean
flutter pub get
cd ios && pod install
```

---

## 📚 學習資源

### 官方文件
- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [Flutter Documentation](https://docs.flutter.dev/)
- [Swift Programming Language](https://swift.org/documentation/)

### 推薦課程
- **Swift**: Stanford CS193p iOS Development
- **Flutter**: Flutter & Dart - The Complete Guide
- **iOS Design**: Apple Human Interface Guidelines

### 社群資源
- [Stack Overflow](https://stackoverflow.com/questions/tagged/ios)
- [Flutter Community](https://flutter.dev/community)
- [iOS Dev Reddit](https://reddit.com/r/iOSProgramming)

### YouTube 頻道
- **Swift**: Sean Allen, CodeWithChris
- **Flutter**: Flutter Official, Reso Coder
- **中文資源**: 彭彭的課程, 六角學院

---

## 🎯 下一步建議

### 如果選擇 Swift 原生開發：
1. 學習 SwiftUI 現代 UI 框架
2. 掌握 UIKit 傳統開發模式
3. 學習 Core Data 資料管理
4. 探索 ARKit、Core ML 等進階功能

### 如果選擇 Flutter：
1. 深入學習 Dart 語言特性
2. 掌握 State Management（Provider、Bloc）
3. 學習原生插件開發
4. 探索 Firebase 整合

### 共同建議：
1. **多做專案**：從簡單的計算機開始
2. **參與開源**：貢獻 GitHub 專案
3. **建立作品集**：展示你的 App
4. **持續學習**：關注 iOS 18+ 新特性

---

## 🧪 實戰驗證紀錄（2026-02-10）

> 以下是在 Jarvis Mac Mini 上實際走完「從零到 iPad 跑起 Hello World」的完整紀錄，含踩過的坑與解法。

### 環境

| 項目 | 規格 |
|------|------|
| Mac | Mac Mini M 系列、24GB RAM、macOS Sequoia |
| Apple ID | jarvis.mac.ai@gmail.com（免費，未付 $99） |
| 測試裝置 | iPad（iPadOS 26.x）+ iPhone 6S（iOS 15.8.6） |
| Xcode | App Store 最新版 |

### 完整 SOP（照做即可）

#### Step 1：安裝 Xcode

1. App Store 搜尋 **Xcode** → 安裝（⚠️ 不要裝「Apple Developer」，那是資訊 App，不能寫程式）
2. 第一次開啟讓它跑完 **Installing additional components**
3. Terminal 執行：
   ```bash
   sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
   sudo xcodebuild -license accept
   ```

#### Step 2：登入 Apple ID（確認 Personal Team）

1. Xcode → **Settings** → **Accounts**
2. 點 `+` → Apple ID → 登入（會需要 2FA 驗證碼）
3. 點進帳號，確認看到 **Personal Team**
4. 看到 `0 Provisioned Devices` 是正常的（第一次 Run 後會自動變 1）

> **重要：** 不需要去 Apple Developer 網站註冊任何東西，也不需要手動綁定裝置。現代 Xcode 在第一次 ⌘R 時會自動處理所有 signing/provisioning。

#### Step 3：準備實體裝置（建議插線）

**iPad/iPhone 端：**
1. 用 USB 線連接 Mac
2. 跳出「信任此電腦？」→ **信任**，輸入解鎖密碼
3. 開啟開發者模式（iPadOS/iOS 16+）：
   - 設定 → 隱私權與安全性 → **開發者模式** → 開啟
   - **必須重新開機一次**（不重開 → Xcode 不會給你 Wi-Fi 選項）

**Mac / Xcode 端：**
1. Xcode → **Window → Devices and Simulators**
2. 左側應看到你的裝置
3. 可能顯示「Preparing...」→ 等它跑完

#### Step 4：建立空白 SwiftUI 專案

1. Xcode → **File → New → Project**
2. 選 **iOS → App → Next**
3. 專案設定：

| 欄位 | 建議值 | 說明 |
|------|--------|------|
| Product Name | `VidCatalog`（或任意） | App 名稱 |
| Team | Personal Team | 選你的免費 Team |
| Organization Identifier | `com.jarvismac` | 反域名格式 |
| Interface | **SwiftUI** | 現代 UI 框架 |
| Language | **Swift** | |
| Testing System | **None** ✅ | 第一個 App 不需要測試框架 |
| Storage | **None** ✅ | 第一個 App 不需要資料庫 |

> **Testing System 和 Storage 一律選 None。** 這兩個是進階功能，選了會讓專案多一堆新手看不懂的檔案。

#### Step 5：Run 到實體裝置

1. **關鍵：** 確認上方 Run Destination **不是 My Mac**
2. 點下拉選單 → 選你的 iPad/iPhone
3. 按 **▶️**（或 ⌘R）
4. 第一次可能需要在裝置上信任開發者：
   - iPad/iPhone → 設定 → 一般 → **VPN 與裝置管理**
   - 找到你的 Apple ID → **信任**
5. 成功後 iPad 桌面出現 App，打開看到 **Hello, world!** 🎉

#### Step 6：Wi-Fi Debug（可選，第二階段）

1. **前提：** 必須先用插線成功跑過一次
2. Xcode → Window → Devices and Simulators → 選裝置
3. 勾選 **Connect via network**（有時插線成功後會自動可用）
4. 條件：Mac 與裝置在同一 Wi-Fi、裝置解鎖、不是 Guest/公司隔離網路

### 踩過的坑

#### 坑 1：「iOS XX.X is not installed」

```
iOS 26.2 is not installed. Please download and install the platform 
from Xcode > Settings > Components
```

**原因：** Xcode 剛裝好，還沒下載對應 iOS 版本的 platform support（約 8GB）。

**解法：** Xcode → Settings → Components → 下載對應版本。第一次必經，之後不用重來。

#### 坑 2：iPhone 6S 太舊

```
Your iPhone's iOS 15.8.6 doesn't match app's iOS 26.x deployment target
```

**原因：** Xcode 新建專案預設用最新 iOS 當 Deployment Target，但 iPhone 6S 最高只能 iOS 15。

**解法 A（推薦）：** 換較新的裝置。
**解法 B：** 專案 → Targets → General → Deployment Target 改為 `15.0`。

> **教學建議：** 第一堂課用新裝置，避免版本問題打擊信心。Deployment Target 降版當第二堂加分題。

#### 坑 3：App Store 裡 Xcode vs Apple Developer 搞混

App Store 搜 Xcode 會同時出現：
- **Xcode** — 開發工具，要裝這個 ✅
- **Apple Developer** — 資訊/帳號管理 App，不需要 ❌

#### 坑 4：「不受信任的開發者」

App 裝上去了但打不開 → iPad 設定 → 一般 → VPN 與裝置管理 → 信任開發者。

### 教學建議（給要帶新手的人）

1. **第一次一定插線** — Wi-Fi 連線的變數太多，新手容易卡在「裝置看不到」
2. **Testing System / Storage 選 None** — 一句話帶過：「等你開始寫邏輯再加」
3. **7 天過期這樣解釋：**「跟手機充電一樣，沒壞，只是期限到了，回 Xcode 按一次 ▶️ 就好」
4. **不要在第一堂提 Wi-Fi Debug** — 成功跑起來再教，當作「驚喜加分」
5. **Deployment Target 問題** — 「Xcode 預設用最新 iOS，手機舊的話要調低」

### 狀態（2026-02-10 驗證完成）

- ✅ Xcode 安裝並設定完成
- ✅ Apple ID 登入，Personal Team 可用
- ✅ SwiftUI 空白專案建立（Testing: None, Storage: None）
- ✅ 插線連 iPad，成功 Build & Run
- ✅ 開發者憑證已信任，App 正常啟動
- ✅ Wi-Fi Debug 可用
- ✅ 整條流程確認可用於教學

---

## 💡 總結

免費開發 iOS App 完全可行，主要限制是 7 天重新簽名和功能限制。對於學習和個人使用來說已經足夠。

**選擇建議**：
- **純 iOS 開發**：選擇 Swift + Xcode
- **跨平台需求**：選擇 Flutter
- **快速原型**：Flutter 更適合
- **性能要求極高**：Swift 原生開發

記住，最重要的是**開始動手做**！選擇一個方案，建立你的第一個 App，然後持續改進。

---

## 🔗 參考資料

1. [Apple Developer Program](https://developer.apple.com/programs/) - Apple 官方開發者計劃
2. [Xcode User Guide](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/Xcode_Overview/) - Xcode 官方指南
3. [Flutter Installation Guide](https://docs.flutter.dev/get-started/install) - Flutter 安裝指南
4. [iOS App Development with Swift](https://developer.apple.com/swift/) - Swift 開發資源
5. [Free iOS App Development](https://medium.com/@abhimuralidharan/how-to-deploy-an-ios-app-on-a-real-device-with-a-free-apple-developer-account-8bf47b6d39b4) - 免費開發教學

---

*最後更新：2026年2月10日*  
*如有問題或建議，歡迎提出 Issue 或 Pull Request*
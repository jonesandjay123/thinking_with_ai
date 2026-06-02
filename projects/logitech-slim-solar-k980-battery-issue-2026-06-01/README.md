# Logitech Signature Slim Solar+ K980 電池與充電異常調查

> 日期：2026-06-01
>
> 目的：調查 Jones 的 Logi / Logitech Signature Slim Solar+ K980 太陽能鍵盤為何使用不到半年就長期顯示約 5% 電量、疑似充不進電，並整理網路使用回饋與處理建議。
>
> 結論先行：如果照官方方式強光補電後仍無法恢復，這不應被視為正常耗電，應優先走 Logitech 保固或退換，而不是自行拆修。

## 1. 型號確認

Jones 說的「logic slim solar+」應是 Logitech / Logi 的：

```text
Logitech Signature Slim Solar+ K980
```

這款鍵盤的核心設計是 `Logi LightCharge`：

- 沒有 USB-C 充電口。
- 沒有可直接抽換的 AA / AAA 電池。
- 只能靠頂部吸光條從自然光或人工光補電。
- 內部是可更換的專用鋰電池 hardpack，但不是日常換電池設計。

官方規格寫明電池是 `70mAh, 3.6V` 鋰電池，完全充滿時可在黑暗中使用最長約 4 個月，電池設計壽命宣稱最高 10 年；但壽命會受使用、環境與運算條件影響。

## 2. 官方對低電量 / 沒電的說法

官方支援文件對「電池耗乾」的處理方式很直接：

```text
關閉鍵盤，放在大於 1,000 lux 的明亮光源下至少 8 小時，之後再開機檢查 LED。
```

官方也特別提醒：

- 不建議放在戶外直射陽光下。
- 過熱可能讓鍵盤停止充電。
- 平常應定期讓鍵盤接觸足夠光線。
- 避免長時間在黑暗或低光環境使用。
- 可以用 Logi Options+ 的 `LightCheck` 功能檢查桌面光線是否足以維持充電。

其他官方入門文件補充：

- K980 在至少 200 lux 的桌燈、天花板燈或自然光下可以有效充電。
- 電量 11% 到 100% 時 LED 維持綠色。
- 電量低於 11% 時 LED 變紅。
- 如果電量到 0%，需要用 1,000 lux 以上明亮光線補電。
- 太陽能板上不要放東西，也不要讓障礙物遮住吸光條。

## 3. 網路評價與使用回饋

### 整體評價

多數媒體評測對 K980 的「不用插線、不用換電池」概念評價正面，但共同指出它有幾個設計限制：

- 沒有 USB 充電備援。
- 沒有背光。
- 沒有可調腳架。
- 若太陽能板或電池出問題，鍵盤會很快變成無法使用的裝置。

WIRED 評測特別指出，這款鍵盤唯一充電方式就是頂部太陽能板；如果太陽能板停止工作或電池失效，因為不能插線使用，鍵盤在耗電後就會變成 paperweight。這點正好對應 Jones 現在遇到的風險。

### 光線不足不是假問題

Notebookcheck 的實測很值得參考：

- 在桌角，即使是晴天，太陽能板量到的光照也可能低於 100 lux。
- 靠窗位置約可到 1,300 lux。
- 他們把鍵盤放在黑暗中兩週後，電量降到 15% 並跳出低電量警告。
- 之後雖可繼續使用，但在暗季節、桌角位置不容易把電量充回去。
- 追加測試提到，秋天日光下放一天大約只回升 5% 左右，充電速度不快。

這代表：K980 的「有室內光就能充」不是說任何昏暗桌面都能快速補電。尤其如果鍵盤平常在窗邊以外、晚上使用、桌燈角度沒有照到上方吸光條，長期可能慢慢淨耗電。

### 但 Jones 的狀況仍偏異常

光線不足可以解釋「慢慢掉電」或「回充很慢」，但比較難完全解釋：

- 使用不到半年就掉到 5%。
- 長時間顯示低電量。
- 感覺怎麼充都充不回來。
- 最後突然完全沒電。

如果 Jones 平常不是長期在暗房使用，這比較像：

1. 長期光線不足造成深度耗電，且補電條件不夠強。
2. Logi Options+ 電量顯示或低光通知有 bug。
3. 太陽能板、光感測、充電管理電路或內部電池 hardpack 有瑕疵。
4. 電池進入過深放電狀態，需要官方建議的長時間強光補電才醒得回來。

目前網路上已有 K980 使用者回報 Logi Options+ 的 `Insufficient lighting` 通知可能與實際狀態矛盾：軟體一邊顯示光線足夠、電量 100%，一邊仍反覆跳低光警告。這比較像軟體通知 / 判定 bug，而不是必然代表硬體壞。

但 Jones 的 case 是電量真的掉到 5% 並完全沒電，嚴重程度比「通知亂跳」高。

## 4. 可能原因排序

### A. 桌面光線長期低於實際維持門檻

可能性：中高。

官方說 200 lux 以上可充，但實測顯示桌面位置差很多。鍵盤在桌角、遠離窗邊、晚上使用、或燈光沒有直接照到頂部吸光條時，可能長期處於淨放電。

判斷方式：

- 安裝 / 開啟 Logi Options+。
- 進入 K980 裝置頁。
- 跑 `LightCheck`。
- 觀察它說「足夠維持充電」還是「光線不足」。

### B. 深度耗電後需要強光喚醒

可能性：中。

官方對 0% 的修復方式不是普通室內光，而是關機後 1,000 lux 以上至少 8 小時。一般桌燈、天花板燈、螢幕光不一定夠。

建議今晚先做一次標準補救：

```text
1. 把鍵盤電源關掉。
2. 清潔頂部吸光條，確認沒有貼紙、灰塵、物品遮住。
3. 放在很亮的室內光源下，目標 > 1,000 lux。
4. 至少 8 小時，不要使用。
5. 不要放戶外直射太陽，避免過熱停止充電。
6. 8 小時後開機，看 LED / Logi Options+ 電量是否恢復。
```

如果可行，可以用手機 lux meter app 粗估。它不一定精準，但足夠判斷「桌面只有 80 lux」還是「檯燈下有 1,000 lux 以上」。

### C. Logi Options+ 電量或通知 bug

可能性：中。

Reddit 上有 K980 使用者回報：實際頁面顯示光線足夠、電量滿，但系統仍每幾分鐘跳 `Insufficient lighting`。這代表 Options+ 對光線 / 電量通知可能有 bug。

但如果鍵盤已經完全沒電、無法使用，問題就不只是通知 bug。

建議：

- 更新 Logi Options+ 到最新版。
- 重啟電腦。
- 重新連接鍵盤。
- 同時用作業系統藍牙電量、Logi Options+ 電量、鍵盤 LED 三者交叉確認。

### D. 電池或充電硬體瑕疵

可能性：中高，尤其如果強光補電後仍無改善。

iFixit 已經有 K980 電池更換指南和零件頁，說明如果鍵盤很快死亡或無法持電，可能需要更換電池。這至少證明這款鍵盤的內部電池是可更換零件，而不是完全報廢式設計。

但 Jones 的鍵盤才使用不到半年，這個時間點不該自己拆。自行拆可能影響保固，而且 K980 的保固 / 支援才是優先路線。

## 5. 對 Jones 的建議行動

### 今晚先做

1. 把鍵盤關機。
2. 清潔頂部太陽能吸光條。
3. 用很亮的室內檯燈或靠窗明亮位置照 8 小時以上，目標 1,000 lux 以上。
4. 不要放戶外直射太陽，避免過熱。
5. 明天開機後用 Logi Options+ 看電量與 LightCheck。

### 如果明天還是 5% / 0% / 無法開機

直接走 Logitech 保固或店家退換。理由：

- 官方宣稱滿電可黑暗使用最長 4 個月。
- 官方宣稱電池壽命最高 10 年。
- 你的使用時間不到半年。
- 已按官方方式強光補電仍無法恢復。
- 這款沒有 USB 備援，一旦無法吸光充電就失去基本功能。

建議不要先拆機換 iFixit 電池。iFixit 是過保或保固拒絕後的路線，不是新機半年內的第一選擇。

### 聯絡客服時可以這樣說

```text
My Logitech Signature Slim Solar+ K980 is less than 6 months old.
Recently it started staying around 5% battery and would not recover even after being exposed to bright indoor light.
It has now fully drained and appears unable to charge.

I understand Logitech recommends switching it off and placing it under bright light over 1,000 lux for at least 8 hours. I have tried / will try that, but if it still does not recover, I would like to request warranty support or replacement because the product has no USB charging fallback and should not fail this early.
```

## 6. 結論

我的判斷：

```text
先不要急著認定電池已死，但這已經超出正常使用體驗。
```

最合理的順序是：

1. 依官方指示做一次「關機 + 1,000 lux 以上 + 8 小時」強光補電。
2. 用 Logi Options+ LightCheck 驗證桌面光線是否真的夠。
3. 如果仍然停在 5% / 0% 或無法開機，直接保固。

如果這把鍵盤在半年內就需要 iFixit 換電池，對一款主打 10 年電池壽命、無需充電的產品來說就是不合理。

## Sources

- Logitech Support: What can I do if I drain the battery of the Signature Slim Solar+ K980 keyboard?  
  https://support.logi.com/hc/en-001/articles/34447065219863-What-can-I-do-if-I-drain-the-battery-of-the-Signature-Slim-Solar-K980-keyboard
- Logitech Support: Getting started - Signature Slim Solar+ K980  
  https://support.logi.com/hc/ja/articles/33318856383127-%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB-Signature-Slim-Solar-K980
- Logitech Support: Specification - Signature Slim Solar+ K980  
  https://support.logi.com/hc/en-us/articles/33318848126999-Specification-Signature-Slim-Solar-K980
- Logitech Business product page: Signature Slim Solar+ K980 for Business  
  https://www.logitech.com/en-gb/products/keyboards/signature-slim-solar-plus-k980-business.html
- Notebookcheck review: Premium keyboard with solar power - Logitech Signature Slim Solar+ K980 review  
  https://www.notebookcheck.net/Premium-keyboard-with-solar-power-Logitech-Signature-Slim-Solar-K980-review.1157858.0.html
- WIRED review: Logitech Signature Solar Slim+ K980 Keyboard Review  
  https://www.wired.com/review/logitech-signature-solar-slim-k980-keyboard/
- Reddit r/logitech: Constant "Insufficient lighting" notification on the Signature Slim Solar+  
  https://www.reddit.com/r/logitech/comments/1s6yczj/constant_insufficient_lighting_notification_on/
- Best Buy customer reviews: Logitech Signature Slim Solar+ K980  
  https://www.bestbuy.com/site/reviews/logitech-signature-slim-solar-k980-for-mac-wireless-bluetooth-solar-and-artificial-light-powered-keyboard-with-customizable-keys-graphite/11230968
- iFixit guide: Logitech Signature Slim Solar+ Battery Replacement  
  https://www.ifixit.com/Guide/Logitech+Signature+Slim+Solar++Battery+Replacement/189668


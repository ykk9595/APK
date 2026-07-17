# 浮水印移除器（Android App）

把之前的 Python 腳本（OpenCV inpainting 去角落浮水印）搬到 Android，做成一個可以直接在手機上使用的 App。

## 功能
- 從相簿選一張圖
- 選浮水印所在角落（右下 / 左下 / 右上 / 左上）
- 拉桿調整偵測區域的寬 / 高比例
- 按「去除浮水印」用 OpenCV 的 inpaint 演算法自動修補
- 處理完可以存到相簿（`Pictures/WatermarkRemover`）或直接分享出去

## 原理（跟 Python 版一樣）
1. 在指定角落切出一小塊區域
2. 用亮閾值（抓白字）+ 暗閾值（抓黑字）找出浮水印筆畫，做出遮罩
3. 膨脹遮罩涵蓋邊緣的半透明像素
4. `Imgproc.inpaint()` 用遮罩周圍的像素把該區域補回去

核心邏輯在 `WatermarkRemover.kt`，是 `remove_watermark.py` 的 Kotlin 版本。

## 產生 APK 安裝到手機

這邊的環境沒有 Android SDK 也連不到 Google/Gradle 的下載伺服器，沒辦法直接編出 APK。有兩個方法：

### 方法一：GitHub Actions 自動編譯（不用裝任何東西）
1. 把這個資料夾整個推到一個新的 GitHub repo
2. GitHub 會自動跑 `.github/workflows/build-apk.yml`（push 到 main/master 就會觸發，或到 Actions 頁籤手動觸發）
3. 跑完後到該次 workflow 的 **Actions → Artifacts**，下載 `WatermarkRemover-debug-apk`
4. 解壓縮拿到 `app-debug.apk`，用 USB 傳到手機或用雲端硬碟分享連結，手機上打開安裝（第一次要在設定裡允許「安裝未知來源 App」）

### 方法二：本機用 Android Studio 編
1. 用 Android Studio 開啟這個專案，等 Gradle 同步完成
2. 選單 `Build → Build App Bundle(s) / APK(s) → Build APK(s)`
3. 編完後點通知裡的 "locate"，會直接跳到 `app/build/outputs/apk/debug/app-debug.apk`
4. 把這個 apk 傳到手機安裝即可

這個 debug APK 沒有簽章成正式版，但可以直接側載安裝，功能完全一樣。若要上架 Google Play 才需要額外做 release 簽章。

## 怎麼打開專案

1. 安裝 **Android Studio**（Koala 版本或更新）
2. `File → Open`，選這個資料夾（`WatermarkRemover/`）
3. 等 Gradle 同步完（第一次會抓 OpenCV 的 aar，檔案較大，需要網路）
4. 接上手機（開啟開發者模式 + USB 偵錯）或用模擬器，按 Run ▶️

## 技術重點
- **OpenCV 直接用 Maven Central 依賴**：`implementation("org.opencv:opencv:4.10.0")`，不用手動下載 SDK（OpenCV 4.9.0 之後才支援這種方式）
- 需要在 `onCreate()` 呼叫 `OpenCVLoader.initLocal()` 載入 native 函式庫
- `minSdk = 24`（Android 7.0+），存檔用 `MediaStore`，在 Android 10+ 不需要存取權限
- 用 ViewBinding，沒有用 Jetpack Compose，UI 是傳統 XML（View-based），比較好照著改

## 可以再加強的方向
- 目前遮罩是靠亮/暗閾值抓文字，如果浮水印顏色跟背景太接近會抓不乾淨，可以改成讓使用者手指直接塗抹要修的區域（畫筆選取），精準度更高
- 加上處理前後對比拖曳滑桿
- 支援批次處理多張圖片
- 加上「復原」功能，保留原圖以便重新調整參數

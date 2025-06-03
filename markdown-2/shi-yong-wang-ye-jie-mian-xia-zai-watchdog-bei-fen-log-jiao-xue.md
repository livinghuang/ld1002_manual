# 使用網頁介面下載 Watchdog 備份 Log 教學

本教學適合不想用命令列下載檔案的用戶，只需透過 Linxdot 的 Web 介面（即瀏覽器進入 Linxdot 網頁）即可完成 Watchdog 備份 log 的下載。

***

### 步驟一、登入 Linxdot 網頁介面

1. 請在瀏覽器輸入 Linxdot 的 IP 位址（預設為 `http://192.168.1.100`，請依照你自己的 IP 調整）。
2. 登入帳號（通常是 `root`），輸入密碼。

<figure><img src="../.gitbook/assets/截圖 2025-06-04 清晨6.07.39.png" alt=""><figcaption></figcaption></figure>

***

### 步驟二、進入「系統」>「備份/燒錄韌體」

1. 網頁左側選單，點選【系統】 > 【備份/燒錄韌體】

<figure><img src="../.gitbook/assets/截圖 2025-06-04 清晨6.08.26.png" alt=""><figcaption></figcaption></figure>

***

### 步驟三、切換到「組態」標籤頁

1. 在頁面上方找到【組態】這個分頁。
2. 點選【組態】。

<figure><img src="../.gitbook/assets/截圖 2025-06-04 清晨6.09.36.png" alt=""><figcaption></figcaption></figure>

***

### 步驟四、指定備份目錄

1. 捲動頁面到底部，你會看到「附加檔案」或類似的輸入框。
2. 在輸入框內填入 `/root/`\
   （這是我們預設備份 log 檔案存放的資料夾）
3. 按下【儲存】。

<figure><img src="../.gitbook/assets/截圖 2025-06-04 清晨6.10.29 (1).png" alt=""><figcaption></figcaption></figure>

***

### 步驟五、下載備份&#x20;

1. 切換到【動作】分頁。
2. 點選「製作壓縮檔」，系統會自動將【組態】 設定的所有目錄下的檔案打包並下載到你的電腦。

<figure><img src="../.gitbook/assets/截圖 2025-06-04 清晨6.14.26.png" alt=""><figcaption></figcaption></figure>

***

### 常見問題與注意事項

* **如果沒有看到 /root/ 目錄或檔案？**\
  請確認備份 log 已經產生，且 backup log script 設定無誤。
* **無法下載檔案或權限錯誤？**\
  請用 root 帳號登入 Linxdot 網頁介面操作。
* **無法切換目錄？**\
  有些舊版 OpenWrt 或客製化韌體可能限制顯示目錄，可考慮先用命令列檢查 `/root/` 內有無目標檔案。

***

### 補充說明

* 如果要改變備份 log 的儲存位置，請同步修改 backup log 腳本的目標資料夾。
* 若有任何問題，請截圖回報錯誤畫面，我們會協助你排除。

***

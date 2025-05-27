---
icon: hand-point-right
---

# Linxdot Watchdog 工具包安裝教學（Windows 版本）

本教學專為**不熟悉 Linux**、**習慣用 Windows** 的用戶設計，指引你如何將 Watchdog 工具包從 Windows 上傳到 Linxdot（OpenWrt）**/opt** 目錄，並在機器上執行腳本。

### 前言

Linxdot 機器（OpenWrt）**無法連外上網**，但你的 Windows 電腦可以上網。你需要將完整的 Watchdog 工具包（資料夾）上傳到 Linxdot，再於 Linxdot 執行設定。

***

### 一、下載工具包

1. 前往 [GitHub 專案](https://github.com/livinghuang/awesome_linxdot)
2. 點選「Code」→「Download ZIP」，下載壓縮檔。
3. 在 Windows 上解壓縮，得到 `awesome_linxdot-main` 資料夾，建議放在 `C:\Users\你的名字\Downloads\`。

***

### 二、將資料夾複製到 Linxdot `/opt` 目錄

#### 直接複製整個資料夾

1.  開啟 PowerShell，進入下載資料夾：

    ```powershell
    cd $HOME\Downloads
    ```
2.  用 SCP 指令遞迴複製整個資料夾到 Linxdot `/opt` 目錄（假設 Linxdot IP 是 192.168.1.100）：

    ```powershell
    scp -r .\awesome_linxdot-main root@192.168.1.100:/opt/
    ```

    * 第一次連線會問你要不要信任，請輸入 `yes`，接著輸入 root 密碼。



***

### 三、登入 Linxdot 並執行 Watchdog 腳本

1.  進入 `/opt/awesome_linxdot-main` 目錄：

    ```sh
    cd /opt/awesome_linxdot-main/awesome-software/backup_log
    ```
2.  賦予腳本執行權限：

    ```sh
    chmod +x backup_log_install.sh
    chmod +x backup_log_uninstall.sh
    ```
3.  執行 Watchdog 腳本：

    ```sh
    ./backup_log_install.sh
    ```
4. 依照畫面提示完成 Watchdog 設定

***

### 常見問題

* **無法 SCP/SSH？**\
  請確認 Windows 已內建 OpenSSH（Win10/11 預設都有），或參考[這裡](https://docs.microsoft.com/zh-tw/windows-server/administration/openssh/openssh_install_firstuse)安裝。
* **/opt 沒有寫入權限？**\
  請用 root 帳號登入操作。
* **unzip 找不到？**\
  輸入 `opkg update && opkg install unzip` 安裝。

***

### 其他說明

* /opt 目錄適合放自訂程式及第三方工具包，不建議放在 /root。
* 如有問題（如權限、找不到檔案），可回報錯誤訊息協助處理。

***

***

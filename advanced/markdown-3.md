---
icon: hand-point-right
---

# Linxdot 安裝LoRaWAN服務

#### **本章將引導您在 Linxdot Hotspot 安裝 LoRaWAN 服務**

如果您 **重新刷機（Reflash）您的 Hotspot**，**LoRaWAN 服務可能不會自動安裝**，因此需要手動重新安裝。請按照以下步驟完成安裝。



#### **安裝與 LoRaWAN 設定指南**

***

### **步驟 1：安裝 ChirpStack LoRaWAN 服務器**

1. **開啟終端機（Terminal）**
2.  **進入 Linxdot 開源工具目錄**：

    ```sh
    cd /etc/linxdot-opensource/
    ```
3.  **執行 ChirpStack 安裝腳本**：

    ```sh
    ./install-chirpstack.sh
    ```
4.  **安裝完成後，確認 Docker 容器是否正在運行**：

    ```sh
    docker ps
    ```

    如果 **ChirpStack 相關容器** 都在運行，表示安裝成功。

***

### **步驟 2：登入 ChirpStack 管理介面**

1.  **開啟網頁瀏覽器**，輸入以下網址：

    ```sh
    http://{LinxdotIP}:8080
    ```

    > **請將 `{LinxdotIP}` 替換為您的 Linxdot HOTSPOT 閘道 IP 地址**
2. **使用預設帳號登入**：
   * **使用者名稱**：`admin`
   * **密碼**：`admin`
3. **成功登入後，新增一個閘道設備**：
   * 進入 **ChirpStack 管理介面**
   * **新增 Gateway**
   * **系統會產生一組 `gateway_id`**
   * **請複製該 `gateway_id`，以便稍後進行 LoRaWAN 設定**

<figure><img src="../.gitbook/assets/截圖 2025-02-12 上午11.56.21.png" alt=""><figcaption></figcaption></figure>

***

### **步驟 3：設定 LoRa Packet Forwarder**

1. **開啟終端機（Terminal）**
2.  **編輯 LoRaWAN 全域設定檔**：

    ```sh
    vi /etc/lora/global_conf.json.sx1250.AS923
    ```
3.  **找到以下行**：

    ```json
    "gateway_ID": "8888880000000000"
    ```
4. **將 `8888880000000000` 替換為剛剛在步驟 2 複製的 `gateway_id`**
5. **儲存並退出 (`ESC -> :wq -> Enter`)**
6.  **執行 LoRa Packet Forwarder 安裝腳本**：

    ```sh
    cd /etc/linxdot-opensource/
    ./install-lora-pkd-fwd.sh
    ```
7. **安裝完成後，回到 ChirpStack 管理介面**，確認 **Gateway 是否已成功連線**。

***

### **最終確認**

#### **成功設定後，您的 Linxdot Hotspot 閘道設備將完全支援 ChirpStack 並提供 LoRaWAN 服務。**

* **確保您的閘道器配置符合網路需求**
* **透過 ChirpStack 管理介面驗證設備連線狀態**

***

### **進階支援**

如有進一步需求，請參閱 **Linxdot 官方文件** 或 **ChirpStack 官方支援頁面**。

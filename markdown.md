---
icon: hand-point-right
---

# 安裝Chirpstack Device Activator 註冊及啟用器

## **在 Linxdot Hotspot 安裝 ChirpStack Device Activator 註冊及啟用器**

本指南將引導您在 **Linxdot Hotspot** 上安裝與設定 **ChirpStack Device Activator**，\
它是一款 **Python 應用程式**，可用於 **管理設備註冊與啟用**，自動化設備的 **LoRaWAN 配置與管理**。

<figure><img src=".gitbook/assets/截圖 2025-02-28 清晨5.49.26.png" alt=""><figcaption></figcaption></figure>

***

### **ChirpStack Device Activator 功能**

* **設備創建**：使用 **CSV 設定檔** 自動化設備創建。
* **設備啟用**：支持 **ABP 與 OTAA** 兩種啟用模式。
* **設備配置更新**：允許啟用設備並設置 **Frame Counter 檢查**。
* **數據驗證**：確保不同協議所需的參數完整性。

***

### **安裝與設定指南**

#### **步驟 1：使用 SSH 登入您的 Hotspot**

請確保您的 **Hotspot** 已連線至網路，然後使用 SSH 連接至設備：

```sh
ssh root@<HOTSPOT_IP>
```

***

#### **步驟 2：下載並安裝 ChirpStack Device Activator**

執行以下指令下載並安裝 **ChirpStack Device Activator**：

```sh
cd /opt
git clone https://github.com/livinghuang/awesome_linxdot.git
cd awesome_linxdot
./install-chirpstack_device_activator.sh
```

***

### **確認 ChirpStack Device Activator 運行狀態**

安裝完成後，請使用以下指令檢查 **ChirpStack Device Activator** 是否正常運行：

```sh
docker ps
```

應該會看到類似以下的輸出：

```
CONTAINER ID   IMAGE                                            COMMAND                  CREATED        STATUS        PORTS                                       NAMES
2ed119806074   livinghuang/chirpstack-device-activator:latest   "python app.py --hos…"   36 hours ago   Up 16 hours   0.0.0.0:5050->5050/tcp, :::5050->5050/tcp   activator
```

您可以修改 **Docker 設定檔** 來調整服務參數：

```sh
vi /opt/awesome_linxdot/chirpstack-software/chirpstack_device_activator/docker-compose.yml
```

***

### **設定 ChirpStack Server**

1.  **開啟 ChirpStack 網頁管理界面**

    ```
    http://<HOTSPOT_IP>:8080
    ```
2.  **建立 Tenant User**

    * 在 **Tenant -> Users** 中，選擇 **"Add Tenant User"**，創建一個 **Gateway Admin 與 Device Admin** 權限的用戶，並記錄 **Tenant User ID**。



    <figure><img src=".gitbook/assets/截圖 2025-02-28 清晨5.32.27.png" alt=""><figcaption></figcaption></figure>
3.  **新增 Device Profile**

    * 進入 **Tenant -> Device Profile**，創建新的 **Device Profile**，並記錄 **Device Profile ID**。



    <figure><img src=".gitbook/assets/截圖 2025-02-28 清晨5.32.19 (1).png" alt=""><figcaption></figcaption></figure>


4.  **新增 Application**

    * 進入 **Tenant -> Applications**，創建新的 Application，並記錄 **Application ID**。



    <figure><img src=".gitbook/assets/截圖 2025-02-28 清晨5.55.53.png" alt=""><figcaption></figcaption></figure>
5.  **創建 API Key**

    * 進入 **Tenant -> API Keys**，點擊 **"Add API Key"**，記錄 API Key（這個 Key 只會顯示一次）。



    <figure><img src=".gitbook/assets/截圖 2025-02-28 清晨5.57.53.png" alt=""><figcaption></figcaption></figure>

***

### **準備 CSV 設備列表**

請建立一個 **CSV 檔案**，內容應符合以下格式：

```
Name,Desc,Protocol,AppID,DevProID,DevEUI,JoinEUI,DevAddr,NwkSKey,AppSKey,SNwkSKey,FNwkSKey,AppKey,NwkKey,SkipFcntChk,IsDisable
20250116142,SQ-7C69C1,ABP104,2bcdb1e7-39ea-492e-9e58-6e2ec9a55773,63224795-5740-4e73-b684-165132eeff74,00007C69C1BDF5F0,11223344556677,7A0A1F70,30303030376336396331626466356630,30303030376336396331626466356630,30303030376336396331626466356630,30303030376336396331626466356630,30303030376336396331626466356630,30303030376336396331626466356630,TRUE,FALSE
20250116143,SQ-6867C1,ABP104,2bcdb1e7-39ea-492e-9e58-6e2ec9a55773,63224795-5740-4e73-b684-165132eeff74,00006867C1BDF5F0,11223344556677,1F577390,30303030363836376331626466356630,30303030363836376331626466356630,30303030363836376331626466356630,30303030363836376331626466356630,30303030363836376331626466356630,30303030363836376331626466356630,TRUE,FALSE
```

請確保 **CSV 的內容符合您的 LoRaWAN 設備配置**。

***

### **使用 ChirpStack Device Activator 進行設備管理**

<figure><img src=".gitbook/assets/截圖 2025-02-28 清晨5.49.26.png" alt=""><figcaption></figcaption></figure>

1.  **開啟 ChirpStack Device Activator 界面**

    ```
    http://<HOTSPOT_IP>:5050
    ```
2. **填寫 ChirpStack Server 設定**
   * **ChirpStack 伺服器地址**：`127.0.0.1` 或目標 ChirpStack 伺服器 IP
   * **API Key**：填入 **API Token**
   * **Tenant ID**：填入 **Tenant ID**
   * **保存設定**
3.  **上傳 CSV 設備清單**

    * 點擊 **"Upload CSV"**
    * 選擇剛剛準備好的 **CSV 檔案** 並上傳
    * 若成功，設備列表將顯示在界面中



    <figure><img src=".gitbook/assets/截圖 2025-02-28 清晨5.59.20.png" alt=""><figcaption></figcaption></figure>
4. **批量啟用設備**
   * 選擇設備，點擊 **"Activate"**，完成設備批量啟用
5. **批量刪除設備**
   * 選擇設備，點擊 **"Delete"**，刪除不需要的設備

***

### **停止 ChirpStack Device Activator**

如需停止 **ChirpStack Device Activator**，請執行以下指令：

```sh
cd /opt/awesome_linxdot
./stop_and_remove_chirpstack_device_activator.sh
```

***

### **進階支援**

如需進一步技術支援，請參閱：

* [ChirpStack 官方支援頁面](https://www.chirpstack.io/)

***

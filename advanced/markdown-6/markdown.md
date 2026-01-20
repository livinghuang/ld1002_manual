---
icon: hand-point-right
---

# Linxdot 安裝Chirpstack Concerntatord 封包集中器

#### 本**章將引導您在 Linxdot Hotspot 安裝** Chirpstack Concerntatord  封包集中器

ChirpStack Concentratord 是一個開源的 LoRaWAN 集中器守護程序，基於 Semtech 的硬體抽象層 (HAL) 所建構。它提供了一個基於 ZeroMQ 的 API，讓一個或多個應用程式可以與閘道硬體互動。 透過將硬體特定的實作與封裝放在單獨的守護程序中，並透過 ZeroMQ API 對外暴露，封包轉發應用程式能完全與閘道硬體解耦。這樣的架構也讓多個應用程式可以同時與閘道硬體互動。例如，多個封包轉發器可將資料分別轉發到不同的 LoRaWAN 網路伺服器。

<figure><img src="../../.gitbook/assets/截圖 2025-02-28 凌晨3.47.12.png" alt=""><figcaption></figcaption></figure>

這意味著：

1. **多重伺服器支援**：一個 LoRaWAN 閘道器可以同時將數據傳輸到多個 ChirpStack 伺服器，而不需要為每個伺服器配置獨立的閘道器。
2. **上行數據選擇性傳輸**：可以將某些伺服器設定為僅接收上行數據（上傳來自 LoRa 裝置的訊息），而不會發送下行數據（如 LoRaWAN 訊息或指令）。
3. **適用於多平台架構**：若一個 LoRaWAN 網絡需要同時將數據發送到不同的應用或服務，這個多工器可以有效地管理流量，確保所有目標伺服器都能夠獲取相同的數據。

這樣的設計可提高數據靈活性，讓使用者能夠更有效地整合不同的 LoRaWAN 平台或數據處理系統。

***

## **在 Linxdot Hotspot 安裝 ChirpStack Concentratord**

本指南將指導您如何在 **Linxdot Hotspot** 上安裝與設定 **ChirpStack Concentratord**。\
**ChirpStack Concentratord** 是專為 **SX1262 LoRa 模組** 設計的軟體，負責與 **LoRaWAN 網絡伺服器** 進行通訊。

***

### **安裝與設定指南**

#### **步驟 1：使用 SSH 登入您的 Hotspot**

請確保您的 **Hotspot** 已連線至網路，然後使用 SSH 連接至設備：

```sh
ssh root@<HOTSPOT_IP>
```

***

#### **步驟 2：下載並安裝 ChirpStack Concentratord**

執行以下指令下載並安裝 **ChirpStack Concentratord**：

```sh
cd /opt
git clone https://github.com/livinghuang/awesome_linxdot.git
cd awesome_linxdot
./install-chirpstack-concentratord.sh
```

***

### **確認 ChirpStack Concentratord 運行狀態**

安裝完成後，您可以使用以下指令檢查 **ChirpStack Concentratord** 是否運行：

```sh
service list
```

如果安裝成功，您應該會看到以下類似輸出：

```sh
/etc/init.d/linxdot-chirpstack-concentratord enabled running
```

您也可以使用以下指令查看運行中的 **LoRa 封包數據**：

```sh
logread -f | grep concentratord
```

當 LoRaWAN 封包被接收時，應該會看到類似以下的日誌輸出：

```
Thu Feb 27 19:57:43 2025 user.notice chirpstack-concentratord: 2025-02-27T19:57:43.147Z INFO  [chirpstack_concentratord_sx1302::handler::uplink] Frame received, uplink_id: 1073536733, count_us: 3844463833, freq: 923600000, bw: 125000, mod: LoRa, dr: SF12, ftime_received: false, ftime_ns: 0
Thu Feb 27 19:57:43 2025 user.notice chirpstack-mqtt-forwarder: 2025-02-27T19:57:43.149Z INFO  [chirpstack_mqtt_forwarder::backend::concentratord] Received uplink frame, uplink_id: 1073536733
```

***

### **取得 Gateway ID**

ChirpStack Concentratord 運行於 **Linxdot Hotspot** 內的 **SX1262 LoRa 模組**，要在 **LoRaWAN 網絡伺服器**（如 **ChirpStack** 或 **The Things Network (TTN)**）中註冊此設備，您需要取得 **Gateway ID**。

<figure><img src="../../.gitbook/assets/截圖 2025-02-28 凌晨3.46.36 (1).png" alt=""><figcaption></figcaption></figure>

#### **步驟 ：透過 Web 界面取得 Gateway ID**

1. **打開瀏覽器**
2.  **輸入 Linxdot 的 IP 位址**

    ```
    http://<LINXDOT_IP>:80
    ```
3. **在導航欄中找到**
   * **儀表板** → **狀態** → **Awesome Linxdot**
4. **記錄 Gateway ID**
   * Gateway ID 將在此處顯示，請將其記錄下來，稍後註冊至 LoRaWAN 網絡伺服器（如 **ChirpStack** 或 **TTN**）。

<figure><img src="../../.gitbook/assets/截圖 2025-02-28 凌晨3.46.43.png" alt=""><figcaption></figcaption></figure>

***

### **設定 ChirpStack Concentratord**

您可以修改設定檔來調整 **ChirpStack Concentratord** 的運行參數：

#### **步驟 1：停止 ChirpStack Concentratord**

```sh
cd /opt/awesome_linxdot
./stop_and_remove_chirpstack_concentratord.sh
```

#### **步驟 2：編輯設定檔（.toml）**

```sh
vi chirpstack-software/chirpstack-concentratord-binary/config/<設定檔>.toml
```

**在 `vi` 編輯器中：**

* 按 **`i`** 進入編輯模式
* 修改完成後，按 **`Esc`** 退出編輯模式
* 輸入 **`:wq`** 保存並退出

#### **步驟 3：重新安裝 ChirpStack Concentratord**

```sh
./install-chirpstack-concentratord.sh
```

***

### **停止 ChirpStack Concentratord 服務**

如需停止 **ChirpStack Concentratord**，請執行以下指令：

```sh
cd /opt/awesome_linxdot
./stop_and_remove_chirpstack_concentratord.sh
```

***

### **進階支援**

如需進一步技術支援，請參閱：

* [ChirpStack 官方支援頁面](https://www.chirpstack.io/)

***

這樣的修訂確保內容更加完整，讓使用者可以輕鬆理解並執行安裝、設定與管理。如有其他需求請告知！

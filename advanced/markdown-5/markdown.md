---
icon: hand-point-right
---

# Linxdot 安裝Chirpstack Concerntatord 封包集中器

#### **章將引導您在 Linxdot Hotspot 安裝** Chirpstack Concerntatord  封包集中器

ChirpStack Concentratord 是一個開源的 LoRaWAN 集中器守護程序，基於 Semtech 的硬體抽象層 (HAL) 所建構。它提供了一個基於 ZeroMQ 的 API，讓一個或多個應用程式可以與閘道硬體互動。 透過將硬體特定的實作與封裝放在單獨的守護程序中，並透過 ZeroMQ API 對外暴露，封包轉發應用程式能完全與閘道硬體解耦。這樣的架構也讓多個應用程式可以同時與閘道硬體互動。例如，多個封包轉發器可將資料分別轉發到不同的 LoRaWAN 網路伺服器。

<figure><img src="../../.gitbook/assets/截圖 2025-02-28 凌晨3.47.12.png" alt=""><figcaption></figcaption></figure>

這意味著：

1. **多重伺服器支援**：一個 LoRaWAN 閘道器可以同時將數據傳輸到多個 ChirpStack 伺服器，而不需要為每個伺服器配置獨立的閘道器。
2. **上行數據選擇性傳輸**：可以將某些伺服器設定為僅接收上行數據（上傳來自 LoRa 裝置的訊息），而不會發送下行數據（如 LoRaWAN 訊息或指令）。
3. **適用於多平台架構**：若一個 LoRaWAN 網絡需要同時將數據發送到不同的應用或服務，這個多工器可以有效地管理流量，確保所有目標伺服器都能夠獲取相同的數據。

這樣的設計可提高數據靈活性，讓使用者能夠更有效地整合不同的 LoRaWAN 平台或數據處理系統。

這是修訂後的 **安裝與設定指南**，移除了 `nano`，改為 **`vi`** 來編輯設定檔：

***

## **安裝與設定指南**

本指南將引導您在 **Linxdot** 上安裝與設定 **ChirpStack Concentratord（封包集中器）**。\
請按照以下步驟完成安裝與配置。

***

### **步驟 1：使用 SSH 登入您的 Hotspot**

請確保您的 **Hotspot** 已連線至網路，然後使用 SSH 連接至設備：

```sh
ssh root@<HOTSPOT_IP>
```

***

### **步驟 2：下載並安裝 ChirpStack Concentratord**

執行以下指令下載並安裝 **ChirpStack Concentratord**：

```sh
cd /opt
git clone https://github.com/livinghuang/awesome_linxdot.git
cd awesome_linxdot
./install-chirpstack-concentratord.sh
```

***

After install chirpstack-concentratord , you could use below command to check it&#x20;

```
service list
```

**Next Step : Get the gateway-id, Chirpstack concentratord is a program to operate the Lora  module SX1262 which inside of Linxdot machine. You will need to get the uni gateway id to setup it in the lorawan network server.**&#x20;

**step1 : open your browser**

**access : lindox.ip:80**&#x20;

**in the navigation column , you could find the "儀表板"->"狀態"-> "awesome linxdot"**&#x20;

&#x20;**you could find the gateway id . save it it will use on later you register it into your chirpstacke or the other gate loarawan network server like TTN.**

**you should found something like this**&#x20;

/etc/init.d/linxdot-chirpstack-concentratord enabled running

you could check the status&#x20;

logread -f | grep concentratord

you could found the some packet frame like this

Thu Feb 27 19:57:43 2025 user.notice chirpstack-concentratord: 2025-02-27T19:57:43.147Z INFO  \[chirpstack\_concentratord\_sx1302::handler::uplink] Frame received, uplink\_id: 1073536733, count\_us: 3844463833, freq: 923600000, bw: 125000, mod: LoRa, dr: SF12, ftime\_received: false, ftime\_ns: 0

Thu Feb 27 19:57:43 2025 user.notice chirpstack-mqtt-forwarder: 2025-02-27T19:57:43.149Z INFO  \[chirpstack\_mqtt\_forwarder::backend::concentratord] Received uplink frame, uplink\_id: 1073536733

Thu Feb 27 19:57:44 2025 user.notice chirpstack-concentratord: 2025-02-27T19:57:44.944Z INFO  \[chirpstack\_concentratord\_sx1302::handler::uplink] Frame received, uplink\_id: 3481726670, count\_us: 3846260724, freq: 924600000, bw: 125000, mod: LoRa, dr: SF12, ftime\_received: false, ftime\_ns: 0



### **設定 ChirpStack Concentratord**

您可以在 **`chirpstack-software/chirpstack-concentratord-binary/config`** 修改 **ChirpStack Concentratord** 的設定。

#### **修改設定檔（.toml）**

如需變更設定，請按照以下步驟進行：

1.  **停止 ChirpStack Concentratord 服務**

    ```sh
    ./stop_and_remove_chirpstack_concentratord.sh
    ```
2.  **使用 `vi` 編輯 `.toml` 設定檔**

    ```sh
    vi chirpstack-software/chirpstack-concentratord-binary/config/<設定檔>.toml
    ```

    **在 `vi` 編輯器中：**

    * 按 **`i`** 進入編輯模式
    * 修改完成後，按 **`Esc`** 退出編輯模式
    * 輸入 **`:wq`** 保存並退出
3.  **重新安裝 ChirpStack Concentratord**

    ```sh
    ./install-chirpstack-concentratord.sh
    ```

***



**停止 ChirpStack Concentratord 服務**

如果需要停止 **ChirpStack Concentratord**，請執行以下指令：

```sh
cd /opt/awesome_linxdot
./stop_and_remove_chirpstack_concentratord.sh
```

***

### **進階支援**

如需進一步技術支援，請參閱：

* [ChirpStack 官方支援頁面](https://www.chirpstack.io/)

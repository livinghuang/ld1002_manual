---
icon: hand-point-right
---

# Linxdot 安裝MQTT封包轉發器

這是修訂後的 **安裝與設定指南**，使內容更清晰、結構更完善：

***

## **在 Linxdot Hotspot 安裝 MQTT 轉發多工器**

本章將指導您如何在 **Linxdot Hotspot** 上安裝 **ChirpStack MQTT Forwarder**，它是一個 **MQTT 封包轉發器**，用於 LoRa 閘道器（Gateway）。\
與 **ChirpStack Gateway Bridge** 不同，**ChirpStack MQTT Forwarder** **必須安裝在 LoRa 閘道器上**。

<figure><img src="../../.gitbook/assets/截圖 2025-02-27 下午5.58.17.png" alt=""><figcaption></figcaption></figure>

#### **支援後端（Backends）**

* **ChirpStack Concentratord**
* **Semtech UDP Packet Forwarder**

***

### **安裝與設定指南**

#### **步驟 1：使用 SSH 登入您的 Hotspot**

請確保您的 **Hotspot** 已連線至網路，然後使用 SSH 連接至設備：

```sh
ssh root@<HOTSPOT_IP>
```

***

#### **步驟 2：下載並安裝 ChirpStack MQTT Forwarder**

執行以下指令下載並安裝 **ChirpStack MQTT Forwarder**：

```sh
cd /opt
git clone https://github.com/livinghuang/awesome_linxdot.git
cd awesome_linxdot
./install-chirpstack-mqtt-forwarder.sh [role]
```

**注意：`[role]` 可選參數**

*   **一般模式（Normal Mode）**

    ```sh
    ./install-chirpstack-mqtt-forwarder.sh
    ```
*   **邊界模式（Border Mode）**

    ```sh
    ./install-chirpstack-mqtt-forwarder.sh border
    ```

***

### **設定 ChirpStack MQTT Forwarder**

您可以修改設定檔來調整 **MQTT Forwarder** 的運行參數：

```sh
vi chirpstack-software/chirpstack-mqtt-forwarder-binary/<設定檔>.toml
```

**在 `vi` 編輯器中：**

* 按 **`i`** 進入編輯模式
* 修改完成後，按 **`Esc`** 退出編輯模式
* 輸入 **`:wq`** 保存並退出

***

### **停止 ChirpStack MQTT Forwarder 服務**

如果需要停止 **ChirpStack MQTT Forwarder**，請執行以下指令：

```sh
cd /opt/awesome_linxdot
./stop_and_remove_chirpstack_mqtt_forwarder.sh
```

***

### **進階支援**

如需進一步技術支援，請參閱：

* [ChirpStack 官方支援頁面](https://www.chirpstack.io/)


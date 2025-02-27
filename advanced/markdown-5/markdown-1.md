---
icon: hand-point-right
---

# Linxdot 安裝 Chirpstack Gateway Mesh 並設定角色為 Relay（中繼閘道器）

#### **本章將引導您在 Linxdot Hotspot** 安裝與設定為 Chirpstack Gateway Mesh Relay

#### **ChirpStack Gateway Mesh 簡介**

**ChirpStack Gateway Mesh** 是一款 **軟體組件**，可以運行在 **LoRa® 閘道器** 上，將這些閘道器轉換為 **兩種角色**：

1. **Relay Gateway（中繼閘道器）**：
   * 主要負責 **轉發 LoRaWAN 資料**（上行與下行）。
   * 適用於無法直接連接網際網路的閘道器，通常為 **太陽能供電** 的遠端設備。
2. **Border Gateway（邊界閘道器）**：
   * 負責 **終止 Mesh 協議**，並與 **ChirpStack 伺服器** 直接通訊。
   * 需要 **網際網路連接**，以處理來自 Relay Gateway 的 LoRaWAN 訊息。

<figure><img src="../../.gitbook/assets/截圖 2025-02-15 清晨7.29.51.png" alt=""><figcaption><p><strong>ChirpStack Gateway Mesh 架構圖</strong></p></figcaption></figure>

***

#### **ChirpStack Gateway Mesh 的目的**

該組件的主要目標是 **擴展 LoRaWAN® 覆蓋範圍**，尤其適用於 **遠端地區** 或 **網際網路覆蓋不足** 的環境。

* 在無法連接網際網路的地區，**Relay Gateway** 可以將 LoRaWAN 訊息轉發給 **Border Gateway**。
* 這種 **中繼機制** 確保了即使裝置處於 **無網路環境**，也能透過 **LoRa 進行資料轉發**，直到訊息傳遞至 Border Gateway 並最終送達 **ChirpStack 伺服器**。
* **與 LoRa 聯盟（LoRa Alliance） 的 Relay Protocol 不同**，此解決方案 **不需要修改裝置端的軟體**，因此更容易部署。

***

#### **技術限制**

1. **支援最多 8 個 Hops（跳點）**
   * 第一個 Relay Gateway 被視為 **第 1 跳（Hop）**。
   * 之後最多可有 **7 個額外的 Relay Gateway**。
2. **受限於 LoRa 空中時間（Airtime）**
   * 由於 Relay Gateway 需要接收並重新傳輸封包，因此空中時間（Airtime） 會影響中繼效果。
   * 如果 Relay 過多，可能會導致網路擁塞，影響效能。

***

#### **Relay Gateway（中繼閘道器）簡介**

Relay Gateway 是一種無需網際網路連接的 LoRaWAN 閘道器，常見硬體為 SX1301/2/3 系列（可選配 ISM2400 集中器模組）。\
該設備可由太陽能供電，適合部署於偏遠或無電力基礎設施的環境中。

***

#### **Relay Gateway 的功能**

**上行（Uplink）封包處理**

* 接收來自 LoRaWAN 裝置的上行資料。
* 將上行訊框封裝成 Mesh 格式。
* 透過 Mesh 網路將封裝後的資料轉發至其他 Relay Gateway 或 Border Gateway。

**下行（Downlink）封包處理**

* 從 Border Gateway 或其他 Relay Gateway 接收 Mesh 格式下行訊框。
* 解封 Mesh 封裝，取得原始 LoRaWAN 下行資料。
* 將資料傳送至對應的 LoRaWAN 裝置。

***

#### **Relay Gateway 的特點**

* 無需網際網路連線，適用於遠端或無網路基礎設施的場景。
* 可由太陽能供電，適合偏遠地區長期運行。
* 透過 ChirpStack Gateway Mesh 管理上行與下行資料的中繼傳輸。
* 支援多跳傳輸（Multi-Hop），可在多個 Relay Gateway 間傳遞資料直至 Border Gateway。
* 與 Border Gateway 的通訊可使用與終端裝置相同的無線頻段，或根據硬體能力與應用需求使用 ISM2400 頻段。

***

#### **Relay Gateway 的組件需求**

在 Relay Gateway 上需安裝以下組件：

* ChirpStack Concentratord
* ChirpStack Gateway Mesh

***

Relay Gateway 作為 Mesh 網路中的中繼節點，能有效擴展 LoRaWAN 網路覆蓋範圍，實現無網路環境下的穩定通訊。

這是修訂後的 **安裝與設定指南**，讓內容更清晰、結構更完善：

***

這是修訂後的 **安裝與設定指南**，確保內容更清晰、易讀，並提供更完善的步驟說明：

***

## **在 Linxdot Hotspot 安裝 ChirpStack Gateway Mesh**

本指南將指導您如何在 **Linxdot Hotspot** 上安裝與設定 **ChirpStack Gateway Mesh**，\
**ChirpStack Gateway Mesh** 用於 **LoRa 中繼閘道器（Relay Gateway）**，支援 LoRaWAN 封包轉發。

***

### **安裝與設定指南**

#### **步驟 1：使用 SSH 登入您的 Hotspot**

請確保您的 **Hotspot** 已連線至網路，然後使用 SSH 連接至設備：

```sh
ssh root@<HOTSPOT_IP>
```

***

#### **步驟 2：下載並安裝 ChirpStack Gateway Mesh**

執行以下指令下載並安裝 **ChirpStack Gateway Mesh**：

```sh
cd /opt
git clone https://github.com/livinghuang/awesome_linxdot.git
cd awesome_linxdot
./install-chirpstack-gateway-mesh.sh relay as923
```

**注意：** `as923` 代表使用 **AS923 頻段**，如果您的網路使用其他頻段，請根據需求替換，例如：

*   **EU868 頻段**

    ```sh
    ./install-chirpstack-gateway-mesh.sh relay eu868
    ```
*   **US915 頻段**

    ```sh
    ./install-chirpstack-gateway-mesh.sh relay us915
    ```

***

### **確認 ChirpStack Gateway Mesh 運行狀態**

安裝完成後，您可以使用以下指令檢查 **ChirpStack Gateway Mesh** 是否運行：

```sh
service list
```

如果安裝成功，您應該會看到類似的輸出：

```sh
/etc/init.d/linxdot-chirpstack-gateway-mesh enabled running
```

您也可以使用以下指令查看運行中的 **LoRa 封包數據**：

```sh
logread -f | grep mesh
```

當 **LoRaWAN 封包被接收時**，應該會看到類似以下的日誌輸出：

```
Thu Feb 27 20:18:10 2025 user.notice chirpstack-gateway-mesh: 2025-02-27T20:18:10.622Z INFO  [chirpstack_gateway_mesh::backend] Frame received - [uplink_id: 2494328473, freq: 924200000, rssi: -36, snr: 12.5, mod: [LORA - sf: 12, bw: 125000]]
Thu Feb 27 20:18:10 2025 user.notice chirpstack-gateway-mesh: 2025-02-27T20:18:10.622Z INFO  [chirpstack_gateway_mesh::mesh] Proxying LoRaWAN uplink, uplink: [uplink_id: 2494328473, freq: 924200000, rssi: -36, snr: 12.5, mod: [LORA - sf: 12, bw: 125000]]
Thu Feb 27 20:18:10 2025 user.notice chirpstack-gateway-mesh: 2025-02-27T20:18:10.622Z INFO  [chirpstack_gateway_mesh::proxy] Sending uplink event - [uplink_id: 2494328473, freq: 924200000, rssi: -36, snr: 12.5, mod: [LORA - sf: 12, bw: 125000]]
```

***

### **設定 ChirpStack Gateway Mesh**

您可以修改設定檔來調整 **ChirpStack Gateway Mesh** 的運行參數：

```sh
vi chirpstack-software/chirpstack-gateway-mesh-binary/config/<設定檔>.toml
```

**在 `vi` 編輯器中：**

* 按 **`i`** 進入編輯模式
* 修改完成後，按 **`Esc`** 退出編輯模式
* 輸入 **`:wq`** 保存並退出

***

### **重新安裝 ChirpStack Gateway Mesh**

如果需要切換至不同頻段（例如 **EU868**），請執行：

```sh
./install-chirpstack-gateway-mesh.sh relay eu868
```

***

### **停止 ChirpStack Gateway Mesh 服務**

如需停止 **ChirpStack Gateway Mesh**，請執行以下指令：

```sh
cd /opt/awesome_linxdot
./stop_and_remove_chirpstack_gateway_mesh.sh
```

***

### **進階支援**

如需進一步技術支援，請參閱：

* [ChirpStack 官方支援頁面](https://www.chirpstack.io/)

***

***


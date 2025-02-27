---
icon: hand-point-right
---

# Linxdot 安裝 Chirpstack Gateway Mesh 並設定角色為 Border (邊界閘道器)

**本章將引導您在 Linxdot Hotspot** 安裝與設定為 Chirpstack Gateway Mesh Border

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

#### **Border Gateway（邊界閘道器）簡介**

Border Gateway 是 **具備網際網路連接功能** 的 LoRaWAN 閘道器，可處理 Mesh 封裝的 LoRaWAN 負載資料。

***

#### **Border Gateway 的功能**

**上行（Uplink）封包處理**

* 接收其他 Relay Gateway 傳來的 Mesh 格式 LoRaWAN 上行訊框。
* 解封（Unwrap）Mesh 格式後，將 LoRaWAN 上行資料傳送至 ChirpStack。
* 在上行傳輸中，設置包含 Mesh 特定資訊的上下文 (Context)，以便下行傳輸時使用。

**下行（Downlink）封包處理**

* 從 ChirpStack 接收下行資料（需包含先前上行設置的上下文資訊）。
* 將下行資料封裝（Encapsulate）成 Mesh 格式。
* 傳送 Mesh 格式下行訊框給對應的 Relay Gateway 或終端裝置。

***

#### **Border Gateway 的特點**

* **可直接與 ChirpStack 伺服器溝通**，實現 LoRaWAN 裝置與伺服器之間的資料傳輸。
* **處理 Mesh 封裝與解封**，確保 Mesh 網路中的資料可正確傳遞至 ChirpStack。
* **保留上下文資訊**，確保上行與下行傳輸過程中，Mesh 特定資料能正確匹配。

這是修訂後的 **安裝與設定指南**，讓內容更清晰、結構更完善：

***

## **安裝與設定指南**

本指南將引導您在 **Linxdot** 上安裝與設定 **ChirpStack Gateway Mesh（封包集中器）**，用於 Borde&#x72;**（邊界閘道器）**。\
請按照以下步驟完成安裝與配置。

***

### **步驟 1：使用 SSH 登入您的 Hotspot**

請確保您的 **Hotspot** 已連線至網路，然後使用 SSH 連接至設備：

```sh
ssh root@<HOTSPOT_IP>
```

***

### **步驟 2：下載並安裝 ChirpStack Gateway Mesh**

執行以下指令下載並安裝 **ChirpStack Gateway Mesh**：

```sh
cd /opt
git clone https://github.com/livinghuang/awesome_linxdot.git
cd awesome_linxdot
./install-chirpstack-gateway-mesh.sh border as923
```

**注意：** `as923` 代表使用 **AS923 頻段**，如果您的網路使用其他頻段，請根據需求替換。

***

### **設定 ChirpStack Gateway Mesh**

您可以修改設定檔來調整 Gateway Mesh 的運行參數：

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
./install-chirpstack-gateway-mesh.sh border eu868
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


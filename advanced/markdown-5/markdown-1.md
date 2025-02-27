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

#### 安裝與設定指南

***

**步驟 1：停止 Docker，關閉 ChirpStack LoRaWAN 服務**

1. 使用 SSH 登入您的 Hotspot。
2.  輸入以下指令以停用並停止 Docker 服務：

    ```bash
    /etc/init.d/dockerd disable
    /etc/init.d/dockerd stop
    ```

***

**步驟 2：設定 ChirpStack Gateway Mesh**

1.  切換至設定目錄：

    ```bash
    cd /etc/linxdot-opensource/chirpstack-border-gateway/chirpstack-gateway-mesh-binary/config
    ```
2.  刪除舊設定檔並建立新檔案：

    ```bash
    rm chirpstack-gateway-mesh.toml
    vi chirpstack-gateway-mesh.toml
    ```
3. 在 `vi` 編輯器中，按下 `i` 進入插入模式，將以下內容貼上並根據需求修改。

***

**chirpstack-gateway-mesh.toml 配置範本**

```toml
[logging]
level="INFO"            
log_to_syslog=false     

[mesh]
signing_key="a08ed9e0cca290514071818786f9f9dd"  
border_gateway=false                           
max_hop_count=8                                
border_gateway_ignore_direct_uplinks=false     

frequencies=[                                  
  923200000,
  923400000,
  923600000,
  923800000,
  924000000,
  924200000,
  924400000,
  924600000,
]

tx_power=16                                    

[mesh.data_rate]
modulation="LORA"                              
spreading_factor=7                             
bandwidth=125000                               
code_rate="4/5"                                

[mesh.proxy_api]
event_bind="ipc:///tmp/gateway_relay_event"
command_bind="ipc:///tmp/gateway_relay_command"

[backend]
  [backend.concentratord]
  event_url="ipc:///tmp/concentratord_event"
  command_url="ipc:///tmp/concentratord_command"

  [backend.mesh_concentratord]
  event_url="ipc:///tmp/concentratord_event"
  command_url="ipc:///tmp/concentratord_command"
```

4. 儲存並退出
   * 按 `ESC`，輸入 `:wq` 後按 `Enter` 儲存檔案並退出。

***

**步驟 3：啟動服務**

1.  回到專案根目錄並執行安裝腳本：

    ```bash
    cd /etc/linxdot-opensource
    ./install-chirpstack-border-gateway-concentratord.sh
    ./install-chirpstack-border-gateway-mesh.sh
    ```
2.  檢查執行狀態\
    查看 Concentratord 執行日誌以確保服務正常運作：

    ```bash
    logread -f | grep concentratord
    ```

***

安裝與設定完成後，請確認 Gateway 已成功連線並可正常轉發 LoRaWAN 封包。\
請務必修改 `signing_key` 為專屬安全金鑰以確保資料傳輸安全。

***


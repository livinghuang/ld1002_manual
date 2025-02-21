---
icon: hand-point-right
hidden: true
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

<figure><img src="../.gitbook/assets/截圖 2025-02-15 清晨7.29.51.png" alt=""><figcaption><p><strong>ChirpStack Gateway Mesh 架構圖</strong></p></figcaption></figure>

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

#### **安裝與設定指南**

***

### **步驟 1：安裝 ChirpStack LoRaWAN 服務器**

1. 使用 **SSH** 登入您的 **Hotspot**。

<figure><img src="../.gitbook/assets/截圖 2025-02-12 上午8.35.21.png" alt=""><figcaption></figcaption></figure>

2.  現在進入資料夾路徑：

    {% code overflow="wrap" %}
    ```sh
    # 進入資料夾路徑
    cd /mnt/opensource-system/chirpstack-docker
    # 刪除預設的 docker-compose.yml 文件
    rm docker-compose.yml
    # Create the new docker-compose.yml
    vi docker-compose.yml
    ```
    {% endcode %}

以下是新的 `docker-compose.yml` 範例，您可以直接複製並使用 **vi 編輯器** 貼上到 **Hotspot** 的 `docker-compose.yml`，然後儲存。

#### 步驟：

1.  使用 **vi 編輯器** 開啟 `docker-compose.yml`：

    ```sh
    vi docker-compose.yml
    ```
2.  進入 **插入模式**（按下 `i`），然後貼上以下內容：

    ```yaml
    version: "3"

    services:
      chirpstack:
        image: chirpstack/chirpstack:4
        command: -c /etc/chirpstack
        restart: unless-stopped
        volumes:
          - ./configuration/chirpstack:/etc/chirpstack
          - ./lorawan-devices:/opt/lorawan-devices
        depends_on:
          - postgres
          - mosquitto
          - redis
        environment:
          - MQTT_BROKER_HOST=mosquitto
          - REDIS_HOST=redis
          - POSTGRESQL_HOST=postgres
        ports:
          - 8080:8080
        logging:
          driver: "json-file"
          options:
            max-size: "10m"
            max-file: "3"

      chirpstack-rest-api:
        image: chirpstack/chirpstack-rest-api:4
        restart: unless-stopped
        command: --server chirpstack:8080 --bind 0.0.0.0:8090 --insecure
        ports:
          - 8090:8090
        depends_on:
          - chirpstack
        logging:
          driver: "json-file"
          options:
            max-size: "10m"
            max-file: "3"

      postgres:
        image: postgres:14-alpine
        restart: unless-stopped
        volumes:
          - ./configuration/postgresql/initdb:/docker-entrypoint-initdb.d
          - postgresqldata:/var/lib/postgresql/data
        environment:
          - POSTGRES_PASSWORD=root
        logging:
          driver: "json-file"
          options:
            max-size: "10m"
            max-file: "3"

      redis:
        image: redis:7-alpine
        restart: unless-stopped
        command: redis-server --save 300 1 --save 60 100 --appendonly no --ignore-warnings ARM64-COW-BUG
        volumes:
          - redisdata:/data
        logging:
          driver: "json-file"
          options:
            max-size: "10m"
            max-file: "3"

      mosquitto:
        image: eclipse-mosquitto:2
        restart: unless-stopped
        ports:
          - 1883:1883
        volumes: 
          - ./configuration/mosquitto/config/:/mosquitto/config/
        logging:
          driver: "json-file"
          options:
            max-size: "10m"
            max-file: "3"

    volumes:
      postgresqldata:
      redisdata:
    ```
3.  設定取得 gateway id：

    {% code overflow="wrap" %}
    ```sh
    cd /etc/linxdot-opensource/chirpstack-border-gateway/chirpstack-concentratord-binary 
    ./chirpstack-concentratord-sx1302 -c config/concentratord.toml -c config/channels_as923.toml | grep gateway_id
    #看到gateway_id的數值後，就可以先中斷
    #在執行log中找到gateway_id, 記錄下來
    ```
    {% endcode %}
4.  設定 chirpstack-gateway-mesh-binary：

    ```sh
    cd /etc/linxdot-opensource/chirpstack-border-gateway/chirpstack-gateway-mesh-binary/config
    rm chirpstack-gateway-mesh.toml
    vi chirpstack-gateway-mesh.toml
    ```
5.  進入 **插入模式**（按下 `i`），然後貼上以下內容：

    ```toml
    # Logging settings.
    [logging]

      # Log level.
      #
      # Valid options are:
      #   * TRACE
      #   * DEBUG
      #   * INFO
      #   * WARN
      #   * ERROR
      #   * OFF
      level="INFO"

      # Log to syslog.
      #
      # When set to true, log messages are being written to syslog instead of stdout.
      log_to_syslog=false


    # Mesh configuration.
    [mesh]

      # Signing key (AES128, HEX encoded).                                           
      #                                                                          
      # This key is used to sign and validate each mesh packet. This key must be
      # configured on every Border / Relay gateway equally.                      
      signing_key="a08ed9e0cca290514071818786f9f9dd" # please change it 

      # Border Gateway.
      #
      # If this is set to true, then the ChirpStack Gateway Mesh will consider
      # this gateway as a Border Gateway, meaning that it will unwrap relayed
      # uplinks and forward these to the proxy API, rather than relaying these.
      border_gateway=false

      # Max hop count.
      #
      # This defines the maximum number of hops a relayed payload will pass.
      max_hop_count=8 # max=8

      # Ignore direct uplinks (Border Gateway).
      #
      # If this is set to true, then direct uplinks (uplinks that are not relay
      # encapsulated) will be silently ignored. This option is especially useful
      # for testing, in which case you want to set this to true for the Border
      # Gateway.
      border_gateway_ignore_direct_uplinks=false

      # Mesh frequencies.
      #
      # The ChirpStack Gateway Mesh will randomly use one of the configured
      # frequencies when relaying uplink and downlink messages.
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


      # TX Power (EIRP).
      #
      # The TX Power in EIRP used when relaying uplink and downlink messages.
      tx_power=16

      # Data-rate properties.
      #
      # The data-rate properties when relaying uplink and downlink messages.
      [mesh.data_rate]
      
        # Modulation.
        #
        # Valid options are: LORA, FSK
        modulation="LORA"

        # Spreading-factor (LoRa).
        spreading_factor=7

        # Bandwidth (LoRa).
        bandwidth=125000

        # Code-rate (LoRa).
        code_rate="4/5"

        # Bitrate (FSK).
        bitrate=0


      # Proxy API configuration.
      #
      # If the gateway is configured to operate as Border Gateway. It
      # will unwrap relayed uplink frames, and will wrap downlink payloads that
      # must be relayed. In this case the ChirpStack MQTT Forwarder must be
      # configured to use the proxy API instead of the Concentratord API.
      #
      # Payloads of devices that are under the direct coverage of this gateway
      # are transparently proxied between the ChirpStack MQTT Forwarder and
      # ChirpStack Concentratord.
      #
      # This configuration is only used when the border_gateway option is set
      # to true.
      [mesh.proxy_api]

        # Event PUB socket bind.
        event_bind="ipc:///tmp/gateway_relay_event"

        # Command REP socket bind.
        command_bind="ipc:///tmp/gateway_relay_command"


    # Backend configuration.
    [backend]

      # ChirpStack Concentratord configuration (end-device communication).
      [backend.concentratord]

        # Event API URL.
        event_url="ipc:///tmp/concentratord_event"

        # Command API URL.
        command_url="ipc:///tmp/concentratord_command"


      # ChirpStack Concentratord configuration (mesh communication).
      #
      # While not required, this configuration makes it possible to use a different
      # Concentratord instance for the mesh communication. E.g. this
      # makes it possible to use ISM2400 for mesh communication and EU868 for
      # communication with the end-devices.
      [backend.mesh_concentratord]

        # Event API URL.
        event_url="ipc:///tmp/concentratord_event"

        # Command API URL.
        command_url="ipc:///tmp/concentratord_command"
    ```
6. **儲存並退出**（按 `ESC`，輸入 `:wq`，然後按 `Enter`）。
7.  啟動服務：

    ```sh
     cd /etc/linxdot-opensource
     ./install-chirpstack-border-gateway-concentratord.sh 
     ./install-chirpstack-border-gateway-mesh.sh
    ```
8. 在LoRaWAN Server 設定 Gateway

<figure><img src="../.gitbook/assets/截圖 2025-02-21 清晨7.03.17.png" alt=""><figcaption></figcaption></figure>



#### **Linxdot-ChirpStack Border Gateway Concentratord 異常處理指南**

若 **`linxdot-chirpstack-border-gateway-concentratord`** 運作不正常，請按照以下步驟處理：

***

**Step 1: 暫時停止服務**

先關閉正在運行的服務：

```bash
/etc/init.d/linxdot-chirpstack-border-gateway-concentratord stop
```

***

**Step 2: 移除背景執行程序**

1.  查找背景執行的 concentratord 程式：

    ```bash
    ps | grep concentratord
    ```
2.  找到相關程序後，使用 `kill` 指令將其停止：

    ```bash
    kill <程序ID>
    ```

***

**Step 3: 獨立執行 Concentratord 進行測試**

1.  切換到 concentratord 執行目錄：

    ```bash
    cd /etc/linxdot-opensource/chirpstack-border-gateway/chirpstack-concentratord-binary
    ```
2.  執行測試命令：

    ```bash
    ./chirpstack-concentratord-sx1302 \
      -c ./config/concentratord.toml \
      -c ./config/region_as923.toml \
      -c ./config/channels_as923.toml
    ```

✅ **正常情況**：終端機會顯示相關執行日誌。

***

**Step 4: 重啟 Concentratord 服務**

執行以下命令重新啟動並啟用開機自動啟動：

```bash
/etc/init.d/linxdot-chirpstack-border-gateway-concentratord enable
/etc/init.d/linxdot-chirpstack-border-gateway-concentratord start
```

***

**Step 5: 查看執行日誌確認狀態**

持續查看 concentratord 執行日誌，確認運作是否正常：

```bash
logread -f | grep concentratord
```

***

🔎 **備註**：

* 如遇到無法停止的背景程序，請使用 `kill -9 <程序ID>` 強制終止。
* 若日誌中有錯誤訊息，請記錄下來以便後續排查或提供給技術支援。

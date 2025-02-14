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

**Relay Gateway** 是 **不直接連接到 ChirpStack**，且 **可在無網際網路連線的環境下運作** 的 **LoRaWAN 閘道器**。

***

#### **Relay Gateway 的功能**

1. **上行（Uplink）封包處理**
   * 接收 **LoRaWAN 裝置的 Uplink 資料**。
   * 將 **LoRaWAN 上行訊框（Uplink Frames）** **封裝（Encapsulate）** 成 **Mesh 格式**，然後轉發給下一個 **Relay Gateway 或 Border Gateway**。
2. **下行（Downlink）封包處理**
   * 從其他 **Relay Gateway 或 Border Gateway** 接收 **Mesh 格式的 LoRaWAN 下行訊框**。
   * **解封（Unwrap）Mesh 格式的 LoRaWAN 下行訊框**，並將資料發送給 **最終的 LoRaWAN 裝置**。

***

#### **Relay Gateway 的特點**

* **不需要網際網路連線**，適用於 **遠端地區、偏遠環境或無法提供網路基礎設施的場景**。
* **僅負責封包轉發**，它不直接與 **ChirpStack 伺服器** 溝通。
* **可多次跳轉（Multi-Hop）**，允許 LoRaWAN 資料在 **多個 Relay Gateway 之間傳遞**，最終送達 **Border Gateway**。

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

      chirpstack-gateway-bridge-as923:
        image: chirpstack/chirpstack-gateway-bridge:4
        restart: unless-stopped
        ports:
          - 1701:1700/udp  # Changed to 1701 to avoid conflict with multiplexer
        volumes:
          - ./configuration/chirpstack-gateway-bridge:/etc/chirpstack-gateway-bridge
        environment:
          - INTEGRATION__MQTT__EVENT_TOPIC_TEMPLATE=as923/gateway/{{ .GatewayID }}/event/{{ .EventType }}
          - INTEGRATION__MQTT__STATE_TOPIC_TEMPLATE=as923/gateway/{{ .GatewayID }}/state/{{ .StateType }}
          - INTEGRATION__MQTT__COMMAND_TOPIC_TEMPLATE=as923/gateway/{{ .GatewayID }}/command/#
        depends_on:
          - mosquitto
        logging:
          driver: "json-file"
          options:
            max-size: "10m"
            max-file: "3"
      
      chirpstack-gateway-bridge-basicstation:
        image: chirpstack/chirpstack-gateway-bridge:4
        restart: unless-stopped
        command: -c /etc/chirpstack-gateway-bridge/chirpstack-gateway-bridge-basicstation-as923.toml
        ports:
          - 3001:3001
        volumes:
          - ./configuration/chirpstack-gateway-bridge:/etc/chirpstack-gateway-bridge
        depends_on:
          - mosquitto
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

      chirpstack-packet-multiplexer:
        image: chirpstack/chirpstack-packet-multiplexer:4.0.0-test.2
        restart: unless-stopped
        command: -c /etc/chirpstack-packet-multiplexer/chirpstack-packet-multiplexer.toml
        ports:
          - 1700:1700/udp  # This service handles the gateway traffic
        volumes:
          - ./configuration/chirpstack-packet-multiplexer:/etc/chirpstack-packet-multiplexer
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
3.  新增與設定 chirpstack-packet-multiplexer.toml：

    {% code overflow="wrap" %}
    ```sh
    # 新增資料夾
    mkdir ./configuration/chirpstack-packet-multiplexer
    # Create chirpstack-packet-multiplexer.toml
    vi ./configuration/chirpstack-packet-multiplexer/chirpstack-packet-multiplexer.toml
    ```
    {% endcode %}
4.  進入 **插入模式**（按下 `i`），然後貼上以下內容：

    ```toml
    [logging]
    level = "info"

    [multiplexer]
    bind = "0.0.0.0:1700"

    [[multiplexer.server]]
    server = "chirpstack-gateway-bridge-as923:1700"
    uplink_only = true # Only forward uplink packets (no downlink)

    # Forward to an additional external server (e.g., remote ChirpStack instance)
    # [[multiplexer.server]]
    # server = "remote-server.example.com:1700"
    # uplink_only = true  # Only forward uplink packets (no downlink)

    # Forward to an additional external server (e.g., TTS instance)
    [[multiplexer.server]]
    server = "linxdot.as1.cloud.thethings.industries:1700"
    uplink_only = false
    ```
5. **儲存並退出**（按 `ESC`，輸入 `:wq`，然後按 `Enter`）。
6.  #### **執行指令以重新啟動 Docker 服務**

    1.  **先停止 Docker 容器**：

        ```sh
        docker-compose down
        ```
    2.  **重新啟動 Docker 容器**：

        ```sh
        docker-compose up -d
        ```

    執行這些指令後，系統會自動重新啟動 **ChirpStack LoRaWAN 伺服器**，並應用新的 `docker-compose.yml` 設定。
7.  現在，您可以使用以下指令來檢查 **日誌文件大小**：

    ```sh
    du -sh /opt/docker/containers/*
    ```

    這將顯示 **Docker 容器目錄**內各個文件夾的大小，幫助您監控 **日誌檔案的使用空間**。

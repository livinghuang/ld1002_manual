---
icon: hand-point-right
hidden: true
---

# Linxdot 安裝Chirpstack Concerntatord and mqtt forwarder

#### **本章將引導您在 Linxdot Hotspot 安裝** 封包轉發多工器

封包轉發多工器 允許使用 **Semtech UDP packet-forwarder** 協議的閘道器同時連接到多個伺服器，並且可以選擇將某些伺服器標記為**僅上行 (uplink only)**。

這意味著：

1. **多重伺服器支援**：一個 LoRaWAN 閘道器可以同時將數據傳輸到多個 ChirpStack 伺服器，而不需要為每個伺服器配置獨立的閘道器。
2. **上行數據選擇性傳輸**：可以將某些伺服器設定為僅接收上行數據（上傳來自 LoRa 裝置的訊息），而不會發送下行數據（如 LoRaWAN 訊息或指令）。
3. **適用於多平台架構**：若一個 LoRaWAN 網絡需要同時將數據發送到不同的應用或服務，這個多工器可以有效地管理流量，確保所有目標伺服器都能夠獲取相同的數據。

這樣的設計可提高數據靈活性，讓使用者能夠更有效地整合不同的 LoRaWAN 平台或數據處理系統。

#### **安裝與設定指南**

***

### **步驟 1：安裝 ChirpStack LoRaWAN** 封包轉發多工器

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

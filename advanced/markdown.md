---
icon: hand-point-right
---

# 設定LOG檔案大小

本章將引導您設定 **日誌文件大小**，以最佳化 **系統記憶體使用量**。

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
    ###### docker-compose.yml example
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
          - 1700:1700/udp
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
3. **儲存並退出**（按 `ESC`，輸入 `:wq`，然後按 `Enter`）。
4.  #### **執行指令以重新啟動 Docker 服務**

    1.  **先停止 Docker 容器**：

        ```sh
        docker-compose down
        ```
    2.  **重新啟動 Docker 容器**：

        ```sh
        docker-compose up -d
        ```

    執行這些指令後，系統會自動重新啟動 **ChirpStack LoRaWAN 伺服器**，並應用新的 `docker-compose.yml` 設定。
5.  現在，您可以使用以下指令來檢查 **日誌文件大小**：

    ```sh
    du -sh /opt/docker/containers/*
    ```

    這將顯示 **Docker 容器目錄**內各個文件夾的大小，幫助您監控 **日誌檔案的使用空間**。

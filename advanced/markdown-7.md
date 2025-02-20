---
icon: hand-point-right
hidden: true
---

# Linxdot 安裝 Chirpstack Gateway Mesh 並設定角色為 Border

#### **本章將引導您在 Linxdot Hotspot 安裝** 封包轉發多工器

封包轉發多工器 允許使用 **Semtech UDP packet-forwarder** 協議的閘道器同時連接到多個伺服器，並且可以選擇將某些伺服器標記為**僅上行 (uplink only)**。

這意味著：

1. **多重伺服器支援**：一個 LoRaWAN 閘道器可以同時將數據傳輸到多個 ChirpStack 伺服器，而不需要為每個伺服器配置獨立的閘道器。
2. **上行數據選擇性傳輸**：可以將某些伺服器設定為僅接收上行數據（上傳來自 LoRa 裝置的訊息），而不會發送下行數據（如 LoRaWAN 訊息或指令）。
3. **適用於多平台架構**：若一個 LoRaWAN 網絡需要同時將數據發送到不同的應用或服務，這個多工器可以有效地管理流量，確保所有目標伺服器都能夠獲取相同的數據。

這樣的設計可提高數據靈活性，讓使用者能夠更有效地整合不同的 LoRaWAN 平台或數據處理系統。

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
      border_gateway=true

      # Max hop count.
      #
      # This defines the maximum number of hops a relayed payload will pass.
      max_hop_count=3 # max=8

      # Ignore direct uplinks (Border Gateway).
      #
      # If this is set to true, then direct uplinks (uplinks that are not relay
      # encapsulated) will be silently ignored. This option is especially useful
      # for testing, in which case you want to set this to true for the Border
      # Gateway.
      border_gateway_ignore_direct_uplinks=true

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
7.  設定 Chirpstack Mqtt Forwarder 參數



    ```sh
     cd /etc/linxdot-opensource/chirpstack-border-gateway/chirpstack-mqtt-forwarder-binary
     rm chirpstack-mqtt-forwarder.toml
     vi chirpstack-mqtt-forwarder.toml
    ```
8.  進入 **插入模式**（按下 `i`），然後貼上以下內容：

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
      # If set to true, log messages are being written to syslog instead of stdout.
      log_to_syslog=false
      
    # MQTT settings.
    [mqtt]
      # Topic prefix.
      #
      # ChirpStack MQTT Forwarder publishes to the following topics:
      #
      #  * [Prefix/]gateway/[Gateway ID]/event/[Event]
      #  * [Prefix/]gateway/[Gateway ID]/state/[State]
      #
      # And subscribes to the following topic:
      #
      #  * [Prefix/]gateway/[Gateway ID]/command/[Command]
      #
      # The topic prefix can be used to define the region of the gateway.
      # Note, there is no need to add a trailing '/' to the prefix. The trailing
      # '/' is automatically added to the prefix if it is configured.
      topic_prefix="as923"
      
      # Use JSON encoding instead of Protobuf (binary).
      #
      # Note, only use this for debugging purposes.
      json=false

      # MQTT server (e.g. scheme://host:port where scheme is tcp, ssl or ws)
      server="tcp://127.0.0.1:1883"

      # Connect with the given username (optional)
      username="border-gateway"

      # Connect with the given password (optional)
      password="QWer!@34"

      # Quality of service level
      #
      # 0: at most once
      # 1: at least once
      # 2: exactly once
      #
      # Note: an increase of this value will decrease the performance.
      # For more information: https://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels
      qos=0

      # Clean session
      #
      # Set the "clean session" flag in the connect message when this client
      # connects to an MQTT broker. By setting this flag you are indicating
      # that no messages saved by the broker for this client should be delivered.
      clean_session=false

      # Client ID
      #
      # Set the client id to be used by this client when connecting to the MQTT
      # broker. A client id must be no longer than 23 characters. If left blank,
      # a random id will be generated by ChirpStack.
      client_id=""

      # CA certificate file (optional)
      #
      # Use this when setting up a secure connection (when server uses ssl://...)
      # but the certificate used by the server is not trusted by any CA certificate
      # on the server (e.g. when self generated).
      ca_cert=""

      # TLS certificate file (optional)
      tls_cert=""

      # TLS key file (optional)
      tls_key=""

    # Backend configuration.
    [backend]

      # Enabled backend.
      #
      # Set this to the backend that must be used by the ChirpStack MQTT Forwarder.
      # Valid options are:
      #   * concentratord
      #   * semtech_udp
      enabled="concentratord"

      # Filters.
      [backend.filters]
        # Forward CRC ok.
        forward_crc_ok=true

        # Forward CRC invalid.
        forward_crc_invalid=false

        # Forward CRC missing.
        forward_crc_missing=false

        # DevAddr prefix filters.
        #
        # Example configuration:
        # dev_addr_prefixes=["0000ff00/24"]
        #
        # The above filter means that the 24MSB of 0000ff00 will be used to
        # filter DevAddrs. Uplinks with DevAddrs that do not match any of the
        # configured filters will not be forwarded. Leaving this option empty
        # disables filtering on DevAddr.
        dev_addr_prefixes=[]

        # JoinEUI prefix filters.
        #
        # Example configuration:
        # join_eui_prefixes=["0000ff0000000000/24"]
        #
        # The above filter means that the 24MSB of 0000ff0000000000 will be used
        # to filter JoinEUIs. Uplinks with JoinEUIs that do not match any of the
        # configured filters will not be forwarded. Leaving this option empty
        # disables filtering on JoinEUI.
        join_eui_prefixes=[]

      # ChirpStack Concentratord backend configuration.
      [backend.concentratord]
        # Event API URL.
        # event_url="ipc:///tmp/concentratord_event"
        # Command API URL.
        # command_url="ipc:///tmp/concentratord_command"
        # Event PUB socket bind.
        event_bind="ipc:///tmp/gateway_relay_event"

        # Command REP socket bind.
        command_bind="ipc:///tmp/gateway_relay_command"

      # Semtech UDP backend configuration.
      [backend.semtech_udp]
        # ip:port to bind the UDP listener to.
        #
        # Example: 0.0.0.0:1700 to listen on port 1700 for all network interfaces.
        # This is the listener to which the packet-forwarder forwards its data
        # so make sure the 'serv_port_up' and 'serv_port_down' from your
        # packet-forwarder matches this port.
        bind="0.0.0.0:1700"

        # Time fallback.
        #
        # In case the UDP packet-forwarder does not set the 'time' field, then the
        # server-time will be used as fallback if this option is enabled.
        time_fallback_enabled=false

      # Gateway metadata configuration.
      [metadata]
      # Static key / value metadata.
      [metadata.static]
        # Example:
        # serial_number="1234"
      # Commands returning metadata.
      [metadata.commands]
        # Example:
        # datetime=["date", "-R"]
    # Executable commands.
    [commands]
      # Example:
      # reboot=["/usr/bin/reboot"]
    ```
9.  啟動服務：

    ```sh
     cd /etc/linxdot-opensource
     ./install-chirpstack-border-gateway-concentratord.sh 
     ./install-chirpstack-border-gateway-mesh.sh
     ./install-chirpstack-mqtt-fowarder.sh
    ```
10. 在LoRaWAN Server 設定 Gateway

<figure><img src="../.gitbook/assets/截圖 2025-02-21 清晨7.03.17.png" alt=""><figcaption></figcaption></figure>

1.

---
icon: hand-point-right
---

# Linxdot 安裝Chirpstack Concerntatord 封包集中器

#### **章將引導您在 Linxdot Hotspot 安裝** Chirpstack Concerntatord  封包集中器

ChirpStack Concentratord 是一個開源的 LoRaWAN 集中器守護程序，基於 Semtech 的硬體抽象層 (HAL) 所建構。它提供了一個基於 ZeroMQ 的 API，讓一個或多個應用程式可以與閘道硬體互動。 透過將硬體特定的實作與封裝放在單獨的守護程序中，並透過 ZeroMQ API 對外暴露，封包轉發應用程式能完全與閘道硬體解耦。這樣的架構也讓多個應用程式可以同時與閘道硬體互動。例如，多個封包轉發器可將資料分別轉發到不同的 LoRaWAN 網路伺服器。

<figure><img src="../.gitbook/assets/截圖 2025-02-23 凌晨4.29.03.png" alt=""><figcaption></figcaption></figure>

這意味著：

1. **多重伺服器支援**：一個 LoRaWAN 閘道器可以同時將數據傳輸到多個 ChirpStack 伺服器，而不需要為每個伺服器配置獨立的閘道器。
2. **上行數據選擇性傳輸**：可以將某些伺服器設定為僅接收上行數據（上傳來自 LoRa 裝置的訊息），而不會發送下行數據（如 LoRaWAN 訊息或指令）。
3. **適用於多平台架構**：若一個 LoRaWAN 網絡需要同時將數據發送到不同的應用或服務，這個多工器可以有效地管理流量，確保所有目標伺服器都能夠獲取相同的數據。

這樣的設計可提高數據靈活性，讓使用者能夠更有效地整合不同的 LoRaWAN 平台或數據處理系統。

#### **安裝與設定指南**

***

### **步驟 1：啟用** Chirpstack Concerntatord 封包集中器

1. 使用 **SSH** 登入您的 **Hotspot**。

<figure><img src="../.gitbook/assets/截圖 2025-02-12 上午8.35.21.png" alt=""><figcaption></figcaption></figure>

2.  設定 Chirpstack Concerntatord 封包集中器：

    {% code overflow="wrap" %}
    ```sh
    # 進入資料夾路徑
    cd /etc/linxdot-opensource/chirpstack-software/chirpstack-concentratord-binary/config
    # 刪除預設的 concentratord.toml 文件
    rm concentratord.toml
    # Create the new concentratord.toml
    vi concentratord.toml
    ```
    {% endcode %}

以下是新的 concentratord.toml 範例，您可以直接複製並使用 **vi 編輯器** 貼上到 **Hotspot** 的 concentratord.toml，然後儲存。

#### 步驟：

1.  使用 **vi 編輯器** 開啟 concentratord.toml：

    ```sh
    vi concentratord.toml
    ```
2.  進入 **插入模式**（按下 `i`），然後貼上以下內容：

    ```yaml
    # Concentratord configuration.
    [concentratord]

    # Log level.
    #
    # Valid options are:
    #   * TRACE
    #   * DEBUG
    #   * INFO
    #   * WARN
    #   * ERROR
    #   * OFF
    log_level="INFO"

    # Log to syslog.
    #
    # When set to true, log messages are being written to syslog instead of stdout.
    log_to_syslog=false

    # Statistics interval.
    stats_interval="30s"

      # Configuration for the (ZeroMQ based) API.
      [concentratord.api]

      # Event PUB socket bind.
      event_bind="ipc:///tmp/concentratord_event"

      # Command REP socket bind.
      command_bind="ipc:///tmp/concentratord_command"


    # LoRa gateway configuration.
    [gateway]

    # Antenna gain (dBi).
    antenna_gain=0

    # Public LoRaWAN network.
    lorawan_public=false

    # Region.
    #
    # The region of the gateway. Options:
    #  EU868, US915, CN779, EU433, AU915, CN470, AS923, AS923_2, AS923_3, AS923_4,
    #  KR923, IN865, RU864
    #
    # Not not all the gateway models implement all regions.
    region="AS923"

    # Gateway vendor / model.
    #
    # This configures various vendor and model specific settings like the min / max
    # frequency, TX gain table.
    model="linxdot_ld1002"

    # Gateway vendor / model flags.
    model_flags=[]
    ```
3.  啟動服務：

    ```sh
    cd /etc/linxdot-opensource
    ./install-chirpstack-concentratord.sh
    ```
4.  **查看執行日誌確認狀態**

    持續查看 concentratord 執行日誌，確認運作是否正常：

    ```bash
    logread -f | grep concentratord
    ```

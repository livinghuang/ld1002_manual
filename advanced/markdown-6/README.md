---
icon: hand-point-right
---

# Linxdot 安裝LoRaWAN服務(進階)

## 在 Linxdot 上安裝 LoRaWAN 服務集中器

本章將指導您如何在 **Linxdot** 上安裝 **LoRaWAN 服務集中器**。\
如果您 **重新刷機（Reflash）** 您的 Hotspot，LoRaWAN 服務可能不會自動安裝，因此需要手動重新安裝。請按照以下步驟完成設定。

<figure><img src="../../.gitbook/assets/截圖 2025-02-28 凌晨4.41.07.png" alt=""><figcaption></figcaption></figure>

***

## **在 Linxdot 安裝與設定 ChirpStack LoRaWAN 服務器**

本指南將引導您在 **Linxdot** 上安裝與設定 **ChirpStack LoRaWAN 服務器**，並確保其正常運行。

***

### **1. 安裝 ChirpStack LoRaWAN 服務器**

#### **步驟 1：使用 SSH 登入您的 Hotspot**

請確保您的 **Hotspot** 已連線至網路，然後使用 SSH 連接至設備：

```sh
ssh root@<HOTSPOT_IP>
```

#### **步驟 2：下載並安裝 ChirpStack**

執行以下指令下載並安裝 **ChirpStack**：

```sh
cd /opt
git clone https://github.com/livinghuang/awesome_linxdot.git
cd awesome_linxdot
./install-chirpstack.sh
```

**注意事項：**\
您可以在以下路徑 **修改 ChirpStack 設定**：

```sh
vi /opt/awesome_linxdot/chirpstack-software/chirpstack-docker/docker-compose.yml
```

***

### **2. 確認運行狀態**

安裝完成後，請使用以下指令檢查 **ChirpStack** 服務是否正常運行：

```sh
docker ps
```

您應該會看到類似以下的輸出：

```
CONTAINER ID   IMAGE                                    COMMAND                  CREATED       STATUS        PORTS                                       NAMES
af6b15b3bb6e   chirpstack/chirpstack-rest-api:4        "/usr/bin/chirpstack…"   2 weeks ago   Up 13 hours   0.0.0.0:8090->8090/tcp, :::8090->8090/tcp   chirpstack-docker_chirpstack-rest-api_1
67d2c8cbbf67   chirpstack/chirpstack:4                 "/usr/bin/chirpstack…"   2 weeks ago   Up 13 hours   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   chirpstack-docker_chirpstack_1
8a105aba202c   chirpstack/chirpstack-gateway-bridge:4  "/usr/bin/chirpstack…"   2 weeks ago   Up 13 hours   0.0.0.0:3001->3001/tcp, :::3001->3001/tcp   chirpstack-docker_chirpstack-gateway-bridge-basicstation_1
75f2a5d7f92f   chirpstack/chirpstack-gateway-bridge:4  "/usr/bin/chirpstack…"   2 weeks ago   Up 13 hours   0.0.0.0:1700->1700/udp, :::1700->1700/udp   chirpstack-docker_chirpstack-gateway-bridge-as923_1
994fa84be4a0   redis:7-alpine                          "docker-entrypoint.s…"   2 weeks ago   Up 13 hours   6379/tcp                                    chirpstack-docker_redis_1
147e129c6fdd   eclipse-mosquitto:2                     "/docker-entrypoint.…"   2 weeks ago   Up 13 hours   0.0.0.0:1883->1883/tcp, :::1883->1883/tcp   chirpstack-docker_mosquitto_1
8a9dd9d6cd8b   postgres:14-alpine                      "docker-entrypoint.s…"   2 weeks ago   Up 13 hours   5432/tcp                                    chirpstack-docker_postgres_1
```

您也可以查看 **ChirpStack 的 Docker 日誌** 以確保其正常運行：

```sh
docker logs --tail 10 chirpstack-docker_chirpstack_1
```

您應該會看到類似的日誌輸出：

```
2025-02-27T07:36:48.729783Z  INFO chirpstack::gateway::backend::mqtt: Starting MQTT event loop
2025-02-27T07:36:48.729798Z  INFO chirpstack::gateway::backend: Setting up gateway backend for region region_id=us915_1 region_common_name=US915
2025-02-27T07:36:48.742459Z  INFO chirpstack::gateway::backend::mqtt: Subscribing to gateway event topic region_id=us915_0 event_topic=$share/chirpstack/us915_0/gateway/+/event/+
2025-02-27T07:36:48.750240Z  INFO chirpstack::gateway::backend::mqtt: Connecting to MQTT broker region_id=us915_1 server_uri=tcp://mosquitto:1883 clean_session=false client_id=fc4e7d1b5705cb5d
2025-02-27T07:36:48.750768Z  INFO chirpstack::downlink: Setting up Class-B/C scheduler loop
2025-02-27T07:36:48.750842Z  INFO chirpstack::gateway::backend::mqtt: Starting MQTT event loop
2025-02-27T07:36:48.751085Z  INFO chirpstack::downlink: Setting up multicast scheduler loop
2025-02-27T07:36:48.751400Z  INFO chirpstack::api: Setting up API interface bind=0.0.0.0:8080
2025-02-27T07:36:48.756679Z  INFO chirpstack::gateway::backend::mqtt: Subscribing to gateway event topic region_id=us915_1 event_topic=$share/chirpstack/us915_1/gateway/+/event/+
2025-02-27T07:36:48.802157Z  INFO chirpstack::api::backend: Backend interfaces API interface is disabled
```

***

### **3. 停止與重新安裝 ChirpStack 服務**

#### **停止 ChirpStack 服務**

若需停止 **ChirpStack 服務**，請執行以下指令：

```sh
cd /opt/awesome_linxdot
./stop_chirpstack.sh
```

#### **修改設定並重新安裝**

如果您需要修改 **ChirpStack 設定**，請按照以下步驟操作：

1.  **停止服務**

    ```sh
    ./stop_chirpstack.sh
    ```
2.  **編輯 `docker-compose.yml` 設定**

    ```sh
    vi /opt/awesome_linxdot/chirpstack-software/chirpstack-docker/docker-compose.yml
    ```

    **在 `vi` 編輯器中：**

    * 按 **`i`** 進入編輯模式
    * 修改完成後，按 **`Esc`** 退出編輯模式
    * 輸入 **`:wq`** 保存並退出
3.  **重新安裝 ChirpStack**

    ```sh
    ./install-chirpstack.sh
    ```

***

### **4. 進階支援**

如需進一步技術支援，請參閱：

* [ChirpStack 官方支援頁面](https://www.chirpstack.io/)


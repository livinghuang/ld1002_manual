---
icon: hand-point-right
---

# Linxdot 安裝LoRaWAN採用服務集中器方法

這是您修訂後的版本，讓內容更加流暢且易讀：

***

## 在 Linxdot 上安裝 LoRaWAN 服務集中器

本章將指導您如何在 **Linxdot** 上安裝 **LoRaWAN 服務集中器**。\
如果您 **重新刷機（Reflash）** 您的 Hotspot，LoRaWAN 服務可能不會自動安裝，因此需要手動重新安裝。請按照以下步驟完成設定。

### 1. 安裝 ChirpStack LoRaWAN 服務器

#### **步驟 1：使用 SSH 登入您的 Hotspot**

確保您的 Hotspot 已連線至網路，然後使用 SSH 登入：

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

#### **注意事項**

* 您可以在 **`/opt/awesome_linxdot/chirpstack-software/chirpstack-docker/docker-compose.yml`** 修改 ChirpStack 的設定。

***

### 2. 安裝 ChirpStack Concentratord（封包集中器）

#### **步驟 1：安裝封包集中器**

為了確保 LoRaWAN 網關正常運作，請執行以下指令安裝 **ChirpStack Concentratord**：

```sh
cd /opt/awesome_linxdot
./install-concentratord.sh
```

#### **步驟 2：確認運行狀態**

安裝完成後，請使用以下指令檢查封包集中器的運行狀態：

```sh
docker ps
```

***

### 3. 停止與重新安裝 ChirpStack 服務

#### **停止 ChirpStack 服務**

若需停止 **ChirpStack** 服務，請執行以下指令：

```sh
cd /opt/awesome_linxdot
./stop_chirpstack.sh
```

#### **修改設定並重新安裝**

如果您需要修改設定，請先停止 ChirpStack 服務，然後重新安裝：

```sh
nano /opt/awesome_linxdot/chirpstack-software/chirpstack-docker/docker-compose.yml
./stop_chirpstack.sh
./install-chirpstack.sh
```

***

### 4. 進階支援

如需進一步技術支援，請參閱以下官方文件 [ChirpStack 官方支援頁面](https://www.chirpstack.io/)

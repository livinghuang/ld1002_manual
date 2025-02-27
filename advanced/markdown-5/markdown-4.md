---
icon: hand-point-right
---

# Linxdot 安裝UDP封包轉發器

***

## **在 Linxdot Hotspot 安裝 UDP 轉發器**

本章將指導您如何在 **Linxdot Hotspot** 上安裝 **ChirpStack UDP Forwarder**。\
**ChirpStack UDP Forwarder** 是專為 **ChirpStack Concentratord** 設計的 **UDP 封包轉發器**，並與 **Semtech UDP 協議** 相容。

<figure><img src="../../.gitbook/assets/截圖 2025-02-27 下午6.05.48.png" alt=""><figcaption></figcaption></figure>

***

### **安裝與設定指南**

#### **步驟 1：使用 SSH 登入您的 Hotspot**

請確保您的 **Hotspot** 已連線至網路，然後使用 SSH 連接至設備：

```sh
ssh root@<HOTSPOT_IP>
```

***

#### **步驟 2：下載並安裝 ChirpStack UDP Forwarder**

執行以下指令下載並安裝 **ChirpStack UDP Forwarder**：

```sh
cd /opt
git clone https://github.com/livinghuang/awesome_linxdot.git
cd awesome_linxdot
./install-chirpstack-udp-forwarder.sh
```

***

### **設定 ChirpStack UDP Forwarder**

您可以修改設定檔來調整 **UDP Forwarder** 的運行參數：

```sh
vi chirpstack-software/chirpstack-udp-forwarder-binary/<設定檔>.toml
```

**在 `vi` 編輯器中：**

* 按 **`i`** 進入編輯模式
* 修改完成後，按 **`Esc`** 退出編輯模式
* 輸入 **`:wq`** 保存並退出

***

### **停止 ChirpStack UDP Forwarder 服務**

如需停止 **ChirpStack UDP Forwarder**，請執行以下指令：

```sh
cd /opt/awesome_linxdot
./stop_and_remove_chirpstack_udp_forwarder.sh
```

***

### **進階支援**

如需進一步技術支援，請參閱：

* [ChirpStack 官方支援頁面](https://www.chirpstack.io/)


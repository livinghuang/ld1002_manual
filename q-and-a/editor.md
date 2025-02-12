---
icon: pen-to-square
---

# 常見問題 FAQ

#### **本章：常見問題解答（FAQ）**

本章將解答 **Linxdot Hotspot** 相關的 **常見問題**，幫助您更順利地使用設備與 LoRaWAN 服務。

***

#### **1. 如何確認我的 Hotspot 是否已成功安裝 LoRaWAN 服務？**

您可以透過 SSH 登入 Hotspot，並執行以下指令來檢查 **ChirpStack 服務** 是否正在運行：

```sh
docker ps
```

如果清單中包含 **ChirpStack 相關的 Docker 容器**，表示 **LoRaWAN 服務已正常運行**。如果沒有，請參考 **LoRaWAN 服務安裝章節** 來重新安裝。

***

#### **2. Linxdot Hotspot 的預設登入帳號與密碼是什麼？**

* **SSH 登入**
  * 帳號（Username）：`root`
  * 密碼（Password）：`linxdot`
* **ChirpStack Web 管理介面**（`http://{LinxdotIP}:8080`）
  * 帳號（Username）：`admin`
  * 密碼（Password）：`admin`

***

#### **3. Linxdot Hotspot 的預設 IP 地址是什麼？**

Hotspot 預設使用 **DHCP** 來分配 IP，您可以檢查 **路由器 DHCP 記錄** 來找到 Hotspot 的 IP。

如果 Hotspot 仍使用預設網路模式，可直接連接到 Linxdot AP（**SSID: Linxdot-AP，密碼: linxdot1234**）並存取：

```sh
http://10.100.100.1
```

***

#### **4. 如何更改 Linxdot Hotspot 的網路設定（DHCP 轉為固定 IP）？**

1. **登入 Hotspot 設定頁面**（`http://10.100.100.1`）。
2. 進入 **網路設定** 頁面。
3. 將 **網路模式** 從 **DHCP** 改為 **靜態 IP（Static IP）**，然後設定 **IP、子網路遮罩、閘道（Gateway）** 等資訊。
4. **儲存並應用設定**，然後重新啟動 Hotspot。

***

#### **5. 如何重新燒錄（Reflash）Linxdot Hotspot？**

如果 Hotspot 需要恢復出廠設定，請參考 **韌體重刷（Firmware Reflash）章節**，以下是基本流程：

1. **下載官方提供的韌體包**，並使用 **USB 連接 Hotspot**。
2. **進入燒錄模式**（按住 **BT Repair Key** 然後插入電源）。
3. **使用 Factory Tool 進行燒錄**，燒錄完成後重新啟動 Hotspot。

***

#### **6. 如何在 ChirpStack 註冊 LoRaWAN 閘道器？**

1. **登入 ChirpStack 管理介面**（`http://{LinxdotIP}:8080`）。
2. 進入 **Gateways（閘道器）** 頁面，點擊 **Add Gateway**。
3. 填入 **Gateway ID**（可從 `/etc/lora/global_conf.json` 內找到），並選擇適當的區域頻段（如 **AS923**）。
4. **儲存設定**，然後在 **Packet Forwarder 設定檔**中填入相同的 `gateway_ID`，確保數據可以順利轉發。

***

#### **7. Linxdot Hotspot 支援哪些 LoRa 頻段？**

Linxdot Hotspot 預設支援 **AS923 頻段**，可用頻率如下：

* **923.2MHz、923.4MHz、923.6MHz、923.8MHz**
* **924.0MHz、924.2MHz、924.4MHz、924.6MHz**

您可以在 **LoRa Packet Forwarder 設定檔（`/etc/lora/global_conf.json`）** 內修改頻段，以符合您的應用需求。

***

#### **8.** 如果您的 Hotspot 突然停止運行，該怎麼辦？

**a. 可能原因：內部記憶體已滿**

如果您的 **Linxdot Hotspot 突然停止運行**，請先檢查 **內部存儲是否已滿**，這可能會導致系統無法正常運行。

**解決方法：檢查記憶體使用情況**

請透過 SSH 連線至 Hotspot，並執行以下指令來檢查記憶體狀況：

```sh
df -h
```

如果 **`/` 根目錄 或 `/opt` 目錄** 的 **可用空間接近 0%**，則表示存儲已滿，可能影響 Hotspot 的運行。請參考[設定LOG檔案大小](../advanced/markdown.md) 章節

***

這些是 **Linxdot Hotspot** 最常見的問題與解決方案。如果您有其他問題，請參閱 **Linxdot 官方文件** 或 **ChirpStack 支援頁面**！ 🚀


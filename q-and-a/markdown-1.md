---
icon: hand-point-right
---

# 設定Internal LoRaWAN Gateway Package Forwarder

本章將引導您設定 **內部 LoRaWAN 閘道器封包轉發器參數**。

如您所知，**Linxdot Hotspot** 內建 **LoRaWAN 通訊模組**，並作為 **LoRaWAN 閘道器**，其 **封包轉發器（Packet Forwarder）** 採用 **Semtech LoRa Gateway Packet Forwarder** 作為基礎程式。

您可以透過修改設定來：

1. **切換目標區域頻段（Region Frequency）**，以符合您的應用需求。
2. **變更 Gateway ID**，確保唯一識別您的閘道器。
3. **修改目標 LoRaWAN 伺服器**，將 LoRaWAN 資料流量轉發至您的指定伺服器。

#### **開始本教學**

我們將逐步引導您 **設定內部 LoRaWAN 閘道器的封包轉發器（Packet Forwarder）**，確保您的 **Linxdot Hotspot** 正確運行並符合您的應用需求。

請按照以下步驟進行：

**1. 使用 SSH 登入 Linxdot Hotspot**

請先透過 **SSH** 連線至您的 Hotspot：

```sh
ssh root@your_hotspot_ip
```

> 預設密碼為 `linxdot`

<figure><img src="../.gitbook/assets/截圖 2025-02-12 上午8.35.21.png" alt=""><figcaption></figcaption></figure>

**2. 進入Linxdot封包轉發器設定檔案，**&#x60A8;可以更改 **目標使用區域（Region）**，但 **LoRa Packet Forwarder** 會透過 **系統服務（**&#x72;un\_lora\_pkt\_fwd.s&#x68;**）** 來讀取設定檔案。

```sh
cat /etc/init.d/linxdot-lora-pkt-fwd
```

```sh
#!/bin/sh /etc/rc.common
    
    START=99
    USE_PROCD=8999

    thisRegion=AS923_1

   start_service() {

        logger -t starting lora_pkt_fwd service!....
        procd_open_instance
        procd_set_param command /etc/linxdot-opensource/run_lora_pkt_fwd.sh $thisRegion
        procd_set_param respawn
        procd_close_instance

        logger -t ora_pkt_fwd service started!
    }

```

讀取 /etc/linxdot-opensource/run\_lora\_pkt\_fwd.sh

```sh
cat /etc/linxdot-opensource/run_lora_pkt_fwd.sh
```

```sh
#!/bin/sh

# Linxdot opensource: 
# Main purpose: to call the lora_pkt_fwd runtime in backgrond;
# By Louis Chuang 2024-04-04.

if [ -z "$1" ]
then

      echo "the parameter is empty use default"
      region="AS923_1"
else
      echo "the regions is $1"
      region=$1
fi

cd /etc/lora
lora_pkt_fwd -c /etc/lora/global_conf.json.sx1250.$region > /var/log/lora_pkt_fwd.log 2>&1 &

while [ true ]; do

    echo 1 > /sys/devices/platform/leds/leds/work-led1/brightness
    echo 1 > /sys/devices/platform/leds/leds/work-led2/brightness  
    sleep 1
    echo 0 > /sys/devices/platform/leds/leds/work-led1/brightness
    echo 1 > /sys/devices/platform/leds/leds/work-led2/brightness 
    sleep 1

done
```

**3. 編輯封包轉發器設定檔**

使用 `vi` 或 `nano` 來修改 `global_conf.json`：

```sh
vi /etc/lora/global_conf.json.sx1250.AS923_1
```

```sh
{
    "SX130x_conf": {
        "spidev_path": "/dev/spidev0.0",
        "com_type": "SPI",
        "com_path": "/dev/spidev0.0",
        "lorawan_public": true,
        "clksrc": 0,
        "antenna_gain": 0,
        "full_duplex": false,
        "precision_timestamp": {
            "enable": false,
            "max_ts_metrics": 255,
            "nb_symbols": 1
        },
        "radio_0": {
            "enable": true,
            "type": "SX1250",
            "freq": 923500000,
            "rssi_offset": -215.4,
            "rssi_tcomp": {"coeff_a": 0, "coeff_b": 0, "coeff_c": 20.41, "coeff_d": 2162.56, "coeff_e": 0},
            "tx_enable": true,
            "tx_freq_min": 915000000,
            "tx_freq_max": 928000000,
            "tx_gain_lut":[
                {"rf_power": 1, "pa_gain": 0, "pwr_idx": 7},
                {"rf_power": 2, "pa_gain": 0, "pwr_idx": 8},
                {"rf_power": 3, "pa_gain": 0, "pwr_idx": 9},
                {"rf_power": 4, "pa_gain": 0, "pwr_idx": 10},
                {"rf_power": 5, "pa_gain": 0, "pwr_idx": 11},
                {"rf_power": 6, "pa_gain": 0, "pwr_idx": 12},
                {"rf_power": 7, "pa_gain": 0, "pwr_idx": 13},
                {"rf_power": 8, "pa_gain": 0, "pwr_idx": 14},
                {"rf_power": 9, "pa_gain": 0, "pwr_idx": 15},
                {"rf_power": 10, "pa_gain": 0, "pwr_idx": 16},
                {"rf_power": 11, "pa_gain": 0, "pwr_idx": 17},
                {"rf_power": 12, "pa_gain": 0, "pwr_idx": 18},
                {"rf_power": 13, "pa_gain": 0, "pwr_idx": 19},
                {"rf_power": 14, "pa_gain": 0, "pwr_idx": 20},
                {"rf_power": 15, "pa_gain": 0, "pwr_idx": 21},
                {"rf_power": 16, "pa_gain": 0, "pwr_idx": 22},
                {"rf_power": 17, "pa_gain": 0, "pwr_idx": 22},
                {"rf_power": 18, "pa_gain": 1, "pwr_idx": 0},
                {"rf_power": 19, "pa_gain": 1, "pwr_idx": 0},
                {"rf_power": 20, "pa_gain": 1, "pwr_idx": 0},
                {"rf_power": 21, "pa_gain": 1, "pwr_idx": 0},
                {"rf_power": 22, "pa_gain": 1, "pwr_idx": 0},
                {"rf_power": 23, "pa_gain": 1, "pwr_idx": 0},
                {"rf_power": 24, "pa_gain": 1, "pwr_idx": 1},
                {"rf_power": 25, "pa_gain": 1, "pwr_idx": 3},
                {"rf_power": 26, "pa_gain": 1, "pwr_idx": 5},
                {"rf_power": 27, "pa_gain": 1, "pwr_idx": 16}
            ]
        },
        "radio_1": {
            "enable": true,
            "type": "SX1250",
            "freq": 924300000,
            "rssi_offset": -215.4,
            "rssi_tcomp": {"coeff_a": 0, "coeff_b": 0, "coeff_c": 20.41, "coeff_d": 2162.56, "coeff_e": 0},
            "tx_enable": false
        },
        "chan_multiSF_All": {"spreading_factor_enable": [ 5, 6, 7, 8, 9, 10, 11, 12 ]},
            "chan_multiSF_0": {"enable": true, "radio": 0, "if": -300000},
            "chan_multiSF_1": {"enable": true, "radio": 0, "if": -100000},
            "chan_multiSF_2": {"enable": true, "radio": 0, "if":  100000},
            "chan_multiSF_3": {"enable": true, "radio": 0, "if":  300000},
            "chan_multiSF_4": {"enable": true, "radio": 1, "if": -300000},
            "chan_multiSF_5": {"enable": true, "radio": 1, "if": -100000},
            "chan_multiSF_6": {"enable": true, "radio": 1, "if":  100000},
            "chan_multiSF_7": {"enable": true, "radio": 1, "if":  300000},
            "chan_Lora_std":  {"enable": true, "radio": 0, "if":  300000, "bandwidth": 500000, "spread_factor": 8,
                               "implicit_hdr": false, "implicit_payload_length": 17, "implicit_crc_en": false, "implicit_coderate": 1},
        "chan_FSK":{
            "enable": false,
            "radio": 1,
            "if":  300000,
            "bandwidth": 125000,
            "datarate": 50000
        }
    },
    "gateway_conf": {
        "gps_i2c_path": "/dev/i2c-1",
        "gateway_ID": "482739121eea031a",
        "server_address":"127.0.0.1",
        "serv_port_up": 1700,
        "serv_port_down": 1700,
        "keepalive_interval": 10,
        "stat_interval": 30,
        "push_timeout_ms": 100,
        "forward_crc_valid": true,
        "forward_crc_error": false,
        "forward_crc_disabled": false
    }
}
```

**4. 變更 Gateway ID**

找到 `"gateway_ID"` 欄位，修改為您的 **唯一閘道器 ID**（應使用 **8-byte EUI** 格式，例如 `AABBCCFFFE112233`）：

```json
"gateway_ID": "AABBCCFFFE112233"
```

**5. 設定目標 LoRaWAN 伺服器**

找到 `"server_address"`，將其修改為您的 **LoRaWAN 伺服器 IP 或域名**：

```json
"server_address": "10.100.100.1"
```

> 預設為 **Linxdot 內建的 LoRaWAN 伺服器**，但您也可以設定為 **外部伺服器**。

**6. 儲存並退出**

若使用 `vi` 編輯器，按 `ESC` 鍵，然後輸入：

```sh
:wq
```

並按 `Enter`。

**8. 重新啟動Hotspot**

***

現在，您的 Linxdot Hotspot 已經根據您的需求進行 **LoRaWAN 頻段、閘道器 ID** 以及 **伺服器地址** 設定。

---
icon: globe-pointer
---

# LORAWAN 快速使用指南

LD1002 內建 ChirpStack LoRaWAN 伺服器，並已預先設定內部 LoRaWAN 通訊模組作為預設 LoRaWAN 閘道器。以下是快速指南，幫助您登入並註冊終端節點至伺服器。

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午2.43.31.png" alt=""><figcaption></figcaption></figure>

1.  開啟網頁瀏覽器，並輸入 `http://your_linxdot_hotspot_ip:8080`，其中 `your_linxdot_hotspot_ip` 預設為 `10.100.100.1`。

    預設帳號：`admin`\
    預設密碼：`admin`

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午2.48.08.png" alt=""><figcaption></figcaption></figure>

2. 進入系統後，您將看到預設的 LoRaWAN 閘道器，該閘道器已設定為 **AS923** 頻段計劃。 &#x20;

note. 它支援以下頻率訊號：

**923.2MHz、923.4MHz、923.6MHz、923.8MHz、924.0MHz、924.2MHz、924.4MHz、924.6MHz**

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午2.52.00.png" alt=""><figcaption></figcaption></figure>

3. 進入 **「Device profiles」** 頁面，設定您的第一個設備配置檔案。

note. 我們將引導您設定第一個設備配置，遵循 **LoRaWAN 1.0.4** 協議，並使用 **ABP** 模式作為基本應用方式。

4. 請按照下圖所示進行設定您的頁面。

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午3.03.29.png" alt=""><figcaption></figcaption></figure>

5. 進入 **「Join (OTAA / ABP)」** 標籤，將加入LORAWAN方式更改為 **ABP**。

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午3.04.09.png" alt=""><figcaption><p>Disable support OTAA</p></figcaption></figure>

6. 請檢查您的 **Device Profiles** 設定，確認結果與下圖相同。

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午3.04.29.png" alt=""><figcaption></figcaption></figure>

7. 現在進入 **「Applications」** 頁面，設定第一個應用程式，然後將第一個終端節點註冊到該應用程式中。

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午3.16.02.png" alt=""><figcaption></figcaption></figure>

8. 建立應用程式後，點擊 **「Add device」** 按鈕，將第一個設備新增至該應用程式。

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午3.22.21.png" alt=""><figcaption></figcaption></figure>

9.  建立第一個設備後，請點擊 **「Activation」** 標籤以啟動該節點。您需要填寫所有 **紅色星號** 標記的必填資訊，以完成此階段。

    完成後，您已成功將第一個終端節點設備啟動並接入 **LoRaWAN 伺服器**。

<figure><img src="../.gitbook/assets/截圖 2025-02-11 下午3.23.15.png" alt=""><figcaption></figcaption></figure>

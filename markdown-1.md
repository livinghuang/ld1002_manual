---
icon: hand-point-right
---

# 停用 Watchcat 自動重啟功能（避免不正常重啟）

#### 問題說明

Linxdot 預設啟用了 Watchcat（網路監控自動重啟）功能，當無法 ping 通外部 IP（如 8.8.8.8）時，會觸發強制重啟。

此機制雖具備網路自復功能，但若：

* 外部網路短暫中斷
* Docker 初始化過慢
* DNS 設定異常或 ping 被封鎖

都會導致 **誤判當機 → 重啟 → Docker 與 LUCI 異常 → 持續循環**。

***

#### 解法：停用 Watchcat 重啟機制

**方法一：透過 LuCI 網頁關閉**

1. 登入 Web UI（LuCI）
2. 前往「服務」>「Watchcat」
3. 將 **模式** 改為「不執行任何操作」
4. 按下 **儲存 & 套用**

**方法二：SSH 命令刪除 Watchcat 設定**

```sh
sh複製編輯/etc/init.d/watchcat stop
/etc/init.d/watchcat disable
rm -f /etc/config/watchcat
```

或進一步移除 package（如不再使用）

```sh
sh複製編輯opkg remove watchcat
```

***

#### 建議替代方案

您可以使用自訂健康檢查腳本 + log 備份機制：

* 檢查 Docker 容器狀態
* 檢查 uhttpd (LUCI) 狀態
* 發現異常時儲存 log 至 /root/backup 再人工處理

***

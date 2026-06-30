## 通過 Creality Helper Script 為 Creality 印表機刷入 iHeater 韌體

為了成功刷入韌體並整合 iHeater，請按照以下逐步說明進行：

### 1. 安裝 Creality Helper Script

進入 Creality Helper Script 專案的文檔頁面，並按照指令安裝指令碼。

**資源：**

* 影片教學：[YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* 文字說明：[guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. 取得對印表機的 root 存取權及檔案系統存取權

該指令碼將開啟對 Mainsail 以及韌體配置檔案的存取權。成功安裝後，請確認您可以透過瀏覽器進入印表機介面並取得對配置檔案的存取權。

### 3. 刪除舊的 `fan-control.cfg`

Creality 印表機上的 Helper Script 預設可能已建立 `fan-control.cfg` 檔案，其中包含 `M141` 和 `M191` 巨集。它與 iHeater 配置中的類似巨集產生衝突。

重新命名檔案：

```
/usr/data/printer_data/config/fan-control.cfg
```
為 fan-control.cfg.bak

### 4. 複製新的 `fan-control.cfg`

將其替換為 [fan-control.cfg](../../../printers/creality/config/fans-control.cfg) 版本，與巨集和室溫管理邏輯相容。

將新檔案放置在相同資料夾中：

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. 新增 iHeater 配置

將 `iheater.cfg` 檔案複製到同一目錄中：

```
/usr/data/printer_data/config/iheater.cfg
```

然後開啟 `printer.cfg` 並在檔案結尾新增以下行：

```ini
[include iheater.cfg]
```

---

接下來，請按照 iHeater 配置說明進行操作 - 配置熱敏電阻、加熱器、工作模式及巨集。

!!! warning "如果無法在印表機上編譯並刷入韌體"
    [請參閱 WSL 部分](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

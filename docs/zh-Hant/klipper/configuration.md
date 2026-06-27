# Klipper 配置

本頁面說明 iHeater 配置文件的安裝以及與 Klipper 的配置。

## 需求

### 硬件
  - iHeater 控制板
  - NTC 100K 3950 溫度傳感器（2 個）
  - PTC 加熱元件 220V 100W，用於腔室
  - 風扇 7530 220V，用於腔室空氣流通
  - Thermal Protector KSD9700 或類似產品（220 V，5 A，130 °C）

### 軟件
  - Klipper（最新版本）
  - 已配置並正常工作的 Klipper 主機

## Klipper 配置

將 iHeater.cfg 配置文件複製到 printer.cfg 所在的文件夾（可能是 /klipper_config），並使用 [include] 指令在 printer.cfg 中引入該文件。

```
cd ~/klipper_config
```

```
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

打開 printer.cfg 並添加

    [include iHeater.cfg]

## 連接 MCU iHeater

編輯 iHeater.cfg 文件，指定獲得的 ID

```
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ

```

## 使用前準備

配置文件包含以下部分：

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # 目標腔室溫度，°C
variable_start_offset: 10           # 足以開始打印的腔室溫度，°C
variable_delta_temp: 10             # 腔室溫度與加熱器溫度之間的差異，°C
variable_min_heater_temp: 50        # 最小加熱器溫度（用於冷卻），°C
variable_max_heater_temp: 100       # 最大加熱器溫度，°C
variable_control_interval: 1.0      # 控制函數調用間隔，秒
variable_air_min_delta: 0.5         # 目標腔室溫度與實際溫度之間的最小差異（加熱器 = 目標 + delta_temp），°C
variable_air_max_delta: 5.0         # 目標腔室溫度與實際溫度之間的最大差異（加熱器 = max_heater_temp），°C
gcode:
```

**最大允許加熱器溫度取決於外殼材料。**

檢驗步驟：

1. 將打印床加熱到 90-100°C
2. 通過 Fluidd 或 Mainsail 界面將加熱器溫度設置為 100°C
3. 確認 iHeater 位於打印機的密閉空間內
4. 達到設置溫度後，檢查加熱器與塑料外殼接觸的區域。塑料不應軟化。
5. 將溫度提高 5-10°C 並重複檢查。
6. 重複此過程，直到找到不會導致外殼變形的最大允許加熱器溫度。

這種方法可以確定安全的最高溫度，並實現 iHeater 的最佳性能。

## 使用

### 腔室加熱控制命令
- 設置腔室溫度：


        M141 S60  ; 將腔室溫度設置為 60°C

- 等待達到溫度：

        M191 S60  ; 等待腔室溫度達到 60°C

- 停止腔室加熱：

        iHEATER_OFF   ; 關閉腔室加熱

- 在切片機 G-code 的末尾添加 `iHEATER_OFF`，以正確關閉腔室加熱。

### 起始 G-code

現代切片機支持在生成打印 G-code 時自動啟用活躍熱腔室。為此，需要在耗材屬性中指定腔室溫度。如果切片機沒有此功能，需要在起始 G-code 中添加啟用活躍熱腔室加熱的命令。

操作步驟：

- 設置目標腔室溫度
- 啟用打印床加熱，以加速和改進腔室加熱
- 繼續標準打印起始 G-code

起始 G-code 示例
```
; --- 起始 G-code 開始 ---

; ****** iHeater 啟動 ******
M141 S60       ; 將腔室溫度設置為 60°C
; ****** iHeater 塊結束 ******

; --- 其餘起始 G-code ---
; 啟用打印床加熱
...
```
!!! warning "為了正確完成 iHeater 控制宏的工作，需要在打印機結束 G-code 中添加 iHEATER_OFF 命令"

```
; --- 結束 G-code 開始 ---

; ****** iHeater 塊啟動 ******
iHEATER_OFF
; ****** iHeater 塊結束 ******

; --- 其餘結束 G-code ---
...
```
## 禁用

要禁用 iHeater，在 printer.cfg 文件中註釋掉 [include iHeater.cfg] 行

```
# [include iHeater.cfg]
```

並從起始和結束 G-code 中刪除相應行

## 注意
- 安全性：

    - 確保所有連接都已正確和安全地進行。
    - 驗證 min_temp 和 max_temp 值符合設備規格。

- 硬件檢查：
    - 在使用前測試加熱器和風扇的工作。
    - 在首次運行期間監控溫度。
- PID 調整：
    - 如需要，執行 PID 校準以實現精確的溫度控制。

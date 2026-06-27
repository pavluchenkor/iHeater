# Klipper 配置

本页说明 iHeater 配置文件的安装，以及与 Klipper 配合使用的设置。

## 要求

### 硬件
  - iHeater 控制板
  - NTC 100K 3950 热敏电阻（2 个）
  - 用于腔体的 220V 100W PTC 加热元件
  - 用于腔体空气循环的 7530 220V 风扇
  - Thermal Protector KSD9700 或类似产品（220 V、5 A、130 °C）

### 软件
  - Klipper（最新版本）
  - 已配置并正常运行的 Klipper 主机

## Klipper 配置


将 iHeater.cfg 配置文件复制到 printer.cfg 所在的文件夹（可能是 /klipper_config），并使用 [include] 指令在 printer.cfg 中引用它


```
cd ~/klipper_config
```

```
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

打开 printer.cfg 并添加

    [include iHeater.cfg]

## 连接 iHeater MCU

修改 iHeater.cfg 文件，填入获取到的 ID

```
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ

```

## 使用前准备

配置文件包含以下 section：

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # Целевая температура камеры, °C
variable_start_offset: 10           # Температура камеры, достаточная для начала печати, °C
variable_delta_temp: 10             # Разница между температурой камеры и нагревателя, °C
variable_min_heater_temp: 50        # Минимальная температура нагревателя (для охлаждения), °C
variable_max_heater_temp: 100       # Максимальная температура нагревателя, °C
variable_control_interval: 1.0      # Интервал вызова функции управления, секунды
variable_air_min_delta: 0.5         # Минимальная разница между целевой и текущей температурой камеры (нагреватель = целевая + delta_temp), °C
variable_air_max_delta: 5.0         # Максимальная разница между целевой и текущей температурой камеры (нагреватель = max_heater_temp), °C
gcode:
```

**加热器允许的最高温度取决于外壳材料。**

检查方法：

!. 开启热床加热至 90-100°C
1. 通过 Fluidd 或 Mainsail 界面将加热器温度设置为 100°C。
2. 确保 iHeater 位于打印机封闭空间内部。
3. 达到设定温度后，检查加热器与外壳塑料部件接触的区域。塑料不应软化。
4. 将温度提高 5-10°C 并重复检查。
5. 重复上述步骤，直到在没有外壳变形风险的情况下达到加热器允许的最高温度。

这种方法可以确定安全的最高温度，并让 iHeater 达到最佳工作效率。


## 使用

### 腔体加热控制命令
- 设置腔体温度：
 

        M141 S60  ; Устанавливает температуру камеры на 60°C

- 等待达到温度：

        M191 S60  ; Ждет, пока температура камеры достигнет 60°C

- 停止腔体加热：

        iHEATER_OFF   ; Отключает нагрев камеры

- 在切片软件的结束 G-code 中添加 `iHEATER_OFF`，以正确关闭腔体加热。

### 启动 g-code

现代切片软件支持在生成打印 g-code 时自动启用主动恒温腔。为此，需要在耗材属性中指定腔体温度。如果切片软件没有此功能，则需要在启动 g-code 中添加启用主动恒温腔加热的命令。

操作顺序：

- 设置腔体目标温度
- 开启热床加热，以便高效、快速地加热腔体 
- 继续执行标准的打印启动 G-code

启动 g-code 示例
```
; --- Начало стартового G-code ---

; ****** Старт iHeater ******
M141 S60       ; Установить температуру камеры на 60°C
; ****** Конец блока iHeater ******

; --- Остальной стартовый g-code ---
; Включение нагрева стола
...
```
!!! warning "为正确结束 iHeater 控制宏的工作，必须在打印机的结束 g-code 中添加 iHEATER_OFF 命令"

```
; --- Начало завершающего g-code ---

; ****** Старт блока iHeater ******
iHEATER_OFF
; ****** Конец блока iHeater ******

; --- Остальной завершающий g-code ---
...
```
## 禁用

要禁用 iHeater，需要在 printer.cfg 文件中注释掉 [include iHeater.cfg] 这一行
```
# [include iHeater.cfg]
```

并从启动和结束 g-code 中删除相应的行

## 备注
- 安全：

    - 确保所有连接都正确且安全。
    - 检查 min_temp 和 max_temp 的值是否符合设备规格。

- 设备检查：
    - 使用前测试加热器和风扇的工作情况。
    - 首次启动期间注意监控温度。
- PID 调整：
    - 如有需要，请执行 PID 校准以实现精确的温度控制。

# 固件

本文档说明如何为 **iHeater** 微控制器构建并刷写 **Katapult** 引导加载程序。Katapult 允许通过 USB 更新 Klipper 固件，也用于在 **iHeater** 控制器上安装 **Klipper** 固件。

---

## 要求

* STM32F042F6P6
* iHeater 板
* 用于首次刷写的 ST-Link V2 编程器，或 USB 线缆
* Linux 系统，例如 Raspberry Pi 或打印机

!!! warning "如果无法直接在打印机上构建并刷写固件"
    [请参阅 WSL 部分](user-mods/software/wsl2-ubuntu-ff/)

---

## 构建 Katapult

1. 克隆 Katapult 仓库：

```bash
git clone https://github.com/Arksine/katapult.git
```

```bash
cd katapult
```

```bash
make menuconfig
```

2. 在 `menuconfig` 中选择：

* MCU Architecture: STM32
* Processor model: STM32F042
* Clock Reference: Internal
* Communication interface: USB (on PA9/PA10)
* Application start offset: **8KiB offset**
* [x] Support bootloader entry on rapid double click of reset button
* [x] Enable bootloader entry on button (or gpio) state
* (!PA4) Button GPIO Pin
* [x] Enable Status LED
* (PA5) Status LED GPIO Pin

![menuconfig](../img/katapult_menuconfig.jpg)

3. 构建固件：

```bash
make
```

固件文件将生成在 `out/katapult.bin` 路径下。

---

## 通过 DFU 刷写 Katapult

> 此步骤只需执行一次，用于写入 Katapult 引导加载程序。

### 准备

如果尚未安装 `dfu-util`，请安装：

```bash
sudo apt install dfu-util
```

安装 BOOT0 跳线，并重新给板子上电，或按下 RESET。
微控制器将进入 DFU 模式。

检查连接：

```bash
lsusb
```

应出现以下设备：

```text
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### 刷写 Katapult

执行：

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

成功输出示例：

```text
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

移除跳线，按住 MODE 按钮，按下并松开 RESET，或重新连接 USB。

重启后执行：

```bash
ls /dev/serial/by-id/*
```

应出现以下设备：

```bash
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

如果出现权限访问错误：

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

### 备注

* Katapult 占用 Flash 的前 8 KB，因此 **必须在 Klipper 中指定 8KiB 偏移**。
* 可以通过双击 RESET 或 GPIO 按钮 (PA4) 进入 DFU。
* PA13/PA14 用于 SWD。
* 刷写 Katapult 后不再需要 ST-Link 编程器；后续更新可以通过 USB 执行。

## 在 iHeater 上安装固件

### 构建固件

```bash
cd klipper/
```

```bash
make menuconfig
```

#### 在配置菜单中选择：

```text
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### 禁用所有未使用的选项：

```text
[*] Support micro-controller based ADC (analog to digital)
[ ] Support communicating with external chips via SPI bus
[ ] Support communicating with external chips via I2C bus
[*] Support GPIO based button reading
[ ] Support Trinamic stepper motor driver UART communication
[ ] Support 'neopixel' type LED control
[ ] Support measuring fan tachometer GPIO pins
    *** LCD chips ***
[ ] Support ST7920 LCD display
[ ] Support HD44780 LCD display
    *** External ADC type chips ***
[ ] Support HX711 and HX717 ADC chips
```

#### 保存设置并退出菜单。

#### 构建固件

```bash
make clean
```

```bash
make
```

!!! note "结果应出现以下文件："

```text
Creating hex file out/klipper.bin
```

### 将固件安装到 iHeater 板

!!! note "如有需要，请安装 python3-serial"

```bash
sudo apt install python3-serial
```

**以下步骤假定 Katapult 引导加载程序已经安装。**

* 将 iHeater 以刷写模式连接到主机：连接 USB 时按住 MODE，或双击 RESET。

* 查找设备：

```bash
ls /dev/serial/by-id/
```

应出现类似以下内容：

```text
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* 如有需要，请安装 `flashtool`：

```bash
pip install flashtool
```

* 将设备 ID 替换为你自己的，然后执行：

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

预期输出：

```text
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...
```

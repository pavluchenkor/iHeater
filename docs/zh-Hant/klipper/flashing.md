#  刷寫固件

本文件包含用於構建和刷寫 **Katapult** 引導程式至 **iHeater** 微控制器的說明。Katapult 引導程式允許通過 USB 刷寫 Klipper 固件，並且包含在 **iHeater** 控制器上安裝 **Klipper** 固件的文檔。

---

## 所需物品

- STM32F042F6P6
- iHeater 板卡
- USB 電纜
- Linux 系統（例如 Raspberry Pi 或印表機）

!!! warning "如果無法在印表機上編譯和刷寫固件"
    [請參考 WSL 部分](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

---

## 構建 Katapult

1. 克隆 Katapult 存儲庫：

```bash
git clone https://github.com/Arksine/katapult.git
```
```
cd katapult
```
```
make menuconfig
```

2. 在 `menuconfig` 中選擇：

![menuconfig](../../img/katapult_menuconfig.jpg)

3. 構建：

```bash
make
```

固件將在 `out/katapult.bin` 中創建。


---

## 通過 DFU 刷寫 Katapult

> 此步驟只需執行一次，用於加載 Katapult 本身。

### 準備：
安裝 dfu-util 工具（如果尚未安裝）：
    
    sudo apt install dfu-util

根據板卡版本：

=== "r1"

    在 BOOT0 上放置跳線並重新啟動板卡電源（或按 RESET 按鈕）。
    微控制器將在 DFU 模式下啟動。

=== "r1.1"

    按下 BOOT 按鈕並重新啟動板卡電源（或按 RESET 按鈕），然後鬆開 BOOT。
    微控制器將在 DFU 模式下啟動。

檢查連接：

    lsusb

結果：

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### 刷寫 Katapult：
進入 DFU 模式。

執行命令：

```
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

成功刷寫的例子：

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

退出 DFU 模式。

重新啟動後
```
ls /dev/serial/by-id/*
```
結果：
```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

如果沒有權限，刷寫時可能會出現錯誤。要獲得訪問權限，請執行命令：
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

!!! warning "如果出現問題"
    擦除微控制器內存並重複前面的步驟
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### 註釋

- Katapult 佔用前 8 KB Flash，因此 **在 Klipper 中必須指定 8 KiB 偏移量**。
- 可以使用雙重 Reset 或 GPIO (PA4) 上的按鈕進入 DFU 模式。
- 如果 PA13/PA14 用於 SWD
- 刷寫 Katapult 後，不再需要使用 ST-Link - 所有後續工作都通過 USB。

## 在 iHeater 上安裝固件

### 編譯固件
```
cd ~/klipper
```
```
make menuconfig
```

#### 在配置菜單中選擇
```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```
#### 禁用所有不必要的功能
```
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

#### 保存並退出菜單。

#### 編譯固件
```
make clean
```
```
make
```

結果：

    Creating hex file out/klipper.bin

### 在 iHeater 板卡上安裝固件

!!! note "可能需要安裝 python3-serial"
    
    sudo apt install python3-serial

**以下考慮已安裝 Katapult 引導程式的安裝方式**

- 將 iHeater 連接到主機的編程模式（按住 Mode 按鈕時連接或按兩次 RESET）。

- 執行搜索
    ```
    ls /dev/serial/by-id/
    ```
    結果應如下所示：

    ```
    usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
    ```

    - 如有必要，安裝 flashtool

    ```
    pip install flashtool
    ```

- 更改為您的 ID 並輸入：
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin

    結果：

        Flashing '/home/pi/klipper/out/klipper.bin'...

        [##################################################]
        
        Write complete: 20 pages
        
        Verifying (block count = 319)...
        
        [##################################################]
        
        Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18
        
        Flash Success

- 檢查：
    ```        
    ls /dev/serial/by-id/
    ```
    結果：

        usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00

    ```iHeater 已准備好與 Klipper 協同工作```

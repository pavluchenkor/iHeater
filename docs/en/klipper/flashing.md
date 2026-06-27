# Firmware

This document provides instructions for building and flashing the **Katapult** bootloader for the **iHeater** microcontroller. The Katapult bootloader enables flashing Klipper firmware via USB and includes guidance on flashing **Klipper** firmware onto the **iHeater** controller.

---

## Requirements

* STM32F042F6P6
* iHeater board
* USB cable
* Linux system (e.g., Raspberry Pi or 3D printer)

!!! warning "If you cannot build and flash firmware on the printer"
    [Refer to the WSL section](../user-mods/software/wsl2-ubuntu-ff/)

---

## Building Katapult

1. Clone the Katapult repository:

```bash
git clone https://github.com/Arksine/katapult.git
cd katapult
make menuconfig
```

2. In `menuconfig`, select:

<!-- - MCU Architecture: STM32
- Processor model: STM32F042
- Clock Reference: Internal
- Communication interface: USB (on PA9/PA10)
- Application start offset: **8KiB offset**
- [x] Support bootloader entry on rapid double click of reset button
- [x] Enable bootloader entry on button (or gpio) state
- (!PA4)  Button GPIO Pin
- [X] Enable Status LED
- (PA5)   Status LED GPIO Pin -->

![menuconfig](../../img/katapult_menuconfig.jpg)

3. Build the firmware:

```bash
make
```

The firmware will be created in `out/katapult.bin`.

---

## Flashing Katapult via DFU

> This step is only needed once to install Katapult.

### Preparation:

Install `dfu-util` if it is not already installed:

```bash
sudo apt install dfu-util
```

According to the board version:

=== "r1"

    Set the jumper to BOOT0 and power cycle the board (or press the RESET button).
    The microcontroller will boot into DFU mode.

=== "r1.1"

    Press and hold BOOT, then power cycle the board (or press the RESET button), and release BOOT.
    The microcontroller will boot into DFU mode.


Check the connection:

```bash
lsusb
```

You should see:

```
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Flash Katapult:

Put it in DFU mode.

Run the command:

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Example of successful flashing:

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Exit DFU mode.

After reboot:

```bash
ls /dev/serial/by-id/*
```

You should see:

```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

If there are permission issues when flashing, run:

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

!!! warning "if something went wrong"
    erase the memory of the MCU and repeat the previous steps
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### Notes

* Katapult uses the first 8 KiB of flash, so **set 8 KiB bootloader offset in Klipper**.
* DFU mode can be entered using either a double Reset or a button on GPIO (PA4).
* If PA13/PA14 are used for SWD.
* Once Katapult is flashed, ST-Link is no longer needed - all further work is done via USB.

---

## Flashing Klipper to iHeater

### Compile the firmware:

```bash
cd ~/klipper
```

```
make menuconfig
```

#### In the menu, select:

```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### Disable all unnecessary options:

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

#### Save and exit the menu.

#### Build the firmware:

```bash
make clean
make
```

!!! Result should look like:
    Creating hex file out/klipper.bin


### Flash firmware to the iHeater board

!!! note "Install python3-serial if needed"
    sudo apt install python3-serial


**The following assumes Katapult bootloader is already installed**

* Connect the iHeater to the host in programming mode (hold the MODE button during connection or double-press RESET).

* List serial devices:

```bash
ls /dev/serial/by-id/
```

Example output:

```
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* Install flashtool if needed:

```bash
pip install flashtool
```

* Replace the ID with your actual ID and run:

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

Expected output:

```
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...

[##################################################]

Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

Flash Success
```

* Verify:

```bash
ls /dev/serial/by-id/
```

Expected output:

```
usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

`iHeater is ready to work with Klipper`


# Katapult Bootloader for STM32F042F6P6

This document contains instructions for building and flashing the **Katapult** bootloader from Klipper for the **STM32F042F6P6** microcontroller. The Katapult bootloader allows flashing Klipper firmware via USB without using an ST-Link.

---

## 🧰 Requirements

- STM32F042F6P6
- iHeater board
- ST-Link V2 programmer (for the initial flashing) or USB cable
- Linux system (e.g. Raspberry Pi or printer)

---

## ⚙️ Building Katapult

1. Clone the Katapult repository:

```bash
git clone https://github.com/Arksine/katapult
cd katapult
make menuconfig
```

2. In `menuconfig`, select:

- MCU Architecture: STM32  
- Processor model: STM32F042  
- Clock Reference: Internal  
- Communication interface: USB (on PA9/PA10)  
- Application start offset: **8KiB offset**  
- [x] Support bootloader entry on rapid double click of reset button  
- [x] Enable bootloader entry on button (or GPIO) state  
- (!PA4)  Button GPIO Pin  
- [X] Enable Status LED  
- (PA5)   Status LED GPIO Pin

![menuconfig](imgweb/katapult_menuconfig.jpg)

3. Build:

```bash
make
```

The firmware will be created at `out/katapult.bin`.

---

## 🔌 Flashing Katapult via ST-Link

> This step is required only once, to flash Katapult initially.

### Install the `st-flash` utility:

```bash
sudo apt install stlink-tools
```

### ST-Link connection:

| ST-Link | STM32     |
|---------|-----------|
| SWDIO   | PA13      |
| SWCLK   | PA14      |
| GND     | GND       |
| 3.3V    | VDD       |

### Flashing:

```bash
sudo st-flash write out/katapult.bin 0x08000000
```

If successful, you’ll see the message:

```
Flash written and verified! jolly good!
```

---

## Flashing Katapult via DFU

> This step is required only once, to flash Katapult initially.

### Preparation:

Install the `dfu-util` utility if it’s not already installed:

```bash
sudo apt install dfu-util
```

Set a jumper on BOOT0 and restart the board (or press the RESET button).  
The microcontroller will boot into DFU mode.

Check connection:

```bash
lsusb
```

You should see:

```
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Flashing Katapult:

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

Remove the jumper, hold the MODE button and press and release the RESET button (or reconnect the USB cable).

After rebooting, the board should appear as:

```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

If you do not have rights, there may be errors during flashing, to get access, run the command:
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

## 📋 Notes

- Katapult occupies the first 8 KB of flash, so **Klipper must be configured with an 8 KiB offset**.
- You can use either a double reset or a GPIO button (PA4) to enter DFU mode.
- If PA13/PA14 are used for SWD
- After flashing Katapult, you no longer need the ST-Link — all future updates can be done via USB.

# Snapmaker U1

This guide covers building and installing the iHeater firmware for the **Snapmaker U1** printer.

It differs from a standard installation in three ways:

1. The firmware is built **from Snapmaker's Klipper fork** ([u1-klipper](https://github.com/Snapmaker/u1-klipper)), not from mainline Klipper. Snapmaker changed about 20% of the Klipper code, so the MCU firmware must be built from the same version that runs on the printer.
2. In the Snapmaker fork, the STM32F042 build with USB **does not compile without a small fix**. The fix is given below.
3. The firmware is built **on a computer**, not on the printer, and is flashed to the iHeater with **STM32CubeProgrammer** in DFU mode. The Katapult bootloader is **not used**.

The remaining setup (configuration, macros, calibration) is the same as for any Klipper printer: see [Configuration](configuration.md).

---

## Requirements

- An iHeater board (STM32F042F6P6) and a USB cable **with data lines**
- A Snapmaker U1 and SSH access to it (login `root`, password `snapmaker`)
- A computer to build the firmware:
    - **Windows** — WSL (Ubuntu inside Windows), the main option in this guide
    - **Linux** — the commands are the same, skip the WSL section
    - **macOS** — any virtual machine with Ubuntu (UTM, Parallels, VirtualBox), or build directly on macOS, whichever is more convenient. The workflow is the same, and the commands inside Ubuntu are identical.
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (free, Windows / macOS / Linux)

---

## 1. Check the printer firmware version

On the printer screen or in Fluidd (the system information section), find the firmware version, for example `1.4.1`. You will need it to choose the Klipper version.

---

## 2. Prepare Ubuntu

#### Windows (WSL)

Open **PowerShell as administrator** and run:

```powershell
wsl --install -d Ubuntu-24.04
```

Restart the computer if Windows asks you to. On the first launch of Ubuntu, set a username and password.

From here on, run all commands **in the Ubuntu window**.

#### macOS / Linux

Start a virtual machine with Ubuntu (on Linux, just open a terminal). From here on, run all commands there.

Install the build tools:

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. Clone Klipper for the U1

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### Choose the version

List the firmware versions available in the repository:

```bash
git log --oneline | grep "Release firmware"
```

Example:

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

Choose the commit that **matches the printer firmware version**. If there is no exact match (for example, the printer runs `1.4.1` and the list only has `1.4.0`), use the closest lower release in the same series: for `1.4.1`, use `1.4.0`.

Create a working branch from the chosen commit (substitute your own hash):

```bash
git switch -c iheater edf309af
```

!!! tip "Simplest option"
    Update the printer to the latest firmware and build from the latest commit. The versions are then guaranteed to match.

---

## 4. USB fix for the STM32F042

In the Snapmaker fork, a function that calls `usb_clear_stall()` was added to `src/generic/usb_cdc.c`. It is implemented only for the USB controller of higher-end STM32 parts (`src/stm32/usbotg.c`). The STM32F042 has a different USB controller (`src/stm32/usbfs.c`), so the build fails with:

```
undefined reference to `usb_clear_stall'
```

Add the missing function: copy the block below **in full** (from `grep` to the last `EOF` line inclusive), paste it into the Ubuntu terminal, and press **Enter**. The command appends the function to the end of `src/stm32/usbfs.c` automatically — no manual editing required.

```bash
grep -q usb_clear_stall src/stm32/usbfs.c || cat >> src/stm32/usbfs.c << 'EOF'

/****************************************************************
 * Snapmaker usb_cdc.c CLEAR_FEATURE(ENDPOINT_HALT) support
 ****************************************************************/

#include "board/irq.h" // irq_save

// Clear STALL condition on an endpoint (port of usbotg.c version)
void
usb_clear_stall(uint32_t ep, int is_in)
{
    irqstatus_t flag = irq_save();
    uint32_t epr = USB_EPR[ep];
    if (is_in) {
        if ((epr & USB_EPTX_STAT) == USB_EP_TX_STALL)
            USB_EPR[ep] = calc_epr_bits(epr, USB_EPTX_STAT, USB_EP_TX_NAK);
    } else {
        if ((epr & USB_EPRX_STAT) == USB_EP_RX_STALL)
            USB_EPR[ep] = calc_epr_bits(epr, USB_EPRX_STAT, USB_EP_RX_VALID);
    }
    irq_restore(flag);
}
EOF
```

Verify:

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

The output should include a line with `usb_clear_stall(uint32_t ep, int is_in)`.

!!! note
    The command starts with `grep -q ... ||`: if Snapmaker fixes this themselves in a newer version, the function will not be added a second time.

To keep the firmware version from being marked `dirty`, commit the change:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. Configure and build

```bash
make menuconfig
```

Select:

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    Katapult is not used, so the firmware is written to the very beginning of flash (`0x08000000`). If you leave `8KiB bootloader` selected, the board will not boot after flashing.

In the `Optional features` section, disable everything you don't need (the STM32F042 has only 32 KB of flash):

```
[*] Support micro-controller based ADC (analog to digital)
[ ] Support communicating with external chips via SPI bus
[ ] Support communicating with external chips via I2C bus
[ ] Support software based I2C "bit-banging"
[ ] Support software based SPI "bit-banging"
[*] Support GPIO based button reading
[ ] Support Trinamic stepper motor driver UART communication
[ ] Support 'neopixel' type LED control
[ ] Support measuring fan tachometer GPIO pins
```

Also disable the remaining items (sensors, displays, external ADCs) if they appear in the list.

Save (`Q` → `Y`) and build:

```bash
make clean
```
```bash
make
```

Result:

```
  Linking out/klipper.elf
  Creating hex file out/klipper.bin
```

The build takes only a few seconds — that's normal.

Check the size (optional):

```bash
arm-none-eabi-size out/klipper.elf
```

The sum of `text` + `data` must be less than 32768 bytes.

!!! tip "Save the settings"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    Next time: `cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. Copy the firmware to your computer

#### Windows (WSL)

Open the firmware folder in Windows Explorer:

```bash
explorer.exe out
```

Or copy the file to your desktop (replace `YourUserName` with your Windows username):

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### Virtual machine

Use the virtual machine's shared folder if you have one set up. Otherwise, use SSH. On the host computer, run:

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

If SSH isn't available, use a temporary web server. In the virtual machine, run:

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

Open `http://VM_IP:8000` in a browser and download `klipper.bin`. Press `Ctrl+C` to stop the server.

To find the virtual machine's IP address: `ip -4 addr | grep inet`

---

## 7. Flash with STM32CubeProgrammer (DFU)

### Enter DFU mode

Depending on the board revision, DFU mode is entered using a **jumper** or a **button**:

- **BOOT0 jumper** — fit the jumper and connect the board to the computer via USB (if the board is already connected, press RESET). Remember to remove the jumper after flashing.
- **BOOT button** — press and hold the button, connect the board to the computer via USB (or press RESET), then release BOOT.

### Flashing

1. Start STM32CubeProgrammer.
2. On the right, select the **USB** interface and click the refresh button — `USB1` will appear in the **Port** field.
3. Click **Connect**.
4. On the left, open the **Erasing & Programming** tab.
5. **File path** — select `klipper.bin`.
6. **Start address** — `0x08000000`.
7. Check **Verify programming**.
8. Click **Start Programming**. The message `File download complete` should appear.
9. Click **Disconnect**, remove the BOOT0 jumper (if your board has one), and reconnect the board.

!!! note "Windows: board not detected in CubeProgrammer"
    The DFU driver is installed together with STM32CubeProgrammer. If the board appears in Device Manager as `STM32 BOOTLOADER` with a yellow icon, reinstall the driver from the CubeProgrammer folder: `Drivers\DFU_Driver`.

!!! note "If the board previously had Katapult"
    It will be overwritten, which is expected: the firmware is built without a bootloader and occupies flash starting from the very beginning.

---

## 8. Connect to the U1

Connect the iHeater to the printer's USB port and log in to the printer over SSH (login `root`, password `snapmaker`):

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    This appears after the printer is reflashed or its IP address changes. Remove the old key and reconnect:
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

Find the board ID:

```bash
ls /dev/serial/by-id/
```

Result:

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. Configuration

From here on, everything is the same as with standard Klipper — see [Configuration](configuration.md):

1. Place `iHeater.cfg` next to `printer.cfg` (using the configuration editor in Fluidd or over SSH).
2. Add the following to `printer.cfg`:
    ```ini
    [include iHeater.cfg]
    ```
3. Specify the board ID in `iHeater.cfg`:
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Restart Klipper (`FIRMWARE_RESTART`).

!!! warning "Updating the printer firmware"
    After updating the U1 firmware, check that `[include iHeater.cfg]` is still in `printer.cfg`. If the Klipper version on the printer has changed, rebuild the iHeater firmware from the matching commit (steps 3–7).

---

## Troubleshooting

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
The iHeater firmware version does not match the Klipper version on the printer. Rebuild from the commit that matches the U1 firmware version (step 3).

**The board does not appear in `/dev/serial/by-id/`**

- The firmware was built with `8KiB bootloader` — rebuild with `No bootloader`.
- The BOOT0 jumper was not removed — the board stays in DFU mode.
- The USB cable has no data lines.
- Check whether the printer sees the device at all: `dmesg | tail -20`.

**`undefined reference to 'usb_clear_stall'` during the build**
Step 4 was skipped.

**`region FLASH overflowed` during the build**
The firmware does not fit in flash: disable unnecessary items in `Optional features` (step 5).

**`gcode command M141 already registered`**
A macro with the same name is already defined in another printer config. Comment out one of them.

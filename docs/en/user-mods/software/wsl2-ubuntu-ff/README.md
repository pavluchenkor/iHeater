# How to Flash a Device on Windows 10/11 Using WSL2 with Ubuntu 20.04

## Steps 1-10 are suitable for flashing:

* Standalone Firmware
* Katapult Bootloader (for later flashing of the Klipper runtime from the printer host)
* Klipper runtime without using Katapult

It is assumed the firmware is already compiled and located on the WSL guest system at `~/iHeater/firmware.bin`

> [!NOTE] At the beginning of each step, brackets indicate where to perform the action: [WIN] - in native Windows, [WSL] - in the guest OS terminal (Ubuntu)

1. **[WIN]** Follow [this guide](https://learn.microsoft.com/en-us/windows/wsl/connect-usb) to install USBIPD-WIN from the [releases page](https://github.com/dorssel/usbipd-win/releases)

2. **[WSL]** Ensure WSL is active (simply start a command-line session in the guest OS)

3. **[WIN]** Connect the device to the PC (ensure the BOOT jumper is installed during connection)

   > [!NOTE] Note that a clean chip (never previously flashed) automatically enters DFU mode without a jumper.

4. **[WIN]** Open Windows PowerShell **as administrator** and enter:

    ```
    > usbipd list
    BUSID  VID:PID    DEVICE                                                        STATE
    5-2    0483:df11  STM32  BOOTLOADER                                             Not shared
    ```

   Remember the BUSID value

5. **[WIN]** Enter the command with the recorded BUSID

   > [!NOTE] This command is necessary if the last column shows `Not shared` (e.g., after flashing Klipper)

   ```
   > usbipd bind --busid 5-2
   ```

   Repeat the command from step 4 to confirm the STATE column shows "Shared"

   ```
   > usbipd list
   Connected:
   BUSID  VID:PID    DEVICE                                                        STATE
   5-2    0483:df11  STM32  BOOTLOADER                                             Shared
   ```

6. **[WIN]** Attach the device to the guest OS (replace `busid` with your actual value):

   ```
   > usbipd attach --wsl --busid 5-2
   ```

   > [!NOTE] This command must be run each time the device is reconnected/rebooted, but can be executed from PowerShell without administrator rights

   The device will then "disconnect" from Windows and attach to the guest OS.

   Verify the device is attached using the command from step 4 (STATE should be `Attached`)

   ```
   > usbipd list
   Connected:
   BUSID  VID:PID    DEVICE                                                        STATE
   5-2    0483:df11  STM32  BOOTLOADER                                             Attached
   ```

7. **[WSL]** In the guest OS, verify device presence with:

   ```
   > lsusb
   Bus 001 Device 006: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
   ```

8. **[WSL]** Install the utility in the guest OS

   ```
   > sudo apt install dfu-util
   ```

9. **[WSL]** Ensure the board is in DFU mode and visible to dfu-util:

   ```
   > sudo dfu-util --list
   ```

10. **[WSL]** Flash the firmware

    ```
    > sudo dfu-util -a 0 -s 0x08000000 -D ~/iHeater/firmware.bin
    ```

## Further Steps - If You Wish to Compile and Flash the Klipper Runtime from WSL (Instead of the Printer Host)

Assuming the firmware is already compiled (as per the official guide) and located at `~/iHeater/klipper.bin`

11. **[WIN]** Reconnect the device (unplug/plug USB) while holding the MODE button. LED3 should start blinking slowly, indicating Katapult is ready to receive the firmware. Release the button.

12. **[WIN]+** **[WSL]** Repeat steps 4-7 (step 5 can likely be skipped—see note therein). The command in step 7 should now show:

    ```
    > lsusb
    Bus 001 Device 005: ID 1d50:6177 OpenMoko, Inc. stm32f042x6
    ```

    This indicates the device name has changed.

13. **[WSL]** The command `ls /dev/serial/by-id/` may return nothing, since the connected board appears as a standard USB-CDC-ACM device.

    To verify, use:

    ```
    > dmesg
    ```

    **Note - In the example, the last line shows "ttyACM0" - this is the device "address".**

14. **[WSL]** Flash the device (with sudo). Ensure paths are correct:

    ```
    > sudo python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM0 -b 250000 -f ~/iHeater/klipper.bin
    ```

That’s it - you can now connect the board to the printer and proceed with configuration per the documentation.

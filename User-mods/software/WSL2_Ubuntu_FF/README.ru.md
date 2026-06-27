# WSL

## Как прошить устройство из под Windows 10/11 используя WSL2 с работающей на ней Ubuntu 20.04

## Пункты 1-10 подходят для прошивки:
- Standalone Firmware
- Загрузчика Katapult (для последующей заливки управляемой части Klipper уже с хоста самого принтера)
- Управляемой части Klipper без использования Katapult

Предполагается что прошивка уже скомпилирована и находится на гостевой системе WSL по пути `~/iHeater/firmware.bin`

> [!NOTE] В начале каждого пункта в квадратных скобках указано где именно проводить указанные действия: [WIN] - в самом Windows, [WSL] - в терминале гостевой ОС (Ubuntu)

1. **[WIN]** Следуя вот [этой инструкции](https://learn.microsoft.com/en-us/windows/wsl/connect-usb) установить USBIPD-WIN со [страницы релизов](https://github.com/dorssel/usbipd-win/releases)

2. **[WSL]** Убедиться что WSL активен (достаточно просто запустить сеанс командной строки гостевой ОС)

3. **[WIN]** Подключаем устройство к ПК (убедитесь что при подключении установлена перемычка BOOT)

    > [!NOTE] Обратите внимание, что чистый чип (ранее не прошивавшийся) автоматически переходит в режим DFU без перемычки.

4. **[WIN]** Открываем Windows PowerShell **в режиме администратора** и вводим

    ```
    > usbipd list
    BUSID  VID:PID    DEVICE                                                        STATE
    5-2    0483:df11  STM32  BOOTLOADER                                             Not shared
    ```
    Запоминаем значение в колоке BUSID

5. **[WIN]** Вводим команду с подстановкой запомненного BUSID

    > [!NOTE] Эта команда нужна в том случае, если в последнем столбце появляется `Not shared` (например после прошивки клиппера)

    ```
    > usbipd bind --busid 5-2
    ```

    повторяем команду из п.4 чтобы убедиться что в столбце STATE для нашего устройства указано "Shared"

    ```
    > usbipd list
    Connected:
    BUSID  VID:PID    DEVICE                                                        STATE
    5-2    0483:df11  STM32  BOOTLOADER                                             Shared
    ```

6. **[WIN]** Прицепляем устройство в гостевую ОС (не забываем заменить значение для `busid` на свой):

    ```
    > usbipd attach --wsl --busid 5-2
    ```
    > [!NOTE]Данную команду данную комманду нужно вводить каждый раз, когда переподключаете/перезагружаете устройство, но делать это уже можно в Power Shell без прав администратора

    При этом устройство "отключиться" от Windows и подключится внутрь гостевой ОС.

    Проверяем что устройство прицепилось используя команду из п.4 (в столбце STATE должно быть написано `Attached`)

    ```
    > usbipd list
    Connected:
    BUSID  VID:PID    DEVICE                                                        STATE
    5-2    0483:df11  STM32  BOOTLOADER                                             Attached
    ```

7. **[WSL]** В гостевой ОС проверяем наличие устройства командой `lsusb`

    ```
    > lsusb
    Bus 001 Device 006: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
    ```

8. **[WSL]** Устанавливаем на гостевую ОС утилиту

    ```
    > sudo apt install dfu-util
    ```

9. **[WSL]** Убедитесь что подключенная плата в режиме DFU и dfu-util ее видит (в квадратных скобках будет идентификатор устройства, который было видно так же в пп.4-7)

    ```
    > sudo dfu-util --list

    dfu-util 0.9

    Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
    Copyright 2010-2016 Tormod Volden and Stefan Schmidt
    This program is Free Software and has ABSOLUTELY NO WARRANTY
    Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

    Found DFU: [0483:df11] ver=2200, devnum=10, cfg=1, intf=0, path="1-1", alt=1, name="@Option Bytes  /0x1FFFF800/01*016 e", serial="FFFFFFFEFFFF"
    Found DFU: [0483:df11] ver=2200, devnum=10, cfg=1, intf=0, path="1-1", alt=0, name="@Internal Flash  /0x08000000/032*0001Kg", serial="FFFFFFFEFFFF"
    ```

10. **[WSL]** Прошиваем

    ```
    > sudo dfu-util -a 0 -s 0x08000000 -D ~/iHeater/firmware.bin

    dfu-util 0.9

    Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
    Copyright 2010-2016 Tormod Volden and Stefan Schmidt
    This program is Free Software and has ABSOLUTELY NO WARRANTY
    Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

    dfu-util: Invalid DFU suffix signature
    dfu-util: A valid DFU suffix will be required in a future dfu-util release!!!
    Opening DFU capable USB device...
    ID 0483:df11
    Run-time device DFU version 011a
    Claiming USB DFU Interface...
    Setting Alternate Setting #0 ...
    Determining device status: state = dfuERROR, status = 10
    dfuERROR, clearing status
    Determining device status: state = dfuIDLE, status = 0
    dfuIDLE, continuing
    DFU mode device DFU version 011a
    Device returned transfer size 2048
    DfuSe interface name: "Internal Flash  "
    Downloading to address = 0x08000000, size = 24252
    Download        [=========================] 100%        24252 bytes
    Download done.
    File downloaded successfully
    ```

## Следующие шаги - если хочется скомпилировать и прошить управляемую часть Klipper также из под WSL (то есть не с хоста принтера).

Допустим вы скомпилировали уже прошивку (согласно инструкции в документации) и теперь она у вас лежит в `~/iHeater/klipper.bin`

11. **[WIN]** Переподключаем устройство к ПК (USB-шнурок отключить-поодключить) с зажатой кнопкой MODE .
При этом должен начать медленно мигать LED3 - это значит что Katapult перешла в режим ожидания получения прошивки (аналог DFU-режима) и кнопку можно отпустить.

12. **[WIN]**+**[WSL]** Повторяем пп.4-7 (п.5 скорее всего можно пропустить - см. в нем пояснения)
при этом команда из п.7 покажет что-то вроде:

    ```
    > lsusb
    Bus 001 Device 005: ID 1d50:6177 OpenMoko, Inc. stm32f042x6
    ```
    То есть название устройства изменилось.

13. **[WSL]** В WSL команда ls /dev/serial/by-id/ скорее всего не даст результата, так как подключенная плата появится там в виде стандартного устройства USB-CDC-ACM

    Проверить это можно введя команду
    ```
    > dmesg

    [53229.047396] vhci_hcd vhci_hcd.0: pdev(0) rhport(0) sockfd(3)
    [53229.048526] vhci_hcd vhci_hcd.0: devid(327682) speed(2) speed_str(full-speed)
    [53229.049243] vhci_hcd vhci_hcd.0: Device attached
    [53229.321619] vhci_hcd: vhci_device speed not set
    [53229.391684] usb 1-1: new full-speed USB device number 7 using vhci_hcd
    [53229.471650] vhci_hcd: vhci_device speed not set
    [53229.541609] usb 1-1: SetAddress Request (7) to port 0
    [53229.579674] usb 1-1: New USB device found, idVendor=1d50, idProduct=6177, bcdDevice= 1.00
    [53229.580696] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [53229.581508] usb 1-1: Product: stm32f042x6
    [53229.581982] usb 1-1: Manufacturer: katapult
    [53229.582912] usb 1-1: SerialNumber: 460024000E53304347373020
    [53229.586466] cdc_acm 1-1:1.0: ttyACM0: USB ACM device
    ```
    
    **Обратите внимание - в примере в последней строчке указан "ttyACM0" - это и есть "адрес" устройства.**

14. **[WSL]** Прошиваем устройство (под sudoб то есть правами суперпользователя)
не забываем проверить и при необходимости поправить пути

    ```
    > sudo python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM0 -b 250000 -f ~/iHeater/klipper.bin

    Connecting to Serial Device /dev/ttyACM0, baud 250000
    Detected USB device running Katapult
    Detected Klipper binary version v0.13.0-51-gbfda326c2, MCU: stm32f042x6
    Attempting to connect to bootloader
    Katapult Connected
    Software Version: v0.0.1-96-gbc1ecea
    Protocol Version: 1.1.0
    Block Size: 64 bytes
    Application Start: 0x8002000
    MCU type: stm32f042x6
    Flashing '/home/kekht/iHeater/klipper_vanilla/klipper.bin'...

    [##################################################]

    Write complete: 14 pages
    Verifying (block count = 213)...

    [##################################################]

    Verification Complete: SHA = 3C7153502764CB6C948E9FD7872F69B2D0CBED55
    Programming Complete
    ```

Вот и все, теперь можно подключать к принтеру и настраивать согласно документации.
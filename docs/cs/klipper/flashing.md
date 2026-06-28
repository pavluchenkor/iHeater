#  Flashování

Tento dokument obsahuje pokyny pro sestavení a flashování bootloaderu **Katapult** pro mikrokontrolér **iHeater**. Bootloader Katapult umožňuje flashovat firmware Klipper přes USB a obsahuje také dokumentaci k instalaci firmwaru **Klipper** na řadič **iHeater** 

---

## Co budete potřebovat

- STM32F042F6P6
- Deska iHeater
- USB kabel
- Linuxový systém (například Raspberry Pi nebo tiskárna)

!!! warning "Pokud není možné sestavit a flashovat firmware na tiskárně"
    [Přejděte do sekce WSL](../user-mods/software/wsl2-ubuntu-ff/)

---

## Sestavení Katapult

1. Naklonujte repozitář Katapult:

```bash
git clone https://github.com/Arksine/katapult.git
```
```
cd katapult
```
```
make menuconfig
```

2. V `menuconfig` vyberte:

![menuconfig](../../img/katapult_menuconfig.jpg)

3. Sestavení:

```bash
make
```

Firmware bude vytvořen v `out/katapult.bin`.


---

## Flashování Katapult přes DFU

> Tento krok je potřeba provést pouze jednou, pro nahrání samotného Katapult.

### Příprava:
Nainstalujte nástroj dfu-util, pokud ještě není nainstalovaný:
    
    sudo apt install dfu-util

Podle verze desky:

=== "r1"

    Nasaďte jumper na BOOT0 a restartujte napájení desky (nebo stiskněte tlačítko RESET).
    Mikrokontrolér se spustí v režimu DFU.

=== "r1.1"

    Stiskněte tlačítko BOOT a restartujte napájení desky (nebo stiskněte tlačítko RESET), poté BOOT uvolněte.
    Mikrokontrolér se spustí v režimu DFU.

Zkontrolujte připojení:

    lsusb

Výsledek:

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### Flashování Katapult:
Přepněte do režimu DFU.

Spusťte příkaz:

```
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Příklad úspěšného flashování:

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Ukončete režim DFU.

Po restartu 
```
ls /dev/serial/by-id/*
```
Výsledek:
```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Pokud nejsou dostatečná oprávnění, mohou při flashování vznikat chyby. Pro získání přístupu spusťte příkaz:
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

!!! warning "pokud se něco pokazilo"
    vymažte paměť MCU a zopakujte předchozí kroky
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### Poznámky

- Katapult zabírá prvních 8 KB Flash, proto je **v Klipper nutné zadat offset 8 KiB**.
- Pro vstup do DFU lze použít buď dvojitý Reset, nebo tlačítko na GPIO (PA4).
- Pokud se PA13/PA14 používají pro SWD
- Po flashování Katapult už není nutné používat ST-Link - veškerá další práce probíhá přes USB.

## Instalace firmwaru na iHeater

### Kompilace firmwaru
```
cd ~/klipper
```
```
make menuconfig
```

#### V konfiguračním menu vyberte
```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```
#### Vypněte vše nepotřebné
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

#### Uložte a opusťte menu.

#### Zkompilujte firmware
```
make clean
```
```
make
```

Výsledek:

    Creating hex file out/klipper.bin

### Instalace firmwaru na desku iHeater

!!! note "Může být potřeba nainstalovat python3-serial"
    
    sudo apt install python3-serial

**Dále je popsána varianta instalace s nainstalovaným bootloaderem Katapult**

- Připojte iHeater k hostiteli v režimu programování (podržte tlačítko Mode při připojení nebo dvakrát stiskněte RESET).

- Spusťte vyhledání 
    ```
    ls /dev/serial/by-id/
    ```
    Výsledek by měl vypadat takto:

    ```
    usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
    ```

    - V případě potřeby nainstalujte flashtool

    ```
    pip install flashtool
    ```

- Změňte na vlastní ID a zadejte:
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin

    Výsledek:

        Flashing '/home/pi/klipper/out/klipper.bin'...

        [##################################################]
        
        Write complete: 20 pages
        
        Verifying (block count = 319)...
        
        [##################################################]
        
        Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18
        
        Flash Success

- Zkontrolujte: 
    ```        
    ls /dev/serial/by-id/
    ```
    Výsledek:

        usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00

    ```iHeater je připraven k práci s Klipper```



# Firmware

Tento dokument popisuje sestavení a nahrání zavaděče **Katapult** pro mikrokontrolér **iHeater**. Katapult umožňuje aktualizovat firmware Klipper přes USB a používá se také při instalaci firmwaru **Klipper** do kontroléru **iHeater**.

---

## Požadavky

* STM32F042F6P6
* Deska iHeater
* Programátor ST-Link V2 pro prvotní nahrání firmwaru nebo USB kabel
* Systém Linux, například Raspberry Pi nebo tiskárna

!!! warning "Pokud se nedaří sestavit a nahrát firmware přímo na tiskárně"
    [Přejděte do sekce WSL](user-mods/software/wsl2-ubuntu-ff/)

---

## Sestavení Katapult

1. Naklonujte repozitář Katapult:

```bash
git clone https://github.com/Arksine/katapult.git
```

```bash
cd katapult
```

```bash
make menuconfig
```

2. V `menuconfig` vyberte:

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

3. Sestavte firmware:

```bash
make
```

Soubor firmwaru bude vytvořen v cestě `out/katapult.bin`.

---

## Nahrání Katapult přes DFU

> Tento krok je potřeba provést pouze jednou, aby se zapsal zavaděč Katapult.

### Příprava

Nainstalujte `dfu-util`, pokud ještě není nainstalovaný:

```bash
sudo apt install dfu-util
```

Nasaďte propojku BOOT0 a restartujte desku odpojením a připojením napájení nebo stiskněte RESET.
Mikrokontrolér přejde do režimu DFU.

Zkontrolujte připojení:

```bash
lsusb
```

Mělo by se objevit zařízení:

```text
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Nahrání Katapult

Spusťte:

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Příklad úspěšného výstupu:

```text
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Sejměte propojku, podržte tlačítko MODE, stiskněte a uvolněte RESET nebo znovu připojte USB.

Po restartu spusťte:

```bash
ls /dev/serial/by-id/*
```

Mělo by se objevit zařízení:

```bash
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Pokud se objeví chyby přístupových práv:

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

### Poznámky

* Katapult zabírá prvních 8 KB Flash, proto je **v Klipper nutné bezpodmínečně nastavit offset 8KiB**.
* Do DFU lze vstoupit dvojitým stisknutím RESET nebo tlačítkem GPIO (PA4).
* PA13/PA14 se používají pro SWD.
* Po nahrání Katapult již programátor ST-Link není potřeba; další aktualizace lze provádět přes USB.

## Instalace firmwaru na iHeater

### Sestavení firmwaru

```bash
cd klipper/
```

```bash
make menuconfig
```

#### V konfigurační nabídce vyberte:

```text
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### Vypněte všechny nepoužívané volby:

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

#### Uložte nastavení a ukončete nabídku.

#### Sestavte firmware

```bash
make clean
```

```bash
make
```

!!! note "Výsledkem by měl být soubor:"

```text
Creating hex file out/klipper.bin
```

### Instalace firmwaru na desku iHeater

!!! note "V případě potřeby nainstalujte python3-serial"

```bash
sudo apt install python3-serial
```

**Následující kroky předpokládají, že zavaděč Katapult je již nainstalován.**

* Připojte iHeater k hostiteli v režimu nahrávání firmwaru: při připojování USB držte MODE nebo dvakrát stiskněte RESET.

* Najděte zařízení:

```bash
ls /dev/serial/by-id/
```

Mělo by se objevit něco jako:

```text
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* V případě potřeby nainstalujte `flashtool`:

```bash
pip install flashtool
```

* Nahraďte ID zařízení vlastním a spusťte:

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

Očekávaný výstup:

```text
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...

[##################################################]

Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

Flash Success
```

* Zkontrolujte:

```bash
ls /dev/serial/by-id/
```

Výstup by měl vypadat přibližně takto:

```text
usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

`iHeater je připraven k práci s Klipper`

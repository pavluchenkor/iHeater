# Firmware

Dieses Dokument beschreibt den Build und das Flashen des **Katapult**-Bootloaders für den **iHeater**-Mikrocontroller. Katapult ermöglicht Firmware-Updates für Klipper über USB und wird außerdem bei der Installation der **Klipper**-Firmware auf dem **iHeater**-Controller verwendet.

---

## Anforderungen

* STM32F042F6P6
* iHeater-Platine
* ST-Link V2-Programmiergerät für das erste Flashen oder USB-Kabel
* Linux-System, zum Beispiel Raspberry Pi oder Drucker

!!! warning "Wenn sich die Firmware nicht direkt auf dem Drucker bauen und flashen lässt"
    [Siehe Abschnitt WSL](user-mods/software/wsl2-ubuntu-ff/)

---

## Katapult bauen

1. Klonen Sie das Katapult-Repository:

```bash
git clone https://github.com/Arksine/katapult.git
```

```bash
cd katapult
```

```bash
make menuconfig
```

2. Wählen Sie in `menuconfig`:

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

3. Bauen Sie die Firmware:

```bash
make
```

Die Firmware-Datei wird unter `out/katapult.bin` erstellt.

---

## Katapult über DFU flashen

> Dieser Schritt ist nur einmal erforderlich, um den Katapult-Bootloader zu schreiben.

### Vorbereitung

Installieren Sie `dfu-util`, falls es noch nicht installiert ist:

```bash
sudo apt install dfu-util
```

Setzen Sie den BOOT0-Jumper und starten Sie die Platine über die Stromversorgung neu oder drücken Sie RESET.
Der Mikrocontroller wechselt in den DFU-Modus.

Prüfen Sie die Verbindung:

```bash
lsusb
```

Das folgende Gerät sollte erscheinen:

```text
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Katapult flashen

Führen Sie aus:

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Beispiel einer erfolgreichen Ausgabe:

```text
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Entfernen Sie den Jumper, halten Sie die MODE-Taste gedrückt, drücken Sie RESET und lassen Sie RESET wieder los, oder verbinden Sie USB erneut.

Führen Sie nach dem Neustart aus:

```bash
ls /dev/serial/by-id/*
```

Das folgende Gerät sollte erscheinen:

```bash
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Falls Berechtigungsfehler auftreten:

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

### Hinweise

* Katapult belegt die ersten 8 KB Flash, daher muss **in Klipper zwingend der 8KiB-Offset angegeben werden**.
* Der DFU-Modus kann durch zweimaliges Drücken von RESET oder über die GPIO-Taste (PA4) aufgerufen werden.
* PA13/PA14 werden für SWD verwendet.
* Nach dem Flashen von Katapult wird das ST-Link-Programmiergerät nicht mehr benötigt; spätere Updates können über USB durchgeführt werden.

## Firmware auf iHeater installieren

### Firmware bauen

```bash
cd klipper/
```

```bash
make menuconfig
```

#### Wählen Sie im Konfigurationsmenü:

```text
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### Deaktivieren Sie alle nicht verwendeten Optionen:

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

#### Speichern Sie die Einstellungen und verlassen Sie das Menü.

#### Bauen Sie die Firmware

```bash
make clean
```

```bash
make
```

!!! note "Als Ergebnis sollte die folgende Datei erscheinen:"

```text
Creating hex file out/klipper.bin
```

### Firmware auf die iHeater-Platine installieren

!!! note "Installieren Sie bei Bedarf python3-serial"

```bash
sudo apt install python3-serial
```

**Die folgenden Schritte setzen voraus, dass der Katapult-Bootloader bereits installiert ist.**

* Verbinden Sie iHeater im Flash-Modus mit dem Host: Halten Sie MODE beim Anschließen von USB gedrückt oder drücken Sie zweimal RESET.

* Suchen Sie das Gerät:

```bash
ls /dev/serial/by-id/
```

Es sollte etwas Ähnliches erscheinen:

```text
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* Installieren Sie bei Bedarf `flashtool`:

```bash
pip install flashtool
```

* Ersetzen Sie die Geräte-ID durch Ihre eigene und führen Sie aus:

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

Erwartete Ausgabe:

```text
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...

[##################################################]

Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

Flash Success
```

* Prüfen Sie:

```bash
ls /dev/serial/by-id/
```

Die Ausgabe sollte ungefähr so aussehen:

```text
usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

`iHeater ist bereit für den Betrieb mit Klipper`

#  Flashen

Dieses Dokument enthält Anweisungen zum Erstellen und Flashen des Bootloaders **Katapult** für den Mikrocontroller **iHeater**. Der Bootloader Katapult ermöglicht das Flashen der Klipper-Firmware über USB und enthält außerdem Dokumentation zur Installation der Firmware **Klipper** auf dem Controller **iHeater** 

---

## Was benötigt wird

- STM32F042F6P6
- iHeater-Platine
- USB-Kabel
- Linux-System (zum Beispiel Raspberry Pi oder Drucker)

!!! warning "Wenn es nicht möglich ist, die Firmware auf dem Drucker zu erstellen und zu flashen"
    [Siehe Abschnitt WSL](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

---

## Katapult erstellen

1. Das Katapult-Repository klonen:

```bash
git clone https://github.com/Arksine/katapult.git
```
```
cd katapult
```
```
make menuconfig
```

2. In `menuconfig` auswählen:

![menuconfig](../../img/katapult_menuconfig.jpg)

3. Erstellung:

```bash
make
```

Die Firmware wird in `out/katapult.bin` erstellt.


---

## Katapult über DFU flashen

> Dieser Schritt ist nur einmal erforderlich, um Katapult selbst zu laden.

### Vorbereitung:
Die Utility dfu-util installieren, falls sie noch nicht installiert ist:
    
    sudo apt install dfu-util

Entsprechend der Platinenversion:

=== "r1"

    Den Jumper auf BOOT0 setzen und die Stromversorgung der Platine neu starten (oder die Taste RESET drücken).
    Der Mikrocontroller startet im DFU-Modus.

=== "r1.1"

    Die Taste BOOT drücken, die Stromversorgung der Platine neu starten (oder die Taste RESET drücken) und BOOT loslassen.
    Der Mikrocontroller startet im DFU-Modus.

Verbindung prüfen:

    lsusb

Ergebnis:

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### Katapult flashen:
In den DFU-Modus wechseln.

Den Befehl ausführen:

```
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Beispiel für erfolgreiches Flashen:

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Den DFU-Modus verlassen.

Nach dem Neustart 
```
ls /dev/serial/by-id/*
```
Ergebnis:
```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Wenn keine Berechtigungen vorhanden sind, können beim Flashen Fehler auftreten. Um Zugriff zu erhalten, den Befehl ausführen:
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

!!! warning "Wenn etwas schiefgelaufen ist"
    Löschen Sie den Speicher des Mikrocontrollers und wiederholen Sie die vorherigen Schritte
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### Hinweise

- Katapult belegt die ersten 8 KB Flash, daher muss **in Klipper unbedingt ein Offset von 8 KiB angegeben werden**.
- Es kann entweder ein doppelter Reset oder die Taste an GPIO (PA4) verwendet werden, um in DFU zu wechseln.
- Wenn PA13/PA14 für SWD verwendet werden
- Nach dem Flashen von Katapult muss ST-Link nicht mehr verwendet werden - alle weiteren Arbeiten erfolgen über USB.

## Firmware auf iHeater installieren

### Firmware kompilieren
```
cd ~/klipper
```
```
make menuconfig
```

#### Im Konfigurationsmenü auswählen
```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```
#### Alles Überflüssige deaktivieren
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

#### Speichern und das Menü verlassen.

#### Firmware kompilieren
```
make clean
```
```
make
```

Ergebnis:

    Creating hex file out/klipper.bin

### Firmware auf der iHeater-Platine installieren

!!! note "Möglicherweise muss python3-serial installiert werden"
    
    sudo apt install python3-serial

**Im Folgenden wird die Installationsvariante mit installiertem Katapult-Bootloader betrachtet**

- iHeater im Programmiermodus mit dem Host verbinden (die Taste Mode beim Anschließen gedrückt halten oder RESET zweimal drücken).

- Suche ausführen 
    ```
    ls /dev/serial/by-id/
    ```
    Das Ergebnis sollte so aussehen:

    ```
    usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
    ```

    - Bei Bedarf flashtool installieren

    ```
    pip install flashtool
    ```

- Durch die eigene ID ersetzen und eingeben:
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin

    Ergebnis:

        Flashing '/home/pi/klipper/out/klipper.bin'...

        [##################################################]
        
        Write complete: 20 pages
        
        Verifying (block count = 319)...
        
        [##################################################]
        
        Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18
        
        Flash Success

- Prüfen: 
    ```        
    ls /dev/serial/by-id/
    ```
    Ergebnis:

        usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00

    ```iHeater ist bereit für die Arbeit mit Klipper```



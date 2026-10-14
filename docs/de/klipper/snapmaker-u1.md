# Snapmaker U1

Anleitung zum Kompilieren und Installieren der iHeater-Firmware für den Drucker **Snapmaker U1**.

Sie unterscheidet sich in drei Punkten von der Standardinstallation:

1. Die Firmware wird **aus dem Klipper-Fork von Snapmaker** ([u1-klipper](https://github.com/Snapmaker/u1-klipper)) gebaut, nicht aus dem offiziellen Klipper. Snapmaker hat rund 20 % des Klipper-Codes geändert, daher muss die MCU-Firmware aus derselben Version stammen wie die Firmware auf dem Drucker.
2. Im Snapmaker-Fork lässt sich die Firmware für den STM32F042 mit USB **ohne eine kleine Korrektur nicht bauen**. Die Korrektur finden Sie weiter unten.
3. Die Firmware wird **auf dem Computer** gebaut, nicht auf dem Drucker, und mit **STM32CubeProgrammer** im DFU-Modus auf den iHeater geflasht. Der Katapult-Bootloader wird **nicht verwendet**.

Die weitere Einrichtung (Konfiguration, Makros, Kalibrierung) erfolgt wie bei jedem Klipper-Drucker: siehe [Konfiguration](configuration.md).

---

## Voraussetzungen

- iHeater-Platine (STM32F042F6P6) und ein USB-Kabel **mit Datenleitungen**
- Snapmaker U1 und SSH-Zugriff darauf (Login `root`, Passwort `snapmaker`)
- Computer zum Bauen der Firmware:
    - **Windows** – WSL (Ubuntu unter Windows), der in dieser Anleitung beschriebene Hauptweg
    - **Linux** – die Befehle sind identisch, der WSL-Abschnitt entfällt
    - **macOS** – eine beliebige virtuelle Maschine mit Ubuntu (UTM, Parallels, VirtualBox) oder ein Build direkt unter macOS, ganz wie es Ihnen lieber ist. Das Vorgehen ist dasselbe, die Befehle innerhalb von Ubuntu unterscheiden sich nicht.
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (kostenlos, Windows / macOS / Linux)

---

## 1. Firmware-Version des Druckers ermitteln

Ermitteln Sie auf dem Display des Druckers oder in Fluidd (im Bereich mit den Systeminformationen) die Firmware-Version, zum Beispiel `1.4.1`. Sie benötigen sie, um die passende Klipper-Version auszuwählen.

---

## 2. Ubuntu vorbereiten

#### Windows (WSL)

Öffnen Sie **PowerShell als Administrator** und führen Sie folgenden Befehl aus:

```powershell
wsl --install -d Ubuntu-24.04
```

Starten Sie den Computer neu, falls Windows Sie dazu auffordert. Legen Sie beim ersten Start von Ubuntu einen Benutzernamen und ein Passwort fest.

Alle weiteren Befehle werden **im Ubuntu-Fenster** ausgeführt.

#### macOS / Linux

Starten Sie die virtuelle Maschine mit Ubuntu (unter Linux genügt ein Terminal). Alle weiteren Befehle werden darin ausgeführt.

Installieren Sie die Build-Werkzeuge:

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. Klipper für U1 klonen

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### Version auswählen

Lassen Sie sich anzeigen, welche Firmware-Versionen im Repository vorhanden sind:

```bash
git log --oneline | grep "Release firmware"
```

Beispiel:

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

Wählen Sie den Commit, der **der Firmware-Version des Druckers entspricht**. Gibt es die exakte Version nicht (zum Beispiel `1.4.1` auf dem Drucker, in der Liste aber nur `1.4.0`), nehmen Sie die nächstniedrigere Version derselben Versionsreihe: für `1.4.1` also `1.4.0`.

Erstellen Sie ausgehend vom gewählten Commit einen Arbeits-Branch (setzen Sie Ihren eigenen Hash ein):

```bash
git switch -c iheater edf309af
```

!!! tip "Die einfachste Variante"
    Aktualisieren Sie den Drucker auf die neueste Firmware und bauen Sie die iHeater-Firmware aus dem neuesten Commit. Dann stimmen die Versionen garantiert überein.

---

## 4. USB-Korrektur für STM32F042

Im Snapmaker-Fork wurde der Datei `src/generic/usb_cdc.c` eine Funktion hinzugefügt, die `usb_clear_stall()` aufruft. Implementiert ist diese nur für den USB-Controller der größeren STM32-Modelle (`src/stm32/usbotg.c`). Der STM32F042 hat einen anderen USB-Controller (`src/stm32/usbfs.c`), daher bricht der Build mit folgendem Fehler ab:

```
undefined reference to `usb_clear_stall'
```

Fügen Sie die fehlende Funktion hinzu: Kopieren Sie den folgenden Block **vollständig** (von `grep` bis einschließlich der letzten Zeile `EOF`), fügen Sie ihn in das Ubuntu-Terminal ein und drücken Sie **Enter**. Der Befehl hängt die Funktion automatisch an das Ende der Datei `src/stm32/usbfs.c` an; manuelles Bearbeiten ist nicht nötig.

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

Prüfen Sie das Ergebnis:

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

Die Ausgabe sollte eine Zeile mit `usb_clear_stall(uint32_t ep, int is_in)` enthalten.

!!! note
    Der Befehl beginnt mit `grep -q ... ||`: Sollte Snapmaker das Problem in einer neueren Version selbst beheben, wird die Funktion nicht ein zweites Mal hinzugefügt.

Damit die Firmware-Version nicht den Zusatz `dirty` erhält, committen Sie die Änderung:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. Konfigurieren und Bauen

```bash
make menuconfig
```

Wählen Sie Folgendes aus:

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    Katapult wird nicht verwendet, die Firmware wird an den Anfang des Flash-Speichers (`0x08000000`) geschrieben. Bleibt `8KiB bootloader` eingestellt, startet die Platine nach dem Flashen nicht.

Deaktivieren Sie im Abschnitt `Optional features` alles Überflüssige (der STM32F042 hat nur 32 KB Flash-Speicher):

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

Deaktivieren Sie auch die übrigen Optionen (Sensoren, Displays, externe ADCs), sofern sie in der Liste vorhanden sind.

Speichern Sie (`Q` → `Y`) und bauen Sie:

```bash
make clean
```
```bash
make
```

Ergebnis:

```
  Linking out/klipper.elf
  Creating hex file out/klipper.bin
```

Der Build dauert einige Sekunden, das ist normal.

Größenprüfung (optional):

```bash
arm-none-eabi-size out/klipper.elf
```

Die Summe aus `text` + `data` muss unter 32768 liegen.

!!! tip "Einstellungen sichern"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    Beim nächsten Mal: `cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. Firmware auf den Computer kopieren

#### Windows (WSL)

Öffnen Sie den Ordner mit der Firmware im Windows-Explorer:

```bash
explorer.exe out
```

Oder kopieren Sie die Datei auf den Desktop (ersetzen Sie `YourUserName` durch Ihren Windows-Benutzernamen):

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### Virtuelle Maschine

Nutzen Sie einen gemeinsamen Ordner der virtuellen Maschine, sofern eingerichtet, andernfalls SSH. Führen Sie dazu auf dem Host-Rechner aus:

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

Ohne SSH geht es über einen temporären Webserver. In der virtuellen Maschine:

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

Öffnen Sie im Browser `http://VM_IP:8000` und laden Sie `klipper.bin` herunter. Beenden Sie den Server mit `Ctrl+C`.

IP der virtuellen Maschine: `ip -4 addr | grep inet`

---

## 7. Flashen über STM32CubeProgrammer (DFU)

### DFU-Modus aktivieren

Je nach Platinenrevision wird der DFU-Modus per **Jumper** oder per **Taster** aktiviert:

- **BOOT0-Jumper** – stecken Sie den Jumper und verbinden Sie die Platine per USB mit dem Computer (ist die Platine bereits angeschlossen, drücken Sie RESET). Nach dem Flashen muss der Jumper wieder entfernt werden.
- **BOOT-Taster** – halten Sie den Taster gedrückt, verbinden Sie die Platine per USB mit dem Computer (oder drücken Sie RESET) und lassen Sie BOOT anschließend los.

### Flashen

1. Starten Sie STM32CubeProgrammer.
2. Wählen Sie rechts die Schnittstelle **USB** und klicken Sie auf das Aktualisieren-Symbol – im Feld **Port** erscheint `USB1`.
3. Klicken Sie auf **Connect**.
4. Öffnen Sie links den Reiter **Erasing & Programming**.
5. **File path** – wählen Sie `klipper.bin` aus.
6. **Start address** – `0x08000000`.
7. Aktivieren Sie **Verify programming**.
8. Klicken Sie auf **Start Programming**. Es sollte die Meldung `File download complete` erscheinen.
9. Klicken Sie auf **Disconnect**, entfernen Sie den BOOT0-Jumper (falls Ihre Platine einen hat) und stecken Sie die Platine ab und wieder an.

!!! note "Windows: Platine wird in CubeProgrammer nicht angezeigt"
    Der DFU-Treiber wird zusammen mit STM32CubeProgrammer installiert. Wird die Platine im Geräte-Manager als `STM32 BOOTLOADER` mit gelbem Symbol angezeigt, installieren Sie den Treiber aus dem CubeProgrammer-Ordner neu: `Drivers\DFU_Driver`.

!!! note "Falls sich zuvor Katapult auf der Platine befand"
    Er wird überschrieben, das ist normal: Die Firmware wurde ohne Bootloader gebaut und belegt den Flash-Speicher ab dessen Anfang.

---

## 8. Mit dem U1 verbinden

Schließen Sie den iHeater an einen USB-Port des Druckers an und melden Sie sich per SSH am Drucker an (Login `root`, Passwort `snapmaker`):

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    Erscheint nach dem Neuflashen des Druckers oder einer Änderung der IP-Adresse. Entfernen Sie den alten Schlüssel und verbinden Sie sich erneut:
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

Ermitteln Sie die ID der Platine:

```bash
ls /dev/serial/by-id/
```

Ergebnis:

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. Konfiguration

Alles Weitere läuft wie bei einer regulären Klipper-Installation – siehe [Konfiguration](configuration.md):

1. Legen Sie `iHeater.cfg` neben `printer.cfg` ab (über den Konfigurationseditor in Fluidd oder per SSH).
2. Fügen Sie in `printer.cfg` hinzu:
    ```ini
    [include iHeater.cfg]
    ```
3. Tragen Sie in `iHeater.cfg` die ID der Platine ein:
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Starten Sie Klipper neu (`FIRMWARE_RESTART`).

!!! warning "Firmware-Update des Druckers"
    Prüfen Sie nach einem Update der U1-Firmware, ob `[include iHeater.cfg]` in `printer.cfg` erhalten geblieben ist. Hat sich die Klipper-Version auf dem Drucker geändert, bauen Sie die iHeater-Firmware aus dem passenden Commit neu (Schritte 3–7).

---

## Fehlerbehebung

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
Die Version der iHeater-Firmware passt nicht zur Klipper-Version auf dem Drucker. Bauen Sie die Firmware aus dem Commit neu, der der U1-Firmware-Version entspricht (Schritt 3).

**Die Platine erscheint nicht in `/dev/serial/by-id/`**

- Die Firmware wurde mit `8KiB bootloader` gebaut – bauen Sie sie mit `No bootloader` neu.
- Der BOOT0-Jumper wurde nicht entfernt – die Platine bleibt im DFU-Modus.
- USB-Kabel ohne Datenleitungen.
- Prüfen Sie, ob der Drucker das Gerät überhaupt erkennt: `dmesg | tail -20`.

**`undefined reference to 'usb_clear_stall'` beim Build**
Schritt 4 wurde nicht ausgeführt.

**`region FLASH overflowed` beim Build**
Die Firmware passt nicht in den Flash-Speicher: Deaktivieren Sie überflüssige Optionen unter `Optional features` (Schritt 5).

**`gcode command M141 already registered`**
Ein Makro mit demselben Namen ist bereits in einer anderen Konfigurationsdatei des Druckers definiert. Kommentieren Sie eines der beiden aus.

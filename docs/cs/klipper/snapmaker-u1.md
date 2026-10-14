# Snapmaker U1

Návod k sestavení a instalaci firmwaru iHeater pro tiskárnu **Snapmaker U1**.

Od běžné instalace se liší ve třech bodech:

1. Firmware se sestavuje **z forku Klipperu od společnosti Snapmaker** ([u1-klipper](https://github.com/Snapmaker/u1-klipper)), nikoli z hlavního Klipperu. Snapmaker upravil přibližně 20 % kódu Klipperu, proto musí být firmware MCU ze stejné verze jako firmware v tiskárně.
2. Ve forku Snapmaker nejde firmware pro STM32F042 s USB **bez drobné opravy sestavit**. Oprava je uvedena níže.
3. Firmware se sestavuje **na počítači**, nikoli v tiskárně, a do iHeater se nahrává přes **STM32CubeProgrammer** v režimu DFU. Bootloader Katapult se **nepoužívá**.

Další nastavení (konfigurace, makra, kalibrace) je stejné jako u jakékoli tiskárny s Klipperem: viz [Konfigurace](configuration.md).

---

## Co budete potřebovat

- Desku iHeater (STM32F042F6P6) a USB kabel **s datovými vodiči**
- Snapmaker U1 a přístup k němu přes SSH (login `root`, heslo `snapmaker`)
- Počítač pro sestavení firmwaru:
    - **Windows** — WSL (Ubuntu uvnitř Windows), hlavní varianta tohoto návodu
    - **Linux** — příkazy jsou stejné, část o WSL přeskočte
    - **macOS** — libovolný virtuální stroj s Ubuntu (UTM, Parallels, VirtualBox) nebo sestavení přímo v macOS, podle toho, co vám vyhovuje. Postup je stejný, příkazy uvnitř Ubuntu se neliší.
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (zdarma, Windows / macOS / Linux)

---

## 1. Zjistěte verzi firmwaru tiskárny

Na displeji tiskárny nebo ve Fluidd (sekce s informacemi o systému) najděte verzi firmwaru, například `1.4.1`. Budete ji potřebovat při výběru verze Klipperu.

---

## 2. Příprava Ubuntu

#### Windows (WSL)

Otevřete **PowerShell jako správce** a spusťte:

```powershell
wsl --install -d Ubuntu-24.04
```

Pokud vás k tomu Windows vyzvou, restartujte počítač. Při prvním spuštění Ubuntu si nastavte uživatelské jméno a heslo.

Všechny další příkazy zadávejte **v okně Ubuntu**.

#### macOS / Linux

Spusťte virtuální stroj s Ubuntu (na Linuxu stačí terminál). Všechny další příkazy zadávejte v něm.

Nainstalujte nástroje pro sestavení:

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. Klonování Klipperu pro U1

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### Výběr verze

Zjistěte, jaké verze firmwaru jsou v repozitáři:

```bash
git log --oneline | grep "Release firmware"
```

Příklad:

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

Vyberte commit, který **odpovídá verzi firmwaru tiskárny**. Pokud přesná verze neexistuje (například tiskárna má `1.4.1`, ale v seznamu je jen `1.4.0`), použijte nejbližší nižší verzi se stejným číslem: pro `1.4.1` je to `1.4.0`.

Vytvořte pracovní větev z vybraného commitu (dosaďte svůj hash):

```bash
git switch -c iheater edf309af
```

!!! tip "Nejjednodušší varianta"
    Aktualizujte tiskárnu na nejnovější firmware a sestavujte z posledního commitu. Verze se pak budou zaručeně shodovat.

---

## 4. Oprava USB pro STM32F042

Ve forku Snapmaker je do souboru `src/generic/usb_cdc.c` přidána funkce, která volá `usb_clear_stall()`. Je implementována pouze pro řadič USB vyšších řad STM32 (`src/stm32/usbotg.c`). STM32F042 má jiný řadič USB (`src/stm32/usbfs.c`) a sestavení skončí chybou:

```
undefined reference to `usb_clear_stall'
```

Přidejte chybějící funkci: zkopírujte blok níže **celý** (od `grep` po poslední řádek `EOF` včetně), vložte jej do terminálu Ubuntu a stiskněte **Enter**. Příkaz funkci sám připíše na konec souboru `src/stm32/usbfs.c`, nic není třeba upravovat ručně.

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

Kontrola:

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

Výstup by měl obsahovat řádek s `usb_clear_stall(uint32_t ep, int is_in)`.

!!! note
    Příkaz začíná `grep -q ... ||`: pokud to Snapmaker v nové verzi opraví sám, funkce se podruhé nepřidá.

Aby verze firmwaru neměla příznak `dirty`, uložte změnu commitem:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. Konfigurace a sestavení

```bash
make menuconfig
```

Vyberte:

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    Katapult se nepoužívá, firmware se zapisuje od samého začátku paměti flash (`0x08000000`). Pokud ponecháte `8KiB bootloader`, deska po nahrání firmwaru nenastartuje.

V sekci `Optional features` vypněte vše nadbytečné (STM32F042 má jen 32 KB paměti flash):

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

Ostatní položky (senzory, displeje, externí ADC) také vypněte, pokud jsou v seznamu.

Uložte (`Q` → `Y`) a sestavte:

```bash
make clean
```
```bash
make
```

Výsledek:

```
  Linking out/klipper.elf
  Creating hex file out/klipper.bin
```

Sestavení trvá jen několik sekund, to je normální.

Kontrola velikosti (nepovinná):

```bash
arm-none-eabi-size out/klipper.elf
```

Součet `text` + `data` musí být menší než 32768.

!!! tip "Uložte nastavení"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    Příště: `cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. Kopírování firmwaru do počítače

#### Windows (WSL)

Otevřete složku s firmwarem v Průzkumníku Windows:

```bash
explorer.exe out
```

Nebo zkopírujte soubor na plochu (nahraďte `YourUserName` svým uživatelským jménem ve Windows):

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### Virtuální stroj

Přes sdílenou složku virtuálního stroje, pokud je nastavena. Pokud ne, přes SSH. Na hlavním počítači spusťte:

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

Bez SSH můžete použít dočasný webový server. Ve virtuálním stroji spusťte:

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

Otevřete v prohlížeči `http://VM_IP:8000` a stáhněte `klipper.bin`. Server zastavíte klávesovou zkratkou `Ctrl+C`.

IP adresa virtuálního stroje: `ip -4 addr | grep inet`

---

## 7. Nahrání firmwaru přes STM32CubeProgrammer (DFU)

### Přepnutí do režimu DFU

Podle revize desky se režim DFU zapíná **propojkou** nebo **tlačítkem**:

- **Propojka BOOT0** — nasaďte propojku a připojte desku k počítači přes USB (pokud je deska již připojena, stiskněte RESET). Po nahrání firmwaru je nutné propojku odstranit.
- **Tlačítko BOOT** — stiskněte a držte tlačítko, připojte desku k počítači přes USB (nebo stiskněte RESET), poté tlačítko BOOT uvolněte.

### Zápis

1. Spusťte STM32CubeProgrammer.
2. Vpravo vyberte rozhraní **USB**, klikněte na tlačítko obnovení — v poli **Port** se objeví `USB1`.
3. Klikněte na **Connect**.
4. Vlevo otevřete záložku **Erasing & Programming**.
5. **File path** — vyberte `klipper.bin`.
6. **Start address** — `0x08000000`.
7. Zaškrtněte **Verify programming**.
8. Klikněte na **Start Programming**. Měla by se objevit zpráva `File download complete`.
9. Klikněte na **Disconnect**, odstraňte propojku BOOT0 (pokud ji vaše deska má) a znovu připojte desku.

!!! note "Windows: CubeProgrammer desku nevidí"
    Ovladač DFU se instaluje společně se STM32CubeProgrammer. Pokud se deska ve Správci zařízení zobrazuje jako `STM32 BOOTLOADER` se žlutou ikonou, přeinstalujte ovladač ze složky CubeProgrammer: `Drivers\DFU_Driver`.

!!! note "Pokud deska dříve měla Katapult"
    Bude přepsán, to je v pořádku: firmware je sestaven bez bootloaderu a zabírá flash od začátku.

---

## 8. Připojení k U1

Připojte iHeater k USB portu tiskárny a přihlaste se k tiskárně přes SSH (login `root`, heslo `snapmaker`):

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    Zobrazí se po přeflashování tiskárny nebo změně IP adresy. Odstraňte starý klíč a připojte se znovu:
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

Zjistěte ID desky:

```bash
ls /dev/serial/by-id/
```

Výsledek:

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. Konfigurace

Dál postupujte stejně jako u běžného Klipperu — viz [Konfigurace](configuration.md):

1. Umístěte `iHeater.cfg` vedle `printer.cfg` (přes editor konfigurace ve Fluidd nebo přes SSH).
2. Do `printer.cfg` přidejte:
    ```ini
    [include iHeater.cfg]
    ```
3. V `iHeater.cfg` uveďte ID desky:
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Restartujte Klipper (`FIRMWARE_RESTART`).

!!! warning "Aktualizace firmwaru tiskárny"
    Po aktualizaci firmwaru U1 zkontrolujte, zda `[include iHeater.cfg]` zůstal v `printer.cfg`. Pokud se verze Klipperu v tiskárně změnila, sestavte firmware iHeater znovu z odpovídajícího commitu (kroky 3–7).

---

## Problémy

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
Verze firmwaru iHeater se neshoduje s Klipperem v tiskárně. Sestavte firmware znovu z commitu odpovídajícího verzi firmwaru U1 (krok 3).

**Deska se neobjevuje v `/dev/serial/by-id/`**

- Firmware je sestaven s `8KiB bootloader` — sestavte znovu s `No bootloader`.
- Nebyla odstraněna propojka BOOT0 — deska zůstává v režimu DFU.
- USB kabel bez datových vodičů.
- Zkontrolujte, zda tiskárna zařízení vůbec vidí: `dmesg | tail -20`.

**`undefined reference to 'usb_clear_stall'` při sestavení**
Nebyl proveden krok 4.

**`region FLASH overflowed` při sestavení**
Firmware se nevejde do paměti flash: vypněte nadbytečné položky v `Optional features` (krok 5).

**`gcode command M141 already registered`**
Makro se stejným názvem je již definováno v jiném konfiguračním souboru tiskárny. Jedno z nich zakomentujte.

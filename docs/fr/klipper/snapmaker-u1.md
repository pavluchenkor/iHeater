# Snapmaker U1

Instructions de compilation et d'installation du micrologiciel iHeater pour l'imprimante **Snapmaker U1**.

Cette procédure diffère de l'installation standard sur trois points :

1. Le micrologiciel est compilé **à partir du fork Klipper de Snapmaker** ([u1-klipper](https://github.com/Snapmaker/u1-klipper)) et non du Klipper principal. Snapmaker a modifié environ 20 % du code de Klipper, le micrologiciel du MCU doit donc provenir de la même version que celle de l'imprimante.
2. Dans le fork Snapmaker, la compilation pour STM32F042 avec USB **échoue sans une petite correction**. Cette correction est fournie ci-dessous.
3. Le micrologiciel est compilé **sur l'ordinateur** et non sur l'imprimante, puis flashé dans l'iHeater via **STM32CubeProgrammer** en mode DFU. Le bootloader Katapult **n'est pas utilisé**.

Le reste du paramétrage (configuration, macros, calibration) est identique à celle de toute imprimante sous Klipper : voir [Configuration](configuration.md).

---

## Prérequis

- Carte iHeater (STM32F042F6P6) et câble USB **avec lignes de données**
- Snapmaker U1 et accès SSH à celle-ci (identifiant `root`, mot de passe `snapmaker`)
- Ordinateur pour compiler le micrologiciel :
    - **Windows** — WSL (Ubuntu dans Windows), option principale de ce guide
    - **Linux** — les commandes sont les mêmes, ignorez la section consacrée à WSL
    - **macOS** — n'importe quelle machine virtuelle Ubuntu (UTM, Parallels, VirtualBox) ou compilation directement sous macOS, au choix. La logique est la même, les commandes dans Ubuntu ne changent pas.
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (gratuit, Windows / macOS / Linux)

---

## 1. Déterminer la version du micrologiciel de l'imprimante

Sur l'écran de l'imprimante ou dans Fluidd (section d'informations système), repérez la version du micrologiciel, par exemple `1.4.1`. Elle servira à choisir la version de Klipper.

---

## 2. Préparation d'Ubuntu

#### Windows (WSL)

Ouvrez **PowerShell en tant qu'administrateur** et exécutez :

```powershell
wsl --install -d Ubuntu-24.04
```

Redémarrez l'ordinateur si Windows le demande. Au premier lancement d'Ubuntu, définissez un nom d'utilisateur et un mot de passe.

Toutes les commandes suivantes s'exécutent **dans la fenêtre Ubuntu**.

#### macOS / Linux

Lancez une machine virtuelle Ubuntu (sous Linux, un simple terminal suffit). Toutes les commandes suivantes s'y exécutent.

Installez les outils de compilation :

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. Clonage de Klipper pour U1

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### Choix de la version

Affichez les versions de micrologiciel présentes dans le dépôt :

```bash
git log --oneline | grep "Release firmware"
```

Exemple :

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

Choisissez le commit **correspondant à la version du micrologiciel de l'imprimante**. Si la version exacte n'existe pas (par exemple `1.4.1` sur l'imprimante alors que la liste ne contient que `1.4.0`), prenez la version inférieure la plus proche de la même série : pour `1.4.1`, prenez `1.4.0`.

Créez une branche de travail à partir du commit choisi (remplacez le hash par le vôtre) :

```bash
git switch -c iheater edf309af
```

!!! tip "Solution la plus simple"
    Mettez l'imprimante à jour avec le dernier micrologiciel et compilez à partir du dernier commit. Les versions correspondront alors à coup sûr.

---

## 4. Correction USB pour STM32F042

Dans le fork Snapmaker, le fichier `src/generic/usb_cdc.c` contient une fonction qui appelle `usb_clear_stall()`. Elle n'est implémentée que pour le contrôleur USB des STM32 haut de gamme (`src/stm32/usbotg.c`). Le STM32F042 possède un autre contrôleur USB (`src/stm32/usbfs.c`) et la compilation échoue avec l'erreur suivante :

```
undefined reference to `usb_clear_stall'
```

Ajoutez la fonction manquante : copiez le bloc ci-dessous **en entier** (de `grep` jusqu'à la dernière ligne `EOF` incluse), collez-le dans le terminal Ubuntu et appuyez sur **Entrée**. La commande ajoute elle-même la fonction à la fin du fichier `src/stm32/usbfs.c`, aucune modification manuelle n'est nécessaire.

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

Vérification :

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

La commande doit afficher une ligne contenant `usb_clear_stall(uint32_t ep, int is_in)`.

!!! note
    La commande commence par `grep -q ... ||` : si Snapmaker corrige ce problème lui-même dans une version ultérieure, la fonction ne sera pas ajoutée une seconde fois.

Pour éviter le suffixe `dirty` dans la version du micrologiciel, enregistrez la modification dans un commit :

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. Configuration et compilation

```bash
make menuconfig
```

Sélectionnez :

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    Katapult n'est pas utilisé, le micrologiciel est écrit dès le début de la mémoire flash (`0x08000000`). Avec `8KiB bootloader`, la carte ne démarrera pas après le flashage.

Dans la section `Optional features`, désactivez tout ce qui est superflu (le STM32F042 ne dispose que de 32 Ko de mémoire flash) :

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

Désactivez également les autres options (capteurs, écrans, CAN externes) si elles figurent dans la liste.

Enregistrez (`Q` → `Y`) et compilez :

```bash
make clean
```
```bash
make
```

Résultat :

```
  Linking out/klipper.elf
  Creating hex file out/klipper.bin
```

La compilation prend quelques secondes, c'est normal.

Vérification de la taille (facultative) :

```bash
arm-none-eabi-size out/klipper.elf
```

La somme `text` + `data` doit être inférieure à 32768.

!!! tip "Enregistrez les paramètres"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    La prochaine fois : `cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. Copie du micrologiciel sur l'ordinateur

#### Windows (WSL)

Ouvrez le dossier du micrologiciel dans l'explorateur Windows :

```bash
explorer.exe out
```

Ou copiez le fichier sur le bureau (remplacez `YourUserName` par votre nom d'utilisateur Windows) :

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### Machine virtuelle

Via le dossier partagé de la machine virtuelle, s'il est configuré. Sinon, via SSH. Sur l'ordinateur principal, exécutez :

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

Sans SSH, passez par un serveur web temporaire. Dans la machine virtuelle :

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

Ouvrez `http://VM_IP:8000` dans votre navigateur et téléchargez `klipper.bin`. Le serveur s'arrête avec `Ctrl+C`.

IP de la machine virtuelle : `ip -4 addr | grep inet`

---

## 7. Flashage via STM32CubeProgrammer (DFU)

### Entrée en mode DFU

Selon la révision de la carte, le mode DFU s'active par un **cavalier** ou par un **bouton** :

- **Cavalier BOOT0** — installez le cavalier et connectez la carte à l'ordinateur par USB (si la carte est déjà connectée, appuyez sur RESET). Après le flashage, retirez le cavalier.
- **Bouton BOOT** — maintenez le bouton enfoncé, connectez la carte à l'ordinateur par USB (ou appuyez sur RESET), puis relâchez BOOT.

### Programmation

1. Lancez STM32CubeProgrammer.
2. À droite, sélectionnez l'interface **USB**, cliquez sur le bouton d'actualisation : `USB1` apparaît dans le champ **Port**.
3. Cliquez sur **Connect**.
4. À gauche, ouvrez l'onglet **Erasing & Programming**.
5. **File path** — sélectionnez `klipper.bin`.
6. **Start address** — `0x08000000`.
7. Cochez **Verify programming**.
8. Cliquez sur **Start Programming**. Le message `File download complete` doit apparaître.
9. Cliquez sur **Disconnect**, retirez le cavalier BOOT0 (si votre carte en possède un) et reconnectez la carte.

!!! note "Windows : la carte n'est pas visible dans CubeProgrammer"
    Le pilote DFU est installé avec STM32CubeProgrammer. Si, dans le gestionnaire de périphériques, la carte apparaît comme `STM32 BOOTLOADER` avec une icône jaune, réinstallez le pilote depuis le dossier de CubeProgrammer : `Drivers\DFU_Driver`.

!!! note "Si Katapult était déjà installé sur la carte"
    Il sera écrasé, c'est normal : le micrologiciel est compilé sans bootloader et occupe la mémoire flash dès le début.

---

## 8. Connexion à l'U1

Branchez l'iHeater sur un port USB de l'imprimante et connectez-vous à celle-ci en SSH (identifiant `root`, mot de passe `snapmaker`) :

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    Ce message apparaît après un reflashage de l'imprimante ou un changement d'IP. Supprimez l'ancienne clé et reconnectez-vous :
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

Trouvez l'ID de la carte :

```bash
ls /dev/serial/by-id/
```

Résultat :

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. Configuration

La suite est identique à n'importe quelle installation Klipper standard — voir [Configuration](configuration.md) :

1. Placez `iHeater.cfg` à côté de `printer.cfg` (via l'éditeur de configuration de Fluidd ou par SSH).
2. Dans `printer.cfg`, ajoutez :
    ```ini
    [include iHeater.cfg]
    ```
3. Dans `iHeater.cfg`, indiquez l'ID de la carte :
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Redémarrez Klipper (`FIRMWARE_RESTART`).

!!! warning "Mise à jour du micrologiciel de l'imprimante"
    Après une mise à jour du micrologiciel de l'U1, vérifiez que `[include iHeater.cfg]` est toujours présent dans `printer.cfg`. Si la version de Klipper de l'imprimante a changé, recompilez le micrologiciel iHeater à partir du commit correspondant (étapes 3 à 7).

---

## Dépannage

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
La version du micrologiciel iHeater ne correspond pas à celle de Klipper sur l'imprimante. Recompilez à partir du commit correspondant à la version du micrologiciel de l'U1 (étape 3).

**La carte n'apparaît pas dans `/dev/serial/by-id/`**

- Le micrologiciel a été compilé avec `8KiB bootloader` — recompilez avec `No bootloader`.
- Le cavalier BOOT0 n'a pas été retiré — la carte reste en mode DFU.
- Câble USB sans lignes de données.
- Vérifiez que l'imprimante détecte bien le périphérique : `dmesg | tail -20`.

**`undefined reference to 'usb_clear_stall'` à la compilation**
L'étape 4 n'a pas été effectuée.

**`region FLASH overflowed` à la compilation**
Le micrologiciel ne tient pas dans la mémoire flash : désactivez les options superflues dans `Optional features` (étape 5).

**`gcode command M141 already registered`**
Une macro portant le même nom est déjà définie dans un autre fichier de configuration de l'imprimante. Commentez l'une des deux.

# Firmware

Ce document décrit la compilation et le flashage du bootloader **Katapult** pour le microcontrôleur **iHeater**. Katapult permet de mettre à jour le firmware Klipper via USB et est également utilisé lors de l'installation du firmware **Klipper** sur le contrôleur **iHeater**.

---

## Prérequis

* STM32F042F6P6
* Carte iHeater
* Programmateur ST-Link V2 pour le flashage initial ou câble USB
* Système Linux, par exemple Raspberry Pi ou imprimante

!!! warning "Si vous ne parvenez pas à compiler et flasher le firmware directement sur l'imprimante"
    [Consultez la section WSL](user-mods/software/wsl2-ubuntu-ff/)

---

## Compilation de Katapult

1. Clonez le dépôt Katapult :

```bash
git clone https://github.com/Arksine/katapult.git
```

```bash
cd katapult
```

```bash
make menuconfig
```

2. Dans `menuconfig`, sélectionnez :

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

3. Compilez le firmware :

```bash
make
```

Le fichier du firmware sera créé au chemin `out/katapult.bin`.

---

## Flashage de Katapult via DFU

> Cette étape n'est nécessaire qu'une seule fois pour écrire le bootloader Katapult.

### Préparation

Installez `dfu-util` s'il n'est pas encore installé :

```bash
sudo apt install dfu-util
```

Installez le cavalier BOOT0 et redémarrez la carte en coupant puis remettant l'alimentation, ou appuyez sur RESET.
Le microcontrôleur passera en mode DFU.

Vérifiez la connexion :

```bash
lsusb
```

L'appareil suivant doit apparaître :

```text
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Flashage de Katapult

Exécutez :

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Exemple de sortie réussie :

```text
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Retirez le cavalier, maintenez le bouton MODE enfoncé, appuyez puis relâchez RESET, ou reconnectez l'USB.

Après le redémarrage, exécutez :

```bash
ls /dev/serial/by-id/*
```

L'appareil suivant doit apparaître :

```bash
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Si des erreurs de droits d'accès apparaissent :

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

### Remarques

* Katapult occupe les 8 premiers Ko de Flash, il faut donc **indiquer obligatoirement le décalage 8KiB dans Klipper**.
* Il est possible d'entrer en DFU par un double appui sur RESET ou par le bouton GPIO (PA4).
* PA13/PA14 sont utilisés pour SWD.
* Après le flashage de Katapult, le programmateur ST-Link n'est plus nécessaire ; les mises à jour suivantes peuvent être effectuées via USB.

## Installation du firmware sur iHeater

### Compilation du firmware

```bash
cd klipper/
```

```bash
make menuconfig
```

#### Dans le menu de configuration, sélectionnez :

```text
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### Désactivez toutes les options inutilisées :

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

#### Enregistrez les paramètres et quittez le menu.

#### Compilez le firmware

```bash
make clean
```

```bash
make
```

!!! note "Le fichier suivant doit apparaître en résultat :"

```text
Creating hex file out/klipper.bin
```

### Installation du firmware sur la carte iHeater

!!! note "Si nécessaire, installez python3-serial"

```bash
sudo apt install python3-serial
```

**Les étapes suivantes supposent que le bootloader Katapult est déjà installé.**

* Connectez iHeater à l'hôte en mode flashage : maintenez MODE pendant la connexion USB ou appuyez deux fois sur RESET.

* Trouvez l'appareil :

```bash
ls /dev/serial/by-id/
```

Quelque chose comme ceci doit apparaître :

```text
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* Si nécessaire, installez `flashtool` :

```bash
pip install flashtool
```

* Remplacez l'ID de l'appareil par le vôtre et exécutez :

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

Sortie attendue :

```text
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...

[##################################################]

Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

Flash Success
```

* Vérifiez :

```bash
ls /dev/serial/by-id/
```

La sortie doit être approximativement la suivante :

```text
usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

`iHeater est prêt à fonctionner avec Klipper`

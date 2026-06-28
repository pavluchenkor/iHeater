#  Flashage

Ce document contient les instructions de compilation et de flashage du chargeur de démarrage **Katapult** pour le microcontrôleur **iHeater**. Le chargeur de démarrage Katapult permet de flasher le firmware Klipper via USB et contient également la documentation d'installation du firmware **Klipper** sur le contrôleur **iHeater**.

---

## Ce qui est nécessaire

- STM32F042F6P6
- Carte iHeater
- Câble USB
- Système Linux (par exemple, Raspberry Pi ou imprimante)

!!! warning "S'il n'est pas possible de compiler et de flasher le firmware sur l'imprimante"
    [Consultez la section WSL](../user-mods/software/wsl2-ubuntu-ff/)

---

## Compilation de Katapult

1. Cloner le dépôt Katapult :

```bash
git clone https://github.com/Arksine/katapult.git
```
```
cd katapult
```
```
make menuconfig
```

2. Dans `menuconfig`, sélectionner :

![menuconfig](../../img/katapult_menuconfig.jpg)

3. Compilation :

```bash
make
```

Le firmware sera créé dans `out/katapult.bin`.


---

## Flashage de Katapult via DFU

> Cette étape n'est nécessaire qu'une seule fois, pour charger Katapult lui-même.

### Préparation :
Installer l'utilitaire dfu-util s'il n'est pas encore installé :
    
    sudo apt install dfu-util

Selon la version de la carte :

=== "r1"

    Installer le cavalier sur BOOT0 et redémarrer l'alimentation de la carte (ou appuyer sur le bouton RESET).
    Le microcontrôleur démarrera en mode DFU.

=== "r1.1"

    Appuyer sur le bouton BOOT et redémarrer l'alimentation de la carte (ou appuyer sur le bouton RESET), puis relâcher BOOT.
    Le microcontrôleur démarrera en mode DFU.

Vérifier la connexion :

    lsusb

Résultat :

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### Flashage de Katapult :
Passer en mode DFU.

Exécuter la commande :

```
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Exemple de flashage réussi :

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Quitter le mode DFU.

Après le redémarrage 
```
ls /dev/serial/by-id/*
```
Résultat :
```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Si les droits sont insuffisants, des erreurs peuvent survenir lors du flashage. Pour obtenir l'accès, exécuter la commande :
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

!!! warning "si quelque chose s'est mal passé"
    effacez la mémoire du MCU et répétez les étapes précédentes
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### Remarques

- Katapult occupe les 8 premiers Ko de Flash, il faut donc **indiquer impérativement un décalage de 8 KiB dans Klipper**.
- Il est possible d'utiliser soit un double Reset, soit le bouton sur GPIO (PA4) pour entrer en DFU.
- Si PA13/PA14 sont utilisés pour SWD
- Après le flashage de Katapult, il n'est plus nécessaire d'utiliser ST-Link : tout le travail suivant se fait via USB.

## Installation du firmware sur iHeater

### Compilation du firmware
```
cd ~/klipper
```
```
make menuconfig
```

#### Dans le menu de configuration, sélectionner
```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```
#### Désactiver tout ce qui est inutile
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

#### Enregistrer et quitter le menu.

#### Compiler le firmware
```
make clean
```
```
make
```

Résultat :

    Creating hex file out/klipper.bin

### Installation du firmware sur la carte iHeater

!!! note "L'installation de python3-serial peut être nécessaire"
    
    sudo apt install python3-serial

**La suite décrit l'installation avec le bootloader Katapult déjà installé**

- Connecter iHeater à l'hôte en mode programmation (en maintenant le bouton Mode lors de la connexion ou en appuyant deux fois sur RESET).

- Effectuer la recherche 
    ```
    ls /dev/serial/by-id/
    ```
    Le résultat doit ressembler à ceci :

    ```
    usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
    ```

    - Installer flashtool si nécessaire

    ```
    pip install flashtool
    ```

- Modifier avec votre propre ID et saisir :
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin

    Résultat :

        Flashing '/home/pi/klipper/out/klipper.bin'...

        [##################################################]
        
        Write complete: 20 pages
        
        Verifying (block count = 319)...
        
        [##################################################]
        
        Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18
        
        Flash Success

- Vérifier : 
    ```        
    ls /dev/serial/by-id/
    ```
    Résultat :

        usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00

    ```iHeater est prêt à fonctionner avec Klipper```


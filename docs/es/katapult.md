# Firmware

Este documento describe la compilación y el flasheo del bootloader **Katapult** para el microcontrolador **iHeater**. Katapult permite actualizar el firmware de Klipper por USB y también se utiliza al instalar el firmware **Klipper** en el controlador **iHeater**.

---

## Requisitos

* STM32F042F6P6
* Placa iHeater
* Programador ST-Link V2 para el flasheo inicial o cable USB
* Sistema Linux, por ejemplo Raspberry Pi o una impresora

!!! warning "Si no es posible compilar y flashear el firmware directamente en la impresora"
    [Consulte la sección WSL](user-mods/software/wsl2-ubuntu-ff/)

---

## Compilación de Katapult

1. Clone el repositorio de Katapult:

```bash
git clone https://github.com/Arksine/katapult.git
```

```bash
cd katapult
```

```bash
make menuconfig
```

2. En `menuconfig`, seleccione:

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

3. Compile el firmware:

```bash
make
```

El archivo de firmware se creará en la ruta `out/katapult.bin`.

---

## Flasheo de Katapult mediante DFU

> Este paso solo es necesario una vez para grabar el bootloader Katapult.

### Preparación

Instale `dfu-util` si aún no está instalado:

```bash
sudo apt install dfu-util
```

Instale el jumper BOOT0 y reinicie la placa desconectando y conectando la alimentación, o pulse RESET.
El microcontrolador entrará en modo DFU.

Compruebe la conexión:

```bash
lsusb
```

Debe aparecer el dispositivo:

```text
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Flasheo de Katapult

Ejecute:

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Ejemplo de salida correcta:

```text
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Retire el jumper, mantenga pulsado el botón MODE, pulse y suelte RESET, o vuelva a conectar el USB.

Después del reinicio, ejecute:

```bash
ls /dev/serial/by-id/*
```

Debe aparecer el dispositivo:

```bash
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Si se producen errores de permisos:

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

### Notas

* Katapult ocupa los primeros 8 KB de Flash, por lo tanto **en Klipper es obligatorio indicar el desplazamiento 8KiB**.
* Es posible entrar en DFU con una doble pulsación de RESET o mediante el botón GPIO (PA4).
* PA13/PA14 se utilizan para SWD.
* Después de flashear Katapult, el programador ST-Link ya no es necesario; las actualizaciones posteriores se pueden realizar por USB.

## Instalación del firmware en iHeater

### Compilación del firmware

```bash
cd klipper/
```

```bash
make menuconfig
```

#### En el menú de configuración, seleccione:

```text
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### Desactive todas las opciones no utilizadas:

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

#### Guarde la configuración y salga del menú.

#### Compile el firmware

```bash
make clean
```

```bash
make
```

!!! note "Como resultado, debe aparecer el archivo:"

```text
Creating hex file out/klipper.bin
```

### Instalación del firmware en la placa iHeater

!!! note "Si es necesario, instale python3-serial"

```bash
sudo apt install python3-serial
```

**Los siguientes pasos suponen que el bootloader Katapult ya está instalado.**

* Conecte iHeater al host en modo de flasheo: mantenga pulsado MODE al conectar el USB o pulse RESET dos veces.

* Busque el dispositivo:

```bash
ls /dev/serial/by-id/
```

Debe aparecer algo similar a:

```text
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* Si es necesario, instale `flashtool`:

```bash
pip install flashtool
```

* Sustituya el ID del dispositivo por el suyo y ejecute:

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

Salida esperada:

```text
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...

[##################################################]

Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

Flash Success
```

* Compruebe:

```bash
ls /dev/serial/by-id/
```

La salida debe ser aproximadamente así:

```text
usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

`iHeater está listo para funcionar con Klipper`

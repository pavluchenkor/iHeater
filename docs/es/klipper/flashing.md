#  Flasheo

Este documento contiene instrucciones para compilar y flashear el bootloader **Katapult** para el microcontrolador **iHeater**. El bootloader Katapult permite flashear el firmware Klipper por USB y también contiene documentación para instalar el firmware **Klipper** en el controlador **iHeater**.

---

## Qué se necesita

- STM32F042F6P6
- Placa iHeater
- Cable USB
- Sistema Linux (por ejemplo, Raspberry Pi o impresora)

!!! warning "Si no es posible compilar y flashear el firmware en la impresora"
    [Consulte la sección WSL](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

---

## Compilación de Katapult

1. Clonar el repositorio Katapult:

```bash
git clone https://github.com/Arksine/katapult.git
```
```
cd katapult
```
```
make menuconfig
```

2. En `menuconfig`, seleccionar:

![menuconfig](../../img/katapult_menuconfig.jpg)

3. Compilación:

```bash
make
```

El firmware se creará en `out/katapult.bin`.


---

## Flasheo de Katapult mediante DFU

> Este paso solo es necesario una vez, para cargar el propio Katapult.

### Preparación:
Instalar la utilidad dfu-util si aún no está instalada:
    
    sudo apt install dfu-util

Según la versión de la placa:

=== "r1"

    Instalar el jumper en BOOT0 y reiniciar la alimentación de la placa (o pulsar el botón RESET).
    El microcontrolador se iniciará en modo DFU.

=== "r1.1"

    Pulsar el botón BOOT y reiniciar la alimentación de la placa (o pulsar el botón RESET), luego soltar BOOT.
    El microcontrolador se iniciará en modo DFU.

Comprobar la conexión:

    lsusb

Resultado:

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### Flasheo de Katapult:
Cambiar al modo DFU.

Ejecutar el comando:

```
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Ejemplo de flasheo correcto:

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Salir del modo DFU.

Después de reiniciar
```
ls /dev/serial/by-id/*
```
Resultado:
```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Si no hay permisos, pueden producirse errores durante el flasheo. Para obtener acceso, ejecutar el comando:
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

!!! warning "si algo salió mal"
    borre la memoria del MCU y repita los pasos anteriores
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### Notas

- Katapult ocupa los primeros 8 KB de Flash, por lo tanto **en Klipper es obligatorio indicar un offset de 8 KiB**.
- Se puede usar un doble Reset o un botón en GPIO (PA4) para entrar en DFU.
- Si PA13/PA14 se usan para SWD
- Después de flashear Katapult, ya no es necesario usar ST-Link: todo el trabajo posterior se realiza por USB.

## Instalación del firmware en iHeater

### Compilación del firmware
```
cd ~/klipper
```
```
make menuconfig
```

#### En el menú de configuración, seleccionar
```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```
#### Desactivar todo lo innecesario
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

#### Guardar y salir del menú.

#### Compilar el firmware
```
make clean
```
```
make
```

Resultado:

    Creating hex file out/klipper.bin

### Instalación del firmware en la placa iHeater

!!! note "Puede ser necesario instalar python3-serial"
    
    sudo apt install python3-serial

**A continuación se considera la opción de instalación con el bootloader Katapult instalado**

- Conectar iHeater al host en modo de programación (manteniendo pulsado el botón Mode al conectarlo o pulsando RESET dos veces).

- Ejecutar la búsqueda
    ```
    ls /dev/serial/by-id/
    ```
    El resultado debe verse así:

    ```
    usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
    ```

    - Si es necesario, instalar flashtool

    ```
    pip install flashtool
    ```

- Cambiar por su propio ID e introducir:
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin

    Resultado:

        Flashing '/home/pi/klipper/out/klipper.bin'...

        [##################################################]
        
        Write complete: 20 pages
        
        Verifying (block count = 319)...
        
        [##################################################]
        
        Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18
        
        Flash Success

- Comprobar:
    ```        
    ls /dev/serial/by-id/
    ```
    Resultado:

        usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00

    ```iHeater está listo para funcionar con Klipper```

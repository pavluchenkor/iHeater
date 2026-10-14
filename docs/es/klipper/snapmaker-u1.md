# Snapmaker U1

Guía para compilar e instalar el firmware de iHeater en la impresora **Snapmaker U1**.

Se diferencia de la instalación habitual en tres puntos:

1. El firmware se compila **desde el fork de Klipper de Snapmaker** ([u1-klipper](https://github.com/Snapmaker/u1-klipper)), no desde el Klipper principal. Snapmaker ha modificado alrededor del 20 % del código de Klipper, por lo que el firmware del MCU debe ser de la misma versión que el de la impresora.
2. En el fork de Snapmaker, el firmware para STM32F042 con USB **no compila sin una pequeña corrección**. La corrección se indica más abajo.
3. El firmware se compila **en el ordenador**, no en la impresora, y se graba en el iHeater mediante **STM32CubeProgrammer** en modo DFU. El bootloader Katapult **no se utiliza**.

Los ajustes posteriores (configuración, macros, calibración) son iguales que en cualquier impresora con Klipper: consulte [Configuración](configuration.md).

---

## Requisitos

- Placa iHeater (STM32F042F6P6) y un cable USB **con líneas de datos**
- Snapmaker U1 y acceso a ella por SSH (usuario `root`, contraseña `snapmaker`)
- Ordenador para compilar el firmware:
    - **Windows** — WSL (Ubuntu dentro de Windows), opción principal de esta guía
    - **Linux** — los comandos son los mismos, se omite la sección de WSL
    - **macOS** — cualquier máquina virtual con Ubuntu (UTM, Parallels, VirtualBox) o compilar directamente en macOS, como prefiera. La lógica es la misma, los comandos dentro de Ubuntu no cambian.
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (gratuito, Windows / macOS / Linux)

---

## 1. Compruebe la versión del firmware de la impresora

En la pantalla de la impresora o en Fluidd (sección de información del sistema), localice la versión del firmware, por ejemplo `1.4.1`. La necesitará para elegir la versión de Klipper.

---

## 2. Preparación de Ubuntu

#### Windows (WSL)

Abra **PowerShell como administrador** y ejecute:

```powershell
wsl --install -d Ubuntu-24.04
```

Reinicie el ordenador si Windows lo solicita. En el primer arranque de Ubuntu, defina un nombre de usuario y una contraseña.

A partir de aquí, todos los comandos se ejecutan **en la ventana de Ubuntu**.

#### macOS / Linux

Inicie la máquina virtual con Ubuntu (en Linux, basta con un terminal). A partir de aquí, todos los comandos se ejecutan en ella.

Instale las herramientas de compilación:

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. Clonación de Klipper para U1

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### Selección de la versión

Consulte qué versiones de firmware hay en el repositorio:

```bash
git log --oneline | grep "Release firmware"
```

Ejemplo:

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

Elija el commit **que coincida con la versión del firmware de la impresora**. Si no existe la versión exacta (por ejemplo, la impresora tiene `1.4.1` y en la lista solo hay `1.4.0`), tome la versión inferior más próxima de la misma serie: para `1.4.1`, `1.4.0`.

Cree una rama de trabajo a partir del commit elegido (sustituya el hash por el suyo):

```bash
git switch -c iheater edf309af
```

!!! tip "La opción más sencilla"
    Actualice la impresora al firmware más reciente y compile desde el último commit. Así se garantiza que las versiones coincidan.

---

## 4. Corrección de USB para STM32F042

En el fork de Snapmaker se añadió al archivo `src/generic/usb_cdc.c` una función que llama a `usb_clear_stall()`. Solo está implementada para el controlador USB de los STM32 de gama alta (`src/stm32/usbotg.c`). El STM32F042 tiene otro controlador USB (`src/stm32/usbfs.c`), y la compilación falla con el error:

```
undefined reference to `usb_clear_stall'
```

Añada la función que falta: copie el bloque siguiente **completo** (desde `grep` hasta la última línea `EOF` inclusive), péguelo en el terminal de Ubuntu y pulse **Enter**. El propio comando añadirá la función al final del archivo `src/stm32/usbfs.c`; no hace falta editar nada a mano.

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

Verificación:

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

Debe aparecer una línea con `usb_clear_stall(uint32_t ep, int is_in)`.

!!! note
    El comando empieza con `grep -q ... ||`: si Snapmaker corrige esto por su cuenta en una versión nueva, la función no se añadirá por segunda vez.

Para que la versión del firmware no lleve la marca `dirty`, guarde el cambio en un commit:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. Configuración y compilación

```bash
make menuconfig
```

Seleccione:

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    Katapult no se utiliza; el firmware se graba desde el inicio de la flash (`0x08000000`). Si deja `8KiB bootloader`, la placa no arrancará después de grabarla.

En la sección `Optional features`, desactive todo lo que no sea necesario (el STM32F042 tiene solo 32 KB de flash):

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

Desactive también el resto de opciones (sensores, pantallas, ADC externos) si aparecen en la lista.

Guarde (`Q` → `Y`) y compile:

```bash
make clean
```
```bash
make
```

Resultado:

```
  Linking out/klipper.elf
  Creating hex file out/klipper.bin
```

La compilación tarda unos segundos; es normal.

Comprobación del tamaño (opcional):

```bash
arm-none-eabi-size out/klipper.elf
```

La suma de `text` + `data` debe ser inferior a 32768.

!!! tip "Guarde la configuración"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    La próxima vez: `cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. Copia del firmware al ordenador

#### Windows (WSL)

Abra la carpeta con el firmware en el Explorador de Windows:

```bash
explorer.exe out
```

O copie el archivo al escritorio (sustituya `YourUserName` por su nombre de usuario de Windows):

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### Máquina virtual

Use la carpeta compartida de la máquina virtual, si la tiene configurada. Si no, copie el archivo por SSH. En el ordenador principal, ejecute:

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

Sin SSH, puede usar un servidor web temporal. En la máquina virtual, ejecute:

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

Abra `http://VM_IP:8000` en el navegador y descargue `klipper.bin`. El servidor se detiene con `Ctrl+C`.

Para ver la IP de la máquina virtual: `ip -4 addr | grep inet`

---

## 7. Grabación con STM32CubeProgrammer (DFU)

### Entrada en modo DFU

Según la revisión de la placa, el modo DFU se activa con un **jumper** o con un **botón**:

- **Jumper BOOT0** — coloque el jumper y conecte la placa al ordenador por USB (si la placa ya está conectada, pulse RESET). Tras la grabación, hay que retirar el jumper.
- **Botón BOOT** — mantenga pulsado el botón, conecte la placa al ordenador por USB (o pulse RESET) y, a continuación, suelte BOOT.

### Grabación

1. Inicie STM32CubeProgrammer.
2. A la derecha, seleccione la interfaz **USB** y pulse el botón de actualizar: en el campo **Port** aparecerá `USB1`.
3. Pulse **Connect**.
4. A la izquierda, abra la pestaña **Erasing & Programming**.
5. **File path** — seleccione `klipper.bin`.
6. **Start address** — `0x08000000`.
7. Marque **Verify programming**.
8. Pulse **Start Programming**. Debe aparecer el mensaje `File download complete`.
9. Pulse **Disconnect**, retire el jumper BOOT0 (si su placa lo tiene) y vuelva a conectar la placa.

!!! note "Windows: la placa no aparece en CubeProgrammer"
    El controlador DFU se instala junto con STM32CubeProgrammer. Si en el Administrador de dispositivos la placa aparece como `STM32 BOOTLOADER` con un icono amarillo, reinstale el controlador desde la carpeta de CubeProgrammer: `Drivers\DFU_Driver`.

!!! note "Si la placa ya tenía Katapult"
    Se sobrescribirá; es normal: el firmware se compila sin bootloader y ocupa la flash desde el inicio.

---

## 8. Conexión a la U1

Conecte el iHeater al puerto USB de la impresora y acceda a ella por SSH (usuario `root`, contraseña `snapmaker`):

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    Aparece tras volver a grabar el firmware de la impresora o cambiar la IP. Elimine la clave antigua y vuelva a conectarse:
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

Localice el ID de la placa:

```bash
ls /dev/serial/by-id/
```

Resultado:

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. Configuración

El resto es igual que en cualquier instalación estándar de Klipper; consulte [Configuración](configuration.md):

1. Coloque `iHeater.cfg` junto a `printer.cfg` (mediante el editor de configuración de Fluidd o por SSH).
2. Añada en `printer.cfg`:
    ```ini
    [include iHeater.cfg]
    ```
3. Indique en `iHeater.cfg` el ID de la placa:
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Reinicie Klipper (`FIRMWARE_RESTART`).

!!! warning "Actualización del firmware de la impresora"
    Tras actualizar el firmware de la U1, compruebe que `[include iHeater.cfg]` sigue en `printer.cfg`. Si la versión de Klipper de la impresora ha cambiado, recompile el firmware de iHeater desde el commit correspondiente (pasos 3–7).

---

## Solución de problemas

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
La versión del firmware de iHeater no coincide con la de Klipper en la impresora. Recompile desde el commit correspondiente a la versión del firmware de la U1 (paso 3).

**La placa no aparece en `/dev/serial/by-id/`**

- El firmware se compiló con `8KiB bootloader`: recompile con `No bootloader`.
- No se ha retirado el jumper BOOT0: la placa permanece en modo DFU.
- El cable USB no tiene líneas de datos.
- Compruebe si la impresora llega a detectar el dispositivo: `dmesg | tail -20`.

**`undefined reference to 'usb_clear_stall'` al compilar**
No se ha realizado el paso 4.

**`region FLASH overflowed` al compilar**
El firmware no cabe en la flash: desactive las opciones innecesarias en `Optional features` (paso 5).

**`gcode command M141 already registered`**
Ya existe una macro con el mismo nombre en otro archivo de configuración de la impresora. Comente una de ellas.

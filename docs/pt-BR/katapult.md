# Firmware

Este documento descreve a compilação e gravação do bootloader **Katapult** para o microcontrolador **iHeater**. O Katapult permite atualizar o firmware Klipper via USB e também é usado ao instalar o firmware **Klipper** no controlador **iHeater**.

---

## Requisitos

* STM32F042F6P6
* Placa iHeater
* Programador ST-Link V2 para a gravação inicial ou cabo USB
* Sistema Linux, por exemplo Raspberry Pi ou impressora

!!! warning "Se não for possível compilar e gravar o firmware diretamente na impressora"
    [Consulte a seção WSL](user-mods/software/wsl2-ubuntu-ff/)

---

## Compilação do Katapult

1. Clone o repositório Katapult:

```bash
git clone https://github.com/Arksine/katapult.git
```

```bash
cd katapult
```

```bash
make menuconfig
```

2. No `menuconfig`, selecione:

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

3. Compile o firmware:

```bash
make
```

O arquivo de firmware será criado no caminho `out/katapult.bin`.

---

## Gravação do Katapult via DFU

> Esta etapa é necessária apenas uma vez, para gravar o bootloader Katapult.

### Preparação

Instale o `dfu-util`, caso ainda não esteja instalado:

```bash
sudo apt install dfu-util
```

Instale o jumper BOOT0 e reinicie a placa pela alimentação ou pressione RESET.
O microcontrolador entrará no modo DFU.

Verifique a conexão:

```bash
lsusb
```

Deve aparecer o dispositivo:

```text
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Gravação do Katapult

Execute:

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Exemplo de saída bem-sucedida:

```text
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Remova o jumper, mantenha pressionado o botão MODE, pressione e solte RESET ou reconecte o USB.

Após reiniciar, execute:

```bash
ls /dev/serial/by-id/*
```

Deve aparecer o dispositivo:

```bash
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Se ocorrerem erros de permissão:

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

### Observações

* O Katapult ocupa os primeiros 8 KB da Flash, portanto **no Klipper é obrigatório definir o offset de 8KiB**.
* É possível entrar no DFU com um duplo clique em RESET ou pelo botão GPIO (PA4).
* PA13/PA14 são usados para SWD.
* Após gravar o Katapult, o programador ST-Link não é mais necessário; as atualizações seguintes podem ser feitas via USB.

## Instalação do firmware no iHeater

### Compilação do firmware

```bash
cd klipper/
```

```bash
make menuconfig
```

#### No menu de configuração, selecione:

```text
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### Desative todas as opções não utilizadas:

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

#### Salve as configurações e saia do menu.

#### Compile o firmware

```bash
make clean
```

```bash
make
```

!!! note "Como resultado, deve aparecer o arquivo:"

```text
Creating hex file out/klipper.bin
```

### Instalação do firmware na placa iHeater

!!! note "Se necessário, instale python3-serial"

```bash
sudo apt install python3-serial
```

**As etapas a seguir pressupõem que o bootloader Katapult já está instalado.**

* Conecte o iHeater ao host no modo de gravação: mantenha MODE pressionado ao conectar o USB ou pressione RESET duas vezes.

* Encontre o dispositivo:

```bash
ls /dev/serial/by-id/
```

Deve aparecer algo como:

```text
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* Se necessário, instale o `flashtool`:

```bash
pip install flashtool
```

* Substitua o ID do dispositivo pelo seu e execute:

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

Saída esperada:

```text
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...

[##################################################]

Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

Flash Success
```

* Verifique:

```bash
ls /dev/serial/by-id/
```

A saída deve ser aproximadamente assim:

```text
usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

`iHeater está pronto para trabalhar com Klipper`

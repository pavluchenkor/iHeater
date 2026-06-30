#  Flashing

Este documento contém instruções para compilar e gravar o bootloader **Katapult** para o microcontrolador **iHeater**. O bootloader Katapult permite gravar o firmware Klipper via USB, além de conter a documentação de instalação do firmware **Klipper** no controlador **iHeater**

---

## O que será necessário

- STM32F042F6P6
- Placa iHeater
- Cabo USB
- Sistema Linux (por exemplo, Raspberry Pi ou impressora)

!!! warning "Se não for possível compilar e gravar o firmware na impressora"
    [Consulte a seção WSL](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

---

## Compilação do Katapult

1. Clonar o repositório Katapult:

```bash
git clone https://github.com/Arksine/katapult.git
```
```
cd katapult
```
```
make menuconfig
```

2. Em `menuconfig`, selecionar:

![menuconfig](../../img/katapult_menuconfig.jpg)

3. Compilação:

```bash
make
```

O firmware será criado em `out/katapult.bin`.


---

## Flashing do Katapult via DFU

> Esta etapa é necessária apenas uma vez, para gravar o próprio Katapult.

### Preparação:
Instalar o utilitário dfu-util, se ainda não estiver instalado:
    
    sudo apt install dfu-util

De acordo com a versão da placa:

=== "r1"

    Instalar o jumper em BOOT0 e reiniciar a alimentação da placa (ou pressionar o botão RESET).
    O microcontrolador será iniciado no modo DFU.

=== "r1.1"

    Pressionar o botão BOOT e reiniciar a alimentação da placa (ou pressionar o botão RESET), depois soltar BOOT.
    O microcontrolador será iniciado no modo DFU.

Verificar a conexão:

    lsusb

Resultado:

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### Flashing do Katapult:
Colocar no modo DFU.

Executar o comando:

```
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Exemplo de flashing bem-sucedido:

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Sair do modo DFU.

Após reiniciar 
```
ls /dev/serial/by-id/*
```
Resultado:
```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Se não houver permissões, podem ocorrer erros durante o flashing. Para obter acesso, executar o comando:
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

!!! warning "se algo deu errado"
    apague a memória do MCU e repita as etapas anteriores
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### Observações

- Katapult ocupa os primeiros 8 KB da Flash, portanto **no Klipper é obrigatório especificar o deslocamento de 8 KiB**.
- É possível usar o Reset duplo ou o botão em GPIO (PA4) para entrar em DFU.
- Se PA13/PA14 forem usados para SWD
- Após gravar o Katapult, não é mais necessário usar ST-Link - todo o trabalho posterior é feito via USB.

## Instalação do firmware no iHeater

### Compilação do firmware
```
cd ~/klipper
```
```
make menuconfig
```

#### No menu de configuração, selecionar
```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```
#### Desativar tudo que for desnecessário
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

#### Salvar e sair do menu.

#### Compilar o firmware
```
make clean
```
```
make
```

Resultado:

    Creating hex file out/klipper.bin

### Instalação do firmware na placa iHeater

!!! note "Pode ser necessário instalar python3-serial"
    
    sudo apt install python3-serial

**A seguir é considerado o cenário de instalação com o bootloader Katapult instalado**

- Conectar o iHeater ao host no modo de programação (mantendo o botão Mode pressionado durante a conexão ou pressionando RESET duas vezes).

- Executar a busca 
    ```
    ls /dev/serial/by-id/
    ```
    O resultado deve ter esta aparência:

    ```
    usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
    ```

    - Se necessário, instalar flashtool

    ```
    pip install flashtool
    ```

- Altere para o seu ID e insira:
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin

    Resultado:

        Flashing '/home/pi/klipper/out/klipper.bin'...

        [##################################################]
        
        Write complete: 20 pages
        
        Verifying (block count = 319)...
        
        [##################################################]
        
        Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18
        
        Flash Success

- Verificar: 
    ```        
    ls /dev/serial/by-id/
    ```
    Resultado:

        usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00

    ```iHeater está pronto para trabalhar com Klipper```



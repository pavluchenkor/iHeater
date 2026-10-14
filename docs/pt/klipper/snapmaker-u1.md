# Snapmaker U1

Instruções para compilar e instalar o firmware do iHeater na impressora **Snapmaker U1**.

Difere da instalação habitual em três aspetos:

1. O firmware é compilado **a partir do fork do Klipper da Snapmaker** ([u1-klipper](https://github.com/Snapmaker/u1-klipper)), e não do Klipper principal. A Snapmaker alterou cerca de 20 % do código do Klipper, pelo que o firmware do MCU tem de ser da mesma versão que a da impressora.
2. No fork da Snapmaker, a compilação para STM32F042 com USB **não compila sem uma pequena correção**. A correção é apresentada abaixo.
3. O firmware é compilado **no computador**, e não na impressora, e é gravado no iHeater através do **STM32CubeProgrammer** em modo DFU. O bootloader Katapult **não é utilizado**.

Os passos seguintes (configuração, macros, calibração) são iguais aos de qualquer impressora com Klipper: consulte [Configuração](configuration.md).

---

## O que é necessário

- Placa iHeater (STM32F042F6P6) e cabo USB **com linhas de dados**
- Snapmaker U1 e acesso a ela por SSH (utilizador `root`, palavra-passe `snapmaker`)
- Computador para compilar o firmware:
    - **Windows** — WSL (Ubuntu dentro do Windows), opção principal destas instruções
    - **Linux** — os comandos são os mesmos; salte a secção sobre o WSL
    - **macOS** — qualquer máquina virtual com Ubuntu (UTM, Parallels, VirtualBox) ou compilação diretamente no macOS, conforme preferir. A lógica é a mesma e os comandos no Ubuntu são idênticos.
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (gratuito, Windows / macOS / Linux)

---

## 1. Descubra a versão do firmware da impressora

No ecrã da impressora ou no Fluidd (secção com informações do sistema), encontre a versão do firmware, por exemplo `1.4.1`. Vai precisar dela para escolher a versão do Klipper.

---

## 2. Preparação do Ubuntu

#### Windows (WSL)

Abra o **PowerShell como administrador** e execute:

```powershell
wsl --install -d Ubuntu-24.04
```

Reinicie o computador, se o Windows o pedir. No primeiro arranque do Ubuntu, defina o nome de utilizador e a palavra-passe.

A partir daqui, todos os comandos são executados **na janela do Ubuntu**.

#### macOS / Linux

Inicie a máquina virtual com Ubuntu (no Linux, basta o terminal). A partir daqui, todos os comandos são executados nela.

Instale as ferramentas de compilação:

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. Clonagem do Klipper para o U1

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### Escolha da versão

Veja que versões de firmware existem no repositório:

```bash
git log --oneline | grep "Release firmware"
```

Exemplo:

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

Escolha o commit **correspondente à versão do firmware da impressora**. Se a versão exata não existir (por exemplo, a impressora tem `1.4.1` e a lista só tem `1.4.0`), utilize a versão anterior mais próxima com o mesmo número: para `1.4.1` — `1.4.0`.

Crie um ramo de trabalho a partir do commit escolhido (substitua pelo seu hash):

```bash
git switch -c iheater edf309af
```

!!! tip "A opção mais simples"
    Atualize a impressora para o firmware mais recente e compile a partir do último commit. Assim, há a garantia de que as versões coincidem.

---

## 4. Correção de USB para STM32F042

No fork da Snapmaker, foi adicionada ao ficheiro `src/generic/usb_cdc.c` uma função que chama `usb_clear_stall()`. No entanto, `usb_clear_stall()` só está implementada para o controlador USB dos STM32 de gama superior (`src/stm32/usbotg.c`). O STM32F042 tem outro controlador USB (`src/stm32/usbfs.c`), e a compilação falha com o erro:

```
undefined reference to `usb_clear_stall'
```

Adicione a função em falta: copie o bloco abaixo **na íntegra** (de `grep` até à última linha `EOF`, inclusive), cole-o no terminal do Ubuntu e prima **Enter**. O comando acrescenta a função ao fim do ficheiro `src/stm32/usbfs.c`, sem necessidade de editar nada manualmente.

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

Verificação:

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

Deve aparecer uma linha com `usb_clear_stall(uint32_t ep, int is_in)`.

!!! note
    O comando começa com `grep -q ... ||`: se a Snapmaker corrigir isto numa versão futura, a função não será adicionada uma segunda vez.

Para que a versão do firmware não tenha a marca `dirty`, guarde a alteração com um commit:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. Configuração e compilação

```bash
make menuconfig
```

Selecione:

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    O Katapult não é utilizado; o firmware é gravado desde o início da flash (`0x08000000`). Se deixar `8KiB bootloader`, a placa não arrancará após a gravação.

Na secção `Optional features`, desative tudo o que for supérfluo (o STM32F042 tem apenas 32 KB de flash):

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

Desative também as restantes opções (sensores, ecrãs, ADC externos), se existirem na lista.

Guarde (`Q` → `Y`) e compile:

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

A compilação demora apenas alguns segundos, o que é normal.

Verificação do tamanho (opcional):

```bash
arm-none-eabi-size out/klipper.elf
```

A soma de `text` + `data` deve ser inferior a 32768.

!!! tip "Guarde as definições"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    Da próxima vez: `cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. Cópia do firmware para o computador

#### Windows (WSL)

Abra a pasta com o firmware no Explorador do Windows:

```bash
explorer.exe out
```

Ou copie o ficheiro para o ambiente de trabalho (substitua `YourUserName` pelo nome de utilizador do Windows):

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### Máquina virtual

Através da pasta partilhada da máquina virtual, se estiver configurada. Caso contrário, através de SSH. No computador principal, execute:

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

Sem SSH, utilize um servidor web temporário. Na máquina virtual, execute:

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

Abra no navegador `http://VM_IP:8000` e transfira `klipper.bin`. Para parar o servidor, prima `Ctrl+C`.

IP da máquina virtual: `ip -4 addr | grep inet`

---

## 7. Gravação através do STM32CubeProgrammer (DFU)

### Entrada em modo DFU

Consoante a revisão da placa, o modo DFU é ativado por **jumper** ou por **botão**:

- **Jumper BOOT0** — coloque o jumper e ligue a placa ao computador por USB (se a placa já estiver ligada, prima RESET). Após a gravação, é necessário retirar o jumper.
- **Botão BOOT** — mantenha o botão premido, ligue a placa ao computador por USB (ou prima RESET) e depois solte BOOT.

### Gravação

1. Inicie o STM32CubeProgrammer.
2. À direita, selecione a interface **USB** e clique no botão de atualizar — no campo **Port** aparecerá `USB1`.
3. Clique em **Connect**.
4. À esquerda, abra o separador **Erasing & Programming**.
5. **File path** — selecione `klipper.bin`.
6. **Start address** — `0x08000000`.
7. Marque **Verify programming**.
8. Clique em **Start Programming**. Deve aparecer a mensagem `File download complete`.
9. Clique em **Disconnect**, retire o jumper BOOT0 (se existir na sua placa) e volte a ligar a placa.

!!! note "Windows: a placa não aparece no CubeProgrammer"
    O driver DFU é instalado juntamente com o STM32CubeProgrammer. Se, no Gestor de Dispositivos, a placa aparecer como `STM32 BOOTLOADER` com um ícone amarelo, reinstale o driver a partir da pasta do CubeProgrammer: `Drivers\DFU_Driver`.

!!! note "Se a placa já tinha o Katapult"
    Será substituído, o que é normal: o firmware foi compilado sem bootloader e ocupa a flash desde o início.

---

## 8. Ligação ao U1

Ligue o iHeater à porta USB da impressora e aceda à impressora por SSH (utilizador `root`, palavra-passe `snapmaker`):

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    Surge depois de voltar a gravar o firmware da impressora ou de o IP mudar. Remova a chave antiga e volte a ligar-se:
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

Encontre o ID da placa:

```bash
ls /dev/serial/by-id/
```

Resultado:

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. Configuração

A partir daqui, o processo é igual ao de um Klipper normal — consulte [Configuração](configuration.md):

1. Coloque `iHeater.cfg` junto a `printer.cfg` (através do editor de configuração no Fluidd ou por SSH).
2. Em `printer.cfg`, adicione:
    ```ini
    [include iHeater.cfg]
    ```
3. Em `iHeater.cfg`, indique o ID da placa:
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Reinicie o Klipper (`FIRMWARE_RESTART`).

!!! warning "Atualização do firmware da impressora"
    Após atualizar o firmware do U1, verifique se `[include iHeater.cfg]` se mantém em `printer.cfg`. Se a versão do Klipper na impressora tiver mudado, volte a compilar o firmware do iHeater a partir do commit correspondente (passos 3–7).

---

## Problemas

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
A versão do firmware do iHeater não coincide com a do Klipper da impressora. Volte a compilar a partir do commit correspondente à versão do firmware do U1 (passo 3).

**A placa não aparece em `/dev/serial/by-id/`**

- O firmware foi compilado com `8KiB bootloader` — volte a compilar com `No bootloader`.
- O jumper BOOT0 não foi retirado — a placa permanece em modo DFU.
- Cabo USB sem linhas de dados.
- Verifique se a impressora sequer deteta o dispositivo: `dmesg | tail -20`.

**`undefined reference to 'usb_clear_stall'` durante a compilação**
O passo 4 não foi executado.

**`region FLASH overflowed` durante a compilação**
O firmware não cabe na flash: desative as opções supérfluas em `Optional features` (passo 5).

**`gcode command M141 already registered`**
Uma macro com o mesmo nome já está definida noutra configuração da impressora. Comente uma delas.

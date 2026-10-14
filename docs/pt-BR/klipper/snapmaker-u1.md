# Snapmaker U1

Instruções para compilar e instalar o firmware do iHeater na impressora **Snapmaker U1**.

Difere da instalação padrão em três pontos:

1. O firmware é compilado **a partir do fork do Klipper da Snapmaker** ([u1-klipper](https://github.com/Snapmaker/u1-klipper)), e não do Klipper principal. A Snapmaker alterou cerca de 20% do código do Klipper, portanto o firmware do MCU precisa ser da mesma versão instalada na impressora.
2. No fork da Snapmaker, a compilação para STM32F042 com USB **não compila sem uma pequena correção**. A correção está descrita abaixo.
3. O firmware é compilado **no computador**, e não na impressora, e é gravado no iHeater pelo **STM32CubeProgrammer** em modo DFU. O bootloader Katapult **não é usado**.

Os passos seguintes (configuração, macros, calibração) são iguais aos de qualquer impressora com Klipper: veja [Configuração](configuration.md).

---

## O que é necessário

- Placa iHeater (STM32F042F6P6) e um cabo USB **com linhas de dados**
- Snapmaker U1 e acesso a ela via SSH (login `root`, senha `snapmaker`)
- Computador para compilar o firmware:
    - **Windows** — WSL (Ubuntu dentro do Windows), opção principal destas instruções
    - **Linux** — os comandos são os mesmos; pule a seção sobre WSL
    - **macOS** — qualquer máquina virtual com Ubuntu (UTM, Parallels, VirtualBox) ou compilação direto no macOS, como preferir. A lógica é a mesma, e os comandos dentro do Ubuntu são idênticos.
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (gratuito, Windows / macOS / Linux)

---

## 1. Descubra a versão do firmware da impressora

Na tela da impressora ou no Fluidd (seção de informações do sistema), encontre a versão do firmware, por exemplo `1.4.1`. Ela será necessária para escolher a versão do Klipper.

---

## 2. Preparação do Ubuntu

#### Windows (WSL)

Abra o **PowerShell como administrador** e execute:

```powershell
wsl --install -d Ubuntu-24.04
```

Reinicie o computador se o Windows solicitar. Ao abrir o Ubuntu pela primeira vez, defina um nome de usuário e uma senha.

A partir daqui, execute todos os comandos **na janela do Ubuntu**.

#### macOS / Linux

Inicie a máquina virtual com Ubuntu (no Linux, basta o terminal). A partir daqui, execute todos os comandos nela.

Instale as ferramentas de compilação:

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. Clonagem do Klipper para a U1

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

Veja quais versões de firmware existem no repositório:

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

Escolha o commit **correspondente à versão do firmware da impressora**. Se a versão exata não existir (por exemplo, a impressora está na `1.4.1` e a lista só tem a `1.4.0`), use a versão anterior mais próxima da mesma série: para a `1.4.1`, use a `1.4.0`.

Crie um branch de trabalho a partir do commit escolhido (use o seu próprio hash):

```bash
git switch -c iheater edf309af
```

!!! tip "A opção mais simples"
    Atualize a impressora para o firmware mais recente e compile a partir do último commit. Assim, as versões vão coincidir com certeza.

---

## 4. Correção de USB para o STM32F042

No fork da Snapmaker, o arquivo `src/generic/usb_cdc.c` ganhou uma função que chama `usb_clear_stall()`. Ela está implementada apenas para o controlador USB dos STM32 mais avançados (`src/stm32/usbotg.c`). O STM32F042 tem outro controlador USB (`src/stm32/usbfs.c`), e a compilação falha com o erro:

```
undefined reference to `usb_clear_stall'
```

Adicione a função que falta: copie o bloco abaixo **por inteiro** (de `grep` até a última linha `EOF`, inclusive), cole-o no terminal do Ubuntu e pressione **Enter**. O próprio comando acrescenta a função ao final do arquivo `src/stm32/usbfs.c`, sem necessidade de editar nada manualmente.

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

Deve aparecer a linha com `usb_clear_stall(uint32_t ep, int is_in)`.

!!! note
    O comando começa com `grep -q ... ||`: se a Snapmaker corrigir isso por conta própria em uma versão futura, a função não será adicionada uma segunda vez.

Para que a versão do firmware não fique marcada como `dirty`, salve a alteração em um commit:

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
    O Katapult não é usado, e o firmware é gravado a partir do início da flash (`0x08000000`). Se você deixar `8KiB bootloader`, a placa não vai iniciar após a gravação.

Na seção `Optional features`, desative tudo o que for desnecessário (o STM32F042 tem apenas 32 KB de flash):

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

Desative também os demais itens (sensores, displays, ADCs externos), se aparecerem na lista.

Salve (`Q` → `Y`) e compile:

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

A compilação leva alguns segundos; isso é normal.

Verificação do tamanho (opcional):

```bash
arm-none-eabi-size out/klipper.elf
```

A soma de `text` + `data` deve ser menor que 32768.

!!! tip "Salve as configurações"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    Da próxima vez: `cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. Cópia do firmware para o computador

#### Windows (WSL)

Abra a pasta com o firmware no Explorador de Arquivos do Windows:

```bash
explorer.exe out
```

Ou copie o arquivo para a área de trabalho (substitua `YourUserName` pelo nome de usuário do Windows):

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### Máquina virtual

Use a pasta compartilhada da máquina virtual, se estiver configurada. Caso contrário, use SSH. No computador principal, execute:

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

Sem SSH, use um servidor web temporário. Na máquina virtual:

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

Abra `http://VM_IP:8000` no navegador e baixe o `klipper.bin`. Para parar o servidor, pressione `Ctrl+C`.

IP da máquina virtual: `ip -4 addr | grep inet`

---

## 7. Gravação pelo STM32CubeProgrammer (DFU)

### Entrada no modo DFU

Dependendo da revisão da placa, o modo DFU é ativado por **jumper** ou por **botão**:

- **Jumper BOOT0** — coloque o jumper e conecte a placa ao computador por USB (se a placa já estiver conectada, pressione RESET). Depois da gravação, remova o jumper.
- **Botão BOOT** — mantenha o botão pressionado, conecte a placa ao computador por USB (ou pressione RESET) e, em seguida, solte o BOOT.

### Gravação

1. Inicie o STM32CubeProgrammer.
2. À direita, selecione a interface **USB** e clique no botão de atualizar — no campo **Port** aparecerá `USB1`.
3. Clique em **Connect**.
4. À esquerda, abra a aba **Erasing & Programming**.
5. **File path** — selecione `klipper.bin`.
6. **Start address** — `0x08000000`.
7. Marque **Verify programming**.
8. Clique em **Start Programming**. Deve aparecer a mensagem `File download complete`.
9. Clique em **Disconnect**, remova o jumper BOOT0 (se a sua placa tiver um) e reconecte a placa.

!!! note "Windows: a placa não aparece no CubeProgrammer"
    O driver DFU é instalado junto com o STM32CubeProgrammer. Se, no Gerenciador de Dispositivos, a placa aparecer como `STM32 BOOTLOADER` com um ícone amarelo, reinstale o driver a partir da pasta do CubeProgrammer: `Drivers\DFU_Driver`.

!!! note "Se a placa já tinha o Katapult"
    Ele será sobrescrito, e isso é normal: o firmware foi compilado sem bootloader e ocupa a flash desde o início.

---

## 8. Conexão com a U1

Conecte o iHeater à porta USB da impressora e acesse a impressora via SSH (login `root`, senha `snapmaker`):

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    Este aviso aparece depois de regravar o firmware da impressora ou de uma mudança de IP. Remova a chave antiga e conecte novamente:
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

A partir daqui, tudo segue como em qualquer Klipper padrão — veja [Configuração](configuration.md):

1. Coloque o `iHeater.cfg` na mesma pasta do `printer.cfg` (pelo editor de configuração do Fluidd ou via SSH).
2. No `printer.cfg`, adicione:
    ```ini
    [include iHeater.cfg]
    ```
3. No `iHeater.cfg`, informe o ID da placa:
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Reinicie o Klipper (`FIRMWARE_RESTART`).

!!! warning "Atualização do firmware da impressora"
    Após atualizar o firmware da U1, verifique se `[include iHeater.cfg]` permanece no `printer.cfg`. Se a versão do Klipper na impressora mudou, recompile o firmware do iHeater a partir do commit correspondente (passos 3–7).

---

## Problemas

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
A versão do firmware do iHeater não coincide com o Klipper da impressora. Recompile a partir do commit correspondente à versão do firmware da U1 (passo 3).

**A placa não aparece em `/dev/serial/by-id/`**

- O firmware foi compilado com `8KiB bootloader` — recompile com `No bootloader`.
- O jumper BOOT0 não foi removido — a placa permanece em modo DFU.
- Cabo USB sem linhas de dados.
- Verifique se a impressora chega a detectar o dispositivo: `dmesg | tail -20`.

**`undefined reference to 'usb_clear_stall'` durante a compilação**
O passo 4 não foi executado.

**`region FLASH overflowed` durante a compilação**
O firmware não cabe na flash: desative os itens desnecessários em `Optional features` (passo 5).

**`gcode command M141 already registered`**
Uma macro com o mesmo nome já está definida em outro arquivo de configuração da impressora. Comente uma das duas.

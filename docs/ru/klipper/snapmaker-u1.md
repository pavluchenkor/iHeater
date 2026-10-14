# Snapmaker U1

Инструкция по сборке и установке прошивки iHeater для принтера **Snapmaker U1**.

От обычной установки она отличается в трёх местах:

1. Прошивка собирается **из форка Klipper от Snapmaker** ([u1-klipper](https://github.com/Snapmaker/u1-klipper)), а не из основного Klipper. Snapmaker изменили около 20 % кода Klipper, поэтому прошивка MCU должна быть из той же версии, что и на принтере.
2. В форке Snapmaker сборка под STM32F042 с USB **не собирается без небольшого исправления**. Исправление приведено ниже.
3. Прошивка собирается **на компьютере**, а не на принтере, и записывается в iHeater через **STM32CubeProgrammer** в режиме DFU. Загрузчик Katapult **не используется**.

Дальнейшая настройка (конфигурация, макросы, калибровка) такая же, как для любого принтера на Klipper: см. [Конфигурация](configuration.md).

---

## Что потребуется

- Плата iHeater (STM32F042F6P6) и USB-кабель **с линиями данных**
- Snapmaker U1 и доступ к нему по SSH (логин `root`, пароль `snapmaker`)
- Компьютер для сборки прошивки:
    - **Windows** — WSL (Ubuntu внутри Windows), основной вариант этой инструкции
    - **Linux** — команды те же, раздел про WSL пропускается
    - **macOS** — любая виртуальная машина с Ubuntu (UTM, Parallels, VirtualBox) или сборка прямо на macOS, как удобнее. Логика та же, команды внутри Ubuntu не отличаются.
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (бесплатный, Windows / macOS / Linux)

---

## 1. Узнайте версию прошивки принтера

На экране принтера или в Fluidd (раздел с информацией о системе) найдите версию прошивки, например `1.4.1`. Она понадобится для выбора версии Klipper.

---

## 2. Подготовка Ubuntu

#### Windows (WSL)

Откройте **PowerShell от имени администратора** и выполните:

```powershell
wsl --install -d Ubuntu-24.04
```

Перезагрузите компьютер, если Windows попросит. При первом запуске Ubuntu задайте имя пользователя и пароль.

Дальше все команды выполняются **в окне Ubuntu**.

#### macOS / Linux

Запустите виртуальную машину с Ubuntu (на Linux — просто терминал). Дальше все команды выполняются в ней.

Установите инструменты сборки:

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. Клонирование Klipper для U1

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### Выбор версии

Посмотрите, какие версии прошивки есть в репозитории:

```bash
git log --oneline | grep "Release firmware"
```

Пример:

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

Выберите коммит, **совпадающий с версией прошивки принтера**. Если точной версии нет (например, на принтере `1.4.1`, а в списке только `1.4.0`), берите ближайшую младшую с тем же номером: для `1.4.1` — `1.4.0`.

Создайте рабочую ветку от выбранного коммита (подставьте свой хеш):

```bash
git switch -c iheater edf309af
```

!!! tip "Самый простой вариант"
    Обновите принтер до последней прошивки и собирайте из последнего коммита. Тогда версии гарантированно совпадут.

---

## 4. Исправление USB для STM32F042

В форке Snapmaker в файл `src/generic/usb_cdc.c` добавлена функция, которая вызывает `usb_clear_stall()`. Реализована она только для USB-контроллера старших STM32 (`src/stm32/usbotg.c`). У STM32F042 другой USB-контроллер (`src/stm32/usbfs.c`), и сборка падает с ошибкой:

```
undefined reference to `usb_clear_stall'
```

Добавьте недостающую функцию: скопируйте блок ниже **целиком** (от `grep` до последней строки `EOF` включительно), вставьте его в терминал Ubuntu и нажмите **Enter**. Команда сама допишет функцию в конец файла `src/stm32/usbfs.c`, ничего редактировать вручную не нужно.

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

Проверка:

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

Должна найтись строка с `usb_clear_stall(uint32_t ep, int is_in)`.

!!! note
    Команда начинается с `grep -q ... ||`: если Snapmaker в новой версии исправят это сами, функция не добавится второй раз.

Чтобы в версии прошивки не было пометки `dirty`, сохраните изменение коммитом:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. Настройка и сборка

```bash
make menuconfig
```

Выберите:

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    Katapult не используется, прошивка записывается с самого начала флеша (`0x08000000`). Если оставить `8KiB bootloader`, плата после прошивки не запустится.

В разделе `Optional features` выключите всё лишнее (у STM32F042 всего 32 КБ флеша):

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

Остальные пункты (датчики, дисплеи, внешние АЦП) тоже выключите, если они есть в списке.

Сохраните (`Q` → `Y`) и соберите:

```bash
make clean
```
```bash
make
```

Результат:

```
  Linking out/klipper.elf
  Creating hex file out/klipper.bin
```

Сборка занимает несколько секунд, это нормально.

Проверка размера (необязательно):

```bash
arm-none-eabi-size out/klipper.elf
```

Сумма `text` + `data` должна быть меньше 32768.

!!! tip "Сохраните настройки"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    В следующий раз: `cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. Копирование прошивки на компьютер

#### Windows (WSL)

Откройте папку с прошивкой в проводнике Windows:

```bash
explorer.exe out
```

Или скопируйте файл на рабочий стол (замените `ИмяПользователя` на имя пользователя Windows):

```bash
cp out/klipper.bin /mnt/c/Users/ИмяПользователя/Desktop/
```

#### Виртуальная машина

Через общую папку виртуальной машины, если она настроена. Если нет — через SSH. На основном компьютере выполните:

```bash
scp user@IP_ВИРТУАЛКИ:~/u1-klipper/out/klipper.bin ~/Desktop/
```

Без SSH — временный веб-сервер. В виртуальной машине:

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

Откройте в браузере `http://IP_ВИРТУАЛКИ:8000` и скачайте `klipper.bin`. Сервер останавливается `Ctrl+C`.

IP виртуальной машины: `ip -4 addr | grep inet`

---

## 7. Прошивка через STM32CubeProgrammer (DFU)

### Вход в режим DFU

В зависимости от ревизии платы режим DFU включается **джампером** или **кнопкой**:

- **Джампер BOOT0** — установите джампер и подключите плату к компьютеру по USB (если плата уже подключена — нажмите RESET). После прошивки джампер нужно снять.
- **Кнопка BOOT** — зажмите кнопку, подключите плату к компьютеру по USB (или нажмите RESET), затем отпустите BOOT.

### Запись

1. Запустите STM32CubeProgrammer.
2. Справа выберите интерфейс **USB**, нажмите обновить — в поле **Port** появится `USB1`.
3. Нажмите **Connect**.
4. Слева откройте вкладку **Erasing & Programming**.
5. **File path** — выберите `klipper.bin`.
6. **Start address** — `0x08000000`.
7. Отметьте **Verify programming**.
8. Нажмите **Start Programming**. Должно появиться сообщение `File download complete`.
9. Нажмите **Disconnect**, снимите джампер BOOT0 (если он есть на вашей плате) и переподключите плату.

!!! note "Windows: плата не видна в CubeProgrammer"
    Драйвер DFU ставится вместе с STM32CubeProgrammer. Если в диспетчере устройств плата видна как `STM32 BOOTLOADER` с жёлтым значком, переустановите драйвер из папки CubeProgrammer: `Drivers\DFU_Driver`.

!!! note "Если на плате раньше был Katapult"
    Он будет перезаписан, это нормально: прошивка собрана без загрузчика и занимает флеш с начала.

---

## 8. Подключение к U1

Подключите iHeater к USB-порту принтера и зайдите на принтер по SSH (логин `root`, пароль `snapmaker`):

```bash
ssh root@IP_ПРИНТЕРА
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    Появляется после перепрошивки принтера или смены IP. Удалите старый ключ и подключитесь заново:
    ```bash
    ssh-keygen -R IP_ПРИНТЕРА
    ```

Найдите ID платы:

```bash
ls /dev/serial/by-id/
```

Результат:

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. Конфигурация

Дальше всё как для обычного Klipper — см. [Конфигурация](configuration.md):

1. Положите `iHeater.cfg` рядом с `printer.cfg` (через редактор конфигурации в Fluidd или по SSH).
2. В `printer.cfg` добавьте:
    ```ini
    [include iHeater.cfg]
    ```
3. В `iHeater.cfg` укажите ID платы:
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Перезапустите Klipper (`FIRMWARE_RESTART`).

!!! warning "Обновление прошивки принтера"
    После обновления прошивки U1 проверьте, что `[include iHeater.cfg]` остался в `printer.cfg`. Если версия Klipper на принтере изменилась, пересоберите прошивку iHeater из соответствующего коммита (шаги 3–7).

---

## Проблемы

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
Версия прошивки iHeater не совпадает с Klipper на принтере. Пересоберите из коммита, соответствующего версии прошивки U1 (шаг 3).

**Плата не появляется в `/dev/serial/by-id/`**

- Прошивка собрана с `8KiB bootloader` — пересоберите с `No bootloader`.
- Не снят джампер BOOT0 — плата остаётся в режиме DFU.
- USB-кабель без линий данных.
- Проверьте, видит ли принтер устройство вообще: `dmesg | tail -20`.

**`undefined reference to 'usb_clear_stall'` при сборке**
Не выполнен шаг 4.

**`region FLASH overflowed` при сборке**
Не помещается во флеш: выключите лишние пункты в `Optional features` (шаг 5).

**`gcode command M141 already registered`**
Макрос с таким же именем уже определён в другом конфиге принтера. Закомментируйте один из них.

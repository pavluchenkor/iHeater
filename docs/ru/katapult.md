# Прошивка

Этот документ описывает сборку и прошивку загрузчика **Katapult** для микроконтроллера **iHeater**. Katapult позволяет обновлять прошивку Klipper по USB, а также используется при установке прошивки **Klipper** на контроллер **iHeater**.

---

## Требования

* STM32F042F6P6
* Плата iHeater
* Программатор ST-Link V2 для первичной прошивки или USB-кабель
* Linux-система, например Raspberry Pi или принтер

!!! warning "Если не получается собрать и прошить прошивку прямо на принтере"
    [Обратитесь к разделу WSL](user-mods/software/wsl2-ubuntu-ff/)

---

## Сборка Katapult

1. Склонируйте репозиторий Katapult:

```bash
git clone https://github.com/Arksine/katapult.git
```

```bash
cd katapult
```

```bash
make menuconfig
```

2. В `menuconfig` выберите:

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

3. Соберите прошивку:

```bash
make
```

Файл прошивки будет создан по пути `out/katapult.bin`.

---

## Прошивка Katapult через DFU

> Этот шаг нужен только один раз, чтобы записать загрузчик Katapult.

### Подготовка

Установите `dfu-util`, если он ещё не установлен:

```bash
sudo apt install dfu-util
```

Установите перемычку BOOT0 и перезагрузите плату по питанию или нажмите RESET.
Микроконтроллер перейдёт в режим DFU.

Проверьте подключение:

```bash
lsusb
```

Должно появиться устройство:

```text
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Прошивка Katapult

Выполните:

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Пример успешного вывода:

```text
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Снимите перемычку, зажмите кнопку MODE, нажмите и отпустите RESET или переподключите USB.

После перезагрузки выполните:

```bash
ls /dev/serial/by-id/*
```

Должно появиться устройство:

```bash
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Если возникнут ошибки прав доступа:

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

### Примечания

* Katapult занимает первые 8 КБ Flash, поэтому **в Klipper нужно обязательно указать смещение 8KiB**.
* Войти в DFU можно двойным нажатием RESET или кнопкой GPIO (PA4).
* PA13/PA14 используются для SWD.
* После прошивки Katapult программатор ST-Link больше не нужен; последующие обновления можно выполнять по USB.

## Установка прошивки на iHeater

### Сборка прошивки

```bash
cd klipper/
```

```bash
make menuconfig
```

#### В меню конфигурации выберите:

```text
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### Отключите все неиспользуемые опции:

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

#### Сохраните настройки и выйдите из меню.

#### Соберите прошивку

```bash
make clean
```

```bash
make
```

!!! note "В результате должен появиться файл:"

```text
Creating hex file out/klipper.bin
```

### Установка прошивки на плату iHeater

!!! note "При необходимости установите python3-serial"

```bash
sudo apt install python3-serial
```

**Следующие шаги предполагают, что загрузчик Katapult уже установлен.**

* Подключите iHeater к хосту в режиме прошивки: удерживайте MODE при подключении USB или дважды нажмите RESET.

* Найдите устройство:

```bash
ls /dev/serial/by-id/
```

Должно появиться что-то вроде:

```text
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* При необходимости установите `flashtool`:

```bash
pip install flashtool
```

* Замените ID устройства на свой и выполните:

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

Ожидаемый вывод:

```text
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...

[##################################################]

Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

Flash Success
```

* Проверьте:

```bash
ls /dev/serial/by-id/
```

Вывод должен быть примерно таким:

```text
usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

`iHeater готов к работе с Klipper`

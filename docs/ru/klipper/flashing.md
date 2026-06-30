#  Прошивка

Этот документ содержит инструкции по сборке и прошивке загрузчика **Katapult** для микроконтроллера **iHeater**. Загрузчик Katapult позволяет прошивать прошивку Klipper по USB, а так же содержит документацию по установке прошивки **Klipper** на контроллер **iHeater** 

---

## Что потребуется

- STM32F042F6P6
- Плата iHeater
- USB кабель
- Linux-система (например, Raspberry Pi или принтер)

!!! warning "Если нет возможности собрать и прошить прошивку на принтере"
    [Обратитесь к разделу WSL](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

---

## Сборка Katapult

1. Клонировать репозиторий Katapult:

```bash
git clone https://github.com/Arksine/katapult.git
```
```
cd katapult
```
```
make menuconfig
```

2. В `menuconfig` выбрать:

![menuconfig](../../img/katapult_menuconfig.jpg)

3. Сборка:

```bash
make
```

Прошивка будет создана в `out/katapult.bin`.


---

## Прошивка Katapult через DFU

> Этот шаг нужен только один раз, для загрузки самого Katapult.

### Подготовка:
Устновить утилиту dfu-util, если она ещё не установлена:
    
    sudo apt install dfu-util

В соответствии с версией платы:

=== "r1"

    Установить джампер на BOOT0 и перезапустить питание платы (или нажать кнопку RESET).
    Микроконтроллер загрузится в режим DFU.

=== "r1.1"

    Нажать кнопку BOOT и перезапустить питание платы (или нажми кнопку RESET) отпустить BOOT.
    Микроконтроллер загрузится в режим DFU.

Проверить подключение:

    lsusb

Результат:

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### Прошивка Katapult:
Перевести в режим DFU.

Выполнить команду:

```
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

Пример успешной прошивки:

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

Выйти из режима DFU.

После перезапуска 
```
ls /dev/serial/by-id/*
```
Результат:
```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

Если нет прав, возможны ошибки при прошивке, чтобы получить доступ выполнить команду:
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

!!! warning "если что-то пошло не так"
    затрите память МК и повторите предыдущие шаги
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### Примечания

- Katapult занимает первые 8 КБ Flash, поэтому **в Klipper обязательно указывать смещение 8 KiB**.
- Можно использовать либо двойной Reset, либо кнопку на GPIO (PA4) для входа в DFU.
- Если PA13/PA14 используются для SWD
- После прошивки Katapult можно больше не использовать ST-Link - вся дальнейшая работа по USB.

## Установка прошивки на iHeater

### Компиляция прошивки
```
cd ~/klipper
```
```
make menuconfig
```

#### В меню конфигурации выбрать
```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```
#### Выключить все лишнее
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

#### Сохранить и выйти из меню.

#### Скомпилировать прошивку
```
make clean
```
```
make
```

Результат:

    Creating hex file out/klipper.bin

### Установка прошивки на плату iHeater

!!! note "Может потребоваться установка python3-serial"
    
    sudo apt install python3-serial

**Далее рассматривается вариант установки с установленным бутлоадером Katapult**

- Подключить iHeater к хосту в режиме программирования (удерживая кнопку Mode при подключении или дважды нажав RESET).

- Выполнить поиск 
    ```
    ls /dev/serial/by-id/
    ```
    Результат должен выглядеть так:

    ```
    usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
    ```

    - При необходимости установить flashtool

    ```
    pip install flashtool
    ```

- Изменить на свой ID и введите:
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin

    Результат:

        Flashing '/home/pi/klipper/out/klipper.bin'...

        [##################################################]
        
        Write complete: 20 pages
        
        Verifying (block count = 319)...
        
        [##################################################]
        
        Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18
        
        Flash Success

- Проверить: 
    ```        
    ls /dev/serial/by-id/
    ```
    Результат:

        usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00

    ```iHeater готов для работы с Klipper```



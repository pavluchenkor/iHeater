# Конфигурация iHeater для Klipper

Данный репозиторий содержит конфигурационные файлы для нагревателя камеры 3D-принтера iHeater на основе прошивки Klipper и одноименной платы управления. Конфигурация предназначена для управления нагревом камеры и вентиляторами с помощью микроконтроллера iHeater.

## Оглавление

- [Требования](#требования)
- [Подготовка](#подготовка)
- [Установка прошивки на iHeater](#установка-прошивки-на-iheater)
- [Конфигурация Klipper](#конфигурация-klipper)
  - [Подключение MCU `iHeater`](#1-подключение-mcu-iheater)
  - [Настройка нагревателя `iHeater_H`](#2-настройка-нагревателя-iheater_h)
  - [Настройка вентилятора `iHeater_F`](#3-настройка-вентилятора-iheater_f)
  - [Настройка датчиков температуры](#4-настройка-датчиков-температуры)
  - [Макросы G-кода](#макросы-g-кода)
- [Использование](#использование)
  - [Команды управления нагревом камеры](#команды-управления-нагревом-камеры)
  - [Автоматизация и логика управления](#автоматизация-и-логика-управления)
- [Примечания](#примечания)
- [Лицензия](#лицензия)

## Требования

- **Аппаратное обеспечение:**
  - Плата управления iHeater
  - Терморезисторы NTC 100K MGB18-104F39050L32 (2 шт.)
  - PTC нагревательный элемент 220В 100Вт, для камеры
  - Вентилятор 7530 220В, для циркуляции воздуха в камере

- **Программное обеспечение:**
  - Klipper (последняя версия)
  - Настроенный и работающий 3D-принтер с Klipper

## Подготовка

1. **Сборка аппаратной части:**
   - Подключите нагревательный элемент и вентиляторы к iHeater.
   - Установите терморезисторы в камере и подключите их к соответствующим пинам MCU.
   - Убедитесь в правильности подключения пинов согласно конфигурационному файлу.

2. **Установка необходимых файлов:**
   - Скопируйте файл `rp2040_pin_aliases.cfg` и `iHeaterMCU.cfg`  в директорию конфигурации Klipper.

## Установка прошивки на iHeater

1. **Соберите прошивку Klipper для RP2040:**

   cd klipper/
   make menuconfig


**В меню конфигурации выберите:**

 - Micro-controller Architecture: RP2040
 - Processor model: rp2040
 - Остальные настройки оставьте по умолчанию.

**Сохраните и выйдите из меню.**

2. **Скомпилируйте прошивку:**

        make

3. ##Установка прошивки на плату iHeter:##

- Подключите iHeaterк компьютеру в режиме программирования (удерживая кнопку BOOTSEL при подключении).

- Смонтируйте устройство и загрузите прошивку:

        sudo mount /dev/sda1 /mnt
        sudo cp out/klipper.uf2 /mnt
        sudo umount /mnt

> **Примечание:** Замените /dev/sda1 на соответствующий путь к вашему устройству.

## Конфигурация Klipper

Скопируйте конфигурационные файлы iHeaterMCU.cfg и rp2040_pin_aliases.cfg в папку с printer.cfg и включите его с помощью директивы [include].
        
        [include iHeater.cfg]

### 1. Подключение MCU iHeater

[Выполните поиск MCU](#https://www.klipper3d.org/Installation.html#building-and-flashing-the-micro-controller)

        [mcu iHeater]
        serial: /dev/serial/by-id/usb-Klipper_rp2040_DE63581213745233-if00

- Описание:
    - Подключает дополнительный микроконтроллер iHeater по указанному серийному порту.


### 2. Настройка нагревателя iHeater_H

    [heater_generic iHeater_H]
    heater_pin: iHeater:H0
    max_power: 1
    sensor_type: NTC 100K MGB18-104F39050L32
    sensor_pin: iHeater:T0
    control: pid
    pwm_cycle_time: 0.3
    min_temp: 0
    max_temp: 120
    pid_Kp=32.923
    pid_Ki=5.628
    pid_Kd=48.150

    [verify_heater iHeater_H]
    max_error: 240
    check_gain_time: 120
    heating_gain: 1

- Описание:
    - heater_generic iHeater_H: Настраивает нагреватель камеры.
heater_pin: Пин, к которому подключен нагревательный элемент.
    - sensor: Использует датчик температуры iHeater_Sens_H.
    - Параметры PID для точного контроля температуры.
-verify_heater iHeater_H: Параметры проверки нагревателя для безопасности.

### 3. Настройка вентилятора iHeater_F

    [fan_generic iHeater_F1]
    pin: iHeater:FAN0
    max_power: 1.0
    shutdown_speed: 0

- Описание:
    
    - Настраивает вентилятор для циркуляции воздуха в камере.
    - Управляется макросами в зависимости от температуры.

### 4. Настройка датчиков температуры

    [temperature_sensor iHeater_Sens_C]
    sensor_pin: iHeater:T0
    sensor_type: NTC 100K MGB18-104F39050L32


- Описание:
    - iHeater_Sens_C: Датчик температуры камеры.
    - iHeater_Sens_H: Датчик температуры нагревателя.


### 5. Макросы G-кода
Переопределение команд M141 и M191

    [gcode_macro M141]
    rename_existing: M141.1
    gcode:
        M141.1 S{params.S|0}
        UPDATE_DELAYED_GCODE ID=_iHEATER_CONTROL DURATION=0

    [gcode_macro M191]
    gcode:
        M191.1 S{params.S|0}
        UPDATE_DELAYED_GCODE ID=_iHEATER_CONTROL DURATION=0

- Описание:
    - Переопределяют стандартные команды для управления нагревом камеры.
    - При вызове запускают макрос _iHEATER_CONTROL.

Макрос управления нагревом и вентилятором

    [delayed_gcode _iHEATER_CONTROL]
    gcode:
        {% set target_heater_temp = printer['gcode_macro HEATER_TARGET'].heater_target %}
        {% set current_heater_temp = printer['heater_generic iHeater_H']temperature %}
        {% set chamber_temp = printer['temperature_sensor iHeater_Sens_C']temperature %}
        {% set delta = printer['gcode_macro DELTA_TEMPERATURE']delta_temp %}
        {% set target_chamber_temp = target_heater_temp - delta %}
        {% set fan_speed = printer['gcode_macro FAN_SPEED']fan_speed %}
        
        {% if target_heater_temp > 0 %}
            {% if chamber_temp < target_chamber_temp %}
                # Поддерживаем температуру нагревателя
                SET_HEATER_TEMPERATURE HEATER=iHeater_H TARGET={target_heater_temp}
                # Включаем вентилятор на заданную скорость
                SET_FAN_SPEED FAN=iHeater_F1 SPEED={fan_speed}
            {% else %}
                # Уменьшаем температуру нагревателя
                {% set reduced_heater_temp = (target_heater_temp - (delta / 4.0)) | round(2) %}
                SET_HEATER_TEMPERATURE HEATER=iHeater_H TARGET={reduced_heater_temp}
                # Включаем вентилятор на максимальную скорость
                SET_FAN_SPEED FAN=iHeater_F1 SPEED={fan_speed}
                # Обновляем переменную скорости вентилятора до максимальной
            {% endif %}
        {% else %}
            # HEATER_TARGET == 0, процесс отключения
            {% if current_heater_temp > 50 %}
                # Поддерживаем вентилятор включенным пока нагреватель не остынет до 50°C
                SET_FAN_SPEED FAN=iHeater_F1 SPEED=1.0
                SET_GCODE_VARIABLE VARIABLE=fan_speed VALUE=1.0
            {% else %}
                # Отключаем вентилятор и останавливаем контролирующий макрос
                SET_FAN_SPEED FAN=iHeater_F1 SPEED=0.0
                SET_GCODE_VARIABLE VARIABLE=fan_speed VALUE=0.0
                CANCEL_DELAYED_GCODE ID=_iHEATER_CONTROL
                RESPOND prefix="iHeater_control" msg="Нагрев камеры и вентилятор отключены."
            {% endif %}
        {% endif %}
        
- Описание:
    - Отслеживает температуру и управляет вентилятором в зависимости от заданной логики.
    - Вентилятор включается при достижении температуры камеры 50°C.

## Использование
### Команды управления нагревом камеры
- Установка температуры камеры:
 

        M141 S60  ; Устанавливает температуру камеры на 60°C

- Ожидание достижения температуры:

        M191 S60  ; Ждет, пока температура камеры достигнет 60°C

- Остановка нагрева камеры:

        M141 S0   ; Отключает нагрев камеры


## Автоматизация и логика управления
Макрос _iHEATER_CONTROL автоматически управляет вентилятором в зависимости от температуры камеры.
При достижении температуры 50°C вентилятор iHeater_F включается.
Макросы можно настроить под свои требования, изменив пороговые значения и логику.


## Примечания
- Безопасность:

    - Убедитесь, что все подключения выполнены правильно и безопасно.
    - Проверьте, что значения min_temp и max_temp соответствуют спецификациям оборудования.

- Проверка оборудования:
    - Перед использованием протестируйте работу нагревателя и вентилятора.
    - Следите за температурой во время первых запусков.
- Настройка PID:
    - При необходимости выполните калибровку PID для точного контроля температуры.


## Лицензия
Данный проект распространяется под лицензией MIT. Подробности смотрите в файле LICENSE.


>** Внимание: Использование нагревательных элементов и управление температурой связано с риском возгорания и повреждения оборудования. Всегда следуйте рекомендациям производителя и соблюдайте меры предосторожности.
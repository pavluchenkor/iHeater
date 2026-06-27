## Прошивка iHeater для принтеров Creality через Creality Helper Script

Чтобы прошивка и интеграция iHeater прошли успешно, следуйте пошаговой инструкции:

### 1. Установите Creality Helper Script

Перейдите на страницу документации проекта Creality Helper Script и следуйте инструкции по установке скрипта.

**Ресурсы:**

* Видеоруководство: [YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* Текстовая инструкция: [guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. Получите root-доступ к принтеру и доступ к файловой системе

Скрипт откроет доступ к Mainsail, а также к файлам конфигурации прошивки. После успешной установки убедитесь, что можете зайти на интерфейс принтера через браузер и получить доступ к файлам конфигурации.

### 3. Удалите старый `fan-control.cfg`

На принтерах Creality с Helper Script по умолчанию уже может быть создан файл `fan-control.cfg` с макросами `M141` и `M191`. Он конфликтует с аналогичными макросам в конфигурации iHeater.

Переименуйте файл:

```
/usr/data/printer_data/config/fan-control.cfg
```
в fan-control.cfg.bak

### 4. Скопируйте новый `fan-control.cfg`

Замените его на версию [fan-control.cfg](../../../printers/creality/config/fans-control.cfg), совместимую с макросами и логикой управления температурой камеры.

Поместите новый файл в ту же папку:

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. Добавьте конфигурацию iHeater

Скопируйте файл `iheater.cfg` в ту же директорию:

```
/usr/data/printer_data/config/iheater.cfg
```

Затем откройте `printer.cfg` и в конце файла добавьте строку:

```ini
[include iheater.cfg]
```

---

Далее следуйте инструкции по настройке iHeater - настройка термистора, нагревателя, режимов работы и макросов.

!!! warning "Если нет возможности собрать и прошить прошивку на принтере"
    [Обратитесь к разделу WSL](../user-mods/software/wsl2-ubuntu-ff/)

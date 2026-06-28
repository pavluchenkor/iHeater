# Klipper-Konfiguration

Diese Seite beschreibt die Installation der iHeater-Konfigurationsdateien und die Einrichtung der Arbeit mit Klipper.

## Anforderungen

### Hardware
  - iHeater-Steuerplatine
  - NTC 100K 3950 Thermistoren (2 Stk.)
  - PTC-Heizelement 220V 100W, für die Kammer
  - Lüfter 7530 220V, für die Luftzirkulation in der Kammer
  - Thermal Protector KSD9700 oder vergleichbar (220 V, 5 A, 130 °C)

### Software
  - Klipper (neueste Version)
  - Eingerichteter und funktionierender Host mit Klipper

## Klipper-Konfiguration


Kopieren Sie die iHeater.cfg-Konfigurationsdateien in den Ordner mit der Datei printer.cfg (das kann /klipper_config sein) und binden Sie sie in printer.cfg mit der Direktive [include] ein


```
cd ~/klipper_config
```

```
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

Öffnen Sie printer.cfg und fügen Sie hinzu

    [include iHeater.cfg]

## Anschluss der iHeater-MCU

Ändern Sie die Datei iHeater.cfg und geben Sie die erhaltene ID an

```
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ

```

## Vorbereitung zur Nutzung

Die Konfigurationsdatei enthält den Abschnitt:

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # Целевая температура камеры, °C
variable_start_offset: 10           # Температура камеры, достаточная для начала печати, °C
variable_delta_temp: 10             # Разница между температурой камеры и нагревателя, °C
variable_min_heater_temp: 50        # Минимальная температура нагревателя (для охлаждения), °C
variable_max_heater_temp: 100       # Максимальная температура нагревателя, °C
variable_control_interval: 1.0      # Интервал вызова функции управления, секунды
variable_air_min_delta: 0.5         # Минимальная разница между целевой и текущей температурой камеры (нагреватель = целевая + delta_temp), °C
variable_air_max_delta: 5.0         # Максимальная разница между целевой и текущей температурой камеры (нагреватель = max_heater_temp), °C
gcode:
```

**Die maximal zulässige Temperatur des Heizers hängt vom Material des Gehäuses ab.**

Zur Prüfung:

!. Aktivieren Sie die Bettheizung auf 90-100°C
1. Stellen Sie die Temperatur des Heizers über die Fluidd- oder Mainsail-Oberfläche auf 100°C ein.
2. Stellen Sie sicher, dass sich iHeater innerhalb des geschlossenen Druckervolumens befindet.
3. Prüfen Sie nach Erreichen der eingestellten Temperatur die Bereiche, in denen der Heizer die Kunststoffelemente des Gehäuses berührt. Der Kunststoff darf nicht weich werden.
4. Erhöhen Sie die Temperatur um 5-10°C und wiederholen Sie die Prüfung.
5. Wiederholen Sie den Vorgang, bis die maximal zulässige Temperatur des Heizers ohne Risiko einer Gehäuseverformung erreicht ist.

Dieser Ansatz ermöglicht es, ein sicheres Temperaturmaximum zu bestimmen und die bestmögliche Effizienz des iHeater zu erreichen.


## Verwendung

### Befehle zur Steuerung der Kammerheizung
- Einstellen der Kammertemperatur:
 

        M141 S60  ; Устанавливает температуру камеры на 60°C

- Warten auf das Erreichen der Temperatur:

        M191 S60  ; Ждет, пока температура камеры достигнет 60°C

- Stoppen der Kammerheizung:

        iHEATER_OFF   ; Отключает нагрев камеры

- Fügen Sie am Ende des Slicer-G-codes `iHEATER_OFF` hinzu, um die Kammerheizung korrekt auszuschalten.

### Start-g-code

Moderne Slicer unterstützen das automatische Einschalten einer aktiven Thermokammer beim Erzeugen des Druck-g-code. Dazu muss in den Filamenteigenschaften die Kammertemperatur angegeben werden. Falls der Slicer diese Funktionalität nicht besitzt, muss dem Start-g-code ein Befehl zum Einschalten der Heizung der aktiven Thermokammer hinzugefügt werden.

Vorgehensweise:

- Zieltemperatur der Kammer einstellen
- Bettheizung für eine effiziente und schnelle Erwärmung der Kammer einschalten 
- Mit dem standardmäßigen Start-G-code des Drucks fortfahren

Beispiel für Start-g-code
```
; --- Начало стартового G-code ---

; ****** Старт iHeater ******
M141 S60       ; Установить температуру камеры на 60°C
; ****** Конец блока iHeater ******

; --- Остальной стартовый g-code ---
; Включение нагрева стола
...
```
!!! warning "Für den korrekten Abschluss des iHeater-Steuermakros muss dem abschließenden g-code des Druckers der Befehl iHEATER_OFF hinzugefügt werden"

```
; --- Начало завершающего g-code ---

; ****** Старт блока iHeater ******
iHEATER_OFF
; ****** Конец блока iHeater ******

; --- Остальной завершающий g-code ---
...
```
## Deaktivierung

Zum Deaktivieren von iHeater muss in der Datei printer.cfg die Zeile [include iHeater.cfg] auskommentiert werden
```
# [include iHeater.cfg]
```

Und entfernen Sie aus dem Start- und Abschluss-g-code die entsprechenden Zeilen

## Hinweise
- Sicherheit:

    - Stellen Sie sicher, dass alle Anschlüsse korrekt und sicher ausgeführt sind.
    - Prüfen Sie, dass die Werte min_temp und max_temp den Spezifikationen der Hardware entsprechen.

- Hardwareprüfung:
    - Testen Sie vor der Nutzung den Betrieb von Heizer und Lüfter.
    - Überwachen Sie die Temperatur während der ersten Starts.
- PID-Einstellung:
    - Führen Sie bei Bedarf eine PID-Kalibrierung für eine genaue Temperaturregelung durch.

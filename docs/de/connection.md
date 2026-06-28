## Anschlussplan

Der Controller **iHeater** kann sowohl als Teil des **Klipper**-Systems (als zusätzlicher MCU) als auch autonom betrieben werden — unter Steuerung der integrierten **standalone**-Firmware.

### Anschluss für den Betrieb mit Klipper

Für den korrekten Betrieb als Teil von Klipper müssen angeschlossen werden:

* **USB-Kabel** zum Haupthost (Host-MCU) - darüber erfolgen Datenübertragung und 5V-Stromversorgung;
* **Netzversorgung 220V / 110V** - abhängig von der Geräteversion und dem Heizungstyp;
* **Heizungs-Thermistor** - zur Überwachung der Temperatur des Heizelements;
* **Kammer-Thermistor** - zur Überwachung der Lufttemperatur in der Druckerkammer;
* **Trigger-Port** - optionaler Anschluss, wird zur automatischen Steuerung über ein externes Signal verwendet.

Im Betriebszustand wird iHeater innerhalb der Kammer des 3D-Druckers platziert.

!!! note annotate "Es wird empfohlen, den Kammer-Thermistor auf Höhe des Druckkopfs zu platzieren, nach Möglichkeit - **über dem Bett**."

![Anschlussplan](../img/iHeater_pinout.png)

## GPIO-Konfiguration

| Pin    | Alias       | Function                          |
|--------|-------------|-----------------------------------|
| PA0    | TH1         | Temperatursensor der Kammer       |
| PA1    | HEATER      | Steuerung der Heizung             |
| PA2    | FAN         | Steuerung des Lüfters             |
| PA3    | TH0         | Temperatursensor der Heizung      |
| PA4    | MODE        | Modustaste                        |
| PA5    | LED3        | LED 3                             |
| PA6    | LED2        | LED 2                             |
| PA7    | LED1        | LED 1                             |
| PB1    | TH2         | Zusätzlicher Temperatursensor     |

---

### Verwendung im standalone-Modus

Im autonomen Modus stehen zusätzliche Funktionen und Anschlussmöglichkeiten zur Verfügung:

* **Trigger-Port im Thermistor-Modus**
  Beim Anschluss eines Thermistors an den Trigger-Port und dessen Platzierung in der Nähe des Heizelements des Betts kann die automatische Steuerung aktiviert werden:
  - bei einer Betttemperatur über **45°C** - wird die Kammerheizung eingeschaltet;
  - bei einem Absinken der Temperatur unter **85°C** - wird die Heizung ausgeschaltet.

* **Stromversorgung über eine externe 5V-Quelle**
  Wenn die USB-Stromversorgung nicht verwendet werden kann, kann die Stromversorgung direkt über den entsprechenden Anschluss zugeführt werden.

* **Experimentelle Option**
  Anschluss einer [5V-Stromversorgung direkt an die Platine](https://sl.aliexpress.ru/p?key=OHtN3Xm).

!!! danger "USB und externe Stromquelle nicht gleichzeitig verwenden"
    Der gleichzeitige Anschluss von zwei Stromquellen ist unzulässig. Dies führt zu einem Konflikt der Stromquellen, Fehlern im Gerätebetrieb und kann die Ausrüstung beschädigen.

![Stromanschluss](../img/IMG_6009.jpg)

---

### Allgemeiner Anschlussplan

![Anschlussplan](../img/iHeater_connection.png)

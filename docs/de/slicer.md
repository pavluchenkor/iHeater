## Einrichten des Start-G-codes im Slicer für den korrekten Betrieb von iHeater

Für eine gleichmäßige Erwärmung und stabilen Druck mit technischen Materialien ist es wichtig, die Reihenfolge der Befehle im Start-G-code korrekt einzurichten. Unten finden Sie Empfehlungen zur Integration der iHeater-Kammerheizung in den Startcode Ihres Slicers (zum Beispiel Cura, PrusaSlicer, OrcaSlicer usw.).

### Was zu tun ist

Vor dem Einschalten der Kammerheizung müssen Sie:

**Heizbett einschalten**

   Das Vorheizen des Heizbetts hilft der Kammer, sich gleichmäßiger und schneller zu erwärmen. Ein solches Heizbett dient als zusätzliche Wärmequelle, die zu einer effizienteren Erwärmung der gesamten Kammer beiträgt.

   ```
   M140 S[first_layer_bed_temperature]
   ```

**Luftumwälzlüfter einschalten (falls verwendet)**

   Dies kann ein seitlicher Lüfter oder ein anderer Lüfter sein, der für die gleichmäßige Verteilung der Wärme in der Kammer vorgesehen ist.
   Wenn er in der Klipper-Konfiguration zum Beispiel `chamber_fan` heißt, dann:

   ```
   SET_FAN_SPEED FAN=chamber_fan SPEED=1.0
   ```

In einigen Druckern sind Abluftlüfter installiert, die eine möglichst niedrige Temperatur in der Kammer halten. Das ist beim Drucken mit Kunststoffen wie PLA und PETG wichtig, beeinträchtigt aber die Aufheizgeschwindigkeit der Kammer. An einen solchen Lüfter kann ein neuer Temperaturparameter übergeben werden, zum Beispiel Zieltemperatur_in_der_Kammer + 10

```
SET_TEMPERATURE_FAN_TARGET TEMPERATURE_FAN=chamber_fan TARGET={chamber_temperature + 10}
```

**Kammerheizung mit einem der Makros einschalten: `M141` oder `M191`**

### Unterschied zwischen `M141` und `M191`

| Makro  | Zweck                                                                        | Blockiert die Codeausführung                                          |
| ------ | ---------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `M141` | Setzt die Zieltemperatur der Kammer                                          | Nein (die Erwärmung beginnt, aber der Druck wird sofort fortgesetzt)  |
| `M191` | Setzt die Kammertemperatur und wartet, bis der angegebene Wert erreicht wird | Ja (der nächste Befehl wird erst nach dem Aufheizen ausgeführt)        |

#### Beispiele:

* Wenn Sie die Kammer sofort aufheizen und alle vorbereitenden Vorgänge parallel zur Erwärmung ausführen möchten und anschließend mit dem Druck fortfahren wollen, ohne auf die vollständige Erwärmung der Kammer zu warten, eignet sich diese Variante. Das funktioniert gut für Kunststoffe wie ABS und große Modelle - während die ersten paar Schichten gedruckt werden, hat die Kammer Zeit, sich aufzuwärmen:

  ```
  M141 S60
  ```

* Wenn ein kleines Teil gedruckt wird oder eine stabile Kammertemperatur erforderlich ist (zum Beispiel beim Drucken von PA, PC und anderen empfindlichen Materialien), ist es besser, den Befehl mit Wartezeit auf die Erwärmung zu verwenden:

  ```
  M191 S60
  ```

### Nach dem Aufheizen der Kammer

Nach dem Aufruf eines der Makros (`M141` oder `M191`) können Sie mit dem normalen Start-G-code fortfahren:

```gcode
M190 S[first_layer_bed_temperature]  Warten auf das Aufheizen des Heizbetts
M109 S[first_layer_temperature]  Warten auf das Aufheizen des Hotends
G28  Homing
G29  (falls automatische Kalibrierung verwendet wird)
... 
```

### Empfehlungen

* Wenn Sie `M191` verwenden, können Sie eine kleine Abweichung festlegen, bei der die Kammer als ausreichend aufgeheizt gilt (zum Beispiel 5°C unter dem Zielwert) - dies wird im Makro [gcode_macro CHAMBER_VARS] über `variable_start_offset` konfiguriert.
* Stellen Sie sicher, dass alle verwendeten Makros (`M141`, `M191`) in der Klipper-Konfiguration eingebunden und korrekt eingerichtet sind.
* Wenn Ihr Slicer Bedingungen unterstützt, können Sie eine Prüfung hinzufügen: zum Beispiel die Kammer nur bei einer Drucktemperatur über 50°C einschalten (für ABS, ASA usw.).

---

### Einrichten der Temperatur in der Heizkammer im Slicer

![Slicer_settings](../img/slicer_settings.jpeg)

In vielen modernen Slicern kann die gewünschte Temperatur der Heizkammer direkt im Filamentprofil angegeben werden. Dieser Wert lässt sich bequem als Parameter `S` für die Makros `M141` oder `M191` verwenden.

Der Screenshot zeigt das Feld "Chamber temperature", in dem der Wert `60°C` eingestellt ist. Das ist nur ein Zahlenwert - **nicht alle Slicer senden einen Befehl zum Aufheizen der Kammer**. Damit die Temperatur tatsächlich angewendet wird, müssen Sie prüfen, wie der Slicer den G-code erzeugt, und bei Bedarf Befehle im Start-G-code verwenden:

```gcode
M141 S{chamber_temperature}
```

oder

```gcode
M191 S{chamber_temperature}
```

Stellen Sie außerdem sicher, dass die Variable `chamber_temperature` in den Slicer-Einstellungen definiert ist, oder ersetzen Sie sie manuell durch eine Zahl.

Wenn Sie die Option "Activate temperature control" aktivieren, fügen einige Slicer automatisch einen `M191`-Befehl mit dem angegebenen Temperaturwert hinzu. Das ist praktisch, wenn die Kammer vor dem Druck mit Wartezeit aufgeheizt werden soll.

Es wird empfohlen, die manuelle Steuerung über Makros zu verwenden, um die Logik des Aufheizens und die Reihenfolge der Aktionen vollständig zu kontrollieren.

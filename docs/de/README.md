# Über das Projekt iHeater

iHeater ist ein kompakter Heizer zum Aufbau einer aktiven Wärmekammer in einem 3D-Drucker. Besonders gefragt ist er bei Modellen mit geschlossener oder proprietärer Elektronik - Creality, Bambu Lab, FlashForge - bei denen keine freien Anschlüsse für den Anschluss von Heizer, Lüfter und Thermistor vorhanden sind.

Er wird per USB angeschlossen und arbeitet unabhängig von den Einschränkungen der Hauptplatine. Je nach Firmware - vollständig in Klipper integriert oder autonom.

In Kombination mit der Heizbettbeheizung sorgt iHeater für eine gleichmäßige Erwärmung der Kammer - ein entscheidender Faktor beim Drucken von ABS, PA, PC und anderen technischen Kunststoffen. Das Gerät regelt die Heizung dynamisch abhängig von der Lufttemperatur und schafft stabile Bedingungen in der Kammer ohne Überhitzung und Temperatursprünge.

Es sind zwei Versionen verfügbar:

- 100 W - für kleinere Drucker (archiviert)
- 200 W - für größere Drucker

![iHeater](../img/iHeater_promo.png)

[Eine Vorabschätzung können Sie mit diesem Rechner durchführen](https://docs.google.com/spreadsheets/d/1u6XrWLFZGOUnRlFPjjGsJB_GLuFFIs3fFWLCp2-K8gc/edit?usp=sharing)

## Einsatzmöglichkeiten

### Unter Steuerung von Klipper

Die Platine arbeitet als separate MCU in Klipper und steuert die Kammerheizung und den Lüfter vollständig autonom. Die Versorgung mit 220 V belastet das Netzteil des Druckers nicht - die serienmäßigen Netzteile arbeiten oft am Limit.

![PCB](../img/iHeater_200_PCB.png)

Die Kosten der Platine sind vergleichbar mit oder niedriger als der Eigenbau einer ähnlichen Lösung auf Basis eines Mikrocontrollers, eines Solid-State-Relais und der erforderlichen Komponenten. Für Enthusiasten bleibt die Möglichkeit, ein entsprechendes System selbst aufzubauen.

### Mit iHeater-Firmware

Die iHeater-Platine ist eigenständig und enthält die gesamte notwendige Peripherie, um als eigenständiges Gerät verwendet zu werden. Die Zieltemperatur wird durch wiederholtes Drücken der MODE-Taste eingestellt und über drei LEDs angezeigt.

## Lizenz

Das Projekt wird unter der MIT-Lizenz veröffentlicht. Details stehen in der Datei [LICENSE](license.md).

!!! danger "Arbeiten mit Heizelementen"
    Die Verwendung von Heizelementen und die Temperaturregelung sind mit Brandgefahr und dem Risiko von Geräteschäden verbunden. Beachten Sie die Vorsichtsmaßnahmen. Weitere Informationen finden Sie im Abschnitt [Sicherheit](safety.md).

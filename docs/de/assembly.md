# Montage

Lesen Sie die Dokumentation, laden Sie die benötigten Teile herunter und drucken Sie sie aus. Stellen Sie sicher, dass alle Komponenten und Werkzeuge vorhanden sind, bevor Sie mit der Montage beginnen.

!!! danger "Arbeiten mit Netzspannung"
    Führen Sie alle Arbeiten am Anschluss an das 110–230-V-Netz nur bei spannungsfreiem Gerät durch. Weitere Informationen finden Sie im Abschnitt [Sicherheit](safety.md).

## Vor der Montage

Es wird empfohlen, zunächst das gesamte System **auf dem Tisch**, ohne Einbau in das Gehäuse, aufzubauen und zu testen:

- **Alle** Komponenten anschließen.
- Die Funktion von Heizer, Lüfter und Temperatursensoren prüfen.
- Das System mit **Klipper** verbinden oder mit der Standalone-Firmware flashen und die korrekte Funktion sicherstellen.

Videoanleitung: [YouTube](https://youtu.be/1QMtVY0Vx-8?si=Ol1u4Ux9wALDcfe2)

## Schrittweise Montage

### Installation der Platine

![Montage iHeater](../img/iHeater_5484.jpg)

### Installation des Thermistors und Thermal Protector

!!! warning "Installation des Thermistors"
    Stellen Sie sicher, dass die freiliegenden Drahtabschnitte an der Basis des Thermistors das Metallgehäuse des Heizers nicht berühren. Isolieren Sie diese Abschnitte bei Bedarf mit Kaptonband oder legen Sie sie in ein Teflonrohr / einen Schrumpfschlauch.

    Die Temperatur des Heizers kann 140 °C erreichen.

!!! warning "Installation des Thermal Protector"
    Es kann ein KSD9700 (Thermal Protector, selbstrückstellend) oder eine einmalige Thermal Fuse installiert werden.

    Der KSD9700 unterbricht den Stromkreis bei Überhitzung und schließt ihn beim Abkühlen automatisch wieder. Die Thermal Fuse (zum Beispiel **RH130**) unterbricht den Stromkreis beim Auslösen dauerhaft - ein zuverlässigerer Schutz bei einer Fehlfunktion.

    Verwenden Sie den KSD9700 während der Debugging-Phase und ersetzen Sie ihn anschließend für den Dauerbetrieb durch eine Thermal Fuse.

![Montage iHeater](../img/iHeater_5489.jpg)
![Montage iHeater](../img/thermistor.jpg)

### Installation des Heizers

!!! warning "Installation des Thermistors"
    Installieren Sie den Thermistor am Rand des Heizers, ungefähr auf halber Höhe der Kühlrippen.

    Die freiliegenden Drahtabschnitte an der Basis des Thermistors dürfen das Metallgehäuse des Heizers nicht berühren. Isolieren Sie diese Abschnitte bei Bedarf mit Kaptonband oder legen Sie sie in ein Teflonrohr / einen Schrumpfschlauch.

    Die Temperatur des Heizers kann 140 °C erreichen.

![Montage iHeater](../img/iHeater_5491.jpg)

### Verdrahtung

![Montage iHeater](../img/iHeater_5494.jpg)

### Installation von Aderendhülsen

![Montage iHeater](../img/iHeater_5496.jpg)

### Verschaltung

![Montage iHeater](../img/iHeater_5498.jpg)

### Finale Montage

![Montage iHeater](../img/iHeater_5500.jpg)

### Fertiges Produkt

![Montage iHeater](../img/iHeater_5506.jpg)

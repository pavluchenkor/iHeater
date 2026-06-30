## iHeater-Firmware für Creality-Drucker über Creality Helper Script

Damit die Firmware und die Integration von iHeater erfolgreich verlaufen, folgen Sie der Schritt-für-Schritt-Anleitung:

### 1. Installieren Sie Creality Helper Script

Gehen Sie zur Dokumentationsseite des Projekts Creality Helper Script und folgen Sie der Anleitung zur Installation des Skripts.

**Ressourcen:**

* Videoanleitung: [YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* Textanleitung: [guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. Erhalten Sie root-Zugriff auf den Drucker und Zugriff auf das Dateisystem

Das Skript öffnet den Zugriff auf Mainsail sowie auf die Konfigurationsdateien der Firmware. Stellen Sie nach erfolgreicher Installation sicher, dass Sie die Oberfläche des Druckers über den Browser öffnen und auf die Konfigurationsdateien zugreifen können.

### 3. Entfernen Sie die alte `fan-control.cfg`

Auf Creality-Druckern mit Helper Script kann standardmäßig bereits eine Datei `fan-control.cfg` mit den Makros `M141` und `M191` erstellt worden sein. Sie steht im Konflikt mit ähnlichen Makros in der iHeater-Konfiguration.

Benennen Sie die Datei um:

```
/usr/data/printer_data/config/fan-control.cfg
```
in fan-control.cfg.bak

### 4. Kopieren Sie die neue `fan-control.cfg`

Ersetzen Sie sie durch die Version [fan-control.cfg](../../../printers/creality/config/fans-control.cfg), die mit den Makros und der Logik zur Steuerung der Kammertemperatur kompatibel ist.

Legen Sie die neue Datei im selben Ordner ab:

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. Fügen Sie die iHeater-Konfiguration hinzu

Kopieren Sie die Datei `iheater.cfg` in dasselbe Verzeichnis:

```
/usr/data/printer_data/config/iheater.cfg
```

Öffnen Sie anschließend `printer.cfg` und fügen Sie am Ende der Datei die folgende Zeile hinzu:

```ini
[include iheater.cfg]
```

---

Folgen Sie anschließend der Anleitung zur Einrichtung von iHeater - Konfiguration des Thermistors, des Heizelements, der Betriebsmodi und der Makros.

!!! warning "Wenn es keine Möglichkeit gibt, die Firmware auf dem Drucker zu bauen und zu flashen"
    [Siehe Abschnitt WSL](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

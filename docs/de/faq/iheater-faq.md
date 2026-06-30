# iHeater FAQ

---

### **Ist iHeater mit meinem Drucker kompatibel?**
iHeater ist mit allen Druckern unter **aktueller Klipper-Version** kompatibel und kann auch eigenständig auf **Standalone-Firmware** laufen.

---

### **Welche Heizleistung wird verwendet?**

- Im Lieferumfang enthalten ist ein Heizelement mit **200 W**.
- Modifikationen mit **100 W, 70 W und 50 W** sind möglich - diese werden hauptsächlich für **SLS-Drucker** mit kleineren Kammern verwendet, bei denen keine hohen Temperaturen erforderlich sind. Solche Modifikationen sind bewährt im Einsatz.

---

### **Welcher Controller wird in iHeater verwendet?**
Es wird **STM32F042** (aktuelle Version) verwendet.

---

### **Aus welchem Material sollte das Gehäuse gedruckt werden?**
Es wird empfohlen, **ABS/ASA** oder temperaturbeständigere Materialien zu verwenden.

---

### **Schmilzt das Gehäuse bei 130 °C?**

- Die Temperatur wird im Konfigurationsprogramm von einem Thermistor gemessen, der sich in der Nähe des Heizelements befindet,
- an den Rändern des Aluminiumrohrs ist die Temperatur aufgrund der Belüftung deutlich niedriger.
- Es sind **Abstandsschrauben** vorgesehen, um die Wärmbelastung zu reduzieren.

⚠️ Beim ersten Start müssen die Betriebsgrenzen überprüft werden:

- Erhöhen Sie die Temperatur schrittweise (um 5-10 °C).
- Überprüfen Sie das Gehäuse mit einem harten Werkzeug.
- Wenn der Kunststoff weich wird - die Temperatur ist zu hoch und muss gesenkt werden.

---

### **Welche typischen Probleme können auftreten?**
Falls der Heizer in **unkontrolliertes Aufheizen** verfällt, überprüfen Sie die **richtige Installation des Thermistors**.
Der Controller trifft Entscheidungen nur auf Grundlage der Thermistor-Messwerte. Ein falsch eingebauter Thermistor führt zu fehlender korrekter Rückmeldung und folglich zu Überhitzung und Beschädigung des Gehäuses. Die Beschädigung des Gehäuses bedeutet zeitaufwendige Neudrucke, daher muss die Installation des Thermistors genau nach Anleitung erfolgen.

---

### **Wie sieht es mit der Sicherheit aus?**
Das Sicherheitssystem ist mehrstufig aufgebaut:

- **Softwaresteuerung**: Der Heizer wird anhand der Thermistor-Messwerte gesteuert. Bei Überhitzung schaltet das Programm die Heizung ab.
- **Watchdog-Timer**: Eingebaut im Mikrocontroller kontrolliert die Firmware-Funktion und verhindert Einfrierungen.
- **Hardwareschutz**: Es wird ein Thermoschutzschalter verwendet. Während der Tests kann **KSD-9700** verwendet werden, das die Schaltung bei Überhitzung unterbricht und sich wieder schließt, wenn es um etwa 20 °C abkühlt. Für den Dauerbetrieb wird empfohlen, ihn gegen **RH-135** auszutauschen (löst bei 135 °C aus und unterbricht die Schaltung dauerhaft).
- Auf der Controller-Platine ist auch eine **Sicherung** installiert, die das Gerät bei Kurzschluss schützt.

---

### **Was ist diese kleine blaue Platine in der Verpackung?**
Das ist ein **220 V**-Netzteil - eine Test-Option. Die iHeater-Platine ist bereits für ihre Verwendung vorbereitet.
⚠️ Sie sollte nur installiert werden, wenn Sie ausreichend qualifiziert sind und die Vorgänge verstehen.
Wenn Sie diese Platine angeschlossen haben, **geben Sie auf keinen Fall Strom über USB**. Diese Platine ist ausschließlich für den **Standalone-Betrieb** gedacht.

---

### **Welche Stromversorgung ist erforderlich?**
iHeater verbraucht etwa **100 mA**. Es gibt keine zusätzlichen Anforderungen.

---

### **Kann iHeater von der Boardnetzversorgung des Druckers (110/220 V) versorgt werden?**
Ja, diese Verwendung ist vorgesehen. Sie können iHeater so anschließen, dass die Stromversorgung des Druckers und iHeater gleichzeitig ein- und ausgeschaltet werden - dazu können Sie iHeater nach dem Schalter Ihres Druckers anschließen.
Darüber hinaus ist im iHeater-Modell ein spezieller Adapter vorgesehen, der anstelle der **C7/C8**-Anschlüsse installiert wird und die direkte Verbindung des Stromkabels zu den Schraubklemmen auf der iHeater-Platine ermöglicht.

---

### **Wo finde ich die Dokumentation?**
Die Dokumentation ist hier verfügbar:

- [GitHub-Projekt](https://github.com/idryer)
- [docs.idryer.org](https://docs.idryer.org)

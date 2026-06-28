# Beheizung der Thermokammer im 3D-Drucker

## 1. Prüfung der Lüfter und Lüftungsöffnungen

* In neuen Druckermodellen werden häufig Lüfter installiert, um heiße Luft aus der Kammer auszublasen. Stellen Sie vor der Inbetriebnahme sicher, dass sie sich nicht automatisch einschalten, wenn die Temperatur in der Kammer überschritten wird..
* Prüfen Sie die Spalte im Druckergehäuse und rund um die Tür
* Prüfen Sie die Lüftungsöffnungen des Gehäuses. Wenn die Kammer Kontakt zum Elektronikfach hat und dort unverschlossene Öffnungen vorhanden sind, müssen diese geschlossen werden:

    * mit normalem Klebeband;
    * mit Aluminiumband (reflektiert Wärme besser);
    * oder mit Wärmedämmung (beste Option).
* Dies ist notwendig, damit die Steuerelektronik nicht überhitzt.


## 2. Wärmequelle: Druckbett

* Das Heizbett ist die wichtigste Wärmequelle für die Kammer.
* iHeater allein erhöht die Temperatur in der Regel nicht ohne Druckbett auf den erforderlichen Wert.
* Wenn der Bauraum des Druckers ohne eingeschaltetes Druckbett aufgeheizt werden soll, verwenden Sie industriell gefertigte Heizgeräte mit einer Leistung von 600 W - 1 kW, um die fehlende Wärme des Druckbetts zu kompensieren (**mit Prüfung der elektrischen Sicherheit und der Belastbarkeit der Stromversorgungskreise**).

## 3. Verwendung zusätzlicher Lüfter

* In modernen Modellen gibt es häufig zusätzliche Lüfter entlang der Seitenwände, die für eine zusätzliche Kühlung des Modells vorgesehen sind, oder Kohlefilter innerhalb der Kammer.
* Während der Aufheizphase der Kammer können sie zum Durchmischen der Luft eingeschaltet werden - das beschleunigt und vereinheitlicht die Erwärmung.
* Optimal ist eine Makro-Logik: Beim Start des Aufheizens werden die Lüfter eingeschaltet, nach Erreichen der Zieltemperatur oder der ersten Druckschichten werden sie ausgeschaltet.

## 4. Wann der Druck gestartet werden sollte

* Beispiel: Zieltemperatur der Kammer - 60°C.
* Der Druck kann bei 50-55°C gestartet werden, da vor dem Start vorbereitende Vorgänge stattfinden: Erstellen der Druckbettkarte, Aufheizen und Reinigen der Düse, Ablegen der ersten Schichten.
* Diese Prozesse dauern einige Minuten; in dieser Zeit kann sich die Kammer dem Zielwert annähern.
* Nach dem Druck der ersten 2-3mm des Modells erreicht die Kammer in der Regel die erforderliche Temperatur und stabilisiert sich; dieser Parameter ist für jeden Drucker individuell

## 5. Installation des Thermistors

* Platzieren Sie den Temperatursensor (Thermistor) ungefähr auf Höhe des Druckkopfs.
* Er darf keine Teile des Druckergehäuses berühren, da er sonst deren Temperatur und nicht die Lufttemperatur misst.

## 6. Sicherheit und zusätzliche Empfehlungen

* Positionieren Sie iHeater so, dass ein gleichmäßiger Luftstrom gewährleistet ist und lokale Überhitzungen vermieden werden.
* Verwenden Sie eine Wärmedämmung des Gehäuses, um Wärmeverluste zu reduzieren.
* Die Versorgungsleitungen der Heizgeräte müssen die Strombelastung aushalten (Kabel, Steckverbinder, Sicherung).
* Die Temperaturmodi müssen zu den Materialien des Druckergehäuses und den Betriebsbedingungen passen.

---

## 7. Kurze Checkliste vor dem Start

* [ ] Lüfter funktionieren; Lüftungsöffnungen sind gereinigt.
* [ ] Das Elektronikfach ist vom heißen Kammervolumen isoliert.
* [ ] Eine zusätzliche Wärmequelle ist eingeschaltet.
* [ ] Lüfter zum Durchmischen der Luft werden während der Aufheizphase aktiviert.
* [ ] Der Temperatursensor ist auf Höhe des Druckkopfs installiert und berührt das Gehäuse nicht.
* [ ] Temperaturgrenzen und Notabschaltungen sind konfiguriert.

# Sicherheit

!!! danger "Arbeiten mit Netzspannung"
    Das Gerät enthält Komponenten, die unter 110–230 V Spannung stehen. Trennen Sie vor allen Arbeiten an der Elektrik die Stromversorgung. Stellen Sie vor dem ersten Einschalten sicher, dass alle Verbindungen zuverlässig isoliert sind.

Die Controller-Firmware — Klipper oder Standalone — bietet softwareseitigen Schutz:

- Temperaturüberwachung mithilfe von Thermistoren;
- Prüfung, ob die Temperatursensoren angeschlossen sind;
- Schutz vor Temperaturen außerhalb sicherer Grenzwerte;
- Verwendung von Timern für den Fall, dass das System hängen bleibt;
- automatische Abschaltung bei Fehlern der Sensoren oder des Controllers.

Zusätzlich ist ein hardwareseitiger Schutz umgesetzt:

Es ist ein Thermal Protector KSD9700 (135 °C) installiert, der bei Überhitzung die Stromversorgung des Heizelements physisch trennt. Wenn die Temperatur unter den Schwellwert sinkt, schließt das Gerät den Stromkreis automatisch wieder und stellt die Stromversorgung wieder her.

Der Controller ist mit einer 2-A-Sicherung ausgestattet, die das Gerät schützt; in einer Notfallsituation brennt sie durch und schaltet das System vollständig stromlos.

Es wird ein PTC-Heizelement mit vollständiger elektrischer Isolierung verwendet. Im Gegensatz zu den meisten Heizlösungen steht das Gehäuse des PTC-Heizelements nicht unter Spannung, wodurch das Risiko eines elektrischen Schlags bei der Installation und Wartung der 3D-Druckerkammer ausgeschlossen wird.

Dieses mehrstufige Schutzsystem macht iHeater zu einer sicheren Lösung für die aktive Beheizung von 3D-Druckerkammern, auch bei langem Dauerbetrieb.

!!! warning "Installation des Thermistors"
    Stellen Sie sicher, dass blanke Drahtabschnitte an der Basis des Thermistors nicht mit dem Metallgehäuse des Heizgeräts in Kontakt kommen. Isolieren Sie diese Bereiche bei Bedarf mit Kaptonband oder legen Sie sie in einen Teflonschlauch / Schrumpfschlauch.

    Die Temperatur des Heizgeräts kann 140 °C erreichen.

!!! danger "KSD9700 — kein finaler Schutz"
    KSD9700 (Thermal Protector) ist ein selbstzurücksetzendes Gerät: Bei Überhitzung öffnet es den Stromkreis, aber sobald die Temperatur unter den Schwellwert fällt, schließt es ihn automatisch wieder. Bei einem Defekt des Heizgeräts wird das Gerät zyklisch überhitzen und abkühlen, ohne dass irgendein Eingriff erfolgt. Das ist keine Notabschaltung, sondern ein endloser Überhitzungszyklus.

    Ersetzen Sie KSD9700 für den dauerhaften Betrieb durch eine einmalige Thermal Fuse (zum Beispiel **RH130**). Sie trennt den Stromkreis beim Auslösen dauerhaft — das Gerät wird stromlos geschaltet und bleibt bis zum Austausch in einem sicheren Zustand.

!!! note "Empfohlene Reihenfolge"
    Verwenden Sie KSD9700 während der Montage und Fehlersuche. Ersetzen Sie ihn nach der Funktionsprüfung durch eine Thermal Fuse.

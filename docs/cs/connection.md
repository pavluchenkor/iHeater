## Schéma zapojení

Kontrolér **iHeater** může pracovat jak jako součást systému **Klipper** (jako dodatečné MCU), tak samostatně — pod řízením vestavěného firmwaru **standalone**.

### Připojení pro provoz s Klipper

Pro správný provoz jako součást Klipper je nutné připojit:

* **USB kabel** k hlavnímu hostu (Host-MCU) - přes něj probíhá přenos dat a napájení 5 V;
* **Silové napájení 220 V / 110 V** - podle verze zařízení a typu topného tělesa;
* **Termistor topného tělesa** - pro kontrolu teploty topného prvku;
* **Termistor komory** - pro kontrolu teploty vzduchu v komoře tiskárny;
* **Port triggeru** - volitelné připojení, používá se pro automatické řízení externím signálem.

V provozním stavu je iHeater umístěn uvnitř komory 3D tiskárny.

!!! note annotate "Doporučuje se umístit termistor komory na úroveň tiskové hlavy, pokud možno - **nad stolem**."

![Schéma zapojení](../img/iHeater_pinout.png)

## Konfigurace GPIO

| Pin    | Alias       | Function                          |
|--------|-------------|-----------------------------------|
| PA0    | TH1         | Teplotní senzor komory            |
| PA1    | HEATER      | Řízení topného tělesa             |
| PA2    | FAN         | Řízení ventilátoru                |
| PA3    | TH0         | Teplotní senzor topného tělesa    |
| PA4    | MODE        | Tlačítko režimu                   |
| PA5    | LED3        | LED 3                             |
| PA6    | LED2        | LED 2                             |
| PA7    | LED1        | LED 1                             |
| PB1    | TH2         | Dodatečný teplotní senzor         |

---

### Použití v režimu standalone

V samostatném režimu jsou dostupné další funkce a způsoby připojení:

* **Port triggeru v režimu termistoru**
  Při připojení termistoru k portu triggeru a jeho umístění vedle topného prvku stolu lze zapnout automatické řízení:
  - při ohřevu stolu nad **45°C** - zapne se ohřev komory;
  - při poklesu teploty pod **85°C** - ohřev se vypne.

* **Napájení z externího zdroje 5 V**
  Pokud není možné použít USB napájení, lze napájení přivést přímo přes příslušný konektor.

* **Experimentální možnost**
  Připojení [zdroje napájení 5 V přímo na desku](https://sl.aliexpress.ru/p?key=OHtN3Xm).

!!! danger "Nepoužívejte současně USB a externí zdroj napájení"
    Současné připojení dvou zdrojů napájení je nepřípustné. Povede ke konfliktu zdrojů napájení, chybám v provozu zařízení a může poškodit vybavení.

![Připojení napájení](../img/IMG_6009.jpg)

---

### Obecné schéma zapojení

![Schéma zapojení](../img/iHeater_connection.png)

## Wiring Diagram

The **iHeater** controller can operate both as part of a **Klipper** system (as an additional MCU) or autonomously using the integrated **standalone** firmware.

### Connecting for use with Klipper

For proper operation within Klipper, connect the following:

* **USB cable** to the main host (Host-MCU) — provides data communication and 5V power supply.
* **Power supply 220V / 110V** — depending on device version and heater type.
* **Heater thermistor** — to monitor heater temperature.
* **Chamber thermistor** — to monitor air temperature inside the printer chamber.
* **Trigger port** — optional, used for automatic control from an external signal.

During operation, the iHeater is located inside the 3D printer chamber.
!!! note annotate "It is recommended to place the chamber thermistor at the level of the toolhead, preferably **above the heated bed**."

![Wiring Diagram](../img/iHeater_pinout.png)

## GPIO Configuration

| Pin | Alias  | Function                   |
| --- | ------ | -------------------------- |
| PA0 | TH1    | Chamber temperature sensor |
| PA1 | HEATER | Heater control             |
| PA2 | FAN    | Fan control                |
| PA3 | TH0    | Heater temperature sensor  |
| PA4 | MODE   | Mode button                |
| PA5 | LED3   | LED 3                      |
| PA6 | LED2   | LED 2                      |
| PA7 | LED1   | LED 1                      |
| PB1 | TH2    |Optional temperature sensor |


---

### Standalone mode usage

In standalone mode, additional functions and connection methods are available:

* **Trigger port in thermistor mode**
  By connecting a thermistor to the trigger port and placing it near the heated bed element, automatic control can be enabled:

  * Heater turns on when the bed heats above **45°C**.
  * Heater turns off when the temperature drops below **85°C**.

* **Power from an external 5V source**
  If USB power is unavailable, you can directly supply power through the corresponding connector.

* **Experimental option**
  Connect [5V power supply directly to the board](https://sl.aliexpress.ru/p?key=OHtN3Xm).

!!! danger "Do not use two power sources simultaneously"
    Connecting two power sources at the same time is not allowed. This will cause a power conflict, result in device errors, and may damage the equipment.

![Power Connection](../img/IMG_6009.jpg)

---

### General Wiring Diagram

![Wiring Diagram](../img/iHeater_connection.png)

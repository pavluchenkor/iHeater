# About iHeater

iHeater is a compact heater for creating an active thermal chamber in a 3D printer. It is especially useful for printers with closed or proprietary electronics — Creality, Bambu Lab, FlashForge — where there are no free connectors for a heater, fan, and thermistor.

Connects via USB and operates independently of the mainboard's limitations. Depending on the firmware — with full Klipper integration or as a standalone device.

Combined with bed heating, iHeater ensures uniform chamber heating — a key factor when printing ABS, PA, PC, and other engineering plastics. The device dynamically controls heating based on air temperature, maintaining stable conditions inside the chamber without overheating or temperature spikes.

Two versions are available:

- 100W — for smaller printers (archived)
- 200W — for larger printers

![iHeater](../img/iHeater_promo.png)

## Usage Options

### Under Klipper Control

The board operates as a separate MCU in Klipper, fully autonomously controlling the chamber heater and fan. The 220V power supply does not load the printer's PSU — stock PSUs often operate at their limit.

![PCB](../img/iHeater_200_PCB.png)

The cost of the board is comparable to or lower than building a similar solution from a microcontroller, solid-state relay, and required components. Enthusiasts can still build their own equivalent.

### With iHeater Firmware

The iHeater board is self-sufficient and includes all the necessary peripherals for use as a standalone device. The target temperature is set by pressing the MODE button sequentially and is displayed using three LEDs.

## License

This project is licensed under the MIT License. See the [LICENSE](license.md) file for details.

!!! danger "Working with heating elements"
    The use of heating elements and temperature control involves a risk of fire and equipment damage. Take all necessary precautions. See the [Safety](safety.md) section for details.

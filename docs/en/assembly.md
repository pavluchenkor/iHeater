# Assembly

Review the documentation, download and print the necessary parts. Ensure all components and tools are available before starting.

!!! danger "Working with mains voltage"
    All work involving connection to 110–230 V mains must be performed with the device de-energized. See the [Safety](safety.md) section for details.

## Before Assembly

It is recommended to first assemble the entire system **on the table**, without mounting it into the enclosure, and perform testing:

- Connect **all** components.
- Verify the functionality of the heater, fan, and temperature sensors.
- Connect the system to **Klipper** or flash the Standalone firmware and ensure correct operation.

Video guide: [YouTube](https://youtu.be/1QMtVY0Vx-8?si=Ol1u4Ux9wALDcfe2)

## Step-by-Step Assembly

### Installing the Board

![iHeater Assembly](../img/iHeater_5484.jpg)

### Installing the Thermistor and Thermal Protector

!!! warning "Thermistor installation"
    Ensure that exposed wire sections at the thermistor base do not touch the metal body of the heater. If necessary, insulate these sections with Kapton tape or place them in a Teflon tube or heat-shrink tubing.

    The heater temperature can reach 140 °C.

!!! warning "Thermal Protector installation"
    You can install either a KSD9700 (Thermal Protector, self-resetting) or a one-time Thermal Fuse.

    The KSD9700 opens the circuit on overheating and automatically closes it when cooled. A Thermal Fuse (e.g. **RH130**) permanently breaks the circuit when triggered — more reliable protection in case of failure.

    Use the KSD9700 during testing, then replace it with a Thermal Fuse for permanent operation.

![iHeater Assembly](../img/iHeater_5489.jpg)
![iHeater Assembly](../img/thermistor.jpg)

### Installing the Heater

!!! warning "Thermistor installation"
    Place the thermistor at the edge of the heater, approximately at mid-height of the heatsink fins.

    Exposed wire sections at the thermistor base must not touch the metal body of the heater. If necessary, insulate these sections with Kapton tape or place them in a Teflon tube or heat-shrink tubing.

    The heater temperature can reach 140 °C.

![iHeater Assembly](../img/iHeater_5491.jpg)

### Wiring

![iHeater Assembly](../img/iHeater_5494.jpg)

### Installing Wire Ferrules

![iHeater Assembly](../img/iHeater_5496.jpg)

### Connections

![iHeater Assembly](../img/iHeater_5498.jpg)

### Final Assembly

![iHeater Assembly](../img/iHeater_5500.jpg)

### Finished Product

![iHeater Assembly](../img/iHeater_5506.jpg)

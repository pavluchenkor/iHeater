# Safety

!!! danger "Working with mains voltage"
    The device contains components operating at 110–230 V. Disconnect power before any electrical work. Ensure all connections are properly insulated before the first power-on.

The firmware — Klipper or Standalone — provides software protection:

- Temperature monitoring using thermistors;
- Verification that temperature sensors are connected;
- Protection against temperature exceeding safe limits;
- Watchdog timers in case of system hang;
- Automatic shutdown on sensor or controller errors.

Hardware protection is also implemented:

A Thermal Protector KSD9700 (135 °C) is installed. In the event of overheating, it physically disconnects power to the heating element. When the temperature drops below the threshold, the device automatically closes the circuit, restoring power.

The controller is equipped with a 2A fuse. In an emergency, it blows and completely de-energizes the system.

A PTC heating element with full electrical insulation is used. Unlike most heating solutions, the PTC heater body is not energized, eliminating the risk of electric shock during installation and chamber maintenance.

This multi-layer protection system makes iHeater a safe solution for active heating of 3D printer chambers, including during extended continuous operation.

!!! warning "Thermistor installation"
    Ensure that exposed wire sections at the thermistor base do not touch the metal body of the heater. If necessary, insulate these sections with Kapton tape or place them in a Teflon tube or heat-shrink tubing.

    The heater temperature can reach 140 °C.

!!! danger "KSD9700 is not the final protection"
    KSD9700 (Thermal Protector) is a self-resetting device: it opens the circuit on overheating, but as soon as the temperature drops below the threshold — it closes again automatically. If the heater malfunctions, the device will cyclically overheat and cool down without any intervention. This is not an emergency shutdown — it is an infinite overheating cycle.

    For permanent use, replace the KSD9700 with a one-time Thermal Fuse (e.g. **RH130**). It permanently breaks the circuit when triggered — the device is de-energized and remains in a safe state until replaced.

!!! note "Recommended order"
    Use KSD9700 during assembly and testing. After verifying correct operation, replace it with a Thermal Fuse.

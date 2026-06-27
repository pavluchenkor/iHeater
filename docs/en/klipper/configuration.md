# Klipper Configuration

This page describes the installation of iHeater configuration files and setup for use with Klipper.

## Requirements

### Hardware

- iHeater control board
- NTC 100K 3950 thermistors (2 pcs)
- PTC heating element 220V 100W, for the chamber
- 7530 220V fan, for air circulation inside the chamber
- Thermal Protector KSD9700 or equivalent (220V, 5A, 130 °C)

### Software

- Klipper (latest version)
- Configured and running Klipper host

## Klipper Configuration

Copy the `iHeater.cfg` configuration file into the directory containing `printer.cfg` (e.g. `/klipper_config`) and include it using the `[include]` directive:

```bash
cd ~/klipper_config
```

```bash
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

Open `printer.cfg` and add:

    [include iHeater.cfg]

## Connecting the iHeater MCU

Edit `iHeater.cfg` and specify the serial ID of your board:

```ini
[mcu iHeater]
serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ
```

## Preparation for Use

The configuration file includes the following section:

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # Target chamber temperature, °C
variable_start_offset: 10           # Chamber temperature sufficient for starting the print, °C
variable_delta_temp: 10             # Difference between chamber temperature and heater temperature, °C
variable_min_heater_temp: 50        # Minimum heater temperature (for cooling), °C
variable_max_heater_temp: 100       # Maximum heater temperature, °C
variable_control_interval: 1.0      # Control function call interval, seconds
variable_air_min_delta: 0.5         # Minimum difference between target and current chamber temperature (heater = target + delta_temp), °C
variable_air_max_delta: 5.0         # Maximum difference between target and current chamber temperature (heater = max_heater_temp), °C
gcode:
```

**The maximum allowable heater temperature depends on the enclosure material.**

To verify:

1. Set the heated bed to 90–100 °C.
2. Set the heater temperature to 100 °C via the Fluidd or Mainsail interface.
3. Ensure the iHeater is inside the printer's enclosed volume.
4. After reaching the set temperature, inspect areas where the heater contacts plastic parts of the enclosure. The plastic must not soften.
5. Increase the temperature by 5–10 °C and repeat the inspection.
6. Continue until the maximum safe heater temperature is identified without risk of enclosure deformation.

This approach allows you to determine the safe temperature maximum and achieve the best efficiency for iHeater.

## Usage

### Chamber Heater Control Commands

- Set chamber temperature:

        M141 S60  ; Sets the chamber temperature to 60°C

- Wait for the chamber to reach temperature:

        M191 S60  ; Waits until the chamber temperature reaches 60°C

- Turn off chamber heating:

        iHEATER_OFF   ; Turns off the chamber heater

- Add `iHEATER_OFF` to the end of your slicer's end G-code to correctly disable chamber heating.

### Start G-code

Modern slicers support automatic activation of an active thermal chamber during G-code generation. To enable this, specify the chamber temperature in the filament properties. If the slicer does not support this, add the chamber heating command manually to the start G-code.

Procedure:

- Set the target chamber temperature.
- Enable bed heating to efficiently and quickly heat the chamber.
- Continue with the standard print start G-code.

Example start G-code:

```
; --- Start of start G-code ---

; ****** iHeater Start ******
M141 S60       ; Set chamber temperature to 60°C
; ****** iHeater End ******

; --- Remaining start G-code ---
; Enable bed heating
...
```

!!! warning "To ensure proper shutdown of the iHeater control macro, add the `iHEATER_OFF` command to the printer's end G-code."

```
; --- Start of end G-code ---

; ****** iHeater Start ******
iHEATER_OFF
; ****** iHeater End ******

; --- Remaining end G-code ---
...
```

## Disable

To disable iHeater, comment out the `[include iHeater.cfg]` line in `printer.cfg`:

```
# [include iHeater.cfg]
```

Then remove the corresponding lines from the start and end G-code.

## Notes

- **Safety:**
    - Ensure all connections are correct and safe.
    - Verify that `min_temp` and `max_temp` values match the hardware specifications.

- **Hardware check:**
    - Test heater and fan operation before use.
    - Monitor temperature during the first runs.

- **PID tuning:**
    - Perform PID calibration if precise temperature control is required.

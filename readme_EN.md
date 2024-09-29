# iHeater configuration for Klipper

This repository contains configuration files for the iHeater 3D printer camera heater based on the Klipper firmware and the control board of the same name. The configuration is designed to control the heating of the camera and fans using the iHeater microcontroller.

## Table of Contents

- [Requirements](#requirements)
- [Preparation](#preparation)
- [Installing firmware on iHeater](#installing-firmware-on-iheater)
- [Klipper configuration](#configuration-klipper)
- [Connecting MCU `iHeater`](#1-connecting-mcu-iheater)
  - [Heater setting `iHeater_H'](#2-heater setting-iheater_h)
- [Fan setting `iHeater_F`](#3-fan setting-iheater_f)
- [Temperature sensor setting](#4-temperature sensor setting)
- [G-code macros](#macros-g-code)
- [Usage](#usage)
- [Camera heating control commands](#camera heating control commands)
- [Automation and control logic](#automation and control logic)
- [Notes](#notes)
- [License](#license)

## Requirements

- **Hardware:**
- iHeater control Board
  - Thermistors NTC 100K MGB18-104F39050L32 (2 pcs.)
- PTC heating element 220V 100W, for camera
  - 7530 220V fan, for air circulation in the chamber

- **Software:**
- Klipper (latest version)
- Configured and working 3D printer with Klipper

## Preparation

1. **Hardware assembly:**
- Connect the heating element and fans to the iHeater.
   - Install the thermistors in the camera and connect them to the appropriate pins of the MCU.
   - Make sure that the pins are connected correctly according to the configuration file.

2. **Installation of the necessary files:**
- Copy the file `rp2040_pin_aliases.cfg` and `iHeaterMCU.cfg` to the Klipper configuration directory.

## Installing firmware on iHeater

1. **Assemble the Klipper firmware for RP2040:**

   ```bash
   cd klipper/
   make menuconfig


**In the configuration menu, select:**

 - Micro-controller Architecture: RP2040
 - Processor model: rp2040
 - Leave the rest of the settings as default.

**Save and exit the menu.**

2. **Compile the firmware:**

        make

3. ##Installing the firmware on the iHeter board:##

- Connect the Iheater to the computer in programming mode (by holding the BOOTSEL button when connected).

- Mount the device and download the firmware:

        sudo mount /dev/sda1 /mnt
        sudo cp out/klipper.uf2 /mnt
        sudo umount /mnt

> **Note:** Replace /dev/sda1 with the appropriate path to your device.

## Klipper Configuration

Copy the iHeaterMCU.cfg and rp2040_pin_aliases.cfg configuration files to the printer.cfg folder and enable it using the [include] directive.
        
        [include iHeater.cfg]

###1. Connecting the iHeater MCU

        [mcu iHeater]
        serial: /dev/serial/by-id/usb-Klipper_rp2040_DE63581213745233-if00

- Description:
- Connects an additional iHeater microcontroller to the specified serial port.


###2. Setting up the iHeater_H heater

    [heater_generic iHeater_H]
    heater_pin: iHeater:H0
    max_power: 1
    sensor: iHeater_Sens_H
    control: pid
    pwm_cycle_time: 0.3
    min_temp: 0
    max_temp: 120
    pid_Kp=32.923
    pid_Ki=5.628
    pid_Kd=48.150

    [verify_heater iHeater_H]
    max_error: 240
    check_gain_time: 120
    heating_gain: 1

- Description:
- heater_generic iHeater_H: Adjusts the camera heater.
heater_pin: The pin to which the heating element is connected.
    - sensor: Uses the iHeater_Sens_H temperature sensor.
    - PID parameters for precise temperature control.
-verify_heater iHeater_H: Parameters for checking the heater for safety.

###3. Setting up the iHeater_F fan

    [fan_generic iHeater_F1]
    pin: iHeater:FAN0
    max_power: 1.0
    shutdown_speed: 0

- Description:
    
    - Adjusts the fan for air circulation in the chamber.
    - Controlled by macros depending on the temperature.

### 4. Setting up temperature sensors

    [temperature_sensor iHeater_Sens_C]
    sensor_pin: iHeater:T0
    sensor_type: NTC 100K MGB18-104F39050L32

    [temperature_sensor iHeater_Sens_H]
    sensor_pin: iHeater:T1
    sensor_type: NTC 100K MGB18-104F39050L32

- Description:
- iHeater_Sens_C: Camera temperature sensor.
    - iHeater_Sens_H: Heater temperature sensor.


### 5. G-code macros
Overriding the M141 and M191 commands

    [gcode_macro M141]
    rename_existing: M141.1
    gcode:
        M141.1 S{params.S|0}
        UPDATE_DELAYED_GCODE ID=_iHEATER_CONTROL DURATION=0

    [gcode_macro M191]
    gcode:
        M191.1 S{params.S|0}
        UPDATE_DELAYED_GCODE ID=_iHEATER_CONTROL DURATION=0

- Description:
- Override the standard commands for controlling the heating of the camera.
    - When called, the _iHEATER_CONTROL macro is run.

Heating and fan control macro

    [delayed_gcode _iHEATER_CONTROL]
    gcode:
        {% set current_temp = printer.heater_generic.iHeater_H.temperature %}
        {% set target_temp = printer.heater_generic.iHeater_H.target %}
        {% if target_temp > 0 %}
            # Fan control based on camera temperature
            {% set chamber_temp = printer.temperature_sensor.iHeater_Sens_C.temperature %}
            {% if chamber_temp >= 50 %}
                SET_FAN_SPEED FAN=iHeater_F SPEED=1.0
            {% else %}
                SET_FAN_SPEED FAN=iHeater_F SPEED=0.0
            {% endif %}
            # Restart the macro after 1 second
            UPDATE_DELAYED_GCODE ID=_iHEATER_CONTROL DURATION=1
        {% else %}
            # Stop the heater and fan
            SET_HEATER_TEMPERATURE HEATER=iHeater_H TARGET=0
            SET_FAN_SPEED FAN=IHEATER_FEED=0.0
RESPONSE prefix="iHeater_control" msg="Camera heating stopped."
        {% endif %}

- Description:
- Monitors the temperature and controls the fan depending on the set logic.
    - The fan turns on when the chamber temperature reaches 50°C.

## Usage
### Camera heating control commands
- Setting the camera temperature:
 

        M141 S60 ; Sets the camera temperature to 60°C

- Waiting for the temperature to reach:

        M191 S60 ; Waits until the camera temperature reaches 60°C

- Stopping the heating of the chamber:

        M141 S0 ; Turns off the heating of the camera


## Automation and control logic
The _iHEATER_CONTROL macro automatically controls the fan depending on the temperature of the camera.
When the temperature reaches 50°C, the iHeater_F fan turns on.
Macros can be customized to your requirements by changing the thresholds and logic.


## Notes
- Security:

    - Make sure that all connections are made correctly and securely.
    - Check that the min_temp and max_temp values match the hardware specifications.

- Checking the equipment:
- Before using, test the operation of the heater and fan.
    - Keep an eye on the temperature during the first launches.
- Setting up the PID:
    - If necessary, perform PID calibration for accurate temperature control.


## License
This project is distributed under the MIT license. For details, see the LICENSE file.


>** Attention: The use of heating elements and temperature control is associated with the risk of fire and damage to the equipment. Always follow the manufacturer's recommendations and take precautions.
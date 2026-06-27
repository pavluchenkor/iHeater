# iHeater Integration via Klipper and Home Assistant

This guide explains how to run the **iHeater** as a standalone Klipper MCU (without a printer mainboard) and control the chamber temperature via **Home Assistant** based on the active print material.

The setup uses:

- **Klipper** running on a Raspberry Pi
- **Moonraker** as API layer
- **Mainsail** for Klipper management
- **Home Assistant** for automation and material-based temperature control

The repository already contains the required configuration files:

- `printer.cfg`
- `iHeater.cfg`

# Prerequisites

Before starting, ensure you have:

- Raspberry Pi (recommended: Pi 3 or newer)
- Raspberry Pi OS or OS Lite (64-bit)
- iHeater board
- USB cable
- Home Assistant instance
- Network access between Home Assistant and the Raspberry Pi


## 1. Raspberry Pi Setup with KIAUH

1. Flash **Raspberry Pi OS (Lite) 64-bit** to an SD card.
2. Boot the Pi, connect via SSH, and install KIAUH:
   ```bash
   sudo apt-get update && sudo apt-get install git -y
   git clone https://github.com/dw-0/kiauh.git
   ./kiauh/kiauh.sh
   ```
3. Use the KIAUH menu to install:
   - **1) Klipper**
   - **2) Moonraker**
   - **3) Mainsail**

## 2. Configure Klipper Firmware

Compile the firmware for the STM32F042 microcontroller.

1. Connect the iHeater to a USB port of your Raspberry Pi.
   Verify that the device is detected:
   ```bash
   ls /dev/serial/by-id/
   ```
   You should see a device similar to:
   ```bash
   /dev/serial/by-id/usb-Klipper_stm32f042...
   ```
   
2. Compile Klipper Firmware
    Navigate to the Klipper directory:
   ```bash
   cd ~/klipper
   make menuconfig
   ```
3. Configure the firmware according to the official iHeater documentation:
   https://github.com/zanex88/iHeater/blob/main/flashing.en.md


## 3. Flash STM32 Firmware

1. See [Flashing Firmware](https://github.com/zanex88/iHeater/blob/main/flashing.en.md)
2. Retrieve the serial path for the Klipper configuration:
   ```bash
   ls /dev/serial/by-id/
   ```
   *Copy the output string (e.g., `/dev/serial/by-id/usb-Klipper_stm32f042...`).*

## 4. Adjust Klipper Configs

Open the Mainsail web interface (via the Pi's IP address) and edit the configuration files (Menu left side "Machine").
Upload the configuration files from this repository. USE your MCU serial in printer.cfg.
* `printer.cfg`
* `iHeater.cfg`

Save and click **FIRMWARE RESTART**.

## 5. Home Assistant Integration

1. In Home Assistant, go to **Settings -> Devices & Services -> Add Integration**.
2. Search for **Moonraker** and enter the IP address of your Raspberry Pi (default port: 7125).
3. **Important:** The default `climate` entity exposed by Moonraker controls the hardware heater block (`iHeater_H`), not the chamber temperature macro. To control the chamber properly via `M141`, add a REST command to your `configuration.yaml`:

```yaml
rest_command:
  iheater_set_temp:
    url: "http://<IP_OF_YOUR_PI>:7125/printer/gcode/script"
    method: POST
    payload: '{"script": "M141 S{{ temp }}"}'
    content_type: "application/json"
```
*Restart Home Assistant after adding this.*

4. Create a Number Helper (`input_number.setpoint_iheater`) in Home Assistant to set the target temperature manually or via automations.

## 6. Home Assistant Automation 

This automation controls the iHeater automatically based on the Bambu Lab bed target temperature and the active material slot. 

* **Trigger:** Bed target temperature changes.
* **Logic:** * If Bed > 0: Checks the material string. If unknown/empty, sets a 35°C pre-heat. Once the material is identified, it applies the specific temperature.
  * If Bed = 0: Shuts down the iHeater.

```yaml
alias: "iHeater: Auto-Start & Material-Update"
mode: restart
trigger:
  - platform: state
    entity_id: number.p1s_01p09c541801823_zieltemperatur_des_druckbett
  - platform: state
    entity_id: sensor.p1s_01p09c541801823_aktiver_slot
action:
  - action: input_number.set_value
    target:
      entity_id: input_number.setpoint_iheater
    data:
      value: >
        {% set bed_target = states('number.p1s_01p09c541801823_zieltemperatur_des_druckbett') | float(0) %}
        {% set mat = states('sensor.p1s_01p09c541801823_aktiver_slot') | lower %}
        
        {% if bed_target > 0 %}
          {% if 'pla' in mat %}
            30
          {% elif 'petg' in mat %}
            45
          {% elif 'abs' in mat or 'asa' in mat %}
            65
          {% elif 'pa' in mat or 'nylon' in mat %}
            65
          {% elif 'pc' in mat %}
            75
          {% elif 'tpu' in mat %}
            0
          {% else %}
            35
          {% endif %}
        {% else %}
          0
        {% endif %}
  - action: rest_command.iheater_set_temp
    data:
      temp: "{{ states('input_number.setpoint_iheater') | int }}"
```
*(Ensure to replace the entity IDs with your actual Bambu Lab integration entity IDs).*

This automation controls the iHeater based on the change of the helper entity e.g. by using a slider

```yaml
alias: "iHeater: Send setpoint to Klipper"
description: ""
triggers:
  - entity_id: input_number.setpoint_iheater
    trigger: state
actions:
  - action: rest_command.iheater_set_temp
    data:
      temp: "{{ trigger.to_state.state | int }}"
mode: single
```
*(Ensure to replace the entity IDs with your actual Bambu Lab integration entity IDs).*

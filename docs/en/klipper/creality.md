## Flashing iHeater for Creality Printers via Creality Helper Script

To ensure successful flashing and integration of iHeater, follow the step-by-step guide below:

### 1. Install Creality Helper Script

Go to the Creality Helper Script documentation page and follow the installation instructions.

**Resources:**

* Video guide: [YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* Text instructions: [guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)

### 2. Gain root access to the printer and access the file system

The script will provide access to Mainsail as well as to the firmware configuration files. After successful installation, ensure that you can access the printer's interface via a web browser and access the configuration files.

### 3. Remove old `fan-control.cfg`

On Creality printers with the Helper Script, a `fan-control.cfg` file containing `M141` and `M191` macros may already exist by default. It conflicts with similar macros in the iHeater configuration.

Rename the file:

```
/usr/data/printer_data/config/fan-control.cfg
```

to fan-control.cfg.bak

### 4. Copy the new `fan-control.cfg`

Replace it with the [fan-control.cfg](../../../printers/creality/config/fans-control.cfg) version compatible with the chamber temperature control logic and macros.

Place the new file in the same folder:

```
/usr/data/printer_data/config/fan-control.cfg
```

### 5. Add the iHeater configuration

Copy the `iheater.cfg` file into the same directory:

```
/usr/data/printer_data/config/iheater.cfg
```

Then open `printer.cfg` and add the following line at the end of the file:

```ini
[include iheater.cfg]
```

---

Proceed with the iHeater setup guide - thermistor configuration, heater setup, operating modes, and macros.

!!! warning "If you cannot build and flash firmware on the printer"
    [Refer to the WSL section](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

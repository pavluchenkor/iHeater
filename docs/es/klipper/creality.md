## Firmware iHeater para impresoras Creality mediante Creality Helper Script

Para que el firmware y la integración de iHeater se realicen correctamente, siga las instrucciones paso a paso:

### 1. Instale Creality Helper Script

Vaya a la página de documentación del proyecto Creality Helper Script y siga las instrucciones de instalación del script.

**Recursos:**

* Videoguía: [YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* Instrucciones de texto: [guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. Obtenga acceso root a la impresora y acceso al sistema de archivos

El script abrirá el acceso a Mainsail, así como a los archivos de configuración del firmware. Después de la instalación correcta, asegúrese de que puede acceder a la interfaz de la impresora mediante el navegador y obtener acceso a los archivos de configuración.

### 3. Elimine el antiguo `fan-control.cfg`

En las impresoras Creality con Helper Script, de forma predeterminada ya puede estar creado el archivo `fan-control.cfg` con las macros `M141` y `M191`. Este entra en conflicto con las macros equivalentes en la configuración de iHeater.

Cambie el nombre del archivo:

```
/usr/data/printer_data/config/fan-control.cfg
```
a fan-control.cfg.bak

### 4. Copie el nuevo `fan-control.cfg`

Sustitúyalo por la versión [fan-control.cfg](../../../printers/creality/config/fans-control.cfg), compatible con las macros y la lógica de control de temperatura de la cámara.

Coloque el nuevo archivo en la misma carpeta:

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. Añada la configuración de iHeater

Copie el archivo `iheater.cfg` en el mismo directorio:

```
/usr/data/printer_data/config/iheater.cfg
```

Luego abra `printer.cfg` y añada la siguiente línea al final del archivo:

```ini
[include iheater.cfg]
```

---

A continuación, siga las instrucciones de configuración de iHeater: configuración del termistor, calentador, modos de funcionamiento y macros.

!!! warning "Si no es posible compilar y flashear el firmware en la impresora"
    [Consulte la sección WSL](../user-mods/software/wsl2-ubuntu-ff/)

# Configuración de Klipper

Esta página describe la instalación de los archivos de configuración de iHeater y la configuración del funcionamiento con Klipper.

## Requisitos

### Hardware
  - Placa de control iHeater
  - Termistores NTC 100K 3950 (2 uds.)
  - Elemento calefactor PTC 220V 100W, para la cámara
  - Ventilador 7530 220V, para la circulación de aire en la cámara
  - Thermal Protector KSD9700 o similar (220 V, 5 A, 130 °C)

### Software
  - Klipper (última versión)
  - Host configurado y funcionando con Klipper

## Configuración de Klipper


Copie los archivos de configuración iHeater.cfg en la carpeta con el archivo printer.cfg (puede ser /klipper_config) e inclúyalo en printer.cfg mediante la directiva [include]


```
cd ~/klipper_config
```

```
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

Abra printer.cfg y añada

    [include iHeater.cfg]

## Conexión del MCU iHeater

Modifique el archivo iHeater.cfg e indique el ID obtenido

```
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ

```

## Preparación para el uso

El archivo de configuración contiene la sección:

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # Temperatura objetivo de la cámara, °C
variable_start_offset: 10           # Temperatura de la cámara suficiente para iniciar la impresión, °C
variable_delta_temp: 10             # Diferencia entre la temperatura de la cámara y la del calentador, °C
variable_min_heater_temp: 50        # Temperatura mínima del calentador (para enfriamiento), °C
variable_max_heater_temp: 100       # Temperatura máxima del calentador, °C
variable_control_interval: 1.0      # Intervalo de llamada de la función de control, segundos
variable_air_min_delta: 0.5         # Diferencia mínima entre la temperatura objetivo y la actual de la cámara (calentador = objetivo + delta_temp), °C
variable_air_max_delta: 5.0         # Diferencia máxima entre la temperatura objetivo y la actual de la cámara (calentador = max_heater_temp), °C
gcode:
```

**La temperatura máxima permitida del calentador depende del material de la carcasa.**

Para comprobarlo:

!. Active el calentamiento de la cama a 90-100°C
1. Establezca la temperatura del calentador en 100°C mediante la interfaz Fluidd o Mainsail.
2. Asegúrese de que iHeater esté dentro del volumen cerrado de la impresora.
3. Después de alcanzar la temperatura establecida, compruebe las zonas donde el calentador entra en contacto con elementos plásticos de la carcasa. El plástico no debe ablandarse.
4. Aumente la temperatura en 5-10°C y repita la comprobación.
5. Repita el proceso hasta alcanzar la temperatura máxima permitida del calentador sin riesgo de deformar la carcasa.

Este enfoque permite determinar un máximo de temperatura seguro y lograr la mejor eficiencia de funcionamiento de iHeater.


## Uso

### Comandos de control del calentamiento de la cámara
- Establecer la temperatura de la cámara:
 

        M141 S60  ; Establece la temperatura de la cámara en 60°C

- Esperar a que se alcance la temperatura:

        M191 S60  ; Espera hasta que la temperatura de la cámara alcance 60°C

- Detener el calentamiento de la cámara:

        iHEATER_OFF   ; Desactiva el calentamiento de la cámara

- En el G-code final del slicer, añada `iHEATER_OFF` para desactivar correctamente el calentamiento de la cámara.

### G-code inicial

Los slicers modernos admiten la activación automática de la cámara térmica activa al generar el G-code de impresión. Para ello, en las propiedades del filamento debe indicar la temperatura de la cámara. Si el slicer no dispone de esta funcionalidad, en el G-code inicial debe añadir el comando de activación del calentamiento de la cámara térmica activa.

Procedimiento:

- Establecer la temperatura objetivo de la cámara
- Activar el calentamiento de la cama para calentar la cámara de forma eficiente y rápida 
- Continuar con el G-code inicial estándar de impresión

Ejemplo de G-code inicial
```
; --- Inicio del G-code inicial ---

; ****** Inicio iHeater ******
M141 S60       ; Establecer la temperatura de la cámara en 60°C
; ****** Fin del bloque iHeater ******

; --- Resto del G-code inicial ---
; Activación del calentamiento de la cama
...
```
!!! warning "Para finalizar correctamente el funcionamiento del macro de control de iHeater, debe añadir el comando iHEATER_OFF al G-code final de la impresora"

```
; --- Inicio del G-code final ---

; ****** Inicio del bloque iHeater ******
iHEATER_OFF
; ****** Fin del bloque iHeater ******

; --- Resto del G-code final ---
...
```
## Desactivación

Para desactivar iHeater en el archivo printer.cfg, debe comentar la línea [include iHeater.cfg]
```
# [include iHeater.cfg]
```

Y eliminar del G-code inicial y final las líneas correspondientes

## Notas
- Seguridad:

    - Asegúrese de que todas las conexiones estén realizadas de forma correcta y segura.
    - Compruebe que los valores min_temp y max_temp correspondan a las especificaciones del equipo.

- Comprobación del equipo:
    - Antes de usarlo, pruebe el funcionamiento del calentador y del ventilador.
    - Supervise la temperatura durante los primeros arranques.
- Ajuste PID:
    - Si es necesario, realice la calibración PID para un control preciso de la temperatura.

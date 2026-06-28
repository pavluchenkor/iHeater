## Esquema de conexión

El controlador **iHeater** puede funcionar tanto como parte del sistema **Klipper** (como MCU adicional) como de forma autónoma, bajo el control del firmware integrado **standalone**.

### Conexión para trabajar con Klipper

Para funcionar correctamente como parte de Klipper, es necesario conectar:

* **Cable USB** al host principal (Host-MCU): por él se realiza la transferencia de datos y la alimentación de 5V;
* **Alimentación de potencia 220V / 110V**: según la versión del dispositivo y el tipo de calentador;
* **Termistor del calentador**: para controlar la temperatura del elemento calefactor;
* **Termistor de la cámara**: para controlar la temperatura del aire en la cámara de la impresora;
* **Puerto de trigger**: conexión opcional, se utiliza para el control automático desde una señal externa.

En estado de funcionamiento, iHeater se coloca dentro de la cámara de la impresora 3D.

!!! note annotate "Se recomienda colocar el termistor de la cámara a la altura del cabezal de impresión, si es posible, **por encima de la cama**."

![Esquema de conexión](../img/iHeater_pinout.png)

## Configuración GPIO

| Pin    | Alias       | Function                          |
|--------|-------------|-----------------------------------|
| PA0    | TH1         | Sensor de temperatura de la cámara|
| PA1    | HEATER      | Control del calentador            |
| PA2    | FAN         | Control del ventilador            |
| PA3    | TH0         | Sensor de temperatura del calentador |
| PA4    | MODE        | Botón de modo                     |
| PA5    | LED3        | LED 3                             |
| PA6    | LED2        | LED 2                             |
| PA7    | LED1        | LED 1                             |
| PB1    | TH2         | Sensor de temperatura adicional   |

---

### Uso en modo standalone

En modo autónomo están disponibles funciones y métodos de conexión adicionales:

* **Puerto de trigger en modo termistor**
  Al conectar un termistor al puerto de trigger y colocarlo cerca del elemento calefactor de la cama, se puede activar el control automático:
  - cuando la cama se calienta por encima de **45°C**, se activa el calentamiento de la cámara;
  - cuando la temperatura baja por debajo de **85°C**, el calentamiento se desactiva.

* **Alimentación desde una fuente externa de 5V**
  Si no es posible utilizar la alimentación por USB, se puede suministrar alimentación directamente mediante el conector correspondiente.

* **Opción experimental**
  Conexión de una [fuente de alimentación de 5V directamente a la placa](https://sl.aliexpress.ru/p?key=OHtN3Xm).

!!! danger "No utilice simultáneamente USB y una fuente de alimentación externa"
    La conexión simultánea de dos fuentes de alimentación no está permitida. Esto provocará un conflicto entre las fuentes de alimentación, errores en el funcionamiento del dispositivo y puede dañar el equipo.

![Conexión de alimentación](../img/IMG_6009.jpg)

---

### Esquema general de conexión

![Esquema de conexión](../img/iHeater_connection.png)

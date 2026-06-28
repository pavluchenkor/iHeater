# Montaje

Lee la documentación, descarga e imprime las piezas necesarias. Asegúrate de tener todos los componentes y herramientas antes de comenzar el montaje.

!!! danger "Trabajo con tensión de red"
    Realiza todos los trabajos de conexión a la red de 110-230 V con el dispositivo sin alimentación. Más detalles en la sección [Seguridad](safety.md).

## Antes del montaje

Se recomienda montar primero todo el sistema **sobre la mesa**, sin instalarlo en la carcasa, y realizar las pruebas:

- Conectar **todos** los componentes.
- Comprobar el funcionamiento del calentador, el ventilador y los sensores de temperatura.
- Conectar el sistema a **Klipper** o flashearlo con el firmware Standalone y asegurarse de que funciona correctamente.

Video guide: [YouTube](https://youtu.be/1QMtVY0Vx-8?si=Ol1u4Ux9wALDcfe2)

## Montaje paso a paso

### Instalación de la placa

![Montaje de iHeater](../img/iHeater_5484.jpg)

### Instalación del termistor y Thermal Protector

!!! warning "Instalación del termistor"
    Asegúrate de que las zonas expuestas de los cables en la base del termistor no entren en contacto con la carcasa metálica del calentador. Si es necesario, aísla esas zonas con cinta Kapton o colócalas en un tubo de teflón / termorretráctil.

    La temperatura del calentador puede alcanzar 140 °C.

!!! warning "Instalación de Thermal Protector"
    Se puede instalar KSD9700 (Thermal Protector, autorrearmable) o un Thermal Fuse de un solo uso.

    KSD9700 abre el circuito en caso de sobrecalentamiento y lo cierra automáticamente al enfriarse. Thermal Fuse (por ejemplo, **RH130**) abre el circuito de forma permanente al activarse: es una protección más fiable en caso de fallo.

    Usa KSD9700 durante la etapa de depuración y luego sustitúyelo por Thermal Fuse para el uso permanente.

![Montaje de iHeater](../img/iHeater_5489.jpg)
![Montaje de iHeater](../img/thermistor.jpg)

### Instalación del calentador

!!! warning "Instalación del termistor"
    Instala el termistor cerca del borde del calentador, aproximadamente a media altura de las aletas del radiador.

    Las zonas expuestas de los cables en la base del termistor no deben tocar la carcasa metálica del calentador. Si es necesario, aísla esas zonas con cinta Kapton o colócalas en un tubo de teflón / termorretráctil.

    La temperatura del calentador puede alcanzar 140 °C.

![Montaje de iHeater](../img/iHeater_5491.jpg)

### Cableado

![Montaje de iHeater](../img/iHeater_5494.jpg)

### Instalación de punteras NShVI

![Montaje de iHeater](../img/iHeater_5496.jpg)

### Conmutación

![Montaje de iHeater](../img/iHeater_5498.jpg)

### Montaje final

![Montaje de iHeater](../img/iHeater_5500.jpg)

### Producto terminado

![Montaje de iHeater](../img/iHeater_5506.jpg)

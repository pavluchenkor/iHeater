# Seguridad

!!! danger "Trabajo con tensión de red"
    El dispositivo contiene componentes bajo tensión de 110-230 V. Antes de realizar cualquier trabajo eléctrico, desconecte la alimentación. Asegúrese de que todas las conexiones estén correctamente aisladas antes del primer encendido.

El firmware del controlador, Klipper o Standalone, proporciona protección por software:

- control de temperatura mediante termistores;
- comprobación de la conexión de los sensores de temperatura;
- protección contra temperaturas fuera de los valores seguros;
- uso de temporizadores en caso de bloqueo del sistema;
- apagado automático ante errores de los sensores o del controlador.

Además, se ha implementado protección por hardware:

Se instala un Thermal Protector KSD9700 (135 °C), que en caso de sobrecalentamiento desconecta físicamente la alimentación del elemento calefactor. Cuando la temperatura baja por debajo del valor umbral, el dispositivo cierra automáticamente el circuito y restablece la alimentación.

El controlador está equipado con un fusible de 2 A que protege el dispositivo; en una situación de emergencia se funde y deja todo el sistema sin alimentación.

Se utiliza un elemento calefactor PTC con aislamiento eléctrico completo. A diferencia de la mayoría de las soluciones de calefacción, la carcasa del calefactor PTC no está bajo tensión, lo que elimina el riesgo de descarga eléctrica durante la instalación y el mantenimiento de la cámara de una impresora 3D.

Este sistema de protección multinivel convierte a iHeater en una solución segura para el calentamiento activo de cámaras de impresoras 3D, incluso durante un funcionamiento continuo prolongado.

!!! warning "Instalación del termistor"
    Asegúrese de que las partes desnudas de los cables en la base del termistor no entren en contacto con la carcasa metálica del calefactor. Si es necesario, aísle estas zonas con cinta Kapton o colóquelas en un tubo de teflón / tubo termorretráctil.

    La temperatura del calefactor puede alcanzar 140 °C.

!!! danger "KSD9700: no es la protección final"
    KSD9700 (Thermal Protector) es un dispositivo autorrearmable: en caso de sobrecalentamiento abre el circuito, pero en cuanto la temperatura cae por debajo del umbral, vuelve a cerrarlo automáticamente. Si el calefactor falla, el dispositivo se sobrecalentará y enfriará cíclicamente sin ninguna intervención. Esto no es una desconexión de emergencia: es un ciclo infinito de sobrecalentamiento.

    Para uso permanente, sustituya el KSD9700 por un Thermal Fuse de un solo uso (por ejemplo, **RH130**). Este abre el circuito de forma permanente al activarse: el dispositivo queda sin alimentación y permanece en un estado seguro hasta su sustitución.

!!! note "Orden recomendado"
    Utilice KSD9700 durante la etapa de montaje y depuración. Después de comprobar el funcionamiento, sustitúyalo por un Thermal Fuse.

# Acerca del proyecto iHeater

iHeater es un calentador compacto para crear una cámara térmica activa en una impresora 3D. Es especialmente útil en modelos con electrónica cerrada o propietaria, como Creality, Bambu Lab y FlashForge, donde no hay conectores libres para conectar un calentador, un ventilador y un termistor.

Se conecta por USB y funciona independientemente de las limitaciones de la placa principal. Según el firmware, puede funcionar con integración completa con Klipper o de forma autónoma.

En combinación con el calentamiento de la cama, iHeater proporciona un calentamiento uniforme de la cámara, un factor clave al imprimir ABS, PA, PC y otros plásticos técnicos. El dispositivo controla dinámicamente el calentamiento según la temperatura del aire, creando condiciones estables dentro de la cámara sin sobrecalentamientos ni fluctuaciones.

Hay dos versiones disponibles:

- 100 W: para impresoras pequeñas (archivada)
- 200 W: para impresoras de mayor tamaño

![iHeater](../img/iHeater_promo.png)

[Puede realizar un cálculo preliminar con esta calculadora](https://docs.google.com/spreadsheets/d/1u6XrWLFZGOUnRlFPjjGsJB_GLuFFIs3fFWLCp2-K8gc/edit?usp=sharing)

## Casos de uso

### Bajo el control de Klipper

La placa funciona como un MCU independiente en Klipper, controlando de forma completamente autónoma el calentamiento de la cámara y el ventilador. La alimentación desde 220 V no carga la fuente de alimentación de la impresora; las fuentes estándar suelen funcionar al límite.

![PCB](../img/iHeater_200_PCB.png)

El coste de la placa es comparable o inferior al de montar por cuenta propia una solución similar basada en un microcontrolador, un relé de estado sólido y los componentes necesarios. Para los entusiastas, sigue existiendo la posibilidad de montar un análogo por su cuenta.

### Con firmware iHeater

La placa iHeater es autosuficiente e incluye toda la periferia necesaria para utilizarse como dispositivo independiente. La temperatura objetivo se establece mediante pulsaciones sucesivas del botón MODE y se muestra mediante tres LED.

## Licencia

El proyecto se distribuye bajo la licencia MIT. Más detalles en el archivo [LICENSE](license.md).

!!! danger "Trabajo con elementos calefactores"
    El uso de elementos calefactores y el control de temperatura conllevan riesgo de incendio y daños al equipo. Tome medidas de precaución. Más detalles en la sección [Seguridad](safety.md).

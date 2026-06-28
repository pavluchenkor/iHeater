# Calentamiento de la cámara térmica en una impresora 3D

## 1. Comprobación de los ventiladores y las aberturas de ventilación

* En los modelos nuevos de impresoras se instalan a menudo ventiladores para expulsar el aire caliente de la cámara. Antes de empezar a usarla, asegúrese de que no se enciendan automáticamente al superar la temperatura de la cámara..
* Revise las ranuras de la carcasa de la impresora y alrededor de la puerta
* Revise las aberturas de ventilación de la carcasa. Si la cámara tiene contacto con el compartimento de la electrónica y allí hay aberturas sin cerrar, deben cerrarse:

    * con cinta adhesiva normal;
    * con cinta de aluminio (refleja mejor el calor);
    * o con aislamiento térmico (la mejor opción).
* Esto es necesario para que la electrónica de control no se sobrecaliente.


## 2. Fuente de calor: la cama de la impresora

* La cama calefactora es la principal fuente de calor para la cámara.
* Por sí solo, iHeater normalmente no elevará la temperatura hasta la requerida sin la cama.
* Si se necesita calentar el volumen de la impresora sin la cama encendida, utilice calentadores de fabricación industrial con una potencia de 600 W - 1 kW para compensar la falta de calor de la cama (**con comprobación de la seguridad eléctrica y de la capacidad de carga de los circuitos de alimentación**).

## 3. Uso de ventiladores auxiliares

* En los modelos modernos suele haber ventiladores adicionales a lo largo de las paredes laterales, destinados a la ventilación adicional del modelo, o filtros de carbón dentro de la cámara.
* Durante la fase de calentamiento de la cámara se pueden encender para mezclar el aire; esto acelera y uniformiza el calentamiento.
* Lo óptimo es prever una macro-lógica: al iniciar el calentamiento, los ventiladores se encienden; después de alcanzar la temperatura objetivo o las capas iniciales de impresión, se apagan.

## 4. Cuándo empezar a imprimir

* Ejemplo: temperatura objetivo de la cámara: 60°C.
* La impresión puede empezar a 50-55°C, porque antes del inicio hay operaciones preparatorias: construcción del mapa de la cama, calentamiento y limpieza de la boquilla, deposición de las primeras capas.
* Estos procesos tardan varios minutos; durante ese tiempo la cámara alcanza aproximadamente el valor objetivo.
* Después de imprimir los primeros 2-3 mm del modelo, la cámara normalmente alcanza la temperatura necesaria y se estabiliza; este parámetro es individual para cada impresora

## 5. Instalación del termistor

* Coloque el sensor de temperatura (termistor) aproximadamente a la altura del cabezal de impresión.
* No debe tocar las piezas de la carcasa de la impresora; de lo contrario leerá su temperatura, no la temperatura del aire.

## 6. Seguridad y recomendaciones adicionales

* Coloque iHeater de modo que proporcione un flujo uniforme y evite sobrecalentamientos locales.
* Aplique aislamiento térmico a la carcasa para reducir las pérdidas de calor.
* Las líneas de alimentación de los calentadores deben soportar la carga de corriente (cable, conectores, fusible).
* Los modos de temperatura deben corresponder a los materiales de la carcasa de la impresora y a las condiciones de uso.

---

## 7. Lista breve de comprobación antes del arranque

* [ ] Los ventiladores funcionan correctamente; las aberturas de ventilación están limpias.
* [ ] El compartimento de la electrónica está aislado del volumen caliente de la cámara.
* [ ] La fuente de calor adicional está encendida.
* [ ] Los ventiladores para mezclar el aire se activan durante la fase de calentamiento.
* [ ] El sensor de temperatura está instalado a la altura del cabezal de impresión y no toca la carcasa.
* [ ] Los límites de temperatura y las desconexiones de emergencia están configurados.

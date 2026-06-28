## Configuración del G-code inicial en el slicer para el funcionamiento correcto de iHeater

Para lograr un calentamiento uniforme y una impresión estable con materiales técnicos, es importante configurar correctamente el orden de los comandos en el G-code inicial. A continuación se muestran recomendaciones para integrar el calentamiento de la cámara iHeater en el código inicial de su slicer (por ejemplo, Cura, PrusaSlicer, OrcaSlicer, etc.).

### Qué hay que hacer

Antes de activar el calentamiento de la cámara, es necesario:

**Activar el calentamiento de la cama**

   El calentamiento de la cama ayuda a que la cámara se caliente de forma más uniforme y rápida. La cama actúa como una fuente de calor adicional que contribuye a un calentamiento más eficiente de toda la cámara.

   ```
   M140 S[first_layer_bed_temperature]
   ```

**Activar el ventilador de mezcla de aire (si se utiliza)**

   Puede ser un ventilador lateral o cualquier otro destinado a distribuir el calor de forma uniforme en la cámara.
   Si en la configuración de Klipper se llama, por ejemplo, `chamber_fan`, entonces:

   ```
   SET_FAN_SPEED FAN=chamber_fan SPEED=1.0
   ```

En algunas impresoras hay ventiladores de extracción que mantienen la temperatura mínima posible en la cámara; esto es importante al imprimir con plásticos como PLA y PETG, pero perjudica la velocidad de calentamiento de la cámara. A este ventilador se le puede pasar un nuevo parámetro de temperatura, por ejemplo temperatura_objetivo_en_la_cámara + 10

```
SET_TEMPERATURE_FAN_TARGET TEMPERATURE_FAN=chamber_fan TARGET={chamber_temperature + 10}
```

**Activar el calentamiento de la cámara con uno de los macros: `M141` o `M191`**

### Diferencia entre `M141` y `M191`

| Macro  | Finalidad                                                                      | Bloquea la ejecución del código                                      |
| ------ | ------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| `M141` | Establece la temperatura objetivo de la cámara                                 | No (el calentamiento empieza, pero la impresión continúa de inmediato) |
| `M191` | Establece la temperatura de la cámara y espera hasta que alcance el valor dado | Sí (se pasará al siguiente comando solo después del calentamiento)    |

#### Ejemplos:

* Si quiere iniciar de inmediato el calentamiento de la cámara y realizar todas las operaciones preliminares en paralelo al calentamiento, y luego continuar la impresión sin esperar a que la cámara se caliente por completo, esta opción es adecuada. Funciona bien para plásticos tipo ABS y modelos grandes: mientras se imprimen las primeras capas, la cámara tendrá tiempo de calentarse:

  ```
  M141 S60
  ```

* Si se imprime una pieza pequeña o se requiere una temperatura estable de la cámara (por ejemplo, al imprimir PA, PC y otros materiales sensibles), es mejor usar el comando con espera de calentamiento:

  ```
  M191 S60
  ```

### Después del calentamiento de la cámara

Después de llamar a uno de los macros (`M141` o `M191`), se puede continuar con el G-code inicial habitual:

```gcode
M190 S[first_layer_bed_temperature]  Espera del calentamiento de la cama
M109 S[first_layer_temperature]  Espera del calentamiento del hotend
G28  Homing
G29  (si se utiliza autocalibración)
... 
```

### Recomendaciones

* Si utiliza `M191`, se puede definir una pequeña desviación con la que la cámara se considera suficientemente calentada (por ejemplo, 5°C por debajo del objetivo); esto se configura en el macro [gcode_macro CHAMBER_VARS] `variable_start_offset`.
* Asegúrese de que todos los macros utilizados (`M141`, `M191`) estén incluidos en la configuración de Klipper y configurados correctamente.
* Si su slicer admite condiciones, puede añadir una comprobación: por ejemplo, activar la cámara solo con una temperatura de impresión superior a 50°C (para ABS, ASA, etc.).

---

### Configuración de la temperatura en la cámara térmica en el slicer

![Slicer_settings](../img/slicer_settings.jpeg)

En muchos slicers modernos se puede indicar la temperatura deseada de la cámara térmica directamente en el perfil del filamento. Este valor se puede usar cómodamente como parámetro `S` para los macros `M141` o `M191`.

En la captura de pantalla se muestra el campo "Chamber temperature", en el que se ha establecido el valor `60°C`. Es solo un valor numérico: **no todos los slicers envían el comando para calentar la cámara**. Para que la temperatura se aplique realmente, es necesario comprobar cómo el slicer genera el G-code y, si es necesario, usar comandos en el G-code inicial:

```gcode
M141 S{chamber_temperature}
```

o

```gcode
M191 S{chamber_temperature}
```

Asegúrese también de que la variable `chamber_temperature` esté definida en la configuración del slicer, o sustitúyala manualmente por un número.

Si se activa la opción "Activate temperature control", algunos slicers añaden automáticamente el comando `M191` con el valor de temperatura indicado. Esto es cómodo si quiere que el calentamiento de la cámara se realice con espera antes de la impresión.

Se recomienda usar el control manual mediante macros para tener control completo sobre la lógica de calentamiento y la secuencia de acciones.

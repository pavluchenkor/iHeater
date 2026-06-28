## Nastavení startovního G-code ve sliceru pro správnou funkci iHeater

Pro dosažení rovnoměrného prohřátí a stabilního tisku z technických materiálů je důležité správně nastavit pořadí příkazů ve startovním G-code. Níže jsou uvedena doporučení pro integraci ohřevu komory iHeater do startovního kódu vašeho sliceru (například Cura, PrusaSlicer, OrcaSlicer atd.).

### Co je potřeba udělat

Před zapnutím ohřevu komory je nutné:

**Zapnout ohřev podložky**

   Nahřívání podložky pomáhá komoře prohřívat se rovnoměrněji a rychleji. Taková podložka funguje jako dodatečný zdroj tepla, který přispívá k efektivnějšímu prohřátí celé komory.

   ```
   M140 S[first_layer_bed_temperature]
   ```

**Zapnout ventilátor pro promíchávání vzduchu (pokud se používá)**

   Může to být boční ventilátor nebo jakýkoli jiný ventilátor určený k rovnoměrnému rozložení tepla v komoře.
   Pokud se v konfiguraci Klipper jmenuje například `chamber_fan`, pak:

   ```
   SET_FAN_SPEED FAN=chamber_fan SPEED=1.0
   ```

V některých tiskárnách jsou nainstalovány odsávací ventilátory udržující co nejnižší teplotu v komoře. To je důležité při tisku materiály jako PLA a PETG, ale zhoršuje rychlost ohřevu komory. Takovému ventilátoru lze předat nový parametr teploty, například cílová_teplota_v_komoře + 10

```
SET_TEMPERATURE_FAN_TARGET TEMPERATURE_FAN=chamber_fan TARGET={chamber_temperature + 10}
```

**Zapnout ohřev komory pomocí jednoho z maker: `M141` nebo `M191`**

### Rozdíl mezi `M141` a `M191`

| Makro  | Účel                                                                           | Blokuje provádění kódu                                             |
| ------ | ------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| `M141` | Nastaví cílovou teplotu komory                                                 | Ne (ohřev začne, ale tisk okamžitě pokračuje)                      |
| `M191` | Nastaví teplotu komory a čeká, dokud nedosáhne zadané hodnoty                  | Ano (přechod na další příkaz proběhne až po prohřátí)              |

#### Příklady:

* Pokud chcete okamžitě zahájit ohřev komory a provádět všechny přípravné operace paralelně s ohřevem, a poté pokračovat v tisku bez čekání na úplné prohřátí komory, je vhodná tato varianta. Dobře funguje pro materiály typu ABS a velké modely - během tisku prvních několika vrstev se komora stihne prohřát:

  ```
  M141 S60
  ```

* Pokud se tiskne malý díl nebo je vyžadována stabilní teplota komory (například při tisku PA, PC a dalších citlivých materiálů), je lepší použít příkaz s čekáním na prohřátí:

  ```
  M191 S60
  ```

### Po ohřevu komory

Po zavolání jednoho z maker (`M141` nebo `M191`) můžete pokračovat běžným startovním G-code:

```gcode
M190 S[first_layer_bed_temperature]  Čekání na nahřátí podložky
M109 S[first_layer_temperature]  Čekání na nahřátí hotendu
G28  Homing
G29  (pokud se používá automatická kalibrace)
... 
```

### Doporučení

* Pokud používáte `M191`, můžete nastavit malou odchylku, při které je komora považována za dostatečně prohřátou (například 5°C pod cílem) - nastavuje se v makru [gcode_macro CHAMBER_VARS] `variable_start_offset`.
* Ujistěte se, že jsou všechna používaná makra (`M141`, `M191`) připojena v konfiguraci Klipper a správně nastavena.
* Pokud váš slicer podporuje podmínky, můžete přidat kontrolu: například zapínat komoru pouze při teplotě tisku vyšší než 50°C (pro ABS, ASA atd.).

---

### Nastavení teploty v termokomoře ve sliceru

![Slicer_settings](../img/slicer_settings.jpeg)

V mnoha moderních slicerech lze požadovanou teplotu termokomory zadat přímo v profilu filamentu. Tuto hodnotu je vhodné použít jako parametr `S` pro makra `M141` nebo `M191`.

Na snímku je zobrazeno pole "Chamber temperature", ve kterém je nastavena hodnota `60°C`. Je to pouze číselná hodnota - **ne všechny slicery odesílají příkaz k ohřevu komory**. Aby se teplota skutečně použila, je nutné zkontrolovat generování G-code slicerem a v případě potřeby použít příkazy ve startovním G-code:

```gcode
M141 S{chamber_temperature}
```

nebo

```gcode
M191 S{chamber_temperature}
```

Také se ujistěte, že je proměnná `chamber_temperature` nastavena v nastavení sliceru, nebo ji ručně nahraďte číslem.

Pokud aktivujete volbu "Activate temperature control", některé slicery automaticky přidají příkaz `M191` se zadanou hodnotou teploty. To je pohodlné, pokud chcete, aby ohřev komory probíhal s čekáním před tiskem.

Doporučuje se používat ruční ovládání pomocí maker, abyste měli plnou kontrolu nad logikou ohřevu a posloupností akcí.

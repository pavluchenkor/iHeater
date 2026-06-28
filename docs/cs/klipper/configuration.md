# Konfigurace Klipperu

Tato stránka popisuje instalaci konfiguračních souborů iHeater a nastavení práce s Klipperem.

## Požadavky

### Hardware
  - Řídicí deska iHeater
  - Termistory NTC 100K 3950 (2 ks)
  - Topný článek PTC 220 V 100 W pro komoru
  - Ventilátor 7530 220 V pro cirkulaci vzduchu v komoře
  - Thermal Protector KSD9700 nebo podobný (220 V, 5 A, 130 °C)

### Software
  - Klipper (nejnovější verze)
  - Nastavený a funkční hostitel s Klipperem

## Konfigurace Klipperu


Zkopírujte konfigurační soubory iHeater.cfg do složky se souborem printer.cfg (může to být /klipper_config) a připojte jej v printer.cfg pomocí direktivy [include]


```
cd ~/klipper_config
```

```
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

Otevřete printer.cfg a přidejte

    [include iHeater.cfg]

## Připojení MCU iHeater

Upravte soubor iHeater.cfg a zadejte získané ID

```
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ

```

## Příprava k použití

Konfigurační soubor obsahuje sekci:

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # Cílová teplota komory, °C
variable_start_offset: 10           # Teplota komory dostatečná pro zahájení tisku, °C
variable_delta_temp: 10             # Rozdíl mezi teplotou komory a topného tělesa, °C
variable_min_heater_temp: 50        # Minimální teplota topného tělesa (pro chlazení), °C
variable_max_heater_temp: 100       # Maximální teplota topného tělesa, °C
variable_control_interval: 1.0      # Interval volání řídicí funkce, sekundy
variable_air_min_delta: 0.5         # Minimální rozdíl mezi cílovou a aktuální teplotou komory (topné těleso = cílová + delta_temp), °C
variable_air_max_delta: 5.0         # Maximální rozdíl mezi cílovou a aktuální teplotou komory (topné těleso = max_heater_temp), °C
gcode:
```

**Maximální přípustná teplota topného tělesa závisí na materiálu skříně.**

Pro kontrolu:

!. Zapněte vyhřívání podložky na 90-100°C
1. Nastavte teplotu topného tělesa na 100°C přes rozhraní Fluidd nebo Mainsail.
2. Ujistěte se, že se iHeater nachází uvnitř uzavřeného prostoru tiskárny.
3. Po dosažení nastavené teploty zkontrolujte místa, kde se topné těleso dotýká plastových prvků skříně. Plast by neměl měknout.
4. Zvyšte teplotu o 5-10°C a kontrolu zopakujte.
5. Opakujte, dokud nebude dosažena maximální přípustná teplota topného tělesa bez rizika deformace skříně.

Tento postup umožňuje určit bezpečné teplotní maximum a dosáhnout nejlepší účinnosti iHeater.


## Použití

### Příkazy pro řízení vyhřívání komory
- Nastavení teploty komory:
 

        M141 S60  ; Nastaví teplotu komory na 60°C

- Čekání na dosažení teploty:

        M191 S60  ; Čeká, dokud teplota komory nedosáhne 60°C

- Zastavení vyhřívání komory:

        iHEATER_OFF   ; Vypne vyhřívání komory

- Na konec G-code sliceru přidejte `iHEATER_OFF`, aby se vyhřívání komory správně vypnulo.

### Startovní g-code

Moderní slicery podporují automatické zapnutí aktivní termokomory při generování tiskového g-code. K tomu je třeba ve vlastnostech filamentu zadat teplotu komory. Pokud slicer tuto funkci nemá, je nutné do startovního g-code přidat příkaz pro zapnutí vyhřívání aktivní termokomory.

Postup:

- Nastavit cílovou teplotu komory
- Zapnout vyhřívání podložky pro účinné a rychlé zahřátí komory 
- Pokračovat standardním startovním G-code tisku

Příklad startovního g-code
```
; --- Začátek startovního G-code ---

; ****** Start iHeater ******
M141 S60       ; Nastavit teplotu komory na 60°C
; ****** Konec bloku iHeater ******

; --- Zbytek startovního g-code ---
; Zapnutí vyhřívání podložky
...
```
!!! warning "Pro správné ukončení práce řídicího makra iHeater je nutné do závěrečného g-code tiskárny přidat příkaz iHEATER_OFF"

```
; --- Začátek závěrečného g-code ---

; ****** Start bloku iHeater ******
iHEATER_OFF
; ****** Konec bloku iHeater ******

; --- Zbytek závěrečného g-code ---
...
```
## Vypnutí

Pro vypnutí iHeater je třeba v souboru printer.cfg zakomentovat řádek [include iHeater.cfg]
```
# [include iHeater.cfg]
```

A odstranit ze startovního a závěrečného g-code odpovídající řádky

## Poznámky
- Bezpečnost:

    - Ujistěte se, že jsou všechna připojení provedena správně a bezpečně.
    - Zkontrolujte, že hodnoty min_temp a max_temp odpovídají specifikacím zařízení.

- Kontrola zařízení:
    - Před použitím otestujte práci topného tělesa a ventilátoru.
    - Sledujte teplotu během prvních spuštění.
- Nastavení PID:
    - V případě potřeby proveďte kalibraci PID pro přesné řízení teploty.

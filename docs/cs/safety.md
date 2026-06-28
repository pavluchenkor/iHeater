# Bezpečnost

!!! danger "Práce se síťovým napětím"
    Zařízení obsahuje komponenty pod napětím 110–230 V. Před jakoukoli prací na elektrické části odpojte napájení. Před prvním zapnutím se ujistěte, že jsou všechny spoje spolehlivě izolované.

Firmware kontroleru — Klipper nebo Standalone — zajišťuje softwarovou ochranu:

- kontrolu teploty pomocí termistorů;
- kontrolu přítomnosti připojení teplotních senzorů;
- ochranu proti překročení bezpečných teplotních mezí;
- použití časovačů pro případ zamrznutí systému;
- automatické vypnutí při chybách senzorů nebo kontroleru.

Dále je implementována hardwarová ochrana:

Je instalován Thermal Protector KSD9700 (135 °C), který v případě přehřátí fyzicky odpojí napájení topného prvku. Po poklesu teploty pod prahovou hodnotu zařízení automaticky sepne obvod a obnoví napájení.

Kontroler je vybaven pojistkou 2 A, která zařízení chrání; v nouzové situaci se přepálí a zcela odpojí systém od napájení.

Používá se PTC topný prvek s úplnou elektrickou izolací. Na rozdíl od většiny topných řešení není tělo PTC topného prvku pod napětím, což eliminuje riziko úrazu elektrickým proudem při instalaci a údržbě komory 3D tiskárny.

Tento víceúrovňový systém ochrany dělá z iHeater bezpečné řešení pro aktivní ohřev komor 3D tiskáren, včetně dlouhodobého nepřetržitého provozu.

!!! warning "Instalace termistoru"
    Ujistěte se, že se odhalené části vodičů u základny termistoru nedotýkají kovového těla topného prvku. V případě potřeby tyto části izolujte kaptonovou páskou nebo je umístěte do teflonové trubičky / smršťovací bužírky.

    Teplota topného prvku může dosáhnout 140 °C.

!!! danger "KSD9700 — není finální ochrana"
    KSD9700 (Thermal Protector) je samoresetovací zařízení: při přehřátí rozpojí obvod, ale jakmile teplota klesne pod práh, automaticky jej znovu sepne. Při poruše topného prvku se zařízení bude cyklicky přehřívat a chladnout bez jakéhokoli zásahu. Nejde o nouzové vypnutí — jde o nekonečný cyklus přehřívání.

    Pro trvalý provoz nahraďte KSD9700 jednorázovou Thermal Fuse (například **RH130**). Při aktivaci trvale přeruší obvod — zařízení se odpojí od napájení a zůstane v bezpečném stavu až do výměny.

!!! note "Doporučený postup"
    Používejte KSD9700 ve fázi sestavení a ladění. Po ověření funkčnosti jej nahraďte Thermal Fuse.

# O projektu iHeater

iHeater je kompaktní ohřívač pro vytvoření aktivní termokomory v 3D tiskárně. Je obzvlášť užitečný u modelů s uzavřenou nebo proprietární elektronikou - Creality, Bambu Lab, FlashForge - kde nejsou volné konektory pro připojení ohřívače, ventilátoru a termistoru.

Připojuje se přes USB a funguje nezávisle na omezeních hlavní desky. Podle firmwaru - v plné integraci s Klipper nebo autonomně.

V kombinaci s vyhříváním podložky zajišťuje iHeater rovnoměrné zahřátí komory - klíčový faktor při tisku ABS, PA, PC a dalších technických plastů. Zařízení dynamicky řídí ohřev podle teploty vzduchu a vytváří stabilní podmínky uvnitř komory bez přehřívání a výkyvů.

Dostupné jsou dvě verze:

- 100 W - pro menší tiskárny (archivní)
- 200 W - pro větší tiskárny

![iHeater](../img/iHeater_promo.png)

## Způsoby použití

### Pod řízením Klipper

Deska funguje jako samostatný MCU v Klipper a zcela autonomně řídí ohřev komory a ventilátor. Napájení z 220 V nezatěžuje napájecí zdroj tiskárny - standardní zdroje často pracují na hranici možností.

![PCB](../img/iHeater_200_PCB.png)

Cena desky je srovnatelná nebo nižší než vlastní sestavení obdobného řešení na bázi mikrokontroléru, polovodičového relé a potřebných součástek. Pro nadšence zůstává možnost sestavit obdobné řešení svépomocí.

### S firmwarem iHeater

Deska iHeater je soběstačná a obsahuje veškeré potřebné periferie pro použití jako samostatné zařízení. Cílová teplota se nastavuje postupným stiskem tlačítka MODE a zobrazuje se třemi LED.

## Licence

Projekt je šířen pod licencí MIT. Podrobnosti najdete v souboru [LICENSE](license.md).

!!! danger "Práce s topnými prvky"
    Používání topných prvků a řízení teploty je spojeno s rizikem požáru a poškození zařízení. Dodržujte bezpečnostní opatření. Více v části [Bezpečnost](safety.md).

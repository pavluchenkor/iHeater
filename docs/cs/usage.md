# Ohřev termokomory ve 3D tiskárně

## 1. Kontrola ventilátorů a ventilačních otvorů

* V nových modelech tiskáren se často instalují ventilátory pro vyfukování horkého vzduchu z komory. Před zahájením provozu se ujistěte, že se automaticky nespouštějí při překročení teploty v komoře..
* Zkontrolujte mezery v těle tiskárny a kolem dveří
* Zkontrolujte ventilační otvory těla. Pokud má komora kontakt s oddílem elektroniky a jsou tam nezakryté otvory, je nutné je uzavřít:

    * běžnou lepicí páskou;
    * hliníkovou lepicí páskou (lépe odráží teplo);
    * nebo tepelnou izolací (nejlepší možnost).
* Je to potřeba proto, aby se řídicí elektronika nepřehřívala.


## 2. Zdroj tepla: stůl tiskárny

* Vyhřívaný stůl je hlavním zdrojem tepla pro komoru.
* Samotný iHeater obvykle nezvýší teplotu na požadovanou hodnotu bez stolu.
* Pokud je potřeba zahřát objem tiskárny bez zapnutého stolu, použijte průmyslově vyráběné ohřívače s výkonem 600 W - 1 kW, aby se kompenzovala absence tepla ze stolu (**s kontrolou elektrické bezpečnosti a zatížitelnosti napájecích obvodů**).

## 3. Použití pomocných ventilátorů

* V moderních modelech jsou často další ventilátory podél bočních stěn, určené pro dodatečné ofukování modelu, nebo uhlíkové filtry uvnitř komory.
* Ve fázi ohřevu komory je lze zapínat pro promíchávání vzduchu - tím se ohřev zrychlí a vyrovná.
* Optimální je připravit makro‑logiku: při startu ohřevu se ventilátory zapnou, po dosažení cílové teploty nebo počátečních vrstev tisku se vypnou.

## 4. Kdy začít tisk

* Příklad: cílová teplota komory - 60°C.
* Tisk lze začít při 50-55°C, protože před startem probíhají přípravné operace: vytvoření mapy stolu, zahřátí a vyčištění trysky, položení prvních vrstev.
* Tyto procesy trvají několik minut; za tu dobu se komora stihne přiblížit k cílové hodnotě.
* Po vytištění prvních 2-3mm modelu komora obvykle dosáhne potřebné teploty a stabilizuje se, tento parametr je individuální pro každou tiskárnu

## 5. Instalace termistoru

* Teplotní senzor (termistor) umístěte přibližně na úrovni tiskové hlavy.
* Nesmí se dotýkat dílů těla tiskárny, jinak bude snímat jejich teplotu, a ne teplotu vzduchu.

## 6. Bezpečnost a další doporučení

* Umístěte iHeater tak, aby zajišťoval rovnoměrné proudění a vylučoval lokální přehřívání.
* Použijte tepelnou izolaci těla pro snížení tepelných ztrát.
* Napájecí vedení ohřívačů musí vydržet proudové zatížení (kabel, konektory, pojistka).
* Teplotní režimy musí odpovídat materiálům těla tiskárny a provozním podmínkám.

---

## 7. Krátký checklist před spuštěním

* [ ] Ventilátory jsou funkční; ventilační otvory jsou vyčištěné.
* [ ] Oddíl elektroniky je izolován od horkého objemu komory.
* [ ] Je zapnutý dodatečný zdroj tepla.
* [ ] Ventilátory pro promíchávání vzduchu se aktivují ve fázi ohřevu.
* [ ] Teplotní senzor je instalován na úrovni tiskové hlavy, nedotýká se těla.
* [ ] Teplotní limity a nouzová vypnutí jsou nastavena.

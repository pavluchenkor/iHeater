# Problemy s komunikaci iHeater a jejich reseni

Pri pouzivani **iHeater** se v nekterych pripadech mohou objevit problemy se stabilitou pripojeni (odpojovani, "ztrata" MCU, nestabilni provoz).  
Ve vetsine pripadu to nesouvisi se samotnym zarizenim, ale s vnejsimi faktory: vibracemi, elektromagnetickym rusenim nebo vlastnostmi zateze.

Nize jsou uvedeny hlavni priciny a zpusoby jejich odstraneni.

---

## 1. Vibrace USB kabelu

!!! warning "Priznaky"
    - Periodicke vypadky pripojeni  
    - Zarizeni "mizi" ze systemu  
    - Komunikace se obnovi pri dotyku kabelu  

!!! info "Pricina"
    Vibrace od tiskarny mohou zpusobovat mikropohyby USB konektoru, coz vede ke kratkodobe ztrate kontaktu.

!!! success "Reseni"
    - Pevne zajistete USB kabel v konektoru  
    - Eliminujte tah kabelu  
    - V pripade potreby:
        - pouzijte kabel s pevnejsim usazenim  
        - zajistete kabel tavnym lepidlem / stahovaci paskou / drzackem  

---

## 2. Ruseni od silovych vodicu

!!! warning "Priznaky"
    - Ztrata komunikace pri zapnuti ohrevu nebo ventilatoru  
    - Nahodne restarty zarizeni  
    - Nestabilni provoz bez zjevne priciny  

!!! info "Pricina"
    Napajeci vodice stridaveho proudu vytvareji elektromagneticke ruseni, ktere se indukuje do USB kabelu.

 ![ferrite bead](../../img/ferrite_bead.png)

!!! success "Reseni"
    - Vzdalenost mezi USB kabelem a silovymi vodici udrzujte co nejvetsi  
    - Nevedte je ve stejnem kabelovem kanalu  
    - Vyhybejte se paralelnimu vedeni na dlouhych usecich  
    - Nainstalujte feritovy filtr (feritovy valec) na USB kabel blize ke kontroleru a/nebo desce tiskarny

---

## 3. Ruseni od ventilatoru

!!! warning "Priznaky"
    - Ztrata komunikace pri zapnuti/vypnuti ventilatoru  
    - Vypadky shodne s provozem ventilatoru  
    - Nestabilita pri rizeni PWM  

!!! info "Pricina"
    Ventilator 110-220V je vybaven spinanym napajecim zdrojem a muze vytvaret ruseni podobne jakemukoli spinanemu zdroji.
    Toto ruseni muze ovlivnovat signalove linky.

![ferrite bead](../../img/snubber1.png)
![ferrite bead](../../img/snubber2.png)

!!! success "Reseni"
    Doporucuje se nainstalovat **RC snubber (snubber)** paralelne k ventilatoru. Nebo pouzit feritovy filtr

---

## 4. Port USB 3.0 - problemy pri provozu

!!! warning "Priznaky"
    - Periodicke vypadky pripojeni behem provozu  
    - Zarizeni "mizi" ze systemu bez zjevne priciny  
    - Problem zmizi po prepnuti na jiny port  

!!! info "Pricina"
    Jde o bezny problem USB zarizeni pracujicich v rezimu Full Speed (USB 2.0) pri pripojeni k portum USB 3.0. V modernich pocitacich porty USB 3.0 pouzivaji opakovace eUSB2, ktere nejsou plne kompatibilni se specifikaci USB 2.0 - to vede k chybam synchronizace a chybam enumerace zarizeni. Problem oficialne potvrdila spolecnost STMicroelectronics: [FAQ na webu ST](https://community.st.com/t5/stm32-mcus/faq-possible-communication-failure-between-stlink-v3-and-some/ta-p/736578).

!!! success "Reseni"
    - Pripojujte iHeater **pouze k portum USB 2.0** (obvykle cerne konektory)  
    - Pokud jsou vsechny porty USB 3.0, pouzijte **aktivni USB hub s porty USB 2.0**

---

## 5. Port USB 3.0 - problemy pri flashovani

!!! warning "Priznaky"
    - Kontroler neni rozpoznan v rezimu DFU  
    - Flashovani skonci chybou nebo se zasekne  
    - `dfu-util` nevidi zarizeni nebo prerusi zapis  

!!! info "Pricina"
    Stejny problem kompatibility USB 3.0 / xHCI. Obzvlaste aktualni je pri flashovani pres porty USB Type-C na modernich noteboocich - ty casteji pouzivaji problematicke opakovace eUSB2.

!!! success "Reseni"
    - Pri flashovani pripojujte kontroler **pouze k portu USB 2.0**  
    - Uprednostnujte porty USB Type-A na zadnim panelu PC  
    - Pokud problem pretrvava, pouzijte **aktivni USB hub s porty USB 2.0**


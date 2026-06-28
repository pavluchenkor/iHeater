## Firmware iHeater pro tiskárny Creality přes Creality Helper Script

Aby firmware a integrace iHeater proběhly úspěšně, postupujte podle podrobného návodu:

### 1. Nainstalujte Creality Helper Script

Přejděte na stránku dokumentace projektu Creality Helper Script a postupujte podle pokynů k instalaci skriptu.

**Zdroje:**

* Videonávod: [YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* Textový návod: [guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. Získejte root přístup k tiskárně a přístup k souborovému systému

Skript zpřístupní Mainsail a také konfigurační soubory firmwaru. Po úspěšné instalaci se ujistěte, že se můžete přes prohlížeč přihlásit do rozhraní tiskárny a získat přístup ke konfiguračním souborům.

### 3. Odstraňte starý `fan-control.cfg`

Na tiskárnách Creality s Helper Script může být ve výchozím stavu již vytvořen soubor `fan-control.cfg` s makry `M141` a `M191`. Je v konfliktu s obdobnými makry v konfiguraci iHeater.

Přejmenujte soubor:

```
/usr/data/printer_data/config/fan-control.cfg
```
na fan-control.cfg.bak

### 4. Zkopírujte nový `fan-control.cfg`

Nahraďte ho verzí [fan-control.cfg](../../../printers/creality/config/fans-control.cfg), která je kompatibilní s makry a logikou řízení teploty komory.

Umístěte nový soubor do stejné složky:

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. Přidejte konfiguraci iHeater

Zkopírujte soubor `iheater.cfg` do stejného adresáře:

```
/usr/data/printer_data/config/iheater.cfg
```

Poté otevřete `printer.cfg` a na konec souboru přidejte řádek:

```ini
[include iheater.cfg]
```

---

Dále postupujte podle návodu k nastavení iHeater - nastavení termistoru, ohřívače, provozních režimů a maker.

!!! warning "Pokud není možné sestavit a nahrát firmware do tiskárny"
    [Přejděte do části WSL](../user-mods/software/wsl2-ubuntu-ff/)

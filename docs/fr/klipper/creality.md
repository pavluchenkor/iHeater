## Firmware iHeater pour les imprimantes Creality via Creality Helper Script

Pour que le firmware et l'integration iHeater se deroulent correctement, suivez les instructions etape par etape :

### 1. Installez Creality Helper Script

Accedez a la page de documentation du projet Creality Helper Script et suivez les instructions d'installation du script.

**Ressources :**

* Guide video : [YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* Instructions textuelles : [guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. Obtenez l'acces root a l'imprimante et l'acces au systeme de fichiers

Le script ouvrira l'acces a Mainsail, ainsi qu'aux fichiers de configuration du firmware. Apres une installation reussie, assurez-vous que vous pouvez acceder a l'interface de l'imprimante via un navigateur et aux fichiers de configuration.

### 3. Supprimez l'ancien `fan-control.cfg`

Sur les imprimantes Creality avec Helper Script, un fichier `fan-control.cfg` contenant les macros `M141` et `M191` peut deja etre cree par defaut. Il entre en conflit avec les macros similaires de la configuration iHeater.

Renommez le fichier :

```
/usr/data/printer_data/config/fan-control.cfg
```
en fan-control.cfg.bak

### 4. Copiez le nouveau `fan-control.cfg`

Remplacez-le par la version [fan-control.cfg](../../../printers/creality/config/fans-control.cfg), compatible avec les macros et la logique de gestion de la temperature de la chambre.

Placez le nouveau fichier dans le meme dossier :

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. Ajoutez la configuration iHeater

Copiez le fichier `iheater.cfg` dans le meme repertoire :

```
/usr/data/printer_data/config/iheater.cfg
```

Ensuite, ouvrez `printer.cfg` et ajoutez la ligne suivante a la fin du fichier :

```ini
[include iheater.cfg]
```

---

Suivez ensuite les instructions de configuration d'iHeater - configuration de la thermistance, du chauffage, des modes de fonctionnement et des macros.

!!! warning "Si vous ne pouvez pas compiler et flasher le firmware sur l'imprimante"
    [Consultez la section WSL](../user-mods/software/wsl2-ubuntu-ff/)

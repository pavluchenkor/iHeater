# Configuration Klipper

Cette page décrit l'installation des fichiers de configuration iHeater et la configuration du fonctionnement avec Klipper.

## Exigences

### Matériel
  - Carte de contrôle iHeater
  - Thermistances NTC 100K 3950 (2 pcs)
  - Élément chauffant PTC 220 V 100 W, pour la chambre
  - Ventilateur 7530 220 V, pour la circulation de l'air dans la chambre
  - Thermal Protector KSD9700 ou équivalent (220 V, 5 A, 130 °C)

### Logiciel
  - Klipper (dernière version)
  - Hôte configuré et fonctionnel avec Klipper

## Configuration Klipper


Copiez les fichiers de configuration iHeater.cfg dans le dossier contenant le fichier printer.cfg (il peut s'agir de /klipper_config) et incluez-le dans printer.cfg à l'aide de la directive [include]


```
cd ~/klipper_config
```

```
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

Ouvrez printer.cfg et ajoutez

    [include iHeater.cfg]

## Connexion du MCU iHeater

Modifiez le fichier iHeater.cfg, indiquez l'ID obtenu

```
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ

```

## Préparation à l'utilisation

Le fichier de configuration contient la section :

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # Température cible de la chambre, °C
variable_start_offset: 10           # Température de la chambre suffisante pour démarrer l'impression, °C
variable_delta_temp: 10             # Différence entre la température de la chambre et celle du chauffage, °C
variable_min_heater_temp: 50        # Température minimale du chauffage (pour le refroidissement), °C
variable_max_heater_temp: 100       # Température maximale du chauffage, °C
variable_control_interval: 1.0      # Intervalle d'appel de la fonction de contrôle, secondes
variable_air_min_delta: 0.5         # Différence minimale entre la température cible et actuelle de la chambre (chauffage = cible + delta_temp), °C
variable_air_max_delta: 5.0         # Différence maximale entre la température cible et actuelle de la chambre (chauffage = max_heater_temp), °C
gcode:
```

**La température maximale admissible du chauffage dépend du matériau du boîtier.**

Pour vérifier :

!. Activez le chauffage du plateau à 90-100°C
1. Réglez la température du chauffage à 100°C via l'interface Fluidd ou Mainsail.
2. Assurez-vous que iHeater se trouve à l'intérieur du volume fermé de l'imprimante.
3. Après avoir atteint la température définie, vérifiez les zones où le chauffage est en contact avec les éléments en plastique du boîtier. Le plastique ne doit pas ramollir.
4. Augmentez la température de 5-10°C et répétez la vérification.
5. Répétez jusqu'à atteindre la température maximale admissible du chauffage sans risque de déformation du boîtier.

Cette approche permet de déterminer le maximum de température sûr et d'obtenir la meilleure efficacité de fonctionnement de iHeater.


## Utilisation

### Commandes de contrôle du chauffage de la chambre
- Réglage de la température de la chambre :
 

        M141 S60  ; Définit la température de la chambre à 60°C

- Attente de l'atteinte de la température :

        M191 S60  ; Attend que la température de la chambre atteigne 60°C

- Arrêt du chauffage de la chambre :

        iHEATER_OFF   ; Désactive le chauffage de la chambre

- À la fin du G-code du slicer, ajoutez `iHEATER_OFF` pour désactiver correctement le chauffage de la chambre.

### G-code de démarrage

Les slicers modernes prennent en charge l'activation automatique de la chambre chauffée active lors de la génération du g-code d'impression. Pour cela, il faut indiquer la température de la chambre dans les propriétés du filament. Si le slicer ne dispose pas de cette fonctionnalité, il faut ajouter la commande d'activation du chauffage de la chambre chauffée active dans le g-code de démarrage.

Procédure :

- Définir la température cible de la chambre
- Activer le chauffage du plateau pour chauffer la chambre efficacement et rapidement 
- Continuer le G-code de démarrage standard de l'impression

Exemple de g-code de démarrage
```
; --- Début du G-code de démarrage ---

; ****** Démarrage iHeater ******
M141 S60       ; Définir la température de la chambre à 60°C
; ****** Fin du bloc iHeater ******

; --- Reste du g-code de démarrage ---
; Activation du chauffage du plateau
...
```
!!! warning "Pour terminer correctement le fonctionnement du macro de contrôle iHeater, il est nécessaire d'ajouter la commande iHEATER_OFF au g-code de fin de l'imprimante"

```
; --- Début du g-code de fin ---

; ****** Début du bloc iHeater ******
iHEATER_OFF
; ****** Fin du bloc iHeater ******

; --- Reste du g-code de fin ---
...
```
## Désactivation

Pour désactiver iHeater dans le fichier printer.cfg, il faut commenter la ligne [include iHeater.cfg]
```
# [include iHeater.cfg]
```

Et supprimer les lignes correspondantes du g-code de démarrage et de fin

## Remarques
- Sécurité :

    - Assurez-vous que toutes les connexions sont effectuées correctement et en toute sécurité.
    - Vérifiez que les valeurs min_temp et max_temp correspondent aux spécifications de l'équipement.

- Vérification de l'équipement :
    - Avant utilisation, testez le fonctionnement du chauffage et du ventilateur.
    - Surveillez la température lors des premiers démarrages.
- Réglage PID :
    - Si nécessaire, effectuez une calibration PID pour un contrôle précis de la température.

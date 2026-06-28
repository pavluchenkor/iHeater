## Configuration du G-code de démarrage dans le slicer pour le bon fonctionnement d’iHeater

Pour obtenir un chauffage homogène et une impression stable avec des matériaux techniques, il est important de configurer correctement l’ordre des commandes dans le G-code de démarrage. Vous trouverez ci-dessous des recommandations pour intégrer le chauffage de la chambre iHeater dans le code de démarrage de votre slicer (par exemple Cura, PrusaSlicer, OrcaSlicer, etc.).

### Ce qu’il faut faire

Avant d’activer le chauffage de la chambre, il est nécessaire de :

**Activer le chauffage du plateau**

   Le préchauffage du plateau aide la chambre à chauffer plus uniformément et plus rapidement. Le plateau sert de source de chaleur supplémentaire, ce qui contribue à un chauffage plus efficace de toute la chambre.

   ```
   M140 S[first_layer_bed_temperature]
   ```

**Activer le ventilateur de brassage de l’air (s’il est utilisé)**

   Il peut s’agir d’un ventilateur latéral ou de tout autre ventilateur destiné à répartir uniformément la chaleur dans la chambre.
   Si, dans la configuration Klipper, il s’appelle par exemple `chamber_fan`, alors :

   ```
   SET_FAN_SPEED FAN=chamber_fan SPEED=1.0
   ```

Sur certaines imprimantes, des ventilateurs d’extraction sont installés pour maintenir la température la plus basse possible dans la chambre ; c’est important lors de l’impression avec des plastiques comme PLA et PETG, mais cela nuit à la vitesse de chauffage de la chambre. On peut transmettre à ce ventilateur un nouveau paramètre de température, par exemple température_cible_dans_la_chambre + 10

```
SET_TEMPERATURE_FAN_TARGET TEMPERATURE_FAN=chamber_fan TARGET={chamber_temperature + 10}
```

**Activer le chauffage de la chambre à l’aide de l’un des macros : `M141` ou `M191`**

### Différence entre `M141` et `M191`

| Macro  | Fonction                                                                       | Bloque-t-elle l’exécution du code                                      |
| ------ | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| `M141` | Définit la température cible de la chambre                                     | Non (le chauffage démarre, mais l’impression continue immédiatement)   |
| `M191` | Définit la température de la chambre et attend qu’elle atteigne la valeur cible | Oui (le passage à la commande suivante se fera seulement après chauffage) |

#### Exemples :

* Si vous souhaitez lancer immédiatement le chauffage de la chambre et exécuter toutes les opérations préliminaires en parallèle du chauffage, puis poursuivre l’impression sans attendre le chauffage complet de la chambre, cette option convient. Cela fonctionne bien pour les plastiques de type ABS et les grands modèles : pendant l’impression des premières couches, la chambre aura le temps de chauffer :

  ```
  M141 S60
  ```

* Si une petite pièce est imprimée ou si une température stable de la chambre est requise (par exemple lors de l’impression de PA, PC et d’autres matériaux sensibles), il est préférable d’utiliser la commande avec attente du chauffage :

  ```
  M191 S60
  ```

### Après le chauffage de la chambre

Après l’appel de l’un des macros (`M141` ou `M191`), vous pouvez continuer avec le G-code de démarrage habituel :

```gcode
M190 S[first_layer_bed_temperature]  Attente du chauffage du plateau
M109 S[first_layer_temperature]  Attente du chauffage du hotend
G28  Homing
G29  (si l’autocalibrage est utilisé)
... 
```

### Recommandations

* Si vous utilisez `M191`, vous pouvez définir un léger écart auquel la chambre est considérée comme suffisamment chauffée (par exemple 5°C sous la cible) : cela se configure dans le macro [gcode_macro CHAMBER_VARS] `variable_start_offset`.
* Assurez-vous que tous les macros utilisés (`M141`, `M191`) sont inclus dans la configuration Klipper et configurés correctement.
* Si votre slicer prend en charge les conditions, vous pouvez ajouter une vérification : par exemple, activer la chambre uniquement lorsque la température d’impression est supérieure à 50°C (pour ABS, ASA, etc.).

---

### Configuration de la température de la chambre thermique dans le slicer

![Slicer_settings](../img/slicer_settings.jpeg)

Dans de nombreux slicers modernes, il est possible d’indiquer la température souhaitée de la chambre thermique directement dans le profil du filament. Cette valeur est pratique à utiliser comme paramètre `S` pour les macros `M141` ou `M191`.

La capture d’écran montre le champ "Chamber temperature", dans lequel la valeur `60°C` est définie. Il ne s’agit que d’une valeur numérique : **tous les slicers n’envoient pas automatiquement la commande de chauffage de la chambre**. Pour que la température soit réellement appliquée, il faut vérifier la génération du G-code par le slicer et utiliser les commandes dans le G-code de démarrage si nécessaire :

```gcode
M141 S{chamber_temperature}
```

ou

```gcode
M191 S{chamber_temperature}
```

Assurez-vous également que la variable `chamber_temperature` est définie dans les paramètres du slicer, ou remplacez-la manuellement par un nombre.

Si vous activez l’option "Activate temperature control", certains slicers ajoutent automatiquement la commande `M191` avec la valeur de température définie. C’est pratique si vous souhaitez que le chauffage de la chambre se fasse avec une attente avant l’impression.

Il est recommandé d’utiliser le contrôle manuel via les macros afin de garder un contrôle complet sur la logique de chauffage et la séquence des actions.

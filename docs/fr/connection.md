## Schéma de connexion

Le contrôleur **iHeater** peut fonctionner aussi bien au sein d'un système **Klipper** (en tant que MCU supplémentaire) qu'en mode autonome, sous le contrôle du firmware intégré **standalone**.

### Connexion pour l'utilisation avec Klipper

Pour un fonctionnement correct au sein de Klipper, il faut connecter :

* **Câble USB** à l'hôte principal (Host-MCU) - il assure le transfert des données et l'alimentation 5 V ;
* **Alimentation secteur 220 V / 110 V** - selon la version de l'appareil et le type de chauffage ;
* **Thermistance du chauffage** - pour contrôler la température de l'élément chauffant ;
* **Thermistance de la chambre** - pour contrôler la température de l'air dans la chambre de l'imprimante ;
* **Port trigger** - connexion optionnelle, utilisée pour le contrôle automatique à partir d'un signal externe.

En fonctionnement, iHeater est placé à l'intérieur de la chambre de l'imprimante 3D.

!!! note annotate "Il est recommandé de placer la thermistance de la chambre au niveau de la tête d'impression, si possible - **au-dessus du plateau**."

![Schéma de connexion](../img/iHeater_pinout.png)

## Configuration GPIO

| Pin    | Alias       | Function                          |
|--------|-------------|-----------------------------------|
| PA0    | TH1         | Capteur de température de la chambre |
| PA1    | HEATER      | Contrôle du chauffage             |
| PA2    | FAN         | Contrôle du ventilateur           |
| PA3    | TH0         | Capteur de température du chauffage |
| PA4    | MODE        | Bouton de mode                    |
| PA5    | LED3        | LED 3                             |
| PA6    | LED2        | LED 2                             |
| PA7    | LED1        | LED 1                             |
| PB1    | TH2         | Capteur de température supplémentaire |

---

### Utilisation en mode standalone

En mode autonome, des fonctions et méthodes de connexion supplémentaires sont disponibles :

* **Port trigger en mode thermistance**
  Lorsqu'une thermistance est connectée au port trigger et placée près de l'élément chauffant du plateau, le contrôle automatique peut être activé :
  - lorsque le plateau chauffe au-dessus de **45°C** - le chauffage de la chambre s'active ;
  - lorsque la température descend sous **85°C** - le chauffage se désactive.

* **Alimentation depuis une source externe 5 V**
  S'il n'est pas possible d'utiliser l'alimentation USB, l'alimentation peut être fournie directement via le connecteur correspondant.

* **Option expérimentale**
  Connexion d'une [source d'alimentation 5 V directement à la carte](https://sl.aliexpress.ru/p?key=OHtN3Xm).

!!! danger "N'utilisez pas simultanément l'USB et une source d'alimentation externe"
    La connexion simultanée de deux sources d'alimentation est interdite. Cela provoquera un conflit entre les sources d'alimentation, des erreurs de fonctionnement de l'appareil et peut endommager l'équipement.

![Connexion de l'alimentation](../img/IMG_6009.jpg)

---

### Schéma de connexion général

![Schéma de connexion](../img/iHeater_connection.png)

# À propos du projet iHeater

iHeater est un chauffage compact destiné à créer une chambre thermique active dans une imprimante 3D. Il est particulièrement utile sur les modèles à électronique fermée ou propriétaire — Creality, Bambu Lab, FlashForge — où il n'y a pas de connecteurs libres pour raccorder un chauffage, un ventilateur et une thermistance.

Il se connecte en USB et fonctionne indépendamment des limitations de la carte principale. Selon le firmware, il fonctionne soit en intégration complète avec Klipper, soit de manière autonome.

Associé au chauffage du plateau, iHeater assure un réchauffement uniforme de la chambre — un facteur clé pour l'impression de l'ABS, du PA, du PC et d'autres plastiques techniques. L'appareil contrôle dynamiquement le chauffage en fonction de la température de l'air, créant des conditions stables à l'intérieur de la chambre sans surchauffe ni variations brusques.

Deux versions sont disponibles :

- 100 W — pour les petites imprimantes (archivée)
- 200 W — pour les imprimantes de plus grande taille

![iHeater](../img/iHeater_promo.png)

[Vous pouvez effectuer un calcul préliminaire à l'aide de cette calculatrice](https://docs.google.com/spreadsheets/d/1u6XrWLFZGOUnRlFPjjGsJB_GLuFFIs3fFWLCp2-K8gc/edit?usp=sharing)

## Cas d'utilisation

### Sous le contrôle de Klipper

La carte fonctionne comme un MCU séparé dans Klipper, en contrôlant de manière entièrement autonome le chauffage de la chambre et le ventilateur. L'alimentation en 220 V ne sollicite pas l'alimentation de l'imprimante — les alimentations d'origine fonctionnent souvent à leur limite.

![PCB](../img/iHeater_200_PCB.png)

Le coût de la carte est comparable, voire inférieur, à celui d'un assemblage équivalent réalisé soi-même à partir d'un microcontrôleur, d'un relais statique et des composants nécessaires. Les passionnés gardent la possibilité de construire eux-mêmes un équivalent.

### Avec le firmware iHeater

La carte iHeater est autonome et contient toute la périphérie nécessaire pour être utilisée comme appareil indépendant. La température cible se règle par pressions successives sur le bouton MODE et s'affiche au moyen de trois LED.

## Licence

Le projet est distribué sous licence MIT. Les détails se trouvent dans le fichier [LICENSE](license.md).

!!! danger "Travail avec des éléments chauffants"
    L'utilisation d'éléments chauffants et le contrôle de la température comportent un risque d'incendie et de détérioration de l'équipement. Respectez les mesures de précaution. Pour en savoir plus, consultez la section [Sécurité](safety.md).

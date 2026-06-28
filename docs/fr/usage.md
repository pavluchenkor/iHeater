# Chauffage de la chambre thermique dans une imprimante 3D

## 1. Vérification des ventilateurs et des ouvertures de ventilation

* Les nouveaux modèles d'imprimantes sont souvent équipés de ventilateurs pour évacuer l'air chaud de la chambre. Avant de commencer l'utilisation, assurez-vous qu'ils ne s'activent pas automatiquement lorsque la température de la chambre est dépassée..
* Vérifiez les fentes dans le boîtier de l'imprimante et autour de la porte
* Vérifiez les ouvertures de ventilation du boîtier. Si la chambre communique avec le compartiment électronique et qu'il y a des ouvertures non obturées, elles doivent être fermées :

    * avec du ruban adhésif ordinaire ;
    * avec du ruban adhésif aluminium (il réfléchit mieux la chaleur) ;
    * ou avec une isolation thermique (meilleure option).
* Cela est nécessaire pour éviter la surchauffe de l'électronique de commande.


## 2. Source de chaleur : plateau de l'imprimante

* Le plateau chauffant est la principale source de chaleur pour la chambre.
* À lui seul, iHeater ne permet généralement pas d'atteindre la température requise sans le plateau.
* Si le chauffage du volume de l'imprimante est nécessaire sans plateau activé, utilisez des chauffages de fabrication industrielle d'une puissance de 600 W à 1 kW afin de compenser l'absence de chaleur provenant du plateau (**avec vérification de la sécurité électrique et de la capacité de charge des circuits d'alimentation**).

## 3. Utilisation de ventilateurs auxiliaires

* Les modèles modernes disposent souvent de ventilateurs supplémentaires le long des parois latérales, destinés au refroidissement additionnel de la pièce, ou de filtres à charbon à l'intérieur de la chambre.
* Pendant la phase de chauffage de la chambre, ils peuvent être activés pour brasser l'air - cela accélère et homogénéise le chauffage.
* Il est optimal de prévoir une macro-logique : au démarrage du chauffage, les ventilateurs s'activent ; une fois la température cible atteinte ou après les premières couches d'impression, ils se désactivent.

## 4. Quand commencer l'impression

* Exemple : température cible de la chambre - 60°C.
* L'impression peut commencer à 50-55°C, car des opérations préparatoires ont lieu avant le démarrage : génération de la carte du plateau, chauffage et nettoyage de la buse, dépôt des premières couches.
* Ces processus prennent quelques minutes ; pendant ce temps, la chambre a le temps de se rapprocher de la valeur cible.
* Après l'impression des 2-3 premiers mm du modèle, la chambre atteint généralement la température nécessaire et se stabilise ; ce paramètre est propre à chaque imprimante

## 5. Installation de la thermistance

* Placez le capteur de température (thermistance) approximativement au niveau de la tête d'impression.
* Il ne doit pas toucher les éléments du boîtier de l'imprimante, sinon il mesurera leur température et non celle de l'air.

## 6. Sécurité et recommandations supplémentaires

* Positionnez iHeater de manière à assurer un flux uniforme et à éviter les surchauffes locales.
* Utilisez une isolation thermique du boîtier pour réduire les pertes de chaleur.
* Les lignes d'alimentation des chauffages doivent supporter la charge en courant (câble, connecteurs, fusible).
* Les régimes de température doivent correspondre aux matériaux du boîtier de l'imprimante et aux conditions d'utilisation.

---

## 7. Courte check-list avant le démarrage

* [ ] Les ventilateurs fonctionnent ; les ouvertures de ventilation sont dégagées.
* [ ] Le compartiment électronique est isolé du volume chaud de la chambre.
* [ ] Une source de chaleur supplémentaire est activée.
* [ ] Les ventilateurs de brassage de l'air s'activent pendant la phase de préchauffage.
* [ ] Le capteur de température est installé au niveau de la tête d'impression et ne touche pas le boîtier.
* [ ] Les limites de température et les arrêts d'urgence sont configurés.

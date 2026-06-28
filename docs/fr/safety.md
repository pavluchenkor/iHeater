# Sécurité

!!! danger "Travail avec la tension secteur"
    L'appareil contient des composants sous tension 110-230 V. Avant toute intervention électrique, coupez l'alimentation. Vérifiez que toutes les connexions sont correctement isolées avant la première mise sous tension.

Le firmware du contrôleur - Klipper ou Standalone - assure une protection logicielle :

- contrôle de la température à l'aide de thermistances ;
- vérification de la présence de la connexion des capteurs de température ;
- protection contre le dépassement des valeurs de température sûres ;
- utilisation de minuteurs en cas de blocage du système ;
- arrêt automatique en cas d'erreurs des capteurs ou du contrôleur.

Une protection matérielle est également mise en place :

Un Thermal Protector KSD9700 (135 °C) est installé ; en cas de surchauffe, il coupe physiquement l'alimentation de l'élément chauffant. Lorsque la température redescend sous la valeur seuil, l'appareil referme automatiquement le circuit et rétablit l'alimentation.

Le contrôleur est équipé d'un fusible de 2 A qui protège l'appareil ; en situation d'urgence, il fond et met complètement le système hors tension.

Un élément chauffant PTC avec isolation électrique complète est utilisé. Contrairement à la plupart des solutions de chauffage, le boîtier du chauffage PTC n'est pas sous tension, ce qui élimine le risque de choc électrique lors de l'installation et de l'entretien de la chambre de l'imprimante 3D.

Ce système de protection multiniveau fait de iHeater une solution sûre pour le chauffage actif des chambres d'imprimantes 3D, y compris pendant un fonctionnement continu prolongé.

!!! warning "Installation de la thermistance"
    Assurez-vous que les parties dénudées des fils à la base de la thermistance n'entrent pas en contact avec le boîtier métallique du chauffage. Si nécessaire, isolez ces zones avec du ruban Kapton ou placez-les dans un tube en téflon / une gaine thermorétractable.

    La température du chauffage peut atteindre 140 °C.

!!! danger "KSD9700 - pas une protection finale"
    KSD9700 (Thermal Protector) est un dispositif autoréarmable : en cas de surchauffe, il ouvre le circuit, mais dès que la température redescend sous le seuil, il le referme automatiquement. En cas de panne du chauffage, l'appareil surchauffera et refroidira de façon cyclique sans aucune intervention. Ce n'est pas un arrêt d'urgence : c'est un cycle de surchauffe infini.

    Pour une exploitation permanente, remplacez le KSD9700 par un Thermal Fuse à usage unique (par exemple, **RH130**). Il ouvre définitivement le circuit lorsqu'il se déclenche : l'appareil est mis hors tension et reste dans un état sûr jusqu'à son remplacement.

!!! note "Ordre recommandé"
    Utilisez le KSD9700 pendant l'assemblage et le débogage. Après avoir vérifié le bon fonctionnement, remplacez-le par un Thermal Fuse.

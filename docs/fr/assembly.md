# Assemblage

Consultez la documentation, téléchargez et imprimez les pièces nécessaires. Assurez-vous de disposer de tous les composants et outils avant de commencer l'assemblage.

!!! danger "Travail avec la tension secteur"
    Tous les travaux de raccordement au réseau 110–230 V doivent être effectués lorsque l'appareil est hors tension. Pour plus de détails, consultez la section [Sécurité](safety.md).

## Avant l'assemblage

Il est recommandé d'assembler d'abord tout le système **sur table**, sans montage dans le boîtier, puis d'effectuer les tests :

- Connecter **tous** les composants.
- Vérifier le fonctionnement du chauffage, du ventilateur et des capteurs de température.
- Connecter le système à **Klipper** ou flasher le firmware Standalone, puis vérifier son bon fonctionnement.

Video guide: [YouTube](https://youtu.be/1QMtVY0Vx-8?si=Ol1u4Ux9wALDcfe2)

## Assemblage étape par étape

### Installation de la carte

![Assemblage iHeater](../img/iHeater_5484.jpg)

### Installation de la thermistance et du Thermal Protector

!!! warning "Installation de la thermistance"
    Assurez-vous que les parties dénudées des fils à la base de la thermistance ne touchent pas le boîtier métallique du chauffage. Si nécessaire, isolez ces parties avec du ruban Kapton ou placez-les dans un tube en téflon / une gaine thermorétractable.

    La température du chauffage peut atteindre 140 °C.

!!! warning "Installation du Thermal Protector"
    Vous pouvez installer un KSD9700 (Thermal Protector, réarmable automatiquement) ou un Thermal Fuse à usage unique.

    Le KSD9700 ouvre le circuit en cas de surchauffe et le referme automatiquement après refroidissement. Le Thermal Fuse (par exemple, **RH130**) coupe définitivement le circuit lorsqu'il se déclenche — une protection plus fiable en cas de défaut.

    Utilisez le KSD9700 pendant la phase de mise au point, puis remplacez-le par un Thermal Fuse pour une utilisation permanente.

![Assemblage iHeater](../img/iHeater_5489.jpg)
![Assemblage iHeater](../img/thermistor.jpg)

### Installation du chauffage

!!! warning "Installation de la thermistance"
    Installez la thermistance près du bord du chauffage, à peu près à mi-hauteur des ailettes du radiateur.

    Les parties dénudées des fils à la base de la thermistance ne doivent pas toucher le boîtier métallique du chauffage. Si nécessaire, isolez ces parties avec du ruban Kapton ou placez-les dans un tube en téflon / une gaine thermorétractable.

    La température du chauffage peut atteindre 140 °C.

![Assemblage iHeater](../img/iHeater_5491.jpg)

### Câblage

![Assemblage iHeater](../img/iHeater_5494.jpg)

### Installation des embouts de câble

![Assemblage iHeater](../img/iHeater_5496.jpg)

### Raccordement

![Assemblage iHeater](../img/iHeater_5498.jpg)

### Assemblage final

![Assemblage iHeater](../img/iHeater_5500.jpg)

### Produit fini

![Assemblage iHeater](../img/iHeater_5506.jpg)

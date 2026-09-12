# Frankenderstein ! An Ender 5 bitten by a Kobra, powered by Klipper, armed with a laser !
# Upgrade Ender 5 - tête Kobra 2 Pro + MKS Robin Nano V3.1 + Klipper

Projet de modernisation d'une imprimante 3D **Creality Ender 5** : remplacement de l'électronique et de la tête d'impression d'origine par une carte mère plus moderne et un ensemble hotend/extrudeur/capteurs récupéré sur une **Anycubic Kobra 2 Pro**, piloté par **Klipper** depuis un Raspberry Pi.

## Vue d'ensemble

| | |
|---|---|
| **Châssis** | Creality Ender 5 (volume d'impression 220 × 220 × 300 mm — identique Ender 5 / Ender 5 Pro) |
| **Carte mère** | MKS Robin Nano V3.1 (STM32F407, drivers TMC2209 sur X/Y/Z/E0) |
| **Tête d'impression** | Anycubic Kobra 2 Pro : extrudeur direct drive (ratio 4:1), capteur ABL inductif intégré, accéléromètre LIS2DW12 |
| **Firmware imprimante** | Klipper |
| **Hôte** | Raspberry Pi 3B+ (Moonraker + Mainsail) |
| **Écran** | Robin TS35 (initialement prévu sur la carte mère, réaffecté au Raspberry Pi via SPI pour libérer le bus utilisé par l'accéléromètre — voir `RPI/RPI.md`) |

## Pourquoi ce projet

L'objectif de départ était un upgrade classique d'électronique (carte MKS Robin Nano V3.1 + écran tactile TS35) sur une Ender 5. En cours de route, le projet s'est étendu à la récupération de la tête d'impression complète d'une Kobra 2 Pro - extrudeur direct drive et capteur de nivellement automatique inductif - pour remplacer l'extrudeur Bowden et le BLTouch initialement prévus. Cette tête intègre aussi un accéléromètre, ce qui a motivé la bascule finale de Marlin vers **Klipper** : Klipper sait exploiter nativement ce capteur pour du resonance testing / input shaping automatique, ce que Marlin ne propose pas pour ce type de puce.

## Décisions clés

- **Volume d'impression** : 220 × 220 × 300 mm, identique que le châssis soit une Ender 5 ou une Ender 5 Pro (seuls l'extrudeur et l'alimentation diffèrent entre les deux variantes, et l'extrudeur d'origine est de toute façon remplacé).
- **Capteur ABL** : le capteur inductif intégré à la tête Kobra 2 Pro remplace le BLTouch prévu initialement. Après vérification du câblage (présence d'un plan de masse logique distinct de la masse châssis sur la carte de la tête, et câblage direct constaté sur un montage de référence identique), le signal s'est révélé déjà en logique compatible — pas besoin de module convertisseur PNP→NPN.
- **Second canal moteur (E1)** : réservé pour un futur module laser, piloté via une sortie GPIO dédiée (`[output_pin]`) plutôt que comme un second extrudeur — pas encore câblé.
- **Marlin → Klipper** : décidé pour exploiter l'accéléromètre de la tête (bus SPI libéré par le déplacement de l'écran TS35 vers le Raspberry Pi) et pour la simplicité d'intégrer le G-code de démarrage personnalisé directement dans `printer.cfg` (une macro `START_PRINT` reçoit les températures réelles du slicer, sans les limitations d'un hook générique côté serveur d'impression).
- **G-code de démarrage personnalisé** : homing → chauffe plateau + buse → déplacement vers un poste de nettoyage dédié → purge à vide → bed mesh via la sonde ABL → début d'impression (inspiré du système de "poop collector" des Kobra 3).

## Structure du dépôt

```
.
├── Firmware/
│   └── Robin_nano_v3.bin      # Firmware Klipper compilé, prêt à flasher sur la carte
├── Sources/
│   └── klipper/                # Sources Klipper + fichier .config exact utilisé pour la compilation
├── printer.cfg                 # Config Klipper complète (pinout tête Kobra 2 Pro, axes, sonde, accéléromètre, macros)
└── RPI/
    └── RPI.md                  # Procédure de déploiement pas à pas sur le Raspberry Pi
```

Pour flasher la carte, installer Klipper/Moonraker/Mainsail sur le Raspberry Pi et mettre la machine en route : voir **[`RPI/RPI.md`](RPI/RPI.md)**.

## État d'avancement

- [x] Choix et validation de la carte mère (MKS Robin Nano V3.1) et de l'écran (Robin TS35)
- [x] Pinout complet de la tête Kobra 2 Pro identifié et vérifié (thermistance, ventilos, chauffe, sonde ABL, accéléromètre)
- [x] Décision de bascule vers Klipper
- [x] `printer.cfg` initial + firmware compilé
- [ ] Câblage physique complet de la tête sur la Ender 5
- [ ] Écran TS35 câblé sur le Raspberry Pi (SPI + fbcp + KlipperScreen)
- [ ] Premiers tests machine (homing, chauffe, sonde ABL, extrusion) et calibrations (courant moteurs, `rotation_distance` extrudeur, offsets sonde, PID, input shaping)
- [ ] Mécanisme physique du poste de nettoyage ("poop collector")
- [ ] Câblage et pilotage du futur module laser (canal E1)

## Ressources

- Specs et pinout de la tête Kobra 2 Pro : https://1coderookie.github.io/Kobra2ProInsights/hardware/printhead/
- Pinout et câblage de référence de la carte mère : https://1coderookie.github.io/Kobra2ProInsights/hardware/mainboard/#wiring-and-pin-assignment
- `printer.cfg` de référence (même combinaison de matériel) : https://github.com/1coderookie/Klipper4Kobra2series
- Klipper : https://github.com/Klipper3d/klipper
- KIAUH (installation Klipper/Moonraker/Mainsail) : https://github.com/dw-0/kiauh

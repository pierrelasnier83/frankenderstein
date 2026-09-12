# Kobra Klipper Firmware — Ender 5 / MKS Robin Nano V3.1 / tête Kobra 2 Pro

Firmware Klipper compilé pour une carte **MKS Robin Nano V3.1** (STM32F407), pilotant un châssis **Ender 5** équipé d'une tête d'impression **Anycubic Kobra 2 Pro** (extrudeur direct drive + capteur ABL inductif + accéléromètre LIS2DW12), le tout piloté par un **Raspberry Pi 3B+**.

## Contenu du dépôt

```
.
├── Firmware/
│   └── Robin_nano_v3.bin      # Firmware compilé, prêt à flasher sur la carte
├── Sources/
│   └── klipper/                # Sources Klipper utilisées pour la compilation, avec le .config exact
└── printer.cfg                 # Config Klipper complète pour cette machine
```

## Matériel

- Châssis Ender 5 (volume 220 × 220 × 300 mm)
- Carte mère MKS Robin Nano V3.1 (STM32F407, drivers TMC2209 sur X/Y/Z/E0)
- Tête d'impression Anycubic Kobra 2 Pro (direct drive ratio 4:1, capteur ABL inductif intégré, accéléromètre LIS2DW12)
- Raspberry Pi 3B+ comme hôte Klipper (Moonraker + Mainsail)

## Déploiement sur le Raspberry Pi

### 1. Installer Klipper, Moonraker et Mainsail (si ce n'est pas déjà fait)

Le plus simple est [KIAUH](https://github.com/dw-0/kiauh) :

```bash
cd ~
git clone https://github.com/dw-0/kiauh.git
cd kiauh
./kiauh.sh
```

Dans le menu : installer Klipper, puis Moonraker, puis Mainsail (ou Fluidd — dans ce cas, remplacer `[include mainsail.cfg]` par `[include fluidd.cfg]` en première ligne de `printer.cfg`).

### 2. Récupérer ce dépôt sur le Pi

```bash
cd ~
git clone https://github.com/<ton-compte>/kobra-klipper-firmware.git
```

### 3. Flasher le firmware sur la carte (carte SD — pas de mode DFU sur ce modèle)

```bash
# Copier le firmware vers une carte micro SD montée sur le Pi (adapter le point de montage)
cp ~/kobra-klipper-firmware/Firmware/Robin_nano_v3.bin /media/<point_de_montage_sd>/
```

- Utiliser une carte micro SD **petite capacité** (128 ou 256 Mo), formatée en **FAT32**.
- Carte insérée, mainboard **éteinte**, puis allumer.
- Attendre quelques secondes (flash automatique par le bootloader MKS).
- Vérifier le succès : le fichier sur la carte doit avoir été renommé automatiquement en `ROBIN_NANO_V3.BIN.CUR`.
- Éteindre, retirer la carte SD, rallumer normalement.

### 4. Connecter la carte au Raspberry Pi en USB

Simple câble USB (port micro-USB de la Robin Nano) — pas de câblage série séparé, Klipper communique en USB.

```bash
ls /dev/serial/by-id/*
```

Noter le chemin exact affiché (`usb-Klipper_stm32f407xx_XXXXXXXXXXXX-if00`).

### 5. Installer printer.cfg

```bash
cp ~/kobra-klipper-firmware/printer.cfg ~/printer_data/config/printer.cfg
```

Puis éditer la ligne `[mcu] serial:` avec le chemin USB relevé à l'étape précédente, et redémarrer Klipper depuis Mainsail/Fluidd ("Firmware Restart").

### 6. Premiers tests — dans cet ordre

1. Vérifier dans la console qu'il n'y a pas d'erreur MCU (carte bien reconnue).
2. `QUERY_ENDSTOPS` puis actionner chaque fin de course à la main pour vérifier l'état affiché, **avant tout homing**.
3. **Vérifier la sonde ABL au multimètre avant de la relier définitivement** (tension attendue 0–3,3 V / 5 V au repos et au déclenchement, jamais 24 V — voir le raisonnement complet dans `printer.cfg`).
4. Premier `G28` complet, prêt à couper l'alimentation si un axe force en butée.
5. Calibrer le `run_current` des drivers TMC2209 et le `rotation_distance` de l'extrudeur (placeholders marqués `TODO(Pierre)` dans `printer.cfg`).
6. `PROBE_CALIBRATE` pour l'offset Z réel, mesure physique des offsets X/Y de la sonde.
7. `PID_CALIBRATE HEATER=heater_bed TARGET=60` (et pour l'extrudeur si besoin).
8. `SHAPER_CALIBRATE` une fois un premier test d'impression basique réussi, pour renseigner `[input_shaper]`.

### 7. Recompiler le firmware si besoin

Le `.config` exact utilisé est conservé dans `Sources/klipper/.config` :

```bash
cd ~/kobra-klipper-firmware/Sources/klipper
make clean
make
# Nouveau binaire : out/klipper.bin — le renommer en Robin_nano_v3.bin avant de reflasher
```

## Points encore ouverts

- Canal laser (E1) : section `[output_pin laser]` laissée en commentaire dans `printer.cfg`, à compléter une fois le câblage/déclenchement du module laser décidé.
- Position exacte du "poop collector" dans la macro `START_PRINT` (`printer.cfg`) — actuellement X0/Y220 par défaut.
- Écran Robin TS35 côté Raspberry Pi (SPI + fbcp + KlipperScreen), indépendant du reste.

## Sources / références

- Pinout mainboard MKS Robin Nano V3.1 : https://1coderookie.github.io/Kobra2ProInsights/hardware/mainboard/#wiring-and-pin-assignment
- `printer.cfg` de référence (Robin Nano V3.1 + Kobra 2 Pro) : https://github.com/1coderookie/Klipper4Kobra2series
- Klipper : https://github.com/Klipper3d/klipper
- KIAUH : https://github.com/dw-0/kiauh

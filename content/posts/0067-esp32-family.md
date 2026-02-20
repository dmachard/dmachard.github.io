---
title: "Le monde des microcontrolleurs ESP32"
summary: "Vue d’ensemble comparative de la famille ESP32 avec leurs principales différences techniques pour les projets IoT."
date: 2026-02-20T00:00:00+01:00
draft: false
tags: ['esp', 'iot']
pin: false
---

# Vue d'ensemble des microcontrolleurs ESP32

Ma préférence pour des projets IoT

### ESP32-C3 : 
- **Avantages** : Prix , processeur RISC-V, BLE, WiFi
- **Limitations** : Moins de GPIO (11), moins de RAM (400 KB)

### ESP32-C6 : plus de connectivité
- **Avantages** : WiFi 6, Zigbee, Thread/Matter


## Comparaison ESP32

| Caractéristique | ESP32 (V1) | ESP32-S2 | ESP32-S3 | ESP32-C3 | ESP32-C5 | ESP32-C6 | ESP32-H2 | ESP32-P4 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Processeur** | Xtensa Dual-Core 32-bit @ 240 MHz | Xtensa Single-Core 32-bit @ 240 MHz | Xtensa Dual-Core 32-bit @ 240 MHz | RISC-V Single-Core 32-bit @ 160 MHz | RISC-V Dual-Core 32-bit @ 300 MHz | RISC-V Dual-Core 32-bit @ 160 MHz | RISC-V Single-Core 32-bit @ 96 MHz | Xtensa Dual-Core 32-bit @ 360 MHz |
| **Année** | 2016 | 2019 | 2021 | 2021 | 2024 | 2023 | 2023 | 2024 |
| **Prix (DevKit)** | **~4 - 6 €** | **~4 - 5 €** | **~6 - 9 €** | **~3 - 5 €** | **~6 - 8 €** | **~5 - 7 €** | **~5 - 7 €** | **~15 - 20 €** |
| **Bluetooth Classic** | ✅ (4.2) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Bluetooth LE** | ✅ (4.2) | ❌ | ✅ (5.0) | ✅ (5.0) | ✅ (5.4) | ✅ (5.3) | ✅ (5.3) | ✅ (5.3)* |
| **Zigbee** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| **Wi-Fi** | 802.11 b/g/n | 802.11 b/g/n | 802.11 b/g/n | 802.11 b/g/n | 802.11ax (Dual-Band¹) | 802.11ax (Wi-Fi 6²) | ❌ | ❌ |
| **DAC (Audio Ana.)** | ✅ (2x 8-bit) | ✅ (2x 8-bit) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Deep Sleep (μA)** | ~10 μA | ~20 μA | ~7 μA | ~5 μA | ~5 μA | ~7 μA | ~7 μA | ~8 μA |
| **Sécurité Mat.** | AES, RSA, SHA | AES, RSA, ECC | AES, RSA, ECC | AES, RSA, ECC | AES, ECC, DS | AES, ECC, DS | AES, ECC, DS | AES, ECDSA, TEE |
| **RAM (SRAM)** | 520 KB | 320 KB | 512 KB | 400 KB | 400 KB | 512 KB | 320 KB | **768 KB** |
| **Flash** | 4/8/16 MB | 4/8/16 MB | 8/16 MB | 4 MB | 4/8 MB | 4/8 MB | 4 MB | **16 MB** |
| **GPIO** | **34** | **43** | **45** | **11** | **30** | **30** | **14** | **54** |
| **ADC** | 12 ch. | 20 ch. | 20 ch. | 6 ch. | 6 ch. | 6 ch. | 6 ch. | Variable |
| **Consommation Active (mA)** | ~85 | ~80 | ~100 | ~45 | ~50 | ~40 | ~20 | **~200** |
| **Thread / Matter** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| **USB Natif** | ❌ | ✅ USB OTG | ✅ **USB OTG** | ✅ Serial/JTAG | ✅ Serial/JTAG | ✅ Serial/JTAG | ✅ Serial/JTAG | ✅ **USB 2.0 HS** |
| **Datasheet** | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-s2_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-c5_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-c6_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-h2_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-p4_datasheet_en.pdf) |

**¹ Dual-Band** : WiFi 6 sur deux bandes de fréquence (2.4 GHz + 5 GHz)  
**² Wi-Fi 6** : WiFi 6 (802.11ax) sur bande unique standard

## Définitions

### Flash
La mémoire **Flash** est une mémoire non-volatile qui conserve les données même après extinction de l'appareil. Elle stocke le firmware et les données de l'application. La plupart des ESP32 proposent des variantes avec différentes capacités (4 MB, 8 MB ou 16 MB).

### USB OTG
**USB On-The-Go** (OTG) permet à l'appareil de fonctionner à la fois en tant que maître USB (pour connecter des périphériques) et en tant que périphérique (pour se connecter à un ordinateur). C'est utile pour les applications nécessitant une plus grande flexibilité.

### JTAG
**Joint Test Action Group** est un protocole de programmation et de débogage utilisé pour charger le firmware et accéder à des informations de débogage pendant le développement. Sur les ESP32 C, H et P, il est intégré via le port Serial/JTAG.

### RISC-V
**RISC-V** (Reduced Instruction Set Computer - Five) est une architecture de processeur open-source basée sur le principe RISC. Contrairement à l'architecture Xtensa propriétaire, RISC-V offre une meilleure standardisation et une plus grande flexibilité.

### RAM (SRAM)
**SRAM** (Static Random-Access Memory) est la mémoire vive utilisée pour stocker les variables, les structures de données et l'exécution du code en cours. Contrairement à la Flash, elle est volatile (perd ses données lors de l'extinction). Les ESP32 ont généralement 320 KB à 768 KB de SRAM.

### Sécurité Matérielle
Les ESP32 intègrent des fonctionnalités de **sécurité matérielle** comme l'accélération cryptographique (AES, RSA, ECC), le hachage (SHA) et les éléments de sécurité (DS - Digital Signature, TEE - Trusted Execution Environment).

### ADC
L'**ADC** (Analog-to-Digital Converter) convertit les signaux analogiques en valeurs numériques. Chaque ESP32 dispose de plusieurs canaux ADC pour mesurer des tensions analogiques provenant de capteurs. Par exemple, l'ESP32 V1 a 12 canaux ADC, tandis que les ESP32-C en ont 6.

### USB 2.0 HS
**USB 2.0 High-Speed** est la version haute vitesse du standard USB 2.0, capable de communiquer à 480 Mbps. Seul l'ESP32-P4 dispose de cette interface native, offrant une bande passante considérablement plus élevée pour les transferts de données.

### Thread / Matter
**Thread** est un protocole réseau maillé (mesh) basé sur 802.15.4, tandis que **Matter** est un standard IoT unifié pour la domotique. Contrairement à Zigbee (qui est aussi 802.15.4 mais avec un écosystème différent), Thread/Matter est orienté vers la maison intelligente moderne et offre une meilleure interopérabilité entre appareils. Les ESP32-C6 et H2 supportent nativement Thread et Matter.

### NVS (Non-Volatile Storage)
**NVS** est un système de stockage persistant qui garde les données même après extinction. Il est utilisé pour sauvegarder les configurations, les paramètres WiFi, les clés de chiffrement, etc. Contrairement à la SRAM qui se vide, NVS permet de conserver des données critiques. C'est une partition de la mémoire Flash, généralement dimensionnée à 20-64 KB selon les besoins de l'application.

### Dual-Band 6 vs Wi-Fi 6
- **Dual-Band 6** : WiFi 6 (802.11ax) sur **deux bandes de fréquence** (2.4 GHz et 5 GHz). Offre plus de flexibilité et de débit.
- **Wi-Fi 6** : WiFi 6 standard, généralement sur une seule bande (2.4 GHz). Le C6 supporte WiFi 6 sur les deux bandes mais la notation "Wi-Fi 6" est simplifiée.

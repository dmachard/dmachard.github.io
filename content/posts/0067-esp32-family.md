---
title: "Comparaison ESP32"
summary: "Vue d’ensemble comparative de la famille ESP32 avec leurs principales différences techniques pour les projets IoT."
date: 2026-02-20T00:00:00+01:00
draft: false
tags: ['esp', 'iot']
pin: false
---

## Comparaison ESP32

| Caractéristique | ESP32 (V1) | ESP32-S2 | ESP32-S3 | ESP32-C3 | ESP32-C6 | ESP32-H2 | ESP32-P4 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Usage cible** | Standard / Polyvalent | USB / Low Power | IA / Performance | IoT Éco / Simple | Wi-Fi 6 / Matter | Zigbee / Thread | Écrans / Haute Perf |
| **CPU Architecture** | Dual Xtensa LX6 | Single Xtensa LX7 | **Dual** Xtensa LX7 | Single RISC-V | Single RISC-V | Single RISC-V | **Dual** RISC-V |
| **Fréquence Max** | 240 MHz | 240 MHz | 240 MHz | 160 MHz | 160 MHz | 96 MHz | **400 MHz** |
| **RAM (SRAM)** | 520 KB | 320 KB | 512 KB | 400 KB | 512 KB | 320 KB | **768 KB** |
| **PSRAM Support** | Oui (SPI) | Oui (SPI/QPI) | Oui (Octal/Quad) | ❌ | ❌ | ❌ | ✅ (Hte Vitesse) |
| **Wi-Fi** | 802.11 b/g/n | 802.11 b/g/n | 802.11 b/g/n | 802.11 b/g/n | **Wi-Fi 6** (2.4GHz) | ❌ | ❌ |
| **Bluetooth** | Classic + BLE 4.2 | ❌ | BLE 5.0 + Mesh | BLE 5.0 + Mesh | BLE 5.3 | BLE 5.3 | BLE 5.3 |
| **802.15.4** | ❌ | ❌ | ❌ | ❌ | ✅ (Zigbee/Thread) | ✅ (Zigbee/Thread) | ❌ |
| **USB Natif** | ❌ | ✅ USB OTG | ✅ **USB OTG** | ✅ USB Serial/JTAG | ✅ USB Serial/JTAG | ✅ USB Serial/JTAG | ✅ **USB 2.0 HS** |
| **Accél. IA** | ❌ | ❌ | ✅ (Vectoriel) | ❌ | ❌ | ❌ | ✅ (Puissant) |
| **GPIO (SoC)** | 34 | 43 | 45 | 22 | 30 | 19 | **50+** |
| **Sortie Vidéo** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ (MIPI-DSI/LCD) |
| **DAC** | 2 × 8 bits | 2 × 8 bits | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Datasheet** | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-s2_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-c6_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-h2_datasheet_en.pdf) | [Lien](https://www.espressif.com/sites/default/files/documentation/esp32-p4_datasheet_en.pdf) |
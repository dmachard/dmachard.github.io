---
title: "Bluetooth LE scanner et sniffing - partie 1"
summary: "Bluetooth LE scanner et sniffing sur ubuntu"
date: 2026-03-25T00:00:00+01:00
draft: false
tags: ['nRF52840', 'ble', 'ubuntu']
pin: false
---

# Bluetooth LE scanner et sniffing - partie 1

## Objectif 

- scanner les devices BLE depuis un laptop
- voir services / caractéristiques
- préparer le sniffing

## Prérequis

-   nRF52840 MDK USB Dongle  
    https://makerdiary.com/products/nrf52840-mdk-usb-dongle
    ![dongle](/images/0069/dongle.jpg)

-   nRF Connect (desktop)  
    https://github.com/NordicPlayground/pc-nrfconnect-ble-standalone/releases/

Lancer :

```bash
./nrfconnect-bluetooth-low-energy-4.0.4-x86_64.AppImage --no-sandbox
```

## Bluetooth Low Energy (BLE) ?

Contrairement au Bluetooth classique (audio, streaming…), BLE est pensé pour objets connectés, capteurs, etc...

Caractéristiques techniques clés
- Bande : ~2.4 GHz
- 40 canaux radio
- Débit max : ~1.4 Mbps
- Portée : jusqu’à ~1 km (selon config)
- Puissance max : 20 dBm

BLE est structuré en plusieurs parties : https://academy.nordicsemi.com/wp-content/uploads/2023/04/MicrosoftTeams-image-29.png
- GAP (Generic Access Profile) gère la visibilité et la connexion
    * scan (découverte des devices)
    * advertising
    * connexion / déconnexion
    * rôles (central / peripheral)

- GATT (Generic Attribute Profile) définit la structure des données
    * services
    * caractéristiques

- ATT (Attribute Protocol): le protocole bas niveau pour accéder aux données
    Opérations :
        - read
        - write
        - notify
        - indicate

## Scan

> Procédure complète et détaillée https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/getting-started/#connecting-the-dongle

* Pour effectuer un scan 

    1. brancher le dongle 
    2. ouvrir nRF Connect
    3. Cliquer sur le boutont "Start scan" pour découvrir les devices

    → liste des devices (advertising)

* Connexion à un device :
    - services GATT visibles
    - caractéristiques accessibles (read/write/notify)

    ![lego](/images/0069/scan_technicmove_lego.png)
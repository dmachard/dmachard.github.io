---
title: "NimBLE-DataPipe : Transport de données BLE pour ESP32"
summary: "Présentation de NimBLE-DataPipe, une bibliothèque légère pour échanger facilement des données JSON et binaires en BLE sans se soucier de la MTU ou de la fragmentation."
date: 2026-05-25T00:00:00+02:00
draft: false
tags: ['esp32', 'ble', 'iot', 'arduino']
pin: false
---

# NimBLE-DataPipe : Transport de données BLE pour ESP32

**NimBLE-DataPipe** est une couche de transport BLE légère pour l'ESP32. Elle permet de faire transiter des données au format **JSON** et **Binaire** sur une unique caractéristique BLE, en gérant automatiquement les limites de MTU et la fragmentation.

Le code source est disponible sur [GitHub](https://github.com/dmachard/NimBLE-DataPipe).

## Pourquoi DataPipe ?

- **Fragmentation automatique** : Les données sont découpées et réassemblées de manière transparente.
- **Support Bi-modal** : Prise en charge native des objets `ArduinoJson` et des buffers bruts (`uint8_t`).
- **Zéro-configuration** : Détection automatique de la meilleure MTU pour la connexion.

## Installation

### PlatformIO
```ini
lib_deps =
    h2zero/NimBLE-Arduino
    bblanchon/ArduinoJson
    NimBLE-DataPipe
```

## Entête du Protocole (3 octets)
Chaque message est précédé d'un en-tête technique de 3 octets :
`[TYPE (1 octet)][LONGUEUR (2 octets LE)]`

| Type | Nom | Description |
|------|------|-------------|
| `0x00`      | **JSON**   | Document structuré (compatible ArduinoJson) |
| `0x01-0xFF` | **Binary** | Modes applicatifs personnalisés |

## Démarrage rapide : Côté ESP32 (C++)

Cet exemple montre comment construire une interface de configuration complète.

```cpp
#include <NimBLE_DataPipe.h>

NimBLE_DataPipe bleDataPipe("ESP32-Config-Demo", "SERVICE-UUID", "CHAR-UUID");

void setup() {
  Serial.begin(115200);

  bleDataPipe.setOnJson([](const JsonDocument &doc) {
    String cmd = doc["cmd"] | "";

    if (cmd == "wifi_save") {
      String ssid = doc["ssid"] | "";
      String pass = doc["pass"] | "";
      Serial.printf("Sauvegarde WiFi : %s\n", ssid.c_str());
      
      JsonDocument res;
      res["status"] = "ok";
      bleDataPipe.sendJson(res);
    } 
    else if (cmd == "get_info") {
      JsonDocument res;
      res["type"] = "device_info";
      res["version"] = "1.0.2";
      res["free_heap"] = ESP.getFreeHeap();
      bleDataPipe.sendJson(res);
    }
  });

  bleDataPipe.begin();
}
```

## Démarrage rapide : Côté Web (JavaScript)

Utilisation de l'[API Web Bluetooth](https://developer.mozilla.org/en-US/docs/Web/API/Web_Bluetooth_API) pour communiquer avec DataPipe depuis un navigateur.

```javascript
const SERVICE_UUID = "votre-service-uuid";
const CHAR_UUID    = "votre-char-uuid";

let device, characteristic;

// --- Connexion ---
async function connect() {
  device = await navigator.bluetooth.requestDevice({
    filters: [{ services: [SERVICE_UUID] }]
  });
  const server = await device.gatt.connect();
  const service = await server.getPrimaryService(SERVICE_UUID);
  characteristic = await service.getCharacteristic(CHAR_UUID);

  // Écoute des indications de l'ESP32
  await characteristic.startNotifications();
  characteristic.addEventListener("characteristicvaluechanged", onReceive);
  console.log("Connecté");
}

// --- Réception (réassemblage des fragments) ---
let rxBuffer = new Uint8Array(0);
let expectedLen = 0;
let expectedType = 0;
let headerReceived = false;

function onReceive(event) {
  const chunk = new Uint8Array(event.target.value.buffer);

  // Ajout du fragment au buffer
  const tmp = new Uint8Array(rxBuffer.length + chunk.length);
  tmp.set(rxBuffer);
  tmp.set(chunk, rxBuffer.length);
  rxBuffer = tmp;

  // Lecture de l'en-tête dès qu'on a 3 octets
  if (!headerReceived && rxBuffer.length >= 3) {
    expectedType = rxBuffer[0];
    expectedLen = rxBuffer[1] | (rxBuffer[2] << 8);
    rxBuffer = rxBuffer.slice(3);
    headerReceived = true;
  }

  // Message complet ?
  if (headerReceived && rxBuffer.length >= expectedLen) {
    const payload = rxBuffer.slice(0, expectedLen);

    if (expectedType === 0x00) {
      const json = JSON.parse(new TextDecoder().decode(payload));
      console.log("JSON reçu :", json);
    } else {
      console.log(`Binaire reçu : type=${expectedType}, ${payload.length} octets`);
    }

    // Reset pour le prochain message
    rxBuffer = new Uint8Array(0);
    headerReceived = false;
  }
}

// --- Envoi de JSON ---
async function sendJson(obj) {
  const text = JSON.stringify(obj);
  const payload = new TextEncoder().encode(text);
  const len = payload.length;

  // En-tête : [TYPE=0x00][LEN_LO][LEN_HI] + payload
  const buffer = new Uint8Array(3 + len);
  buffer[0] = 0x00;
  buffer[1] = len & 0xFF;
  buffer[2] = (len >> 8) & 0xFF;
  buffer.set(payload, 3);

  await characteristic.writeValueWithResponse(buffer);
}

// --- Utilisation ---
await connect();
await sendJson({ cmd: "get_info" });
```


## Utilisation avancée : Configuration orientée classe

Pour des projets plus importants, il est conseillé d'encapsuler `NimBLE-DataPipe` dans une classe de gestion de configuration. Cela permet de garder un `main.cpp` propre et de centraliser la logique du protocole JSON.

### `BleConfig.h`
```cpp
#include <NimBLE_DataPipe.h>
#include <ArduinoJson.h>

class BleConfig {
public:
  BleConfig() : _pipe("MyDevice", "SERVICE_UUID", "CHAR_UUID") {}

  void begin() {
    _pipe.setOnJson([this](const JsonDocument &doc) {
      handleCommand(doc);
    });
    _pipe.begin();
  }

private:
  NimBLE_DataPipe _pipe;

  void handleCommand(const JsonDocument &doc) {
    String cmd = doc["cmd"] | "";
    
    if (cmd == "set_wifi") {
      // Logique pour sauvegarder le WiFi...
      JsonDocument res;
      res["status"] = "saved";
      _pipe.sendJson(res);
    }
  }
};
```

### `main.cpp`
```cpp
#include "BleConfig.h"

BleConfig bleConfig;

void setup() {
  bleConfig.begin();
}

void loop() {
  // Logique principale
}
```

## Mode Binaire

```cpp
bleDataPipe.setOnBinary([](uint8_t type, const uint8_t *data, size_t len) {
  if (type == 0x01) { 
     // Gérer votre mode binaire personnalisé
  }
});

// Envoi de données binaires brutes
uint8_t myBuffer[128] = { ... };
bleDataPipe.sendBinary(0x01, myBuffer, 128);
```

## Logs / Verbosité

Par défaut, la bibliothèque affiche ses messages d'état sur `Serial`. Il est possible de désactiver toutes les sorties en définissant `DATAPIPE_SILENT` **avant** d'importer la bibliothèque :

```cpp
#define DATAPIPE_SILENT
#include <NimBLE_DataPipe.h>
```

---
title: "NimBLE-DataPipe: Seamless BLE Data Transfer for ESP32"
summary: "How to easily stream JSON and binary payloads over BLE with automatic fragmentation, MTU negotiation, and zero setup."
date: 2026-05-25T00:00:00+02:00
draft: false
tags: ['esp32', 'ble', 'iot', 'arduino']
pin: false
---

# NimBLE-DataPipe: Seamless BLE Data Transfer for ESP32

**NimBLE-DataPipe** is a lightweight transport layer designed to eliminate this hassle. It lets you "pipe" both **JSON** and **Binary** payloads over a single BLE characteristic with zero configuration required.

The repository is hosted on [GitHub](https://github.com/dmachard/NimBLE-DataPipe).

---

## Key Benefits

*   **Transparent Fragmentation**: Send payloads up to 64KB without worrying about packet limits. DataPipe handles split-and-reassemble operations behind the scenes.
*   **Bi-modal Messaging**: Exchange structured `ArduinoJson` objects or raw binary buffers on the same channel.
*   **Automatic MTU Tuning**: DataPipe queries and uses the optimal MTU size dynamically based on the active connection.
*   **Indication-based Reliability**: Uses BLE Indications (GATT-level acknowledgments) to guarantee packet delivery.

---

## How It Works: The 3-Byte Protocol Header

Every message payload sent through the pipe is prepended with a minimal 3-byte header to identify the type and structure:

`[TYPE (1 byte)][LENGTH (2 bytes LE)]`

| Type | Mode | Description |
|------|------|-------------|
| `0x00` | **JSON** | Structured JSON document (fully compatible with ArduinoJson) |
| `0x01-0xFF` | **Binary** | Custom application modes defined by the user |

---

## Installation

Please refer to the [github repo](https://github.com/dmachard/NimBLE-DataPipe) for detailed installation instructions.

---

## Quick Start: ESP32 Implementation (C++)

Here is how to set up a complete bidirectional communication interface to save Wi-Fi configurations or fetch system information:

```cpp
#include <NimBLE_DataPipe.h>

// Instantiate with Device Name, Service UUID, and Characteristic UUID
NimBLE_DataPipe bleDataPipe("ESP32-Config-Demo", "SERVICE-UUID", "CHAR-UUID");

void setup() {
  Serial.begin(115200);

  // Set up the JSON callback handler
  bleDataPipe.setOnJson([](const JsonDocument &doc) {
    String cmd = doc["cmd"] | "";

    if (cmd == "wifi_save") {
      String ssid = doc["ssid"] | "";
      String pass = doc["pass"] | "";
      Serial.printf("Saving Wi-Fi credentials for: %s\n", ssid.c_str());
      
      JsonDocument response;
      response["status"] = "ok";
      bleDataPipe.sendJson(response);
    } 
    else if (cmd == "get_info") {
      JsonDocument response;
      response["type"] = "device_info";
      response["version"] = "1.0.2";
      response["free_heap"] = ESP.getFreeHeap();
      bleDataPipe.sendJson(response);
    }
  });

  // Start the BLE advertising and GATT service
  bleDataPipe.begin();
}
```

---

## Quick Start: Web Bluetooth Client (JavaScript)

To interact with the ESP32 from a web browser, we use the standard Web Bluetooth API. Below is a helper class structure to handle connection, chunk reassembly, and message sending:

```javascript
const SERVICE_UUID = "your-service-uuid";
const CHAR_UUID    = "your-char-uuid";

let device, characteristic;

// Connect to the device
async function connect() {
  device = await navigator.bluetooth.requestDevice({
    filters: [{ services: [SERVICE_UUID] }]
  });
  const server = await device.gatt.connect();
  const service = await server.getPrimaryService(SERVICE_UUID);
  characteristic = await service.getCharacteristic(CHAR_UUID);

  // Start listening for GATT indications
  await characteristic.startNotifications();
  characteristic.addEventListener("characteristicvaluechanged", onReceive);
  console.log("Connected to ESP32");
}

// Reassemble incoming chunks
let rxBuffer = new Uint8Array(0);
let expectedLen = 0;
let expectedType = 0;
let headerReceived = false;

function onReceive(event) {
  const chunk = new Uint8Array(event.target.value.buffer);

  // Concatenate new chunks
  const tmp = new Uint8Array(rxBuffer.length + chunk.length);
  tmp.set(rxBuffer);
  tmp.set(chunk, rxBuffer.length);
  rxBuffer = tmp;

  // Process header
  if (!headerReceived && rxBuffer.length >= 3) {
    expectedType = rxBuffer[0];
    expectedLen = rxBuffer[1] | (rxBuffer[2] << 8);
    rxBuffer = rxBuffer.slice(3);
    headerReceived = true;
  }

  // Check if the complete payload has arrived
  if (headerReceived && rxBuffer.length >= expectedLen) {
    const payload = rxBuffer.slice(0, expectedLen);

    if (expectedType === 0x00) {
      const json = JSON.parse(new TextDecoder().decode(payload));
      console.log("Received JSON:", json);
    } else {
      console.log(`Received Binary (type=${expectedType}):`, payload);
    }

    // Reset buffer for the next incoming transmission
    rxBuffer = new Uint8Array(0);
    headerReceived = false;
  }
}

// Send JSON payloads
async function sendJson(obj) {
  const text = JSON.stringify(obj);
  const payload = new TextEncoder().encode(text);
  const len = payload.length;

  // Build the frame: [TYPE][LEN_LO][LEN_HI] + payload
  const buffer = new Uint8Array(3 + len);
  buffer[0] = 0x00;
  buffer[1] = len & 0xFF;
  buffer[2] = (len >> 8) & 0xFF;
  buffer.set(payload, 3);

  await characteristic.writeValueWithResponse(buffer);
}
```

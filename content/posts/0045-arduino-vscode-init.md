---
title: "Procedure Arduino IDE 1.8.19 for Ubuntu 22.04 and VScode"
date: 2022-09-25T00:00:00+01:00
draft: false
tags: ['arduino', 'vscode', 'ide', 'esp']
---

# Install arduino IDE

```
# sudo apt update && sudo apt upgrade -y
# sudo snap install arduino
arduino 1.8.19 from Merlijn Sebrechts installed

# sudo usermod -a -G dialout <username>
```

# Configure Arduino

File > Preference 

	Additional Boards Manager URLS: https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

Tools > Board > Boards Manager
	Search esp32 and install the extension

Tools > Board > ESP32 Arduino > DOIT ESP32 DEVKIT V1 
Tools > Port > /dev/ttyUSB0

# Run example

File > Examples > 01.Basics > Blink
Sketch > Upload

```
Sketch uses 219037 bytes (16%) of program storage space. Maximum is 1310720 bytes.
Global variables use 16088 bytes (4%) of dynamic memory, leaving 311592 bytes for local variables. Maximum is 327680 bytes.
esptool.py v4.2.1
Serial port /dev/ttyUSB0
Connecting.....
Chip is ESP32-D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 30:c6:f7:30:4b:60
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 921600
Changed.
Configuring flash size...
Flash will be erased from 0x00001000 to 0x00005fff...
Flash will be erased from 0x00008000 to 0x00008fff...
Flash will be erased from 0x0000e000 to 0x0000ffff...
Flash will be erased from 0x00010000 to 0x00045fff...
Compressed 17472 bytes to 12125...
Writing at 0x00001000... (100 %)
Wrote 17472 bytes (12125 compressed) at 0x00001000 in 0.3 seconds (effective 400.3 kbit/s)...
Hash of data verified.
Compressed 3072 bytes to 128...
Writing at 0x00008000... (100 %)
Wrote 3072 bytes (128 compressed) at 0x00008000 in 0.1 seconds (effective 394.7 kbit/s)...
Hash of data verified.
Compressed 8192 bytes to 47...
Writing at 0x0000e000... (100 %)
Wrote 8192 bytes (47 compressed) at 0x0000e000 in 0.1 seconds (effective 531.5 kbit/s)...
Hash of data verified.
Compressed 219424 bytes to 120815...
Writing at 0x00010000... (12 %)
Writing at 0x0001d9f4... (25 %)
Writing at 0x00022fe7... (37 %)
Writing at 0x00028296... (50 %)
Writing at 0x0002d706... (62 %)
Writing at 0x00035f7c... (75 %)
Writing at 0x0003e05e... (87 %)
Writing at 0x000438f3... (100 %)
Wrote 219424 bytes (120815 compressed) at 0x00010000 in 2.1 seconds (effective 853.2 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

# Configure VScode

```
# sudo python3 -m pip install pyserial
```

Install the following extension: Arduino for Visual Studio Code

File > Preferences > Settings
Swap to json view and add the following line to settings.json 
in arduino.path the number 70 may vary.

```
{
....
    "arduino.path": "/snap/arduino/70",
    "arduino.commandPath": "arduino",
    "arduino.additionalUrls": [
    "https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json"
    ],

}
```

Ctrl+Shift+p and type Arduino: Board Manager and search esp32
Then install the available package

```
[Done] Installed board package - esp32
```
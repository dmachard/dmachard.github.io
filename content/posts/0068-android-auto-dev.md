---
title: "Android Auto Development Guide"
summary: "Steps to set up the development environment for Android Auto on Ubuntu"
date: 2026-03-17T00:00:00+01:00
draft: false
tags: ['android', 'auto']
pin: false
---

# Android Auto Development & Debugging Guide

This guide documents the steps to set up the development environment for Android Auto.

## Prerequisites

Java JDK and Android SDK components (Android Studio)

```bash
./android_sdk/cmdline-tools/latest/bin/sdkmanager --sdk_root=android_sdk "platform-tools" "platforms;android-35" "build-tools;35.0.0"
```

### Prepare the Phone
1.  Enable **Developer Options** in the Android Auto app (tap "Version" 10 times in AA settings).
2.  In the 3-dot menu, select **Start Head Unit Server**.
3.  Connect phone via USB (Mode: File Transfer / MTP).


## Android Auto Desktop Head Unit (DHU)

To simulate Android Auto on your computer without a car:

Install DHU (Desktop Head Unit)

```bash
./sdkmanager --sdk_root=android_sdk "extras;google;auto"
```

### Run the Simulator

1.  Forward the port:
    ```bash
    adb forward tcp:5277 tcp:5277
    ```

2.  Start DHU:
    ```bash
    ./desktop-head-unit
    ```

### Logs

- **View Logs:**
  ```bash
  adb logcat -v time
  ```
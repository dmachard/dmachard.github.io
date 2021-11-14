---
title: "Proxmox: Passthrough Physical Disk to Virtual Machine"
date: 2021-11-14T00:00:00+01:00
draft: false
tags: ['proxmox', 'disk']
---

How to attach physical disk to Virtual Machine on Promox

# Table of contents

* [Identify Disks](#identify-disks)
* [Attach disks](#attach-disks)
* [Setup on Windows](#setup-on-windows)

# Identify Disks

Identify all disks installed on the Promox server.

```bash
$ ls /dev/disk/by-id/ | grep ata
ata-ST3000VN000-1HJxxx_W6Axxxx
ata-ST3000VN000-1HJxxx_W6Axxx
ata-WDC_WD30EFRX-68xxx_WD-WCC1Txxxx
ata-WDC_WD30EFRX-68xxx_WD-WCC1Txxxx
ata-WDC_WD40EFRX-68xxx_WD-WCC7Kxxx
```

# Attach disks

Get the ID of the VM and attach the disk to them.

```bash
qm set 100 -virtio2 /dev/disk/by-id/ata-ST3000VN000-1HJxxx_W6Axxxx
qm set 100 -virtio3 /dev/disk/by-id/ata-WDC_WD30EFRX-68xxx_WD-WCC1Txxxx
```

# Setup on Windows

Download the latest stable Windows VirtIO drivers from https://github.com/virtio-win/virtio-win-pkg-scripts
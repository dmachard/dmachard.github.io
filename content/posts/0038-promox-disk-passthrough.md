---
title: "Passthrough Physical Disk to Virtual Machine on Proxmox"
summary: "How to attach physical disk to Virtual Machine on Promox"
date: 2021-11-14T00:00:00+01:00
draft: false
tags: ['proxmox', 'howto']
---

# Passthrough Physical Disk to Virtual Machine on Proxmox

How to attach physical disk to Virtual Machine on Promox

## Identify Disks

Identify all disks installed on the Promox server.

```bash
$ ls /dev/disk/by-id/ | grep ata
ata-TOSHIBA_MK1237GSX_775BT2QFT
```

Display size

```bash
$ lshw -class disk -class storage
*-disk
    description: SCSI Disk
    product: ASM235CM
    vendor: ASMT
    version: 0
    size: 111GiB (120GB)
```

## Attach disks

Get the ID of the VM and attach the disk to them.

```bash
qm set 100 -scsi2 /dev/disk/by-id/ata-ST3000VN000-1HJxxx_W6Axxxx
qm set 100 -scsi3 /dev/disk/by-id/ata-WDC_WD30EFRX-68xxx_WD-WCC1Txxxx
```

## Remove disk

```bash
qm unlink 100 --idlist scsi2
```

## Smart

```bash
$ smartctl --health --info /dev/disk/by-id/ata-ST950042xxxxxxxxxx
smartctl 7.3 2022-02-28 r5338 [x86_64-linux-6.5.11-7-pve] (local build)
Copyright (C) 2002-22, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Seagate Momentus 7200.4
Device Model:     xxxxxxxxx
Serial Number:    xxxxxx
LU WWN Device Id: 5 000c50 01xxxxxx
Firmware Version: 0004SDM1
User Capacity:    500,107,862,016 bytes [500 GB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    7200 rpm
Device is:        In smartctl database 7.3/5319
ATA Version is:   ATA8-ACS T13/1699-D revision 4
SATA Version is:  SATA 2.6, 3.0 Gb/s
Local Time is:    Mon Jan  1 18:12:41 2024 CET
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

## Setup on Windows

Download the latest stable Windows VirtIO drivers from https://github.com/virtio-win/virtio-win-pkg-scripts
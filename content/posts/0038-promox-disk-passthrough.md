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

## Setup on Windows

Download the latest stable Windows VirtIO drivers from https://github.com/virtio-win/virtio-win-pkg-scripts
---
title: "Proxmox: building a cloud init template with Almalinux8"
date: 2021-11-14T00:00:00+01:00
draft: false
tags: ['proxmox', 'almalinux', 'cloudinit']
---

How to build a cloud init template with Almalinux8

# Table of contents

* [Prerequisites](#build)
* [Build](#build)

# Prerequisites

Connect on your proxmox server and export the following variables

```bash
export CI_IMAGE="AlmaLinux-8-GenericCloud-latest.x86_64.qcow2"

export PM_STORAGE="local-lvm"

export VM_NAME="almalinux-8-cloudinit-template"
export VM_ID=1000
export VM_MEM=1024
export VM_OPTS=-"-net0 virtio,bridge=vmbr1"
```

# Build

Download a pre-configured image from AlmaLinux’s official repositories.

```bash
wget https://repo.almalinux.org/almalinux/8/cloud/x86_64/images/$CI_IMAGE
```

Create empty VM

```bash
qm create $VM_ID --name $VM_NAME --memory $VM_MEM $VM_OPTS
```

Import the downloaded image to the VM

```bash
$ qm importdisk $VM_ID $CI_IMAGE $PM_STORAGE
```

Assign the imported disk to scsi0

```bash
$ qm set $VM_ID -scsihw virtio-scsi-pci -scsi0 $PM_STORAGE:vm-$VM_ID-disk-0
```

Add special IDE device  for cloud-init :

```bash
$ qm set $VM_ID --ide2 $PM_STORAGE:cloudinit
```

Set it as the boot disk

```bash
$ qm set $VM_ID --boot c --bootdisk scsi0
```

VGA interface for console

```bash
$ qm set $VM_ID --serial0 socket --vga serial0
```

Convert to template:

```bash
$ qm template $VM_ID
```

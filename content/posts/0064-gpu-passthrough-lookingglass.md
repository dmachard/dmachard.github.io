---
title: "GPU Passthrough + Looking Glass Guide"
summary: "Setup Guide for Ubuntu"
date: 2025-06-16T00:00:00+01:00
draft: false
tags: ['passthrough', 'gpu', 'game', 'vm', 'qemu']
pin: false
---

# 🎮 GPU Passthrough + Looking Glass Guide

This guide explains how to set up a Windows VM with GPU passthrough and Looking Glass on Ubuntu 25.04.

## Prerequisites

- CPU with **VT-d/AMD-Vi** enabled in BIOS:

    ```bash
    lscpu | grep "Virtualization"
    Virtualization: VT-x   # Intel
    Virtualization: AMD-V  # AMD
    ```

- Two GPUs (one for Linux, one for the VM):

    ```bash
    lspci -nn | grep VGA
    00:02.0 VGA compatible controller [0300]: Intel Corporation Meteor Lake-P [Intel Arc Graphics] [8086:7d55] (rev 08)
    01:00.0 VGA compatible controller [0300]: NVIDIA Corporation AD106M [GeForce RTX 4070 Max-Q / Mobile] [10de:2820] (rev a1)
    ```

    Note the IDs in square brackets for the GPU you want to passthrough — in this case: `10de:2820`.

- Two audio controllers:

    ```bash
    lspci -nn | grep -i audio
    00:1f.3 Multimedia audio controller [0401]: Intel Corporation Meteor Lake-P HD Audio Controller [8086:7e28] (rev 20)
    01:00.1 Audio device [0403]: NVIDIA Corporation AD106M High Definition Audio Controller [10de:22bd] (rev a1)
    ```

    Again, note the IDs — in this case: `10de:22bd`.

- **HDMI Dummy Plug (optional)**: helpful for Looking Glass to trick the guest GPU into enabling display output.

## Isolating your GPU

### Install QEMU/KVM

QEMU is an open-source hypervisor that works with KVM to take advantage of hardware acceleration.

```bash
sudo apt update
sudo apt install qemu-kvm virt-manager bridge-utils linux-headers-$(uname -r) dkms
sudo reboot now
```

Enable libvirtd and add your user to the `libvirt` group:

```bash
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
sudo usermod -aG libvirt $USER
```

### Enable IOMMU in GRUB

IOMMU (Input-Output Memory Management Unit) is required to isolate and assign hardware devices to VMs securely.

Edit the GRUB configuration to enable IOMMU and specify the device IDs:

```bash
sudo vim /etc/default/grub
```

For **Intel** CPUs:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on iommu=pt vfio-pci.ids=10de:2820,10de:22bd"
```

For **AMD** CPUs, replace `intel_iommu=on` with `amd_iommu=on`.

Update GRUB:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot now
```

### Bind the GPU to VFIO

VFIO allows direct and secure assignment of devices to VMs.

Create the config:

```bash
sudo vim /etc/modprobe.d/vfio.conf
```

Add:

```text
options vfio-pci ids=10de:2820,10de:22bd
```

Blacklist other drivers to prevent them from binding to the GPU:

```bash
sudo tee /etc/modprobe.d/blacklist-nvidia.conf > /dev/null <<EOF
blacklist nouveau
blacklist nvidia
blacklist nvidiafb
blacklist rivafb
blacklist nvidia_drm
blacklist nvidia_uvm
blacklist nvidia_modeset
EOF
```

Update the initramfs:

```bash
sudo update-initramfs -c -k $(uname -r)
```

### Enable DMA GPU transfers (Looking Glass)

IVSHMEM (Inter-VM Shared Memory) is a QEMU feature allowing memory segments to be shared between the host and guest. Looking Glass uses it to facilitate high-speed, low-latency frame sharing via DMA (Direct Memory Access).

Download the Looking Glass source: [https://looking-glass.io/artifact/B7/source](https://looking-glass.io/artifact/B7/source)

Extract the archive and navigate to the `module/` directory:

```bash
cd looking-glass-B7/module
```

Install the kernel module:

```bash
dkms install .
```

Create `/etc/modprobe.d/kvmfr.conf`:

```bash
options kvmfr static_size_mb=128
```

Create `/etc/modules-load.d/kvmfr.conf`:

```bash
kvmfr
```

Create a udev rule to allow your user access to `/dev/kvmfr0` (replace `user` with your username):

```bash
sudo tee /etc/udev/rules.d/99-kvmfr.rules > /dev/null <<EOF
SUBSYSTEM=="kvmfr", OWNER="user", GROUP="kvm", MODE="0660"
EOF
```

### Verify GPU Isolation

Reboot and verify

Run:

```bash
lspci -k | grep -E "Audio|VGA|vfio-pci"
```

Expected output:

```text
01:00.0 VGA compatible controller: NVIDIA Corporation AD106M [...]
    Kernel driver in use: vfio-pci
01:00.1 Audio device: NVIDIA Corporation AD106M High Definition Audio [...]
    Kernel driver in use: vfio-pci
```

`vfio-pci` should be listed as the kernel driver in use.

You should now also have the character device /dev/kvmfr0

```bash
ls -l /dev/kvmfr0
```

Expected:

```text
crw------- 1 user kvm 242, 0 Jun 16 12:00 /dev/kvmfr0
```

## Create your dedicated gaming VM

### Download the Windows ISO

Get a copy of Windows from:

- [Windows 10](https://www.microsoft.com/en-us/software-download/windows10)
- [Windows 11](https://www.microsoft.com/en-us/software-download/windows11)

Download the `virtio-win` ISO from the [official repository](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md)

and move all ISO to `/var/lib/libvirt/images/`

```bash
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/*.iso
```

### Launch the VM with virt-install

```bash
sudo virt-install --name=Win10-gaming --vcpus=8 --memory=16384 --os-variant=win10 --features kvm_hidden=on --machine q35 --cdrom=/var/lib/libvirt/images/Win10_22H2_French_x64v1.iso --disk path=/var/lib/libvirt/images/win10.qcow2,size=100,bus=virtio,format=qcow2 --disk path=/var/lib/libvirt/images/virtio-win-0.1.271.iso,device=cdrom,bus=sata --graphics spice,listen=127.0.0.1 --video virtio --network network=default,model=virtio --hostdev pci_0000_01_00_0 --hostdev pci_0000_01_00_1 --wait -1
```

Command details

| Option | Purpose | Example Value |
| --- | --- | --- |
| `--name` | Name of the VM | `Win10-gaming` |
| `--vcpus` | Number of virtual CPUs exposed to the guest | `8` |
| `--memory` | Guest RAM in **MiB** (16 384 MiB ≈ 16 GB) | `16384` |
| `--os-variant` | Optimised defaults for the guest OS | `win10` |
| `--features kvm_hidden=on` | Hides KVM from the guest to avoid DRM/anti‑cheat issues | `kvm_hidden=on` |
| `--machine` | Chipset / machine type | `q35` |
| `--cdrom` | Windows installation ISO | `Win10_22H2_French_x64v1.iso` |
| `--disk` (1) | **System disk**: path, size (GiB), virtio bus | `win10.qcow2,size=100` |
| `--disk` (2) | **VirtIO driver ISO** attached as CD‑ROM | `virtio-win-0.1.271.iso,device=cdrom` |
| `--graphics` | Uses SPICE on localhost for display | `spice,listen=127.0.0.1` |
| `--video` | Video adapter model inside the VM | `virtio` |
| `--network` | Connects to default virtual network, virtio model | `network=default,model=virtio` |
| `--hostdev pci_0000_01_00_0` | **GPU passthrough** – PCIe address *0000:01:00.0* (NVIDIA RTX 4070) |  |
| `--hostdev pci_0000_01_00_1` | **Audio controller passthrough** – PCIe address *0000:01:00.1* (HD Audio) |  |
| `--wait -1` | Waits until installation completes before returning | `-1` |

### Install Windows

Use SPICE to connect and install Windows:

```bash
virt-viewer Win10-gaming
```

During installation:
- Browse to the VirtIO drive to load storage drivers when prompted.

### Install Drivers

Once inside Windows:
- Run `virtio-win-guest-tools.exe` from the VirtIO ISO.
- Install GPU drivers: [Nvidia](https://www.nvidia.com/en-us/geforce/drivers/), [AMD](https://www.amd.com/en/support/download/drivers.html), or [Intel](https://www.intel.com/content/www/us/en/download-center/home.html).

### Install LookingGlass

Looking Glass is an open source application that allows the use of a KVM (Kernel-based Virtual Machine) configured for VGA PCI Pass-through without an attached physical monitor, keyboard or mouse. 

On your VM, install Windows binary from https://looking-glass.io/artifact/B7/host

On your host, install the application client

```bash
mkdir host/build
cd host/build
cmake ..
make
make install
```

### Tune the VM

To optimize performance, edit the XML:

```xml
<cpu mode='host-passthrough' check='none' migratable='on'>
    <topology sockets='1' dies='1' clusters='1' cores='4' threads='2'/>
    <maxphysaddr mode='passthrough' limit='40'/>
    <feature policy='disable' name='hypervisor'/>
</cpu>
```

Modifying the XML domain namespace to support <qemu:commandline> block 

<domain type='kvm' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>

Add the following <qemu:commandline> block under `devices`

```xml
  <qemu:commandline>
    <qemu:arg value='-device'/>
    <qemu:arg value='ivshmem-plain,id=shmem0,memdev=looking-glass,bus=pcie.0,addr=0x11'/>
    <qemu:arg value='-object'/>
    <qemu:arg value='memory-backend-file,id=looking-glass,mem-path=/dev/kvmfr0,size=128M,share=yes'/>
  </qemu:commandline>
```

Then restart your VM.

### Final test

Connect your HDMI Dummy Plug
On your host, connect to your VM with `./looking-glass-client -F`


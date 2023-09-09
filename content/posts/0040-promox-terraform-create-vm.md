---
title: "Infra as code with Terraform and Proxmox"
summary: "This post details how to create a virtual machine with Terraform on your proxmox infrastructure."
date: 2021-11-15T00:00:00+01:00
draft: false
tags: ['proxmox', 'almalinux', 'cloudinit', 'terraform']
---

# Infra as code with Terraform and Proxmox

This post details how to create a virtual machine with Terraform on your proxmox infrastructure.

## Prerequisites

- Terraform installed on your system
- AlmaLinux cloudinit template on your promox system

## Configure

Create a *main.tf* file and install the provider [Telmate/promox](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs)

```tf
terraform {
  required_providers {
    proxmox = {
      source = "Telmate/proxmox"
      version = "2.9.0"
    }
  }
}
```

Run terraform init to install the provider.

```bash
terraform init
```

Configure the provider with the address of your promox and credentials

```tf
provider "proxmox" {
    pm_tls_insecure = true
    pm_api_url = "https://192.168.1.250:8006/api2/json"
    pm_password = "****"
    pm_user = "userapi@pam"
    pm_otp = ""
}
```

Create a VM ressource with specific CPU, memory and minimal network configuration.
A cloud-init template is used in this example.

```tf
resource "proxmox_vm_qemu" "machine01" {
    name = "machine01"
    desc = "Test server"

    target_node = "proxmox"
    pool = "Lab"

    clone = "almalinux-8-cloudinit-template"

    cores = 2
    sockets = 2
    memory = 2048
    onboot = true

    nameserver = "192.168.1.2"
    ipconfig0 = "ip=192.168.1.221/24,gw=192.168.1.1"

    ciuser = "osadmin"
    sshkeys = var.ssh_key
}
```

## Proxmox as Code

Finally run-it

```bash
terraform apply

proxmox_vm_qemu.machine01: Creating...
proxmox_vm_qemu.machine01: Still creating... [10s elapsed]
proxmox_vm_qemu.machine01: Creation complete after 18s [id=proxmox/qemu/106]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

## Cloud-init custom config

Add snippets to local storage

```
pvesm set local --content images,rootdir,vztmpl,backup,iso,snippets
```

An example is available [here](https://github.com/dmachard/terraform-samples/blob/main/proxmox/main_custom.tf) to provide a custom configuration for cloud-init.
---
title: "How to Deploy Vyos With Terraform on Proxmox Ve"
date: 2024-02-21T17:30:09+08:00
draft: true
image: https://picsum.photos/800/600.webp?random=e7e0ad45
categories:
  - proxmox ve
  - pve
  - vyos
  - terraform
tags:
  - proxmox ve
  - pve
  - vyos
  - terraform
  - iac
slug:
  - iac
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

## Install Terraform 




## Generate tf login token on pve node

```shell
pveum role delete TerraformProv
pveum user delete terraform-prov@pve
pveum role add TerraformProv -privs "Datastore.AllocateSpace Datastore.Audit Pool.Allocate Sys.Audit Sys.Console Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.Monitor VM.PowerMgmt"
pveum user add terraform-prov@pve
pveum aclmod / -user terraform-prov@pve -role TerraformProv
pveum user token add terraform-prov@pve terraform-token --privsep=0
```
Output:
```
┌──────────────┬──────────────────────────────────────┐
│ key          │ value                                │
╞══════════════╪══════════════════════════════════════╡
│ full-tokenid │ terraform-prov@pve!terraform-token   │
├──────────────┼──────────────────────────────────────┤
│ info         │ {"privsep":"0"}                      │
├──────────────┼──────────────────────────────────────┤
│ value        │ b092fe96-4c36-46c6-a477-b0bb5919e653 │
└──────────────┴──────────────────────────────────────┘
```

## Create `main.tf` file

```shell
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
    }
  }
}

provider "proxmox" {
  pm_tls_insecure     = true
  pm_api_url          = "https://192.168.11.53:8006/api2/json"
  pm_api_token_id     = "terraform-prov@pve!terraform-token"
  pm_api_token_secret = "xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}

resource "proxmox_vm_qemu" "proxmox-vyos" {
  count = 1
  name  = "vyos-${count.index + 1}"
  desc  = "vyos Iac Test environment"

  # PVE Node
  target_node = "debian"

  # cloud-init template
  clone = "template-vyos-1.3.6"

  # guest agent
  agent   = 0
  os_type = "cloudinit"
  onboot  = true
  # CPU
  cores    = 4
  sockets  = 1
  cpu      = "host"
  # mem
  memory   = 512
  scsihw   = "virtio-scsi-single"
  bootdisk = "scsi0"

  # disk 
  disk {
    slot     = 0
    size     = "2G"
    type     = "scsi"
    storage  = "SSD"
    iothread = 1
  }

  # newtork
  network {
    model  = "virtio"
    bridge = "vmbr0"
  }

  network {
    model  = "virtio"
    bridge = "vmbr1"
  }
  lifecycle {
    ignore_changes = [
      network,
    ]
  }
  #  set fix ip address
  ipconfig0 = "ip=192.168.11.9${count.index + 1}/24,gw=192.168.11.1"
  ipconfig1 = "ip=192.168.110.9${count.index + 1}/24,gw=192.168.110.1"

  # ssh key SSH key
  ciuser  = "user"
  sshkeys = <<EOF
  %%YOUR_SSH_KEY%%
  EOF
}
```

##  Apply

```shell
# init
terraform init

# format tf file
terraform fmt

# validate
terraform validate

terraform plan
terraform apply
```

## Destroy

```shell
terraform destroy
```


## Reference
---
  - [快速搭建实验环境：使用 Terraform 部署 Proxmox 虚拟机](https://atbug.com/deploy-vm-on-proxmox-with-terraform/)
  - [Proxmox: Deploying Infrastructure with Terraform | Ron Amosa](https://ronamosa.io/docs/engineer/LAB/proxmox-terraform/)
  - [Terraform Registry](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#enable-debug-mode-in-proxmox-api-go)
  - [使用terraform创建pve虚拟机](https://www.bboy.app/2023/08/14/%E4%BD%BF%E7%94%A8terraform%E5%88%9B%E5%BB%BApve%E8%99%9A%E6%8B%9F%E6%9C%BA/)
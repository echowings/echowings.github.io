---
title: "Some_tips_for_management_promxmoxve"
date: 2019-10-22T10:23:46+08:00
lastmode: 2019-10-22T10:23:46+08:00
draft: false
tags: [ "pve"]
categories: ["virtualization"]
reward: true
mathjax: true
---

# Some tips for management proxmxo VE



## Force shutdown a vm

```bash
#List all vm in pve
qm list 

# qm unlock the specific vm
qm unlock <vmid>

# qm stop the specific vm
qm stop <vmid>

# qm shutdown the specific vm
qm shutdown <vmid>


# Check vm status
qm list | grep <vmid>
```
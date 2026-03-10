---
title: "QEMU+KVM & Virtual Machine Manager on Linux Host"
date: 2026-03-10 11:39:00 +0700
categories: [Guide]
tags: [Linux, Windows, QEMU, Virtualization]
image: /assets/img/posts/qemu-virt-manager/banner.png
alt: "Linux Host Virtualization Setup with QEMU and virt-manager"
description: "Quick guide to setup QEMU and virt-manager to start playing with VM."
pin: false
---


## **What is QEMU, KVM & Virtual Machine Manager?**
> Check out [QEMU Documentation](https://www.qemu.org/docs/master/about/index.html) and [Virtual Machine Manager](https://virt-manager.org/)
{: .prompt-tip}

`Quick Emulator (QEMU)` is an open source virtualization and emulation tool, QEMU can emulate the hardware like CPU, memory, networks, and other devices. by using QEMU you can start deplyoying as much Virtual Machines (VMs) as you want.

`Kernel based Virtual Machine (KVM)` is an virtualization technology for Linux OS that function as hypervisor enabling multiple creation of VMs.

`Virtual Machine Manager (VMM)` is a desktop graphical user interface (GUI) for managing VMs

> **So in short, QEMU will use KVM and you as the user interact with the VMs through VMM.**

---

## **Why Use QEMU/KVM+VMM Instead of Virtualbox or VMWare Workstation?**

> **SPEED!!!**  
{: .prompt-warning}

With KVM, this turns the Linux Kernel into a Type-1 hypervisor, why this matter?  

With **Type-1** hypervisor, the hypervisor can directly access the resources (hardware) of Host OS (in this case our Laptop/PC). **Type-2** hypervisor on the other hand runs as an application and need talk with the Host OS, then the Host OS manage the resources.

> Learn more detailed Information [AWS Type 1 VS Type 2 Hypervisors](https://aws.amazon.com/compare/the-difference-between-type-1-and-type-2-hypervisors/)
{: .prompt-info}

---

## **Quick Setup for QEMU & Virtual Machine Manager**

> Check out Linux Mint [HERE](https://linuxmint.com/) and Linux Mint Debian Edition [HERE](https://linuxmint.com/download_lmde.php)
{: .prompt-tip}

This guide uses **Linux Mint Debian Edition (LMDE)** as the Host OS. Below are the steps to install QEMU and Virtual Machine Manager (VMM)

![Host OS](/assets/img/posts/qemu-virt-manager/fastfetch.png){: width="500" height="400"}
_Host OS Fastfetch_

### Update Your Package Manager

```bash
sudo apt update
```
This command refreshes your local package list with the latest available version from the repo.

### Install QEMU and VMM

> Check out QEMU Installation [HERE](https://www.qemu.org/download/#linux) and Virtual Machine Manager (VMM) Installation [HERE](https://virt-manager.org/)
{: .prompt-tip}

```bash
sudo apt install qemu-system virt-manager
```
This command installs **QEMU** and **Virtual Machine Manager (VMM)**

### That's It!

After installing `qemu-system` and `virt-manager`, you can start using the virtual machine manager and deploy your own VMs!

![VMM](/assets/img/posts/qemu-virt-manager/virtual-machine-manager.png){: width="500" height="400"}
_VMM GUI_

## Deploying Your First VM

Deployingg VM in QEMU/VMM is pretty straight forward, first just click the "Create a New virtual Machine" button top left of the VMM GUI.

![DeployVM1](/assets/img/posts/qemu-virt-manager/deployvm1.png){: width="500" height="400"}
_Create a New Virtual Machine_

After clicking that, there will be a new window pop up with a step by step instruction on how you want to deploy your VM, what OS the VM gonna be and what the specification of the VM you want.

![DeployVM2](/assets/img/posts/qemu-virt-manager/deployvm2.png){: width="500" height="400"}
_Choosing Install Media Source_

![DeployVM3](/assets/img/posts/qemu-virt-manager/deployvm3.png){: width="500" height="400"}
_Choosing ISO for the VM_

![DeployVM4](/assets/img/posts/qemu-virt-manager/deployvm4.png){: width="500" height="400"}
_Setting Up Memory & CPU Specification_

![DeployVM5](/assets/img/posts/qemu-virt-manager/deployvm5.png){: width="500" height="400"}
_Setting Up VM Storage_

![DeployVM6](/assets/img/posts/qemu-virt-manager/deployvm6.png){: width="500" height="400"}
_Settings Overview and VM Rename_

![DeployVM7](/assets/img/posts/qemu-virt-manager/deployvm7.png){: width="500" height="400"}
_Deploy VM Finished_

![DeployVM8](/assets/img/posts/qemu-virt-manager/deployvm8.png){: width="500" height="400"}
_New VM Deployed in VMM_

---

## Conclusion

VM are deployed, and it's running!. 

With that being said, feel free to create more VMs, break it, redeploy it, and test it. 

Do whatever you want!.


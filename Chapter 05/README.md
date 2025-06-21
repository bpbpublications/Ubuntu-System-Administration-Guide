
# Preparing a Virtualization Environment

## Introduction

Virtualization allows multiple operating systems (OS) to run simultaneously on a single physical machine. It is widely used in cloud computing, testing, and development environments. In this guide, you’ll learn how to set up KVM, create virtual machines (VMs), and configure them for various use cases. We'll also explore networking, snapshots, hardware passthrough, and alternative tools like Vagrant.

---

## Chapter Contents

- [Overview of Virtualization in Ubuntu](#overview-of-virtualization-in-ubuntu)
- [Installing KVM Virtualization](#installing-kvm-virtualization)
- [Creating a Bridge Network](#creating-a-bridge-network)
- [Installing Your First Virtual Machine (VM)](#installing-your-first-virtual-machine-vm)
- [Configuring VM Settings](#configuring-vm-settings)
- [Using Snapshots](#using-snapshots)
- [Accessing Your VM](#accessing-your-vm)
- [Passing Through Hardware Devices](#passing-through-hardware-devices)
- [Using Other Virtualization Tools](#using-other-virtualization-tools)
- [Building and Running a Vagrant Box Inside KVM](#building-and-running-a-vagrant-box-inside-kvm)
- [Converting Virtual Machine Images](#converting-virtual-machine-images)
- [Conclusion](#conclusion)

---

## Overview of Virtualization in Ubuntu

Virtualization enables the creation of virtual environments that allow a single physical machine to run multiple OS instances. It is ideal for:

- Running multiple versions of Ubuntu or other operating systems
- Testing server setups before production deployment
- Providing isolated environments for different projects or clients

Before proceeding, ensure your CPU supports virtualization:

```bash
sudo kvm-ok
```

If the result includes **"KVM acceleration can be used"**, you're good to go.

---

## Installing KVM Virtualization

KVM (Kernel-based Virtual Machine) is the native Linux virtualization engine.

1. Install KVM and supporting tools:

   ```bash
   sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
   ```

2. Add your user to the `libvirt` group:

   ```bash
   sudo adduser $USER libvirt
   ```

3. Verify virtualization support:

   ```bash
   sudo apt install cpu-checker
   sudo kvm-ok
   ```

---

## Creating a Bridge Network

Bridge networking allows VMs to access the host network directly.

1. Check current network connections:

   ```bash
   nmcli con show
   ```

2. Create a bridge:

   ```bash
   sudo nmcli con add ifname br0 type bridge con-name br0
   sudo nmcli con add type bridge-slave ifname ens3 master br0
   sudo nmcli con mod br0 bridge.stp no
   sudo nmcli con down ens3
   sudo nmcli con up br0
   ```

> Replace `ens3` with your actual network interface (e.g., `enp3s0`, `eth0`).

---

## Installing Your First Virtual Machine (VM)

1. Download an Ubuntu ISO from [ubuntu.com/download](https://ubuntu.com/download/desktop).
2. Open `virt-manager`.
3. Click **"Create a new virtual machine"** and follow the prompts.
4. Assign CPU, RAM, disk space, and select the **bridge network (br0)**.
5. Install and launch the VM.

---

## Configuring VM Settings

You can customize VM hardware and resources via `virt-manager`.

1. Select the VM → Click the **"i" icon** for settings.
2. Modify CPU, memory, disk, and network.
3. You can also add:
   - Additional storage
   - USB devices
   - Network interfaces
   - TPM modules

> Most changes require the VM to be shut down first.

---

## Using Snapshots

Snapshots preserve the current state of a VM, allowing quick restoration.

1. In `virt-manager`, select your VM and go to the **Snapshots** tab.
2. Click **"Take Snapshot"** and name it (e.g., `Stable`).
3. You can roll back to this state if needed.

---

## Accessing Your VM

### Option 1: SSH Access

1. Inside the VM, install SSH server:

   ```bash
   sudo apt install openssh-server
   ```

2. From the host, connect via:

   ```bash
   ssh username@<vm-ip-address>
   ```

### Option 2: Graphical Access

Use `virt-manager` or configure VNC for remote desktop access.

---

## Passing Through Hardware Devices

You can pass real hardware to a VM—useful for gaming, USB devices, or GPU compute tasks.

1. Open VM settings → Click **"Add Hardware"**.
2. Choose **PCI Host Device** (for GPU) or **USB Host Device**.
3. Select the device to attach it to the VM.

> You may need to enable IOMMU in your BIOS/UEFI for GPU passthrough.

---

## Using Other Virtualization Tools

### VirtualBox

A user-friendly, cross-platform alternative. Download from:  
[https://www.virtualbox.org](https://www.virtualbox.org)

### VMware Player

A lightweight virtualization tool. Get it from:  
[https://www.vmware.com/products/workstation-player.html](https://www.vmware.com/products/workstation-player.html)

> ⚠️ VirtualBox may conflict with KVM. Avoid running both simultaneously.

---

## Building and Running a Vagrant Box Inside KVM

[Vagrant](https://www.vagrantup.com/) is a tool for building portable dev environments.

1. Install Vagrant:

   ```bash
   sudo apt install vagrant
   ```

2. Set up a basic Vagrant environment:

   ```bash
   mkdir ~/vagrant && cd ~/vagrant
   vagrant init hashicorp/bionic64
   vagrant up
   ```

3. Vagrant will automatically use VirtualBox unless configured for libvirt.

To use KVM with Vagrant, install the `vagrant-libvirt` plugin:

```bash
vagrant plugin install vagrant-libvirt
```

---

## Converting Virtual Machine Images

Convert VM images between formats using `qemu-img`.

### Convert QCOW2 to VMDK (for VirtualBox):

```bash
qemu-img convert -p -f qcow2 -O vmdk ubuntu22.04.qcow2 ubuntu22.04.vmdk
```

### Convert VMDK back to QCOW2:

```bash
qemu-img convert -f vmdk -O qcow2 ubuntu22.04.vmdk ubuntu22.04.qcow2
```

---

## Conclusion

You’ve now learned how to:

- Install and configure KVM
- Create virtual machines
- Set up bridged networking
- Use snapshots and hardware passthrough
- Explore tools like Vagrant and VirtualBox
- Convert VM disk images across platforms

With this knowledge, you can build flexible, secure virtual environments for development, testing, or isolated services.

**Next Up:** Learn how to run Kubernetes and Docker inside your virtual machines.
```

```

# Install Ubuntu Server on Metal, Cloud, and Network

## Table of Contents
- [Introduction](#introduction)
- [Cattle vs. Pets](#cattle-vs-pets)
- [Using SSH to Connect to Your Server](#using-ssh-to-connect-to-your-server)
- [Installing Ubuntu Server with USB](#installing-ubuntu-server-with-usb)
- [Using Ubuntu in a Virtual Machine](#using-ubuntu-in-a-virtual-machine)
- [Ubuntu Server in Google Cloud and Hetzner](#ubuntu-server-in-google-cloud-and-hetzner)
- [Large Ubuntu Installations with MAAS](#large-ubuntu-installations-with-maas)

---

## Introduction

Ubuntu can be installed and run on cloud providers, physical hardware, and even small computers like the Raspberry Pi. The installation method varies depending on your target environment—whether it's a single server or a large-scale deployment.

In this guide, we cover several installation scenarios:

- **Cattle vs. Pets**: Understanding different approaches to server management.
- **SSH Keys**: Securely connecting to your servers.
- **Ubuntu Server Installations**: From USB, VMs, Google Cloud, and Hetzner.
- **Scaling with MAAS**: Managing large-scale Ubuntu installations.

---

## Cattle vs. Pets

In infrastructure management, the terms **"Cattle"** and **"Pets"** describe two server management philosophies.

### Pets
- Servers are treated like unique entities.
- You care for and maintain them over time.
- Common in personal or legacy systems.

### Cattle
- Servers are disposable and replaceable.
- Automation is prioritized—when a problem occurs, you destroy and recreate the server.
- Ideal for cloud-native and production environments.

**Recommendation:** In production, treat servers like **cattle** to improve scalability and manageability.

---

## Using SSH to Connect to Your Server

**SSH (Secure Shell)** is the standard method for securely accessing Ubuntu servers.

### Generate SSH Keys

```bash
ssh-keygen
```

This generates a private and public key pair, usually located at `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`.

### View the Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

Copy this key to your server or cloud provider's SSH configuration.

### Copy Your SSH Key to the Server

```bash
ssh-copy-id mahe@192.168.122.27
```

This lets you log in without entering a password.

### Connect to the Server

```bash
ssh mahe@192.168.122.27
```

---

## Installing Ubuntu Server with USB

### Steps:

1. **Download** the Ubuntu Server ISO from the [official website](https://ubuntu.com/download/server).
2. **Write to USB** using tools like **Balena Etcher** or **Rufus**.
3. **Boot from USB** and follow the installer.

   - Choose **Regular** or **Minimal** installation:
     - *Regular*: Full-featured system.
     - *Minimal*: Lightweight, ideal for server workloads (e.g., Docker).

4. **Disk Setup**: Use default or custom partitioning.
5. **Network Setup**: Connect to a network during install.
6. **Install OpenSSH** to enable remote access:
   ```text
   (Select "Install OpenSSH Server" during setup)
   ```

7. **Select Optional Packages** as needed.

---

## Using Ubuntu in a Virtual Machine

Ubuntu Server can be installed inside VMs using **libvirt/KVM**, **VirtualBox**, or **VMware**.

### Steps:

1. **Install Ubuntu Server** inside the VM using the ISO.
2. **Clone the VM** for rapid replication.
   ```text
   Figure 7.5: Cloning Ubuntu Server VM
   ```

3. **Deploy Clones** for testing or scaling.
   ```text
   Figure 7.6: Example of cloned server
   ```

---

## Ubuntu Server in Google Cloud and Hetzner

Platforms like **Google Cloud** and **Hetzner** provide easy-to-deploy Ubuntu Server images.

### Steps:

1. **Create a Cloud Instance** with Ubuntu.
2. **Add SSH Key** during instance creation.
3. **Configure Networking and Storage**.
4. **Connect via SSH** once the server is ready.

---

## Large Ubuntu Installations with MAAS

**MAAS (Metal as a Service)** is used for managing and provisioning multiple physical machines automatically.

### Steps:

1. **Install MAAS** on a host server.
2. **Configure Networking** and add your hardware.
3. **Provision Ubuntu** on many machines from a central dashboard.

---

# How to Set Up Ubuntu Server in Google Cloud and Hetzner

This section covers how to set up Ubuntu servers on **Google Cloud** and **Hetzner**, including how to connect via SSH.

## Table of Contents
- [Hetzner Cloud Setup](#hetzner-cloud-setup)
- [Google Cloud Setup](#google-cloud-setup)
- [SSH Access to Cloud Servers](#ssh-access-to-cloud-servers)
- [Additional Information](#additional-information)

---

## Hetzner Cloud Setup

### Step 1: Log in and Create Server

1. Sign in to the [Hetzner Cloud Console](https://console.hetzner.cloud/).
2. Select **Ubuntu** as the OS.
3. Choose server size and region.
4. Add your **SSH key**.

```text
Figure 7.7: Hetzner OS selection screen
```

---

### Step 2: Add SSH Key

During setup:

- Paste your public SSH key.
- This key will be used to access the server securely.

```text
Figure 7.8: Add SSH Key interface
```

---

### Step 3: Server Deployed

Once your server is running, retrieve:

- External IP address
- SSH access details

```text
Figure 7.9: Server details screen
```

---

### Step 4: SSH Into Hetzner Server

```bash
ssh root@37.27.8.127
```

This connects you to a live Ubuntu 22.04 LTS instance.

---

## Google Cloud Setup

### Step 1: Create Ubuntu Instance

1. Go to **Compute Engine** → **VM Instances**.
2. Click **Create Instance**.
3. Select Ubuntu under boot disk settings.

```text
Figure 7.10: Google Cloud Ubuntu selection
```

---

### Step 2: Connect via SSH

After creation, use the web console or your local terminal.

### Step 3: Add Your SSH Key

1. Go to **Metadata** → **SSH Keys**.
2. Add your public key.

```text
Figure 7.12: Google Cloud SSH Key screen
```

---

### Step 4: Connect from Local Machine

```bash
ssh USERNAME@EXTERNAL_IP
```

Replace `USERNAME` and `EXTERNAL_IP` accordingly.

---

## SSH Access to Cloud Servers

Steps are the same for both providers:

1. Retrieve the external IP.
2. Ensure SSH key is set.
3. Connect:

```bash
ssh USERNAME@EXTERNAL_IP
```

---

## Additional Information

### Production Considerations

For production:

- Enable firewalls/security groups.
- Set up regular backups and monitoring.
- Apply hardening techniques (fail2ban, UFW, etc.).

---

# How to Provision a VM with MAAS

A guide to provisioning Ubuntu VMs using **MAAS** and PXE boot via **KVM**.

## Table of Contents
- [Install MAAS on Ubuntu](#install-maas-on-ubuntu)
- [Initialize MAAS](#initialize-maas)
- [Create Admin User](#create-admin-user)
- [Set Up Network](#set-up-network)
- [Provision VM & PXE Boot](#provision-vm--pxe-boot)
- [Power Management](#power-management)
- [Conclusion](#conclusion)

---

## Install MAAS on Ubuntu

```bash
sudo snap install --channel=3.3 maas
```

---

## Initialize MAAS

### Help Command

```bash
maas init --help
```

### Install Demo DB

```bash
sudo snap install maas-test-db
sudo maas init region+rack --database-uri maas-test-db:///
```

---

## Create Admin User

```bash
sudo maas createadmin
```

Follow prompts for username, password, email, and SSH key.

---

## Set Up Network

### Configure Virtual Network

1. Use Virtual Manager to create a **provision** network.
2. Disable DHCP, set subnet (e.g., `10.33.33.0/24`).

### Assign Static IP

```bash
ip a  # Identify interface name
sudo nano /etc/netplan/00-installer-config.yaml
```

Update config:

```yaml
network:
  ethernets:
    enp1s0:
      dhcp4: true
    enp7s0:
      dhcp4: no
      addresses: [10.33.33.2/24]
  version: 2
```

Apply config:

```bash
sudo netplan apply
```

---

## Provision VM & PXE Boot

1. Create a new VM in KVM with the **provision** network.
2. Set boot to PXE.
3. Register VM in MAAS using MAC address.
4. Boot VM to trigger provisioning.

---

## Power Management

Connect MAAS

 to your KVM host to manage VM power states from the MAAS UI.

---

## Conclusion

You've learned how to:

- Install and configure MAAS
- Set up a network for PXE
- Provision Ubuntu on VMs using MAAS

For more, visit the [MAAS documentation](https://maas.io/).

---

Let me know if you want this as a downloadable `.md` file or if you'd like help turning it into a PDF or HTML version!
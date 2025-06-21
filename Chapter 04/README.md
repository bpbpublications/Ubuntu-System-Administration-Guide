
# Setting Up Firewall, VPN, and Wi-Fi Networks

This guide provides step-by-step instructions for configuring networking, securing connections with VPNs, and protecting your Ubuntu Desktop with firewall and antivirus tools.

---

## Table of Contents

1. [Introduction](#1-introduction)  
2. [Network Settings (DHCP/Static)](#2-network-settings-dhcpstatic)  
3. [Connect to VLAN Networks](#3-connect-to-vlan-networks)  
4. [Connect to Wireless Networks](#4-connect-to-wireless-networks)  
5. [Hide Your Computer by Changing MAC Address](#5-hide-your-computer-by-changing-mac-address)  
6. [Secure Your Connections with VPN Service](#6-secure-your-connections-with-vpn-service)  
7. [Protect Your Traffic Using DNS](#7-protect-your-traffic-using-dns)  
8. [Protect Your Computer Using a Firewall](#8-protect-your-computer-using-a-firewall)  
9. [Detect and Stop Computer Viruses](#9-detect-and-stop-computer-viruses)  
10. [Conclusion](#10-conclusion)

---

## 1. Introduction

Ubuntu offers powerful tools for managing your network, setting up VPNs, configuring firewalls, and enhancing privacy. This guide will help you secure your Ubuntu desktop in modern network environments.

---

## 2. Network Settings (DHCP/Static)

Ubuntu supports both dynamic and static IP addressing.

- **DHCP**: Automatically obtains an IP from the router.
- **Static IP**: Manually configured IP address for advanced control.

### Set a Static IP:

1. Open **Settings** → **Network** → Select your network interface.
2. Click the gear icon next to the interface.
3. Go to the **IPv4** tab.
4. Set **Method** to `Manual`.
5. Add the following:
   - **Address**: e.g., `192.168.1.49`
   - **Netmask**: `255.255.255.0`
   - **Gateway**: e.g., `192.168.1.1`
6. Save and reconnect.

---

## 3. Connect to VLAN Networks

Virtual LANs (VLANs) segment your network for better performance and security.

### Set Up a VLAN Interface:

1. Ensure your switch supports VLAN tagging.
2. Add a VLAN interface using this command:
   ```bash
   sudo ip link add link enp1s0 name enp1s0.10 type vlan id 10
   ```
3. Bring the VLAN interface up:
   ```bash
   sudo ip link set enp1s0.10 up
   ```
4. Assign IP manually or via DHCP.

---

## 4. Connect to Wireless Networks

### Connect to Wi-Fi:

1. Go to **Settings** → **Wi-Fi**.
2. Select your network and click **Connect**.
3. Enter the Wi-Fi password when prompted.
4. For advanced settings, click the gear icon next to the network.

---

## 5. Hide Your Computer by Changing MAC Address

Randomizing your MAC address helps improve privacy on public networks.

### Install and Use macchanger:

1. Install macchanger:
   ```bash
   sudo apt-get install macchanger
   ```
2. Create a script:
   ```bash
   sudo nano /etc/network/if-pre-up.d/macchanger
   ```
3. Paste the following:
   ```bash
   #!/bin/sh
   /usr/bin/macchanger -e "$IFACE"
   ```
4. Make it executable:
   ```bash
   sudo chmod +x /etc/network/if-pre-up.d/macchanger
   ```

---

## 6. Secure Your Connections with VPN Service

A VPN encrypts your connection, protecting you on public or untrusted networks.

### Install VPN Tools:

```bash
sudo apt-get install openvpn wireguard network-manager-openvpn-gnome
```

### OpenVPN Setup:

1. Sign up with a VPN provider (e.g., NordVPN, ProtonVPN).
2. Download their OpenVPN config files.
3. Import into Ubuntu:
   - Go to **Settings** → **Network** → **VPN** → **Add VPN** → **Import from file**.
4. Connect and verify with [What’s My IP](https://www.whatsmyip.org/).

### WireGuard Setup:

1. Go to **Settings** → **Network** → **VPN** → **Add VPN** → **WireGuard**.
2. Paste configuration provided by your VPN service.

---

## 7. Protect Your Traffic Using DNS

Secure DNS prevents third-party tracking and DNS spoofing.

### Use DNS over HTTPS (DoH) with `dnss`:

1. Install `dnss`:
   ```bash
   sudo apt install dnss
   ```
2. Start the server:
   ```bash
   sudo dnss --enable_dns_to_https --dns_listen_addr=:5553
   ```
3. Test with:
   ```bash
   dig google.com @127.0.0.1 -p5553
   ```
4. Edit `/etc/systemd/resolved.conf`:
   ```ini
   DNSStubListener=no
   ```
5. Restart systemd-resolved:
   ```bash
   sudo systemctl restart systemd-resolved
   ```

---

## 8. Protect Your Computer Using a Firewall

A firewall filters incoming and outgoing network traffic.

### Lock Down with iptables:

1. Create a script named `lock-down.sh`:
   ```bash
   nano lock-down.sh
   ```
2. Add:
   ```bash
   #!/bin/bash
   iptables -F
   iptables -X
   iptables -P INPUT DROP
   iptables -P OUTPUT DROP
   iptables -P FORWARD DROP
   iptables -A INPUT -i lo -j ACCEPT
   iptables -A OUTPUT -o lo -j ACCEPT
   iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
   iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
   iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT
   iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
   ```
3. Make executable and run:
   ```bash
   chmod +x lock-down.sh
   sudo ./lock-down.sh
   ```

> Tip: Use `ufw` (Uncomplicated Firewall) for simpler rule management.

---

## 9. Detect and Stop Computer Viruses

Linux systems can benefit from occasional antivirus scans, especially for file sharing with Windows systems.

### Install and Use ClamAV:

1. Install ClamAV and GUI:
   ```bash
   sudo apt-get install clamav clamtk
   ```
2. Update virus definitions:
   ```bash
   sudo freshclam
   ```
3. Run a manual scan:
   ```bash
   clamscan -r ~/Downloads
   ```
4. Launch **ClamTK** for scheduled or GUI scans.

---

## 10. Conclusion

You now know how to:

- Configure your network using DHCP or static IP.
- Connect to VLANs and wireless networks.
- Protect privacy using VPNs and MAC address randomization.
- Encrypt DNS traffic.
- Set up a firewall for secure traffic control.
- Detect and remove potential malware.

Your Ubuntu desktop is now better equipped to operate securely across various networks.
```

```
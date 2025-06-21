
# Set Up Advanced Networking, Firewall, and VPN Servers

## Introduction

Ubuntu Server can be used as a powerful firewall solution, often outperforming traditional shell-based firewalls. In this chapter, we'll use Ubuntu as our primary firewall to control inbound and outbound traffic. We’ll set up a DHCP server to manage IP address allocation and a DNS server to direct domain names to IP addresses of our choosing. Ubuntu also allows the creation of secure tunnels between servers and clients, enabling private network communication over the internet.

## Structure

In this chapter, we will cover:

- Using Ubuntu as the main firewall
- Setting up network clients with DHCP and DNS
- Securing communications with VPN
- Troubleshooting VPN connectivity

## Objectives

By the end of this chapter, you will be able to:

- Configure Ubuntu Server as a firewall to manage network traffic
- Deploy DHCP and DNS services to support both server and desktop clients
- Establish secure tunnels using OpenVPN and WireGuard
- Troubleshoot common VPN issues

---

## Using Ubuntu as a Firewall

At Fareoffice, we used a Linux-based firewall powered by `iptables` to handle traffic for our high-load sites. This allowed us to run a simple yet extensible Linux server to manage all network access.

To follow this setup, your Ubuntu Server will need two network interfaces—one for external (public) traffic and one for internal (private) networking.

### Checking Network Interface

You can inspect your network interfaces using:

```bash
ip -a
```

Example output:

```text
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:65:75:89 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.27/24 metric 100 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 3291sec preferred_lft 3291sec
    inet6 fe80::5054:ff:fe65:7589/64 scope link
       valid_lft forever preferred_lft forever
```

To add more flexibility, you can configure virtual NICs and VLANs.

### VLANs in Ubuntu

If your server has only one NIC, you can create virtual NICs (aliases). VLANs allow you to segment networks logically. Devices on the same VLAN can communicate, while others cannot.

Example VLAN configuration in `/etc/netplan/00-installer-config.yaml`:

```yaml
network:
  version: 2
  ethernets:
    enp1s0:
      dhcp4: true
    enp7s0:
      dhcp4: no
      addresses: [10.33.33.2/24]
  vlans:
    vlan.10:
      id: 10
      link: enp7s0
      addresses: [10.33.34.2/24]
```

Apply changes using:

```bash
sudo netplan apply
```

---

## Controlling Traffic with iptables

`iptables` allows us to filter and route traffic through our network interfaces.

Enable IP forwarding:

```bash
echo "1" > /proc/sys/net/ipv4/ip_forward
```

> **Tip:** Create a backup access method (e.g., screen + keyboard or virtual console) before applying firewall rules.

To prevent lockouts, you can also add a rule in `/etc/crontab` that flushes `iptables` daily:

```cron
00 1 * * * root iptables -F
```

### Basic Firewall Script

Create a script named `fw.sh`:

```bash
iptables -t nat -A POSTROUTING -s 10.33.33.0/24 ! -d 10.33.33.0/24 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.33.34.0/24 ! -d 10.33.34.0/24 -j MASQUERADE
iptables -A FORWARD -d 10.33.33.0/24 -o enp7s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -d 10.33.34.0/24 -o vlan.11@enp7s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -s 10.33.33.0/24 -i enp7s0 -j ACCEPT
iptables -A FORWARD -s 10.33.34.0/24 -i vlan.11@enp7s0 -j ACCEPT
```

Set default input/output policies:

```bash
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```

---

## Network Clients with DHCP and DNS

To support clients, we’ll install and configure `dnsmasq` for both DHCP and DNS services.

### Install dnsmasq

```bash
sudo apt install dnsmasq
```

### Configure dnsmasq

Edit the config file:

```bash
sudo vi /etc/dnsmasq.conf
```

Example settings:

```ini
dhcp-range=10.33.33.10,10.33.34.200,255.255.255.0,12h
dhcp-host=3C:98:72:F9:14:D8,server,10.33.33.12,infinite
dhcp-option=3,10.33.33.2
dhcp-option=6,10.33.33.2
```

### Disable systemd-resolved

```bash
systemctl stop systemd-resolved
systemctl disable systemd-resolved
systemctl mask systemd-resolved
```

Fix `/etc/resolv.conf`:

```bash
rm /etc/resolv.conf
echo "nameserver 1.1.1.1" > /etc/resolv.conf
```

To add custom DNS records, edit `/etc/hosts`:

```bash
10.100.0.40 sidero-cp
```

Test DNS:

```bash
dig google.com @127.0.0.1
```

---

# Securing Communications

We’ll now secure traffic using VPNs. Both **OpenVPN** and **WireGuard** will be covered.

## OpenVPN

Install OpenVPN and Easy-RSA:

```bash
sudo apt install openvpn easy-rsa
```

### Certificate Setup

```bash
mkdir /etc/pki/easy-rsa
cd /etc/pki/easy-rsa
ln -s /usr/share/easy-rsa/* .
```

Edit `vars`:

```bash
vi vars
```

Add:

```bash
set_var EASYRSA_ALGO "ec"
set_var EASYRSA_DIGEST "sha512"
```

Then:

```bash
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req server nopass
./easyrsa sign-req server server
openvpn --genkey --secret ta.key
```

### Configure OpenVPN Server

```bash
cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/
vi /etc/openvpn/server/server.conf
```

Update these lines:

```ini
ca /etc/pki/easy-rsa/pki/ca.crt
cert /etc/pki/easy-rsa/pki/issued/server.crt
key /etc/pki/easy-rsa/pki/private/server.key
dh /etc/pki/easy-rsa/dh2048.pem
tls-auth /etc/pki/easy-rsa/ta.key 0
```

Start the server:

```bash
sudo openvpn --config /etc/openvpn/server/server.conf
```

### Connect a Client

On the server:

```bash
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

Transfer `client1.crt`, `client1.key`, `ca.crt`, `ta.key` to the client and edit `/etc/openvpn/client/client.conf`:

```ini
remote vpn.server.robots.beer 1194
ca /etc/openvpn/tls/ca.crt
cert /etc/openvpn/tls/client1.crt
key /etc/openvpn/tls/client1.key
tls-auth /etc/openvpn/tls/ta.key 1
```

Start the client:

```bash
sudo openvpn --config /etc/openvpn/client/client.conf
```

Test VPN:

```bash
ip a
ping 10.8.0.1
```

---

## WireGuard VPN

Install WireGuard:

```bash
sudo apt install wireguard
```

### Key Generation

```bash
wg genkey | tee private.key | wg pubkey > public.key
chmod go= private.key
```

### Server Config (`wg0.conf`)

```ini
[Interface]
Address = 10.13.13.1
ListenPort = 51820
PrivateKey = <ServerPrivateKey>
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE
```

### Client Config

```ini
[Interface]
Address = 10.13.13.11
PrivateKey = <ClientPrivateKey>
DNS = 10.13.13.1

[Peer]
PublicKey = <ServerPublicKey>
PresharedKey = <PresharedKey>
Endpoint = <SERVER_IP>:51820
AllowedIPs = 10.13.13.0/24
```

Generate preshared key:

```bash
wg genpsk > preshared.key
```

### Add Peer on Server

```ini
[Peer]
PublicKey = <ClientPublicKey>
PresharedKey = <PresharedKey>
AllowedIPs = 10.13.13.11/32
```

### Start WireGuard

```bash
wg-quick down wg0
wg-quick up wg0
```

---

```
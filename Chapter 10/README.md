
# Running Virtualization Server Environments

## Introduction

Running multiple services like web servers and databases on a single Ubuntu server is a common need. To isolate each service securely, virtualization is essential. Virtualization lets you run multiple virtual servers (VMs), each in its own environment. Alternatively, containerization offers a lightweight way to isolate services.

This guide covers both virtualization using KVM and containerization with Podman, including monitoring setups with Grafana, Prometheus, and Loki.

---

## Structure

This guide includes:

- Installing KVM on Ubuntu Server
- Connecting to KVM from a desktop GUI
- Installing the Cockpit web interface
- Creating and managing virtual machines (VMs)
- Managing VMs with `virsh`
- Running dedicated VMs or considering tools like Proxmox or OpenStack
- Introduction to containers and Podman
- Monitoring setup with Grafana, Prometheus, Loki, and Promtail
- Extending monitoring to additional nodes

---

## Objectives

By the end, you will:

1. Set up Ubuntu as a virtual host.
2. Create and manage virtual machines.
3. Manage VMs via command line and remote tools.
4. Use Podman as a container runtime.
5. Set up full monitoring for hosts and containers using Grafana, Prometheus, Loki, and Promtail.

---

## Installing KVM on Ubuntu

### Step 1: Install KVM and Required Tools

```bash
sudo apt update
sudo apt -y install bridge-utils cpu-checker libvirt-clients libvirt-daemon-system qemu-kvm
```

### Step 2: Verify KVM Support

```bash
kvm-ok
```

Expected output:

```
INFO: /dev/kvm exists
KVM acceleration can be used
```

---

## Connecting to KVM from Desktop

### Step 1: Enable SSH Access and Add Key

- Add your SSH public key to the server.
- Add your user to the `libvirt` and `kvm` groups:

```bash
sudo usermod -aG libvirt,kvm $USER
```

### Step 2: Connect via Virt-Manager

1. Launch **Virtual Machine Manager** on your desktop.
2. Add a new remote SSH connection to the server.
3. You can now manage VMs remotely.

---

## Web Interface with Cockpit

### Step 1: Install Cockpit and VM Plugin

```bash
sudo apt install cockpit cockpit-machines
sudo systemctl enable --now cockpit.socket
```

### Step 2: Access the Web UI

Navigate to `http://<your-server-ip>:9090` in your browser and log in.

---

## Creating a VM

Use `virt-install` to create a VM:

```bash
sudo virt-install \
  --name ubuntu-guest \
  --os-variant ubuntu20.04 \
  --vcpus 2 \
  --ram 2048 \
  --location http://ftp.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/ \
  --network bridge=virbr0,model=virtio \
  --graphics none \
  --extra-args='console=ttyS0,115200n8 serial' \
  --disk size=5
```

---

## Managing VMs with `virsh`

### Clone or Backup a VM

```bash
virsh dumpxml ubuntu-guest > ubuntu-guest.xml
```

### Shared Storage via NFS (Example)

You can mount a shared disk image on multiple hosts to support shared VMs.

---

## Dedicated VM Solutions

For advanced environments:

- **Proxmox VE**: Easy to set up and manage.
- **OpenStack**: Scalable and modular, ideal for cloud infrastructure.

---

## Containers with Podman

### Install Podman

```bash
sudo apt install podman
```

### Verify Installation

```bash
podman --version
```

---

## Key Podman Features

- Docker-compatible CLI
- Rootless containers
- OCI-compliant
- Native pod support
- REST API support
- Daemonless architecture

---

## Podman CNI Error Fix

If you encounter CNI errors:

```bash
sudo nano /etc/cni/net.d/87-podman-bridge.conflist
```

Ensure:

```json
"cniVersion": "0.4.0"
```

---

# Monitoring Setup

## Prerequisites

- Podman or Docker
- Basic YAML and Compose knowledge

---

## 1. Setup Folder Structure

```bash
mkdir -p /opt/monitoring
cd /opt/monitoring
```

Create `docker-compose.yaml`.

---

## 2. Grafana Service

```yaml
services:
  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    ports:
      - "3000:3000"
    volumes:
      - ./grafana:/var/lib/grafana
```

```bash
podman-compose up
```

Access at: `http://<SERVER_IP>:3000`

---

## 3. Add Prometheus

### Config File: `prom/prometheus.yml`

```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

### Add to Compose:

```yaml
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prom:/etc/prometheus
    ports:
      - "9090:9090"
```

---

## 4. Add Node Exporter

```yaml
  node-exporter:
    image: prom/node-exporter:v1.6.1
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
```

Restart the stack:

```bash
podman-compose up
```

---

## 5. Add Loki and Promtail

### Loki Service

```yaml
  loki:
    image: grafana/loki:2.9.1
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - ./loki:/loki:Z
```

### Promtail Service

```yaml
  promtail:
    image: grafana/promtail:2.9.1
    volumes:
      - ./promtail:/etc/promtail
      - /var/log:/var/log
    command: -config.file=/etc/promtail/config.yml
```

---

## 6. Promtail Config for Journal Logs

Create `promtail/config.yml`:

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets: ['localhost']
        labels:
          job: varlogs
          __path__: /var/log/*log

  - job_name: journal
    journal:
      path: /var/log/journal
      max_age: 12h
      labels:
        job: systemd-journal
    relabel_configs:
      - source_labels: ['__journal__systemd_unit']
        target_label: 'unit'
```

---

## 7. Extending Monitoring to Clients

On the client server (e.g., `g1`), deploy a similar stack with:

- **Promtail** (use the same `config.yml`, update Loki URL)
- **Node Exporter**

Update Prometheus on main server to include:

```yaml
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['192.168.1.11:9100', '10.0.0.17:9100']
```

---

## 8. Grafana Dashboards

- Add **Prometheus** and **Loki** as data sources
- Import dashboard ID `1860` for Node Exporter metrics
- Use **Explore** to view logs via Loki

---

This full-stack setup ensures visibility into both system and application metrics and logs across virtual machines and containers.
```

---



# Keep a Health Check on Your Ubuntu Server

## Table of Contents
- [Basic Monitoring Commands](#basic-monitoring-commands)
- [Cockpit Installation and Access](#cockpit-installation-and-access)
- [Install and Configure Grafana](#install-and-configure-grafana)
- [Install and Configure Prometheus](#install-and-configure-prometheus)
- [Install Node Exporter](#install-node-exporter)
- [Integrate Node Exporter with Prometheus](#integrate-node-exporter-with-prometheus)
- [Log Monitoring with Filebeat and Kibana](#log-monitoring-with-filebeat-and-kibana)
- [Install and Configure Fail2Ban](#install-and-configure-fail2ban)
- [OSSEC: Host Intrusion Detection System](#ossec-host-intrusion-detection-system)
- [Forward OSSEC Logs to Elasticsearch](#forward-ossec-logs-to-elasticsearch)

---

## Basic Monitoring Commands

### `top`

Displays live information about CPU, memory usage, running processes, and more.

```bash
top
```

---

### `netstat`

Shows active network connections, ports, and associated processes.

```bash
netstat -anp
```

---

### `lsof`

Lists open files and the processes that opened them.

```bash
lsof
```

---

### `du`

Helps determine folder sizes—great for disk usage troubleshooting.

```bash
du -h --max-depth=1
```

---

## Cockpit Installation and Access

### Install Cockpit

Cockpit is a lightweight web-based UI for monitoring and managing servers.

```bash
sudo apt install -t ${VERSION_CODENAME}-backports cockpit
```

### Access the Web GUI

Open your browser and navigate to:

```
https://<SERVER-IP>:9090
```

---

## Install and Configure Grafana

Grafana is a visualization tool for monitoring system and application metrics.

### Installation Steps

```bash
# Add Grafana GPG key
sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key

# Add Grafana repository
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

# Update and install
sudo apt update
sudo apt install grafana
```

### Restart Grafana Service

```bash
sudo systemctl restart grafana-server
```

---

## Install and Configure Prometheus

Prometheus collects metrics from various sources.

### Setup Steps

```bash
# Create Prometheus user and group
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus

# Create directories
sudo mkdir -p /var/lib/prometheus /etc/prometheus/{rules,rules.d,files_sd}
```

### Download and Install Prometheus

```bash
cd /tmp
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
tar xvf prometheus*.tar.gz
cd prometheus*/
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
```

### Create Prometheus systemd Service

```bash
sudo tee /etc/systemd/system/prometheus.service <<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

### Set Directory Permissions

```bash
for i in rules rules.d files_sd; do sudo chown -R prometheus:prometheus /etc/prometheus/${i}; done
for i in rules rules.d files_sd; do sudo chmod -R 775 /etc/prometheus/${i}; done
sudo chown -R prometheus:prometheus /var/lib/prometheus/
```

### Enable and Start Prometheus

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

### Optional: Stop Cockpit if Port Conflict

```bash
sudo systemctl stop cockpit.socket
sudo systemctl disable cockpit.socket
```

---

## Install Node Exporter

Node Exporter exposes OS-level metrics for Prometheus.

### Installation Steps

```bash
# Download and extract
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
tar zxvf node_exporter-1.6.0.linux-amd64.tar.gz
cd node_exporter-1.6.0.linux-amd64
sudo cp node_exporter /usr/local/bin

# Create user and set permissions
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### Create systemd Service File

```bash
sudo tee /etc/systemd/system/node_exporter.service <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```

### Enable and Start Node Exporter

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

### Test Node Exporter Output

```bash
curl http://127.0.0.1:9100/metrics
```

---

## Integrate Node Exporter with Prometheus

### Update Prometheus Configuration

Edit `/etc/prometheus/prometheus.yml`:

```yaml
- job_name: "g1"
  static_configs:
    - targets: ["localhost:9100"]
```

### Restart Prometheus

```bash
sudo systemctl restart prometheus
```

---

## Log Monitoring with Filebeat and Kibana

### Install Filebeat

```bash
sudo apt install filebeat
```

### Run Initial Setup

```bash
filebeat setup -e
```

### Start Filebeat

```bash
sudo systemctl restart filebeat
```

### Create a View in Kibana

1. Open **Kibana** → **Stack Management** → **Data Views**.
2. Create a new view to visualize logs.

---

## Install and Configure Fail2Ban

Fail2Ban protects your server against brute-force attacks.

### Install Fail2Ban

```bash
sudo apt install fail2ban
```

### Enable and Start Service

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Configure Fail2Ban

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Set the following:

```ini
backend = systemd
```

### Restart and Monitor Logs

```bash
sudo systemctl restart fail2ban
tail -f /var/log/fail2ban.log
```

---

## OSSEC: Host Intrusion Detection System

### Install Dependencies

```bash
sudo apt-get install build-essential make zlib1g-dev libpcre2-dev libevent-dev libssl-dev libsystemd-dev
```

### Download and Install OSSEC

```bash
cd /opt
mkdir ossec && cd ossec
wget https://github.com/ossec/ossec-hids/archive/3.7.0.tar.gz
tar -zxvf 3.7.0.tar.gz
cd ossec-hids-3.7.0/
./install.sh
```

> Choose **local** mode and follow prompts.

### OSSEC via apt (Optional)

```bash
# Server:
sudo apt install ossec-hids-server

# Agent (optional):
# sudo apt install ossec-hids-agent
```

### Restart OSSEC

```bash
/var/ossec/bin/ossec-control restart
```

### List Agents

```bash
/var/ossec/bin/agent_control -l
```

---

## Forward OSSEC Logs to Elasticsearch

### Enable JSON Logging

Edit `/var/ossec/etc/ossec.conf`:

```xml
<global>
  <jsonout_output>yes</jsonout_output>
</global>
```

### Restart OSSEC

```bash
/var/ossec/bin/ossec-control restart
```

### Configure Filebeat for OSSEC Logs

Edit Filebeat configuration:

```yaml
- type: log
  paths:
    - /var/ossec/logs/

alerts/alerts.json
  json.keys_under_root: true
  fields:
    log_type: osseclogs
```

---


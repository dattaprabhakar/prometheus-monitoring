Perfect — to **monitor 10 systems** (servers or VMs) with **Prometheus + Grafana**, you’ll need to:

1. **Install node\_exporter** on each of the 10 systems.
2. **Expose port 9100** from each system (default for node\_exporter).
3. **Update `prometheus.yml`** to scrape metrics from all 10 systems.

---

## 🧱 Step 1: Install Node Exporter on Each System

### On each of the 10 systems:

```bash
useradd --no-create-home --shell /bin/false node_exporter
cd /tmp
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar -xzf node_exporter-1.8.1.linux-amd64.tar.gz
cp node_exporter-1.8.1.linux-amd64/node_exporter /usr/local/bin/
chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### Create systemd service:

```bash
cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
EOF

systemctl daemon-reload
systemctl enable --now node_exporter
```

Confirm it's running on: `http://<host_ip>:9100/metrics`

---

## 🧱 Step 2: Update `prometheus.yml` to Scrape All 10 Hosts

Example for 10 hosts:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'nodes'
    static_configs:
      - targets:
        - '192.168.1.101:9100'
        - '192.168.1.102:9100'
        - '192.168.1.103:9100'
        - '192.168.1.104:9100'
        - '192.168.1.105:9100'
        - '192.168.1.106:9100'
        - '192.168.1.107:9100'
        - '192.168.1.108:9100'
        - '192.168.1.109:9100'
        - '192.168.1.110:9100'
```

> Replace the IPs with your actual system IPs.

---

## 🧱 Step 3: Restart Prometheus Container

```bash
docker compose down
docker compose up -d
```

Prometheus will now start scraping metrics from all 10 systems.

---

## 🧱 Step 4: View in Grafana

1. Login to Grafana.
2. Go to **Dashboards → Import**.
3. Use Dashboard ID: **1860** (Node Exporter Full).
4. Choose Prometheus as the data source.

---

Would you like a **pre-built script** to install `node_exporter` on all 10 systems over SSH?

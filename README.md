# Deploy Prometheus and Grafana on Ubuntu 22.04 LTS with Node Exporter — Configure Grafana Dashboard
Guide to deploy infrastructure for monitoring server and node performance

![cover-grafana-prometheus-node_exporter_2](https://github.com/user-attachments/assets/b36dc858-3feb-4322-a097-25203aad3c29)



## Prepare Server by Configurating Firewall
```
sudo ufw allow ssh
```
```
sudo ufw enable
```
```
sudo ufw status verbose
```

## Prometheus
### Step #1: Creating Prometheus System Users and Directory
Create a system user for Prometheus

```
sudo useradd --no-create-home --shell /bin/false prometheus
```
```
sudo mkdir /etc/prometheus
```
```
sudo mkdir /var/lib/prometheus
```
```
sudo chown prometheus:prometheus /etc/prometheus
```
```
sudo chown prometheus:prometheus /var/lib/prometheus
```
### Step #2: Download Prometheus Binary File and Check version

```
cd /tmp/
```
```
wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
```
```
tar -xvf prometheus-2.46.0.linux-amd64.tar.gz
```
```
cd prometheus-2.46.0.linux-amd64
```
```
sudo mv console* /etc/prometheus
```
```
sudo mv prometheus.yml /etc/prometheus
```
```
sudo chown -R prometheus:prometheus /etc/prometheus
```
```
sudo mv prometheus /usr/local/bin/
```
```
sudo mv promtool /usr/local/bin/
```
```
sudo chown prometheus:prometheus /usr/local/bin/prometheus
```
```
sudo chown prometheus:prometheus /usr/local/bin/promtool
```
```
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
```
```
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```
```
sudo chown -R prometheus:prometheus /etc/prometheus/prometheus.yml
```
```
prometheus --version && promtool --version
```

### Step #3: Prometheus configuration file

We have already copied /opt/prometheus-2.46.0.linux-amd64/prometheus.yml file /etc/prometheus directory, verify if it present and  modify it as per your requirement.
```
nano /etc/prometheus/prometheus.yml
```
```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ["localhost:9090", "localhost:9200"]

  - job_name: "node_exporter"
    static_configs:
    - targets: [server-ip1:9200]
      labels:
        label: "<label_1>"
    - targets: [server-ip2:9200]
      labels:
        label: "<label_2>"
    - targets: [server-ip3:9200]
      labels:
        label: "<label_3>"
```

### Step #4: Creating Prometheus Systemd file

```
sudo nano /etc/systemd/system/prometheus.service
```
Create the service file
```
[Unit]

Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]

User=prometheus
Group=prometheus
Type=simple

ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]

WantedBy=multi-user.target
```
```
ufw allow 9090/tcp
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl start prometheus
```
```
sudo systemctl enable prometheus
```
```
sudo systemctl status prometheus
```

http://server-ip-address:9090

## Node Exporter Setup
```
sudo groupadd -f node_exporter
```
```
sudo useradd -g node_exporter --no-create-home --shell /bin/false node_exporter
```
```
sudo mkdir /etc/node_exporter
```
```
sudo chown node_exporter:node_exporter /etc/node_exporter
```
```
wget \
  https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```
```
tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
```
```
mv node_exporter-1.8.2.linux-amd64 node_exporter-files
```
```
sudo cp node_exporter-files/node_exporter /usr/bin/
```
```
sudo chown node_exporter:node_exporter /usr/bin/node_exporter
```
```
sudo nano /usr/lib/systemd/system/node_exporter.service
```
```
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
ExecStart=/usr/bin/node_exporter \
  --web.listen-address=:9200

[Install]
WantedBy=multi-user.target
```
```
sudo chmod 664 /usr/lib/systemd/system/node_exporter.service
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl start node_exporter
```
```
sudo systemctl status node_exporter
```
```
sudo systemctl enable node_exporter.service
```
```
sudo systemctl enable node_exporter.service
```
```
ufw allow 9200/tcp
```
```
nano /etc/prometheus/prometheus.yml
```

`- targets: [‘localhost:9090’, ‘localhost:9200’]`

```
sudo systemctl restart prometheus
```

[https://node_exporter-ip:9200/metrics](http://node_exporter-ip:9200/metrics)

```
rm -rf node_exporter-1.8.2.linux-amd64.tar.gz node_exporter-files
```

## Grafana
```
cd ~
```
```
sudo apt install -y apt-transport-https
```
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```
```
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```
```
sudo apt-get update
```
```
sudo apt-get install grafana -y
```
```
sudo systemctl enable --now grafana-server
```
```
sudo systemctl start grafana-server
```
```
sudo systemctl status grafana-server
```
```
sudo systemctl enable grafana-server.service
```
```
ufw allow 3000/tcp
```

http://server-ip-address:3000


Configure Prometheus as a Data Source:

In the Grafana web interface, click on “Connections” > “Data Sources.” Add a new data source and choose Prometheus. Provide the URL of your Prometheus server (e.g., http://server-ip:9090) and save the configuration.

## Dashboard ID

### 11074
### 14513
### 1860

<img width="1351" alt="Screenshot 2024-11-04 at 16 47 48" src="https://github.com/user-attachments/assets/a3f5e0b8-ba9b-4bbf-ad1c-56eb0fb684e6">

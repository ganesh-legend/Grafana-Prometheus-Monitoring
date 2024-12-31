### Node_Exporter setup:-

In node_exporter server , write below command

```bash
 nano /etc/systemd/system/node_exporter.service
```

Then copy following data inside it and save th file

```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
ExecStart=/etc/node_exporter/node_exporter
Restart=always
[Install]
WantedBy=multi-user.target
```

#

### Prometheus Setup:-

In Prometheus server , write below command

```bash
 sudo nano /etc/systemd/system/prometheus.service
```

Then copy following data inside it and save th file

```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
ExecStart=/etc/prometheus/prometheus --config.file=/etc/prometheus/prometheus.yml
Restart=always
[Install]
WantedBy=multi-user.target
```

setup scrape file of prometheus . Follow the below commands

```bash
sudo rm -rf /etc/prometheus/prometheus.yml
sudo nano /etc/prometheus/prometheus.yml
```

Now copy below data inside it and save it
###### This for Node Exporter configuration in prometheus      
```bash  
global:
  scrape_interval: 15s

scrape_configs:
- job_name: node
  static_configs:
  - targets: ['localhost:9100']
```

#

### Black Exporter setup:-

- You need to setup blackbox exporter systemd file.

```bash
[Unit]
Description=Blackbox Exporter Service
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=ubuntu
Group=ubuntu
ExecStart=/etc/blackbox_exporter/blackbox_exporter --config.file=/etc/blackbox_exporter/blackbox.yml

[Install]
WantedBy=multi-user.target
```

- You will copy next in prometheus.yml file. Copy carefully, understand indentation.

```bash
global:
  scrape_interval: 15s

scrape_configs:
- job_name: 'blackbox'
  metrics_path: /probe
  params:
    module: [http_2xx]  # Look for a HTTP 200 response.
  static_configs:
    - targets:
      - http://prometheus.io    # Target to probe with http.
      - https://prometheus.io   # Target to probe with https.
      - https://portfolio.ganeshpawar.one
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: 127.0.0.1:9115
```

#

### Setup Grafana Loki and promtail using docker

##### Create loki directory at your home directory.
```bash
cd ~
mkdir loki
cd loki
```

##### download configuration file loki and promtail
```bash
wget https://raw.githubusercontent.com/grafana/loki/v3.0.0/cmd/loki/loki-local-config.yaml -O loki-config.yaml
wget https://raw.githubusercontent.com/grafana/loki/v3.0.0/clients/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml
```

##### run docker containers for loki and promtail
```bash
docker run --name loki -d -p 3100:3100 -v $(pwd):/mnt/config --network loki-promtail grafana/loki:3.2.1 -config.file=/mnt/config/loki-config.yaml
docker run --name promtail -d -p 9080:9080 -v $(pwd):/mnt/config -v /var/log:/var/log --network loki-promtail grafana/promtail:3.2.1 -config.file=/mnt/config/promtail-config.yaml
```

```bash
```

You are done......😊
Thanks 🙏

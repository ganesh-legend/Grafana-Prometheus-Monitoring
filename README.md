### Node_Exporter setup:-

In node_exporter server , write below command

```bash
sudo nano /etc/systemd/system/node_exporter.service
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

#### This command according to your configuration. node_exporter what i have set up here, may differ in your case
```bash
sudo mv node_exporter.0.27-amd /etc/node_exporter
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

#### This command according to your configuration. prometheus what i have set up here, may differ in your case
```bash
sudo mv prometheus.0.27-amd /etc/prometheus
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
 sudo nano /etc/systemd/system/blackbox_exporter.service
```

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

#### This command according to your configuration. prometheus what i have set up here, may differ in your case
```bash
sudo mv blackbox_exporter.0.27-amd /etc/blackbox_exporter
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

### Alert Manager setup:-

- You need to setup alert manager systemd file.

```bash
 sudo nano /etc/systemd/system/alertmanager.service
```

```bash
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target
[Service]
ExecStart=/etc/alertmanager/alertmanager --config.file=/etc/alertmanager/alertmanager.yml
Restart=always
[Install]
WantedBy=multi-user.target
```

#### This command according to your configuration. prometheus what i have set up here, may differ in your case
```bash
sudo mv alertmanager.0.27-amd /etc/alertmanager
```

- You will copy next in prometheus.yml file. Copy carefully, understand indentation.
```bash
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9093'          # Alertmanager endpoint

rule_files:
   - "alert_rules.yml"                # Path to alert rules file
  # - "second_rules.yml"              # Additional rule files can be added here
```

#

#### final prometheus file would look like
```bash
global:
  scrape_interval: 15s

scrape_configs:
- job_name: "prometheus"            # Job name for Prometheus

  # metrics_path defaults to '/metrics'
  # scheme defaults to 'http'.

  static_configs:
  - targets: ["localhost:9090"]   # Target to scrape (Prometheus itself)

- job_name: node
  static_configs:
  - targets: ['13.71.57.58:9100']

- job_name: 'blackbox'
  metrics_path: /probe
  params:
    module: [http_2xx]  # Look for a HTTP 200 response.
  static_configs:
    - targets:
      - https://portfolio.ganeshpawar.one
      - http://monitor.ganeshpawar.one:9090
      - http://monitor.ganeshpawar.one:3000
      - http://monitor.ganeshpawar.one:9115
      - http://monitor.ganeshpawar.one:9093
      - http://13.71.57.58:9100
      - http://13.71.57.58:8080
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: 127.0.0.1:9115

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9093'          # Alertmanager endpoint

rule_files:
   - "alert_rules.yml"                # Path to alert rules file
  # - "second_rules.yml"              # Additional rule files can be added here
```

#

#### set up alertmanager file /etc/alertmanager/alertmanager.yml
```bash
route:
  group_by: ['alertname']             # Group by alert name
  group_wait: 30s                     # Wait time before sending the first notification
  group_interval: 5m                  # Interval between notifications
  repeat_interval: 1h                 # Interval to resend notifications
  receiver: 'email-notifications'     # Default receiver

receivers:
- name: 'email-notifications'         # Receiver name
  email_configs:
  - to: gp420083@gmail.com       # Email recipient
    from: ganeshpawar06969@gmail.com              # Email sender
    smarthost: smtp.gmail.com:587     # SMTP server
    auth_username: ganeshpawar06969@gmail.com         # SMTP auth username
    auth_identity: ganeshpawar06969@gmail.com         # SMTP auth identity
    auth_password: "zhql dubm jbnu qirt"  # SMTP auth password
    send_resolved: true               # Send notifications for resolved alerts

inhibit_rules:
  - source_match:
      severity: 'critical'            # Source alert severity
    target_match:
      severity: 'warning'             # Target alert severity
    equal: ['alertname', 'dev', 'instance']  # Fields to match
```

# 

#### create alert_rules.yml file 
```bash
sudo nano /etc/prometheus/alert_rules.yml
```

```bash
groups:
- name: alert_rules                   # Name of the alert rules group
  rules:
    - alert: InstanceDown
      expr: up == 0                   # Expression to detect instance down
      for: 1m
      labels:
        severity: "critical"
      annotations:
        summary: "Endpoint {{ $labels.instance }} down"
        description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."

    - alert: WebsiteDown
      expr: probe_success == 0        # Expression to detect website down
      for: 1m
      labels:
        severity: critical
      annotations:
        description: The website at {{ $labels.instance }} is down.
        summary: Website down

    - alert: HostOutOfMemory
      expr: node_memory_MemAvailable / node_memory_MemTotal * 100 < 25  # Expression to detect low memory
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Host out of memory (instance {{ $labels.instance }})"
        description: "Node memory is filling up (< 25% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HostOutOfDiskSpace
      expr: (node_filesystem_avail{mountpoint="/"} * 100) / node_filesystem_size{mountpoint="/"} < 50  # Expression to detect low disk space
      for: 1s
      labels:
        severity: warning
      annotations:
        summary: "Host out of disk space (instance {{ $labels.instance }})"
        description: "Disk is almost full (< 50% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HostHighCpuLoad
      expr: (sum by (instance) (irate(node_cpu{job="node_exporter_metrics",mode="idle"}[5m]))) > 80  # Expression to detect high CPU load
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Host high CPU load (instance {{ $labels.instance }})"
        description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: ServiceUnavailable
      expr: up{job="node_exporter"} == 0  # Expression to detect service unavailability
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Service Unavailable (instance {{ $labels.instance }})"
        description: "The service {{ $labels.job }} is not available\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HighMemoryUsage
      expr: (node_memory_Active / node_memory_MemTotal) * 100 > 90  # Expression to detect high memory usage
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: "High Memory Usage (instance {{ $labels.instance }})"
        description: "Memory usage is > 90%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: FileSystemFull
      expr: (node_filesystem_avail / node_filesystem_size) * 100 < 10  # Expression to detect file system almost full
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "File System Almost Full (instance {{ $labels.instance }})"
        description: "File system has < 10% free space\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
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

#

### Setup grafana

```bash
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_11.4.0_amd64.deb
sudo dpkg -i grafana-enterprise_11.4.0_amd64.deb
```

#

You are done......😊
Thanks 🙏

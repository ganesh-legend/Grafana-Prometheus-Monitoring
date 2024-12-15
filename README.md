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
 
   2) setup scrape file of prometheus . Follow the below commands

     # sudo rm -rf /etc/prometheus/prometheus.yml

     # sudo nano /etc/prometheus/prometheus.yml

      now copy below data inside it and save it
      
```bash  
global:
  scrape_interval: 15s

scrape_configs:
- job_name: node
  static_configs:
  - targets: ['localhost:9100']
```



### Black Exporter setup:-

- You don't need to configure any blackbox exporter file. Install and run it.
- You will write next in prometheus.yml file

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

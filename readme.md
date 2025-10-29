# check-global-websites-grafana-prometheus

<img width="2335" height="1343" alt="image" src="https://github.com/user-attachments/assets/81c62a7b-d57e-484b-a051-4c31746e6b65" />

Quick prometheus-grafana project to do web observability project

https://github.com/HubDamian95/check-global-websites-grafana-prometheus/blob/main/grafana_dashboard/dashboard.json contains the dashboard.

https://github.com/HubDamian95/check-global-websites-grafana-prometheus/blob/main/prometheus/blackbox_http.json contains HTTP endpoints

https://github.com/HubDamian95/check-global-websites-grafana-prometheus/blob/main/prometheus/blackbox_icmp.json contains ICMP endpoints

https://github.com/HubDamian95/check-global-websites-grafana-prometheus/blob/main/prometheus/prometheus.yml contains main prometheus data to poll 


Install prometheus and grafana-server locally.

```
# 1. UPDATE
sudo apt update && sudo apt upgrade -y

# 2. INSTALL PROMETHEUS
sudo apt install prometheus -y
sudo systemctl enable --now prometheus

# 3. INSTALL BLACKBOX EXPORTER
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar xvf blackbox_exporter-*.tar.gz
sudo mv blackbox_exporter-*/blackbox_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false blackbox
sudo mkdir /etc/blackbox
sudo chown blackbox:blackbox /etc/blackbox
sudo tee /etc/systemd/system/blackbox.service > /dev/null <<EOF
[Unit]
Description=Blackbox Exporter
After=network.target

[Service]
User=blackbox
Group=blackbox
Type=simple
ExecStart=/usr/local/bin/blackbox_exporter --config.file=/etc/blackbox/blackbox.yml

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now blackbox

# 4. INSTALL GRAFANA
sudo apt install -y apt-transport-https software-properties-common
wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
sudo systemctl enable --now grafana-server

# 5. CHOWN FILES CORRECTLY
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo chown -R blackbox:blackbox /etc/blackbox
sudo chown -R grafana:grafana /etc/grafana /var/lib/grafana

# 6. OPEN PORTS
sudo ufw allow 9090/tcp  # Prometheus
sudo ufw allow 9115/tcp  # Blackbox
sudo ufw allow 3000/tcp  # Grafana

# 7. DONE
echo "Grafana: http://$(hostname -I | awk '{print $1}'):3000 (admin/admin)"
echo "Prometheus: http://$(hostname -I | awk '{print $1}'):9090"
```

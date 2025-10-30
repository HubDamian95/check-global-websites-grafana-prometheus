# Global Website Latency Map  
**Prometheus + Blackbox + Grafana = Live World Observability**

<img width="2546" height="1387" alt="image" src="https://github.com/user-attachments/assets/f15a5d1d-54f7-45ae-84d4-414b72ff19f5" />


> **25+ global targets** | **ICMP + HTTP** | **Live latency heatmap** | **Dark CartoDB map**

---

## Files

| File | Link |
|------|------|
| **Grafana Dashboard** | [`dashboard.json`](https://github.com/HubDamian95/check-global-websites-grafana-prometheus/blob/main/grafana_dashboard/dashboard.json) |
| **HTTP Targets** | [`blackbox_http.json`](https://github.com/HubDamian95/check-global-websites-grafana-prometheus/blob/main/prometheus/blackbox_http.json) |
| **ICMP Targets** | [`blackbox_icmp.json`](https://github.com/HubDamian95/check-global-websites-grafana-prometheus/blob/main/prometheus/blackbox_icmp.json) |
| **Prometheus Config** | [`prometheus.yml`](https://github.com/HubDamian95/check-global-websites-grafana-prometheus/blob/main/prometheus/prometheus.yml) |

---

## Install (Ubuntu/Debian) — <10 mins

```bash
# 1. UPDATE
sudo apt update && sudo apt upgrade -y

# 2. PROMETHEUS
sudo apt install prometheus -y
sudo systemctl enable --now prometheus

# 3. BLACKBOX EXPORTER
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar xvf blackbox_exporter-*.tar.gz
sudo mv blackbox_exporter-*/blackbox_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false blackbox
sudo mkdir /etc/blackbox
sudo tee /etc/blackbox/blackbox.yml > /dev/null <<EOF
modules:
  http_2xx:
    prober: http
    http:
      preferred_ip_protocol: "ip4"
  icmp:
    prober: icmp
EOF
sudo chown blackbox:blackbox /etc/blackbox/blackbox.yml
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

# 4. GRAFANA
sudo apt install -y apt-transport-https software-properties-common wget
wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
sudo systemctl enable --now grafana-server

# 5. CHOWN (CRITICAL)
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo chown -R blackbox:blackbox /etc/blackbox
sudo chown -R grafana:grafana /etc/grafana /var/lib/grafana

# 6. FIREWALL
sudo ufw allow 9090/tcp  # Prometheus
sudo ufw allow 9115/tcp  # Blackbox
sudo ufw allow 3000/tcp  # Grafana

# 7. DONE
IP=$(hostname -I | awk '{print $1}')
echo "Grafana: http://$IP:3000 (admin/admin) by default"
echo "Prometheus: http://$IP:9090"
echo "Blackbox: http://$IP:9115"

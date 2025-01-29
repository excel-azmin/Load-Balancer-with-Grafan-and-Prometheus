# Grafana Monitoring Setup for CarePro-LB

## Overview
This guide provides step-by-step instructions to set up **Grafana** on **CarePro-LB** to monitor your **Windows servers (CarePro-WebServer-1, CarePro-WebServer-2, CarePro-DB)**, **IIS traffic**, and **send email notifications** based on performance metrics.

---
## Step 1: Prepare the Environment
### Access CarePro-LB Server
SSH into **CarePro-LB**:
```bash
ssh root@202.91.42.206
```
Enter the password: `adminetl`.

### Update the System
Ensure the system is up-to-date:
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Required Dependencies
Install necessary packages:
```bash
sudo apt install -y curl wget gnupg2 software-properties-common apt-transport-https
```

---
## Step 2: Install Grafana
### Add Grafana Repository
```bash
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

### Install Grafana
```bash
sudo apt update
sudo apt install grafana -y
```

### Start and Enable Grafana Service
```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### Verify Grafana Installation
```bash
sudo systemctl status grafana-server
```
Open **Grafana** in a browser:
```
http://202.91.42.206:3000
```
- **Username:** `admin`
- **Password:** `admin` (change when prompted)

---
## Step 3: Install Prometheus (Metrics Collector)
### Download and Install Prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.47.0.linux-amd64.tar.gz
sudo mv prometheus-2.47.0.linux-amd64 /opt/prometheus
```

### Configure Prometheus
```bash
sudo nano /opt/prometheus/prometheus.yml
```

Add the following configuration:
```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds.

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
  - job_name: 'windows_servers'
    static_configs:
      - targets: ['192.168.10.125:9182', '192.168.10.127:9182', '192.168.10.234:9182']

  - job_name: 'iis_metrics'
    static_configs:
      - targets: ['192.168.10.125:9182', '192.168.10.127:9182']
  

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

```

### Start Prometheus
```bash
sudo /opt/prometheus/prometheus --config.file=/opt/prometheus/prometheus.yml &
```

### Verify Prometheus
Visit **Prometheus Web UI**:
```
http://202.91.42.206:9090
```
Check **Targets** to ensure Windows servers are being scraped.

---
## Step 4: Install and Configure Windows Exporter
### Download & Install Windows Exporter
On **each Windows server** (**CarePro-WebServer-1, CarePro-WebServer-2, CarePro-DB**):
- Download the [Windows Exporter](https://github.com/prometheus-community/windows_exporter/releases)
- Install it using the MSI installer.

### Verify Windows Exporter
On each **Windows server**, open a browser and visit:
```
http://localhost:9182/metrics
```
Ensure metrics are being exposed.

---
## Step 5: Add Data Sources to Grafana
### Add Prometheus as a Data Source
1. Go to **Configuration → Data Sources**.
2. Click **Add data source** → Select **Prometheus**.
3. Set the URL:
   ```
   http://localhost:9090
   ```
4. Click **Save & Test**.

### Add IIS Metrics (Optional)
To monitor IIS, **enable IIS Collector** in Windows Exporter:
```powershell
"C:\Program Files\windows_exporter\windows_exporter.exe" --collectors.enabled "iis"
```

---
## Step 6: Create Dashboards in Grafana
### Import Dashboards
1. Go to **Dashboards → Manage**.
2. Click **Import** and use the following dashboard IDs:
   - **Windows Node Exporter**: `3662, 20763, 22321, 21697`
   - **IIS Metrics**: `12495`
3. Customize as needed.

### Create Alerts
1. Open the **Dashboard** in Grafana.
2. Set alerts for specific metrics (CPU usage, memory, traffic).
3. Configure email notifications.

---
## Step 7: Configure Email Notifications
### Configure SMTP in Grafana
```bash
sudo nano /etc/grafana/grafana.ini
```

Modify the `[smtp]` section:
```ini
[smtp]
enabled = true
host = smtp.your-email-provider.com:587
user = your-email@example.com
password = your-email-password
from_address = your-email@example.com
from_name = Grafana
```

### Restart Grafana
```bash
sudo systemctl restart grafana-server
```

### Test Email Notifications
1. Go to **Alerting → Notification channels**.
2. Add an **email notification channel**.
3. Send a test notification.

---
## Step 8: Monitor Traffic
### Install and Configure SNMP Exporter
If you want to monitor **network traffic**, install the **SNMP Exporter** on **CarePro-LB**:
```bash
sudo apt install snmp snmpd
```
Configure SNMP to monitor your network devices and add SNMP as a data source in Grafana.

---
## Step 9: Finalize and Test
### Verify All Metrics
1. Check all metrics (**Windows, IIS, Traffic**) in Grafana dashboards.
2. Ensure Prometheus is scraping data correctly.

### Set Up Additional Alerts
1. Define critical **CPU, Memory, Disk, IIS, and Traffic** thresholds.
2. Set **email alerts** for high utilization.

### Schedule Regular Backups
1. Export Grafana dashboards.
2. Create automated **backups of configurations**.

---
## 🎯 **Final Setup Overview**
✅ **Windows & Linux Server Health Monitoring**  
✅ **IIS Performance & Traffic Monitoring**  
✅ **Network Bandwidth & Traffic Alerts**  
✅ **Real-Time Email Notifications for Performance Issues**  
✅ **Grafana Dashboards for System Insights**  

🚀 **Your monitoring system is now ready!** 🎉


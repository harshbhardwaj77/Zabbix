# Zabbix Agent Auto-Setup & Auto-Registration Guide (GCP)

This guide explains how to **install, configure, and auto-register** a Zabbix Agent on a GCP VM **automatically** using a **startup script**.  
Auto-registration allows new agents to be **automatically added** to the Zabbix Server without manual intervention.

---

## **1️⃣ Automated Setup Using a Startup Script**
To ensure every new VM registers automatically, **use this startup script** when launching instances.

### **📌 Startup Script for Zabbix Agent Installation & Auto-Configuration**
Create a file called `zabbix-agent-setup.sh` and copy the script below:

```bash
#!/bin/bash

# Variables - Update these as per your environment
ZABBIX_SERVER_IP="65.2.181.194"  # Replace with your Zabbix Server IP
ZABBIX_METADATA="cloudgo"  # Metadata to trigger auto-registration
LOG_FILE="/var/log/zabbix_agent_setup.log"

echo "🚀 Starting Zabbix Agent Setup on $(hostname)..." | tee -a $LOG_FILE

# Ensure script runs as root
if [[ $EUID -ne 0 ]]; then
    echo "❌ This script must be run as root. Use sudo!" | tee -a $LOG_FILE
    exit 1
fi

# Update system and install required packages
echo "📦 Updating system and installing dependencies..." | tee -a $LOG_FILE
apt update && apt install -y wget curl >> $LOG_FILE 2>&1

# Download and install Zabbix repository
echo "🌍 Downloading Zabbix repository..." | tee -a $LOG_FILE
wget -q https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb -O /tmp/zabbix-release_latest.deb
dpkg -i /tmp/zabbix-release_latest.deb >> $LOG_FILE 2>&1
apt update >> $LOG_FILE 2>&1

# Install Zabbix Agent
echo "🛠 Installing Zabbix Agent..." | tee -a $LOG_FILE
apt install -y zabbix-agent2 >> $LOG_FILE 2>&1

# Configure Zabbix Agent
echo "⚙ Configuring Zabbix Agent..." | tee -a $LOG_FILE
sed -i "s|^Server=.*|Server=$ZABBIX_SERVER_IP|" /etc/zabbix/zabbix_agent2.conf
sed -i "s|^ServerActive=.*|ServerActive=$ZABBIX_SERVER_IP|" /etc/zabbix/zabbix_agent2.conf
sed -i "s|^Hostname=.*|Hostname=$(hostname)|" /etc/zabbix/zabbix_agent2.conf
sed -i "s|^# HostMetadata=.*|HostMetadata=$ZABBIX_METADATA|" /etc/zabbix/zabbix_agent2.conf

# Restart and enable Zabbix Agent
echo "🔄 Restarting Zabbix Agent..." | tee -a $LOG_FILE
systemctl restart zabbix-agent2
systemctl enable zabbix-agent2

# Verify Zabbix Agent status
echo "🔍 Checking Zabbix Agent status..." | tee -a $LOG_FILE
if systemctl is-active --quiet zabbix-agent2; then
    echo "✅ Zabbix Agent is running successfully!" | tee -a $LOG_FILE
else
    echo "❌ Zabbix Agent failed to start! Check logs in /var/log/zabbix/zabbix_agent2.log" | tee -a $LOG_FILE
    exit 1
fi

echo "🎉 Zabbix Agent Setup Completed Successfully on $(hostname)!" | tee -a $LOG_FILE

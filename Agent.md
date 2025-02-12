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

```

## 2️⃣ Using the Startup Script in GCP
### Method 1: Run Manually on a GCP VM
Save the script as zabbix-agent-setup.sh
Run the following commands:
```bash
chmod +x zabbix-agent-setup.sh
sudo ./zabbix-agent-setup.sh
```
### Method 2: Use as a Startup Script When Launching a VM
Go to Google Cloud Console → Compute Engine → VM Instances
Click "Create Instance"
Scroll down to "Advanced options" → "Automation" → "Startup script"
Paste the entire script into the box
Click "Create"
🚀 The VM will install and configure the Zabbix Agent automatically on first boot!

## 3️⃣ Configure Auto-Registration in Zabbix Server
### Step 1: Enable Auto-Registration in Zabbix UI
Log in to Zabbix Web UI:
```bash
http://<ZABBIX_SERVER_IP>/zabbix
```
Navigate to:
```bash
Alerts → Actions → Auto-Registration
```
Click "Create Action".
### Step 2: Set Auto-Registration Conditions
Action Name: Auto Register GCP VMs
Condition Type: Host metadata
Operator: Contains
Value: cloudgo
(This must match ZABBIX_METADATA in the script)
### Step 3: Define Auto-Registration Actions
Under the Operations tab:

Click "New" to add an operation.
Operation Type: Add Host
Assign to a Host Group:
Example: GCP Instances (Create a new group if needed)
Assign Templates:
Select Template OS Linux (or any other relevant template)
✅ Click "Update" to Save Changes!

## 4️⃣ Verify Auto-Registration
### Step 1: Wait for a Few Minutes
Auto-registration may take some time depending on the agent's update interval.
### Step 2: Check in Zabbix Web UI
Go to **Data collection → Hosts**.
Look for the newly registered GCP VM.
Ensure it is in the correct Host Group and has the correct Template.
If the host does not appear, check the Auto-registration Logs in:
```bash
Alerts → Actions → Auto-Registration
```
## 5️⃣ Troubleshooting Auto-Registration Issues
### 1️⃣ Check Zabbix Agent Logs on the Monitored VM
```bash
sudo tail -f /var/log/zabbix/zabbix_agent2.log
```
Look for errors like "cannot connect to server".
Ensure HostMetadata matches exactly with what is set in Zabbix UI.
### 2️⃣ Check Zabbix Server Logs
```bash
sudo tail -f /var/log/zabbix/zabbix_server.log
```
Look for auto-registration processing logs.
### 3️⃣ Test Manual Registration
Run this on the Monitored VM to check if it can send data to Zabbix:
```bash
zabbix_sender -z 65.2.181.194 -s "$(hostname)" -k system.cpu.load[percpu,avg1] -o 0.5
```
✅ Expected Output: sent: 1; failed: 0;

## ✅ Final Summary
✔ Zabbix Agent Installed & Configured Automatically on GCP /
✔ Auto-Registration Enabled in Zabbix Server /
✔ Agent Restarts & Monitored VM Auto-Added to Zabbix /
✔ Checked Logs & Troubleshooting Steps for Issues

🚀 Now, every new GCP VM will auto-register and start monitoring itself in Zabbix automatically! 🎉

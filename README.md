# Zabbix Installation Guide

Zabbix is a powerful open-source monitoring platform that provides enterprise-level features to monitor servers, applications, and services. It allows proactive monitoring with alerting capabilities for various IT infrastructure components.

---

## **Pre-Requisites**

### **1. Machine Requirements**
- **Minimum Machine Type**: `t2.medium` (or equivalent) with 2 vCPUs and 4GB RAM.
- **Recommended Machine Type**: 2 vCPUs and 8GB RAM for better performance.
- **Operating System**: Ubuntu 20.04, 22.04, or 24.04.

### **2. Open Required Ports**
- **Zabbix Agent (Passive Mode)**: Port `10050` must be open on the monitored machines.
- **Zabbix Server**: Port `10051` must be open for agent communication.

---

## **Step 1: Install Apache, PHP, and MySQL**

### **1️⃣ Install Apache**
Run the following commands to install Apache:
```bash
sudo apt update
sudo apt install -y apache2
sudo systemctl start apache2
sudo systemctl enable apache2
```
### **2️⃣ Install PHP**
Install PHP along with required modules for Zabbix:
```bash
sudo apt install -y php php-cli php-mbstring php-bcmath php-xml php-mysql libapache2-mod-php
sudo systemctl restart apache2
```
### **3️⃣ Install MySQL**
Install MySQL Server and secure the installation:
```bash
sudo apt install -y mysql-server
sudo mysql_secure_installation
```
## **Step 2: Install and Configure Zabbix**
### **1️⃣ Become Root User**
Start a new shell session with root privileges:
```bash
sudo -s
```

### **2️⃣ Install Zabbix Repository**
Run the following commands to install the latest Zabbix 7.0 repository:
```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
apt update
```

### **3️⃣ Install Zabbix Server, Frontend, and Agent**
```bash
apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

### **4️⃣ Create Initial Database**
Ensure that your MySQL server is running, then run the following commands:
```bash
mysql -uroot -p
```

Inside MySQL:
```bash
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'YourStrongPassword123!';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
EXIT;
```

### **5️⃣ Import Zabbix Database Schema**
```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

After importing, disable log_bin_trust_function_creators:
```bash
mysql -uroot -p
```

Inside MySQL:
```bash
SET GLOBAL log_bin_trust_function_creators = 0;
EXIT;
```

### **6️⃣ Configure the Database for Zabbix Server**
Edit the Zabbix Server configuration file:
```bash
sudo vim /etc/zabbix/zabbix_server.conf

Find the line:
```bash
# DBPassword=
```
Replace it with:
```bash
DBPassword=YourStrongPassword123!
```
Save and exit (CTRL+X, then Y, then Enter).

### **7️⃣ Start Zabbix Server and Agent Processes**
Run the following commands:
```bash
systemctl restart zabbix-server
systemctl restart zabbix-agent
systemctl restart apache2
systemctl enable zabbix-server
systemctl enable zabbix-agent
systemctl enable apache2
```

### **8️⃣ Open Zabbix UI**
Once the services are running, open Zabbix Web UI in a browser:
```bash
http://<ZABBIX_SERVER_IP>/zabbix
```

## **Step 3: Configure Zabbix**
### **1️⃣ Connect to the Zabbix Web UI**
Open your browser and navigate to:
```bash
http://<ZABBIX_SERVER_IP>/zabbix
Login with the default credentials:
Username: Admin
Password: zabbix
```
Change the password after first login.


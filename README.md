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
sudo systemctl enable apache2    ```

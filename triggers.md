# Configuring High CPU & Memory Usage Triggers in Zabbix (GCP VMs)

## Overview
When monitoring multiple VMs (50+ or more) in **Zabbix**, manually setting up triggers for each VM is inefficient. Instead, we use **templates** to apply triggers automatically across all monitored VMs. This guide explains how to configure **high CPU & memory usage triggers** using the **"Linux by Zabbix agent"** template and how to set up **email alerts**.

---

## 1️⃣ Applying the "Linux by Zabbix agent" Template to All VMs
To monitor Linux VMs, we use the **"Linux by Zabbix agent"** template, which replaces the older **"Template OS Linux"** in newer Zabbix versions.

### Steps to Assign the Template to All VMs
1. **Go to** `Data collection → Hosts`
2. **Click on a monitored VM**
3. **Navigate to the "Templates" tab**
4. **Click "Select"**, then choose **"Linux by Zabbix agent"**
5. **Click "Update"**
6. Repeat for all VMs or use the **bulk update option** to apply to multiple VMs at once.

✅ Now, all assigned VMs will start collecting **CPU, Memory, and Disk usage metrics** automatically!

---

## 2️⃣ Creating Triggers for High CPU & Memory Usage
Once the template is applied to all VMs, we create **triggers** inside the template. This ensures that **all VMs using the template will inherit the same trigger conditions automatically**.

### **Steps to Create Triggers for High CPU Usage**
1. **Go to** `Data collection → Templates`
2. **Find and select** "Linux by Zabbix agent"
3. **Click on "Triggers" → "Create trigger"**
4. **Enter the Trigger Details:**
   - **Name:** `High CPU Usage Alert`
   - **Severity:** `High`
   - **Expression:**
     ```ini
     last(/Linux by Zabbix agent/system.cpu.util,#2)>80
     ```
     **Explanation:**
     - This checks if the **CPU usage of the last two collected values** exceeds **80%**.
     - If the condition is met, an **alert is triggered**.
5. **Click "Add" → Apply**

✅ Now, all VMs using this template will automatically trigger an alert when CPU usage goes above **80%**.

---

### **Steps to Create Triggers for High Memory Usage**
1. **Go to** `Data collection → Templates`
2. **Find and select** "Linux by Zabbix agent"
3. **Click on "Triggers" → "Create trigger"**
4. **Enter the Trigger Details:**
   - **Name:** `High Memory Usage Alert`
   - **Severity:** `High`
   - **Expression:**
     ```ini
     last(/Linux by Zabbix agent/vm.memory.utilization,#2)>90
     ```
     **Explanation:**
     - This checks if **RAM utilization of the last two collected values** exceeds **90%**.
     - If the condition is met, an **alert is triggered**.
5. **Click "Add" → Apply**

✅ Now, memory alerts will automatically trigger when usage goes above **90%**.

---

## 3️⃣ Setting Up Alerts Based on These Triggers
Once the triggers are set up, you need to configure **alert notifications** so that the right people are notified when CPU or memory usage is high.

### **Steps to Configure Alerting (Email Notification Example)**
1. **Go to Zabbix Web UI** → `Alerts → Actions`
2. Click **"Create Action"**
3. **Enter Action Details:**
   - **Name:** `Notify Admins for High CPU/Memory`
   - **Event Source:** `Trigger`
   - **Condition:**  
     - **Trigger severity** → `>= High`
     - **Host group** → `GCP Instances`
4. **Under Operations Tab:**
   - Click **"New"** to add an operation.
   - **Operation Type:** `Send Message`
   - **Send To:** `Zabbix Administrators`
   - **Use Media Type:** `Email` (or configure Slack, Telegram, etc.)
   - **Message:**  
     ```ini
     High CPU/Memory Alert on {HOST.NAME}
     Issue: {TRIGGER.NAME}
     Severity: {TRIGGER.SEVERITY}
     Current Value: {ITEM.VALUE1}%
     ```
5. **Click "Add" → "Update"**

✅ Now, you will receive **email alerts** whenever CPU or Memory usage is high!

---

## 4️⃣ Setting Up Media Type (Email) for Notifications
Before Zabbix can send email alerts, you need to **configure an email media type and assign it to users**.

### **Steps to Configure Email Media Type**
1. **Go to** `Alerts → Media types`
2. Click **"Create Media Type"**
3. **Set the following details:**
   - **Name:** `Email`
   - **Type:** `Email`
   - **SMTP Server:** `smtp.gmail.com`
   - **SMTP Port:** `587` (TLS) or `465` (SSL)
   - **SMTP Authentication:** Enabled
   - **SMTP Username:** `your-email@gmail.com`
   - **SMTP Password:** (Use **Google App Password**, not your normal password)
   - **From email:** `your-email@gmail.com`
   - **Connection Security:** `STARTTLS` (for `587`) or `SSL/TLS` (for `465`)
4. **Click "Update"**

✅ Now, Zabbix can send emails using this media type.

---

### **Assign Email Media to Users**
1. **Go to** `Administration → Users`
2. **Select the user receiving alerts (e.g., Admin)**
3. **Go to "Media" tab → Click "Add"**
4. **Enter the details:**
   - **Type:** `Email`
   - **Send to:** `your-email@gmail.com`
   - **Severity:** Check all (`Not classified, Information, Warning, High, Disaster`)
   - **Status:** Enabled
5. **Click "Update"**

✅ Now, the user can receive email alerts!

---

## ✅ Final Summary
✔ **Use "Linux by Zabbix agent" as the template** for Linux VMs.
✔ **Apply the template to all monitored VMs** for automated monitoring.
✔ **Define triggers inside the template**, so they apply to **all VMs automatically**.
✔ **Set up CPU & memory triggers** using:
   ```ini
   last(/Linux by Zabbix agent/system.cpu.util,#2)>80  # CPU Trigger
   last(/Linux by Zabbix agent/vm.memory.utilization,#2)>90  # Memory Trigger
   ```
✔ **Configure Alerts** under `Alerts → Actions` to send notifications for high resource usage.
✔ **Set up Media Type (Email)** under `Alerts → Media types` and **assign it to users**.

🚀 Now, Zabbix will **automatically alert you** when CPU or Memory usage is high for any VM without needing manual configuration! 🎉



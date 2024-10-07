# **EKS Cluster Log Collection, Storage, and Error Notification via Email**

This document explains how to gather logs from pods running on worker nodes in an Amazon EKS cluster, store them in files, and automatically send an email notification if specific keywords (e.g., "localhost," "500," "400," or query-related errors) are found in the logs.

## **Table of Contents**

1. [Cluster Setup and Log Sources](#cluster-setup-and-log-sources)
2. [Python Script for Log Collection and Error Notification](#python-script-for-log-collection-and-error-notification)
3. [Setting Up the Cron Job](#setting-up-the-cron-job)
4. [Conclusion](#conclusion)
5. [Command Summary](#command-summary)

---

## **1. Cluster Setup and Log Sources**

### **1.1 Master and Worker Nodes:**

- **Master Node:** The master node orchestrates the cluster.
- **Worker Nodes:** Each worker node runs multiple pods, depending on its capacity.
- **Pod Logs:** Every pod generates logs that need to be collected every 10 minutes.

---

## **2. Python Script for Log Collection and Error Notification**

Python script collects logs from each pod every 10 minutes, stores the logs in a file named after the pod (e.g., `podname.txt`), and checks the log content for specific keywords. If any log contains the keywords "localhost," "500," "400," or any query-related errors, the script sends an email notification.

### **2.1 Working Python Script:**

---

## **3. Setting Up the Cron Job**

### **3.1 Installing Cron:**

Ensure that cron is installed and running on the master node:

```bash
sudo apt-get update
sudo apt-get install cron
sudo systemctl enable cron
sudo systemctl start cron
```

### **3.2 Adding the Python Script to Cron:**

Create a cron job to run the Python script every 10 minutes.

1. Open the crontab editor:

```bash
crontab -e
```

2. Add the following entry to run the script every 10 minutes:

```bash
*/10 * * * * /usr/bin/python3 /path/to/your/python_script.py >> /var/log/cron_log.log 2>&1
```

3. Save and exit the editor.

### **3.3 Verify Cron Job:**

To verify that the cron job is running, you can check the cron log:

```bash
tail -f /var/log/cron_log.log
```

---

## **4. Conclusion**

This approach collects logs from worker nodes' pods every 10 minutes, stores them in files, and sends an email notification if specific keywords indicating errors are found in the logs. By using a cron job, the process is automated and runs at regular intervals.

---

## **5. Command Summary:**

### **Install cron:**

```bash
sudo apt-get update && sudo apt-get install cron
sudo systemctl enable cron
sudo systemctl start cron
```

### **Set up a cron job to run the Python script every 10 minutes:**

```bash
crontab -e
*/10 * * * * /usr/bin/python3 /path/to/your/python_script.py >> /var/log/cron_log.log 2>&1
```

---

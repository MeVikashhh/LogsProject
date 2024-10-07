---
# **EKS Cluster Log Collection, CloudWatch, and Error Notification via AWS SNS**

This document explains how to collect logs from pods running on worker nodes in an Amazon EKS cluster, push those logs to Amazon CloudWatch, and automatically send an SMS notification via AWS SNS if specific keywords (e.g., "localhost," "500," "400," or query-related errors) are found in the logs. AWS Lambda will be used to process the logs.

## **Table of Contents**

1. [Cluster Setup and Log Sources](#cluster-setup-and-log-sources)
2. [Python Script for Log Collection and Error Notification Using Lambda]
3. [Setting Up Lambda Function](#setting-up-lambda-function)
4. [CloudWatch and SNS Setup](#cloudwatch-and-sns-setup)
5. [Cost Estimate](#cost-estimate)
6. [Conclusion](#conclusion)
7. [Command Summary](#command-summary)

---

## **1. Cluster Setup and Log Sources**

### **1.1 Master and Worker Nodes:**

- **Master Node:** Orchestrates the EKS cluster.
- **Worker Nodes:** Each worker node runs multiple pods, depending on its capacity.
- **Pod Logs:** Each pod generates logs which are collected for analysis.

---

## **2. Python Script for Log Collection and Error Notification Using Lambda**

The following Python script collects logs from each pod, pushes those logs to Amazon CloudWatch, and checks for specific keywords. If the logs contain keywords like "localhost," "500," "400," or query-related errors, an SMS notification is sent using AWS SNS.

### **2.1 Working on Python Script for Lambda:**


## **3. Setting Up Lambda Function**

### **3.1 Creating a Lambda Function:**

1. **Navigate to AWS Lambda Console:**

   Go to the [AWS Lambda Console](https://console.aws.amazon.com/lambda).

2. **Create a New Lambda Function:**

   - Choose "Author from scratch."
   - Name the function (e.g., `EKSLogMonitor`).
   - Set runtime to `Python 3.x`.
   - Create or assign an existing IAM role with permissions for CloudWatch, EKS, and SNS.

3. **Upload the Python Script:**

   - Either write the script directly in the inline editor or upload it as a `.zip` file.
   - Set the handler to `lambda_function.lambda_handler`.

4. **Set Environment Variables:**

   Configure the following environment variables:
   
   - `SNS_TOPIC_ARN`: The ARN of your SNS topic for alerts.
   - `LOG_GROUP_NAME`: The CloudWatch Log Group name.
   - `LOG_STREAM_NAME`: The CloudWatch Log Stream name.

---

## **4. CloudWatch and SNS Setup**

### **4.1 CloudWatch Log Group Setup:**

1. **Create a Log Group:**

```bash
aws logs create-log-group --log-group-name /eks/worker-node-logs
```

2. **Create a Log Stream:**

```bash
aws logs create-log-stream --log-group-name /eks/worker-node-logs --log-stream-name worker-node-log-stream
```

### **4.2 Create an SNS Topic for Alerts:**

1. **Create an SNS Topic:**

```bash
aws sns create-topic --name critical-log-alerts
```

2. **Subscribe a Phone Number:**

```bash
aws sns subscribe --topic-arn arn:aws:sns:your-region:your-account-id:critical-log-alerts --protocol sms --notification-endpoint +1234567890
```

3. **Get the SNS Topic ARN:**

```bash
aws sns list-topics
```

Use the SNS Topic ARN in your Lambda script to send alerts.

---

## **5. Cost Estimate**

### **5.1 AWS Lambda Costs:**

- **Free Tier:** The first 1 million requests per month are free. After that, you pay $0.20 per 1 million requests.
- **Duration:** Charges are based on the duration your code runs, measured in milliseconds. The first 400,000 GB-seconds per month are free.

### **5.2 Amazon CloudWatch Costs:**

- **Log Ingestion:** $0.50 per GB ingested.
- **Log Data Storage:** $0.03 per GB per month.
- **Log Data Retrieval:** $0.01 per 1,000 requests.

### **5.3 Amazon SNS Costs:**

- **SMS Pricing:** Varies by destination. In the U.S., it's approximately $0.0075 per SMS message sent.

### **5.4 Example Monthly Costs:**

- **Lambda:** If your Lambda function runs 100,000 times for 1 second each month, the cost would be around $0.20 for requests.
- **CloudWatch:** If you ingest 10 GB of logs and store them for a month, the cost would be about $5.30 ($5.00 for ingestion + $0.30 for storage).
- **SNS:** If you send 100 SMS alerts, the cost would be about $0.75.

### **5.5 Total Estimated Cost:**  
Total monthly costs could be approximately $6.25 for the described usage scenario.

---

## **6. Conclusion**

This setup collects logs from worker nodes' pods, pushes them to CloudWatch, and sends an SMS alert via AWS SNS if specific errors are detected. AWS Lambda automates the entire process, ensuring efficient log monitoring and error notification.

---

## **7. Command Summary:**

### **CloudWatch Setup:**

```bash
aws logs create-log-group --log-group-name /eks/worker-node-logs
aws logs create-log-stream --log-group-name /eks/worker-node-logs --log-stream-name worker-node-log-stream
```

### **SNS Setup:**

```bash
aws sns create-topic --name critical-log-alerts
aws sns subscribe --topic-arn arn:aws:sns:your-region:your-account-id:critical-log-alerts --protocol sms --notification-endpoint +1234567890
```

### **Set up Lambda:**

- Create the Lambda function.
- Upload the Python script.
- Set environment variables (`SNS_TOPIC_ARN`, `LOG_GROUP_NAME`, `LOG_STREAM_NAME`).

--- 

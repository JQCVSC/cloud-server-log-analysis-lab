---

# Cloud Server Log Analysis Cybersecurity Lab

## Description

In this lab, you will learn how to analyze web server logs on a Google Cloud Platform (GCP) virtual machine using the Google Cloud SDK Shell from your local Windows computer. You will gain hands-on experience in setting up a cloud environment, accessing server logs remotely, and using both Windows and Linux commands to identify potential security threats such as SQL injections and brute force attacks. This lab is designed for cybersecurity students and beginners interested in learning practical log analysis techniques in a cloud environment.

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Lab Steps](#lab-steps)
- [Lab Challenge](#lab-challenge)
- [Conclusion](#conclusion)
- [Next Steps](#next-steps)

---

## Introduction

Welcome to the Cloud Server Log Analysis Lab! In this lab, you will simulate a real-world scenario where you are a security analyst tasked with identifying potential security threats in web server logs. You will use the Google Cloud SDK Shell on your local Windows machine to connect to a Google Cloud Platform virtual machine and analyze these logs. This lab provides hands-on experience with cloud technologies and basic log analysis techniques.

---

## Prerequisites

- A Windows computer with internet access
- Google Cloud Platform account (free tier is sufficient)
- Basic familiarity with command-line interfaces

---

## Setup Instructions

1. **Create a Google Cloud Platform (GCP) Account:**

   If you haven't already, create a free account on Google Cloud Platform to access free credits for new users.

2. **Install Google Cloud SDK:**

   - Download and install the Google Cloud SDK for Windows from: https://cloud.google.com/sdk/docs/install
   - Follow the installation prompts, and ensure you select the option to install the Google Cloud CLI.

3. **Create a New Project:**

   - Open the Google Cloud Console in your web browser.
   - Create a new project for this lab.

4. **Enable Compute Engine API:**

   - In your new project, enable the Compute Engine API to create virtual machines.

---

## Lab Steps

### Step 1: Creating a Virtual Machine

1. Open the Google Cloud Console in your web browser.
2. Navigate to Compute Engine > VM instances.
3. Click "Create Instance".
4. Configure your instance:
   - Name: log-analysis-vm
   - Region: Choose a region close to you
   - Machine type: e2-micro (2 vCPU, 1 GB memory)
   - Boot disk: Ubuntu 20.04 LTS
   - Firewall: Allow HTTP and HTTPS traffic
5. Click "Create" to launch your VM.

### Step 2: Connecting to Your VM

1. Open the Google Cloud SDK Shell on your Windows computer.
2. Authenticate with your Google Cloud account:
   ```
   gcloud auth login
   ```
3. Set your project ID:
   ```
   gcloud config set project YOUR_PROJECT_ID
   ```
   Replace YOUR_PROJECT_ID with your actual project ID.
4. Connect to your VM using SSH:
   ```
   gcloud compute ssh log-analysis-vm --zone=YOUR_VM_ZONE
   ```
   Replace YOUR_VM_ZONE with the zone where your VM is located.

### Step 3: Setting Up the Web Server Log

1. In the SSH session, create a sample log file:
   ```
   sudo nano /var/log/apache2/access.log
   ```
2. Copy and paste the following log entries:
   ```
   192.168.1.1 - - [01/Jul/2023:10:00:01 +0000] "GET /login.php HTTP/1.1" 200 1234
   192.168.1.2 - - [01/Jul/2023:10:00:02 +0000] "GET /products.php?id=1 UNION SELECT * FROM users HTTP/1.1" 400 567
   192.168.1.1 - - [01/Jul/2023:10:00:03 +0000] "POST /login.php HTTP/1.1" 401 789
   192.168.1.3 - - [01/Jul/2023:10:00:04 +0000] "GET /admin.php HTTP/1.1" 403 321
   192.168.1.2 - - [01/Jul/2023:10:00:05 +0000] "GET /products.php?id=2 OR 1=1 HTTP/1.1" 200 987
   192.168.1.1 - - [01/Jul/2023:10:00:06 +0000] "POST /login.php HTTP/1.1" 401 789
   192.168.1.4 - - [01/Jul/2023:10:00:07 +0000] "GET /wp-admin.php?user=admin&password=admin HTTP/1.1" 404 452
   192.168.1.5 - - [01/Jul/2023:10:00:08 +0000] "POST /upload.php HTTP/1.1" 500 1337
   192.168.1.6 - - [01/Jul/2023:10:00:09 +0000] "GET /login.php?username=admin&password=' OR '1'='1'-- HTTP/1.1" 200 1432
   192.168.1.7 - - [01/Jul/2023:10:00:10 +0000] "GET /setup.php?cmd=cat /etc/passwd HTTP/1.1" 400 678
   ```
3. Save the file (Ctrl+X, then Y, then Enter).

### Step 4: Basic Log Analysis

1. View the entire log file:
   ```
   sudo cat /var/log/apache2/access.log
   ```
2. Count the number of entries in the log:
   ```
   sudo wc -l /var/log/apache2/access.log
   ```
3. Search for POST requests:
   ```
   sudo grep POST /var/log/apache2/access.log
   ```
4. Find unique IP addresses:
   ```
   sudo awk '{print $1}' /var/log/apache2/access.log | sort | uniq
   ```

### Step 5: Identifying SQL Injection Attempts

1. Search for common SQL injection keywords:
   ```
   sudo grep -iE "UNION|SELECT|INSERT|UPDATE|DELETE|DROP|FROM|WHERE" /var/log/apache2/access.log
   ```

### Step 6: Detecting Brute Force Attempts

1. Look for failed login attempts:
   ```
   sudo grep -iE "login|authenticate|auth" /var/log/apache2/access.log | grep "401"
   ```

### Step 7: Analyzing HTTP Status Codes

1. Count occurrences of each HTTP status code:
   ```
   sudo awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -rn
   ```

### Step 8: Disconnecting from the VM

1. When you're done with your analysis, type `exit` to close the SSH connection and return to the Google Cloud SDK Shell on your local machine.

---

## Lab Challenge

### Scenario

You are a security analyst for a small e-commerce company. The company's web server has been experiencing unusual traffic, and you've been asked to investigate the logs for any potential security threats.

### Tasks

1. Analyze the provided log file for:
   - SQL injection attempts
   - Brute force login attempts
   - Unusual HTTP status codes
   - Attempts to access sensitive files or directories
2. Identify the IP addresses associated with suspicious activities
3. Determine the most frequently accessed resources
4. Create a brief report summarizing your findings and recommendations

### IP Scenarios

- 192.168.1.1: A legitimate user who occasionally misplaces their password
- 192.168.1.2: A malicious actor attempting SQL injection
- 192.168.1.3: An employee trying to access restricted areas
- 192.168.1.4: A bot scanning for common CMS admin pages
- 192.168.1.5: A user encountering a server error during file upload
- 192.168.1.6: Another malicious actor attempting SQL injection via login
- 192.168.1.7: A potential attacker trying to exploit a misconfiguration

### Skills Practiced

- **Remote Server Access:** Using Google Cloud SDK Shell to connect to a cloud-based VM
- **Log Analysis:** Manually parsing and analyzing log files to identify potential security threats
- **Linux Commands:** Using basic Linux commands for text processing and analysis
- **Pattern Recognition:** Identifying suspicious patterns in log entries
- **Cloud Computing:** Working with virtual machines in a cloud environment

---

## Conclusion

Congratulations on completing the Cloud Server Log Analysis Lab! You've gained hands-on experience in analyzing web server logs for security threats using a Google Cloud Platform virtual machine, accessed remotely from your local Windows machine. This lab has provided you with practical skills in log analysis and familiarized you with cloud technologies in a cybersecurity context.

---

## Next Steps

- Explore more advanced log analysis techniques using tools like awk, sed, and regular expressions
- Learn about log aggregation and analysis tools such as ELK stack (Elasticsearch, Logstash, Kibana)
- Investigate automated log analysis solutions and how they can be integrated into cloud environments
- Study common web application attack patterns and how they manifest in server logs
- Practice creating custom scripts to automate log analysis tasks

---

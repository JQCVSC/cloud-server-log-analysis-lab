---

# Cloud Web Server Log Analysis Cybersecurity Lab

## Description

In this lab, you will use Python to analyze web server logs stored in Google Cloud Storage. You will learn how to write scripts to identify potential security threats, such as SQL injections and brute force attacks. This hands-on project demonstrates the use of Python for log analysis, Google Cloud Functions for automation, and Google Cloud Storage for data management. It's ideal for security analysts and students interested in learning how to use Python for cybersecurity log analysis in a cloud environment.

## Table of Contents

- [Introduction](#introduction)
- [Setup Instructions](#setup-instructions)
- [Step 1: Creating Sample Logs](#step-1-creating-sample-logs)
- [Step 2: Setting Up Google Cloud Environment](#step-2-setting-up-google-cloud-environment)
- [Step 3: Writing the Python Log Analyzer Script](#step-3-writing-the-python-log-analyzer-script)
- [Step 4: Deploying the Script as a Cloud Function](#step-4-deploying-the-script-as-a-cloud-function)
- [Step 5: Testing Your Cloud Function](#step-5-testing-your-cloud-function)
- [Lab Challenge](#lab-challenge)
- [Conclusion](#conclusion)
- [Next Steps](#next-steps)

---

## Introduction

Welcome to the Web Server Log Analysis Lab! In this lab, you will simulate a real-world scenario where you are a security analyst tasked with identifying potential security threats in web server logs. You will use Python to parse and analyze these logs and Google Cloud Functions to automate the process. This lab provides hands-on experience with using Python for cybersecurity tasks and demonstrates how to leverage cloud technologies to enhance your workflow.

---

## Setup Instructions

### Step 1: Creating Sample Logs

1. **Create a Sample Log File:**

   Start by creating a sample log file (`web_server.log`) with some suspicious entries to analyze. Here’s an example of what the log entries might look like:

   ```plaintext
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

2. **Save the Log File:**

   Save this file as `web_server.log` on your local machine.

### Step 2: Setting Up Google Cloud Environment

1. **Create a Google Cloud Platform (GCP) Account:**

   If you haven't already, create a free account on [Google Cloud Platform](https://cloud.google.com/) to access free credits for new users.

2. **Create Google Cloud Storage Buckets:**

   - **Log Storage Bucket:** Create a bucket named `your-log-bucket` for storing log files.
   - **Report Storage Bucket:** Create another bucket named `your-report-bucket` for storing generated reports.

3. **Upload the Sample Log File:**

   Upload your `web_server.log` file to the `your-log-bucket` bucket in Google Cloud Storage.

### Step 3: Writing the Python Log Analyzer Script

1. **Create Your Python Script:**

   Use the following Python script to parse the log files, detect potential security threats, and generate a report:

   ```python
   import re
   from google.cloud import storage
   from collections import Counter
   import json

   def download_blob(bucket_name, source_blob_name, destination_file_name):
       storage_client = storage.Client()
       bucket = storage_client.bucket(bucket_name)
       blob = bucket.blob(source_blob_name)
       blob.download_to_filename(destination_file_name)

   def analyze_logs(data, context):
       # Download the log file from Cloud Storage
       bucket_name = "your-log-bucket"
       source_blob_name = "web_server.log"
       destination_file_name = "/tmp/web_server.log"
       download_blob(bucket_name, source_blob_name, destination_file_name)



       # Patterns to look for
       sql_injection_pattern = r"(?i)(UNION|SELECT|INSERT|UPDATE|DELETE|DROP|FROM|WHERE)"
       brute_force_pattern = r"(?i)(login|authenticate|auth).*failed"
       
       sql_injection_attempts = []
       brute_force_attempts = []
       ip_counts = Counter()

       with open(destination_file_name, 'r') as file:
           for line in file:
               # Extract IP address (assuming it's the first element in each log line)
               ip = line.split()[0]
               ip_counts[ip] += 1

               # Check for SQL injection attempts
               if re.search(sql_injection_pattern, line):
                   sql_injection_attempts.append(line.strip())

               # Check for brute force attempts
               if re.search(brute_force_pattern, line):
                   brute_force_attempts.append(line.strip())

       # Generate report
       report = {
           "sql_injection_attempts": sql_injection_attempts,
           "brute_force_attempts": brute_force_attempts,
           "top_10_ips": ip_counts.most_common(10)
       }

       # Save report to Cloud Storage
       storage_client = storage.Client()
       report_bucket = storage_client.bucket("your-report-bucket")
       report_blob = report_bucket.blob("security_report.json")
       report_blob.upload_from_string(json.dumps(report, indent=2))

       print(f"Analysis complete. Report saved to your-report-bucket/security_report.json")

   # For local testing
   if __name__ == "__main__":
       analyze_logs(None, None)
   ```

2. **Save the Python Script:**

   Save the script as `log_analyzer.py`.

### Step 4: Deploying the Script as a Cloud Function

1. **Create a New Google Cloud Function:**

   - Go to **Cloud Functions** in the Google Cloud Console.
   - Click **Create Function**.
   - Set the runtime to Python 3.9 or higher.
   - Name the function `analyze_logs`.
   - Set the trigger type to **Cloud Storage** and select the `your-log-bucket`.

2. **Upload the Python Script:**

   - Copy and paste the contents of `log_analyzer.py` into the inline editor.

3. **Deploy the Function:**

   - Click **Deploy** to deploy your cloud function.

### Step 5: Testing Your Cloud Function

1. **Upload a Log File:**

   Upload a log file to your `your-log-bucket` in Google Cloud Storage to trigger the function.

2. **Check the Generated Report:**

   Once the function has executed, check `your-report-bucket` for a new `security_report.json` file that contains the analysis results.

---

## Lab Challenge

### Scenario

As a security analyst, you are tasked with identifying suspicious activities in the company's web server logs. Your Python script should be able to detect SQL injection attempts and brute force login attempts, generating a report with the findings.

### Tasks

1. Set up the Google Cloud environment as described above.
2. Implement the Python script to analyze logs for SQL injection and brute force attempts.
3. Deploy the script as a Cloud Function.
4. Test the function by uploading sample log files with different patterns of activity.
5. Verify that the function generates accurate reports in the report bucket.

### Skills Practiced

- **Log Analysis:** Parsing and analyzing log files to identify potential security threats.
- **Pattern Matching:** Using regular expressions to detect suspicious activities.
- **Cloud Security:** Working with cloud storage and serverless functions.
- **Automation:** Creating a script that can automatically analyze logs and generate reports.

---

## Conclusion

Congratulations on completing the Web Server Log Analysis Lab! You've used Python and Google Cloud to analyze web server logs for security threats. This lab provided hands-on experience with real-world cybersecurity tasks and demonstrated the power of cloud technologies in automating security processes.

---

## Next Steps

- Explore more advanced log analysis techniques, such as using machine learning to detect anomalies.
- Learn about other cloud services that can enhance your cybersecurity workflows, such as Google Cloud Pub/Sub for real-time log processing.
- Experiment with integrating your Python scripts with other cloud functions and services to build a comprehensive security monitoring system.

---

This updated README file includes a total of 10 log entries, providing a more comprehensive set of data for the lab challenge. The instructions remain aligned with the setup process for using Python and Google Cloud Functions for log analysis.

# SAA-C03 Chapter 10: Monitoring, Auditing & Observability

---

## 📖 1. The Big Picture: Visibility in the Cloud
**The Objective:** To monitor application performance in real-time, audit every single action taken in your AWS account, and ensure resources remain compliant with corporate security rules.
**The Problem:** In a traditional data center, if a server crashes or gets hacked, you have to dig through multiple physical machines to find out what happened. In the cloud, if you don't enable centralized monitoring, you will never know *why* your app is slow, *who* deleted your database, or *when* an S3 bucket was accidentally made public.
**The Solution:** We use **Amazon CloudWatch** to monitor performance metrics (like CPU and RAM) and trigger alarms. We use **AWS CloudTrail** to record every API call (who logged in and what they clicked). We use **AWS Config** to track configuration changes over time, and **AWS X-Ray** to trace bottlenecks in complex microservices.
**The Trade-offs:** Storing petabytes of log data indefinitely can become incredibly expensive. You must configure strict retention policies to balance visibility with cost.

---

## 🧠 2. Core Theory: Amazon CloudWatch (Performance)

### CloudWatch Metrics & Alarms
CloudWatch is the primary monitoring service for AWS. It collects data in the form of "Metrics" (e.g., CPUUtilization, NetworkIn).
* **Standard Metrics:** Collected every 5 minutes (Free).
* **Detailed Metrics:** Collected every 1 minute (Paid). Important for Auto Scaling groups that need to react quickly.
* **Custom Metrics:** You can push your own application metrics (e.g., "Number of items in shopping cart") to CloudWatch.
* **CloudWatch Alarms:** Used to trigger actions based on metrics. (e.g., "If CPU > 80% for 5 minutes, trigger an Auto Scaling Group to add a server, and send an SNS email to the admin").



### CloudWatch Logs
A centralized place to store logs from EC2, Lambda, ECS, and VPC Flow Logs.
* **Log Groups:** The top-level folder (e.g., `/aws/lambda/my-function`).
* **Log Streams:** Individual files inside the group (representing a specific instance or container).
* **Metric Filters:** You can search logs for specific words (like "ERROR" or "Exception") and turn those occurrences into a CloudWatch Metric that triggers an alarm.
* **Retention:** By default, logs are kept indefinitely. You must manually set an expiration (e.g., 30 days) to save money.

---

## 🕵️ 3. Core Theory: AWS CloudTrail (Auditing)

### What is CloudTrail?
CloudTrail answers the question: *"Who did what, when, and from where?"*
It records every single API call made in your AWS account, whether it was done via the AWS Management Console, the AWS CLI, or an SDK.
* **Default Behavior:** CloudTrail is enabled by default and stores 90 days of management events for free.
* **Long-term Auditing:** To keep logs for years (for compliance), you must create a "Trail" that continuously dumps logs into an **Amazon S3 bucket**.
* **CloudTrail Insights:** Uses Machine Learning to detect unusual API activity (e.g., someone suddenly creating 50 EC2 instances when your daily average is 2).



---

## 📋 4. Core Theory: AWS Config (Compliance)

### What is AWS Config?
Config tracks the *configuration history* of your AWS resources. It answers the question: *"What did this resource look like 3 days ago, and is it compliant with our rules?"*
* **Configuration Recorder:** Takes a "snapshot" every time a resource changes (e.g., a Security Group rule is added).
* **Config Rules:** You write rules (or use managed ones) like "All EBS volumes must be encrypted" or "No S3 buckets can be public".
* **Auto-Remediation:** If a resource violates a rule, Config can automatically trigger an AWS Systems Manager (SSM) document to fix it (e.g., instantly blocking the public S3 bucket).

---

## 🔬 5. Core Theory: AWS X-Ray (Tracing)

### What is AWS X-Ray?
When you have an architecture with an API Gateway calling a Lambda, which calls an SNS topic, which triggers an SQS queue, which triggers another Lambda to write to DynamoDB... if the system takes 5 seconds to load, *where is the bottleneck?*
* **Distributed Tracing:** X-Ray follows a user's request from start to finish across multiple microservices.
* **Service Map:** It automatically generates a visual map of your architecture, showing error rates and latency between components.



---

## 🏗️ 6. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Monitor the CPU and Network usage of an EC2 instance. | **Amazon CloudWatch Metrics** |
| Find out which IAM user deleted a critical production database. | **AWS CloudTrail** |
| Automatically scale up EC2 instances when CPU exceeds 75%. | **CloudWatch Alarms + Auto Scaling Group** |
| Receive an alert if the word "ERROR" appears 10 times in a log file. | **CloudWatch Logs Metric Filter + Alarm** |
| Verify if a specific Security Group had port 22 open two weeks ago. | **AWS Config** |
| Ensure all new S3 buckets are encrypted; if not, automatically fix them. | **AWS Config Rules + Auto-Remediation** |
| Find out why a complex microservice architecture is experiencing high latency. | **AWS X-Ray** |
| Retain an immutable, encrypted log of all AWS API activity for 7 years. | **CloudTrail dumping to an S3 Bucket (with Object Lock)** |

---

## 🤝 7. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is the primary difference between CloudWatch, CloudTrail, and AWS Config?
**Answer:** CloudWatch monitors performance metrics, CloudTrail logs API auditing activity, and Config tracks resource configuration and compliance.

**Question:** What is the default frequency for standard CloudWatch metrics on EC2?
**Answer:** 5 minutes.

**Question:** How do you get 1-minute metric resolution for EC2 instances?
**Answer:** By enabling Detailed Monitoring (which incurs extra costs).

**Question:** Can CloudWatch automatically monitor the amount of RAM (Memory) used inside an EC2 instance?
**Answer:** No, RAM is an OS-level metric. You must install the CloudWatch Agent on the EC2 instance to push RAM metrics.

**Question:** What is a CloudWatch Alarm?
**Answer:** A threshold set on a metric that triggers an action (like an SNS notification or an Auto Scaling event) when breached.

**Question:** What is a CloudWatch Logs Metric Filter?
**Answer:** A feature that scans incoming log data for specific keywords or patterns and turns them into a numerical CloudWatch Metric.

**Question:** How long are CloudWatch Logs retained by default?
**Answer:** Indefinitely (Never Expire). You must manually configure a retention period to save costs.

**Question:** What does AWS CloudTrail do?
**Answer:** It records all API calls made within an AWS account, providing a history of user activity and resource changes.

**Question:** How long does CloudTrail keep event history by default in the AWS Console?
**Answer:** 90 days.

**Question:** How can you store CloudTrail logs for longer than 90 days for compliance audits?
**Answer:** By creating a Trail that delivers the log files to an Amazon S3 bucket.

**Question:** How do you protect CloudTrail logs stored in S3 from being deleted or altered by hackers?
**Answer:** Enable S3 MFA Delete, use IAM strict bucket policies, or enable S3 Object Lock (WORM).

**Question:** What is CloudTrail Insights?
**Answer:** A feature that uses machine learning to analyze API calls and detect unusual operational activity or spikes in usage.

**Question:** What is AWS Config?
**Answer:** A service that enables you to assess, audit, and evaluate the configurations and relationships of your AWS resources.

**Question:** What happens if an AWS Config rule evaluates an S3 bucket and finds it is unencrypted?
**Answer:** The resource is marked as "Noncompliant" in the Config dashboard.

**Question:** Can AWS Config automatically fix noncompliant resources?
**Answer:** Yes, by configuring AWS Config Auto-Remediation using Systems Manager (SSM) automation documents.

**Question:** What is AWS X-Ray used for?
**Answer:** To trace user requests from end-to-end through a distributed application to identify performance bottlenecks and errors.

**Question:** What is the X-Ray Service Map?
**Answer:** A visual representation of your application's architecture that highlights relationships, latency, and failure rates between microservices.

**Question:** How do you send data from an EC2 instance to AWS X-Ray?
**Answer:** By installing and running the X-Ray daemon on the EC2 instance and giving it the appropriate IAM permissions.

**Question:** What is Amazon EventBridge (formerly CloudWatch Events)?
**Answer:** A serverless event bus that detects changes in your AWS environment (like an EC2 instance starting) and triggers automated responses (like a Lambda function).

**Question:** Which service would you use to trigger a Lambda function every day at 5:00 PM?
**Answer:** Amazon EventBridge (using a Scheduled Rule/Cron expression).

---

## 🛠️ 8. Hands-on Labs (Test Your Skills)

**Lab 1:** You need to monitor the disk space (e.g., 80% full) on your EC2 Linux instance, but you notice CloudWatch doesn't show this metric by default.
<br><br><br>

**Lab 2:** You want to receive an email alert immediately if anyone in your organization logs in as the "Root" user.
<br><br><br>

**Lab 3:** An auditor asks you to prove that an EC2 instance had a specific Security Group attached to it three months ago.
<br><br><br>

**Lab 4:** You want to ensure that no developer can launch an EC2 instance without attaching a specific tag (e.g., `Environment = Production`). If they do, you want to be alerted.
<br><br><br>

**Lab 5:** Your serverless e-commerce application is slow. Users complain that checking out takes 10 seconds. The architecture involves API Gateway, Lambda, and DynamoDB. How do you find the exact bottleneck?
<br><br><br>

**Lab 6:** Your application generates a log file `/var/log/app.log`. You want to automatically search this file in real-time and trigger an alarm if the phrase "Database Timeout" occurs more than 5 times in a minute.
<br><br><br>

**Lab 7:** You need to retain your company's AWS API auditing logs for 7 years to comply with strict financial regulations.
<br><br><br>

**Lab 8:** You want to automatically trigger an AWS Lambda function the exact millisecond an EC2 instance enters the "Terminated" state.
<br><br><br>

**Lab 9:** You notice a massive spike in your AWS bill. You suspect someone is making thousands of unauthorized API calls to create expensive resources, but you don't know who.
<br><br><br>

**Lab 10:** Your company policy mandates that all EBS volumes must be encrypted. A developer accidentally creates an unencrypted volume. You want AWS to automatically encrypt it or delete it without human intervention.
<br><br><br>

---

## 🔧 9. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Install the **CloudWatch Agent** on the EC2 instance. Configure the agent to collect OS-level metrics (like disk space and RAM) and push them to CloudWatch as Custom Metrics.

**Solution 2:** Create an **Amazon EventBridge Rule** that listens for "Console Login" events where the user is "Root". Set the target of the rule to an Amazon SNS Topic, which will send you an email.

**Solution 3:** Navigate to **AWS Config**. Search for the specific EC2 instance resource ID, and view the "Resource Timeline" to see exactly what its configuration was three months ago.

**Solution 4:** Enable an **AWS Config Rule** for `required-tags`. If an EC2 instance is launched without the correct tags, Config will flag it as non-compliant and can send an SNS notification to the admin team.

**Solution 5:** Enable **AWS X-Ray** tracing on the API Gateway and the Lambda function. View the X-Ray Service Map to see exactly which hop (API to Lambda, or Lambda to DynamoDB) is taking the most milliseconds.

**Solution 6:** Push the log file to CloudWatch Logs using the CloudWatch Agent. Create a **Metric Filter** looking for "Database Timeout". Create a **CloudWatch Alarm** based on that metric to alert you when it crosses the threshold.

**Solution 7:** Configure an **AWS CloudTrail Trail** to deliver all log files to an Amazon S3 Bucket. Configure an S3 Lifecycle Policy to transition the logs to Glacier Deep Archive after 90 days, and expire them after 7 years.

**Solution 8:** Create an **Amazon EventBridge Rule**. Select the Event Pattern: "EC2 Instance State-change Notification" and specify the "Terminated" state. Set your Lambda function as the target.

**Solution 9:** Open the **AWS CloudTrail** console and view the "Event History" (or query it using Amazon Athena). Filter the events by "Event Name" (e.g., `RunInstances`) to identify the IAM user making the API calls.

**Solution 10:** Implement an **AWS Config Auto-Remediation** rule. When the rule detects an unencrypted EBS volume, it automatically executes an AWS Systems Manager (SSM) Automation Document to take a snapshot, encrypt it, and recreate the volume.

# SAA-C03 Chapter 7: Serverless & Event-Driven Architecture

---

## 📖 1. The Big Picture: Code Without Servers
**The Objective:** To build and run applications without provisioning, managing, or scaling the underlying servers or operating systems.
**The Problem:** Running EC2 instances means you pay for idle time. If your code only needs to run for 5 seconds when a user uploads a photo, keeping an EC2 server running 24/7 is a massive waste of money. Furthermore, scaling EC2 instances takes minutes, which is too slow for sudden, massive traffic spikes.
**The Solution:** We use **AWS Lambda** to run code only when triggered by an event (paying by the millisecond). We use **Amazon API Gateway** to act as the "front door" for external users to trigger our code, and we use **AWS Step Functions** to orchestrate complex workflows involving multiple Lambda functions.
**The Trade-offs:** Serverless architectures introduce "Cold Starts" (a slight delay when a function is triggered after being idle). Lambda also has hard limits, such as a maximum execution time of 15 minutes, meaning it is not suitable for long-running batch jobs.

---

## 🧠 2. Core Theory: AWS Lambda (Compute)

### What is AWS Lambda?
Lambda is a serverless compute service. You upload your code (Python, Node.js, Java, etc.), and Lambda runs it in response to "events" (like an S3 upload, a DynamoDB update, or an HTTP request).
* **Pricing:** You pay only for the compute time you consume, billed in **milliseconds**. If your code doesn't run, you pay $0.
* **Scaling CPU and Network:** You cannot manually choose CPU or Network speed in Lambda. You only allocate **RAM (Memory)**. As you increase the RAM, AWS automatically provisions more CPU and network bandwidth proportionally.

### Lambda Limitations (Crucial Exam Traps)
* **Execution Timeout:** Maximum of **15 minutes**. If a task takes 16 minutes, Lambda will forcefully kill it.
* **RAM Limit:** Maximum of **10 GB**.
* **Deployment Package Size:** Maximum of 50 MB zipped, or 250 MB unzipped. If your code is larger, you must use a container image (up to 10 GB).
* **Concurrency Limit:** Default limit of 1,000 concurrent executions per region (can be increased via a support ticket).



---

## 🌐 3. Core Theory: Amazon API Gateway

### What is API Gateway?
API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. It is the "front door" for your serverless applications.
* **Integration:** It seamlessly connects external HTTP requests to backend AWS services, most commonly AWS Lambda, but it can also talk directly to DynamoDB or Kinesis.

### API Gateway Features
* **Throttling:** Protects your backend from DDoS attacks or traffic spikes by limiting the number of requests per second (e.g., returning a `429 Too Many Requests` error).
* **Caching:** You can enable an API Gateway Cache to store responses. If 10,000 users request the exact same data, API Gateway serves it from the cache, preventing 10,000 executions of your Lambda function (saving money).
* **Authentication:** Integrates easily with **Amazon Cognito** (for mobile/web users) or **IAM** to secure your API endpoints.

---

## ⚙️ 4. Core Theory: Orchestration & Event Buses

### AWS Step Functions
When you have complex logic (e.g., "Run Lambda A, if it succeeds run Lambda B, if it fails wait 10 minutes and run Lambda C"), doing this inside Lambda code is an anti-pattern.
* **What it is:** A serverless orchestration service that lets you combine AWS Lambda functions and other services into visual workflows (State Machines).
* **Standard vs. Express Workflows:** Standard workflows can run for up to **1 year** (great for long approval processes). Express workflows run for a maximum of **5 minutes** (great for high-volume data processing).

### Amazon EventBridge (formerly CloudWatch Events)
* **What it is:** A serverless event bus. It connects applications together using data from your own apps, SaaS apps, and AWS services.
* **Use Case:** It replaces the old "Cron Jobs" on Linux. You use EventBridge to schedule a Lambda function to run at exactly 8:00 AM every day.

---

## 🏗️ 5. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Run a piece of code automatically every time a user uploads an image to S3. | **S3 Event Notification triggering Lambda** |
| Create a public-facing REST API that scales instantly to zero when not in use. | **API Gateway + AWS Lambda** |
| Process a complex order workflow that takes 3 days to complete. | **AWS Step Functions (Standard Workflow)** |
| Run a database cleanup script every night at 3:00 AM. | **Amazon EventBridge scheduled rule + Lambda** |
| Prevent an API from being overwhelmed by a sudden surge of millions of requests. | **API Gateway Throttling** |
| A Lambda function needs more CPU power to process video files faster. | **Increase the RAM allocated to the Lambda function** |
| Reduce costs on a highly requested API that always returns the same database result. | **Enable API Gateway Caching** |

---

## 🤝 6. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is AWS Lambda?
**Answer:** A serverless compute service that runs code in response to events and automatically manages the underlying compute resources.

**Question:** What is the maximum execution time limit for an AWS Lambda function?
**Answer:** 15 minutes.

**Question:** How do you increase the CPU power of a Lambda function?
**Answer:** By increasing the amount of allocated memory (RAM); AWS automatically scales CPU power proportionally.

**Question:** What is a Lambda "Cold Start"?
**Answer:** The slight delay that occurs when AWS provisions the execution environment for a function that hasn't been used recently.

**Question:** How can you prevent Lambda Cold Starts for latency-sensitive applications?
**Answer:** By enabling Provisioned Concurrency, which keeps a specified number of execution environments initialized and ready to respond immediately.

**Question:** What is the maximum deployment package size for AWS Lambda?
**Answer:** 50 MB zipped, or 250 MB unzipped (unless using a container image, which supports up to 10 GB).

**Question:** What is Amazon API Gateway?
**Answer:** A managed service that allows developers to create, publish, maintain, monitor, and secure APIs at any scale.

**Question:** How does API Gateway protect backend services from being overwhelmed?
**Answer:** By utilizing request throttling, which returns an HTTP 429 error if traffic exceeds defined limits.

**Question:** What is the benefit of enabling caching on API Gateway?
**Answer:** It reduces the number of calls made to your backend endpoint (like Lambda), reducing latency and saving on execution costs.

**Question:** What is AWS Step Functions?
**Answer:** A serverless orchestration service used to coordinate multiple AWS services into visual workflows.

**Question:** When should you use Step Functions instead of chaining Lambda functions together?
**Answer:** When your workflow requires complex logic, state management, long delays, or error handling/retries that exceed Lambda's 15-minute limit.

**Question:** What is the maximum duration a Step Functions Standard Workflow can run?
**Answer:** Up to 1 year.

**Question:** What is Amazon EventBridge?
**Answer:** A serverless event bus that connects applications together by routing events between AWS services, custom apps, and SaaS applications.

**Question:** How do you run a serverless task on a recurring schedule (like a cron job)?
**Answer:** Create an Amazon EventBridge scheduled rule that sets AWS Lambda as the target.

**Question:** What happens if a synchronous Lambda invocation fails?
**Answer:** It returns an error to the caller immediately. The caller is responsible for retrying the request.

**Question:** What happens if an asynchronous Lambda invocation (like from S3) fails?
**Answer:** Lambda automatically retries the event up to two times before sending it to a Dead Letter Queue (DLQ) if configured.

**Question:** How do you securely pass database passwords to a Lambda function?
**Answer:** Store the passwords in AWS Secrets Manager or SSM Parameter Store and retrieve them via the Lambda code.

**Question:** What is a Lambda Layer?
**Answer:** A .zip file archive that contains library dependencies, custom runtimes, or configuration files, allowing you to share code across multiple Lambda functions.

**Question:** Can AWS Lambda functions access resources inside a private VPC?
**Answer:** Yes, by attaching the Lambda function to private subnets within your VPC and assigning it a Security Group.

**Question:** If a Lambda function is in a private VPC, how does it access the public internet?
**Answer:** It must be routed through a NAT Gateway located in a public subnet; Lambda cannot use an Internet Gateway directly.

---

## 🛠️ 7. Hands-on Labs (Test Your Skills)

**Lab 1:** Every time a user uploads a profile picture to your S3 bucket, you need to automatically generate a thumbnail version without provisioning servers.
<br><br><br>

**Lab 2:** You have a Python script that takes 20 minutes to process a large financial report. You want to move this script to a serverless architecture.
<br><br><br>

**Lab 3:** Your API is facing a DDoS attack. You need to immediately restrict users to a maximum of 100 requests per second to protect your backend database.
<br><br><br>

**Lab 4:** You need to build an e-commerce checkout process: Charge Credit Card -> Update Inventory -> Send Email. If "Update Inventory" fails, you must automatically refund the credit card.
<br><br><br>

**Lab 5:** You need a script to check for unused EBS volumes and delete them to save money. This script must run automatically every Sunday at midnight.
<br><br><br>

**Lab 6:** Your Lambda function is processing video frames but is running too slowly. You check the configuration and see it only has 128 MB of RAM allocated.
<br><br><br>

**Lab 7:** An API Gateway endpoint triggers a Lambda function that queries a static pricing table in DynamoDB. The query costs are getting too high because millions of users are requesting the same prices.
<br><br><br>

**Lab 8:** You have 50 different Lambda functions that all use the exact same external Python library (e.g., Pandas). You want to manage this library centrally instead of zipping it into 50 different deployment packages.
<br><br><br>

**Lab 9:** You want to ensure only authenticated users from your mobile app can trigger your Lambda backend via API Gateway.
<br><br><br>

**Lab 10:** Your application experiences sudden, massive traffic spikes every morning at 9:00 AM. The first few users experience 3-second delays (Cold Starts) while AWS spins up the Lambda functions.
<br><br><br>

---

## 🔧 8. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Configure an S3 Event Notification on the bucket. Set the destination to trigger an AWS Lambda function that contains image-resizing code.

**Solution 2:** You cannot use Lambda directly because the 20-minute execution exceeds the 15-minute hard limit. You must refactor the code to run on AWS Fargate (Serverless Containers) or break the script into smaller chunks orchestrated by Step Functions.

**Solution 3:** Open the API Gateway console, navigate to the API's Usage Plan, and configure Throttling limits (Rate = 100 requests per second).

**Solution 4:** Build the workflow using AWS Step Functions. Configure "Catch" and "Retry" blocks to handle errors, and use the "Saga Pattern" to trigger a compensating transaction (the refund) if a later step fails.

**Solution 5:** Create an Amazon EventBridge rule. Set the event source to a "Schedule" using a Cron expression (`cron(0 0 * * ? *)`), and set the target to your AWS Lambda cleanup function.

**Solution 6:** Increase the RAM allocation in the Lambda configuration. Because CPU is tied to RAM, increasing the memory to 1024 MB or higher will give the function the compute power it needs to process videos faster.

**Solution 7:** Enable API Gateway Caching. When enabled, API Gateway caches the Lambda response for a specified TTL (Time to Live), serving users directly from the cache and dropping DynamoDB execution costs to zero for those hits.

**Solution 8:** Create an AWS Lambda Layer containing the Pandas library. Attach this single Layer to all 50 Lambda functions, keeping your deployment packages small and centrally managed.

**Solution 9:** Integrate Amazon Cognito User Pools with your API Gateway as an Authorizer. Users must present a valid JWT (JSON Web Token) from Cognito to successfully call the API.

**Solution 10:** Configure Provisioned Concurrency on your Lambda function just before 9:00 AM. This instructs AWS to pre-warm the execution environments, entirely eliminating cold start latency for the morning spike.

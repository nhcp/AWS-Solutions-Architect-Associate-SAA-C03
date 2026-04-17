# SAA-C03 Chapter 3: IAM, Security & Encryption

---

## 📖 1. The Big Picture: Identity and Zero Trust
**The Objective:** To ensure that only the right people and machines have access to the right resources, and that all data is encrypted both at rest and in transit.
**The Problem:** Hardcoding passwords into application code leads to massive data breaches. Giving developers full "Admin" access leads to accidental deletions. Managing hundreds of AWS accounts individually is an auditing nightmare.
**The Solution:** We implement the **Principle of Least Privilege** using AWS Identity and Access Management (IAM). We use **IAM Roles** instead of static passwords for machines. For enterprise scale, we use **AWS Organizations** to centrally manage accounts and enforce **Service Control Policies (SCPs)**. We protect data using **AWS KMS** and edge firewalls like **AWS WAF**.
**The Trade-offs:** Strict security policies can slow down developer agility. Encryption adds slight processing overhead. Advanced security tools (like Shield Advanced or Macie) can be very expensive.

---

## 🧠 2. Core Theory: IAM & Organizations

### IAM (Identity and Access Management)
IAM is a global service used to manage access to AWS.
* **Users:** Physical people or service accounts.
* **Groups:** Collections of users (e.g., "Developers").
* **Policies:** JSON documents that explicitly Allow or Deny permissions. **Deny always overrides an Allow.**
* **Roles:** Temporary identities. Unlike a User, a Role does not have long-term passwords or access keys. Services (like EC2) or users "assume" a role temporarily using **AWS STS (Security Token Service)**.



### AWS Organizations & SCPs
When a company has 50 different AWS accounts (Prod, Dev, Test, HR), managing them individually is impossible.
* **AWS Organizations:** Allows you to centrally manage multiple AWS accounts, consolidate billing, and apply organization-wide rules.
* **Service Control Policies (SCPs):** The ultimate "God Mode" firewall for permissions. If an SCP denies access to a service (e.g., "Deny Redshift"), *nobody* in that account can use Redshift, not even the Root user.



---

## 🛡️ 3. Core Theory: Identity Federation & SSO

When dealing with thousands of employees or millions of mobile app users, you do not create IAM users for them. You "federate" them.
* **AWS IAM Identity Center (formerly SSO):** The modern way to manage workforce access. You connect your corporate Active Directory (AD) or Okta, and users log in once to access multiple AWS accounts.
* **Amazon Cognito:** Designed for customer-facing web and mobile apps. It handles user sign-up, sign-in, and gives temporary AWS access to mobile users (e.g., letting an app upload a photo to S3 directly).

---

## 🔐 4. Core Theory: Data Security & Encryption

### Key Management Service (KMS)
KMS is a managed service to create and control cryptographic keys.
* **Symmetric KMS:** The same key encrypts and decrypts the data.
* **Customer Managed Keys (CMK):** You control the key rotation and policies.
* **Envelope Encryption:** KMS doesn't encrypt your massive 100GB files directly. It encrypts a "Data Key," and that Data Key encrypts your file.

### CloudHSM
If you have strict government or financial regulations requiring a *dedicated hardware security module* where AWS cannot even access the underlying hardware, you use CloudHSM (FIPS 140-2 Level 3 compliance).

### Secrets Manager vs. Parameter Store
* **Systems Manager Parameter Store:** Free storage for configuration data and passwords.
* **AWS Secrets Manager:** Paid service, but its superpower is **Automatic Secret Rotation**. It can automatically change your RDS database passwords every 30 days.

---

## 🌐 5. Core Theory: Edge Security & Threat Detection

* **AWS WAF (Web Application Firewall):** Protects against Layer 7 attacks like SQL Injection (SQLi) and Cross-Site Scripting (XSS). Attached to ALBs, API Gateways, or CloudFront.
* **AWS Shield:** Protects against DDoS attacks. "Shield Standard" is free. "Shield Advanced" costs $3,000/month and offers financial protection against DDoS-related billing spikes.
* **Amazon Macie:** Uses Machine Learning to scan S3 buckets and discover sensitive data, like exposed credit card numbers or passports.
* **Amazon GuardDuty:** An intelligent threat detection service that monitors CloudTrail logs, VPC Flow Logs, and DNS logs for suspicious activity (e.g., an EC2 instance suddenly mining cryptocurrency).

---

## 🏗️ 6. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| An EC2 instance needs to securely upload files to an S3 bucket. | **IAM Role attached to the EC2 instance** |
| Ensure no developer can ever launch an EC2 instance outside of the US-East region. | **AWS Organizations + SCP (Service Control Policy)** |
| Automatically rotate a database password every 60 days without downtime. | **AWS Secrets Manager** |
| Protect a web application from SQL injection and Cross-Site Scripting. | **AWS WAF (Web Application Firewall)** |
| A mobile app needs users to log in with Facebook/Google and upload profile pictures. | **Amazon Cognito** |
| Detect if an EC2 instance is communicating with a known malicious IP address. | **Amazon GuardDuty** |
| Company requires encryption keys to be managed on single-tenant, dedicated hardware. | **AWS CloudHSM** |

---

## 🤝 7. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is the Principle of Least Privilege?
**Answer:** Giving a user or system only the bare minimum permissions necessary to perform their specific job.

**Question:** What is the difference between an IAM User and an IAM Role?
**Answer:** A User has long-term static credentials (passwords/access keys), while a Role provides temporary, rotating credentials via STS.

**Question:** If an IAM policy has an explicit "Allow" and an explicit "Deny" for the same action, which wins?
**Answer:** The explicit "Deny" always wins and overrides any Allow permissions.

**Question:** What is AWS STS?
**Answer:** The Security Token Service, which grants temporary, limited-privilege credentials for AWS resource access.

**Question:** How should you grant an EC2 instance access to a DynamoDB table?
**Answer:** Create an IAM Role with DynamoDB access and attach that Role to the EC2 instance profile.

**Question:** What is AWS Organizations?
**Answer:** An account management service that lets you consolidate multiple AWS accounts into an organization that you centrally manage.

**Question:** What is a Service Control Policy (SCP)?
**Answer:** A policy used in AWS Organizations to manage the maximum available permissions for all accounts in an organization.

**Question:** Can an SCP grant permissions?
**Answer:** No, SCPs only filter or restrict permissions; IAM policies must still be used to actually grant the access.

**Question:** What is AWS KMS?
**Answer:** A managed service that makes it easy to create, manage, and control cryptographic keys used to encrypt your data.

**Question:** What is Envelope Encryption?
**Answer:** The practice of encrypting plaintext data with a data key, and then encrypting that data key under another key (the KMS Master Key).

**Question:** What is the main difference between KMS and CloudHSM?
**Answer:** KMS is a multi-tenant managed service; CloudHSM provides a dedicated, single-tenant hardware appliance for strict regulatory compliance.

**Question:** When should you choose AWS Secrets Manager over SSM Parameter Store?
**Answer:** When you require built-in, automated password rotation for services like RDS databases.

**Question:** What does Amazon Cognito do?
**Answer:** It provides authentication, authorization, and user management for customer-facing web and mobile apps.

**Question:** What is Amazon Macie?
**Answer:** A data security service that uses machine learning to discover and protect sensitive data (like PII) stored in Amazon S3.

**Question:** What is Amazon GuardDuty?
**Answer:** A continuous threat detection service that monitors for malicious activity and unauthorized behavior to protect your AWS accounts.

**Question:** What is AWS WAF?
**Answer:** A Web Application Firewall that helps protect web applications from common web exploits like SQL injection and XSS at Layer 7.

**Question:** What services can AWS WAF be attached to?
**Answer:** Application Load Balancers (ALB), Amazon CloudFront, Amazon API Gateway, and AWS AppSync.

**Question:** What is the difference between AWS Shield Standard and Advanced?
**Answer:** Standard provides free, basic DDoS protection; Advanced provides enhanced 24/7 protection and DDoS cost protection.

**Question:** What is IAM Identity Center (formerly AWS SSO)?
**Answer:** A service that centrally manages single sign-on access to multiple AWS accounts and business applications for your workforce.

**Question:** How can you track exactly who made an API call to terminate an EC2 instance?
**Answer:** By reviewing the logs in AWS CloudTrail, which records all account activity and API usage.

---

## 🛠️ 8. Hands-on Labs (Test Your Skills)

**Lab 1:** An application running on EC2 needs to read data from an S3 bucket. Design the most secure way to grant this permission without putting access keys in the code.
<br><br><br>

**Lab 2:** Your company has a "Dev" account and a "Prod" account. You need to ensure absolutely no one can launch an expensive `p4d.24xlarge` instance in the "Dev" account.
<br><br><br>

**Lab 3:** You have an RDS database. Security policy dictates that the database password must be changed every 30 days without manual human intervention.
<br><br><br>

**Lab 4:** A mobile game needs to allow users to log in using their Apple or Google accounts, and then give them temporary access to save their game state to DynamoDB.
<br><br><br>

**Lab 5:** You need to encrypt a massive 50GB video file. Explain how KMS interacts with this file.
<br><br><br>

**Lab 6:** An external auditor requires proof that all cryptographic keys are generated and stored on a dedicated hardware appliance that AWS employees cannot logically access.
<br><br><br>

**Lab 7:** Your web server is experiencing a Layer 7 attack where hackers are putting SQL commands into your website's login form. How do you block this?
<br><br><br>

**Lab 8:** Your company wants to automatically scan petabytes of data in S3 to find out if any developers accidentally uploaded files containing customer credit card numbers.
<br><br><br>

**Lab 9:** You need a centralized dashboard to check if any of your EC2 instances are secretly communicating with known cryptocurrency mining IP addresses.
<br><br><br>

**Lab 10:** An employee needs to access AWS resources across 3 different company AWS accounts. Instead of creating 3 IAM users for them, what is the modern architectural solution?
<br><br><br>

---

## 🔧 9. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Create an IAM Role with an S3 ReadOnly policy. Attach this IAM Role to the EC2 Instance via an Instance Profile. The application will use STS to get temporary credentials automatically.

**Solution 2:** Use AWS Organizations. Create an Organizational Unit (OU) for "Dev" and apply a Service Control Policy (SCP) that explicitly denies the `ec2:RunInstances` action for the `p4d.24xlarge` instance type.

**Solution 3:** Store the database credentials in AWS Secrets Manager and configure an automated rotation schedule. Secrets Manager will trigger a Lambda function to change the password in RDS securely.

**Solution 4:** Implement Amazon Cognito Identity Pools. Cognito will authenticate the user via Google/Apple and exchange that authentication for temporary AWS STS credentials mapped to a limited IAM Role.

**Solution 5:** Use Envelope Encryption. AWS KMS will generate a "Data Key". The plaintext Data Key encrypts the 50GB file locally. Then, KMS encrypts the Data Key itself, and you store the encrypted Data Key alongside the encrypted video file.

**Solution 6:** Provision an AWS CloudHSM cluster. This provides single-tenant, FIPS 140-2 Level 3 validated hardware security modules where the customer has exclusive control over the cryptographic keys.

**Solution 7:** Deploy AWS WAF (Web Application Firewall) and attach it to your Application Load Balancer (ALB). Enable the managed rule set that specifically blocks SQL Injection (SQLi).

**Solution 8:** Enable Amazon Macie. Macie uses machine learning and pattern matching to automatically discover and alert you to sensitive data, such as Personally Identifiable Information (PII), stored in S3.

**Solution 9:** Enable Amazon GuardDuty. GuardDuty will continuously analyze VPC Flow Logs, DNS logs, and CloudTrail events to detect malicious activity, including communication with known malicious IP addresses.

**Solution 10:** Use AWS IAM Identity Center (formerly SSO). Connect it to your corporate directory (like Active Directory). The employee logs in once and is granted federated role access to the respective accounts.

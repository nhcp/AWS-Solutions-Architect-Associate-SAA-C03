# SAA-C03 Chapter 13: Infrastructure as Code & Automation

---

## 📖 1. The Big Picture: Automating the Cloud
**The Objective:** To provision, manage, and update AWS infrastructure using code rather than clicking through the AWS Management Console manually.
**The Problem:** Manually creating a VPC, 10 subnets, an Auto Scaling Group, and an RDS database takes hours. If you make a typo in a Security Group, your app crashes. Worse, if you need to replicate this exact environment in a different region for Disaster Recovery, doing it manually is a nightmare.
**The Solution:** We use **Infrastructure as Code (IaC)**. We write a text file (using **AWS CloudFormation**) that describes our infrastructure. AWS reads the file and builds everything automatically, in exactly the right order. For developers who don't want to learn infrastructure at all, we use **AWS Elastic Beanstalk**. For patching and managing servers at scale, we use **AWS Systems Manager (SSM)**.
**The Trade-offs:** Writing CloudFormation templates requires learning its specific syntax (YAML or JSON) and debugging failed stack creations can be complex.

---

## 🧠 2. Core Theory: AWS CloudFormation (IaC)

### What is CloudFormation?
CloudFormation is a declarative way of outlining your AWS Infrastructure. You state *what* you want (e.g., "I want a t2.micro EC2 instance and an S3 bucket"), and AWS figures out *how* to provision it.
* **Templates:** A JSON or YAML file containing the blueprint of your architecture.
* **Stacks:** When you upload a Template to AWS, it creates a "Stack". The Stack is the actual collection of running resources (the EC2 instances, the buckets, etc.).
* **Updating:** If you want to change your infrastructure (e.g., change the instance from t2.micro to t2.large), you update the template text file and upload it. CloudFormation calculates the difference and updates the Stack.
* **Deletion:** If you delete a CloudFormation Stack, *every single resource* created by that stack is automatically destroyed (unless you configure a DeletionPolicy to retain it, like for databases).

### CloudFormation Advanced Features
* **StackSets:** Allows you to deploy the same CloudFormation template across multiple AWS Accounts and multiple AWS Regions simultaneously. (Crucial for enterprise governance).
* **Drift Detection:** If someone bypasses CloudFormation and manually changes a Security Group rule in the console, your infrastructure has "drifted" from the code. Drift Detection flags these manual changes so you can fix them.



---

## 🚀 3. Core Theory: AWS Elastic Beanstalk (PaaS)

### What is Elastic Beanstalk?
Elastic Beanstalk is a Platform as a Service (PaaS). It is designed specifically for developers who just want to write code (Java, Python, Node.js, PHP) and not worry about the underlying infrastructure.
* **How it works:** The developer uploads a ZIP file of their code. Elastic Beanstalk automatically handles the provisioning of the EC2 instances, the Application Load Balancer, the Auto Scaling Group, and the database.
* **Control:** Unlike fully managed serverless platforms, Elastic Beanstalk still gives you full access to the underlying EC2 instances if you need to SSH into them and tweak the OS.

---

## 🛠️ 4. Core Theory: AWS Systems Manager (SSM)

### What is Systems Manager?
SSM is an operational hub that helps you automate management tasks across thousands of EC2 instances and even on-premises servers. You install the SSM Agent on your instances to enable this.

### Key SSM Features (Exam Favorites)
* **SSM Run Command:** Run a script (like installing an antivirus update) across 1,000 EC2 instances simultaneously without needing to SSH into any of them.
* **SSM Patch Manager:** Automatically apply OS and software patches to your EC2 instances on a defined schedule.
* **SSM Session Manager:** Allows you to securely access the terminal/command line of an EC2 instance directly from the AWS Console. **No Bastion Hosts, no SSH keys, and no open Port 22 required.**
* **SSM Parameter Store:** A secure, hierarchical storage service for configuration data and secrets (like database passwords).

---

## 🏗️ 5. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Provision identical environments (Dev, Test, Prod) automatically without human error. | **AWS CloudFormation** |
| Deploy baseline security roles to 50 different AWS accounts at the same time. | **AWS CloudFormation StackSets** |
| Find out if an admin manually changed a firewall rule that was deployed via code. | **CloudFormation Drift Detection** |
| A developer wants to deploy a Java web app by simply uploading a ZIP file of the code. | **AWS Elastic Beanstalk** |
| Run an emergency security patch script on 500 EC2 instances simultaneously. | **AWS Systems Manager (SSM) Run Command** |
| Securely log into a private EC2 instance without opening Port 22 or using SSH keys. | **SSM Session Manager** |
| Store API keys and database strings securely for an application to retrieve at runtime. | **SSM Parameter Store** |

---

## 🤝 6. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is Infrastructure as Code (IaC)?
**Answer:** The process of managing and provisioning cloud resources using machine-readable definition files (code) rather than physical hardware configuration or interactive configuration tools.

**Question:** What is AWS CloudFormation?
**Answer:** An AWS service that allows you to model, provision, and manage AWS and third-party resources by treating infrastructure as code.

**Question:** What is the difference between a CloudFormation Template and a Stack?
**Answer:** A template is the blueprint (the code file); a stack is the actual collection of provisioned AWS resources created from that template.

**Question:** What happens when you delete a CloudFormation Stack?
**Answer:** By default, all the resources that were created by the stack are deleted.

**Question:** How can you prevent CloudFormation from deleting a critical RDS database when a stack is deleted?
**Answer:** Set the `DeletionPolicy` attribute to `Retain` or `Snapshot` in the CloudFormation template for that specific resource.

**Question:** What is CloudFormation Drift Detection?
**Answer:** A feature that detects whether the actual configuration of your resources differs (has drifted) from their expected configuration defined in the CloudFormation template.

**Question:** What are CloudFormation StackSets?
**Answer:** A feature that extends Stacks by enabling you to create, update, or delete stacks across multiple AWS accounts and regions with a single operation.

**Question:** What is AWS Elastic Beanstalk?
**Answer:** An easy-to-use service for deploying and scaling web applications and services developed with Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker.

**Question:** Is Elastic Beanstalk free?
**Answer:** The Elastic Beanstalk service itself is free; you only pay for the underlying AWS resources (like EC2 instances or S3 buckets) that it provisions.

**Question:** Can you access the underlying EC2 instances provisioned by Elastic Beanstalk?
**Answer:** Yes, you retain full control over the AWS resources powering your application and can seamlessly take over some or all of the elements of their infrastructure.

**Question:** What is AWS Systems Manager (SSM)?
**Answer:** An AWS service that gives you visibility and control of your infrastructure, helping you automate operational tasks across your AWS resources.

**Question:** What does SSM Run Command do?
**Answer:** It allows you to safely and securely remotely manage the configuration of your managed instances at scale without logging into them.

**Question:** What is the primary security benefit of SSM Session Manager?
**Answer:** It allows secure, auditable terminal access to instances without needing to open inbound ports (like SSH port 22), maintain bastion hosts, or manage SSH keys.

**Question:** What is SSM Parameter Store?
**Answer:** A service that provides secure, hierarchical storage for configuration data management and secrets management.

**Question:** What is the difference between SSM Parameter Store and AWS Secrets Manager?
**Answer:** Parameter Store is free for standard parameters but requires manual updates; Secrets Manager is paid but features built-in, automated password rotation.

**Question:** What is AWS OpsWorks?
**Answer:** A configuration management service that provides managed instances of Chef and Puppet. (Note: Being phased out/deprecated by AWS, focus on SSM instead).

**Question:** What is AWS Serverless Application Model (SAM)?
**Answer:** An open-source framework built on top of CloudFormation specifically designed for building and deploying serverless applications.

**Question:** In CloudFormation, what are the `Outputs` used for?
**Answer:** To export values (like a VPC ID or an ALB endpoint URL) so they can be imported and used by other CloudFormation stacks.

**Question:** In CloudFormation, what is the `Parameters` section used for?
**Answer:** To input custom values to your template each time you create or update a stack, making your templates reusable.

**Question:** What happens if a resource fails to create during a CloudFormation stack deployment?
**Answer:** By default, CloudFormation initiates a "Rollback" and deletes all the resources it successfully created up to the point of failure to avoid a broken, partial state.

---

## 🛠️ 7. Hands-on Labs (Test Your Skills)

**Lab 1:** You created a massive VPC architecture manually in `us-east-1`. Your boss wants the exact same architecture replicated in `eu-west-1`. How do you do this efficiently?
<br><br><br>

**Lab 2:** You deploy your infrastructure using CloudFormation. A junior admin logs into the console and manually adds a rule to a Security Group. How do you find this out?
<br><br><br>

**Lab 3:** Your company just acquired 5 new subsidiaries, each with their own AWS account. You need to deploy a standard set of IAM security roles to all 5 accounts simultaneously.
<br><br><br>

**Lab 4:** A team of Node.js developers knows nothing about AWS networking, Auto Scaling, or Load Balancers. They just want their web app hosted and scalable immediately.
<br><br><br>

**Lab 5:** A critical zero-day vulnerability requires you to run a bash script to update a software package on 1,000 EC2 Linux instances immediately.
<br><br><br>

**Lab 6:** Your security team mandates that no EC2 instance can have Port 22 (SSH) open to the internet, and Bastion Hosts are not allowed. However, admins still need command-line access to troubleshoot the servers.
<br><br><br>

**Lab 7:** Your application needs to access a third-party API key. You do not want this key hardcoded in your GitHub repository or written in plain text in your CloudFormation template.
<br><br><br>

**Lab 8:** You are deploying a CloudFormation stack, but the deployment fails halfway through because you reached your account limit for Elastic IPs. What happens to the EC2 instances that were successfully created before the failure?
<br><br><br>

**Lab 9:** You are writing a CloudFormation template for a web server, but you want the template to ask the user which EC2 instance type (e.g., t2.micro or m5.large) they want to use before launching.
<br><br><br>

**Lab 10:** You want to ensure that all EC2 instances in your account are automatically patched with the latest Windows or Linux security updates every Sunday at 3:00 AM.
<br><br><br>

---

## 🔧 8. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Write an **AWS CloudFormation** template defining the architecture. Deploy this template as a Stack in `eu-west-1`. You can now use this code to replicate the environment anywhere instantly.

**Solution 2:** Run **Drift Detection** on the CloudFormation stack. AWS will compare the deployed resources against the original template and flag the Security Group as "Drifted."

**Solution 3:** Use **CloudFormation StackSets**. This feature allows an administrator account to deploy a single CloudFormation template into multiple target AWS accounts simultaneously.

**Solution 4:** Instruct the developers to use **AWS Elastic Beanstalk**. They simply upload their Node.js application package, and Beanstalk automatically provisions the Load Balancer, EC2 instances, and Auto Scaling rules.

**Solution 5:** Use **AWS Systems Manager (SSM) Run Command**. Select all 1,000 instances based on their resource tags, input the bash script, and SSM will execute it concurrently across the fleet.

**Solution 6:** Use **SSM Session Manager**. Ensure the EC2 instances have the SSM Agent installed and an IAM role allowing SSM access. Admins can access the terminal securely through the AWS web console.

**Solution 7:** Store the API key in **SSM Parameter Store** as a `SecureString` (which encrypts it using KMS). Configure your application code or CloudFormation to retrieve the parameter dynamically at runtime.

**Solution 8:** CloudFormation automatically triggers a **Rollback**. It will cleanly delete the EC2 instances and any other resources it created during that session so you aren't charged for a half-broken deployment.

**Solution 9:** Use the **`Parameters`** section of the CloudFormation template to define an input variable for the instance type. Provide a list of `AllowedValues` (e.g., t2.micro, m5.large) so the user selects from a dropdown.

**Solution 10:** Configure an **SSM Patch Manager** baseline and link it to an SSM Maintenance Window set for Sunday at 3:00 AM. Systems Manager will automatically scan and install the approved patches.

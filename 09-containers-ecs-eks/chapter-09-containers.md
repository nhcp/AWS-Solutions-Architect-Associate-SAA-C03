# SAA-C03 Chapter 9: Containers & Microservices (ECS, EKS, Fargate)

---

## 📖 1. The Big Picture: Docker & Orchestration
**The Objective:** To package applications and their dependencies into standardized, lightweight units (containers) that run consistently across any environment, and to orchestrate thousands of them efficiently.
**The Problem:** Running microservices directly on EC2 instances leads to "dependency hell" (Code works on the developer's laptop but breaks in production due to different OS versions). Furthermore, deploying one small microservice per EC2 instance wastes massive amounts of compute resources and money.
**The Solution:** We use **Docker** to containerize the application. We store these container images in **Amazon ECR**. To manage, scale, and load-balance these containers, we use a Container Orchestrator like **Amazon ECS** (AWS native) or **Amazon EKS** (Kubernetes). To run them without managing underlying servers, we use **AWS Fargate**.
**The Trade-offs:** Containers introduce a steeper learning curve compared to standard EC2 deployments. Kubernetes (EKS) specifically requires deep specialized knowledge to configure and maintain securely.

---

## 🧠 2. Core Theory: Docker & Amazon ECR

### Docker Basics
Docker is a software platform that packages software into standardized units called containers.
* **Image:** A read-only template with instructions for creating a Docker container (contains the OS, code, and libraries).
* **Container:** A runnable instance of an image. It shares the host server's operating system kernel but runs in isolated user space.

### Amazon ECR (Elastic Container Registry)
ECR is a fully managed Docker container registry (AWS's private version of Docker Hub).
* **Security:** Images are stored securely, encrypted at rest, and access is controlled via IAM.
* **Vulnerability Scanning:** ECR can automatically scan your container images for known software vulnerabilities (CVEs) as soon as they are pushed to the registry.

---

## 🚢 3. Core Theory: Amazon ECS & Fargate

### Amazon ECS (Elastic Container Service)
ECS is AWS's highly scalable, high-performance container orchestration service. It is deeply integrated with other AWS services (ALB, IAM, CloudWatch).
* **Task Definition:** A JSON file that describes *how* a container should run (Which image? How much CPU/RAM? Which ports to open?). It is the blueprint.
* **Task:** A running instance of a Task Definition.
* **Service:** Guarantees that a specified number of Tasks are constantly running and registers them with an Application Load Balancer.

### ECS Launch Types (Crucial Exam Topic)
When you run a container in ECS, where does the actual compute power come from?
* **EC2 Launch Type:** You provision, manage, and patch the underlying EC2 instances. You must scale the EC2 instances yourself using an Auto Scaling Group. **Best for:** Strict compliance requirements, custom EC2 AMIs, or requiring specialized hardware like GPUs.
* **Fargate Launch Type:** Serverless compute for containers. You do not manage any underlying EC2 servers. You simply specify how much CPU and RAM your container needs, and AWS provisions the compute on-demand. **Best for:** Reducing operational overhead and building pure serverless architectures.



---

## ☸️ 4. Core Theory: Amazon EKS (Kubernetes)

### Amazon EKS (Elastic Kubernetes Service)
EKS is a managed service that makes it easy to run **Kubernetes** on AWS. Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.
* **Why choose EKS over ECS?**
    * **Multi-Cloud/Hybrid:** If your company runs containers on on-premises data centers, Azure, Google Cloud, and AWS, Kubernetes acts as a universal standard.
    * **Migration:** If you already have an application heavily architected for Kubernetes (using Helm charts, Pods, Services), migrating it to EKS is much easier than rewriting it for ECS.
* **Compute:** Like ECS, EKS can run its "Pods" (Kubernetes equivalent of an ECS Task) on either managed EC2 instances (EKS Worker Nodes) or serverlessly via AWS Fargate.



---

## 🏗️ 5. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Store proprietary Docker images securely with automatic vulnerability scanning. | **Amazon ECR (Elastic Container Registry)** |
| Run Docker containers natively on AWS without managing any underlying servers. | **Amazon ECS with Fargate Launch Type** |
| Run a containerized application that requires heavy GPU processing power. | **Amazon ECS with EC2 Launch Type (GPU Instances)** |
| Migrate an existing open-source Kubernetes workload to AWS. | **Amazon EKS (Elastic Kubernetes Service)** |
| Give a specific ECS container access to an S3 bucket securely. | **ECS Task IAM Role** |
| Run a batch processing job via containers that takes 3 hours to complete. | **ECS with Fargate (Lambda cannot run for 3 hours)** |
| Ensure highly available container routing based on URL paths (`/api` vs `/web`). | **ECS integrated with an Application Load Balancer (ALB)** |

---

## 🤝 6. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is the primary advantage of using Docker containers over Virtual Machines?
**Answer:** Containers are lightweight, start in milliseconds, and share the host OS kernel, allowing for much higher density and efficiency than VMs.

**Question:** What is Amazon ECR?
**Answer:** A fully managed, highly secure container registry used to store, manage, and deploy Docker container images.

**Question:** How do you automatically check your container images for security flaws in AWS?
**Answer:** Enable "Image Scanning on Push" within your Amazon ECR repository.

**Question:** What is Amazon ECS?
**Answer:** Elastic Container Service is a highly scalable, fully managed container orchestration service native to AWS.

**Question:** What is an ECS Task Definition?
**Answer:** A JSON text file that serves as a blueprint describing one or more containers, including their CPU, memory, and networking configurations.

**Question:** What is the difference between an ECS Task and an ECS Service?
**Answer:** A Task is a single running instance of a container; a Service ensures a desired number of Tasks are continuously running and load-balanced.

**Question:** What is AWS Fargate?
**Answer:** A serverless compute engine for containers that removes the need to provision, manage, or scale underlying EC2 instances.

**Question:** When should you choose the ECS EC2 Launch Type instead of Fargate?
**Answer:** When you need strict control over the underlying server OS, require specialized hardware like GPUs, or want to leverage deep discounts from Spot Instances for the host servers.

**Question:** How do you securely grant an ECS container permissions to read from a DynamoDB table?
**Answer:** Create an IAM Role with DynamoDB access and assign it as the "Task Role" in the ECS Task Definition.

**Question:** What is the difference between an ECS Task Role and an ECS Task Execution Role?
**Answer:** The Task Execution Role allows the ECS agent to pull images from ECR and send logs to CloudWatch; the Task Role grants your actual application code permissions to use AWS services.

**Question:** What is Amazon EKS?
**Answer:** A managed service that allows you to run Kubernetes on AWS without needing to install or operate your own Kubernetes control plane.

**Question:** When is Amazon EKS preferred over Amazon ECS?
**Answer:** When an organization requires a multi-cloud or hybrid-cloud strategy using the open-source Kubernetes standard.

**Question:** Can EKS run serverless containers?
**Answer:** Yes, Amazon EKS can run Pods on AWS Fargate to eliminate the need to manage EC2 worker nodes.

**Question:** In Kubernetes terminology, what is the equivalent of an ECS Task?
**Answer:** A Pod, which is the smallest deployable computing unit in Kubernetes and can contain one or more containers.

**Question:** How does an Application Load Balancer (ALB) dynamically route traffic to ECS containers that are assigned random port numbers?
**Answer:** ECS dynamically maps the container ports to the host EC2 instance ports, and automatically updates the ALB Target Group with these dynamic ports.

**Question:** What is Amazon ECS Anywhere?
**Answer:** A feature that allows you to run and manage ECS containers on your own on-premises infrastructure using the AWS ECS control plane.

**Question:** What happens to the ephemeral storage of an ECS Fargate task when it stops?
**Answer:** The ephemeral storage is tied to the lifecycle of the task; when the task stops, the data is permanently deleted.

**Question:** How can you provide persistent shared storage to containers running in ECS?
**Answer:** By mounting an Amazon EFS (Elastic File System) volume to the ECS Task Definition.

**Question:** What is AWS App Mesh?
**Answer:** A service mesh that provides application-level networking, making it easy to monitor and control communications across multiple microservices.

**Question:** How do you capture the standard output (stdout) logs of your containers running in ECS or Fargate?
**Answer:** By configuring the `awslogs` log driver in the Task Definition, which automatically sends container logs to Amazon CloudWatch Logs.

---

## 🛠️ 7. Hands-on Labs (Test Your Skills)

**Lab 1:** You are tired of patching the underlying Linux OS for your fleet of 50 Docker containers. You want AWS to handle all infrastructure management while you just run the code.
<br><br><br>

**Lab 2:** Your containerized application requires access to highly specialized, custom-built hardware attached to specific EC2 instance types.
<br><br><br>

**Lab 3:** You have built a custom Docker image containing your company's proprietary code. You need to store it in AWS securely and ensure no known security vulnerabilities exist within the image libraries.
<br><br><br>

**Lab 4:** You need to pass database passwords and API keys to your ECS containers securely without hardcoding them into the Docker image or the Task Definition JSON.
<br><br><br>

**Lab 5:** Your company acquired a startup that runs their entire microservice architecture on Kubernetes. You need to migrate this workload to AWS with the least amount of code refactoring.
<br><br><br>

**Lab 6:** A Python script running in a Docker container inside ECS Fargate needs permission to read files from a specific S3 bucket.
<br><br><br>

**Lab 7:** You need to deploy 5 instances of your web-facing container and automatically distribute incoming internet traffic across all of them based on the URL path.
<br><br><br>

**Lab 8:** Your containers generate massive amounts of log data. You need to centrally collect, view, and set alarms based on these logs without installing custom logging agents inside the containers.
<br><br><br>

**Lab 9:** You are running a stateful application in ECS Fargate. If a container crashes and is restarted, it must retain its data and files from the previous session.
<br><br><br>

**Lab 10:** You want to run your ECS Tasks locally in your own company's physical data center to meet latency requirements, but you want to manage them using the AWS Cloud Console.
<br><br><br>

---

## 🔧 8. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Migrate the workload to use **Amazon ECS with the Fargate Launch Type**. AWS handles all server provisioning, scaling, and OS patching automatically.

**Solution 2:** Deploy the containers using **Amazon ECS with the EC2 Launch Type**. This allows you to choose specific, specialized EC2 instance types (like GPU instances) and manage the underlying hosts.

**Solution 3:** Create a repository in **Amazon ECR (Elastic Container Registry)**. Enable the "Scan on Push" setting to automatically check the image for Common Vulnerabilities and Exposures (CVEs) upon upload.

**Solution 4:** Store the sensitive data in **AWS Secrets Manager** or **SSM Parameter Store**. Reference the ARNs of these secrets in the `secrets` section of your ECS Task Definition, allowing ECS to inject them securely as environment variables at runtime.

**Solution 5:** Provision an **Amazon EKS (Elastic Kubernetes Service)** cluster. Since the startup already uses Kubernetes manifests and tooling, EKS provides a fully compatible environment for a smooth migration.

**Solution 6:** Create an IAM Role containing an S3 ReadOnly Access policy. In the ECS Task Definition, assign this role as the **Task Role** (not the Task Execution Role) so the Python application can assume it.

**Solution 7:** Create an **ECS Service** with the desired task count set to 5. Configure the service to register its tasks with an **Application Load Balancer (ALB)**, and use ALB listener rules to route traffic based on the URL path.

**Solution 8:** Configure the **`awslogs` log driver** within the ECS Task Definition. This natively routes the `stdout` and `stderr` streams directly into an Amazon CloudWatch Logs log group.

**Solution 9:** Because Fargate local storage is ephemeral, you must configure the Task Definition to mount an **Amazon EFS (Elastic File System)** volume. EFS provides persistent, shared storage that survives container restarts.

**Solution 10:** Install the ECS agent on your on-premises servers and register them using **Amazon ECS Anywhere**. This allows the AWS cloud control plane to manage containers running on your physical hardware.

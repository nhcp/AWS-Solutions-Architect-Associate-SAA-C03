# SAA-C03 Chapter 2: EC2 Compute & High Availability

---

## 📖 1. The Big Picture: Compute & Resiliency
**The Objective:** To provision scalable virtual servers (EC2) and design architectures that can survive a data center failure without human intervention.
**The Problem:** Hardware breaks. If you run your application on a single server, and that server crashes, your business goes offline. Furthermore, if your application suddenly gets a spike in traffic (e.g., Black Friday), a single server will be overwhelmed and crash.
**The Solution:** We implement **High Availability (HA)** and **Scalability**. We deploy multiple EC2 instances across different Availability Zones. We place an **Elastic Load Balancer (ELB)** in front of them to distribute traffic, and an **Auto Scaling Group (ASG)** behind them to automatically add or remove servers based on demand.
**The Trade-offs:** Running multiple servers across different Availability Zones increases costs. Auto-scaling requires careful configuration of health checks and "cooldown" periods to prevent the system from continuously turning servers on and off.

---

## 🧠 2. Core Theory: EC2 Fundamentals

### Amazon EC2 (Elastic Compute Cloud)
EC2 is the backbone of AWS. It allows you to rent virtual machines in the cloud.
* **AMI (Amazon Machine Image):** The template for your EC2 instance (Operating System + Software). AMIs are built for specific AWS Regions.
* **Instance Types:** AWS categorizes servers by their strength:
    * **Compute Optimized (C-Class):** For gaming servers, high-performance web servers, and batch processing.
    * **Memory Optimized (R-Class):** For in-memory databases (like Redis) and real-time data processing.
    * **Storage Optimized (I-Class):** For large data warehousing and distributed file systems.

### EC2 Purchasing Options (Crucial for Exam)
* **On-Demand:** Pay by the second. Highest cost, but no long-term commitment. Good for short, unpredictable workloads.
* **Reserved Instances (RI):** Commit to 1 or 3 years. Offers up to 72% discount. Best for steady-state databases.
* **Spot Instances:** Bid on unused AWS capacity. Up to 90% discount, but AWS can terminate your instance with a 2-minute warning. Best for batch processing and fault-tolerant jobs.
* **Dedicated Hosts:** A physical server fully dedicated to your use. Required for strict compliance or bringing your own software licenses (BYOL).

---

## ⚖️ 3. Core Theory: Elastic Load Balancing (ELB)

### Why use a Load Balancer?
An ELB acts as the single point of contact for your clients. It seamlessly routes incoming traffic across multiple healthy EC2 instances in different Availability Zones.

### Types of Load Balancers
* **Application Load Balancer (ALB):** Operates at **Layer 7** (HTTP/HTTPS). It can route traffic based on the URL path (e.g., `/images` vs `/api`) or hostname. It is the default choice for web applications.
* **Network Load Balancer (NLB):** Operates at **Layer 4** (TCP/UDP). It handles millions of requests per second with ultra-low latency. It provides a **Static IP address** for each AZ.
* **Gateway Load Balancer (GWLB):** Operates at **Layer 3** (Network). Used specifically for deploying third-party virtual firewalls (like Palo Alto or Cisco) in AWS.

---

## 📈 4. Core Theory: Auto Scaling Groups (ASG)

### The Purpose of ASG
An Auto Scaling Group ensures you have the correct number of EC2 instances running to handle the load of your application, automatically scaling out (adding) or scaling in (removing) servers.

### Scaling Strategies
* **Target Tracking Scaling:** The easiest and most common. You set a target (e.g., "Keep average CPU at 50%"). The ASG adds or removes instances to hit that exact target.
* **Step Scaling:** Adds instances based on specific alarm thresholds (e.g., "If CPU > 70%, add 2 instances. If CPU > 90%, add 5 instances").
* **Scheduled Scaling:** Anticipates known traffic patterns (e.g., "Increase capacity to 10 instances every Friday at 5:00 PM").
* **Predictive Scaling:** Uses Machine Learning to analyze past traffic patterns and scales up before the traffic spike even happens.

---

## 🏗️ 5. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Run a stateless image rendering job that takes 4 hours and must be as cheap as possible. | **EC2 Spot Instances** |
| Run an enterprise database that will be online 24/7 for the next 3 years. | **EC2 Reserved Instances** |
| Route web traffic to a specific microservice based on the URL path (`/video`). | **Application Load Balancer (ALB)** |
| Expose an application to a client who requires a single, permanent IP address. | **Network Load Balancer (NLB)** |
| Automatically add more EC2 servers to a fleet when the average CPU hits 75%. | **Auto Scaling Group (Target Tracking)** |
| You have complex third-party security firewalls that all traffic must pass through. | **Gateway Load Balancer (GWLB)** |
| An application needs the absolute lowest network latency and millions of requests/sec. | **Network Load Balancer (NLB)** |

---

## 🤝 6. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is an Amazon Machine Image (AMI)?
**Answer:** A pre-configured template containing the OS and software needed to launch an EC2 instance.

**Question:** Can an AMI be shared across different AWS Regions directly?
**Answer:** No, AMIs are bound to a specific region. You must copy the AMI to another region to use it there.

**Question:** What is the most cost-effective EC2 purchasing option for a fault-tolerant batch job?
**Answer:** Spot Instances, as they offer the deepest discount for flexible workloads.

**Question:** What happens if AWS needs your Spot Instance capacity back?
**Answer:** AWS will give you a 2-minute warning (Spot Instance Interruption Notice) before terminating or stopping the instance.

**Question:** When must you use a Dedicated Host?
**Answer:** When you have strict compliance requirements or need to bring your own per-socket or per-core software licenses (BYOL).

**Question:** What is an Application Load Balancer (ALB)?
**Answer:** A Layer 7 load balancer that routes HTTP/HTTPS traffic based on advanced rules like path or host headers.

**Question:** What is a Network Load Balancer (NLB)?
**Answer:** A Layer 4 load balancer capable of handling millions of requests per second at ultra-low latency.

**Question:** Which load balancer provides a static Elastic IP address?
**Answer:** The Network Load Balancer (NLB).

**Question:** What is the purpose of an ELB Health Check?
**Answer:** To constantly monitor the status of registered instances and stop sending traffic to instances that are unhealthy or failing.

**Question:** What is an Auto Scaling Group (ASG)?
**Answer:** A service that automatically adjusts the number of EC2 instances in your application based on demand or rules.

**Question:** How does an ASG determine if an instance is healthy?
**Answer:** By utilizing EC2 status checks or integrating with the ELB health checks.

**Question:** What is ASG "Cooldown Period"?
**Answer:** A wait time after a scaling activity completes, preventing the ASG from launching or terminating additional instances too quickly.

**Question:** What is Target Tracking scaling in an ASG?
**Answer:** A scaling policy where you specify a metric target (e.g., 50% CPU), and the ASG automatically adjusts capacity to maintain that target.

**Question:** What is an EC2 Placement Group?
**Answer:** A configuration that lets you control how EC2 instances are placed on underlying AWS hardware.

**Question:** What is a Cluster Placement Group?
**Answer:** Packs instances close together inside a single AZ for the absolute lowest network latency (used for Big Data).

**Question:** What is a Spread Placement Group?
**Answer:** Spreads instances across distinct underlying hardware to minimize the risk of simultaneous hardware failure (max 7 instances per AZ).

**Question:** What is the AWS service used to store and manage startup scripts for EC2?
**Answer:** EC2 User Data, which runs exactly once when the instance first boots.

**Question:** How do you safely SSH into a private EC2 instance without opening port 22?
**Answer:** Use AWS Systems Manager (SSM) Session Manager.

**Question:** What is connection draining (deregistration delay) on a Load Balancer?
**Answer:** The time an ELB waits for in-flight requests to complete before fully deregistering or removing an instance.

**Question:** Can an ASG scale instances across different AWS Regions?
**Answer:** No, an ASG can only scale instances across multiple Availability Zones within a single Region.

---

## 🛠️ 7. Hands-on Labs (Test Your Skills)

**Lab 1:** Boot up a web server that automatically installs Apache and creates an "index.html" file upon its first launch.
<br><br><br>

**Lab 2:** Deploy an application that processes non-critical video files in the background at a 90% cost reduction.
<br><br><br>

**Lab 3:** Route traffic coming to `myapp.com/api` to one group of servers, and `myapp.com/images` to a completely different group of servers.
<br><br><br>

**Lab 4:** Guarantee that your application has a single, unchanging IP address that a third-party vendor can whitelist in their firewall.
<br><br><br>

**Lab 5:** Ensure that if a web server physically crashes, a new one is automatically created to replace it without human intervention.
<br><br><br>

**Lab 6:** Prevent an Auto Scaling Group from launching 100 new servers rapidly during a brief, 10-second CPU spike.
<br><br><br>

**Lab 7:** Deploy a high-performance database cluster where the servers must be physically located on the exact same server rack for microsecond latency.
<br><br><br>

**Lab 8:** Ensure that your critical backend servers are strictly kept on different hardware racks so a single power failure doesn't take them all down.
<br><br><br>

**Lab 9:** Stop an Application Load Balancer from sending user traffic to a server that has frozen or crashed.
<br><br><br>

**Lab 10:** Scale up your fleet of EC2 instances at exactly 8:00 AM every Monday in anticipation of heavy login traffic.
<br><br><br>

---

## 🔧 8. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Provide a bash script containing the Apache installation commands in the **EC2 User Data** field during the instance launch.

**Solution 2:** Launch the processing servers using **EC2 Spot Instances**, which provide deep discounts for flexible, non-critical workloads.

**Solution 3:** Deploy an **Application Load Balancer (ALB)** and configure "Path-Based Routing" rules in the listener to forward traffic to different Target Groups.

**Solution 4:** Provision a **Network Load Balancer (NLB)** and assign an Elastic IP address to its subnets during creation.

**Solution 5:** Create an **Auto Scaling Group (ASG)** and set both the "Minimum" and "Desired" capacity to 1. If the instance fails, the ASG will spin up a replacement to maintain the desired capacity of 1.

**Solution 6:** Configure an **ASG Cooldown Period** (e.g., 300 seconds). This forces the ASG to wait and see if the load stabilizes before making further scaling decisions.

**Solution 7:** Launch the database instances inside an **EC2 Cluster Placement Group** within a single Availability Zone.

**Solution 8:** Launch the critical instances inside an **EC2 Spread Placement Group**, which ensures they are on distinct hardware (max 7 per AZ).

**Solution 9:** Configure strict **ELB Health Checks** targeting a specific file (like `/health.html`). If the server stops responding 200 OK, the ELB automatically pulls it from the rotation.

**Solution 10:** Configure a **Scheduled Scaling Policy** on the Auto Scaling Group using a Cron expression to increase the desired capacity every Monday at 8:00 AM.

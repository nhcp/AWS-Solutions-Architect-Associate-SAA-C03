# SAA-C03 Chapter 1: Networking Foundations & Advanced VPC Design

## 📖 Introduction: What is this chapter about?
As a Solutions Architect, you aren't just "using" a network—you are **designing** it. This chapter covers how to create a secure, isolated environment for a company. We start with the basics (VPC/Subnets) and move to advanced "Design Decisions" like how to save money on data transfer and how to connect an office to the cloud secretly.

---

## 🧠 Part 1: The Basics (Everything you must know first)

### 1. The VPC (Your Private Island)
A **Virtual Private Cloud (VPC)** is a private network for your AWS account. It is logically isolated from other networks. 
* **CIDR Block:** The range of IP addresses for your network (e.g., `10.0.0.0/16`).

### 2. Subnets (Dividing the Island)
* **Public Subnet:** Has a "Route Table" that sends traffic to an **Internet Gateway (IGW)**. Use this for Web Servers.
* **Private Subnet:** No direct route to the internet. Use this for Databases.
* **The Rule:** An Availability Zone (AZ) can have many subnets, but a subnet can only be in **one** AZ.

### 3. Security (The Guards)
* **Security Group (SG):** A firewall for the **Instance**. It is **Stateful** (if you allow traffic in, it automatically goes out).
* **Network ACL (NACL):** A firewall for the **Subnet**. It is **Stateless** (you must explicitly allow traffic in AND out).

---

## 🏗️ Part 2: Advanced Architecture (The Solutions Architect Level)

### 1. NAT Gateway vs. NAT Instance
Private subnets often need to reach the internet for updates but shouldn't be reached from the outside.
* **NAT Gateway:** A managed AWS service. Highly available and scales automatically. **(Always choose this for the exam).**
* **NAT Instance:** A manual EC2 instance you manage yourself. Not recommended for production.

### 2. VPC Endpoints (The Private Tunnels)
Instead of going over the public internet to reach S3 or DynamoDB, you stay inside the AWS network.
* **Gateway Endpoints:** Free tunnels for S3 and DynamoDB.
* **Interface Endpoints (PrivateLink):** Paid tunnels for everything else.

### 3. Connecting Networks
* **VPC Peering:** Connects two VPCs. Not transitive (A->B, B->C doesn't mean A->C).
* **Transit Gateway:** A regional hub that connects thousands of VPCs and on-premises networks.
* **Direct Connect (DX):** A physical fiber line from your office to AWS. High speed, low latency.

---

## 🎙️ Part 3: Top 10 SAA-C03 Master Scenarios

| # | Scenario | Architect Solution |
| :--- | :--- | :--- |
| 1 | Connect 1000 VPCs together easily? | **Transit Gateway** |
| 2 | Talk to S3 privately for $0? | **Gateway Endpoint** |
| 3 | Static IP for a Load Balancer? | **Network Load Balancer (NLB)** |
| 4 | Block one single "bad" IP address? | **Network ACL (NACL)** |
| 5 | Office to AWS (Consistent Speed)? | **Direct Connect** |
| 6 | Private server needs internet updates? | **NAT Gateway** |
| 7 | Connect 2 VPCs with overlapping IPs? | **AWS PrivateLink** |
| 8 | Monitor all network traffic packets? | **VPC Flow Logs** |
| 9 | Route users to the nearest server? | **Route 53 Latency Policy** |
| 10 | Connect office to AWS in 30 mins? | **Site-to-Site VPN** |

---

## 🛠️ Part 4: 10 Hands-on Lab Tasks (Practice & Think)

*Try to solve these in your head first before checking the instructions at the bottom.*

**Lab 1:** Create a "Database Subnet" that can reach the internet for patches but is hidden from hackers.
<br><br><br><br><br><br><br><br><br><br>

**Lab 2:** You move 100TB to S3 monthly. How do you stop paying NAT Gateway data fees for this?
<br><br><br><br><br><br><br><br><br><br>

**Lab 3:** Why does VPC Peering fail when trying to connect VPC A to VPC C through VPC B?
<br><br><br><br><br><br><br><br><br><br>

**Lab 4:** Set up a permanent, non-changing IP for your application entry point.
<br><br><br><br><br><br><br><br><br><br>

**Lab 5:** Block the IP `1.2.3.4` from even reaching your subnet gateway.
<br><br><br><br><br><br><br><br><br><br>

**Lab 6:** Connect two VPCs that belong to different companies without using peering.
<br><br><br><br><br><br><br><br><br><br>

**Lab 7:** Ensure that an EC2 instance in a private subnet can be managed via SSH without a Bastion Host.
<br><br><br><br><br><br><br><br><br><br>

**Lab 8:** Resolve a "Connection Timeout" for an instance in a public subnet.
<br><br><br><br><br><br><br><br><br><br>

**Lab 9:** Enable logging to see which Security Group rules are being triggered.
<br><br><br><br><br><br><br><br><br><br>

**Lab 10:** Design a high-speed link to an office that works even if the local ISP goes down.
<br><br><br><br><br><br><br><br><br><br>

---

## 🤝 Part 5: 20 Interview Questions & Professional Answers

1. **What is a VPC?**
   * *Answer:* A logically isolated section of the AWS cloud where you define your own IP address range, subnets, and route tables.
2. **Explain the difference between a Public and Private Subnet.**
   * *Answer:* A public subnet has a route to an Internet Gateway; a private subnet does not.
3. **What are the reserved IP addresses in an AWS subnet?**
   * *Answer:* AWS reserves 5 IPs: the first four (.0, .1, .2, .3) and the last one (.255).
4. **Is a Security Group stateful or stateless?**
   * *Answer:* Stateful. It automatically allows return traffic.
5. **How is a NACL different from a Security Group?**
   * *Answer:* NACLs are stateless, operate at the subnet level, and can explicitly "Deny" traffic.
6. **When would you use a NAT Gateway?**
   * *Answer:* When instances in a private subnet need to initiate outbound traffic (like updates) without being exposed to the internet.
7. **What is VPC Peering?**
   * *Answer:* A networking connection between two VPCs that allows them to communicate using private IP addresses.
8. **What does "Non-Transitive Peering" mean?**
   * *Answer:* If VPC A is peered with B, and B with C, A cannot talk to C through B. You need a direct peer or Transit Gateway.
9. **What is AWS Transit Gateway?**
   * *Answer:* A network transit hub that acts as a regional router to connect thousands of VPCs and on-premises networks.
10. **Explain the difference between Interface and Gateway Endpoints.**
    * *Answer:* Gateway endpoints are free and only for S3/DynamoDB. Interface endpoints use PrivateLink and are for most other services.
11. **How do you troubleshoot an EC2 that can't reach the internet?**
    * *Answer:* Check the IGW attachment, the Route Table for 0.0.0.0/0, the Security Group rules, and the NACL.
12. **What is AWS Direct Connect?**
    * *Answer:* A dedicated physical network connection from an on-premises data center to AWS, bypassing the public internet.
13. **Why use a Site-to-Site VPN over Direct Connect?**
    * *Answer:* VPN is much faster to set up (minutes vs weeks) and is cheaper, though less stable/fast.
14. **What is VPC Flow Logs?**
    * *Answer:* A feature that captures IP traffic information going to and from network interfaces in your VPC.
15. **How can you share a service with 1000 other VPCs securely?**
    * *Answer:* Use AWS PrivateLink (VPC Endpoint Service).
16. **What is an Internet Gateway (IGW)?**
    * *Answer:* A horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.
17. **What is an Egress-Only Internet Gateway?**
    * *Answer:* Used for IPv6 traffic only. It allows outbound traffic while preventing inbound connections.
18. **Can you change the CIDR block of an existing VPC?**
    * *Answer:* No, but you can add secondary CIDR blocks to an existing VPC.
19. **What is the purpose of a Route Table?**
    * *Answer:* It contains a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed.
20. **What is AWS Global Accelerator?**
    * *Answer:* A service that uses the AWS global network to improve the performance and availability of your applications for users worldwide.

---

## 🔧 Part 6: Lab Steps (How to do it)

**Lab 1 (Private Updates):** Create NAT Gateway in Public Subnet -> Edit Private Route Table -> Add route `0.0.0.0/0` targeting the NAT Gateway.
**Lab 2 (S3 Tunnel):** VPC -> Endpoints -> Create -> Select `s3` (Gateway) -> Select Private Route Table.
**Lab 4 (Static IP):** Create Network Load Balancer (NLB) -> Assign an Elastic IP during the setup.
**Lab 5 (Block IP):** VPC -> Network ACLs -> Inbound Rules -> Add Rule (Number 50, Source `1.2.3.4/32`, Action: DENY).
**Lab 7 (No Bastion):** Enable AWS Systems Manager (SSM) on the instance and use "Session Manager" to connect.

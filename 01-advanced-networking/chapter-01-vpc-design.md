# SAA-C03 Chapter 1: Networking & Advanced VPC Architecture

## 📖 1. Introduction
This chapter covers the "Connectivity" domain of the SAA-C03 exam. As an architect, you must design networks that are secure (isolation), performant (low latency), and cost-effective. We build from simple IP basics up to complex global routing.

---

## 🧠 2. Basic Theory (The Foundation)

### What is a VPC? (Virtual Private Cloud)
A VPC is your own **private data center** in the AWS cloud. 
* **Scope:** A VPC is a **Regional** service. It covers all Availability Zones in a region.
* **CIDR Block:** The IP range for your network (e.g., `10.0.0.0/16`).
* **Reserved IPs:** AWS always reserves **5 IP addresses** in every subnet (.0, .1, .2, .3, and .255).

### Subnets: Public vs. Private
* **Public Subnet:** Has a Route Table entry pointing to an **Internet Gateway (IGW)**. This is where your Load Balancers and Web Servers live.
* **Private Subnet:** Has no direct route to the internet. This is where your Databases live.
* **Placement:** A subnet must reside within **one** Availability Zone (AZ).


### Basic Security (The Guards)
* **Security Group (SG):** A virtual firewall for the **Instance**. It is **Stateful** (it remembers traffic).
* **Network ACL (NACL):** A firewall for the **Subnet**. It is **Stateless** (it does not remember traffic).

---

## 🏗️ 3. Advanced Theory (Architect Level)

### Connectivity Logic: NAT vs. Endpoints
How do private instances talk to the world without being public?
* **NAT Gateway:** A managed service that lets private instances go "out" to the internet (for updates) but prevents hackers from coming "in."
* **VPC Endpoints:** Private "tunnels" to AWS services so traffic stays on the AWS network.
    * **Gateway Endpoints:** Free; used for **S3** and **DynamoDB**.
    * **Interface Endpoints:** Paid; used for everything else (Kinesis, SNS, etc.) via **PrivateLink**.

### Global Connections
* **VPC Peering:** A direct bridge between two VPCs. It is **not transitive**.
* **Transit Gateway:** A central hub (regional router) to connect thousands of VPCs.
* **Direct Connect (DX):** A physical fiber line from your office to AWS. Bypasses the internet.
* **Global Accelerator:** Uses Anycast IPs and the AWS global network to speed up traffic for users worldwide.


---

## 🏗️ 4. Architecture Scenarios (The Exam Brain)

| # | Business Requirement | Architect's Design Solution |
| :--- | :--- | :--- |
| 1 | Connect 1,000 VPCs together with minimal management. | **AWS Transit Gateway** |
| 2 | Move Petabytes of data to S3 without paying NAT data fees. | **S3 Gateway Endpoint** |
| 3 | Static IP address required for a Load Balancer. | **Network Load Balancer (NLB)** |
| 4 | Block a specific malicious IP address from a subnet. | **Network ACL (NACL)** |
| 5 | High-speed link from office to AWS bypassing the internet. | **AWS Direct Connect** |
| 6 | Connect two companies with overlapping CIDR blocks. | **AWS PrivateLink** |
| 7 | Securely manage private instances without a Bastion Host. | **AWS Systems Manager (SSM)** |
| 8 | Monitor every packet entering the VPC for auditing. | **VPC Flow Logs** |
| 9 | Route users to the server with the lowest latency. | **Route 53 Latency Policy** |
| 10 | Encrypted tunnel to AWS over the internet in 30 minutes. | **Site-to-Site VPN** |

---

## 🤝 5. 20 Priority Interview Q&A (Strict Spacing)

1. **Question:** What is a VPC?
**Answer:** A logically isolated virtual network within an AWS Region.

2. **Question:** What is a Subnet?
**Answer:** A range of IP addresses in your VPC restricted to a single Availability Zone.

3. **Question:** What is the difference between a Public and a Private subnet?
**Answer:** A public subnet has a route to an Internet Gateway; a private subnet does not.

4. **Question:** How many IP addresses does AWS reserve in each subnet?
**Answer:** Five addresses (.0, .1, .2, .3, and .255).

5. **Question:** Is a Security Group stateful or stateless?
**Answer:** Stateful; it automatically allows return traffic for any allowed request.

6. **Question:** Is a Network ACL stateful or stateless?
**Answer:** Stateless; you must explicitly define both inbound and outbound rules.

7. **Question:** When would you use a NAT Gateway?
**Answer:** To allow instances in a private subnet to initiate outbound internet traffic safely.

8. **Question:** What is a Gateway Endpoint?
**Answer:** A free, private tunnel connecting your VPC directly to S3 or DynamoDB.

9. **Question:** What is an Interface Endpoint?
**Answer:** A paid private connection for most AWS services using AWS PrivateLink.

10. **Question:** What is VPC Peering?
**Answer:** A 1-to-1 private network connection between two VPCs.

11. **Question:** Can VPC Peering be transitive?
**Answer:** No; you cannot use one VPC as a "hop" to reach a third VPC.

12. **Question:** What is AWS Transit Gateway?
**Answer:** A regional hub that simplifies connectivity between thousands of VPCs.

13. **Question:** What is AWS Direct Connect?
**Answer:** A physical, private fiber connection from your data center to an AWS location.

14. **Question:** What is the difference between a VPN and Direct Connect?
**Answer:** VPN is fast to set up over the internet; DX is a private fiber line.

15. **Question:** What is VPC Flow Logs?
**Answer:** A feature that captures metadata about the IP traffic entering and leaving your network.

16. **Question:** How do you block a specific IP address?
**Answer:** By adding an Inbound Deny rule to your Network ACL (NACL).

17. **Question:** What is an Internet Gateway (IGW)?
**Answer:** A scaled, redundant component that allows communication between the VPC and the internet.

18. **Question:** What is an Egress-Only Internet Gateway?
**Answer:** An IPv6-only component that allows outbound traffic but blocks inbound connections.

19. **Question:** What is a Route Table?
**Answer:** A set of rules (routes) used to determine where network traffic is directed.

20. **Question:** What is AWS Global Accelerator?
**Answer:** A service that uses Anycast IPs to improve performance via the AWS global network.

---

## 🛠️ 6. Hands-on Lab Tasks (Practice Section)

**Lab 1: The Private Update Design**
Task: Design a network where a database can get updates but is totally hidden from the internet.
<br><br><br><br><br><br><br><br><br><br>

**Lab 2: The Data Transfer Cost Fix**
Task: You move 100TB to S3 monthly. Your NAT Gateway cost is too high. How do you fix it?
<br><br><br><br><br><br><br><br><br><br>

---

## 🔧 7. Lab Solutions & Step-by-Step Instructions

### Solution 1 (Private Updates):
* **Logic:** Use a Private Subnet for the DB and a NAT Gateway in the Public Subnet for the exit.
* **Steps:** 1. Create NAT Gateway in Public Subnet. 
2. Edit Private Route Table. 
3. Add Route `0.0.0.0/0` targeting the NAT Gateway.

### Solution 2 (S3 Cost):
* **Logic:** Use an S3 Gateway Endpoint (Free).
* **Steps:** 1. Go to VPC Dashboard -> Endpoints. 
2. Create S3 Gateway Endpoint. 
3. Associate with Private Route Tables.

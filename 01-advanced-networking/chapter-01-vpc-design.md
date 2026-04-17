# SAA-C03 Chapter 1: Networking & Advanced VPC Architecture

---

## 🧠 1. Foundation Theory (The Basics)

**What is a VPC?**
A Virtual Private Cloud (VPC) is your own private network in the cloud. It is isolated from other users and belongs to one specific AWS Region.

**Subnets: Public vs. Private**
* **Public Subnet:** Has a connection to the internet via an Internet Gateway. Use this for Load Balancers.
* **Private Subnet:** No direct internet connection. Use this for Databases to keep them hidden.

**Security: SGs and NACLs**
* **Security Group (SG):** A firewall for the server. It remembers who you let in (Stateful).
* **Network ACL (NACL):** A firewall for the whole subnet. It does not remember traffic (Stateless).

---

## 🏗️ 2. Advanced Theory (Architect Level)

**NAT Gateway**
Lives in the Public Subnet. Allows private servers to go "out" for updates but blocks incoming traffic.

**Gateway Endpoints**
Free tunnels for S3 and DynamoDB that keep traffic inside the AWS network.

**Interface Endpoints**
Paid tunnels for almost all other services using PrivateLink technology.

**Transit Gateway**
A central hub to connect thousands of VPCs and offices together in a "hub-and-spoke" model.

---

## 🏗️ 3. Architecture Scenarios

| # | Business Requirement | Architect Solution |
| :--- | :--- | :--- |
| 1 | Connect 1,000 VPCs easily. | **Transit Gateway** |
| 2 | Move S3 data without NAT fees. | **S3 Gateway Endpoint** |
| 3 | Static IP for Load Balancer. | **Network Load Balancer (NLB)** |
| 4 | Block a specific bad IP address. | **Network ACL (NACL)** |
| 5 | Fiber link bypassing internet. | **AWS Direct Connect** |
| 6 | Connect overlapping CIDR blocks. | **AWS PrivateLink** |
| 7 | Manage instances without Bastion. | **Systems Manager (SSM)** |
| 8 | Audit every packet entering VPC. | **VPC Flow Logs** |
| 9 | Route users to fastest server. | **Route 53 Latency Policy** |
| 10 | Fast encrypted tunnel to AWS. | **Site-to-Site VPN** |

---

## 🤝 4. 20 Priority Interview Q&A (Spaced for Study)

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
**Answer:** To allow instances in a private subnet to initiate outbound internet traffic.

8. **Question:** What is a Gateway Endpoint?
**Answer:** A free, private tunnel that connects your VPC directly to S3 or DynamoDB.

9. **Question:** What is an Interface Endpoint?
**Answer:** A paid private connection for most AWS services using AWS PrivateLink technology.

10. **Question:** What is VPC Peering?
**Answer:** A 1-to-1 private network connection between two VPCs.

11. **Question:** Can VPC Peering be transitive?
**Answer:** No; you cannot use one VPC as a "hop" to reach a third VPC.

12. **Question:** What is AWS Transit Gateway?
**Answer:** A regional hub that simplifies connectivity between thousands of VPCs.

13. **Question:** What is AWS Direct Connect?
**Answer:** A physical, private fiber connection from your data center to an AWS location.

14. **Question:** What is the difference between a VPN and Direct Connect?
**Answer:** VPN is fast over the internet; DX is a slow private fiber line.

15. **Question:** What is VPC Flow Logs?
**Answer:** A feature that captures metadata about the IP traffic entering and leaving your network.

16. **Question:** How do you block a specific IP address?
**Answer:** By adding an Inbound Deny rule to your Network ACL (NACL).

17. **Question:** What is an Internet Gateway (IGW)?
**Answer:** A horizontally scaled component that allows your VPC to talk to the internet.

18. **Question:** What is an Egress-Only Internet Gateway?
**Answer:** An IPv6-only component that allows outbound traffic but blocks inbound connections.

19. **Question:** What is a Route Table?
**Answer:** A set of rules (routes) used to determine where network traffic is directed.

20. **Question:** What is AWS Global Accelerator?
**Answer:** A service that uses Anycast IPs to improve performance via the AWS global network.

---

## 🛠️ 5. 10 Hands-on Lab Tasks

**Lab 1:** Design a network where a database can get updates but is totally hidden from the internet.
<br><br><br><br><br>

**Lab 2:** You move 100TB to S3 monthly. Your NAT Gateway cost is too high. How do you fix it?
<br><br><br><br><br>

**Lab 3:** You have 3 VPCs (A, B, C). A is peered to B, B to C. Why can't A talk to C?
<br><br><br><br><br>

**Lab 4:** A client needs one single permanent IP to whitelist for your web app.
<br><br><br><br><br>

**Lab 5:** You need to block exactly ONE IP address: `1.2.3.4`.
<br><br><br><br><br>

**Lab 6:** Connect VPCs in different regions to a single office using one router.
<br><br><br><br><br>

**Lab 7:** Two VPCs both use `10.0.0.0/16`. How can they share a service?
<br><br><br><br><br>

**Lab 8:** Connect your home office to your VPC privately in under 30 minutes.
<br><br><br><br><br>

**Lab 9:** Record the source and destination of all VPC traffic for an audit.
<br><br><br><br><br>

**Lab 10:** Improve speed for global users talking to a single region server.
<br><br><br><br><br>

---

## 🔧 6. Lab Solutions & Step-by-Step Instructions

**Solution 1:** Create NAT Gateway in Public Subnet. Add Route `0.0.0.0/0` targeting NAT Gateway in the Private Route Table.

**Solution 2:** Create an S3 Gateway Endpoint and associate it with your Private Route Tables.

**Solution 3:** Create a direct peering connection between A and C, or migrate to Transit Gateway.

**Solution 4:** Create a Network Load Balancer (NLB) and assign an Elastic IP during setup.

**Solution 5:** Go to NACL -> Inbound Rules -> Add Rule #50 -> Source `1.2.3.4/32` -> DENY.

**Solution 6:** Use AWS Transit Gateway with Inter-Region Peering.

**Solution 7:** Use AWS PrivateLink (Interface Endpoints).

**Solution 8:** Set up an AWS Site-to-Site VPN.

**Solution 9:** Enable VPC Flow Logs and send them to an S3 bucket.

**Solution 10:** Use AWS Global Accelerator.

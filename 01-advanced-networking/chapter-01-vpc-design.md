# SAA-C03 Chapter 1: Networking & Advanced VPC Architecture

## 📖 1. Introduction
This chapter focuses on the **Security, Isolation, and Connectivity** domains of the Solutions Architect Associate exam. As an architect, you must design a network that is "Default Deny," meaning nothing enters or leaves unless specifically allowed. We will build from the simple "Island" metaphor up to complex global connectivity.

---

## 🧠 2. The Foundation: Core Networking Concepts

### What is a VPC? (Virtual Private Cloud)
A VPC is your own private data center in the AWS cloud. It is a **Regional service**, meaning it stays within one AWS Region but spans across all Availability Zones in that region.

### The Anatomy of an IP (CIDR)
* **CIDR Block:** You define the size of your network (e.g., `10.0.0.0/16` gives you 65,536 IPs).
* **Subnetting:** You break the VPC into smaller pieces. **Crucial Rule:** One subnet = One Availability Zone.
* **Reserved IPs:** AWS always takes 5 IP addresses per subnet (.0, .1, .2, .3, and .255). You cannot use these.

### Public vs. Private Tiers
* **Public Subnet:** Contains an entry in its Route Table pointing to an **Internet Gateway (IGW)**. This allows the "World" to see the resources (like a Load Balancer).
* **Private Subnet:** Does NOT have a route to the IGW. It is invisible to the outside.
* **NAT Gateway:** To let a private instance go "Out" to the internet (for updates) without letting anyone "In," you place a NAT Gateway in the **Public** subnet.



---

## 🏗️ 3. Advanced Design: Connectivity & Security

### VPC Endpoints (The Private Link)
When your app talks to S3 or DynamoDB, you don't want traffic to leave the AWS network.
* **Gateway Endpoints:** Free, virtual routes for S3 and DynamoDB only.
* **Interface Endpoints:** Powered by **AWS PrivateLink**. These are paid ENIs (Elastic Network Interfaces) for all other services.

### Global Connectivity
* **VPC Peering:** A direct 1-to-1 connection between two VPCs.
* **AWS Transit Gateway:** A central hub to connect thousands of VPCs. It is the "Regional Router."
* **Direct Connect:** A physical, private fiber line from your building to AWS. No internet involved.

### Firewalls: SG vs. NACL
| Feature | Security Group (SG) | Network ACL (NACL) |
| :--- | :--- | :--- |
| **Level** | Instance Level | Subnet Level |
| **Type** | Stateful (Returns traffic automatically) | Stateless (Explicitly allow In/Out) |
| **Rules** | Allow only | Allow and **Deny** |

---

## 🎙️ 4. Master Scenario-Based Strategy

* **Scenario:** Connect 1,000 VPCs? -> **Transit Gateway.**
* **Scenario:** High-speed fiber with no internet? -> **Direct Connect.**
* **Scenario:** Encrypted tunnel over the web in 10 mins? -> **Site-to-Site VPN.**
* **Scenario:** Database needs internet but must be hidden? -> **NAT Gateway.**
* **Scenario:** Static IP for a Load Balancer? -> **Network Load Balancer (NLB).**
* **Scenario:** Block a specific IP address? -> **Network ACL (NACL).**

---

## 🛠️ 5. Hands-on Lab Tasks (Practice Section)

*Instruction: Use the space below to design your solution before looking at the steps.*

### Lab 1: High-Availability Private Updates
Design a network where instances in a Private Subnet can reach the internet across 2 different Availability Zones even if one NAT Gateway fails.
<br><br><br><br><br><br><br><br><br><br>

### Lab 2: Zero-Cost S3 Transfers
You are moving Petabytes of data from EC2 to S3. Your NAT Gateway bill is exploding. How do you eliminate the data processing cost?
<br><br><br><br><br><br><br><br><br><br>

### Lab 3: Overlapping CIDR Fix
You need to connect two VPCs that both use `10.0.0.0/16`. Peering is impossible. How do you share services between them?
<br><br><br><br><br><br><br><br><br><br>

### Lab 4: Permanent IP Whitelisting
A client firewall only accepts one IP. Your architecture uses 10 Web Servers behind an ALB. How do you provide a single static IP?
<br><br><br><br><br><br><br><br><br><br>

---

## 🤝 6. 20 Priority Interview Questions & Answers

1. **What is a VPC?** A logically isolated section of the AWS Cloud where you have full control over your networking environment.
2. **Explain the difference between a Security Group and a NACL.** SG is stateful/instance-level; NACL is stateless/subnet-level and can deny traffic.
3. **What is a NAT Gateway?** A managed service that allows instances in a private subnet to connect to the internet but prevents the internet from connecting to them.
4. **When do you use a Gateway Endpoint?** For private, free connectivity to S3 and DynamoDB.
5. **What is AWS Transit Gateway?** A hub-and-spoke router used to simplify connectivity between thousands of VPCs and on-premise networks.
6. **Explain 'VPC Peering' limitations.** It is non-transitive. If A peers with B, and B peers with C, A cannot talk to C.
7. **What is 'PrivateLink'?** A technology that allows you to share services between VPCs privately without using the public internet or peering.
8. **What is Direct Connect?** A physical, dedicated network connection from an on-premise data center to AWS.
9. **How many IP addresses does AWS reserve in a subnet?** Five (.0, .1, .2, .3, and .255).
10. **What is an Internet Gateway?** A horizontally scaled, redundant, and highly available VPC component that allows communication between the VPC and the internet.
11. **Difference between Latency and Geolocation routing?** Latency routes to the fastest region; Geolocation routes based on the user's physical location.
12. **What are VPC Flow Logs?** A feature that captures IP traffic information for network interfaces in your VPC for security and troubleshooting.
13. **How do you connect overlapping VPCs?** Using AWS PrivateLink.
14. **What is an Egress-Only Internet Gateway?** A gateway that allows outbound IPv6 traffic only.
15. **What is the benefit of an NLB?** It supports static IP addresses and handles millions of requests with ultra-low latency.
16. **What is a 'Bastion Host'?** A public-facing server used as a bridge to reach private-subnet servers via SSH/RDP.
17. **What is the AWS Global Accelerator?** A service that uses Anycast IPs and the AWS private backbone to speed up user traffic globally.
18. **Can a VPC span multiple Regions?** No. A VPC is a Regional service.
19. **What is a 'Route Table'?** A set of rules (routes) used to determine where network traffic is directed.
20. **What is a 'Virtual Private Gateway'?** The VPN concentrator on the AWS side of a Site-to-Site VPN connection.

---

## 🔧 7. Lab Steps (The Master Solutions)

### Solution 1 (Multi-AZ NAT):
1. Deploy two NAT Gateways (one in Public Subnet AZ1, one in Public Subnet AZ2).
2. Create two Private Route Tables.
3. Route Table 1 -> `0.0.0.0/0` to NAT 1.
4. Route Table 2 -> `0.0.0.0/0` to NAT 2.

### Solution 2 (S3 Cost Fix):
1. Go to VPC Console -> Endpoints.
2. Create Endpoint -> Service: `com.amazonaws.[region].s3` (Type: Gateway).
3. Associate with your Private Subnet Route Tables.

### Solution 3 (Overlapping IPs):
1. In the "Provider" VPC, create a Network Load Balancer (NLB) for your service.
2. Create a **VPC Endpoint Service** and point it to the NLB.
3. In the "Consumer" VPC, create an **Interface Endpoint** pointing to that service.

### Solution 4 (Static IP):
1. Create a **Network Load Balancer (NLB)**.
2. During setup, assign an **Elastic IP** to the NLB for each Availability Zone.

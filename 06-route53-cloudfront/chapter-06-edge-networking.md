# SAA-C03 Chapter 6: Route 53 & Global Content Delivery

---

## 📖 1. The Big Picture: Global Performance & DNS
**The Objective:** To route users to your application reliably, survive regional disasters, and deliver content globally with the absolute lowest latency possible.
**The Problem:** If your servers are in Virginia (US-East-1) and a customer is in Tokyo, their network request has to cross the Pacific Ocean, resulting in terrible lag. Furthermore, if the Virginia data center goes offline, how do your customers automatically find your backup servers in London?
**The Solution:** We push the network to the "Edge." We use **Amazon Route 53** as our highly available DNS system to route traffic smartly (e.g., sending users to the server physically closest to them). We use **Amazon CloudFront** to cache static assets (like images and videos) in edge locations around the world. For non-HTTP applications, we use **AWS Global Accelerator**.
**The Trade-offs:** Caching data globally means dealing with "cache invalidation" (if you update an image, old versions might still be cached for hours). Global routing architectures also require maintaining infrastructure in multiple regions, which increases costs.

---

## 🧠 2. Core Theory: Amazon Route 53 (DNS)

### What is Route 53?
Route 53 is AWS's highly available, scalable Domain Name System (DNS) web service. It translates human-readable names (like `google.com`) into IP addresses (like `192.0.2.1`).
* **Hosted Zones:** A container for records that define how to route traffic for a domain.
    * **Public Hosted Zone:** Determines how traffic is routed on the public internet.
    * **Private Hosted Zone:** Determines how traffic is routed *within* your VPCs (e.g., `my-internal-db.local`).

### CNAME vs. Alias Records (Critical Exam Topic)
* **CNAME:** Maps a hostname to another hostname (e.g., `www.example.com` to `lb.amazonaws.com`). **Cannot** be used for the root domain (the "Zone Apex" like `example.com`).
* **Alias Record:** An AWS-specific extension. Maps a hostname to an AWS resource (like an ALB, CloudFront, or S3 bucket). **Can** be used for the root domain. Always use Alias records over CNAMEs in AWS because they are free and natively track AWS IP changes.

### Route 53 Routing Policies
* **Simple:** Standard DNS routing. Returns one or more IPs. No health checks.
* **Weighted:** Distributes traffic across multiple resources based on proportions (e.g., 80% to Server A, 20% to Server B). Great for A/B testing or gradual rollouts.
* **Latency:** Routes users to the AWS Region that provides the lowest network latency.
* **Failover:** Configures Active/Passive disaster recovery. If the primary resource fails a health check, traffic is routed to a secondary backup resource.
* **Geolocation:** Routes traffic based on the physical location of your users (e.g., "Send all users in Germany to the Frankfurt region"). Used for compliance and language localization.
* **Geoproximity:** Routes traffic based on the geographic location of your users *and* your resources, allowing you to shift traffic by creating "biases". Requires Route 53 Traffic Flow.
* **Multi-Value Answer:** Like Simple routing but with health checks. Returns multiple healthy IP addresses to the client, providing basic client-side load balancing.



---

## ⚡ 3. Core Theory: Amazon CloudFront (CDN)

### What is CloudFront?
CloudFront is a Content Delivery Network (CDN). It caches your static and dynamic web content closer to your users via a global network of "Edge Locations."
* **Origin:** The source of your files (usually an S3 Bucket, an Application Load Balancer, or an EC2 instance).
* **Edge Location:** Data centers scattered globally where CloudFront caches copies of your files.
* **TTL (Time to Live):** How long a file stays in the cache before CloudFront asks the Origin for a fresh copy.

### Securing CloudFront and S3
If you use S3 as an origin, you don't want users bypassing CloudFront and accessing the S3 bucket directly via its public URL.
* **OAC (Origin Access Control):** The modern, secure way to lock down an S3 bucket so it *only* allows access from your specific CloudFront distribution. (Replaces the older OAI - Origin Access Identity).
* **Geo-Restriction:** You can block entire countries from viewing your CloudFront distribution (e.g., blocking embargoed countries).



---

## 🚀 4. Core Theory: AWS Global Accelerator

### What is Global Accelerator?
A networking service that improves the performance of your users' traffic by up to 60% using the AWS global network backbone.
* **Anycast IP:** It provides you with **two static Anycast IP addresses**. These IPs act as a fixed entry point to your application worldwide.
* **How it works:** A user in India connects to your Anycast IP at the nearest Indian Edge Location. From there, the traffic jumps onto the private, dedicated AWS fiber backbone to reach your server in the USA, completely bypassing the congested, public internet.
* **CloudFront vs. Global Accelerator:**
    * **CloudFront:** Best for caching HTTP/HTTPS traffic (images, videos, web pages) at the edge.
    * **Global Accelerator:** Best for non-HTTP traffic (UDP, TCP gaming servers, IoT), applications that require fixed IP addresses, and applications requiring instant regional failover without relying on DNS caches.

---

## 🏗️ 5. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Point the root domain `myapp.com` to an Application Load Balancer. | **Route 53 Alias Record** |
| Roll out a new application version to only 10% of your users for testing. | **Route 53 Weighted Routing** |
| Ensure European customers hit European servers for GDPR compliance. | **Route 53 Geolocation Routing** |
| Automatically failover to a backup website in S3 if your primary EC2 fleet dies. | **Route 53 Failover Routing + Health Checks** |
| Distribute global 4K video files with the lowest possible latency to users. | **Amazon CloudFront** |
| Prevent users from accessing an S3 bucket directly; force them through a CDN. | **CloudFront with Origin Access Control (OAC)** |
| Provide global users low-latency access to a UDP-based multiplayer game server. | **AWS Global Accelerator** |
| A third-party firewall requires you to provide two static IPs for global routing. | **AWS Global Accelerator** |

---

## 🤝 6. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is a Domain Name System (DNS)?
**Answer:** The phonebook of the internet; it translates human-readable domain names into IP addresses.

**Question:** What is the difference between a Public and a Private Hosted Zone in Route 53?
**Answer:** A public zone routes internet traffic; a private zone routes traffic strictly within your VPCs.

**Question:** Why should you use an Alias Record instead of a CNAME in AWS?
**Answer:** Alias records are free, natively track IP changes of AWS resources like ALBs, and can be used on the root domain (Zone Apex).

**Question:** What is Route 53 Latency Routing?
**Answer:** It routes users to the AWS Region that provides them with the lowest network latency.

**Question:** What is Route 53 Geolocation Routing?
**Answer:** It routes traffic based on the physical location of the user (e.g., country or state).

**Question:** How does Route 53 Failover Routing work?
**Answer:** It uses Health Checks to monitor a primary resource; if it fails, Route 53 automatically directs traffic to a passive secondary resource.

**Question:** What is Amazon CloudFront?
**Answer:** A global Content Delivery Network (CDN) that caches data at Edge Locations to reduce latency for end-users.

**Question:** What is an Edge Location?
**Answer:** A data center in the AWS global network used specifically to cache CloudFront content closer to users.

**Question:** What is a CloudFront Origin?
**Answer:** The original source of the files being distributed, such as an S3 Bucket or an Application Load Balancer.

**Question:** What is Time to Live (TTL) in CloudFront?
**Answer:** The amount of time an object is kept in the Edge Location cache before CloudFront checks the origin for an updated version.

**Question:** How do you forcefully remove an old image from CloudFront before its TTL expires?
**Answer:** By creating a CloudFront Invalidation (Note: this costs money if done frequently).

**Question:** How do you secure an S3 bucket so it can only be read by CloudFront?
**Answer:** By configuring Origin Access Control (OAC) on CloudFront and updating the S3 Bucket Policy to allow access only from that OAC.

**Question:** What is CloudFront Geo-Restriction?
**Answer:** A feature that allows you to block or allow access to your content based on the geographic location of your viewers.

**Question:** Can CloudFront protect against DDoS attacks?
**Answer:** Yes, CloudFront integrates seamlessly with AWS Shield and AWS WAF to block malicious traffic at the edge.

**Question:** What is AWS Global Accelerator?
**Answer:** A networking service that routes user traffic through the AWS global network backbone to improve performance and reliability.

**Question:** How many IP addresses does Global Accelerator provide?
**Answer:** It provides two static Anycast IP addresses.

**Question:** What is an Anycast IP?
**Answer:** A single IP address that is routed to the nearest physical edge location, rather than a single specific server.

**Question:** When would you choose Global Accelerator over CloudFront?
**Answer:** When your application uses non-HTTP/HTTPS protocols (like TCP/UDP), or when you need rapid regional failover independent of DNS caching.

**Question:** How does Global Accelerator handle failover compared to Route 53?
**Answer:** Global Accelerator handles failover almost instantly because it doesn't rely on client-side DNS caches expiring.

**Question:** Can you use Route 53 to buy domain names?
**Answer:** Yes, Route 53 acts as a Domain Registrar as well as a DNS hosting provider.

---

## 🛠️ 7. Hands-on Labs (Test Your Skills)

**Lab 1:** You purchased `mycompany.com`. You deployed an Application Load Balancer. How do you link the root domain to the ALB?
<br><br><br>

**Lab 2:** You are releasing a massive UI update, but you only want 5% of your global users to see it so you can monitor for bugs.
<br><br><br>

**Lab 3:** You have an active primary application in `us-east-1` and a passive static "maintenance page" in an S3 bucket. If the primary app crashes, you want the maintenance page to display automatically.
<br><br><br>

**Lab 4:** You want to ensure that users in France are routed to your web server in Paris, while users in Germany are routed to Frankfurt, specifically for legal compliance.
<br><br><br>

**Lab 5:** Your company has internal EC2 instances that need to communicate with a database using a friendly URL like `database.internal.corp`, without exposing this URL to the internet.
<br><br><br>

**Lab 6:** Users in Australia are complaining that images loading from your S3 bucket in New York are taking 5 seconds to load. How do you reduce this to milliseconds?
<br><br><br>

**Lab 7:** You set up a CDN in front of your S3 bucket, but users figured out the direct URL to your S3 bucket and are bypassing the CDN entirely. How do you stop this?
<br><br><br>

**Lab 8:** You updated your company logo on S3, but your global users are still seeing the old logo on your website. How do you fix this immediately?
<br><br><br>

**Lab 9:** You are hosting a multiplayer UDP-based online game. You want users globally to connect to your US-based servers with the lowest possible jitter and packet loss.
<br><br><br>

**Lab 10:** You need to whitelist your application's IP address with a highly secure bank, but your backend servers span across 3 different AWS regions. How do you provide the bank with just two static IPs?
<br><br><br>

---

## 🔧 8. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** In Route 53, create an **Alias Record** at the Zone Apex (empty record name) pointing to the Application Load Balancer endpoint.

**Solution 2:** Create two records in Route 53 using **Weighted Routing**. Give the old environment a weight of 95 and the new environment a weight of 5.

**Solution 3:** Set up a Route 53 Health Check on the primary application. Use **Failover Routing**, setting the primary app as "Primary" and the S3 website endpoint as "Secondary".

**Solution 4:** Use Route 53 **Geolocation Routing**. Create one record for Location: France pointing to Paris, and another for Location: Germany pointing to Frankfurt.

**Solution 5:** Create a **Private Hosted Zone** in Route 53 and associate it with your VPC. Create an A-record for `database.internal.corp` pointing to the database's private IP.

**Solution 6:** Create an **Amazon CloudFront Distribution** with the S3 bucket as the origin. The images will be cached at Edge Locations in Australia.

**Solution 7:** Configure **Origin Access Control (OAC)** on your CloudFront distribution. Then, update the S3 bucket policy to deny all direct access unless the request originates from the CloudFront service principal.

**Solution 8:** Go to the CloudFront console and create a **Cache Invalidation** for the specific file path (e.g., `/images/logo.png`), forcing Edge Locations to fetch the new file.

**Solution 9:** Implement **AWS Global Accelerator**. Users will connect to the Anycast IPs at their nearest Edge Location, and UDP traffic will route over the optimized AWS fiber backbone.

**Solution 10:** Provision **AWS Global Accelerator**, which provides two static Anycast IP addresses. Route those IPs to your regional endpoints (ALBs or EC2 instances) behind the scenes.

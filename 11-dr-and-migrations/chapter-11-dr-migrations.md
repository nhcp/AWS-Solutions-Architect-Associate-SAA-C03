# SAA-C03 Chapter 11: Disaster Recovery & Migrations

---

## 📖 1. The Big Picture: Surviving the Worst
**The Objective:** To ensure business continuity when a natural disaster or massive outage destroys a data center, and to seamlessly migrate on-premises workloads into the AWS Cloud.
**The Problem:** Hardware fails, natural disasters happen, and human errors (like deleting a production database) are inevitable. If your entire application lives in one AWS Region and that region goes down, your business is dead. Furthermore, moving 100 TB of data from an old corporate data center to AWS over a standard internet connection would take years.
**The Solution:** We implement **Disaster Recovery (DR) Strategies** across multiple AWS Regions, ranging from cheap "Backup & Restore" to expensive "Active-Active" global sites. To move data, we use specialized tools like **AWS DataSync** for files, **AWS DMS** for databases, and **AWS Application Migration Service (MGN)** for full servers.
**The Trade-offs:** Disaster recovery is a balance of Cost vs. Speed. Having a fully scaled, identical backup site in a second region provides instant recovery but doubles your AWS monthly bill.

---

## 🧠 2. Core Theory: Disaster Recovery (DR) Strategies

### RPO and RTO (Critical Exam Concepts)
* **RPO (Recovery Point Objective):** How much *data* can you afford to lose? (e.g., If RPO is 1 hour, you must back up data every hour).
* **RTO (Recovery Time Objective):** How much *time* can your business be offline before you are fired? (e.g., If RTO is 4 hours, your systems must be back online within 4 hours of the crash).

### The 4 Disaster Recovery Tiers
1. **Backup and Restore (Cheapest, Slowest):**
   * **How it works:** Data is backed up to S3 in another region. If a disaster hits, you manually provision EC2 servers and restore the data.
   * **RTO:** Hours to Days.
2. **Pilot Light (Cheap, Faster):**
   * **How it works:** The critical core of your app (the Database) is kept continuously running and replicating in the DR region. The compute (EC2) is completely shut off. In a disaster, you turn on the EC2 instances.
   * **RTO:** Tens of minutes to Hours.
3. **Warm Standby (Expensive, Fast):**
   * **How it works:** A scaled-down version of your *entire* environment (DB + EC2) is constantly running in the DR region. In a disaster, you simply scale up the EC2 instances to handle production traffic.
   * **RTO:** Minutes.
4. **Multi-Site / Hot Standby (Most Expensive, Instant):**
   * **How it works:** A full 100% production-scale environment runs continuously in the DR region. Route 53 routes traffic to both regions actively.
   * **RTO:** Seconds to Zero.



---

## 🚚 3. Core Theory: Data & Database Migrations

### AWS DataSync
Used to move massive amounts of file data from on-premises to AWS, or between AWS storage services (e.g., EFS to S3).
* **How it works:** You install a DataSync Agent on-premises. It connects to your NFS/SMB file servers and securely transfers data into S3, EFS, or FSx at incredibly high speeds using a proprietary protocol.
* **Use Case:** One-time massive file migrations, or scheduled daily data replication.



### AWS Database Migration Service (DMS)
Used to migrate relational databases, data warehouses, and NoSQL databases into AWS quickly and securely.
* **Continuous Replication:** The source database remains fully operational during the migration, minimizing downtime.
* **Homogeneous Migration:** Oracle to Oracle, or MySQL to MySQL.
* **Heterogeneous Migration:** Oracle to Amazon Aurora.
* **Schema Conversion Tool (SCT):** If you are moving from Oracle to Aurora (heterogeneous), the schema (tables, rules, types) won't match. You use SCT first to translate the schema, then use DMS to move the actual data.

---

## 🖥️ 4. Core Theory: Server Migrations & Backups

### AWS Application Migration Service (MGN)
The primary service for "lift-and-shift" (rehosting) migrations to AWS. (Note: This replaced the older AWS Server Migration Service - SMS).
* **How it works:** You install an agent on your physical on-premises servers. MGN continuously replicates the server's OS, application stack, and disks into AWS EBS volumes. During cutover, it automatically launches EC2 instances from those volumes.

### AWS Backup
A centralized, fully managed service to automate and manage backups across multiple AWS services (EC2, EBS, RDS, EFS, DynamoDB, Storage Gateway).
* **Backup Plans:** You define policies (e.g., "Backup all resources tagged `Env=Prod` every 12 hours, keep for 30 days").
* **Cross-Region/Cross-Account:** AWS Backup can automatically copy your backups to a different AWS Region or a completely different AWS Account for strict disaster recovery compliance.

---

## 🏗️ 5. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Migrate a live 5TB MySQL database to Amazon RDS with zero downtime. | **AWS Database Migration Service (DMS)** |
| Migrate an old Microsoft SQL Server to Amazon Aurora PostgreSQL. | **AWS Schema Conversion Tool (SCT) + DMS** |
| Move 100 TB of files from an on-premises NAS to Amazon EFS securely over the internet. | **AWS DataSync** |
| DR Strategy where the database is running, but web servers are fully powered off. | **Pilot Light** |
| DR Strategy providing a scaled-down, fully functional environment ready to take traffic immediately. | **Warm Standby** |
| Centrally manage daily snapshots of EBS volumes and RDS databases across 5 accounts. | **AWS Backup** |
| Perform a "Lift and Shift" migration of 50 physical Windows servers to AWS EC2. | **AWS Application Migration Service (MGN)** |
| Transfer 5 Petabytes of data from a data center with a very slow internet connection. | **AWS Snowball Edge (Snow Family)** |

---

## 🤝 6. The Interview Room: 20 Q&A (Flashcards)

**Question:** What does RTO stand for?
**Answer:** Recovery Time Objective; the maximum acceptable downtime for an application before severe business impact occurs.

**Question:** What does RPO stand for?
**Answer:** Recovery Point Objective; the maximum acceptable amount of data loss, measured in time (e.g., 1 hour of lost data).

**Question:** Which DR strategy requires you to manually provision EC2 instances from scratch after a failure?
**Answer:** Backup and Restore.

**Question:** What is the "Pilot Light" DR strategy?
**Answer:** Keeping core services (like databases) continuously running and replicating, while compute resources are only provisioned during a disaster.

**Question:** How does "Warm Standby" differ from "Pilot Light"?
**Answer:** Warm Standby keeps a scaled-down version of the *entire* environment (both compute and databases) running continuously.

**Question:** What is the "Multi-Site Active/Active" DR strategy?
**Answer:** Running full-scale production environments in two or more regions simultaneously, routing traffic to all of them.

**Question:** What is AWS DataSync used for?
**Answer:** High-speed, automated data transfer between on-premises storage systems (NFS, SMB) and AWS storage services (S3, EFS, FSx).

**Question:** Does AWS DataSync run over the internet?
**Answer:** It can run over the internet, AWS Direct Connect, or AWS Site-to-Site VPN, and encrypts data in transit.

**Question:** What is AWS Database Migration Service (DMS)?
**Answer:** A service that migrates databases to AWS easily and securely while the source database remains fully operational.

**Question:** If migrating from an Oracle database to Amazon Aurora PostgreSQL, what tool must you use before DMS?
**Answer:** AWS Schema Conversion Tool (SCT), to convert the source database schema and code objects to a format compatible with the target.

**Question:** Can AWS DMS continuously replicate ongoing changes after the initial migration?
**Answer:** Yes, by enabling Change Data Capture (CDC) in DMS.

**Question:** What is AWS Application Migration Service (MGN)?
**Answer:** The recommended service for lift-and-shift migrations of physical, virtual, or cloud servers to AWS EC2.

**Question:** What is AWS Application Discovery Service?
**Answer:** A tool that gathers information about your on-premises data center (server usage, dependencies) to help plan your AWS migration.

**Question:** What is the Snow Family used for in migrations?
**Answer:** Physical offline data transfer for massive datasets (Terabytes to Exabytes) when internet bandwidth is insufficient or too expensive.

**Question:** What is AWS Backup?
**Answer:** A fully managed service that centralizes and automates the backup of data across AWS services in the cloud and on-premises.

**Question:** Can AWS Backup copy a backup from the `us-east-1` region to the `eu-west-1` region?
**Answer:** Yes, Cross-Region backup is a native feature used for disaster recovery compliance.

**Question:** What happens to your Route 53 DNS routing in a Multi-Site Active/Active setup if one region fails?
**Answer:** Route 53 Health Checks detect the failure and automatically failover all traffic to the remaining healthy region.

**Question:** How can you protect your automated backups from being deleted by a rogue administrator?
**Answer:** Use AWS Backup Vault Lock, which enforces a WORM (Write Once Read Many) policy preventing deletion.

**Question:** Is Amazon S3 Cross-Region Replication (CRR) a valid DR strategy?
**Answer:** Yes, it is widely used in Backup & Restore strategies to ensure data survives a regional disaster.

**Question:** What is a common way to achieve RPO of a few seconds for an RDS database across regions?
**Answer:** Use Amazon Aurora Global Database, which replicates data globally with typical latency under 1 second.

---

## 🛠️ 7. Hands-on Labs (Test Your Skills)

**Lab 1:** Your company mandates an RTO of 4 hours and an RPO of 24 hours. Design the most cost-effective DR strategy.
<br><br><br>

**Lab 2:** You need to migrate an on-premises MySQL database to Amazon RDS. The database is actively processing customer orders, so you cannot afford more than 5 minutes of downtime.
<br><br><br>

**Lab 3:** You have 500 TB of video archives on an old corporate NFS server. Your company internet connection is 100 Mbps. You need to move this to S3 Glacier Deep Archive.
<br><br><br>

**Lab 4:** You want to ensure that every single EBS volume in your AWS account is backed up at 2:00 AM every night and retained for 30 days automatically.
<br><br><br>

**Lab 5:** Your application has a critical zero-downtime requirement. You have deployed it in both `us-east-1` and `us-west-2`. How do you split user traffic 50/50 and handle regional failover?
<br><br><br>

**Lab 6:** You are tasked with migrating 100 physical Linux web servers to AWS. You don't have time to rewrite the applications into containers.
<br><br><br>

**Lab 7:** You need to migrate an enterprise Oracle Database to Amazon Aurora MySQL to save on licensing costs.
<br><br><br>

**Lab 8:** Your company requires an RTO of 15 minutes for a critical web application, but you have a strict budget and cannot afford to run a full secondary environment 24/7.
<br><br><br>

**Lab 9:** You are setting up AWS Backup. To protect against a total AWS Account compromise, your security team demands that backups cannot be deleted by anyone, including the root user.
<br><br><br>

**Lab 10:** You need to continuously synchronize a massive file system between an on-premises data center and Amazon EFS so that the cloud environment is always ready for a failover.
<br><br><br>

---

## 🔧 8. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Implement **Backup & Restore**. Take daily automated EBS/RDS snapshots and use S3 Cross-Region Replication to copy data to the DR region. Provision EC2 instances manually only if a disaster occurs.

**Solution 2:** Use **AWS Database Migration Service (DMS)** with Change Data Capture (CDC) enabled. DMS will migrate the bulk of the data and continuously replicate new transactions until you are ready to update the DNS for cutover.

**Solution 3:** Order an **AWS Snowball Edge** device. With a 100 Mbps connection, transferring 500 TB over the internet would take over a year. Load the data locally onto the Snowball and ship it back to AWS.

**Solution 4:** Implement **AWS Backup**. Create a Backup Plan that targets all resources in the account, set the backup window to 2:00 AM, and set the lifecycle retention rule to 30 days.

**Solution 5:** Use **Route 53 Weighted Routing** (set to 50/50) across both regions. Attach Route 53 Health Checks to both endpoints. If one region fails the health check, Route 53 stops sending traffic to it (Multi-Site Active/Active).

**Solution 6:** Use **AWS Application Migration Service (MGN)**. Install the MGN replication agent on all 100 physical servers to perform a continuous block-level "lift-and-shift" replication into AWS EBS volumes.

**Solution 7:** First, use the **AWS Schema Conversion Tool (SCT)** to translate the Oracle schema, views, and stored procedures into MySQL format. Then, use **AWS DMS** to migrate the actual table data.

**Solution 8:** Implement the **Pilot Light** strategy. Keep the database running and replicating in the secondary region. Configure an Auto Scaling Group for the web servers with a desired capacity of 0, and scale it up only if disaster strikes.

**Solution 9:** Create an AWS Backup Vault and apply an **AWS Backup Vault Lock** in Compliance mode. Once locked, the backups become immutable and cannot be deleted by anyone until the retention period expires.

**Solution 10:** Deploy the **AWS DataSync** agent in your on-premises data center. Configure a DataSync task to run continuously, replicating file changes from your local NFS/SMB share directly into Amazon EFS.

# SAA-C03 Chapter 4: Storage Mastery (S3, EBS, EFS)

---

## 📖 1. The Big Picture: Data Persistence & Storage
**The Objective:** To select the most cost-effective, performant, and highly available storage solution based on the specific type of data and how it is accessed.
**The Problem:** Storing massive video files on a high-speed database drive is a massive waste of money. Conversely, running a database on slow, archival storage will crash the application. Furthermore, EC2 instances are ephemeral—if the server dies, the internal hard drive dies with it.
**The Solution:** We decouple compute from storage. We use **EBS (Elastic Block Store)** for EC2 operating systems and databases, **EFS (Elastic File System)** when multiple Linux servers need to share the same files, and **Amazon S3 (Simple Storage Service)** for infinite, cheap object storage like images and backups.
**The Trade-offs:** S3 is cheap and infinite but cannot run an operating system. EBS is extremely fast but is locked to a single Availability Zone. EFS can be shared across multiple AZs but is significantly more expensive than EBS.

---

## 🧠 2. Core Theory: Object Storage (Amazon S3)

### Amazon S3 (Simple Storage Service)
S3 is an object storage service offering industry-leading scalability, data availability, security, and performance. Data is stored as "Objects" inside "Buckets".
* **Buckets:** Must have a globally unique name across all of AWS.
* **Objects:** Files (up to 5TB each). They consist of data, a key (the file name/path), and metadata.

### S3 Storage Classes (Critical Exam Topic)
You must match the data lifecycle to the correct storage tier to optimize costs:
* **S3 Standard:** For frequently accessed data. High availability and low latency.
* **S3 Standard-Infrequent Access (IA):** For data accessed less than once a month, but requires rapid access when needed. Cheaper storage, but charges a retrieval fee.
* **S3 One Zone-IA:** Same as IA, but data is only stored in one Availability Zone. Cheaper, but data is lost if the AZ is destroyed.
* **S3 Glacier Flexible Retrieval:** For archives. Data retrieval takes minutes to hours.
* **S3 Glacier Deep Archive:** The absolute cheapest storage in AWS. Retrieval takes 12 to 48 hours. Best for regulatory backups kept for years.
* **S3 Intelligent-Tiering:** Uses machine learning to automatically move objects between frequent and infrequent access tiers based on usage patterns.



---

## 💾 3. Core Theory: Block Storage (EBS & Instance Store)

### Amazon EBS (Elastic Block Store)
EBS is a network drive you can attach to your EC2 instances while they run. It allows your data to persist even after the instance is terminated.
* **Scope:** An EBS volume is locked to a **single Availability Zone (AZ)**.
* **gp2 / gp3 (General Purpose SSD):** Balances price and performance for a wide variety of workloads (boot volumes, virtual desktops). gp3 allows you to scale IOPS and throughput independently of storage size.
* **io1 / io2 (Provisioned IOPS SSD):** For critical business applications that require sustained IOPS performance (e.g., large relational databases).
* **st1 (Throughput Optimized HDD):** Low-cost HDD designed for frequently accessed, throughput-intensive workloads (Big Data, Data Warehouses). Cannot be a boot volume.
* **sc1 (Cold HDD):** The lowest-cost HDD design for less frequently accessed workloads (File Archives). Cannot be a boot volume.

### EC2 Instance Store
High-performance hardware disks physically attached to the host computer running your EC2 instance.
* **Ephemeral:** If the instance is stopped or terminated, the data on the Instance Store is **lost forever**.
* **Use Case:** Caches, buffers, scratch data, and temporary content.

---

## 📁 4. Core Theory: File Storage (EFS & FSx)

### Amazon EFS (Elastic File System)
A fully managed, scalable NFS (Network File System) that can be mounted to hundreds of EC2 instances simultaneously.
* **Scope:** Multi-AZ by default. EC2 instances in different AZs can access the same EFS file system at the same time.
* **Compatibility:** Works with **Linux** instances only.
* **Performance Modes:** General Purpose (default for web serving) vs. Max I/O (for highly parallel, big data applications).

### Amazon FSx
When you need a specialized file system that EFS cannot provide:
* **FSx for Windows File Server:** A fully managed native Microsoft Windows file system. Supports SMB protocol and Active Directory integration.
* **FSx for Lustre:** A high-performance file system optimized for fast processing of workloads like machine learning, high performance computing (HPC), and video processing.



---

## 🚚 5. Core Theory: Hybrid Storage & Data Migration

### AWS Storage Gateway
A hybrid cloud storage service that gives your on-premises environment access to virtually unlimited cloud storage.
* **File Gateway:** Exposes S3 buckets as local file shares (NFS or SMB) on-premises.
* **Volume Gateway:** Presents cloud-backed block storage volumes (iSCSI) to your on-premises servers.

### AWS Snow Family
Physical devices used to migrate petabytes of data into and out of AWS, bypassing the internet.
* **Snowcone:** Small, rugged, 8 TB device. Good for edge computing.
* **Snowball Edge:** Suitcase-sized device. Can hold 80 TB. Used for large-scale data migrations.
* **Snowmobile:** A 45-foot long shipping container pulled by a semi-trailer truck. Used to move Exabytes of data.

---

## 🏗️ 6. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Store 50TB of video files that are accessed daily with the lowest latency. | **Amazon S3 Standard** |
| Archive 10 years of financial records that are rarely accessed but must be kept for compliance. | **Amazon S3 Glacier Deep Archive** |
| Provide shared file storage for 50 Linux EC2 instances across multiple AZs. | **Amazon EFS** |
| Provide shared file storage for 20 Windows EC2 instances integrating with Active Directory. | **Amazon FSx for Windows** |
| Attach a high-performance boot drive to an EC2 instance running a relational database. | **EBS io2 (Provisioned IOPS)** |
| Store temporary cache data requiring millions of IOPS and zero network latency. | **EC2 Instance Store** |
| Migrate 500 TB of data from an on-premises data center to AWS without using the internet. | **AWS Snowball Edge** |
| Connect on-premises servers to an S3 bucket via standard NFS/SMB protocols. | **AWS Storage Gateway (File Gateway)** |

---

## 🤝 7. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is the maximum size of a single object you can store in Amazon S3?
**Answer:** The maximum size of a single S3 object is 5 Terabytes (TB).

**Question:** How does Amazon S3 guarantee that you do not accidentally overwrite or delete a critical file?
**Answer:** By enabling S3 Versioning, which keeps multiple variants of an object in the same bucket.

**Question:** What is the difference between S3 Standard and S3 Standard-IA?
**Answer:** Standard is for frequent access; Standard-IA is cheaper for storage but charges a fee every time you retrieve data.

**Question:** When should you use S3 Intelligent-Tiering?
**Answer:** When your data has unknown or changing access patterns, as it automatically moves objects to the most cost-effective tier.

**Question:** What is an S3 Lifecycle Policy?
**Answer:** A set of rules that automatically transitions objects between storage classes (e.g., move to Glacier after 30 days) or expires them.

**Question:** What is the scope of an EBS volume?
**Answer:** An EBS volume is locked to a single Availability Zone (AZ) and cannot be natively attached across AZs.

**Question:** How can you move an EBS volume from one Availability Zone to another?
**Answer:** Take an EBS Snapshot of the volume, and then restore that snapshot as a new volume in the target Availability Zone.

**Question:** What is the main difference between an EBS volume and an EC2 Instance Store?
**Answer:** EBS is persistent network storage; Instance Store is ephemeral, physical storage that is wiped if the instance stops.

**Question:** Which EBS volume type should you choose for a high-performance relational database?
**Answer:** Provisioned IOPS SSD (io1 or io2).

**Question:** Can an EBS volume be attached to multiple EC2 instances at the same time?
**Answer:** Normally no, but EBS Multi-Attach allows attaching a single Provisioned IOPS (io1/io2) volume to multiple instances in the same AZ.

**Question:** What is Amazon EFS?
**Answer:** Elastic File System is a managed, scalable NFS file system that can be shared across multiple Linux instances and multiple AZs.

**Question:** Can a Windows EC2 instance mount an Amazon EFS file system?
**Answer:** No, EFS only supports Linux. For Windows, you must use Amazon FSx for Windows File Server.

**Question:** What is Amazon FSx for Lustre?
**Answer:** A high-performance file system optimized for High Performance Computing (HPC), machine learning, and fast processing of S3 data.

**Question:** What happens to an EBS root volume by default when an EC2 instance is terminated?
**Answer:** By default, the root EBS volume is deleted upon instance termination (though this behavior can be changed).

**Question:** What is S3 Cross-Region Replication (CRR)?
**Answer:** A feature that automatically asynchronously copies objects across buckets in different AWS Regions for disaster recovery.

**Question:** What is S3 Object Lock?
**Answer:** A feature that stores objects using a "Write Once, Read Many" (WORM) model, preventing them from being deleted or modified for a fixed amount of time.

**Question:** How do you migrate 100 TB of data to AWS if your internet connection is very slow?
**Answer:** Order an AWS Snowball Edge device, load the data locally, and ship it physically to AWS.

**Question:** What is AWS Storage Gateway?
**Answer:** A hybrid storage service that enables your on-premises applications to seamlessly use AWS cloud storage.

**Question:** What specific Storage Gateway type acts as a local cache for Amazon S3 objects using NFS/SMB?
**Answer:** Amazon S3 File Gateway.

**Question:** How do you encrypt an existing unencrypted EBS volume?
**Answer:** Create a snapshot of the unencrypted volume, copy the snapshot and select "Encrypt", then create a new volume from the encrypted snapshot.

---

## 🛠️ 8. Hands-on Labs (Test Your Skills)

**Lab 1:** You are tasked with keeping server log files for 7 years to comply with financial regulations. The logs are almost never read. Design a cost-effective S3 solution.
<br><br><br>

**Lab 2:** You need to share a directory of configuration files across 20 web servers running Amazon Linux 2. The servers span across three Availability Zones.
<br><br><br>

**Lab 3:** An EC2 instance running a MySQL database is out of storage space on its 100GB `gp3` drive. How do you increase the size to 500GB without experiencing any downtime?
<br><br><br>

**Lab 4:** You accidentally deleted an important PDF file from an S3 bucket. Design a configuration that prevents this from happening again.
<br><br><br>

**Lab 5:** Your on-premises file server is running out of physical hard drive space. You want to seamlessly extend this storage into AWS S3 using standard SMB protocols.
<br><br><br>

**Lab 6:** A machine learning workload requires extreme IOPS and microsecond latency for a temporary dataset. Data persistence after the instance terminates is not required.
<br><br><br>

**Lab 7:** You need to migrate an entire on-premises data center (10 Petabytes of data) to AWS within 2 months. Your corporate internet connection is 1 Gbps.
<br><br><br>

**Lab 8:** Your company has a strict policy that no one, not even the Root account, can delete a specific set of compliance documents stored in S3 for exactly 5 years.
<br><br><br>

**Lab 9:** You need to transfer an exact copy of an EBS volume from the `us-east-1a` Availability Zone to the `us-east-1b` Availability Zone.
<br><br><br>

**Lab 10:** You are running a legacy Windows application that relies on the Microsoft SMB protocol and requires integration with your corporate Active Directory.
<br><br><br>

---

## 🔧 9. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Upload the logs to an Amazon S3 bucket. Configure an S3 Lifecycle Rule to transition objects to S3 Glacier Deep Archive immediately after upload.

**Solution 2:** Create an Amazon EFS (Elastic File System). Mount this EFS file system to all 20 EC2 instances. Because EFS is inherently Multi-AZ, all instances can read and write to it concurrently.

**Solution 3:** Go to the EC2 Console -> Volumes. Select the `gp3` volume, click "Modify Volume", and change the size to 500GB. Once the AWS modification completes, log into the OS and extend the file system.

**Solution 4:** Enable S3 Versioning on the bucket. If a file is deleted, S3 simply places a "Delete Marker" over it, allowing you to restore the previous version at any time.

**Solution 5:** Deploy an AWS Storage Gateway virtual appliance on-premises and configure it as an S3 File Gateway. Map your on-premises network drives to the File Gateway.

**Solution 6:** Launch the EC2 instance using an instance type that includes an EC2 Instance Store (e.g., `i3` instances). Use the physically attached NVMe SSDs for the workload.

**Solution 7:** Order an AWS Snowmobile. A 1 Gbps connection would take years to transfer 10 Petabytes. Snowmobile is a specialized shipping container designed specifically for exabyte-scale migrations.

**Solution 8:** Store the documents in an S3 Bucket with S3 Object Lock enabled in Compliance Mode, setting the retention period to 5 years.

**Solution 9:** Create an EBS Snapshot of the volume in `us-east-1a`. Navigate to the Snapshots menu, select the snapshot, click "Create Volume", and specify `us-east-1b` as the target Availability Zone.

**Solution 10:** Provision Amazon FSx for Windows File Server. Join the FSx file system to your existing AWS Managed Microsoft AD or self-managed Active Directory.

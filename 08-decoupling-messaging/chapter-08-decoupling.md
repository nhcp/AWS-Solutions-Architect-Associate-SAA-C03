# SAA-C03 Chapter 8: Decoupling Architecture (SQS, SNS, Kinesis)

---

## 📖 1. The Big Picture: Microservices & Decoupling
**The Objective:** To design applications where components communicate asynchronously, ensuring that if one piece fails, the rest of the system stays online.
**The Problem:** In a tightly coupled, synchronous application, if the Web Server talks directly to the Database, and the Database crashes, the Web Server crashes too. Traffic spikes will overwhelm the backend, dropping user requests entirely.
**The Solution:** We introduce **Decoupling** using message queues and topics. We put **Amazon SQS** between the web tier and the backend tier. The web server drops a message into the queue and immediately tells the user "Success." The backend servers slowly pull messages from the queue at their own pace. We use **Amazon SNS** to broadcast alerts, and **Amazon Kinesis** to handle massive real-time data streaming.
**The Trade-offs:** Asynchronous architectures add complexity. You must design for "eventual consistency" (the user's action might take a few seconds or minutes to fully process in the background) and handle duplicate messages.

---

## 🧠 2. Core Theory: Amazon SQS (Queues)

### What is Amazon SQS?
Simple Queue Service (SQS) is AWS's oldest service. It is a fully managed message queue that lets you decouple and scale microservices. Messages are "Pulled" by consumers (like EC2 or Lambda).
* **Retention:** Messages stay in the queue for 4 days by default (maximum 14 days). Once processed, the consumer explicitly deletes the message.
* **Standard Queues:** Offers unlimited throughput. Provides "at-least-once" delivery, meaning a message might occasionally be delivered twice. Messages are delivered in "best-effort" order (not strictly guaranteed).
* **FIFO Queues (First-In-First-Out):** Guarantees exactly-once processing and strict ordering. Limited to 300 messages per second (or 3,000 with batching). The queue name must end with `.fifo`.

### Critical SQS Concepts
* **Visibility Timeout:** When a consumer pulls a message, SQS hides that message from other consumers so it isn't processed twice. If the consumer doesn't process and delete the message before the timeout expires (default 30 seconds), the message reappears in the queue.
* **Dead Letter Queue (DLQ):** If a message fails to process multiple times (a "poison pill"), SQS moves it to a DLQ so your developers can analyze it later without clogging up the main queue.
* **Long Polling:** Instead of pinging SQS 100 times a second and getting empty responses (which costs money), a consumer can use Long Polling to wait up to 20 seconds for a message to arrive. This drastically reduces AWS API costs.



---

## 📢 3. Core Theory: Amazon SNS (Pub/Sub)

### What is Amazon SNS?
Simple Notification Service (SNS) is a highly scalable Publisher/Subscriber (Pub/Sub) system. Unlike SQS (where consumers "pull" data), SNS "pushes" data to subscribers.
* **Topics:** You publish a message to a Topic.
* **Subscribers:** Multiple endpoints can subscribe to the same Topic (e.g., Email, SMS texts, HTTP endpoints, AWS Lambda, and Amazon SQS).
* **Message Filtering:** Subscribers can define filter policies so they only receive specific messages (e.g., only send "Error" logs to the admin email, ignore "Info" logs).

### The "Fan-Out" Architecture (Exam Favorite)
You publish a single message to one SNS Topic. That SNS Topic has 5 different SQS queues subscribed to it. The message is instantly replicated ("fanned out") to all 5 queues, allowing 5 different independent microservices to process the exact same event at their own pace.



---

## 🌊 4. Core Theory: Amazon Kinesis & Amazon MQ

### Amazon Kinesis Data Streams (Real-Time Data)
Used for ingesting massive amounts of real-time data (like website clickstreams, financial market data, or IoT sensor logs).
* **Shards:** Kinesis capacity is defined by Shards. You must provision enough shards to handle your incoming data throughput.
* **Data Retention:** Data is kept in Kinesis for 1 to 365 days. 
* **Key Difference from SQS:** When a consumer reads data from Kinesis, the data is **NOT deleted**. Multiple different applications can read the exact same data stream simultaneously, replaying the data from the past if needed.

### Amazon Kinesis Data Firehose
Used to reliably load streaming data into data lakes and data warehouses in *near real-time* (typically a 60-second delay).
* **Serverless:** No shards to manage; it auto-scales completely.
* **Destinations:** Automatically writes the streaming data into **Amazon S3, Amazon Redshift, or Amazon OpenSearch**.

### Amazon MQ
If you are migrating an old, on-premises enterprise application that relies on legacy protocols like MQTT, AMQP, or JMS (Java Message Service), rewriting it to use SQS/SNS is difficult. Amazon MQ is a managed message broker for **ActiveMQ** and **RabbitMQ** to make cloud migrations easy.



---

## 🏗️ 5. Architecture Scenarios (Exam Cheat Sheet)

| Business Requirement | Architect's Solution |
| :--- | :--- |
| Decouple a web app from a backend video processing fleet. | **Amazon SQS (Standard)** |
| Ensure financial stock market trades are processed in exact order. | **Amazon SQS (FIFO)** |
| Send an instant email and text message to 10,000 customers. | **Amazon SNS** |
| Replicate one uploaded file event to 4 different processing worker groups. | **SNS to SQS Fan-Out Pattern** |
| Ingest 1 million IoT sensor metrics per second for real-time dashboarding. | **Amazon Kinesis Data Streams** |
| Capture real-time clickstream data and dump it into S3 for long-term storage. | **Amazon Kinesis Data Firehose** |
| Migrate a legacy Java application using JMS queues to the cloud. | **Amazon MQ** |
| Isolate corrupt messages that keep crashing your backend workers. | **Amazon SQS Dead Letter Queue (DLQ)** |

---

## 🤝 6. The Interview Room: 20 Q&A (Flashcards)

**Question:** What is the main difference between synchronous and asynchronous architectures?
**Answer:** Synchronous requires components to wait for a response directly; asynchronous uses queues/topics so components work independently.

**Question:** What is Amazon SQS?
**Answer:** A fully managed, highly scalable message queuing service that enables you to decouple and scale microservices.

**Question:** What is the difference between SQS Standard and SQS FIFO?
**Answer:** Standard provides high throughput but "best-effort" ordering; FIFO guarantees strict ordering and exactly-once processing but has lower throughput limits.

**Question:** What is an SQS Visibility Timeout?
**Answer:** The period of time during which SQS prevents other consuming components from receiving and processing a message currently being processed.

**Question:** What happens if a consumer crashes before finishing a message in SQS?
**Answer:** The visibility timeout expires, and the message automatically becomes visible again in the queue for another consumer to pick up.

**Question:** What is a Dead Letter Queue (DLQ)?
**Answer:** A secondary queue where messages are sent if they fail to process successfully after a maximum number of retries.

**Question:** How can you reduce the AWS billing costs of an empty SQS queue?
**Answer:** Enable Long Polling, allowing consumers to wait up to 20 seconds for a message instead of making constant, empty API requests.

**Question:** What is Amazon SNS?
**Answer:** A fully managed Pub/Sub messaging service used to push messages to multiple subscribers, including emails, SMS, and other AWS services.

**Question:** What is the "Fan-Out" pattern?
**Answer:** Publishing a single message to an SNS topic that has multiple SQS queues subscribed to it, copying the message to all queues simultaneously.

**Question:** What is Amazon Kinesis Data Streams used for?
**Answer:** Ingesting and storing massively large streams of real-time data, allowing multiple consumers to analyze the stream concurrently.

**Question:** How does Kinesis differ from SQS regarding data consumption?
**Answer:** In SQS, messages are deleted after processing; in Kinesis, data persists in the stream (up to 365 days) and can be read by multiple consumers repeatedly.

**Question:** What is a Kinesis Shard?
**Answer:** The base throughput unit of a Kinesis Data Stream; scaling a stream requires adding or splitting shards.

**Question:** What is Amazon Kinesis Data Firehose?
**Answer:** A fully managed service that captures, transforms, and loads streaming data into data lakes (like S3) or data warehouses (like Redshift).

**Question:** Does Kinesis Firehose provide true real-time processing?
**Answer:** No, it provides "near real-time" processing because it batches data before delivering it (minimum 60-second or 1MB buffer).

**Question:** What is Amazon MQ?
**Answer:** A managed message broker service for Apache ActiveMQ and RabbitMQ, used primarily for migrating legacy applications to AWS.

**Question:** Can an SQS FIFO queue subscribe to a standard SNS topic?
**Answer:** No, an SQS FIFO queue must be subscribed to an SNS FIFO topic to maintain strict ordering guarantees.

**Question:** What is SNS Message Filtering?
**Answer:** A feature that allows subscribers to specify filter policies so they only receive messages from a topic that match certain attributes.

**Question:** What is the maximum message size allowed in standard SQS and SNS?
**Answer:** 256 KB. (Larger payloads require using the S3 Extended Client Library).

**Question:** How do you guarantee exactly-once processing in AWS messaging?
**Answer:** By using SQS FIFO queues or SNS FIFO topics.

**Question:** In SQS, what is Short Polling?
**Answer:** The default behavior where SQS samples only a subset of its servers to find messages, returning immediately even if the queue appears empty.

---

## 🛠️ 7. Hands-on Labs (Test Your Skills)

**Lab 1:** Your backend database crashed for 2 hours, and all user orders were lost. How do you redesign the architecture so no orders are ever lost during a database outage?
<br><br><br>

**Lab 2:** Your EC2 worker takes 2 minutes to process a video file. However, after 30 seconds, a second EC2 worker grabs the exact same video and starts processing it too. How do you fix this duplication?
<br><br><br>

**Lab 3:** A software bug in your code causes a specific message format to crash your application. This bad message keeps returning to the queue and crashing your workers endlessly.
<br><br><br>

**Lab 4:** Your finance department complains about high AWS API costs from your EC2 instances polling an SQS queue that is empty 90% of the time.
<br><br><br>

**Lab 5:** When a customer uploads a photo, you need to resize the photo, update the database, and send a welcome email. Doing this sequentially takes too long.
<br><br><br>

**Lab 6:** You are building a stock trading application. It is critical that "Buy" and "Sell" orders are processed in the exact order the user clicked them.
<br><br><br>

**Lab 7:** You need to ingest clickstream data from 5 million mobile devices in real-time, keep the data available for 7 days, and run real-time analytics on it.
<br><br><br>

**Lab 8:** You want to capture all server logs in real-time and dump them directly into an Amazon S3 bucket for cheap, long-term storage without managing any streaming servers.
<br><br><br>

**Lab 9:** You are migrating a 15-year-old on-premises banking application. It uses the AMQP protocol and Java Message Service (JMS).
<br><br><br>

**Lab 10:** You have an SNS Topic that broadcasts all application events. You have a "Fraud Detection" SQS queue, but it only needs to receive events where the transaction amount is > $10,000.
<br><br><br>

---

## 🔧 8. Lab Solutions (The Step-by-Step Architect Fix)

**Solution 1:** Place an **Amazon SQS Queue** between the web servers and the database. If the database crashes, orders safely accumulate in the queue for up to 14 days until the database comes back online.

**Solution 2:** Increase the **SQS Visibility Timeout** to a value greater than the processing time (e.g., set it to 3 minutes). This hides the message from other workers until the first worker is finished and deletes it.

**Solution 3:** Configure a **Dead Letter Queue (DLQ)**. Set the `MaximumReceives` threshold to 3. If the message fails to process 3 times, SQS automatically moves it to the DLQ, saving your workers.

**Solution 4:** Enable **Long Polling** by setting the `ReceiveMessageWaitTimeSeconds` to 20. The workers will wait up to 20 seconds for a message to arrive before consuming an API request.

**Solution 5:** Implement the **SNS to SQS Fan-Out** pattern. Publish the photo upload event to one SNS Topic, and have three separate SQS queues subscribed to it so the tasks are processed in parallel by different microservices.

**Solution 6:** Use an **Amazon SQS FIFO Queue** (First-In-First-Out) with a `.fifo` extension. This guarantees messages are processed exactly once and in the exact order they were sent.

**Solution 7:** Implement **Amazon Kinesis Data Streams**. Provision enough shards to handle the throughput and configure the data retention period to 7 days.

**Solution 8:** Use **Amazon Kinesis Data Firehose**. Configure the source as your log producers and set the destination directly to your Amazon S3 bucket.

**Solution 9:** Provision an **Amazon MQ** broker. It natively supports legacy messaging protocols like JMS, AMQP, and MQTT, allowing a seamless lift-and-shift migration without rewriting code.

**Solution 10:** Apply an **SNS Message Filter Policy** to the SQS subscription. Define the policy so that the queue only receives the message if the JSON payload includes the attribute `amount > 10000`.

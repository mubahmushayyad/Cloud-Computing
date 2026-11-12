# ☁️ Week 7 — Decoupling, Monitoring, CDN & Infrastructure as Code

> **Course:** HCCDP – Huawei Cloud Computing & Data Analytics Professional (Solution Architect learning path)
> **Covers:** Lecture 14 (Decoupling Applications: SQS, SNS/SMN, Cloud Eye, Splunk) · Lecture 15 (Global Services, CloudFront CDN, CloudFormation, Elastic Beanstalk)
> **Clouds touched:** AWS + Huawei Cloud

How do you stop one slow part of your system from crashing everything else? How do you deliver content fast to users around the world? And how do you build infrastructure by running code instead of clicking through a console? This week answered all three.

---

## 📑 Table of Contents

- [Lecture 14 — Decoupling Applications](#lecture-14--decoupling-applications)
  - [Synchronous vs asynchronous communication](#1-synchronous-vs-asynchronous-communication)
  - [Amazon SQS (queues)](#2-amazon-sqs--queues)
  - [SQS + Auto Scaling](#3-sqs--auto-scaling-group)
  - [Amazon SNS / Huawei SMN (notifications)](#4-amazon-sns--huawei-smn-notifications)
  - [SNS + SQS fan-out](#5-sns--sqs-fan-out)
  - [Huawei Cloud Eye monitoring lab](#6-monitoring-huawei-cloud-eye-vs-aws-cloudwatch)
  - [Log analytics with Splunk](#7-log-analytics-with-splunk)
- [Lecture 15 — Global Services, CDN & Infrastructure as Code](#lecture-15--global-services-cdn--infrastructure-as-code)
  - [Regional vs global services](#1-regional-vs-global-services)
  - [CDN and Amazon CloudFront](#2-content-delivery-network-cdn--amazon-cloudfront)
  - [Lab: CloudFront with S3](#3-lab-cloudfront-with-an-s3-origin)
  - [AWS CloudFormation (IaC)](#4-aws-cloudformation--infrastructure-as-code)
  - [AWS Elastic Beanstalk](#5-aws-elastic-beanstalk)
  - [Lab: Elastic Beanstalk](#6-lab-elastic-beanstalk)
- [Key Takeaways](#-key-takeaways)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)

---

# Lecture 14 — Decoupling Applications

> **Core idea:** components should **not depend directly on each other.**

## 1. Synchronous vs asynchronous communication

When we deploy multiple applications, they inevitably need to talk to each other. There are **two patterns:**

| | **Synchronous** | **Asynchronous / event-based** |
|---|---|---|
| Flow | Application → Application | Application → **Queue** → Application |
| Example | Buying service calls Shipping service directly | Buying service drops the order in a queue; Shipping service picks it up when ready |

**The problem with synchronous:**

- A buying message arrives and the shipping service must respond *immediately*.
- Suddenly **1,000 orders become 10,000 orders** and shipping can't keep up.
- If one component comes under heavy load, the other can't respond instantly, so **many orders get dropped.**

**The solution — asynchronous:**

- An incoming order does **not** go directly to shipping. It goes into a **queue**, and the shipping service **fetches from the queue** at its own pace.

> ✅ **In the cloud, always use a queue.**

![Synchronous vs asynchronous](images/l14-01-sync-vs-async.jpg)

## 2. Amazon SQS — Queues

**Amazon SQS (Simple Queue Service)** is a managed message queue.

- **Producers** send messages into the queue.
- **Consumers** poll the queue to receive and process messages.
- Many producers and many consumers can use the same queue.

![What's a queue](images/l14-02-sqs-what-is-a-queue.jpg)

**Use SQS to decouple application tiers** — for example, web servers (an Auto Scaling Group of EC2 instances) put requests into a queue, and a separate video-processing tier pulls them out and scales on its own.

![SQS decouples tiers](images/l14-03-sqs-decouple-tiers.jpg)

**Why a queue matters — the "lost transactions" problem**

If the application writes directly into the database (RDS, Aurora, DynamoDB) and the load gets too big, **some transactions may be lost.**

![If the load is too big, transactions may be lost](images/l14-04-load-too-big-transactions-lost.jpg)

**Fix: SQS as a buffer for database writes.** The application *enqueues* messages (`SendMessage`), the queue is **infinitely scalable**, and a consumer tier *dequeues* them (`ReceiveMessages`) and inserts them into the database at a steady pace.

![SQS as a buffer to database writes](images/l14-05-sqs-buffer-db-writes.jpg)

**Benefits of loose coupling with SQS**

- Loosely couples application components
- Uses **asynchronous processing**
- Creates **tolerance for failed steps**
- **Absorbs demand spikes**
- A **dead-letter queue** collects messages that the consumer cannot process

![Loose coupling with Amazon SQS](images/l14-06-loose-coupling-dead-letter-queue.jpg)

## 3. SQS + Auto Scaling Group

How do the backend servers know when to scale? **Watch the queue length.**

1. Consumers (EC2 instances in an Auto Scaling Group) poll the queue.
2. A **CloudWatch metric** tracks the queue length — `ApproximateNumberOfMessages`.
3. A **CloudWatch alarm** fires when the metric breaches a threshold.
4. The alarm triggers the **Auto Scaling Group** to **scale out.**

> **Example:** if the queue length goes **above 200**, there are more orders than normal → raise an alarm → scale up the backend instances.

| AWS | Huawei Cloud |
|---|---|
| **CloudWatch** alarm | **Cloud Eye** alarm |

![SQS with Auto Scaling Group](images/l14-07-sqs-with-auto-scaling-group.jpg)

## 4. Amazon SNS / Huawei SMN (notifications)

**SNS (Simple Notification Service)** is a simple messaging service for **notifying people or systems at important moments** — by email, SMS/phone, or push notification.

**The question:** *what if you want to send one message to many receivers?*

**Without SNS (direct integration):** the Buying service has to do its own work **and** send separate notifications to the email service, the fraud service, the shipping service and the queue. All of that extra work **loads the buying application.**

**With SNS (publish/subscribe model):** the Buying service sends **one message to an SNS topic.** The topic holds a **list of subscribers**, and SNS delivers the message to every one of them.

> 🔔 **Analogy:** subscribing to a YouTube channel and clicking the bell icon adds you to the notification list — the creator posts once, and everyone on the list is notified. The Buying application's load is reduced.

![SNS direct integration vs pub/sub](images/l14-08-sns-direct-vs-pubsub.jpg)

**SNS integrates with many AWS services:** CloudWatch alarms, AWS Budgets, Lambda, Auto Scaling Group notifications, S3 bucket events, DynamoDB, CloudFormation state changes, AWS DMS (new replica), RDS events — they can all publish directly to SNS.

![SNS integrations](images/l14-09-sns-integrations.jpg)

**Use cases for Amazon SNS**

- CloudWatch **alarm notifications**
- **Email and SMS** messages for a mailing list
- **Push notifications** for app updates

![SNS use cases](images/l14-10-sns-use-cases.jpg)

### Huawei Cloud equivalent: SMN

**SMN (Simple Message Notification)** is Huawei Cloud's version of SNS.

- A **reliable and flexible large-scale message notification service.**
- Sends messages to endpoints such as **phone numbers and email addresses.**
- Uses a **publish/subscribe model** for **one-to-many** subscriptions and notifications.
- Can integrate with multiple cloud services for automation.

![What is SMN](images/l14-13-what-is-smn.jpg)

**SMN architecture — two roles:**

- **Publisher:** publishes messages to a **topic.** Publishers can be cloud services such as CES (Cloud Eye), OBS or AS (Auto Scaling).
- **Subscriber:** SMN then **delivers** the messages to subscribers in the topic. Subscribers can be **email addresses, phone numbers, URLs** (HTTP/HTTPS) or **FunctionGraph.**

![SMN architecture](images/l14-14-smn-architecture.jpg)

| AWS | Huawei Cloud |
|---|---|
| SNS | **SMN** |
| SNS topic | SMN topic |
| Email / SMS / HTTP(S) / Lambda subscribers | Email / SMS / HTTP(S) / FunctionGraph subscribers |

## 5. SNS + SQS fan-out

**Fan-out** = push **once** into an SNS topic and have the message delivered to **every SQS queue** subscribed to it. Each queue feeds a different service, and they all work **in parallel.**

**Example:** an online order arrives → SNS topic → two parallel queues:
- one queue → **Order Processing / Shipping service**
- one queue → **Analysis / Fraud service** (for example, checking for blacklisted customers)

![SNS publish to multiple SQS queues](images/l14-11-sns-publish-to-multiple-sqs.jpg)

**Why fan-out is powerful**

- ✅ Push once in SNS, **receive in all subscribed SQS queues**
- ✅ **Fully decoupled, no data loss**
- ✅ SQS adds **data persistence, delayed processing and retries** of work
- ✅ You can **add more SQS subscribers over time**
- ⚠️ Make sure the **SQS queue access policy allows SNS to write** to it

![SNS + SQS fan-out](images/l14-12-sns-sqs-fanout.jpg)

## 6. Monitoring: Huawei Cloud Eye vs AWS CloudWatch

**Cloud Eye** is Huawei Cloud's **monitoring service** — it tracks metrics such as **CPU, network and disk** for your resources. **Alarm rules** define a threshold, and Cloud Eye raises an alarm when a metric crosses it. **SMN** then delivers those alarms to you **by email.**

**Lab: Cloud Eye Basics & Practices** — *monitor Elastic Cloud Server (ECS) metrics with Cloud Eye, then alert by email through SMN.*

![Cloud Eye lab overview](images/l14-15-cloud-eye-lab-overview.jpg)

**Lab flow**

1. **Provision an ECS** — the resource to watch.
2. **Install the agent** — it collects metrics from *inside* the server.
3. **Watch metrics** — CPU, network and disk in real time.
4. **Set an alarm** — it fires when a metric (e.g. CPU) crosses your threshold.
5. **Notify by email** — SMN sends the alert.

![What Cloud Eye does](images/l14-16-what-cloud-eye-does.jpg)

## 7. Log analytics with Splunk

**The problem:** servers, firewalls, applications and databases all **generate logs.** There should be **one place to collect the logs** and evaluate them.

**The solution: Splunk** — *turning raw machine data into useful insights.* It works as a **log collector → indexer → search → insight** tool. If a security issue occurs, you start the investigation by checking the **login logs.**

Example Splunk search (SPL):
```
index=security action=failure | stats count
```

![Splunk fundamentals](images/l14-17-splunk-fundamentals.jpg)

**Data sources — "if it produces data, Splunk can ingest it"**

| Source | Examples |
|---|---|
| Log files | Web, app, OS |
| Network | Firewall, DNS, flows |
| Metrics | CPU, memory, latency |
| Config files | Changes and drift |
| APIs | REST, data streams |
| Cloud | AWS, Azure, Microsoft 365 |

![Splunk data sources](images/l14-18-splunk-data-sources.jpg)

**Use cases — "ask any question of your data"**

| Area | What you do |
|---|---|
| **IT Ops** | Troubleshoot outages and crashes |
| **Security** | Detect threats and fraud, prove compliance |
| **Apps** | Track errors and response times |
| **Business** | Understand customers and sales |
| **IoT** | Monitor sensors and machines |

![Splunk use cases](images/l14-19-splunk-use-cases.jpg)

**From data to insight — what the data shows → what it means**

| Area | Pattern in the data | Root cause / action |
|---|---|---|
| Security | HTTP **401 errors spike** from one IP | Credential **brute-force attack** |
| E-commerce | Carts **abandoned at /payment** | **Payment gateway API timeouts** |
| Operations | CPU I/O wait **peaks at 02:00** | **Backup job** saturating disk I/O |
| Sales | Buyers of A **often buy B** | Offer a **"bought together" bundle** |

![From data to insight](images/l14-20-splunk-data-to-insight.jpg)

**What Splunk gives you:** **Log management** (collect and organize all logs) · **Search & analytics** (search, filter and correlate with SPL) · **Real-time alerts** (get notified the moment something happens) · **Dashboards** (charts and reports).

![What Splunk gives you](images/l14-21-splunk-key-features.jpg)

---

# Lecture 15 — Global Services, CDN & Infrastructure as Code

## 1. Regional vs global services

Cloud services fall into two scopes:

| Scope | Meaning | Examples |
|---|---|---|
| **Regional service** | Visible only in the **Region where you created it** | **EC2** |
| **Global service** | Not tied to one Region | **IAM** (Identity & Access Management), **S3** (object storage), **CloudFront** (CDN) |

> 💡 For S3, the **console and bucket names are global**, but each bucket's data is stored in the Region you choose.

## 2. Content Delivery Network (CDN) & Amazon CloudFront

**CDN = caching.** A cloud that provides a CDN is a mature cloud.

> A **Content Delivery Network** is a network of servers located in different places that **delivers website content to users faster.**

**How it works**

- The **first request** goes to the **origin server.**
- After that, the **CDN handles** the requests from its cached copy.

**Benefits**

| Benefit | Meaning |
|---|---|
| ⚡ **Faster** | Content is served from a location close to the user |
| 📉 **Offloads the origin** | Less load on your server |
| 🔒 **Secure** | You can put a **WAF** (Web Application Firewall) in front |

**Local example:** Pakistan has **3 Cloudflare edge servers** (Lahore, Karachi, Islamabad).

**Edge locations are interconnected:** when a user anywhere requests content, the response is cached at that edge, and the lecture explained that the other edges in the network also get updated, so another user can access the same data instantly. *(In practice, caching behavior differs by CDN provider, but the idea is that content is placed close to many users.)*

**DDoS mitigation example:** put the **WAF at the very first layer**, so attack traffic is filtered at the edge before it reaches your servers.

**Origin:** the "origin" is where the original content lives — for example an **S3 bucket** (the **cloud origin**).

**S3 as an origin:** CloudFront has its own **identity**, and you must **grant that identity access on the S3 side.** Attach the identity to the bucket when you set it up, so only CloudFront can read the content.

## 3. Lab: CloudFront with an S3 origin

> ⚠️ **Do not run this lab in a trial account** — CloudFront may be restricted there.

**Step 1 — Create the S3 bucket**
1. Open **S3** → **Create bucket.**
2. Give it a **globally unique name.**
3. Keep **Block all public access** turned **ON.**
4. Click **Create bucket.**

**Step 2 — Upload content**
1. Open the bucket → **Upload** → **Add files.**
2. Upload some images from your local machine.

**Step 3 — Create the CloudFront distribution**
1. Open **CloudFront** in a new tab (it's a **global** service, so the Region selector at the top shows *Global*).
2. Click **Create distribution.**
3. **Name:** `MyCloudFrontDistribution` · keep the defaults → **Next.**
4. **Origin type:** Amazon S3 → **S3 origin:** *Browse* and choose the bucket you created. (You can also customize the origin settings.) → **Next.**
5. **Enable WAF** → **Next.**
6. Do **not** enable additional security protection → **Next.**
7. Click **Create distribution.** ⏱️ It takes about **5 minutes** to deploy.

**Step 4 — Check the bucket permissions**
- Go back to **S3 → your bucket → Permissions.** A **bucket policy** has been added automatically — it's the link that grants the CloudFront distribution access to the bucket.

**Step 5 — Test**
- Open the CloudFront **distribution domain name** and request your image. It's now served through the CDN, while the bucket itself stays private.

## 4. AWS CloudFormation — Infrastructure as Code

**Infrastructure as Code (IaC)** = **you only run code, and the code builds the infrastructure for you.**

- CloudFormation is AWS's IaC service.
- It is comparable to **Terraform.** Terraform is **open source** and works on **any cloud.**

**Why IaC?** Without it, building infrastructure means **a lot of manual work that is hard to reproduce** — in **another Region**, in **another AWS account**, or even in the **same Region if everything was deleted.**

| Benefit | Meaning |
|---|---|
| ♻️ **Repeatable** | Rebuild the same setup in any Region or account |
| 🆘 **Disaster recovery** | Just **run the code again** to rebuild everything |
| 📝 **Documented** | The template *is* the documentation of your infrastructure |

> The CloudFormation template and hands-on lab continue in the Lecture 16 notes.

## 5. AWS Elastic Beanstalk

> **When to use it:** you only want to **write code**, and you don't want to deal with infrastructure concerns like security groups, load balancers and auto scaling.

**Elastic Beanstalk** is a **Platform as a Service (PaaS)** — *deploy web apps without managing infrastructure.*

| # | | |
|---|---|---|
| 1 | **What it is** | **Upload your code** — Beanstalk deploys and runs it. Supports Java, .NET, Node.js, Python, Go, Docker… |
| 2 | **What it manages** | Automatically provisions **EC2, Elastic Load Balancing, Auto Scaling and health monitoring** |
| 3 | **Scales automatically** | Adds or removes EC2 instances as load changes |
| 4 | **You stay in control** | Override the defaults any time — e.g. pick a larger EC2 instance type |

💰 **Cost:** Beanstalk itself is **free** — you only pay for the underlying services it creates (EC2, load balancer, etc.).

**How it works:** you create an **environment**, **upload your code**, and Beanstalk builds the infrastructure to match.

![AWS Elastic Beanstalk](images/l15-01-elastic-beanstalk.jpg)

### Elastic Beanstalk vs CloudFormation

| | **CloudFormation** | **Elastic Beanstalk** |
|---|---|---|
| You write | **Code for the infrastructure** (a template) | **Only your application code** |
| Situation | You want to define the infrastructure itself | You've built an app and just want infrastructure for it |
| Relationship | — | Beanstalk **uses CloudFormation behind the scenes** |

### Two types of environments

| | **Web server environment** | **Worker environment** |
|---|---|---|
| Purpose | Handles **requests coming from users** (the front end) | Handles **any task that takes time** — **background** work |
| Scaling | **Auto scaling and load balancing** sit in front | Scales on the work waiting in the queue |
| Speed | Must respond quickly | Runs **long-running / background jobs** |
| Gets a public URL? | ✅ **Yes** | ❌ **No** |
| How it gets work | HTTP requests | **Pulls jobs from an Amazon SQS queue** |
| Extras | — | Supports **scheduled jobs** |

> This ties back to Lecture 14: the **worker environment is the SQS consumer** and the **web server environment is the producer.**

## 6. Lab: Elastic Beanstalk

1. Open **Elastic Beanstalk.**
2. Choose **US East (N. Virginia)** — services are most mature there and labs fail less often in other Regions.
3. Click **Create application / Deploy application → Create environment.**
4. Choose the **Web server environment** (standard).
5. **Name:** `Mytestapp` · **Platform:** **Node.js** · keep the defaults → **Create.**
6. Wait for the environment to launch, then open its **public URL.**

**Good to know:** an Elastic Beanstalk **application** is a container that can hold **multiple environments** (for example dev, test and production) that you can manage together.

**Clean up:** **delete the application** when you're done, so the EC2 instances and load balancer stop costing money.

---

# 🎯 Key Takeaways

1. **Don't let components depend directly on each other.** Synchronous calls fail under sudden load — orders get dropped.
2. **In the cloud, always use a queue.** **SQS** buffers requests, absorbs spikes and tolerates failures.
3. **Scale on queue length:** a CloudWatch / Cloud Eye alarm on `ApproximateNumberOfMessages` triggers the Auto Scaling Group.
4. **SNS / SMN = publish once, notify many.** It takes the notification burden off your main application.
5. **Fan-out (SNS + SQS)** delivers one message to many queues — fully decoupled, with no data loss.
6. **Cloud Eye ↔ CloudWatch** monitor; **SMN ↔ SNS** notify.
7. **Splunk** collects logs from everywhere and turns them into insight — from brute-force detection to "bought together" bundles.
8. **EC2 is regional; IAM, S3 and CloudFront are global.**
9. A **CDN** caches content close to users: **faster, less origin load, and more secure with a WAF.**
10. **CloudFormation = Infrastructure as Code** — repeatable builds and easy disaster recovery. **Elastic Beanstalk = just upload code** and let AWS build the infrastructure (using CloudFormation underneath).

---

# 📝 Quick Revision Cheat Sheet

| Topic | Remember |
|---|---|
| Synchronous | App → App, breaks under load |
| Asynchronous | App → **Queue** → App |
| SQS | Queue: producers send, consumers poll |
| Dead-letter queue | Holds messages that can't be processed |
| Scaling on a queue | Alarm on `ApproximateNumberOfMessages` → ASG scale out |
| SNS / SMN | Pub/sub topic → email, SMS, HTTP, Lambda/FunctionGraph |
| Fan-out | 1 SNS topic → many SQS queues |
| Cloud Eye ↔ CloudWatch | Monitoring + alarms |
| Splunk | Log collector → index → search → insight (SPL) |
| Regional service | EC2 |
| Global services | IAM, S3 (console), CloudFront |
| CDN benefits | Faster · offloads origin · secure (WAF) |
| Origin | Where the original content lives (e.g. S3) |
| CloudFormation | IaC; like Terraform (Terraform is open source, multi-cloud) |
| Beanstalk | PaaS — upload code, AWS builds infra; free, pay for resources |
| Web vs Worker env | Public URL + fast responses / background jobs from SQS, no URL |

---

## 📸 Source note

Notes are based on class lecture slides, whiteboard explanations and lab walkthroughs (AWS Training material, Huawei Cloud course material and Splunk Fundamentals slides) from the HCCDP Solution Architect course. Slide images are photographed from the lecture screen and are included for study and reference.

---

*Part of my cloud computing learning journey — follow along for the next weeks! ☁️*

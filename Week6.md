# ☁️ Week 6 — Storage & Databases in the Cloud

> **Course:** HCCDP – Huawei Cloud Computing & Data Analytics Professional (Solution Architect learning path)
> **Covers:** Lecture 12 (Data Storage: DAS, NAS, SAN, Block/File/Object) · Lecture 13 (Database Services)
> **Clouds touched:** Huawei Cloud (EVS, OBS, SFS, RDS, GaussDB) with AWS comparisons

Every application needs two things: **somewhere to keep its files** and **somewhere to keep its data in an organized, queryable form.** This week covered both:

1. **Storage** — how data is physically and logically stored, and the three cloud storage types (block, file, object).
2. **Databases** — relational vs non-relational, OLTP vs OLAP, ACID, managed vs unmanaged services, and how to pick the right database.

---

## 📑 Table of Contents

- [Lecture 12 — Data Storage](#lecture-12--data-storage)
  - [What is data storage?](#1-what-is-data-storage)
  - [Storage system basics](#2-storage-system-basics-controller-disks-jbod)
  - [DAS, NAS and SAN](#3-das-nas-and-san)
  - [Storage evaluation criteria](#4-storage-evaluation-criteria)
  - [HDD vs SSD](#5-hdd-vs-ssd)
  - [Block vs File vs Object storage](#6-block-vs-file-vs-object-storage)
  - [Lab: Huawei Cloud storage (EVS, OBS, SFS)](#7-huawei-cloud-storage-evs-obs-sfs)
  - [Quick VPC recap](#8-quick-recap-what-is-a-vpc)
- [Lecture 13 — Database Services](#lecture-13--database-services)
  - [What is a database?](#1-what-is-a-database)
  - [Relational databases](#2-relational-databases)
  - [Non-relational (NoSQL) databases](#3-non-relational-nosql-databases)
  - [OLTP vs OLAP](#4-oltp-vs-olap)
  - [ACID](#5-acid-what-makes-a-transaction-reliable)
  - [Managed vs unmanaged services](#6-managed-vs-unmanaged-database-services)
  - [Choosing the right database](#7-choosing-the-right-database)
  - [RTO, RPO and HA](#8-rto-rpo-and-ha)
  - [Huawei Cloud database portfolio](#9-huawei-cloud-database-portfolio)
  - [RDS architecture & MySQL features](#10-rds-architecture--rds-for-mysql-features)
  - [Lab: RDS for MySQL](#11-lab-rds-for-mysql)
- [Key Takeaways](#-key-takeaways)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)

---

# Lecture 12 — Data Storage

## 1. What is data storage?

Storage can be understood in two ways:

- **Narrow sense** — the physical media you can hold: CDs, DVDs, zip disks, external drives, hard disks.
- **Broad sense** — the whole storage *ecosystem*: storage arrays, racks, adapter cards, cables and the network (switches, Fibre Channel) that connects servers to storage.

![What is data storage](images/l12-01-what-is-data-storage.jpg)

**Types of storage media**

| Category | Examples |
|---|---|
| **Magnetic** | HDD (magnetic disks & platters), external hard drive, floppy disk (legacy), magnetic tape (high-capacity backup) |
| **Solid state** | SSD — fast solid-state memory; NVMe SSD uses the ultra-fast NVMe protocol |
| **Removable solid state** | USB flash drive (pen drive), SD card (cameras), microSD card (phones & drones), CompactFlash card (pro photography) |
| **Optical** | CD (basic optical disc), DVD (video & data), Blu-ray (high-definition video) |
| **Network & memory** | NAS (centralized network storage), cloud storage (remote), RAM (working memory, volatile), ROM (permanent system data, non-volatile) |

![Storage media: solid state and removable](images/l12-02-storage-media-ssd-removable.jpg)

![Optical, network and memory storage](images/l12-03-optical-network-memory.jpg)

## 2. Storage system basics: Controller, Disks, JBOD

A **storage system** is a **controller** that receives read/write requests and spreads the work across several **disks**.

- **JBOD = "Just a Bunch Of Disks"** — disks put together without a smart controller or RAID layer.
- A **storage array / storage system** adds a controller in front of the disks.

![Storage system: controller and JBOD](images/l12-04-storage-system-controller-jbod.jpg)

**Initiator and target** (from the whiteboard):

- The **CPU / host PC / server** that sends the read/write commands is the **Initiator**.
- The **HDD / storage array / JBOD** that serves them is the **Target**.
- Protocols used between them: **SCSI**, **SATA / SAS**, and **NVMe** (for SSDs).
- Parallel SCSI (older) evolved into **Serial Attached SCSI (SAS)**.

![Initiator, target and protocols](images/l12-05-initiator-target-protocols.jpg)

## 3. DAS, NAS and SAN

### DAS — Direct Attached Storage

Storage is **directly attached to one server**. Other computers reach that data **through the server** over the LAN/WAN (using file-sharing protocols such as **NFS** or **CIFS**).

- Simple and fast for one server
- ❌ Hard to share between servers, and storage is stranded inside each server

![DAS](images/l12-06-das-direct-attached.jpg)

![Servers each with their own disks](images/l12-07-das-servers-and-disks.jpg)

### NAS — Network Attached Storage

- **SCSI cables are replaced by a network.**
- **Storage devices exist independently of servers** — they sit on the LAN (via an Ethernet switch) and are shared by many clients.
- Shares **files** over the network.

![NAS](images/l12-08-nas-network-attached.jpg)

![NAS with Ethernet switch](images/l12-09-nas-ethernet-switch.jpg)

### SAN — Storage Area Network

A dedicated high-speed network that connects **servers to a storage array through SAN switches**.

- Uses **Fibre Channel** — an open technical standard for connecting servers to storage in a SAN.
- Carries **SCSI commands using FCP (Fibre Channel Protocol)**.
- Combines the **channel transport** characteristics of an I/O bus with the **flexible connectivity and distance** of a traditional network.
- Hosts and applications see SAN storage as **locally attached** disks.
- Can use both **fibre and copper** cables.
- Communication happens between **nodes** — the **initiator** and the **target**.

![SAN and Fibre Channel](images/l12-10-san-fibre-channel.jpg)

| | DAS | NAS | SAN |
|---|---|---|---|
| Connection | Directly to one server | Over the LAN (Ethernet) | Dedicated storage network (Fibre Channel) |
| Data presented as | Block | **File** | Block |
| Shared by many servers? | No | Yes | Yes |
| Typical protocol | SCSI / SATA / SAS | NFS / CIFS | FCP (SCSI over Fibre Channel) |

## 4. Storage evaluation criteria

How do we judge a storage system? Two main measures:

1. **IOPS** — *Input/Output Operations Per Second.* Every **write** is an *input* and every **read** is an *output*; IOPS counts how many of these the storage can handle each second. **Higher is better.**
2. **Bandwidth** — how much data can be transferred per second (throughput).

![Storage evaluation criteria](images/l12-11-storage-evaluation-iops.jpg)

## 5. HDD vs SSD

| | HDD (3.5") | SSD (2.5") |
|---|---|---|
| Main parts | Platters, spindle, read/write head, actuator arm, actuator axis, actuator | NAND flash memory, controller, cache |
| Moving parts | **Yes** | **No** |
| Shock resistance | Up to **55g** operating, **350g** non-operating | Up to **1500g** (operating and non-operating) |

SSDs have no moving parts, so they are far more shock resistant and much faster.

![HDD vs SSD](images/l12-12-hdd-vs-ssd.jpg)

## 6. Block vs File vs Object storage

This is the most important idea of the lecture. The class whiteboard compared the three types:

| | **Block** | **File** | **Object** |
|---|---|---|---|
| How data is stored | Data divided into **blocks** (multiples of 512 bytes) | The **complete file**, in a directory tree | Data saved **as an object** |
| Structure | Raw blocks | Hierarchy of directories and sub-directories | **Flat structure** with a unique object ID |
| Each item holds | Just the data blocks | File + path | **Data** (e.g. an mp3) + **metadata** + **unique ID** |
| Best for | **Structured data**, databases, operating systems | **File sharing** | **Unstructured data** (video, images, documents), websites, backups |
| AWS example | Amazon **EBS** (an EC2 instance's drive) | Amazon **EFS** | Amazon **S3** / Glacier |

**Data vs information:** data can be **structured** (rows and columns, like a table) or **unstructured** (photos, videos, documents).

![Block, File and Object whiteboard](images/l12-13-block-file-object-whiteboard.jpg)

**Easy way to remember:**

| Type | Can you boot an OS from it? | Can you mount it? | Typical access |
|---|---|---|---|
| **Block** | ✅ Yes — bootable, good for databases | ✅ Yes | DAS / SAN |
| **File** | ❌ Not bootable | ✅ Mountable | NAS |
| **Object** | ❌ Not bootable | ❌ Not mountable | Accessed over the **Internet** |

![Block, File and Object summary](images/l12-14-block-file-object-summary.jpg)

**Amazon EBS (Elastic Block Store)**

- Block-level storage for **EC2 instances**.
- Can be used for the **OS and databases**.
- One or more EBS volumes can be attached to a **single** EC2 instance.
- **Detachable** — you can move EBS volumes between EC2 instances as needed.
- Volumes live in one Availability Zone (the slide shows volumes of 10, 100 and 50 GB in `us-east-1a`, and one unattached 10 GB volume in `us-east-1b`).

![Amazon EBS](images/l12-18-amazon-ebs.jpg)

## 7. Huawei Cloud storage: EVS, OBS, SFS

A hands-on lab covered the **three Huawei Cloud storage types**:

![Huawei storage lab overview](images/l12-15-huawei-storage-lab.jpg)

| Service | Type | What it is | Good for |
|---|---|---|---|
| **EVS** — Elastic Volume Service | **Block** | A **virtual hard disk attached to one server.** Partition, format and mount it like a physical SSD. | Databases, boot disks, low-latency data |
| **OBS** — Object Storage Service | **Object** | **Buckets** that hold files (objects), reached over the web by API. Hugely scalable, pay for what you use. | Backups, media, static web content |
| **SFS** — Scalable File Service | **File** | A **shared file system** that many servers mount at once over **NFS**. Everyone sees the same files. | Shared workspaces, content across a fleet |

> All three run in the **same region and VPC** and attach to your **ECS** cloud servers.

![Three kinds of cloud storage](images/l12-16-three-kinds-of-cloud-storage.jpg)

**Part 1 — Block storage (EVS):** persistent block storage for cloud servers. **Buy a disk → attach it to an ECS → initialize and use it** — on both **Windows and Linux**.

![EVS — Elastic Volume Service](images/l12-17-evs-block-storage.jpg)

## 8. Quick recap: What is a VPC?

The lecture closed with an overview of the **Virtual Private Cloud** from last week.

> A VPC is a **private section of a cloud provider's network that you control.** It's *"virtual"* because it's software-defined, and *"private"* because only you can use that isolated space.

**You decide three things:**

1. **IP address ranges** — which private IP ranges (CIDR blocks) the network may use.
2. **Internal connectivity** — how servers, databases and services inside it talk to each other.
3. **Public vs private** — what connects to the public internet and what stays isolated.

![VPC overview](images/l12-19-vpc-overview.jpg)

**Simple analogy:** the public cloud (AWS, Azure, Google Cloud) is a **shared building** used by everyone. Your VPC is **your own locked unit** inside it — **only you hold the key.**

![VPC analogy](images/l12-20-vpc-analogy.jpg)

---

# Lecture 13 — Database Services

## 1. What is a database?

A **database** is a digital repository where data is **organized so that it can be easily retrieved, manipulated and stored.**

> **Why do we need a database?** To **retrieve data efficiently.**

**Common operations — CRUD:**

| Letter | Operation | Meaning |
|---|---|---|
| **C** | Create | Add new records |
| **R** | Read | Retrieve records |
| **U** | Update | Modify existing records |
| **D** | Delete | Remove records |

**Queries:** you work with data using a query language like **SQL**. For example:
- Retrieve all customers from a certain city.
- Find all products with prices greater than $100.

![Databases overview](images/l13-01-databases-overview.jpg)

## 2. Relational databases

- Look just like **Excel spreadsheets, with links between them.**
- Use the **SQL** language for queries / lookups.
- **All fields must be filled.**
- **Best suited for OLTP.**
- Examples: **MySQL, PostgreSQL, Oracle, SQL Server, IBM DB2.**

The slide's hospital example shows tables (Doctor, Hospital, Patient, Visit, Medical Treatment) linked by IDs.

![Relational database](images/l13-02-relational-database.jpg)

**Relationships use keys:**

- **PK = Primary Key** — uniquely identifies each row in a table.
- **FK = Foreign Key** — a column that points to the primary key of another table.
- Relationships can be **one-to-many** (1 → ∞), as in the movie-rental example (Member → Rental → Movie_Rental ← Movie).

![Relational database relationships](images/l13-03-relational-relationships.jpg)

**Example table:**

| id | name | price | quantity | department |
|---|---|---|---|---|
| 1 | Whole Milk | 1.99 | 12 | Dairy |
| 2 | Almonds | 3.39 | 112 | Nuts |
| 3 | Chocolate Bar | 0.99 | 99 | Candy |

![Example of relational database](images/l13-05-relational-example-table.jpg)

## 3. Non-relational (NoSQL) databases

**NoSQL = "non-SQL" = non-relational.**

- Purpose-built for **specific data models**, with **flexible schemas** for building modern applications.
- Store data in a more flexible format: **key-value pairs, documents or graphs.**
- Per the lecture slide, best suited for **analytical** workloads.

**Benefits**

| Benefit | Meaning |
|---|---|
| **Flexibility** | Easy to evolve the data model |
| **Scalability** | Designed to **scale out** using distributed clusters (horizontal scaling) |
| **High performance** | Optimized for a specific data model; faster reads/writes than relational DBs |
| **Highly functional** | Types optimized for the data model |

**Examples:** MongoDB, Cassandra, DynamoDB, Redis — covering key-value, document, graph, in-memory and search databases.

![Types of databases](images/l13-04-types-of-databases.jpg)

![Non-relational databases](images/l13-06-non-relational-databases.jpg)

| | Relational (SQL) | Non-relational (NoSQL) |
|---|---|---|
| Data format | Tables with rows and columns | Key-value, document, graph… |
| Schema | Fixed, all fields filled | Flexible |
| Query language | SQL | Varies by database |
| Scaling | Mostly scale **up** | Scale **out** (distributed) |
| Examples | MySQL, PostgreSQL, Oracle, SQL Server | MongoDB, Cassandra, DynamoDB, Redis |

## 4. OLTP vs OLAP

- **OLTP — Online Transaction Processing:** you enter **one transaction and the complete information for it is fetched.** It pulls up **one row of data.** Ideally suited for **relational databases.**
  - *Example:* look up **Order Number 2120121** → returns Name, Date, Delivery address, Delivery status, etc.

- **OLAP — Online Analytical Processing:** you need **analysis over a large number of records.** It pulls in **large numbers of records.**
  - *Example:* *"Calculate the net profit of the Digital Radio product for the **EMEA** and **Pacific** regions."* This requires:
    - Sum of radios sold in EMEA
    - Sum of radios sold in Pacific
    - Unit cost of the radio in each region
    - Sales price of each radio
    - Sales price − unit cost
  - The lecture noted that non-relational databases suit this kind of workload.

![OLTP vs OLAP](images/l13-07-oltp-vs-olap.jpg)

| | OLTP | OLAP |
|---|---|---|
| Purpose | Day-to-day transactions | Analysis and reporting |
| Query touches | **One row / a few rows** | **Large numbers of records** |
| Example | Find order #2120121 | Net profit for 2 regions |
| Database type (per lecture) | Relational | Non-relational / analytical |

> 💡 **Extra note:** in Huawei Cloud's portfolio (see below), OLAP is served by a dedicated data warehouse service, **GaussDB(DWS)**, while OLTP is served by RDS / GaussDB.

## 5. ACID: What makes a transaction reliable

ACID is the set of four guarantees that make database transactions trustworthy. The running example is **transferring $100 from Account A to Account B.**

| Letter | Property | Meaning | In the $100 transfer | Enforced by |
|---|---|---|---|---|
| **A** | **Atomicity** | **All or nothing.** Either all operations in the transaction complete, or none of them do. | Debit A **and** credit B both happen — or neither does. | Undo log & rollback |
| **C** | **Consistency** | Data moves from one **valid state** to another; all rules hold. | If we transfer $100, exactly $100 moves — not less, not more (**no partial transaction**). Total money stays the same; no negative balance. | Constraints & keys |
| **I** | **Isolation** | When multiple transactions run at the same time, one must **not interfere** with another's work. | No one sees $100 gone from A but not yet in B. | Locks / MVCC |
| **D** | **Durability** | Once a transaction is **committed**, the data is **permanently saved**. | Once confirmed, a crash can't undo the transfer. | Write-ahead log |

**Durability example:** you transfer **Rs. 2,000** at the bank and the transaction is successfully **committed.** Even if the system crashes or the power fails right afterwards, the record of the transfer **stays safe in the database.**

![ACID](images/l13-10-acid.jpg)

## 6. Managed vs unmanaged database services

- **Managed service:** **Huawei RDS** is a **ready-made, managed** database service.
- **Unmanaged:** installing a database yourself on an **ECS** (infrastructure) server.

**Who is responsible for what?**

| Layer | DB installed on **ECS** (unmanaged) | **Huawei RDS** (managed) |
|---|---|---|
| Application optimization | You | You |
| Scaling | You | **Huawei** |
| High availability | You | **Huawei** |
| Database backups | You | **Huawei** |
| Database software patches & installs | You | **Huawei** |
| OS patches | You | **Huawei** |
| OS installation | Huawei | Huawei |
| Server maintenance, rack & stack | Huawei | Huawei |
| Power, HVAC, networking | Huawei | Huawei |

With RDS, **you only optimize your application** — the cloud provider handles everything below it.

![Managed and unmanaged services](images/l13-08-managed-vs-unmanaged.jpg)

**Cloud vs other database solutions**

| Approach | What you pay for / do |
|---|---|
| **On-premises database** | Server procurement, hardware and OS deployment · equipment-room hosting fees · **high DBA costs** |
| **Database built on ECS** | Purchase and install database software · ECS rental fees · **high DBA costs** |
| **Cloud database (RDS)** | **No need to buy or install** any software or hardware · **just pay for the database** · **focus on architecture design and performance optimization** |

![Cloud vs other database solutions](images/l13-09-cloud-vs-other-db-solutions.jpg)

## 7. Choosing the right database

After this lecture you can say you know how to **implement a database both on-premises and in the cloud.** When recommending a database, ask these **six questions:**

| # | Question | What to ask |
|---|---|---|
| 1 | **Workload** | Is your application **read-heavy or write-heavy**? (Video streaming or a blog is read-heavy — users only *view* data, like a newspaper you can read but never change.) Is traffic **steady or spiky**? |
| 2 | **Data size** | What is the data volume and retention? What is the **record size**, and is it **growing**? (1 MB today could be 10 MB tomorrow — **always ask the client** when proposing a database solution.) |
| 3 | **Data model** | **Structured → SQL.** **Flexible → NoSQL.** Do you need joins or full-text search? |
| 4 | **Performance** | What **latency target** (in ms) can the application tolerate? How many **concurrent users** (people using it at the same time)? What throughput is needed? |
| 5 | **Reliability** | Is the source of truth **reliable**? Do you need ACID / consistency? What are the **HA / RPO / RTO** targets? |
| 6 | **Cost** | License fees? Would a cloud-native option like **GaussDB** be cheaper? |

**Match the engine to the workload:** RDS · GaussDB · DDS · GeminiDB · DCS · CSS

![Choosing the right database](images/l13-11-choosing-the-right-database.jpg)

## 8. RTO, RPO and HA

Three terms that describe how resilient a database must be:

### RTO — Recovery Time Objective ⏱️
**How quickly must the system be back up after a failure?** (The longest outage you can bear.)

> *Example:* RTO = **2 hours** → after a failure, the system must be restored **within 2 hours.**

### RPO — Recovery Point Objective 💾
**How much data can we afford to lose?** (The maximum data loss you can tolerate.)

> *Example:* RPO = **30 minutes** → if the system crashes, at most **30 minutes** of data may be lost.
>
> *Backup example:* if a backup runs every hour, say at 11:00, and the system crashes at 11:59, you lose **59 minutes** of data. If backups only run every 12 hours, you could lose up to **12 hours** of data.

### HA — High Availability 🟢
**Keep the system running and available as much as possible,** usually with multiple servers or redundancy.

> *Example:* if one server fails, another server **automatically** takes over the requests, and users keep getting service.

| Term | Remember it as | Question it answers |
|---|---|---|
| **RTO** | **Time** ⏱️ | How fast do we recover? |
| **RPO** | **Data** 💾 | How much data can we lose? |
| **HA** | **Availability** 🟢 | How do we keep running? |

## 9. Huawei Cloud database portfolio

> **GaussDB** — Huawei took an **open-source database** and **customized** it to build its own database. GaussDB is a Huawei-developed database designed to meet the high reliability and performance requirements of governments and enterprises.

| Category | Services |
|---|---|
| **Relational — OLTP** | GaussDB (for openGauss), GaussDB (for MySQL), RDS for MySQL, RDS for PostgreSQL |
| **Relational — OLAP** | GaussDB (DWS) |
| **Non-relational** | DDS Community Edition, GaussDB (for Mongo), GaussDB (for Cassandra), GaussDB (for Redis), GaussDB (for Influx) |
| **Database tools** | **DDM** — Distributed Database Middleware · **DRS** — Data Replication Service · **UGO** — Database and Application Migration · **DAS** — Data Admin Service |

![Huawei Cloud database portfolio](images/l13-12-huawei-database-portfolio.jpg)

## 10. RDS architecture & RDS for MySQL features

**RDS (Relational Database Service)** is a reliable and scalable cloud database service that is **easy to manage.**

**Advantages**

- ✅ **High availability and durability**
- ✅ **More flexibility and lower upfront costs**
- ✅ **Optimized for performance and security**
- ✅ **Simple deployment and maintenance**

The architecture shows a **database cluster** with a **primary DB instance**, a **standby DB instance** (for failover) and **read replicas**, with data replicated between them. Admins manage everything from the **RDS console** (create/start/stop/scale/back up/restore instances, monitor metrics, manage logs and parameters), while developers and users just connect to the database.

![RDS architecture and advantages](images/l13-13-rds-architecture-advantages.jpg)

### Feature 1 — Cross-AZ High Availability

**Write on the primary, read from the replicas.**

- **Primary/standby** instances run across **different Availability Zones.**
- Cross-AZ HA supports **switchover in seconds.**
- Up to **5 read replicas** can be created to **offload read traffic.**
- The **standby instance is invisible** to users — they connect through a **virtual IP address.**
- **Read replicas cannot exist alone** — they must come with a single instance or a primary/standby pair.

![RDS for MySQL cross-AZ HA](images/l13-14-rds-mysql-cross-az-ha.jpg)

### Feature 2 — Read/Write Splitting

A **proxy with one unified address** sends **write requests to the primary** and **read requests to the read replicas.**

| Functions | Advantages |
|---|---|
| A single read/write splitting address, **transparent to applications** | **No application rebuild needed** to use read/write splitting |
| **Read-only permissions** can be configured per node | The **read weight** of each read replica is **configurable** |
| **Instance health check:** if a DB instance breaks down or its latency is too high, read requests are no longer sent to it | |

![RDS for MySQL read/write splitting](images/l13-15-rds-mysql-read-write-splitting.jpg)

## 11. Lab: RDS for MySQL

**Huawei Cloud hands-on lab — Database Service Exercise: RDS for MySQL** (Region: AP-Singapore · MySQL 5.7 · 4 tasks)

> **Goal:** buy a managed MySQL instance, then connect to it **three ways** — through **DAS**, over the **private network**, and over the **public network.**

![Lab: database service](images/l13-16-lab-database-service.jpg)

![Lab: RDS for MySQL exercise](images/l13-17-lab-rds-mysql-exercise.jpg)

| Task | What you do | Network setup needed |
|---|---|---|
| **Task 1** | **Buy** an RDS for MySQL DB instance | — |
| **Task 2** | Connect through **DAS** (Data Admin Service) | **None** — it goes through Huawei's own console |
| **Task 3** | Connect over the **private network** from an ECS | ECS and RDS in the **same security group**, so the ECS reaches the RDS private IP directly |
| **Task 4** | Connect over the **public network** | Bind a **public EIP** to RDS and open **port 3306** |

**Instance connectivity architecture**

1. **Same security group** — ECS and RDS share a security group, so the ECS reaches the private IP directly. *(Task 3)*
2. **Different security group** — same VPC, but a different group: add an **inbound rule for port 3306** on the RDS side.
3. **Internet through an EIP** — bind a **public EIP** to RDS and open **3306.** *(Task 4)*

![Instance connectivity architecture](images/l13-18-lab-connectivity-architecture.jpg)

---

# 🎯 Key Takeaways

1. **Storage comes in three flavors:** **Block** (like a raw disk — OS and databases), **File** (shared folders — NAS), **Object** (flat buckets accessed over the internet — backups and media).
2. **DAS** = attached to one server; **NAS** = file sharing over the LAN; **SAN** = dedicated Fibre Channel network that presents block storage.
3. On Huawei Cloud, the three types map to **EVS (block), SFS (file), OBS (object).** On AWS they are **EBS, EFS, S3.**
4. Judge storage by **IOPS** (operations per second) and **bandwidth.**
5. **Relational databases** (SQL) suit **OLTP** — a few rows per query. **Non-relational databases** suit flexible, scalable workloads.
6. **ACID** keeps transactions reliable: **A**ll-or-nothing, **C**onsistent, **I**solated, **D**urable.
7. **Managed services (RDS)** hand the patching, backups, HA and scaling to the cloud provider — **you only optimize your application.**
8. Before choosing a database ask six questions: **workload, data size, data model, performance, reliability, cost.**
9. **RTO = how fast we recover; RPO = how much data we can lose; HA = keep running.**
10. **Write to the primary, read from replicas** — RDS for MySQL supports up to 5 read replicas and read/write splitting.

---

# 📝 Quick Revision Cheat Sheet

| Topic | Remember |
|---|---|
| JBOD | Just a Bunch Of Disks (no smart controller) |
| Initiator / Target | Server sends commands / storage serves them |
| DAS / NAS / SAN | Direct attached / file over LAN / block over Fibre Channel |
| IOPS | Input/Output operations per second — higher is better |
| Block | Bootable, databases, EBS / EVS |
| File | Mountable, not bootable, NAS / EFS / SFS |
| Object | Not bootable, not mountable, internet — S3 / OBS |
| CRUD | Create · Read · Update · Delete |
| PK / FK | Primary key / foreign key |
| OLTP | One row, transactions → relational |
| OLAP | Many rows, analysis (e.g. net profit by region) |
| ACID | Atomicity · Consistency · Isolation · Durability |
| Managed DB | Provider handles patches, backups, HA — you handle the app |
| Six DB questions | Workload · Data size · Data model · Performance · Reliability · Cost |
| RTO / RPO / HA | Time ⏱️ / Data 💾 / Availability 🟢 |
| GaussDB | Huawei's own database, built on an open-source base |
| RDS MySQL | Cross-AZ HA, up to 5 read replicas, read/write splitting |
| RDS lab ports | MySQL port **3306** |

---

## 📸 Source note

Notes are based on class lecture slides, whiteboard sessions and lab walkthroughs (Huawei Cloud course material and AWS Training material) from the HCCDP Solution Architect course. Slide images are photographed from the lecture screen and are included for study and reference.

---

*Part of my cloud computing learning journey — follow along for the next weeks! ☁️*

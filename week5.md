# ☁️ Week 5 — Scaling & Networking in the Cloud

> **Course:** HCCDP – Huawei Cloud Computing & Data Analytics Professional (Solution Architect learning path)
> **Covers:** Lecture 10 (Scalability & Auto Scaling) · Lecture 11 (VPC, NAT, Peering, Transit Gateway)
> **Clouds touched:** AWS + Huawei Cloud (with an Azure comparison)

This week was about two questions every cloud architect has to answer:

1. **How does my application grow (and shrink) automatically when demand changes?** → *Scalability, Elasticity, Auto Scaling*
2. **How do I build a private, secure network around it and connect it to other networks?** → *VPC, Subnets, Gateways, NACLs, Peering, Transit Gateway*

---

## 📑 Table of Contents

- [Lecture 10 — Scalability, Elasticity & Auto Scaling](#lecture-10--scalability-elasticity--auto-scaling)
  - [Vertical vs Horizontal Scalability](#1-vertical-vs-horizontal-scalability)
  - [Scalability vs Elasticity vs Agility](#2-scalability-vs-elasticity-vs-agility)
  - [Auto Scaling Group (ASG)](#3-auto-scaling-group-asg)
  - [Lab A — AWS: ALB + Auto Scaling Group](#4-lab-a--aws-alb--auto-scaling-group)
  - [Lab B — Huawei Cloud: Auto Scaling Basics](#5-lab-b--huawei-cloud-auto-scaling-basics-practice)
- [Lecture 11 — Virtual Private Cloud (VPC)](#lecture-11--virtual-private-cloud-vpc)
  - [VPC fundamentals](#1-what-is-a-vpc)
  - [Subnets, Route Tables & Internet Gateway](#2-subnets-route-tables--internet-gateway)
  - [Lab C — Create a Custom VPC](#3-lab-c--create-a-custom-vpc)
  - [NAT Gateway](#4-nat-gateway)
  - [Security Groups vs NACLs](#5-security-groups-vs-nacls)
  - [VPC Peering](#6-vpc-peering)
  - [Transit Gateway](#7-transit-gateway)
  - [AWS vs Huawei Cloud vs Azure](#8-networking-services--aws-vs-huawei-cloud-vs-azure)
- [Key Takeaways](#-key-takeaways)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)

---

# Lecture 10 — Scalability, Elasticity & Auto Scaling

## 1. Vertical vs Horizontal Scalability

**Vertical scalability (scale up)** = make *one* instance bigger (more CPU, more RAM).

- Example: move from `t2.micro` → `t2.large`.
- Very common for **non-distributed systems**, such as databases.
- Services like **RDS** and **ElastiCache** can scale vertically.
- There is a **hardware limit** — you can't scale up forever.

![Vertical scalability](images/l10-01-vertical-scalability.jpg)

**Horizontal scalability (scale out)** = add *more* instances.

- Implies a **distributed system**.
- Very common for web / modern applications.
- Easy in the cloud thanks to services like **Amazon EC2**.

![Horizontal scalability](images/l10-02-horizontal-scalability.jpg)

| | Vertical (Scale Up) | Horizontal (Scale Out) |
|---|---|---|
| What changes | Size of one instance | Number of instances |
| Typical use | Databases, non-distributed apps | Web apps, modern apps |
| Limit | Hardware ceiling | Practically very high |
| Needs a load balancer? | No | Yes |

A real-world example of both ideas together is this WordPress reference architecture: multiple Auto Scaling Groups across two Availability Zones, an Elastic Load Balancer in front, and managed data services (Aurora, ElastiCache, EFS) behind.

![WordPress reference architecture](images/l10-03-wordpress-reference-architecture.jpg)

## 2. Scalability vs Elasticity vs Agility

These three words get mixed up all the time:

| Concept | Meaning | Key phrases |
|---|---|---|
| **Scalability** | Handle a bigger load | Scale up, scale out, planned capacity growth |
| **Elasticity** | Scale *automatically* with demand | Auto-scaling, grows & shrinks with load, pay-per-use |
| **Agility** | Speed of getting resources | New resources are one click away; provisioning drops from **weeks → minutes** |

> ⚠️ **Agility is not about scaling** — it's about how fast you can get resources. That is a favourite exam distractor.

![Scalability vs Elasticity vs Agility](images/l10-04-scalability-elasticity-agility.jpg)

## 3. Auto Scaling Group (ASG)

**What is Auto Scaling?**

- Launches or terminates instances based on specified conditions → this is what provides **elasticity**.
- Works with a **load balancer** — new instances are registered with it automatically.
- Can launch across **multiple Availability Zones**.
- Can **scale out** (add) and **scale in** (remove).

![What is Auto Scaling](images/l10-05-what-is-auto-scaling.jpg)

**Capacity settings** — every ASG is defined by three numbers:

| Setting | Meaning |
|---|---|
| **Minimum capacity** | The fewest instances that must always run |
| **Desired capacity** | The number the group tries to maintain right now |
| **Maximum capacity** | The most instances the group may ever scale out to |

![ASG capacity](images/l10-06-asg-capacity.jpg)

**ASG + Load Balancer** — the ELB can check the **health** of the EC2 instances, so unhealthy ones get replaced and traffic only goes to healthy ones.

![ASG with load balancer](images/l10-07-asg-with-load-balancer.jpg)

**What goes into an ASG?** A **Launch Template** (it replaces the older, deprecated *Launch Configurations*) plus sizing and policies:

- AMI + Instance type
- EC2 user data
- EBS volumes
- Security groups
- SSH key pair
- IAM roles for the EC2 instances
- Network + subnet information
- Load balancer information
- Min size / Max size / Initial capacity
- **Scaling policies**

![ASG launch template](images/l10-08-asg-launch-template.jpg)

## 4. Lab A — AWS: ALB + Auto Scaling Group

1. Create an **Application Load Balancer** (no EC2 instances yet).
2. Create a **Launch Template**:
   - AMI: Amazon Linux 2
   - Type: `t2.micro`
   - Security group: allow **port 80**
   - User data: install **Apache**
3. Create an **Auto Scaling Group**:
   - **Min 1 · Desired 2 · Max 4**
   - Attach it to the ALB from step 1, then verify.

![Lab ALB + ASG](images/l10-09-lab-alb-asg.jpg)

## 5. Lab B — Huawei Cloud: Auto Scaling Basics Practice

A three-part lab: **configure, scale and stress-test an AS group on Huawei Cloud.**

| Part | Topic |
|---|---|
| **Part 1** | AS configuration and AS group |
| **Part 2** | Scaling policies |
| **Part 3** | Load balancer stability test |

![Huawei AS lab overview](images/l10-10-huawei-as-lab-overview.jpg)

### Part 1 — Basic process

**1.1 Create an AS Configuration** — a template for every ECS the group adds (type, vCPUs, memory, image, disk, login mode).

| Parameter | Value |
|---|---|
| Billing mode | Pay-per-use |
| Region | AP-Singapore |
| Name | `as-config-service01` |
| CPU architecture | x86 |
| Flavors | `s6.small.1`, `s6.medium.2` |
| Flavor policy | Sequenced |
| Image | Public – CentOS 8.0 |
| Disk | EVS – High I/O – 40 GB |
| Security group | `sg-as` |
| EIP | Dynamic BGP – 5 Mbit/s |
| Login mode | Password (complex) |
| ECS group | `SvrGrp-as` – Anti-affinity |

![AS configuration](images/l10-11-as-configuration.jpg)

**Flavor selection policy** — how the group picks a flavor when it needs a new instance:

| Policy | Behaviour |
|---|---|
| **Sequenced** *(used in this lab)* | Tries flavors in the order you selected. If one is unavailable, it moves to the next. |
| **Cost-centered** | Tries the cheapest flavor per vCPU first, then the next cheapest. |

Up to **8 flavors** can be selected for scale-out.

![Flavor selection policy](images/l10-12-flavor-selection-policy.jpg)

**1.2 Create an AS Group**

| Setting | Value |
|---|---|
| Min. instances | **1** |
| Expected instances | **2** |
| Max. instances | **3** |
| Region – AZs | AP-Singapore – all AZs (default) |
| Name | `as-group-service01` |
| AS configuration | `as-config-service01` |
| VPC – Subnet | `vpc-as` – `subnet-test` |
| Load balancing | *Do not use* (added later in Part 3) |
| Multi-AZ policy | Balanced |
| Removal policy | Oldest instance |
| EIP – Data disk | Release – Delete |
| Health check | ECS – every 5 min – grace period 600 s |

![AS group](images/l10-13-as-group.jpg)

**Key choices inside the AS group**

| Choice | Option used | Other options |
|---|---|---|
| **Multi-AZ scaling** | **Balanced** – adds to the AZ with the fewest instances first | Sequenced – tries AZs in the order selected |
| **Instance removal** | **Oldest instance** | Newest instance, oldest/newest from the oldest AS configuration |
| **Health check** | **ECS** – replaces stopped or deleted instances | **ELB** – replaces instances a listener finds unhealthy (needs a load balancer) |

![AS group key choices](images/l10-14-as-group-key-choices.jpg)

### Part 2 — Scaling policies

There are **three types** of AS policy. Every policy = a **trigger** + an **action** (add, reduce, or set to a fixed number).

| Type | When it runs | Best for | Lab example |
|---|---|---|---|
| **Alarm-based** | Reacts to a metric alarm (e.g. CPU usage) | Demand that changes often and irregularly | Train ticket booking |
| **Scheduled** | Runs once at a specific time | Known one-off events | Course selection |
| **Periodic** | Repeats daily, weekly or monthly | Regular, predictable peaks | Nightly live-stream bandwidth |

![Three types of AS policy](images/l10-15-three-policy-types.jpg)

**2.1 Alarm-based policies** — *train ticket booking: add an ECS when CPU is high, remove it when CPU is low.*

| | Scale OUT | Scale IN |
|---|---|---|
| Policy | `as-policy-service01` | `as-policy-service02` |
| Trigger | Max CPU **> 80 %** | Max CPU **< 40 %** |
| Action | **Add 1** instance | **Reduce 1** instance |
| Alarm rule | `as-alarm-cpu01` | `as-alarm-cpu02` |

Both use: system monitoring · 5-min interval · 1 consecutive occurrence · **300 s cooldown**.

![Alarm-based policies](images/l10-16-alarm-policies.jpg)

**Stress test — trigger the alarms**

1. Check the **Monitoring** tab: CPU near 0 %, 1 instance.
2. ECS console → **Remote Login** to the group's ECS.
3. Create `cpu_usage.sh` (with `sudo vim`) containing the CPU-burning loop line twice.
4. Run `sh cpu_usage.sh` → wait about **5 min** → CPU > 80 % → **+1 instance**.
5. Stop the load (`pkill dd`) → wait about **8 min** → back to **1 instance**.

![Stress test](images/l10-17-stress-test.jpg)

**2.2 Scheduled policies** — *course selection is busy at the start of a semester and quiet afterwards.*

| Event | Policy | Action |
|---|---|---|
| Selection starts | `as-policy-service03` – scheduled for *now + 1 min* | **Add 1** instance |
| Selection ends | `as-policy-service04` – scheduled for a later time | **Reduce 1** instance |

> The trigger time must be **later than now**.

![Scheduled policies](images/l10-18-scheduled-policies.jpg)

**2.3 Periodic bandwidth scaling** — a live stream runs **20:00–22:00 daily** and needs 10 Mbit/s; the rest of the day 5 Mbit/s is enough.

| Policy | Rule |
|---|---|
| `as-policy-001` | Periodic – daily **20:00** → set to **10 Mbit/s** |
| `as-policy-002` | Periodic – daily **22:00** → set to **5 Mbit/s** |

Verify the result in **Policy Execution Logs**.

![Periodic bandwidth scaling](images/l10-19-periodic-bandwidth.jpg)

### Part 3 — Load balancer stability test

**3.1 Create the load balancer**

| Parameter | Value |
|---|---|
| Type – Billing | Dedicated – Pay-per-use |
| Region – AZ | AP-Singapore – AZ1 |
| IP as a backend | Enabled |
| Network | Public + private IPv4 |
| EIP | New – Dynamic BGP – 5 Mbit/s |
| Specification | Application – Small I |
| Name | `elb-as` |

Traffic flow: **Browser → `http://<ELB EIP>:89` → listener (HTTP, port 89) → backend server group (HTTP, weighted round robin) → backend ECSs on port 8089.**
Access logs go to an LTS group/stream.

![Create the load balancer](images/l10-20-create-elb.jpg)

**3.2–3.3 Attach the ELB and test**

- Attach to AS group: `as-group-service01` → Modify → Load balancing: **ELB**, load balancer `elb-as`, server group `server_group-as`, port **8089**, weight **1**.
- ⚠️ The **ELB and the AS group must be in the same VPC.**
- Test:
  1. Browse to the ELB EIP on port 89 — access logs show requests **spread across backend IPs**.
  2. **Simulate a failure:** remove a running server from the backend group.
  3. Browse again — logs show traffic going to the **remaining healthy server**.

![Attach the ELB and test](images/l10-21-attach-elb-test.jpg)

### ✅ What I practised

- Built an AS configuration and AS group, and scaled it by hand
- Scaled out and in automatically on CPU alarms
- Planned capacity with scheduled and periodic policies, including bandwidth
- Proved the service survives a server failure behind an ELB
- 🧹 **Cleanup:** delete the AS group, ELB, bandwidth policies and LTS log group to stop billing

![Summary](images/l10-22-summary.jpg)

---

# Lecture 11 — Virtual Private Cloud (VPC)

## 1. What is a VPC?

A **VPC** is **your own private, isolated network inside the cloud.**

1. **Isolated network** – logically separated, like an on-prem network.
2. **Your IP addressing** – you pick the private CIDR, e.g. `10.10.0.0/16`.
3. **Launch resources inside** – EC2, RDS and more run in its subnets.
4. **You control traffic** – subnets, routes, gateways, security groups and NACLs.
5. **Regional scope** – a VPC spans **all AZs in a region**; each **subnet sits in exactly one AZ**.

Hierarchy: **Account → Region → VPC → Resources**

![What is a VPC](images/l11-01-what-is-vpc.jpg)

### VPC sizing rules

- Up to **5 VPCs per region** (soft limit) and up to **5 CIDRs per VPC**.
- **Smallest** block: `/28` = **16 IP addresses**.
- **Largest** block: `/16` = **65,536 IP addresses**.
- Because a VPC is private, only **private IP ranges (RFC 1918)** are allowed:

| Range | CIDR |
|---|---|
| 10.0.0.0 – 10.255.255.255 | `10.0.0.0/8` |
| 172.16.0.0 – 172.31.255.255 | `172.16.0.0/12` *(default VPC is in this range)* |
| 192.168.0.0 – 192.168.255.255 | `192.168.0.0/16` |

- ⚠️ Your VPC CIDR **should not overlap** with your other networks (e.g. your corporate network).

![VPC overview](images/l11-02-vpc-overview-cidr.jpg)

## 2. Subnets, Route Tables & Internet Gateway

A VPC (`10.0.0.0/16`) is divided into **subnets**, and each subnet lives in **one Availability Zone**. In the diagram below, `us-east-1a`, `1b` and `1c` each hold subnets such as `10.0.1.0/24`, `10.0.4.0/24`, `10.0.7.0/24` …

![Subnets across AZs](images/l11-03-subnets-across-azs.jpg)

The **Internet Gateway (IGW)** is what connects the VPC to the internet, and the **router / route table** decides where traffic goes. Every VPC has an **implicit router** with a **local route** for the VPC CIDR so subnets can talk to each other.

![Internet gateway and subnets](images/l11-04-internet-gateway-subnets.jpg)

**Public vs Private subnet** (DemoVPC `10.0.0.0/16`):

| | Public subnet `10.0.1.0/24` | Private subnet `10.0.2.0/24` |
|---|---|---|
| Route to internet | Yes — `0.0.0.0/0 → IGW` | No |
| Typical resources | Web server, bastion host, NAT gateway | Databases, app servers |

![DemoVPC diagram](images/l11-05-demovpc-diagram.jpg)

![Public and private subnets across two AZs](images/l11-07-public-private-subnets.jpg)

## 3. Lab C — Create a Custom VPC

1. **Create VPC** – `DemoVPC`, `10.0.0.0/16`
2. **Subnets** – Public `10.0.1.0/24`, Private `10.0.2.0/24`
3. **Route tables** – Public RT, Private RT
4. **Public subnet** – enable *auto-assign public IPv4*
5. **Launch EC2** – in the public subnet, SG: SSH 22
6. **Test SSH** – ❌ **fails** (no internet path yet)
7. **Internet Gateway** – create it and attach it to DemoVPC
8. **Public RT** – add route `0.0.0.0/0 → IGW`
9. **SSH again** – ✅ **succeeds**

> 🔑 **Key idea:** A subnet is **public only when its route table sends `0.0.0.0/0` to an Internet Gateway.**

![Lab: create a custom VPC](images/l11-06-lab-custom-vpc.jpg)

## 4. NAT Gateway

Resources in a **private subnet** shouldn't be reachable from the internet — but they often still need to *go out* (for updates, patches, APIs). That's the job of a **NAT Gateway**.

- The NAT gateway sits in the **public subnet**.
- The private subnet's route table sends internet-bound traffic to the NAT gateway.
- The NAT gateway forwards it out through the **IGW**.
- Result: **outbound works, inbound from the internet is blocked.**

![IGW and NAT diagram](images/l11-08-igw-nat-diagram.jpg)

**Lab: NAT Gateway**

- Add an EC2 instance in the **private subnet**.
- Access the private EC2 using a **bastion host** (in the public subnet).
- Add a **NAT gateway** so the private EC2 can access the internet.

![Lab: NAT Gateway](images/l11-09-lab-nat-gateway.jpg)

## 5. Security Groups vs NACLs

Traffic is filtered at **two levels**:

![SG, NACL, IGW and VGW](images/l11-17-sg-nacl-igw-vgw.jpg)

| | Security Group | Network ACL (NACL) |
|---|---|---|
| Level | Instance (ENI) | **Subnet** |
| Rules | Allow rules | Allow **and Deny** rules, evaluated in numbered order |

**Default NACL**

- Accepts everything inbound/outbound for the subnets it's associated with.
- ⚠️ **Do NOT modify the default NACL — create custom NACLs instead.**
- Rule `100` allows all IPv4 traffic; the final `*` rule denies anything that doesn't match.

![Default NACL](images/l11-18-default-nacl.jpg)

## 6. VPC Peering

**VPC peering connects networks between two VPCs** using the cloud provider's private backbone — no internet needed.

- Works **intra-region and inter-region**
- Works **cross-account**
- **IP ranges cannot overlap**
- You must **update the route tables on both sides** (e.g. `10.2.0.0/16 → PCX-1` in VPC A and `10.1.0.0/16 → PCX-1` in VPC B)

![VPC peering route tables](images/l11-11-vpc-peering-route-tables.jpg)

**How peering works — 5 steps**

1. Create two VPCs
2. Send a peering request (from VPC A to VPC B)
3. Accept the request (VPC B accepts)
4. Update route tables in both VPCs
5. Configure security groups to allow the traffic

![How VPC peering works](images/l11-10-vpc-peering-how-it-works.jpg)

### Lab D — VPC Peering (target time: 30 min)

- **VPC A** (Singapore) – `10.10.0.0/16`, public subnet `10.10.0.0/24` — the **Requester VPC**
- **VPC B** (N. Virginia) – `10.20.0.0/16`, private subnet `10.20.0.0/24` — the **Accepter VPC**
- Also practised across **two accounts** (Account A ↔ Account B), with an Apache server in VPC B.

![Lab: VPC peering](images/l11-12-lab-vpc-peering.jpg)

![Lab: cross-account VPC peering](images/l11-13-lab-vpc-peering-cross-account.jpg)

**Testing:** from the internet, reach the public EC2 in VPC A, then **SSH or ping** the private IP of the EC2 in VPC B across the peering connection.

![Peering SSH/ping test](images/l11-14-peering-ssh-ping-test.jpg)

### ⚠️ VPC Peering limitations

- ❌ **No overlapping CIDRs**
- ❌ **Not transitive** — if A↔B and B↔C, then A and C still can't talk. Each pair that needs to communicate needs its **own** peering connection.
- Only **1** peering connection between any 2 VPCs
- Max **125** peering connections per VPC

![Peering limitations](images/l11-16-peering-limitations.jpg)

![Transitive peering not allowed](images/l11-15-transitive-peering-not-allowed.jpg)

### Shared-services pattern

A common design: put shared tools (e.g. logging, AD, monitoring) in a **Shared Services VPC** and peer each app VPC to it.

- App VPCs have **no peering with each other**.
- You **cannot** use the shared-services VPC as a **transit point** between app VPCs.
- Security groups and network ACLs **still apply**.

![Shared services VPC](images/l11-19-shared-services-vpc.jpg)

![Shared services with VPN to on-prem](images/l11-20-shared-services-vpn.jpg)

### The full-mesh problem

If every VPC must talk to every other VPC, the number of peering connections explodes:

$$\text{connections} = \frac{n(n-1)}{2}$$

| VPCs (n) | Peering connections |
|---|---|
| 3 | 3 |
| 5 | 10 |
| 6 | **15** |
| 10 | 45 |
| 20 | 190 |

![Full mesh VPC peering](images/l11-21-full-mesh.jpg)

![Mesh formula](images/l11-22-mesh-formula.jpg)

![Multiple VPCs without Transit Gateway](images/l11-23-without-transit-gateway.jpg)

## 7. Transit Gateway

**Transit Gateway** fixes the full-mesh mess. Instead of N×(N-1)/2 peerings, every network connects **once** to a central **hub**.

- Interconnects **thousands of VPCs and on-premises networks** (up to **5,000**).
- Acts as a **hub** for all traffic to flow through (hub-and-spoke).
- Supports **multicast** and **inter-Region peering**.
- Supported attachments:
  - One or more **VPCs**
  - **VPN**
  - **Direct Connect Gateway**
  - Peering connection with **another Transit Gateway**
  - **Connect** (SD-WAN / third-party network appliance)

![What is Transit Gateway](images/l11-24-what-is-transit-gateway.jpg)

![Transit Gateway as a hub](images/l11-25-transit-gateway-hub.jpg)

| | VPC Peering | Transit Gateway |
|---|---|---|
| Topology | Point-to-point | Hub-and-spoke |
| Transitive routing | ❌ No | ✅ Yes |
| Scales to many VPCs | Poorly (n(n-1)/2) | Very well (1 attachment per VPC) |
| On-prem (VPN / Direct Connect) | No | Yes |
| Best for | A few VPCs, simple links | Many VPCs + hybrid cloud |

A full picture of everything together — VPC, AZs, public/private subnets, NAT gateway, ALB, flow logs, peering, Client VPN, Direct Connect, VPN and a corporate data center:

![Full VPC architecture](images/l11-26-full-vpc-architecture.jpg)

## 8. Networking services — AWS vs Huawei Cloud vs Azure

| Concept | AWS | Huawei Cloud | Azure |
|---|---|---|---|
| Shared VPC (one network, many accounts) | VPC Sharing (RAM) | VPC Sharing (subnet sharing via RAM) | No direct equivalent; a VNet stays in one subscription. Use hub-and-spoke with VNet peering |
| Hub-and-spoke transit | **Transit Gateway** | **Enterprise Router** | VNet with VNet peering + gateway transit |
| Inter-region transit | VPC inter-region peering | Cloud Connect central network | Global VNet Peering / Virtual WAN hub-and-hub |
| Point-to-point network link | **VPC Peering** | **VPC Peering** | **VNet Peering** |
| Private service access | PrivateLink / VPC Endpoints | VPC Endpoint (VPCEP) | Private Link / Private Endpoint |
| Cross-account resource sharing | Resource Access Manager | Resource Access Manager | No RAM service; Azure RBAC across subscriptions |
| Multi-account governance | AWS Organizations | Huawei Organizations | Management Groups + subscriptions |

![AWS vs Huawei Cloud vs Azure networking](images/l11-27-aws-huawei-azure-comparison.jpg)

---

# 🎯 Key Takeaways

1. **Scale up = bigger instance; scale out = more instances.** Databases tend to scale up, web tiers scale out.
2. **Scalability ≠ Elasticity ≠ Agility.** Elasticity is the *automatic* part; agility is about *speed of provisioning*.
3. An **Auto Scaling Group** is defined by **min / desired / max** capacity, a **launch template**, and **scaling policies**.
4. Use the right policy: **alarm-based** for unpredictable load, **scheduled** for one-off events, **periodic** for regular peaks.
5. Put a **load balancer** in front of the ASG and use **ELB health checks** so failed instances are replaced and traffic avoids them.
6. A **VPC** is your private network; subnets live in **one AZ**, the VPC spans the **whole region**.
7. A subnet is **public only if its route table has `0.0.0.0/0 → Internet Gateway`.**
8. **NAT Gateway** = private resources can go *out* but nobody can come *in*.
9. **Security Groups** guard instances; **NACLs** guard subnets — and don't edit the default NACL.
10. **VPC peering is not transitive** and needs non-overlapping CIDRs; at scale use **Transit Gateway** (AWS) / **Enterprise Router** (Huawei).

---

# 📝 Quick Revision Cheat Sheet

| Topic | Remember |
|---|---|
| Vertical scaling | Bigger instance, hardware limit |
| Horizontal scaling | More instances, needs distributed design |
| ASG capacity | Min ≤ Desired ≤ Max |
| Launch Template | Replaces deprecated Launch Configuration |
| Huawei flavor policies | Sequenced vs Cost-centered (up to 8 flavors) |
| AS policy types | Alarm-based · Scheduled · Periodic |
| Cooldown (lab) | 300 s |
| VPC CIDR size | `/16` (65,536 IPs) down to `/28` (16 IPs) |
| Private ranges | `10/8` · `172.16/12` · `192.168/16` |
| Public subnet rule | `0.0.0.0/0 → IGW` |
| NAT Gateway location | Public subnet |
| Peering limits | 1 per VPC pair · 125 per VPC · no overlap · not transitive |
| Full-mesh formula | n(n−1)/2 |
| Transit Gateway | Hub-and-spoke, up to 5,000 VPCs |

---

## 📸 Source note

Notes are based on class lecture slides and lab walkthroughs (AWS Training material and the Huawei Cloud Auto Scaling lab) from the HCCDP Solution Architect course. Slide images are photographed from the lecture screen and are included for study and reference.

---

*Part of my cloud computing learning journey — follow along for the next weeks! ☁️*

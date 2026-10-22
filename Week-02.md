# Cloud Computing Course Journey: Week 2 Notes

> Lectures 3 and 4: Virtualization, Containers, and Cloud Deployment Models

---

## Lecture 3: Virtualization

### Eight Common Characteristics of Cloud Computing

| # | Characteristic | Meaning |
|---|----------------|---------|
| 1 | Massive Scale | Practically no limit on resources |
| 2 | Homogeneity | Same quality of service everywhere (what is available in Pakistan is available in the USA) |
| 3 | Virtualization | Dividing one physical machine into multiple logical machines |
| 4 | Resilient Computing | Services keep running despite failures |
| 5 | Low Cost Software | Pay-as-you-go, lower software costs |
| 6 | Advanced Security Technologies | Strong, modern security features |
| 7 | Geographical Distribution | Resources spread across regions worldwide |
| 8 | Service Orientation | Everything is offered as a service |

---

### Legacy Servers

A **legacy server** is an older server or system that is still in use because replacing or modernizing it is difficult, expensive, or risky.

**Disadvantages**
- Resource utilization was only about 4-5%
- Servers were underutilized
- Only one OS could be installed
- Only one application could run

**Solution: Virtualization**

---

### What is Virtualization?

Virtualization is the process of creating multiple virtual computers or servers from a single physical computer or server.

```
1 Physical Server  ->  Multiple Virtual Servers
```

**Analogy:** A house is the physical server. The 3 separate rooms inside it are the virtual servers.

---

### Hypervisor

A **hypervisor** is software that allows you to create and run multiple virtual machines (VMs) on one physical server.

- It provides each VM with the resources it needs (CPU, RAM, storage).
- Each VM *feels* like it has dedicated resources, but they are actually shared.

#### Type 1 Hypervisor (Bare Metal)
- No OS layer between the hardware and the hypervisor; it is installed directly on the hardware.
- A normal OS manages resources for applications (e.g. MS Word). Here, the hypervisor provides the resources.
- **Better performance** than Type 2 because there are fewer layers.
- **Examples:**
  - AWS uses **Citrix Xen**
  - Huawei now uses **KVM** (Linux), previously Citrix Xen

---

### Benefits / Essence of Virtualization

1. **Partitioning**
   - Each VM has its own OS.
   - Multiple VMs run at the same time on a single server.

2. **Isolation**
   - If one VM becomes faulty, it does not affect the others.
   - Management policies keep VMs isolated; VM A can only be accessed by user A.

3. **Encapsulation**
   - A VM is essentially a folder containing two things:
     1. Configuration file
     2. State file
   - This makes VMs easy to copy and move (e.g. with vMotion).

4. **Hardware Independence**
   - A VM can run on any server, with any hardware, without modification.

---

### VMware Features

**VMware** is a virtualization technology used to create and manage virtual machines.

#### High Availability (HA)
Ensures services remain available even when a server fails.
> Server A fails -> the VM automatically restarts on Server B.

#### Fault Tolerance (FT)
The VM keeps running continuously on a second server, so there is **zero downtime**.
> Server A faults -> Server B immediately continues the VM; the VM never stops.

| | High Availability | Fault Tolerance |
|--|------------------|-----------------|
| On failure | VM **restarts** on another server | VM **continues** on another server |
| Downtime | Brief | None |

#### vMotion
A VMware technology that moves a **running** VM from one physical server to another **without shutting it down**.

**Reasons to use it:**
- **Scheduled maintenance:** to upgrade the OS, migrate the VMs to another server first, then upgrade.

#### Distributed Resource Scheduling (DRS)
Balances workload automatically.
> CPU A is at 90% and CPU B is at 30% -> some VMs are automatically vMotioned to server B.

#### Auto-load Adjustment
- VMs, applications, and security settings are all transferred along.
- Shifting is both **time-based** and **load-based**.

---

### Disadvantages of VMs

| Problem | Explanation |
|---------|-------------|
| **Licensing** | If a server hosts 50 VMs, 50 OS licenses are required |
| **High storage** | Every VM carries its own full OS, which takes separate storage |
| **Startup delay** | The server must start first, then the VM (OS boot is time-consuming) |
| **Poor portability** | Hard to run the same VM on another environment |

**Solution: Containers**

---

### Containers

- All applications share **one OS**, so only **one license** is needed.
- No extra OS per app, so there is **no extra storage problem**.
- The OS is already running, so startup is fast.
- Each application and its dependencies and libraries are packaged together.
- **Lightweight:** a server that supports about 50 VMs can support up to about **1000 containers**.
- **Agile:** configuration can be done with a few clicks.
- Real-world use: many banking applications run on containers.

#### VM vs Container

| Feature | Virtual Machine | Container |
|---------|----------------|-----------|
| OS | One OS per VM | Shared OS |
| Licenses | Many | One |
| Storage | High | Low |
| Startup | Slow | Fast |
| Density per server | ~50 | ~1000 |
| Portability | Poor | Good |

---

## Lecture 4: Deployment Models for Cloud Computing

Each model is described by 4 aspects: resource ownership, who can access it, examples, and use cases.

### 1. Public Cloud

- **Resource ownership:** The cloud provider owns the resources.
- **Who can access:** Everyone, over the internet, after paying for the resources they use.
- **Examples:** AWS, GCP, Oracle, Huawei, Rackspace, Heroku, DigitalOcean, Alibaba
- **Advantages:** Excellent for optimization, migration strategies, and **startups**.

### 2. Private Cloud

**VPC (Virtual Private Cloud):** a private section of a cloud provider's network that you control. It is *virtual* because it is software-defined, and *private* because only you can use that isolated space. Compute, storage, and databases are organized inside the VPC.

- **Resource ownership:** Always **built**, not purchased.
- **Who can access:** Only authorized users of the organization.
- **Examples:** VMware, telecom companies (Jazz, Ufone, Zong)

**OpenStack**
- An open-source cloud platform, open to everyone.
- Huawei and AWS use OpenStack-style protocols.
- Used to build an **orchestration layer**.

**Private cloud product examples (as noted in lecture):**
- VeloCloud: VMware products/components
- D-Cloud: Cisco products
- Cisco U

### 3. Hybrid Cloud

- **Resource ownership:** A combination of public and private cloud.
- **Who can access:** Depends on the public and private parts.
- **Example:** Private infrastructure + AWS/Azure, etc.
- **Infrastructure:** Mixed cloud infrastructure.
- **Use:** Public sector.

| Model | Ownership | Access | Examples |
|-------|-----------|--------|----------|
| Public | Provider | Anyone with payment | AWS, GCP, Alibaba |
| Private | Built by the organization | Authorized users only | VMware, Jazz, Zong |
| Hybrid | Both | Depends on the part | Private + AWS/Azure |

---

### AWS Global Infrastructure

- The **first AWS region** was **US East (N. Virginia)**.
- Each **region** has **3 to 6 Availability Zones (AZs)**.
- Each availability zone can consist of **1, 2, or 3 data centers** in close proximity.

### Side Note: Huawei Kunpeng

- **Kunpeng** is Huawei's processor.
- US restrictions stopped Intel from selling to China/Huawei, which pushed Huawei toward its own processors.

---

## Quick Revision Summary

- **Virtualization** fixes the problem of underutilized single-purpose legacy servers.
- A **hypervisor** (Type 1 runs directly on hardware) creates and manages VMs.
- Core benefits: **partitioning, isolation, encapsulation, hardware independence**.
- VMware features: **HA, FT, vMotion, DRS**.
- VM drawbacks (licensing, storage, startup, portability) are solved by **containers**.
- Cloud deployment models: **Public, Private, Hybrid**.
- AWS structure: **Region -> Availability Zones -> Data Centers**.

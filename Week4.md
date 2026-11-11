# Week 4: EC2 User Data & Load Balancing

> Part of my cloud learning journey. These are my personal study notes, covering concepts only.

---

## 📚 Topics Covered

1. Launching Amazon EC2 Instances with User Data
2. Load Balancing (Lecture 9)

---

# Part 1: Launching Amazon EC2 Instances with User Data

## What is EC2 User Data?

**User Data** is a **bootstrap script** that runs automatically the **first time** an EC2 instance boots, so the instance configures itself with **no manual setup**.

## 🏠 Layman Example

Imagine you buy a new house. When you get the keys, it is completely empty. You would have to manually:

1. Turn on electricity
2. Install lights
3. Set up Wi-Fi
4. Install furniture
5. Put everything in its place

Instead, you give a worker a list of instructions **before** you move in:

> "When you open the house for the first time, turn on the electricity, install the lights, set up Wi-Fi, and arrange the furniture."

The worker follows those instructions automatically.

### Now map it to EC2

| House analogy | EC2 equivalent |
|---|---|
| New house | EC2 instance |
| Instruction list | User Data script |
| Worker following the list | AWS running the script at first boot |

For example, you could tell the instance: *"Install Apache web server and start it."*

Instead of connecting to the instance and typing commands manually:

```bash
sudo yum install httpd
sudo systemctl start httpd
```

you put those commands in User Data when launching the instance.

### Flow to remember

```
Launch EC2 → Instance starts → User Data runs → Apache installed → Apache starts
```

---

## Common Uses

- Install OS updates and software
- Download files or configs from the internet
- Create users and set environment variables
- Run configuration tools (e.g., Chef, Ansible)

---

## Key Facts

| Fact | Detail |
|---|---|
| **Runs once** | Only at first boot, **not** on every restart |
| **Runs as root** | No `sudo` needed inside the script |
| **Max size** | **16 KB** per script |

---

## Anatomy of a User Data Script

```bash
#!/bin/bash
# ...commands go here...
```

- `#!` → called the **shebang**
- `/bin/bash` → tells the system to use **Bash** to run the script
- Everything after it → the commands Bash should execute

---

## 📖 Quick Glossary

| Term | Meaning |
|---|---|
| **Firewall** | Software that controls allowed/blocked network traffic |
| **Apache** | A web server (it is a *service*) |
| **MySQL** | A database |
| **Docker** | Runs applications in containers |
| **Git repo** | A place where project code is stored |
| **Clone** | Copy project code |
| **Environment variable** | An application setting |
| **Hostname** | The server's name |
| **Service** | A background program |
| **Cron job** | A scheduled automatic task |
| **Auto Scaling** | Automatically adds/removes EC2 servers |
| **Cloud-init** | Automatically configures a new server at boot |
| **Root** | Highest-level Linux administrator |
| **SSH** | Remotely connects to a server |
| **Provisioning** | Preparing a server for use |

---

## 💡 Key Takeaway

User Data turns a blank server into a ready-to-use server automatically. This is the foundation of **automation** and **repeatable infrastructure** in the cloud.

---

# Part 2: Load Balancing (Lecture 9)

**Guiding question:** How do we add **high availability** to our Amazon EC2 workloads and distribute traffic across multiple targets?

## Definition

**Load balancing** means distributing incoming work/traffic among multiple servers so that **no single server gets overloaded**.

## Load-Balancing Algorithms

### 1. Round Robin

Requests are distributed one by one, in order:

```
Server 1 → Server 2 → Server 3 → Server 1 → Server 2 → Server 3 ...
```

⚠️ It does **not** consider which server is more powerful or busy.

---

### 2. Weighted Round Robin

Each server is given a **weight** based mainly on its **capacity/performance**. The stronger server receives more requests.

| Server | Capacity | Weight |
|---|---|---|
| Server 1 | Powerful | 3 |
| Server 2 | Medium | 2 |
| Server 3 | Less powerful | 1 |

Weight can be based on:

- CPU capacity
- RAM
- Number of resources
- Overall server performance
- How much traffic the server can comfortably handle

> **Important:** Weight is about **server capacity**, not security.

---

### 3. Least Connections

The load balancer checks how many **active connections** each server currently has and sends the next request to the least busy one.

| Server | Active Connections |
|---|---|
| Server 1 | 20 |
| Server 2 | **8** ← next request goes here |
| Server 3 | 15 |

---

## DNS Name of a Load Balancer

In a cloud load-balancing setup, you are usually given a **DNS name** for the load balancer.

Instead of users accessing a single server IP directly, they access something like:

```
my-load-balancer.example.com
```

The DNS name points users to the load balancer, and the load balancer distributes their requests among the backend servers.

## Easy Flow to Remember

```
User → DNS Name → Load Balancer → Algorithm → Server
```

## Quick Summary

| Algorithm | How it works |
|---|---|
| **Round Robin** | 1, 2, 3, 1, 2, 3 (takes turns) |
| **Weighted Round Robin** | Stronger server gets more traffic |
| **Least Connections** | Server with fewer active connections gets the request |

---

## ✅ Week 4 Summary

- **User Data** automates server setup at first boot, so no manual configuration is needed.
- **Load balancing** spreads traffic across servers to improve **availability** and **performance**.
- Together, they form the building blocks of scalable, self-configuring cloud architectures.

---

*📝 Personal learning notes, shared as part of my cloud learning journey.*

# EC2 Purchasing Options — What Problem They Solve

---

## On-Demand
> *"I have no idea what's going to happen"*

**When to use**
- Unpredictable traffic or sudden spikes
- Short-term testing, demos, dev environments
- First time running an app (before committing to anything)

**Exam scenarios**
- Startup that just launched and doesn't know its load profile
- QA environment that spins up for a few hours

**Don't use when** you have predictable workloads — you're paying the highest rate for no reason.

Pros: Maximum flexibility  
Cons: Most expensive per hour

---

## Reserved Instances (RI)
> *"I know exactly what I'll use for 1 or 3 years"*

**When to use**
- Production databases (MySQL, PostgreSQL, Oracle)
- Always-on application servers
- Any stable, predictable workload

**Key variants (the exam distinguishes these)**

| Type | Flexibility | Discount |
|---|---|---|
| Standard RI | Can only change AZ or size (same family) | Higher (~72%) |
| Convertible RI | Can change family, OS, tenancy | Lower (~54%) |

**Exam scenarios**
- RDS instance running 24/7 → Standard RI
- Workload that might migrate from `m5` to `c5` in the future → Convertible RI

**Exam tip:** Standard RIs can be sold on the *Reserved Instance Marketplace* if you no longer need them.

Pros: Big discount with commitment  
Cons: Low flexibility; you're billed even if you don't use the instance

---

## Savings Plans
> *"I know how much compute I'll consume, but not exactly how"*

**When to use**
- You use Lambda + EC2 + Fargate in combination
- You change instance types frequently
- You want a discount without locking into a specific instance

**Key variants**

| Type | Scope | Covers |
|---|---|---|
| Compute SP | Any region, family, OS | EC2, Lambda, Fargate |
| EC2 Instance SP | Fixed family + region | EC2 only, higher discount |

**Exam scenarios**
- Company with microservices on Lambda and containers on ECS Fargate → Compute Savings Plan
- Only ever running `m5` in `us-east-1` → EC2 Instance SP (cheaper)

Pros: Discount + configuration flexibility  
Cons: Committed to a $/hour spend for 1 or 3 years

---

## Spot Instances
> *"I want the cheapest compute and can tolerate interruptions"*

**When to use**
- Batch jobs, image/video processing
- Machine learning training
- Big data (EMR), distributed analytics
- Workloads you can pause and resume

**When NOT to use (critical for the exam)**
- Databases
- Applications that can't be interrupted (production e-commerce)
- Anything stateful that you can't checkpoint

**Key mechanics**
- AWS gives you a **2-minute warning** before terminating your instance
- If the spot price exceeds your bid → instance is terminated
- **Spot Blocks** (fixed duration 1–6h): the exam mentions them but they are being discontinued

**Exam scenarios**
- Rendering movie frames → Spot (if a node fails, just re-render)
- Training an ML model overnight → Spot + checkpoint state to S3

Pros: Up to 90% cheaper than On-Demand  
Cons: Can be terminated at any time

---

## Dedicated Hosts
> *"I have licenses tied to physical hardware"*

**When to use**
- Software with **per-socket** or **per-core** licenses (Oracle DB, Windows Server, SQL Server)
- Compliance requiring knowledge of the exact physical server
- Regulations that prohibit hardware-level multi-tenancy

**Key difference vs Dedicated Instances**
- Dedicated Host: **you control VM placement** on the physical server. You can see the number of sockets/cores.
- Dedicated Instance: AWS manages the hardware; it only guarantees no sharing with other accounts.

**Exam scenarios**
- Bank with audits requiring a documented dedicated server → Dedicated Host
- Existing Oracle DB with a per-socket license → Dedicated Host (BYOL)

Pros: Full physical server control, BYOL support  
Cons: Most expensive; you reserve the entire host even if not fully utilized

---

## Dedicated Instances
> *"I need hardware isolation, but don't care about the exact server"*

**When to use**
- Internal security policies that prohibit shared hardware
- Moderate compliance requirements (not at the Dedicated Host level)

**What it guarantees:** your instance does not share hardware with instances from **other AWS accounts**.  
**What it does NOT guarantee:** you may still share the same host with other instances from **your own account**.

**Exam scenarios**
- Company with a no-shared-hardware-with-third-parties policy but no per-socket licenses → Dedicated Instance

Pros: Hardware isolation without managing the host  
Cons: More expensive than On-Demand; less control than Dedicated Host

---

## Capacity Reservations
> *"I MUST have capacity available — cost is secondary"*

**When to use**
- Disaster recovery: you must be able to launch instances in a specific AZ under any circumstance
- Critical time-bound events (product launches, Black Friday)
- Regulated workloads requiring guaranteed availability

**Details the exam exploits**
- No discount on their own — **combine with RI or Savings Plans** to get a discount
- You're billed even if you never launch an instance
- Scoped **per AZ**, not per region — you must specify the AZ

**Exam scenarios**
- Need to guarantee you can launch 50 `c5.4xlarge` instances in `us-east-1a` during an event → Capacity Reservation + RI for the discount

Pros: Guaranteed capacity in a specific AZ  
Cons: No built-in discount; you pay regardless of whether you use the capacity

---

## Quick Decision Table

| Situation | Option |
|---|---|
| No idea how much I'll use | On-Demand |
| Stable workload, same instance for years | Standard RI |
| Stable workload, might change instance family | Convertible RI or Savings Plan |
| Mix of Lambda + EC2 + Fargate | Compute Savings Plan |
| Fault-tolerant batch jobs | Spot |
| Oracle/Windows per-socket licenses | Dedicated Host |
| Hardware isolation without BYOL | Dedicated Instance |
| Capacity guarantee for DR or events | Capacity Reservation |
| Want discount AND capacity guarantee | Capacity Reservation + RI |

---

# EC2 Spot — Requests, Fleets & Blocks

> Spot gives you up to 90% savings in exchange for one thing: AWS can take the instance back.
> Understanding *how* that works — and how to defend against it — is what this section is about.

---

## Spot Instance Requests
> *"I want cheap compute and I'll set my price ceiling"*

You bid a **max price**. The instance runs as long as `spot_price < max_price`.
Spot price fluctuates based on supply and demand per AZ and instance type.

### Interruption behavior
When `spot_price > max_price`, AWS can:
- **Stop** the instance (if EBS-backed — state is preserved)
- **Terminate** the instance (data on instance store is lost)
- **Hibernate** the instance (RAM saved to EBS)

You always get a **2-minute warning** via the instance metadata endpoint:
```
http://169.254.169.254/latest/meta-data/spot/interruption-notice
```

### Request types (the exam distinguishes these)

| Type | Behavior |
|---|---|
| One-time | Request fulfilled once; if interrupted, it's gone |
| Persistent | AWS re-submits the request after interruption until valid or expired |

**Exam trap:** to fully cancel a persistent Spot request, you must **cancel the request first**, then terminate the instance. If you only terminate, AWS re-launches it.

### Use cases
- Batch jobs, ETL pipelines
- Image/video processing
- ML training with checkpointing to S3
- Any stateless, fault-tolerant workload

### Not suitable for
- Databases
- Critical or stateful systems
- Anything that can't handle abrupt termination

---

## Spot Fleet
> *"I want Spot at scale — across multiple instance types and AZs"*

A Spot Fleet is a **collection of Spot Instances (+ optional On-Demand)** that together meet a target capacity.

Instead of betting on one instance type, you define **multiple pools** and let AWS fulfill the capacity from whichever combination works best.

### What you define
- Target capacity (units: instances, vCPUs, or memory)
- One or more **launch templates** with different instance types, AZs, and OS configs
- Optional On-Demand base capacity (for the critical portion of your workload)
- Max total cost

AWS stops launching instances when target capacity or max cost is reached.

### Allocation strategies

| Strategy | How it works | Best for |
|---|---|---|
| `lowestPrice` | Picks the cheapest pool | Cost-first, short jobs |
| `diversified` | Spreads across all pools | Long-running, high availability |
| `capacityOptimized` | Picks pools with most available capacity | Reducing interruptions |
| `priceCapacityOptimized` ✅ | Balances lowest price + available capacity | **Recommended default** |

**Exam tip:** `priceCapacityOptimized` is the answer when the question asks for *both* cost savings *and* availability. `lowestPrice` is cheaper but gets interrupted more. `diversified` is the answer when the question emphasizes availability over cost.

### Use cases
- Large-scale batch processing (hundreds of instances)
- Big data clusters (EMR)
- CI/CD farms that need to scale quickly
- ML training distributed across many nodes

---

## Spot Blocks (Legacy ⚠️)
> *"I need Spot capacity for a fixed window without interruption"*

Spot Blocks let you reserve a Spot Instance for a **defined duration of 1 to 6 hours** with no interruption during that window — in most cases.

AWS has deprecated this feature for new customers. It still appears on the exam as a concept.

### How it differs from a regular Spot request

| | Spot Request | Spot Block |
|---|---|---|
| Duration | Indefinite (until interrupted) | Fixed: 1–6 hours |
| Interruption risk | Yes, anytime | Rare, only in extreme capacity constraints |
| Price | Fluctuates | Fixed for the duration |
| Status | Active | Being phased out |

### Historical use cases
- Short batch jobs with a hard deadline
- Time-bound processing that couldn't afford mid-job interruption
- Jobs too short for Reserved Instances but too critical for regular Spot

**Exam approach:** if a question mentions Spot Blocks, it's testing whether you know they existed and what they solved — not whether you'd architect with them today. Prefer Spot Fleet with `capacityOptimized` for modern fault-tolerant designs.

---

# 🧠 AWS Placement Groups – Simple Explanation

## What are Placement Groups for?
They let you **control how AWS physically places your EC2 instances** on hardware.

👉 Basically: you decide if instances should be **close together or far apart**

---

## 🔹 Types

### 1. Cluster (close together = fast)
- Instances are **very close together**
- 🔥 Low latency, high throughput  
- Use case: HPC, intensive workloads

### Real use cases:
- HPC (High Performance Computing)
- Financial trading systems
- Distributed computing jobs
- In-memory workloads

---

### 2. Spread
- Instances are **placed on different hardware**
- 🛡️ High fault tolerance  
- Use case: critical applications  

### Real use cases:
- Critical applications
- Small number of important instances
- Systems where failure is not acceptable

---

### 3. Partition
- Instances are grouped into **isolated partitions**
- ⚖️ Balance between performance and resilience  
- Use case: big data (Hadoop, Kafka)

### Real use cases:
- Big Data (Hadoop, Spark)
- Kafka clusters
- Cassandra / NoSQL clusters

---

## 🧠 Easy way to remember

- **Cluster** → close = fast  
- **Spread** → separate = safe  
- **Partition** → grouped = balanced  

---

## ⚡ TL;DR
Placement Groups = control whether your instances are:
- close (performance)
- separated (resilience)
- grouped (balance)

# 🧠 Elastic Network Interface (ENI) – Simple Explanation

## What is an ENI?
> A **virtual network card** in AWS.

👉 It connects your EC2 instance to the network (VPC).

---

## 🔹 What an ENI includes
- Private IP  
- (Optional) Public IP  
- Security Groups  
- MAC address  
- Subnet  

👉 Basically: all networking configuration

---

## 🔹 Easy way to think about it

- EC2 = the computer  
- ENI = its **network card (NIC)**  

---

## 🔹 What is it used for?

### ✅ Use cases
- Connect instances to a VPC  
- Move network configuration between instances  
- Attach multiple network interfaces to one EC2

# EC2 Volumes

# AWS EC2 Storage Overview

## 1. EBS (Elastic Block Store)
**Definition:** Network-attached persistent storage for EC2 instances.

**Key Features:**
- Persistent even after instance termination (unless root volume with `Delete on Termination`).  
- Can only be attached to **one instance at a time** (except io1/io2 Multi-Attach).  
- Bound to an **Availability Zone (AZ)**; moving requires a **snapshot**.  
- Provisioned capacity: Size (GB) and IOPS; billed based on provisioned capacity.  
- Snapshots can be copied across AZs or regions.  
- Volume types:  
  - **gp2 / gp3 (SSD):** General purpose, balanced price-performance.  
  - **io1 / io2 / io2 Block Express:** High-performance SSD, low latency, PIOPS.  
  - **st1 (HDD):** Throughput-optimized, low cost.  
  - **sc1 (HDD):** Cold storage, infrequently accessed, cheapest.  
- **Multi-Attach:** io1/io2 volumes can attach to **up to 16 instances** in the same AZ (requires cluster-aware FS).  
- **Encryption:** At rest, in transit, and snapshots encrypted via KMS (AES-256), transparent to user.  

**Use Cases:**
- Root or boot volumes for EC2 instances.  
- Databases requiring consistent IOPS (gp3/io2).  
- Persistent storage for logs or critical data with snapshot backups.  
- Multi-Attach for clustered Linux applications (e.g., Teradata).  

---

## 2. EC2 Instance Store
**Definition:** Local physical storage on the EC2 host.

**Key Features:**
- Very high IOPS and low latency.  
- **Ephemeral:** Lost if the instance stops or the host fails.  
- Best for **temporary data**: buffers, caches, scratch data.  

**Use Cases:**
- High-performance cache.  
- Temporary or intermediate storage for data processing.  
- Scratch space for streaming or big data workloads.  

---

## 3. AMI (Amazon Machine Image)
**Definition:** Preconfigured EC2 image with OS, software, and settings.

**Types:** Public, private, or AWS Marketplace AMIs.  

**Process:** Stop instance → create AMI → includes EBS snapshots → launch new instances from AMI.  

**Use Cases:**
- Rapid deployment of identical EC2 instances.  
- Backup of instance configuration.  
- Reproducible dev/test environments.  

---

## 4. EBS Snapshots
**Definition:** Point-in-time backup of EBS volumes.

**Features:**
- Can copy across AZs or regions.  
- **Snapshot Archive:** 75% cheaper, restore takes 24–72 hours.  
- **Recycle Bin:** Recover deleted snapshots, configurable retention.  
- **Fast Snapshot Restore (FSR):** Initialize snapshot fully to remove first-use latency (extra cost).  

**Use Cases:**
- Volume backups.  
- Migration between AZs or regions.  
- Fast recovery in case of volume failure.  

---

## 5. EFS (Elastic File System)
**Definition:** Network file system, multi-AZ, POSIX compliant, Linux only.

**Key Features:**
- Scales automatically; pay-per-use.  
- Protocol: NFSv4.1, concurrent access for thousands of clients.  
- **Performance Modes:**  
  - **General Purpose:** Low-latency, web servers, CMS.  
  - **Max I/O:** High throughput, parallel workloads.  
- **Throughput Modes:**  
  - Bursting (size-based)  
  - Provisioned (fixed throughput)  
- **Storage Classes:** Standard, Infrequent Access (EFS-IA), Archive.  

**Use Cases:**
- Shared content across multiple EC2 instances (e.g., WordPress).  
- Highly concurrent and scalable workloads.  
- POSIX-compliant applications needing multi-AZ access.  

---

## 6. Quick Comparison

| Feature                  | EBS                      | Instance Store         | EFS                          |
|--------------------------|-------------------------|----------------------|-----------------------------|
| Persistence               | Yes                     | No                   | Yes                          |
| Multi-AZ                  | No (only snapshot copy) | No                   | Yes                          |
| Multi-instance Mount      | No (except io2 Multi-Attach) | No               | Yes                          |
| Performance               | Medium-High             | Very High            | Variable (GP vs Max I/O)    |
| Typical Use Case          | Boot volumes, DB, apps  | Cache/temp           | Shared files, multi-EC2     |

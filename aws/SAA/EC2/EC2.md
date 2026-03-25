# 🧠 EC2 Purchasing Options = Problems They Solve

## 🟢 On-Demand  
> ❓ *“I don’t know what will happen”*

- Unpredictable traffic  
- Short-term usage  
- Need instant start/stop  

✅ **Solution:** maximum flexibility  
❌ **Tradeoff:** expensive  

---

## 🔵 Reserved Instances (RI)  
> 📊 *“I know exactly what I’ll use”*

- Steady workload (e.g. database)  
- Long-running systems  
- Predictable capacity  

✅ **Solution:** big discount for committing  
❌ **Tradeoff:** low flexibility  

---

## 🟣 Savings Plans  
> 🔄 *“I know I’ll use compute, but not exactly how”*

- Predictable usage but changing configs  
- Might switch instance types  

✅ **Solution:** discount + flexibility  
❌ **Tradeoff:** time commitment  

---

## 🟡 Spot Instances  
> 💸 *“I want the cheapest compute possible”*

- Can tolerate interruptions  
- Distributed / fault-tolerant systems  

✅ **Solution:** massive cost savings  
❌ **Tradeoff:** can be terminated anytime  

---

## 🔴 Dedicated Hosts  
> ⚖️ *“I have licensing or compliance constraints”*

- BYOL (per-core, per-socket licenses)  
- Regulatory requirements  

✅ **Solution:** full physical server control  
❌ **Tradeoff:** very expensive  

---

## 🟠 Dedicated Instances  
> 🔐 *“I need isolation, but not full hardware control”*

- No shared hardware with other customers  

✅ **Solution:** tenant isolation  
❌ **Tradeoff:** less control than hosts  

---

## 🟤 Capacity Reservations  
> 🚨 *“I MUST have capacity available”*

- Critical workloads  
- Need guaranteed capacity in AZ  

✅ **Solution:** capacity guarantee  
❌ **Tradeoff:** no discount, pay even if unused  

---

# ⚡ Ultra-Compact Mental Model

- Don’t know usage → **On-Demand**  
- Know usage → **Reserved / Savings Plans**  
- Want cheapest → **Spot**  
- Need compliance → **Dedicated Hosts**  
- Need isolation → **Dedicated Instances**  
- Need guaranteed capacity → **Capacity Reservations**  

---

# 🧪 Quick Decision Framework

Ask yourself:

1. Is usage predictable?  
2. Can the workload be interrupted?  
3. Do I need guaranteed capacity or compliance?

## 🔹 Spot Instance Requests
- Up to **90% cheaper** than On-Demand  
- You define a **max price**  
- The instance runs while: **spot_price < max_price**
  - Price **fluctuates dynamically** (supply/demand)

### ⛔ Interruptions
- If: **spot_price > max_price**
→ AWS may:
- stop
- terminate

- You get a **2-minute warning**

### 🧠 Typical Use Cases
- Batch jobs  
- Data processing  
- Fault-tolerant workloads  

### ❌ Not suitable for
- Databases  
- Critical systems

## 🔹 Spot Fleet
> A set of Spot Instances (+ optional On-Demand Instances)

- Define a **target capacity**
- AWS fulfills it using multiple pools

### 🔧 You can define:
- Instance types (e.g., `m5.large`)
- Availability Zones
- OS
- Multiple pools

### 🧠 Behavior
- AWS selects instances based on:
- price
- available capacity

- Stops launching when:
- target capacity is reached
- max cost is reached

---

## ⚙️ Allocation Strategies

- **lowestPrice**  
→ cheapest  
→ less stable  

- **diversified**  
→ spread across pools  
→ higher availability  

- **capacityOptimized**  
→ prioritizes pools with capacity  

- **priceCapacityOptimized (RECOMMENDED)**  
→ balances price + capacity  

---

## 🔹 Spot Block (Legacy ⚠️)
> Not commonly used anymore

- “Reserve” Spot for:
- 1 to 6 hours  
- **No interruptions** during that window (in most cases)
- May still be reclaimed in rare situations

### 🧠 Typical Use Cases (historical)
- Short batch jobs  
- Time-bound processing workloads  

---

## 🧩 What is each concept?

| Concept              | Description |
|----------------------|------------|
| Spot Instance        | Discounted instance type |
| Spot Request         | Individual Spot request |
| Spot Fleet           | Managed group of Spot instances |
| Spot Block           | Legacy fixed-duration Spot |

---

## ⚡ TL;DR
- `run-instances` → On-Demand  
- Spot → cheaper but interruptible  
- Spot Fleet → automated orchestration  
- Spot Block → legacy, mostly ignore   
  
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

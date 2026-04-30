---
module: 02
title: The Nutanix Stack (Node, CVM, Cluster)
estimated_reading_time: 32 min
prerequisites:
  - Module 01 (HCI Foundations)
  - Familiarity with ESXi host architecture
  - Understanding of VMs and virtual hardware
  - Working CE lab (from Module 01)
key_terms:
  - Node
  - Block
  - Cluster
  - CVM (Controller VM)
  - DSF (Distributed Storage Fabric)
  - Data locality
  - Stargate, Cassandra, Curator, Zeus, Pithos, Acropolis
  - Foundation
  - LCM (Life Cycle Manager)
  - NCC (Nutanix Cluster Check)
  - Cluster Virtual IP (VIP)
diagrams:
  - nutanix-node-anatomy
  - nutanix-cluster-topology
  - cvm-services-architecture
cert_coverage:
  NCA: ~22%
  NCP-MCI: ~18%
  NCM-MCI: ~10%
sa_toolkit:
  related_objections: [obj-002, obj-003, obj-008, obj-018]
  related_discovery: [q-arch-04, q-cap-01, q-cap-02]
---

# Module 2: The Nutanix Stack (Node, CVM, Cluster)

> **Cert coverage:** NCA (~22%) · NCP-MCI (~18%) · NCM-MCI (~10%)
> **SA toolkit:** Objections #2, #3, #8, #18 · Discovery Q-ARCH-04, Q-CAP-01, Q-CAP-02

---

## The Promise

By the end of this module you will:

1. **Draw a Nutanix deployment on a whiteboard, top to bottom, in 90 seconds.** No prompting. The kind of drawing that wins customer credibility in the first 10 minutes of a meeting.
2. **Explain the Controller VM (CVM): what it is, why it's there, and what it costs.** Use language a senior VMware admin will believe. The CVM is the single most important architectural decision in the entire product. It is also the most common technical objection you will hear. You need this cold.
3. **Pass roughly 22% of NCA and 18% of NCP-MCI.** The architectural fundamentals (node, cluster, CVM, data locality, Foundation, LCM, NCC) show up across nearly every exam domain. Master this module and the rest of cert prep gets dramatically easier.
4. **Handle the four most common technical objections** about Nutanix architecture: the CVM tax, the vSAN comparison, "what happens when the CVM fails," and "why can't I just put it on my existing servers." Each has a clean, honest, customer-credible answer. By the end of this module you will have all four.

This is the bone structure of the platform. Everything in Modules 3 through 10 hangs off it. If Module 1 was the category, this is the product.

---

## Foundation: What You Already Know

Picture an ESXi host. It is a physical server: a motherboard, two sockets, RAM, NICs, a couple of M.2 boot devices, and a place where it gets its storage from (an HBA pointing at a SAN, or in some cases, local disks for vSAN).

ESXi is the hypervisor. It runs on the metal. It schedules CPU, allocates memory, presents virtual hardware to VMs, and talks to whatever storage it has been given.

vCenter is somewhere else. It is software (a vCenter Server Appliance, or VCSA) that runs as a VM and aggregates many ESXi hosts into a cluster, exposing one management plane.

Hold that picture. We are about to change three things:

1. The hypervisor might not be ESXi. It could be AHV, Nutanix's own KVM-based hypervisor (Module 3 goes deep on that).
2. There is a *second* VM running on every host whose job is to own storage. This is the **CVM**.
3. The "shared storage" the hypervisor sees is not a SAN. It is the local disks of every node in the cluster, presented as one logical pool by the CVMs cooperating with each other.

That is Nutanix architecture in three sentences. The rest of this module unpacks it, builds the fence around it, and arms you to defend it.

> [!FROM-THE-SA-CHAIR]
> The first time you explain Nutanix to a VMware admin in a customer meeting, lead with this exact framing. *"Three things change. You keep your hypervisor concept, but it might be a different hypervisor. Every node now runs an extra VM that owns storage. And your shared storage isn't a SAN anymore; it's the local disks of every node, pooled by software."* Three sentences. Watch their eyes. The third sentence is where the architecture clicks for them. That moment is when the meeting moves from skepticism to engagement.

---

## Core Content

### The Node

A Nutanix node is a physical x86 server. That is it. It has CPUs, RAM, NICs, and disks. The disks are local: NVMe and SSD typically, sometimes with an HDD tier on older hardware (rare in 2026). The NICs are 10/25/100 GbE.

If you took the lid off a Nutanix node and compared it to an ESXi host, you would not see anything visually different. Same chassis, same components, same form factor. Nutanix sells appliances (the NX line, NX-3000, NX-8000, NX-9000 series), but the platform also runs on Dell (XC), HPE (DX), Cisco (UCS), Lenovo (HX), Supermicro, and others. Many customers run Nutanix on hardware they were already buying for VMware.

What makes the node a *Nutanix* node is the software stack that gets installed on it.

> [!ON-THE-EXAM] **NCA · NCP-MCI**
> Both exams test that you understand Nutanix runs on multiple OEM hardware platforms. Trap question pattern: "Nutanix software requires Nutanix-branded hardware." False. The answer is the platform is hardware-agnostic across qualified OEM SKUs (NX, XC, HX, DX, UCS C-Series, etc.). Memorize at least three OEM partners. The NX line is Nutanix's own appliance, but it is not a requirement.

### What's Running on a Single Node

On a single Nutanix node, three things run:

1. **A hypervisor.** This is AHV (Nutanix's own, KVM-based) or ESXi or Hyper-V. The hypervisor sits directly on the metal.
2. **The Controller VM (CVM).** This is a privileged virtual machine that runs on top of the hypervisor on every node, no exceptions. The CVM owns storage. This is the architectural lever.
3. **Your user VMs.** Whatever workloads you actually care about. They run on the hypervisor alongside the CVM.

That third item should provoke a question: *if the CVM and my user VMs are both running on the same hypervisor, doesn't the CVM steal my CPU and RAM?*

Yes. Nutanix reserves cores and RAM for the CVM. On a typical 2026 node this is 8 to 12 vCPUs and 32 to 48 GB of RAM, scaling with feature set (more is required when you enable deduplication, capacity tiers, or NCM Intelligent Operations on-cluster). This is one of the genuine costs of the architecture and you should know about it before you read about it on Reddit. We will spend an entire section on this honestly. It deserves it.

---

### Diagram: Anatomy of a Nutanix Node

**id:** `nutanix-node-anatomy`
**type:** layered
**caption:** What lives on a single Nutanix node, from physical hardware up. The CVM is the architectural inversion: a virtual machine that owns the disks below it.
**exam_relevance:** [NCA, NCP-MCI]
**whiteboard_ready:** true

**Elements (bottom to top):**
- Physical layer (full-width rectangle, gray): "Physical Server (CPU / RAM / NICs / NVMe + SSD disks)"
- Hypervisor layer (full-width, muted blue): "Hypervisor (AHV or ESXi or Hyper-V)"
- Above the hypervisor, two side-by-side boxes:
  - **Controller VM (CVM).** Rust/orange, larger, with internal sub-labels: "Stargate · Cassandra · Curator · Zeus · Pithos · Acropolis"
  - **User VMs.** Light gray, smaller, with sub-labels: "VM-01, VM-02, VM-03, ..."

**Connections:**
- User VMs → CVM (sideways arrow): "All storage I/O"
- CVM → Local disks (downward arrow): "Local writes, data locality"
- CVM → Other nodes' CVMs (outward arrow at top): "Replicated writes (RF2/RF3)"

**Annotations:**
- Beside the CVM: "The CVM owns storage. The hypervisor never talks to disks directly."
- Beside the user VMs: "User VMs see normal storage. They do not know the CVM exists."
- Beside the disks: "Disks are passed through to the CVM, not consumed by the hypervisor."

**Why this diagram exists:** To make the architectural inversion obvious. In a traditional server, the hypervisor owns the disks. In Nutanix, the hypervisor delegates that responsibility to a virtual machine running on top of itself. This single inversion is the entire premise of the platform.

---

### The CVM Is the Whole Argument

This is the section that, if you internalize it, makes everything else click.

The Controller VM is a Linux VM that runs on every Nutanix node. It is not optional. It is not configurable away. It is part of the platform. When a Nutanix node boots, the hypervisor comes up first, then the CVM auto-starts, then the cluster forms, then user VMs can run.

The CVM owns the local disks of its node. The hypervisor passes them through (via PCIe passthrough or controller passthrough or similar mechanisms, depending on hypervisor and configuration). From the hypervisor's point of view, those disks are not its problem. The CVM has them.

The CVMs across all nodes communicate over the network. Together, they implement the **Distributed Storage Fabric (DSF)**. Module 5 goes deep on DSF. For now, treat DSF as: *the software running inside the CVMs that takes all the local disks across all nodes and presents them as one logical storage pool.*

When a user VM writes data, here is what happens:

1. The user VM issues a write to its virtual disk (which the hypervisor presents as a block device).
2. The hypervisor sees the write going to what it thinks is shared storage. In AHV, this is presented over an internal interface; in ESXi, it shows up as an NFS datastore presented by the local CVM.
3. The write actually arrives at the CVM, on the same physical box as the user VM.
4. The CVM writes one copy locally, this is **data locality**, and one copy to a CVM on another node (this is **replication**, the basis of RF2).
5. Acknowledgment goes back up the chain.

The user VM does not know any of this happened. To it, this is just disk I/O.

> [!FAMILIAR]
> The user VM experience is identical to vSphere. A Windows VM running on Nutanix sees the same C:\, the same NTFS filesystem, the same disk geometry it would see anywhere. None of the architectural complexity bleeds into the guest. VMware Tools-equivalent tooling (Nutanix Guest Tools, NGT) provides similar guest integration features.

> [!DIFFERENT]
> The "storage controller" is now a virtual machine. In a traditional array, the storage controller is dedicated hardware with its own CPU, memory, OS, and firmware. In Nutanix, that controller is a regular Linux VM running on the same hypervisor as your workloads. This is, simultaneously, the most clever and the most contested decision in the entire product. It is clever because it makes storage software portable, upgradeable, and software-defined. It is contested because that VM consumes resources that would otherwise be available for workloads. We address both honestly below.

---

### The Cycle, Frame Two: The CVM as a "Storage Controller in Software"

Step back and look at this architecturally. In the three-tier world, your storage array has controllers. Those controllers are servers running storage software. They cost money, they consume rack units, they need their own power and cooling, they need their own firmware management, they have their own support contract.

Nutanix asked: what if instead of buying separate storage controller hardware, we just ran the controller as a VM on the same servers we already have? Same CPUs, same RAM, same chassis. We pay a tax (the CVM consumes resources), but we eliminate an entire class of hardware and an entire class of dedicated controller management.

That is the trade. It is a real trade. There are workloads where you would rather have dedicated controllers (we covered those in Module 1). For most workloads, the trade is favorable: the cost of the CVM is less than the cost of separate array hardware, and the operational simplification is substantial.

### The Cycle, Frame Three: The CVM as the Boundary of the System

Here is a third way to think about it, useful for troubleshooting. The CVM is the boundary between *your stuff* and *Nutanix's stuff*. Above the CVM, on each node, are your VMs and your hypervisor. Below the CVM, on each node, are the raw disks. Across the network, between CVMs, is the DSF.

When something goes wrong in storage, you do not go to a separate device. You SSH into a CVM. You run cluster commands. You look at logs in `/home/nutanix/data/logs/`. Storage troubleshooting all happens at the CVM level.

This is genuinely different from a SAN, where you would SSH into the array, into a fundamentally different operating system, with its own CLI and its own troubleshooting language. Here it is Linux. With NCC (Nutanix Cluster Check) commands and Nutanix-specific tools, but Linux underneath.

### The Cycle, Frame Four: The CVM as a Distributed Storage Daemon Set

For the technically-inclined customer, here is the cleanest framing. The CVM runs a set of cooperating Linux services. The important ones, by name:

- **Stargate.** The data path. Every read and write hits Stargate. If Stargate on a node fails, that node's I/O is redirected to other CVMs.
- **Cassandra.** Distributed metadata store (a forked, optimized version of Apache Cassandra). Tracks where every block of every vDisk lives.
- **Curator.** The background scrubber. Runs periodic full and partial scans, rebalances data, handles deduplication/compression/tiering, drives ILM (information lifecycle management).
- **Zeus.** Cluster configuration and quorum management (built on Apache ZooKeeper). The cluster's source of truth for what nodes exist, what services are healthy, who is the master for various roles.
- **Pithos.** vDisk configuration manager. Tracks every vDisk's properties and where its replicas should live.
- **Acropolis.** When running AHV, this is the VM lifecycle / HA / scheduling service. (On ESXi, this responsibility stays with vCenter and ESXi.)

You will see these names in NCC output, log files, and (occasionally) in customer-troubleshooting calls. You don't need to memorize implementation details. You do need to recognize the names and know what each does.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI**
> Stargate, Cassandra, Curator, Zeus, Pithos, Acropolis show up explicitly on NCP-MCI and especially NCM-MCI (which is lab-based and forces you to actually troubleshoot). The most common exam question pattern: identify which service is responsible for a given function. **Cheat sheet:** Data path = Stargate. Metadata = Cassandra. Background scrub/rebalance = Curator. Cluster config/quorum = Zeus. vDisk config = Pithos. AHV VM lifecycle = Acropolis. The trap: distractors will swap Cassandra and Pithos (both involve metadata-ish things) or Curator and Stargate (both involve data movement). Anchor each to its specific responsibility.

---

### Diagram: CVM Services Architecture

**id:** `cvm-services-architecture`
**type:** architecture
**caption:** Inside one CVM. These services run on every CVM in the cluster and coordinate with their peers via the network.
**exam_relevance:** [NCP-MCI, NCM-MCI]
**whiteboard_ready:** false

**Elements:**
- Outer rectangle (rust/orange, dashed border): "Controller VM (CVM)"
- Inside, six service boxes arranged in a 2x3 grid:
  - **Stargate.** Labeled "Data path · I/O processing"
  - **Cassandra.** Labeled "Distributed metadata store"
  - **Curator.** Labeled "Background scrub · Rebalance · ILM"
  - **Zeus.** Labeled "Cluster config · Quorum"
  - **Pithos.** Labeled "vDisk config manager"
  - **Acropolis.** Labeled "AHV VM lifecycle (AHV only)"
- Below the grid, a thin band labeled "CVM OS (Linux) · NCC · ncli/acli endpoints"

**Connections:**
- Bidirectional arrows to "Other CVMs in cluster" (drawn as a peer network outside the CVM box), labeled "Cassandra ring · Stargate I/O · Zeus consensus"
- Down-arrow from CVM to "Local disks" labeled "Pass-through disk access"
- Up-arrow from CVM to "Hypervisor" labeled "Storage presentation (NFS / iSCSI / direct)"

**Why this diagram exists:** Customers and technical interviewers will ask "what's actually running in the CVM?" This diagram is the answer. It also gives you the vocabulary to read NCC output and AOS logs.

---

### Data Locality (The Concept That Matters Operationally)

Nutanix marketing leans hard on the term "data locality." You should understand what it actually means and what it actually does, because it comes up on the exam, in customer conversations, and in real performance discussions.

**Definition:** When a VM writes data, one copy of that data is written to the local disks of the node where the VM is running. The other copy (or two, for RF3) goes to other nodes for replication. As long as the VM stays on its node, future reads come from local disk, no network hop.

**Why this matters:** Reading from local NVMe is faster than reading from a peer CVM over the network. Specifically: local NVMe reads are sub-100µs; network-traversed reads (even on 25/100GbE) add a few hundred microseconds. For most workloads this is invisible. For high-IOPS, low-latency workloads, it's measurable.

**When data locality matters less:** When the cluster is balanced and reads are well cached, the network overhead for non-local reads is small. AOS extensively caches metadata and frequently-read blocks in CVM RAM (the Content Cache).

**What happens on vMotion / live migration:** When a VM moves to a different node, its data does not immediately follow. The VM reads from the original node over the network until Curator (the background service) decides it's worth migrating the data to be local again. This happens automatically over time.

> [!FROM-THE-SA-CHAIR]
> Data locality comes up in two places. **First**, when a customer asks "is local-only access really faster than my SAN's all-flash array?" The answer: for active reads of frequently-touched data, yes, local NVMe beats a SAN round trip. For everything else, the difference is small. **Second**, when a customer running a chatty cluster asks "what happens to performance when a VM moves?" The answer: a brief read penalty over the network, automatically resolved by background data migration. Don't oversell data locality. It's a real architectural advantage, not a magic performance multiplier.

> [!ON-THE-EXAM] **NCA · NCP-MCI**
> Data locality definitions show up on both. NCA tests recognition: "What is data locality?" NCP-MCI tests scenarios: "A VM is migrated from Node A to Node B. What happens to its data?" The correct answer is: data is read remotely from Node A initially, and migrated locally to Node B over time by background services. Trap distractors: "data is migrated immediately during vMotion" (false), or "data must be manually migrated using a CLI command" (false), or "the VM cannot be migrated until data is moved" (false).

---

### The CVM Tax: The Honest Section

This is the section that every BlueAlly SA needs cold. The CVM consumes resources. Customers will ask. You answer with numbers, not adjectives.

**What the CVM consumes (typical defaults, AOS 7.5 generation):**

| Resource | Minimum (basic features) | Typical (with dedup, EC, NCM-IO) | Heavy (large clusters) |
|---|---|---|---|
| vCPU | 8 | 12 | 14-16 |
| RAM | 32 GB | 48 GB | 64+ GB |
| Boot/system storage | ~40 GB | ~40 GB | ~40 GB |

**Aggregate cluster overhead:** On a 4-node cluster with typical CVM sizing (12 vCPU / 48 GB), the CVMs collectively consume **48 vCPUs and 192 GB of RAM** that would otherwise be available for workloads. On a 16-node cluster, that's **192 vCPUs and 768 GB**.

That number sounds large. It is large. It is also smaller than the cost of dedicated storage controllers, dedicated array RAM, and dedicated array CPUs in an equivalent three-tier deployment, which the customer is currently paying for, just on different hardware.

**The honest framing for customers:**

> *"The CVM does consume real CPU and RAM on every node. Roughly 12 vCPUs and 48 GB per node, depending on what features you enable. That's the cost of having the storage controller run as software on the same hardware. In exchange, you eliminate the array's controller hardware, controller licensing, controller-to-storage fabric, and a vendor relationship. For most workloads, the math works. For some workloads it does not, and we can talk about which is which."*

That answer wins more rooms than any feature comparison.

> [!FROM-THE-SA-CHAIR]
> When a customer pushes back hard on the CVM tax, the failure mode is to defend by minimizing. Don't. The right move is to acknowledge it precisely (with numbers) and then reframe the comparison. *"You're not getting compute for free; you're paying for software-defined storage in CPU and RAM instead of in array hardware. Let's compare the total resource footprint, not just the workload-available footprint."* Most three-tier customers haven't done this math. When you walk them through it, the CVM tax goes from a deal-breaker to a side note. Honest numbers beat defensive talking points.

> [!DIFFERENT]
> **vSAN does this too, but differently.** VMware vSAN runs as a kernel module inside ESXi rather than as a VM. The resource consumption is real but harder to measure precisely (it shows up as ESXi overhead rather than as a discrete VM). The architectural debate between Nutanix's "storage as a VM" model and vSAN's "storage as a kernel module" model is genuine. VM-based isolation makes the CVM cleanly upgradeable, debuggable, and portable across hypervisors. Kernel-mode integration makes vSAN slightly leaner on resource overhead but couples it tightly to a specific hypervisor version. Neither approach is objectively better; they are real engineering tradeoffs. Have this answer ready when a vSAN-friendly customer asks.

---

### The Cluster

A Nutanix **cluster** is a set of nodes (minimum three for production; one is allowed for Community Edition home-lab use only; two-node clusters exist for ROBO with a witness VM) running together as a single unit. The cluster is the smallest functional thing in Nutanix. You do not deploy a single Nutanix node and use it. You deploy a cluster.

A cluster has:

- **A shared identity.** A cluster name, a Cluster Virtual IP (VIP), a Prism Element URL.
- **A shared storage pool.** The DSF, made from the union of all nodes' local disks.
- **Shared cluster services.** DNS, NTP, alerting, snapshot scheduling, NCC health checks.
- **A consensus mechanism among the CVMs** (Zeus / ZooKeeper) that keeps them coordinated. This is why you need three CVMs minimum for production: consensus protocols need an odd-numbered quorum.

The cluster is what gets registered to Prism Central (Module 4). The cluster is what your VMs run on. The cluster is what you upgrade as a single unit (LCM, below).

> [!ON-THE-EXAM] **NCA · NCP-MCI**
> Minimum cluster sizes are exam staples. Memorize: production minimum is 3 nodes. Two-node clusters exist for ROBO/edge but require a Witness VM (running elsewhere) for quorum. Single-node "clusters" are Community Edition only and not supported in production. The trap question: "What is the minimum cluster size?" The wrong-but-tempting answer is 2 (because two-node exists). The correct answer for general production is **3**. Two-node is a special case for ROBO with a separate witness.

> [!FAMILIAR]
> A vCenter cluster is conceptually similar. It groups hosts, presents shared resources, manages HA and DRS. The Nutanix cluster does the same, plus it owns its own storage pool, plus it does its own snapshots and DR.

> [!DIFFERENT]
> The cluster owns its storage. A vCenter cluster does not "own" the SAN; the SAN is a separate device that the vCenter cluster *consumes*. A Nutanix cluster *is* its storage. There is no external array to consume. This means a Nutanix cluster cannot be half-down in the way a vSphere cluster can be half-down (with the array still healthy). Nutanix cluster failures and storage failures are tightly correlated. Plan your fault tolerance with this in mind.

---

### Diagram: Cluster Topology

**id:** `nutanix-cluster-topology`
**type:** architecture
**caption:** A 4-node Nutanix cluster. The DSF spans all four nodes; data is replicated between them; there is no external storage device.
**exam_relevance:** [NCA, NCP-MCI]
**whiteboard_ready:** true

**Elements:**
- Four node-boxes laid out left to right, each containing internally:
  - Hypervisor strip (muted blue): "AHV / ESXi"
  - CVM strip (rust/orange): "CVM"
  - User VM strips (gray): "VM-01, VM-02, ..." (varying counts to show realistic placement)
  - Local disk strip at the bottom (gray with disk icons): "NVMe + SSD"
- A horizontal bar above the four nodes (rust/orange): "DSF, Distributed Storage Fabric (single logical pool)"
- Above the DSF bar: a small box labeled "Cluster Virtual IP" (gold/teal accent) connected to all four CVMs

**Connections:**
- Horizontal arrows between adjacent CVMs: "Replication traffic"
- Inter-node arrows showing data redundancy across nodes: "RF2: every block has 2 copies on 2 different nodes"
- VIP connection: "Prism / API / SMB endpoints land here"

**Annotations:**
- "Lose any one node, all VMs survive (HA + RF2). Lose two with RF2, you have a problem."
- "Adding a 5th node adds compute, RAM, and storage proportionally."
- "There is no separate storage array. The cluster is the storage."

**Why this diagram exists:** This is your second whiteboard diagram for customer meetings. It pairs with the node-anatomy diagram from above. Together they answer the two questions every VMware admin asks: "what's on a node?" and "how do nodes work together?"

---

### The Block (Mostly Historical)

A "block" in Nutanix vocabulary is a physical chassis that contains one to four nodes. Early Nutanix appliances (NX-3000 series) were 2U chassis with four nodes each, the whole chassis was a "block." The point was dense compute: four servers in 2U.

Modern Nutanix deployments often run on standard 1U or 2U servers from Dell, HPE, etc., where the chassis contains a single node. In these cases, "block" and "node" become the same thing, and the term loses much of its meaning.

You will still see "block awareness" referenced in fault-domain configuration: the cluster can be configured to ensure that data replicas land on nodes in *different* blocks, so that a chassis failure (rare but possible) does not cost you data. On modern hardware where one node equals one block, block awareness is automatic and uninteresting.

**You do not need to think about blocks much.** Nodes and clusters are what matter. But know the term, because it will appear on the NCA exam.

> [!ON-THE-EXAM] **NCA**
> The NCA may ask you to identify what a "block" is and what "block awareness" means. **Block:** a physical chassis containing 1-4 nodes. **Block awareness:** a fault-domain configuration that places data replicas on nodes in different blocks, protecting against a chassis-level failure. Trap distractor: "a block is a unit of storage." False. A block is hardware (the chassis), not a storage construct.

---

### Foundation, LCM, and NCC: The Operational Trio

Three named tools you must know cold. They are how Nutanix gets deployed (Foundation), upgraded (LCM), and kept healthy (NCC). All three appear in the cert blueprints and all three come up in customer conversations about operational maturity.

**Foundation, the deployment / imaging tool.**

Foundation is the bare-metal deployment tool. You point it at a set of nodes (powered on, with a baseboard management controller, IPMI/iDRAC/iLO, accessible) and it images them, installs the hypervisor, installs the CVM, and forms a cluster. This is the day-zero tool.

Foundation runs as either a standalone application (Foundation Standalone, runs on a laptop) or embedded in the AOS image (Foundation Central, runs on a Prism Central instance for fleet-scale deployment). For a single cluster deployment, Foundation Standalone is sufficient and what most field deployments use.

**Concrete:** to image three new nodes into a 3-node cluster, you launch Foundation, give it the IPMI/iLO/iDRAC IPs of the nodes plus the desired cluster IPs, point it at the AOS and AHV images, and walk away for ~45-60 minutes. At the end you have a working cluster.

> [!FAMILIAR]
> Conceptually similar to vSphere Auto Deploy or PowerCLI host provisioning. Different in that it deploys a full HCI cluster, hypervisor + storage + management, from a single tool. There is no equivalent in legacy three-tier where you'd be using OEM imaging tools, vSphere installers, and array provisioning UIs separately.

**LCM, Life Cycle Manager. The day-two tool.**

LCM is one-click upgrade. You log into Prism Central (or Prism Element), navigate to LCM, and it scans the cluster for current versions of every upgradeable component: AOS, AHV, BIOS, BMC, NIC firmware, drive firmware, NCC, Foundation, Files, Objects, Volumes, NKE, NCM, and on. It then offers a coordinated, ordered upgrade that maintains cluster availability throughout.

This is genuinely a Nutanix strength. The customer comparison: in three-tier, you upgrade vCenter on its schedule, ESXi hosts on theirs, the array firmware on its schedule, the SAN switch on its schedule, the OS on each through Update Manager, usually with separate change windows, separate runbooks, separate vendor coordination. LCM collapses this into one workflow with dependencies handled.

**Limitations to know:** LCM is opinionated. It will not let you upgrade to a combination of versions that hasn't been validated. This is generally a feature, not a bug, but it does mean "I want to upgrade AOS but stay on the older AHV" is sometimes constrained. Always check the LCM dashboard for current compatibility.

> [!FROM-THE-SA-CHAIR]
> One-click upgrade is the customer demo that converts skeptics. When you POC, schedule a 30-minute slot to walk through LCM on a working customer-style cluster (or your CE lab if no POC is available). Show the dependency-resolution UI. Show the rolling upgrade that doesn't take VMs down. This is the operational story Nutanix marketing leans on, and unlike most marketing, it actually delivers the goods. Don't tell, show.

**NCC, Nutanix Cluster Check. The health-check tool.**

NCC is a suite of health and diagnostic checks (currently several hundred individual checks). It runs on a schedule and can also be invoked on-demand. It catches issues across hardware health, cluster configuration, network, storage, replication, and feature-specific checks (e.g., NKE-specific or Files-specific checks).

Run from Prism, from the CVM CLI as `ncc health_checks run_all`, or specifically as `ncc health_checks <category> <check_name>`. Output is a report with PASS / WARN / FAIL / INFO per check.

In production, NCC runs daily by default and emails findings. In troubleshooting, you run it on-demand. In support cases, Nutanix will often ask you to attach NCC output to the ticket. In NCM-MCI exam labs, NCC is the first tool you reach for to diagnose a failing cluster.

> [!ON-THE-EXAM] **NCA · NCP-MCI · NCM-MCI**
> NCC, LCM, and Foundation are tested across all three exams. Memorize the one-liners:
> - **Foundation:** day-zero deployment / imaging
> - **LCM:** day-two upgrades (one-click)
> - **NCC:** ongoing health checks / diagnostics
>
> Trap pattern: questions that swap their purposes ("Which tool is used to image new nodes? Foundation / LCM / NCC / Prism"), pick Foundation. ("Which tool is used to upgrade firmware? Foundation / LCM / NCC / Prism"), pick LCM. ("Which tool is used to run health checks? Foundation / LCM / NCC / Prism"), pick NCC.

---

### What Happens When the CVM Fails

A real, common customer question. Here is the answer, with the right level of detail.

**On a single CVM failure (one node's CVM crashes or is upgrading):**
1. The hypervisor on that node detects the loss of its local CVM.
2. AOS reroutes that node's I/O to a remote CVM (over the network). This is called **autopath** (or, in newer AOS, the I/O simply uses the standard cluster mechanisms, same outcome).
3. User VMs on that node continue running. Performance degrades slightly because their I/O now traverses the network instead of going local.
4. When the CVM recovers, I/O routes back to local.

**On a node failure (the entire physical node, including its CVM, goes down):**
1. AHV / Acropolis (or vSphere HA, if running ESXi) restarts the failed node's VMs on surviving nodes.
2. The cluster has lost one replica of any data that was uniquely on that node's disks. AOS / Curator immediately starts re-replicating from surviving copies to restore RF2 (or RF3). This is the cluster's self-healing.
3. The cluster runs at reduced redundancy until re-replication completes (minutes to hours, depending on cluster size and data volume).

**On multiple simultaneous failures:**
- RF2 cluster, two simultaneous node losses: data loss is possible. (This is why RF3 exists.)
- RF3 cluster, two simultaneous node losses: cluster survives, no data loss. Re-replication restores RF3.
- Larger failure scenarios: see Module 7 on data protection.

> [!FROM-THE-SA-CHAIR]
> "What happens when the CVM fails?" is one of the top three architecture objections you will hear. The right answer, said calmly: *"A single CVM failure is a non-event. The hypervisor detects it, reroutes I/O to a peer CVM over the network for the duration of the outage, and your VMs keep running with slightly elevated latency. The cluster has full self-healing for node failures; we re-replicate data automatically. The CVM is a service, not a single point of failure, every node has one."* Practice that paragraph until it's automatic. It's worth memorizing word for word.

---

## Lab Exercise: Stand Up a 3-Node CE Cluster

> [!LAB] **Time:** ~4 hours · **Platform:** Nutanix Community Edition 2.1

Module 1 had you stand up a single node. Now upgrade your lab to a real cluster. Three nodes is the minimum production size and the correct lab target. You cannot pass NCM-MCI with a 1-node lab. You also cannot internalize cluster behavior, replication, failure handling, or LCM properly without three nodes.

**Prerequisites:** Either three physical machines OR a host capable of running three nested CE VMs (96+ GB RAM recommended for nested).

**Steps (nested approach, on VMware Workstation or ESXi):**

1. **Provision three CE VMs.** Each: 4 vCPU minimum, 32 GB RAM, expose hardware-assisted virtualization, VMXNET3 NIC, all promiscuous-mode-style settings enabled. Each VM gets its own boot disk + hot tier SSD-backed disk (200 GB minimum) + cold tier disk (500 GB minimum).
2. **Boot each from the CE ISO.** Run the installer on each. Set the CVM IP, hypervisor IP, gateway, and netmask for each node. Critical: all three nodes must be on the same subnet (or at least L2-adjacent for the cluster to form).
3. **After install on all three, SSH into the first node's CVM** as `nutanix` (default password `nutanix/4u`).
4. **Form the cluster:**
   ```
   cluster -s <node1_cvm_ip>,<node2_cvm_ip>,<node3_cvm_ip> create
   ```
   This may take 5-15 minutes. Watch the output.
5. **Verify cluster health:**
   ```
   cluster status
   ncli cluster info
   ncc health_checks run_all
   ```
6. **Set the Cluster Virtual IP** via Prism Element (or `ncli cluster set-external-ip-address external-ip-address=<vip>`).
7. **Connect to Prism Element** at `https://<vip>:9440`. You should see all three nodes in the Hardware view. The Storage view should show one Storage Pool with the aggregated capacity from all three.

**What to do once it's up:**

- Open the **Hardware** view in Prism. You should see three nodes and three CVMs.
- Open **Cluster Details**. Note the AOS version, AHV version, and the Cluster VIP.
- Open the **Storage** view. Find the default Storage Pool. Note that it is the union of all three nodes' disks.
- Run `ncc health_checks run_all` from any CVM and read the output. Identify a few of the categories.
- Reboot one of the CVMs (`sudo reboot` from inside the CVM). Watch what happens in Prism. Note the alerts, watch the cluster's automatic recovery.
- Open **LCM**. Even if no upgrades are pending, click into LCM and look at the inventory it generates. This is what you'll demo to customers.

**Optional stretch:**

- Power off a node entirely and watch the cluster's response. RF2 should kick in; you'll see a "data resiliency" alert. Power the node back on.
- Use Foundation Standalone (download separately) to image a fourth node into a different cluster, just to see Foundation in action.

---

## Practice Questions

Twelve questions. Six knowledge-MCQ (NCA-style), four scenario-MCQ (NCP-MCI-style), two open-ended design questions (NCX-MCI-style). The first ten test recognition and discrimination. The last two test architectural reasoning, which is what the NCX-MCI design defense actually evaluates. Read each, answer in your head, then read the explanation.

---

**Q1.** Which of the following best describes the Controller VM (CVM) in a Nutanix cluster?
**Cert relevance:** NCA · NCP-MCI

A) An optional management VM that can be deployed for clusters that need centralized monitoring
B) A privileged Linux VM that runs on every node and owns local storage, presented as shared storage to the cluster
C) A hypervisor that replaces ESXi when AHV is in use
D) A vCenter equivalent that manages the cluster from a single location

**Answer:** B

**Why this answer:** The CVM is a per-node, mandatory, privileged Linux VM that owns the local disks and cooperates with peer CVMs over the network to present a single distributed storage pool.

**Why not the others:**
- A) Wrong on two counts: the CVM is not optional, and its job is not management, it's storage. Management is Prism (Module 4).
- C) The CVM and the hypervisor (AHV or ESXi or Hyper-V) are different things. The CVM runs *on top of* the hypervisor.
- D) Cluster-wide management is Prism (Module 4), not the CVM.

**The trap:** D is the seductive distractor. It conflates "centralized control plane" with "the CVM" because both feel like cluster-level abstractions. The CVM is per-node storage software; Prism is the cluster management plane. Different layers.

---

**Q2.** A Nutanix cluster has three nodes, each configured with 12 vCPUs and 48 GB allocated to its CVM. What is the total CVM-reserved resource footprint for the cluster?
**Cert relevance:** NCP-MCI · sales-relevant

A) 12 vCPU and 48 GB RAM (the CVM is shared across the cluster)
B) 36 vCPU and 144 GB RAM (each node reserves its own CVM resources)
C) 24 vCPU and 96 GB RAM (one CVM is active, others passive)
D) 0 vCPU and 0 GB RAM (the CVM consumes only excess capacity, not reserved)

**Answer:** B

**Why this answer:** Every node runs its own CVM. CVM resources are reserved per-node. Three nodes × 12 vCPU × 48 GB = 36 vCPU and 144 GB total reserved across the cluster.

**Why not the others:**
- A) The CVM is per-node, not shared. Each node runs its own.
- C) There is no "active/passive" CVM model. All CVMs are active simultaneously.
- D) CVM resources are reserved, not opportunistic.

**The trap:** A is a common misconception. New Nutanix learners hear "distributed storage software" and assume a single instance shared across the cluster. The architecture is the opposite: a coordinated set of per-node instances. Memorize: CVMs are per-node and reserved.

---

**Q3.** What is the minimum number of nodes required for a production Nutanix cluster?
**Cert relevance:** NCA

A) 1
B) 2
C) 3
D) 4

**Answer:** C

**Why this answer:** Three nodes is the production minimum. This is driven by the consensus requirements of the Zeus / ZooKeeper service that maintains cluster configuration; an odd-numbered quorum is required and three is the smallest viable.

**Why not the others:**
- A) Single-node "clusters" are Community Edition only and not supported in production.
- B) Two-node clusters exist for ROBO/edge scenarios but require a separately-deployed Witness VM (running on a different cluster or on a small VM) to provide quorum. Standalone two-node is not production.
- D) Four works but is not the minimum.

**The trap:** B. Test-takers who have heard about ROBO two-node deployments may pick B, but two-node requires the Witness and is a special-case configuration. The unqualified answer for "minimum production cluster" is 3.

---

**Q4.** A user VM is migrated (live) from Node A to Node B in a Nutanix cluster. What happens to the VM's data?
**Cert relevance:** NCP-MCI

A) The data is migrated synchronously during the live migration; the VM cannot start on Node B until all data is moved
B) The data remains on Node A; reads from the VM on Node B traverse the network until background services migrate the data over time
C) The data is duplicated to Node B during migration, leaving copies on both nodes permanently
D) Data locality requires the VM to be migrated back to Node A for reads to be local again

**Answer:** B

**Why this answer:** Live migration moves the VM, not its data. After migration, the VM on Node B reads from Node A's disks over the network. Curator (the background service) eventually migrates the data to be local to the VM's new node.

**Why not the others:**
- A) Migration would be impossibly slow if data had to move synchronously. AOS doesn't work this way.
- C) Permanent duplication would defeat the storage efficiency mechanisms. Data does not stay on both nodes.
- D) The VM doesn't need to migrate back; data follows the VM eventually via background processes.

**The trap:** A is intuitive ("of course the data has to move with the VM!") and wrong. The architectural insight: data locality is a *gradual optimization*, not a *prerequisite for migration*. The cluster is designed so that remote reads work fine; locality is the optimized state, not the only state.

---

**Q5.** Which CVM service is responsible for the data path, handling read and write I/O for user VMs?
**Cert relevance:** NCP-MCI · NCM-MCI

A) Cassandra
B) Curator
C) Stargate
D) Zeus

**Answer:** C

**Why this answer:** Stargate is the data path. Every user VM read and write is processed by Stargate.

**Why not the others:**
- A) Cassandra is the distributed metadata store. It tracks where data lives but doesn't process I/O.
- B) Curator is the background scrubber and rebalancer. It runs periodic scans, not the active data path.
- D) Zeus is cluster configuration and quorum management. It tracks cluster state, not I/O.

**The trap:** Test-writers love to make Cassandra and Stargate distractors for each other because both deal with "data" in some sense. Anchor: **Cassandra = metadata, Stargate = data.** Memorize that distinction.

---

**Q6.** Which Nutanix tool is used to perform a coordinated upgrade of AOS, AHV, and node firmware in a single workflow?
**Cert relevance:** NCA · NCP-MCI

A) Foundation
B) LCM (Life Cycle Manager)
C) NCC (Nutanix Cluster Check)
D) Prism Central

**Answer:** B

**Why this answer:** LCM is the day-two upgrade tool. It scans for current versions across AOS, AHV, BIOS, BMC, NIC firmware, drive firmware, and other upgradeable components, then performs coordinated, ordered upgrades.

**Why not the others:**
- A) Foundation is the day-zero tool, initial deployment / imaging of new nodes.
- C) NCC is the health-check tool. It runs diagnostic checks; it does not perform upgrades.
- D) Prism Central is the management plane. LCM runs *within* Prism Central (or Prism Element), but the question is asking for the specific tool, which is LCM.

**The trap:** D is technically adjacent (LCM is launched from Prism), but the question asks which tool performs the upgrade workflow, which is LCM specifically. Read the question precisely.

---

**Q7.** A customer's VMware administrator says: "The CVM tax sounds like it's eating 30% of every node. How can that possibly be more efficient than a dedicated array?" What is the strongest SA response?
**Cert relevance:** sales-relevant (NCX-MCI tone)

A) "The CVM is much smaller than you think, it only uses idle resources."
B) "We have very efficient CVMs that don't impact your workloads."
C) "Yes, the CVM reserves real CPU and RAM, typically 12 vCPUs and 48 GB per node. In exchange, you eliminate the array's controller hardware, controller licensing, dedicated array RAM, and a separate vendor relationship. Let's compare total resource footprints rather than just workload-available footprints."
D) "vSAN does the same thing as a kernel module, so the comparison is moot."

**Answer:** C

**Why this answer:** Acknowledge the tax with specific numbers. Reframe to total cost (including the resources the customer is currently paying for in their array). Invite a real comparison.

**Why not the others:**
- A) Untrue. The CVM reserves resources; it doesn't run on idle.
- B) Vague defensive language. Customers have heard this from every vendor and tune it out.
- D) True (vSAN does have its own overhead) but evasive, it doesn't address the customer's actual question, just deflects to a competitor.

**The trap:** A and B are the natural defensive responses. Practiced SAs resist the urge to defend and answer with numbers. The customer is not testing whether you'll back down; they're testing whether you'll be honest.

---

**Q8.** A Nutanix cluster has lost one node due to a hardware failure. The cluster is configured for RF2. What is the cluster's state?
**Cert relevance:** NCP-MCI · NCM-MCI

A) The cluster is offline until the failed node is replaced
B) The cluster is online and self-healing; user VMs from the failed node have been restarted on surviving nodes; data is being re-replicated to restore RF2
C) The cluster requires manual intervention to restart user VMs and re-replicate data
D) The cluster is in read-only mode until RF2 is restored

**Answer:** B

**Why this answer:** Single-node failure with RF2 is a recoverable scenario. Hypervisor HA restarts the VMs on surviving nodes; AOS / Curator automatically re-replicates data to restore RF2. No manual intervention required for the basic recovery flow.

**Why not the others:**
- A) RF2 is specifically designed to survive a single-node loss. The cluster does not go offline.
- C) AOS automation handles VM restart (via AHV Acropolis or vSphere HA) and data re-replication automatically.
- D) Read-only mode is not a normal AOS state. The cluster remains read-write.

**The trap:** C is plausible if you don't trust automation. The answer requires you to know that AOS is genuinely self-healing for single-node failures, which is the whole point of RF2.

---

**Q9.** Which of the following describes "block awareness" in a Nutanix cluster?
**Cert relevance:** NCA · NCP-MCI

A) The ability of the cluster to track storage block utilization across nodes
B) A fault-domain configuration that ensures data replicas are placed on nodes in different physical chassis (blocks)
C) A diagnostic feature that reports on block-level I/O performance
D) A cluster-wide feature flag that enables block-level deduplication

**Answer:** B

**Why this answer:** A "block" is a physical chassis containing one or more nodes. Block awareness ensures that if you have multiple nodes in the same chassis, replicas are placed in different chassis, protecting against a chassis-level failure.

**Why not the others:**
- A) "Storage block utilization" sounds related but conflates two senses of "block" (storage block vs hardware chassis). Block awareness is about hardware fault domains.
- C) Not a feature.
- D) Deduplication is unrelated to block awareness.

**The trap:** The word "block" is used in two senses (storage block, hardware block/chassis). Test-writers exploit this. Block awareness is hardware. Storage blocks are different concepts.

---

**Q10.** You are troubleshooting a cluster where one CVM is reported by Prism as "down." What is the appropriate first diagnostic step?
**Cert relevance:** NCM-MCI

A) Reboot the affected node's hypervisor
B) Run `ncc health_checks run_all` from a working CVM and review the output
C) Initiate a cluster-wide shutdown to reset the cluster state
D) Replace the failed node's hardware

**Answer:** B

**Why this answer:** First step in any cluster issue is to gather diagnostic data. NCC is the canonical health-check tool; running the full suite gathers data across cluster, hardware, network, and storage layers. From there you can target specific issues.

**Why not the others:**
- A) Rebooting the hypervisor is premature without diagnostics. It might also be unnecessary or harmful.
- C) Cluster-wide shutdown is a drastic action that's almost never the first step.
- D) Replacing hardware before diagnosing is the most expensive possible mistake.

**The trap:** A and C are both "do something" answers. The discipline in operational troubleshooting is to gather data before taking action. NCM-MCI specifically tests this discipline.

---

**Q11.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep
**Format:** Short-answer / design discussion. Write your reasoning out loud. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A customer is sizing a new 12-node Nutanix cluster to consolidate three existing environments: a Tier-1 OLTP database (Microsoft SQL Server, 8 TB, latency-sensitive, mostly random reads with bursts of writes), a VDI deployment (800 desktops, predictable boot storms at 8am, persistent profiles), and a general-purpose VM tier (~200 mixed VMs, file/print/web/internal apps). They are debating RF2 vs RF3 across the board, asking how to size CVMs, and asking whether to mix all three workloads in one cluster or split into multiple clusters.

**The challenge:**
Walk through your architectural reasoning. Recommend a configuration. Acknowledge tradeoffs. Identify the questions you still need answered.

**A strong answer covers:**
- **Cluster topology recommendation:** one cluster vs multiple. Most practitioners would consolidate into one cluster at this scale; the only strong reason to split is regulatory or operational isolation. State the recommendation and the reasoning.
- **RF decision per workload (or globally):** RF2 is sufficient for VDI and general-purpose; many architects make the case for RF3 on the OLTP database for resilience during long Curator scans on large datasets. Some architects argue RF2 plus solid backup is the right answer. Both are defensible. Naming the tradeoff explicitly is the point.
- **CVM sizing:** acknowledge that enabling deduplication (often valuable for VDI with linked clones) and capacity tiering will push the CVM toward 48-64 GB RAM and 12-16 vCPUs per node. Mixed-workload clusters typically size CVMs to the heavier configuration.
- **Storage tiering and capacity:** all-NVMe vs hybrid. For Tier-1 OLTP plus VDI boot storms, all-NVMe is typically warranted. Discuss data locality implications for the OLTP VM (it benefits if the VM stays on its node; pin it via affinity if the customer's HA tolerance permits).
- **Failure domain math:** with 12 nodes and RF2, the cluster tolerates a single-node loss with no data loss; calculate effective usable capacity (roughly 50% of raw after RF2 and reserved overhead). With RF3, usable drops to roughly 33%. State the cost.
- **Open questions to ask the customer:** backup strategy, RTO/RPO for the database, network topology, whether the OLTP needs synchronous DR (Module 7), licensing model preference (AOS Pro vs Ultimate, Module 9), and whether they want hypervisor consolidation (AHV) or are staying on ESXi.

**A weak answer misses:**
- Recommending RF3 globally without acknowledging the 33% usable-capacity penalty.
- Recommending all-NVMe without acknowledging cost.
- Not asking about backup strategy (a Nutanix cluster is not a backup target for itself).
- Glossing over the CVM sizing implications of feature enablement.
- Not naming the workload affinity question for the OLTP VM.

**Why this question matters for NCX:** The NCX-MCI exam is a live design defense. You will sit in front of a panel and walk through a customer scenario very much like this one. The panel does not look for one correct answer; they look for whether you can reason through tradeoffs, name what you don't know, and ask for the missing data. Practice this now. Talk out loud. Write your version. The first time you try it, you will discover gaps. That is the point.

---

**Q12.** *(NCX-style design defense, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant in customer-architecture conversations
**Format:** Short-answer / architectural defense. Write your response.

**The scenario:**
A customer's senior VMware architect, who knows vSAN well, raises the following challenge during a technical deep-dive:

> *"Running storage as a VM means context-switching overhead between the hypervisor and the CVM, scheduling contention with workload VMs, network round-trips for things vSAN handles in-kernel, and a Linux OS attack surface I don't have with vSAN's kernel module. vSAN is more efficient by design. Why is the Nutanix CVM model defensible?"*

**The challenge:**
Respond. The customer is technical, well-prepared, and not interested in marketing. Address each of his technical claims. Give him a fair comparison. Do not bash vSAN.

**A strong answer covers:**
- **Acknowledge the kernel-mode advantage where it exists:** vSAN does avoid VM-to-hypervisor context switches for storage I/O. That is a real efficiency. The question is how much it actually matters at the workload's IOPS profile.
- **Reframe the comparison:** the CVM-as-VM model has compensating advantages. (a) The CVM is hypervisor-portable, which lets Nutanix run on ESXi, AHV, and Hyper-V. (b) The CVM is independently upgradeable, which lets AOS roll forward without a hypervisor-coupled release cadence. (c) Memory and CPU isolation is explicit and observable, vs vSAN's overhead which is hard to attribute.
- **Address the "scheduling contention" claim:** the CVM has reserved resources (CPU and RAM); it does not compete with workload VMs for those reserved resources. The hypervisor enforces the reservation. The contention argument applies to *unreserved* resources, which is the same situation in any virtualized environment.
- **Address the "network round-trip" claim:** AOS uses data locality specifically to avoid the network round-trip for hot data. For data not in local cache, both vSAN and Nutanix incur a network hop in different shapes (vSAN cross-host I/O over its own network; Nutanix CVM-to-CVM). The architectures both pay this cost; the question is throughput and latency under specific workload profiles, which is what POCs are for.
- **Address the "Linux attack surface" claim:** valid. The CVM is a Linux VM and you have to keep AOS patched. Nutanix patches AOS through LCM with one-click. vSAN's surface is the ESXi kernel, which is a different (smaller) target but with no less serious patching cadence. Both vendors run their own security programs.
- **Close with the meta-argument:** "Both architectures have real engineering tradeoffs. The right answer for your environment depends on whether you want hypervisor portability and independent upgrade cadence (Nutanix), or single-stack VMware integration with kernel-level efficiency (vSAN). I am not claiming one is universally better. I am claiming the CVM model is engineered, defensible, and has won over plenty of customers who started where you are. Let us run a POC against your specific workloads."

**A weak answer misses:**
- Defending the CVM model on marketing grounds rather than technical grounds.
- Bashing vSAN. (Customer respect drops.)
- Conceding too much. (CVM scheduling contention is, in fact, mitigated by reservations.)
- Not naming the portability advantage, which is the durable architectural argument.
- Not closing with a POC offer. (Technical defense in customer conversations should always end with "let us prove it on your workloads.")

**Why this question matters for NCX:** This is the kind of architectural challenge that comes up in NCX design defenses, in vendor-final selection meetings, and in technical due-diligence sessions. The skill being tested is your ability to defend your platform's architecture against an informed competitor's argument *without* losing technical credibility. Practice it. Out loud. With a colleague playing the customer if possible.

---

## What You Now Have

You can now draw a Nutanix node and a Nutanix cluster on a whiteboard from memory. You can name what's running on a node, hypervisor, CVM, user VMs, and you can explain why the CVM exists, what it owns, and what it costs.

You have four different mental models for the CVM: the storage controller in software, the boundary of the system, the architectural inversion, and the Linux distributed storage daemon set. When a customer pushes on the architecture, you have a frame ready that fits the conversation.

You know the CVM tax with numbers (8-16 vCPU, 32-64 GB RAM per node, scaling with feature set) and you can defend it without flinching by reframing the comparison to total resource footprint.

You know the cluster minimum (three for production, two with witness for ROBO), the consensus reason it has to be at least three, and what happens on single-node failure (HA restart + automatic re-replication).

You know the operational trio: **Foundation** for day-zero, **LCM** for day-two upgrades, **NCC** for ongoing health. You can identify which tool answers which question on the exam, and you can sequence them in a customer demo.

You have twelve practice questions worth of architectural discrimination. Ten cover CVM purpose, cluster sizing, data locality on migration, CVM services (Stargate, Cassandra, Curator, Zeus), block awareness, failure recovery, and operational troubleshooting. Two are open-ended NCX-style design questions covering mixed-workload cluster sizing and architectural defense against a vSAN-savvy customer. The first ten earn you points on NCA, NCP-MCI, and the foundation for NCM-MCI. The last two are practice for the kind of design reasoning the NCX-MCI defense panel actually evaluates.

You have four customer-facing answers ready: the CVM tax response, the CVM-failure response, the architecture-explanation framing, and the LCM demo path.

You are now ready to look at the hypervisor question, AHV, what it is, what it does, what it lacks compared to ESXi, and when it's the right answer or the wrong one. That is Module 3.

---

## Cross-References

- **Previous:** [Module 1: HCI Foundations](./01-hci-foundations.md)
- **Next:** [Module 3: AHV, The Hypervisor Question](./03-ahv-hypervisor.md)
- **Glossary:** [CVM](./appendix-a-glossary.md#cvm) · [DSF](./appendix-a-glossary.md#dsf) · [Stargate](./appendix-a-glossary.md#stargate) · [Cassandra](./appendix-a-glossary.md#cassandra) · [Curator](./appendix-a-glossary.md#curator) · [Foundation](./appendix-a-glossary.md#foundation) · [LCM](./appendix-a-glossary.md#lcm) · [NCC](./appendix-a-glossary.md#ncc) · [Data Locality](./appendix-a-glossary.md#data-locality) · [Block](./appendix-a-glossary.md#block) · [Cluster VIP](./appendix-a-glossary.md#cluster-vip)
- **Comparison Matrix:** [Architecture · Storage Controller Row](./appendix-b-comparison-matrix.md#storage-controller) · [Lifecycle Tooling Row](./appendix-b-comparison-matrix.md#lifecycle)
- **Objections:** [#2 "The CVM tax is too high"](./appendix-d-objections.md#obj-002) · [#3 "What if a CVM fails?"](./appendix-d-objections.md#obj-003) · [#8 "Why not just use vSAN?"](./appendix-d-objections.md#obj-008) · [#18 "I'm worried about Linux VMs running my storage"](./appendix-d-objections.md#obj-018)
- **Discovery Questions:** [Q-ARCH-04 Existing storage controller architecture](./appendix-e-discovery-questions.md#q-arch-04) · [Q-CAP-01 Workload CPU/RAM ratios](./appendix-e-discovery-questions.md#q-cap-01) · [Q-CAP-02 Growth pattern (linear vs bursty)](./appendix-e-discovery-questions.md#q-cap-02)
- **Sizing Rules:** [CVM resource overhead per node](./appendix-f-sizing-rules.md#cvm-overhead) · [Cluster minimum sizing](./appendix-f-sizing-rules.md#minimum-cluster)
- **CLI Reference:** [`cluster` command](./appendix-g-cli-reference.md#cluster) · [`ncli` basics](./appendix-g-cli-reference.md#ncli) · [`ncc` health checks](./appendix-g-cli-reference.md#ncc)
- **Reference Architectures:** [Small enterprise (3-8 nodes)](./appendix-i-reference-architectures.md#ra-small) · [Mid-market (8-12 nodes)](./appendix-i-reference-architectures.md#ra-mid)

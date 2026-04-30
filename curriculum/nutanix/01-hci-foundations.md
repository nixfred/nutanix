---
module: 01
title: What HCI Is and Why It Exists
estimated_reading_time: 22 min
prerequisites:
  - vSphere/ESXi familiarity
  - Working knowledge of SAN/NAS storage
  - Operational experience with three-tier infrastructure
key_terms:
  - HCI (Hyperconverged Infrastructure)
  - Three-tier architecture
  - Scale-out vs scale-up
  - Distributed storage
  - Software-defined storage
  - Converged Infrastructure (CI)
diagrams:
  - three-tier-vs-hci
  - hci-resource-tax-visual
cert_coverage:
  NCA: ~12%
  NCP-MCI: ~5%
  NCM-MCI: ~2%
sa_toolkit:
  related_objections: [obj-001, obj-007, obj-014]
  related_discovery: [q-arch-01, q-arch-02, q-arch-03]
---

# Module 1: What HCI Is and Why It Exists

> **Cert coverage:** NCA (~12%) · NCP-MCI (~5%) · NCM-MCI (~2%)
> **SA toolkit:** Objections #1, #7, #14 · Discovery Q-ARCH-01 through Q-ARCH-03

---

## The Promise

By the end of this module you will:

1. **Explain HCI without using a single marketing word.** Out loud, on a customer's whiteboard, in two minutes.
2. **Pass every HCI-fundamentals question on NCA.** This module covers roughly 12% of the NCA blueprint, the part that builds the entire foundation for everything else on the exam.
3. **Walk into a discovery call with three questions ready** that determine whether the customer should be looking at HCI in the first place.
4. **Recognize when HCI is the wrong answer**, and say so to a customer. That posture buys you trust the rest of the deal runs on.

Here's the thing about HCI as a category. Everyone you talk to has an opinion. Most of those opinions are five years old. Your job, before you can sell or design anything specific to Nutanix, is to know the category cold. Until you do, every Nutanix-specific concept will land softly. After you do, the rest of this curriculum compounds.

This module is the ground.

---

## Foundation: What You Already Know

You manage (or have managed) a vSphere environment. Right now, on a whiteboard, you can sketch the physical and logical layers without thinking:

1. A rack (or many) of ESXi hosts. **Compute.**
2. A storage array. FC or iSCSI. NetApp, Pure, Dell, EMC. **Storage.**
3. A network fabric tying them together. Top-of-rack switches, often a separate storage network. **Network.**

Three tiers. Three vendors, often. Three teams: compute, storage, network. Three refresh cycles. Three support contracts. Three sets of firmware. Three places a problem can be hiding when a VM goes slow at 3am.

This three-tier model has been the dominant pattern in enterprise virtualization for two decades. It is not stupid. It exists because, historically, building a really good storage array required different silicon, different software, different expertise than building a really good compute server. Specialization paid off.

The Foundation question is: **what changed?**

Two things changed.

**First**, x86 servers got obscenely fast and obscenely cheap. The CPU in a modern dual-socket server has more capability than a 2010-era midrange storage array's controller, and there are 64 of them in a rack. The hardware that used to be "specialized storage controller" is now "what you have lying around."

**Second**, software got better at distributed coordination. Distributed file systems, consensus protocols, software-defined networking. The kind of thing Google was doing in 2005 became table stakes for infrastructure software by 2015.

Hold those two facts in your head. They are why HCI exists, and they are the answer when a customer asks you why this is a category and not a fad.

> [!FROM-THE-SA-CHAIR]
> When a customer says, "I've been running three-tier for 15 years, why change?" you do not lead with feature comparisons. You lead with what changed in the underlying technology that made HCI possible. *"Two things flipped: x86 caught up to dedicated controllers, and distributed software got mature. That's not a Nutanix thing. That's a category shift. The question isn't whether to look at HCI; it's which HCI, and for which workloads."* That answer earns you the next 30 minutes. Feature lists earn you 30 seconds.

---

## Core Content

### What HCI Actually Is (Plain English Version)

Hyperconverged infrastructure is a software architecture in which compute, storage, and (sometimes) networking are delivered by software running on the same physical servers, with no separate storage array.

Read that again. There is no SAN. There is no array. There are servers, each with local disks, and software that takes those local disks across all servers and presents them as a single shared pool.

That's it.

The compute layer is your hypervisor (ESXi, AHV, Hyper-V). The storage layer is software running on each host that pools the local disks. The pool is shared across the cluster. Every VM sees one logical "datastore" (or its equivalent) regardless of which host it runs on.

> [!ON-THE-EXAM] **NCA · NCP-MCI**
> The NCA blueprint tests whether you can identify HCI vs converged infrastructure vs traditional three-tier in a multiple-choice setup. The trap: distractors will use the word "converged" loosely. **Converged Infrastructure (CI)** like VxBlock or FlexPod is *prepackaged three-tier*, not HCI. **Hyperconverged** specifically means the storage array has been collapsed into the compute layer via software. If a question describes "a vendor-integrated stack with prepackaged compute, storage, and network components," that is CI, not HCI. If it describes "a software architecture where local disks across nodes form a shared storage pool," that is HCI.

---

### The Cycle, Frame Two: The Procurement Angle

You have lived the three-tier procurement cycle. Compute refresh in year one. Storage refresh in year three. They never align. Your storage vendor sells you a controller upgrade that requires a forklift migration. Your network team needs to certify firmware against both the new compute and the existing storage. The Visio diagrams take six months to update.

HCI collapses this. One platform. One refresh cadence. You add capacity by adding nodes, which give you more compute and more storage at the same time, in the same proportion, from the same vendor, with one upgrade procedure (LCM, in Nutanix-land, Module 4).

This is the procurement pitch. It is not wrong. It is also not the most interesting part.

### The Cycle, Frame Three: The Architectural Angle

In a three-tier world, compute and storage have *independent fate*. A storage array failure takes down storage but not compute. A compute host failure takes down compute but not storage. Your HA model treats them as separate things to protect.

In an HCI world, compute and storage **share fate at the node level**. A node failure is both a compute failure and a (partial) storage failure simultaneously. The software has to handle this. It does, by replicating data across nodes so that any single node loss is recoverable. But the *architectural assumption* is fundamentally different.

This is why HCI is interesting and also why HCI is, in some scenarios, the wrong answer. We will return to this.

---

### Diagram: Three-Tier vs HCI

**id:** `three-tier-vs-hci`
**type:** comparison
**caption:** The same workload in two architectural models. The storage array vanishes; its function is now distributed across the same nodes that run compute.
**exam_relevance:** [NCA, NCP-MCI]
**whiteboard_ready:** true

**Elements (left side, "Three-Tier"):**
- Top row: 4 boxes labeled "ESXi Host" (compute layer, muted blue)
- Middle: a fabric box labeled "SAN / Storage Network" (network, teal)
- Bottom: a single large box labeled "Storage Array (FC/iSCSI)" (storage, gray)
- Side annotations: "Compute team", "Network team", "Storage team", "3 refresh cycles", "3 vendors", "Independent fate"

**Elements (right side, "HCI"):**
- 4 stacked boxes, each containing internally: hypervisor section (blue), CVM/storage software section (rust), local disks section (gray)
- Horizontal bar above: "Distributed Storage Fabric, single logical pool"
- Side annotations: "One platform", "One team", "One refresh cycle", "Add a node = scale-out", "Shared fate at node level"

**Connections:**
- Left: arrows from each ESXi host down through the SAN to the array
- Right: horizontal arrows between the four nodes showing data replication

**Annotations:**
- Center label between diagrams: "Same workload. Different architecture."
- Beneath HCI side: "The 'storage array' is the cluster itself."

**Why this diagram exists:** To make it visually obvious HCI is not three-tier with a different label. The storage layer has been *consumed into* the compute layer.

> [!FROM-THE-SA-CHAIR]
> This is your whiteboard diagram for first customer meetings. Memorize it. When a VMware admin says "show me what makes Nutanix different from my current setup," you draw the left side first (their world), then redraw the right side (where they're going). Two boxes, one cluster pool. Takes 90 seconds. The diagram does the talking.

---

### Scale-Up vs Scale-Out

The next concept the reader needs is the shape of growth.

Three-tier storage scales **up**. When the array fills, you buy bigger controllers and more shelves. Eventually you hit the controller's ceiling and you replace the array. There is a top end.

HCI scales **out**. To grow capacity or performance, you add a node. Adding a node adds CPU, RAM, and storage proportionally. The cluster gets bigger, not denser. There is no controller to outgrow because there is no controller.

This sounds like a small distinction. It is not. It changes how you forecast capacity, how you justify spend, and how you negotiate with finance. In the three-tier world, you spend big once every three to five years. In the HCI world, you spend smaller amounts more frequently and you tie growth directly to demand.

> [!FAMILIAR]
> You already understand this pattern. You have probably watched your VMware cluster grow from 4 hosts to 8 to 12. That is scale-out compute. HCI applies the same pattern to storage.

> [!DIFFERENT]
> Where the model breaks down: with three-tier storage, you can size storage independently of compute. If you have a workload that needs 500 TB but only 4 vCPUs, you buy a small compute footprint and a big storage footprint. With HCI, every node brings both. If your workload is wildly imbalanced toward storage or toward compute, HCI forces you into uneconomic node configurations or into specialized "storage-heavy" nodes that erode the simplicity HCI was supposed to deliver.

> [!ON-THE-EXAM] **NCA · NCP-MCI**
> Both exams test the scale-out concept. The classic distractor: a question about "how do you increase storage capacity in a Nutanix cluster" with options like "add disk shelf to controller," "expand the LUN," "add a node," "increase RF." The correct answer is **add a node** (occasionally "expand storage with new disks" if the node has empty slots, but the canonical Nutanix answer for capacity expansion is adding a node). The trap answers reflect three-tier thinking. Anchor your answer to scale-out.

---

### Building the Fence: What HCI Is Not

Confusion in this category is rampant because every vendor uses the word "converged" or "hyperconverged" to mean something slightly different. Build the fence:

**HCI is not "converged infrastructure" (CI).** Converged infrastructure (VCE/VxBlock, FlexPod, UCS-with-NetApp reference architectures) is *prepackaged three-tier*. Still a separate array, still separate compute, still separate fabric. The vendor integrated procurement and support. Every part is still distinguishable. HCI removes the array.

**HCI is not "all-in-one appliances" of the SMB variety.** A $5,000 box that runs a hypervisor with local disks is just a server. HCI is specifically a clustered, distributed, software-defined storage architecture. Single-node "HCI" claims are mostly marketing.

**HCI is not the same as software-defined storage (SDS) on its own.** SDS is a component. Pure SDS solutions (Ceph, GlusterFS, ScaleIO, StorPool) provide a distributed storage layer but don't necessarily integrate with a hypervisor management plane. HCI is SDS plus virtualization plus management, sold as one operational unit.

**HCI is not composable infrastructure.** Composable (HPE Synergy and similar) lets you assemble logical servers from disaggregated pools of compute, storage, and fabric over a high-speed backplane. It is, in a sense, the architectural opposite: where HCI fuses the layers together at the node, composable splits them apart at the rack. Both claim flexibility. They go in opposite directions.

**HCI is not cloud.** Running an HCI cluster in your datacenter does not give you AWS. You still own the hardware, the lifecycle, the capacity planning. HCI does, however, present a shared substrate that makes it easier to *consume* like a cloud (single API, declarative provisioning), which is why most "private cloud" implementations sit on HCI today.

> [!ON-THE-EXAM] **NCA**
> The NCA explicitly tests HCI vs CI vs SDS distinctions. Common question pattern: a description of an architecture, four labels (HCI, CI, SDS, Composable), pick the right one. **Memorize:** "Storage array eliminated, distributed across nodes" = HCI. "Prepackaged three-tier with vendor integration" = CI. "Distributed storage layer alone, no virtualization stack" = SDS. "Disaggregated pools assembled into logical servers" = Composable.

---

### Where HCI Is the Wrong Answer

This is the section nobody at a vendor conference will give you. Read it twice. As a BlueAlly SA, this is the section that buys you customer trust. When you say *"actually, for that workload, HCI is the wrong answer,"* the customer hears that you're not selling. After that, when you do recommend Nutanix for the rest of the workloads, the recommendation lands with weight.

**1. Workloads with extreme storage-to-compute imbalance.** Backup repositories that need petabytes of cheap, slow storage and almost no compute. Massive media archives. These workloads still belong on dedicated, dense storage. Putting them on HCI means buying a lot of CPUs you will not use just to get the disks attached. *(BlueAlly play: in these conversations, position Nutanix Files / Mine for backup or partner with a storage-heavy partner. Don't force HCI where it doesn't fit.)*

**2. Workloads with hard, predictable, latency-sensitive performance contracts.** A Tier-0 OLTP database with a 99.999% latency SLA at the single-digit-millisecond level may still benefit from a dedicated all-flash array. HCI has gotten dramatically better here, but for the genuine top of the performance pyramid, dedicated arrays still win on tail latency. *(Note: NVMe-heavy modern Nutanix nodes close most of this gap. Have specifics ready: AOS 7.5 + all-NVMe routinely delivers sub-millisecond latency for OLTP. Don't concede more than the data forces you to.)*

**3. Environments where storage and compute teams are politically separate and will remain so.** This is not a technology problem; it is an org problem. HCI assumes one team owns the platform. If your customer's storage team will fight to the death to keep their kingdom, HCI causes damage that exceeds the technical wins. *(Discovery question: "Who would own the platform on day one, and is that a fight or a foregone conclusion?")*

**4. Very small environments.** Two or three VMs do not need distributed storage. A pair of clustered servers with shared storage (or local disks and a backup) is simpler and cheaper. HCI pays off when you have at least three nodes, ideally more. *(There is a 2-node ROBO Nutanix offering, but the economics only make sense above a certain scale.)*

**5. Cases where the workload genuinely belongs in cloud.** If the question is "should this run on HCI in our datacenter or in AWS," the answer might be AWS, or NC2 (Nutanix on AWS bare metal, Module 8). HCI is a compelling on-prem story; it is not always a compelling on-prem-vs-cloud story.

> [!FROM-THE-SA-CHAIR]
> Memorize these five. When a discovery call surfaces one of them, you say so directly. *"From what you're describing, HCI is probably the wrong answer for that specific workload. Let's talk about the rest of the environment."* That sentence has won more deals for SAs than any feature list. Customers can smell sales pressure. They cannot smell honesty often enough.

---

### Why HCI Wins Where It Wins

The other side, with equal honesty.

**Operational simplicity is not a hand-wave.** When you halve the number of vendors, halve the number of upgrade cycles, halve the number of escalation paths, you get back team capacity that you can apply elsewhere. This compounds. Real customers report 50-70% reduction in infrastructure operational hours after migration. (Source: standard Nutanix customer-evidence anecdotes; verify against your own customer base before quoting specific percentages.)

**The economics of scale-out match the economics of demand.** Capacity grows in the same shape as need. You do not pre-pay for a forklift you might not use.

**Software-driven means automation-friendly.** Distributed storage exposing a clean API (Nutanix v4 REST) is dramatically easier to automate than a SAN with a vendor-specific CLI and a CIM provider. Infrastructure-as-code is the natural fit.

**One control plane is genuinely better than three.** Once you have used a single management console that knows about VMs, storage, and capacity simultaneously, going back to "open vCenter, open the array UI, open Solarwinds, correlate manually" is painful.

> [!FROM-THE-SA-CHAIR]
> When you're in front of a customer and they ask "what's the actual benefit?", do not lead with technology. Lead with what changes operationally. *"Three vendor support contracts become one. Three patch cycles become one. Three teams converge on one platform. The team capacity you free up is the real ROI, the licensing math is real but it's secondary."* Operational simplification is the durable benefit. Licensing math changes; ops simplification compounds.

---

### The Honest Verdict on HCI as a Category

HCI is not a religion. It is an architectural pattern that fits a wide and growing band of enterprise workloads and does not fit a narrow but real set of others. Treat it as a tool. Match the tool to the job.

For the BlueAlly SA in 2026, the relevant question is not "is HCI good." The relevant question is: *which* HCI, *for which workloads*, *under what licensing model*, and given Broadcom's VMware pricing, what is the *migration path* and what is the *real cost* to the customer over five years. We will spend the rest of this curriculum answering all four.

---

## Lab Exercise: Stand Up Your First CE Cluster

> [!LAB] **Time:** ~3 hours · **Platform:** Nutanix Community Edition 2.1 (free)

You cannot teach a VMware admin Nutanix from a slide deck. Same goes for you. Build a lab. Today.

**Goal of this lab:** Get Nutanix CE running so you can touch every concept in this curriculum yourself. This is the foundation lab. Every subsequent module assumes you have a working CE environment.

**Two paths:**

**Path A, Nested on VMware Workstation or ESXi** (recommended for first-time setup)
1. Download CE 2.1 from `nutanix.com/products/community-edition` (requires free MyNutanix account)
2. Provision a VM: 4 vCPU, 32 GB RAM recommended (the documented absolute minimum is 16 GB, but CE running Prism Central is realistically a 32 GB+ workload). Expose hardware-assisted virtualization, VMXNET3 NIC, vSwitch promiscuous + MAC changes + forged transmits all enabled
3. Add disks: 1× 32GB (boot), 1× 200GB SSD-backed (hot tier), 1× 500GB (cold tier)
4. Boot from the CE ISO. Walk through the installer.
5. After install, connect to Prism Element on `https://<cluster-IP>:9440`
6. Default creds: `admin` / `nutanix/4u`, change immediately

**Path B, Bare metal** (best long-term lab, more work up front)
- Used Intel-based desktop with VT-x, ≥32 GB RAM, ≥1 SSD + 1 HDD, 1 GbE NIC
- Same install process; install directly to USB or SATA-DOM boot device

**What to do once it's up:**
1. Log into Prism Element. Click around. The dashboard is your first introduction to the Prism aesthetic.
2. Find the Cluster Details page. Note the Cluster Virtual IP, AOS version, AHV version.
3. Open the Hardware view. See "your one node" represented.
4. Open Storage > Storage Containers. Note the default container.
5. Open VM > Create VM. Don't create one yet, just look at the form. Compare it mentally to vCenter's New-VM wizard.

**Bookmark this:** every module from here on assumes you can reproduce its concept in your CE lab. If you skip the lab, the rest of this curriculum is entertainment, not training.

---

## Practice Questions

Ten questions. Six are NCA-style knowledge or NCP-MCI-style scenario MCQs. Two are short-answer / open-ended in the style of NCX-MCI design defense. Read each, answer in your head, then read the explanation.

---

**Q1.** Which of the following is the defining architectural characteristic of hyperconverged infrastructure?
**Cert relevance:** NCA

A) A single-vendor stack with prepackaged compute, storage, and network components
B) A distributed storage layer running as software on the same nodes that run compute workloads
C) A composable architecture where logical servers are assembled from disaggregated resource pools
D) A cloud-native deployment model that runs entirely on public cloud infrastructure

**Answer:** B

**Why this answer:** HCI is fundamentally defined by collapsing the storage array into software running on the compute nodes. There is no separate array. The same physical servers host both VMs and the distributed storage layer.

**Why not the others:**
- A) Describes Converged Infrastructure (CI), like VxBlock or FlexPod. CI is prepackaged three-tier, the storage array is still separate.
- C) Describes composable infrastructure (HPE Synergy). Composable disaggregates resources at the rack level rather than fusing them at the node level.
- D) Describes public cloud / SaaS, not HCI. HCI is on-premises (or in a colo); it is a deployment architecture, not a sourcing model.

**The trap in this question:** A and C both contain words that *sound* HCI-adjacent (single-vendor, prepackaged, composable). Test-writers count on candidates conflating "single-vendor stack" with "hyperconverged." They are different concepts.

---

**Q2.** A customer is evaluating HCI for an environment with a single workload: a 2 PB media archive accessed sequentially with very low compute requirements (less than 50 vCPUs total). Which is the most accurate response?
**Cert relevance:** NCP-MCI · sales-relevant

A) HCI is ideal for this workload because all storage is local and performance will be excellent
B) HCI may be a poor fit because the workload is heavily storage-imbalanced, requiring far more storage capacity than compute, and node sizing will be uneconomic
C) HCI is required for this workload because it is the only way to scale beyond a petabyte
D) HCI will not work because it has a hard storage capacity limit per cluster

**Answer:** B

**Why this answer:** When a workload is wildly imbalanced toward storage relative to compute, HCI forces you into uneconomic configurations. You buy CPU you don't need to get the disks. Either oversized nodes or specialized storage-heavy SKUs that erode the operational simplicity HCI was supposed to deliver. A purpose-built object or scale-out file system (or partner storage) is often the better answer.

**Why not the others:**
- A) "Local storage" doesn't matter for sequential bulk reads on cold data. Wrong reasoning.
- C) HCI is not the *only* way to scale to petabyte; dedicated scale-out object storage absolutely scales beyond a PB.
- D) Nutanix does not have a hard PB ceiling; this is factually wrong.

**The trap:** A and C are both Nutanix-positive but technically wrong. The exam (and the customer) reward you for naming the limitation honestly.

---

**Q3.** Which statement best describes the relationship between scale-up and scale-out in storage architecture?
**Cert relevance:** NCA

A) Scale-up adds nodes; scale-out adds disk shelves
B) Scale-up adds disks or replaces controllers; scale-out adds nodes
C) Scale-up and scale-out are interchangeable terms
D) Scale-up applies only to virtualization; scale-out applies only to storage

**Answer:** B

**Why this answer:** Scale-up is the traditional pattern: bigger boxes, more disks behind the same controller, eventually replacing the controller. Scale-out adds full nodes (compute + storage together), grows horizontally, has no central controller to outgrow.

**Why not the others:**
- A) Inverted. The exam loves to swap these definitions to test whether the candidate truly internalized them.
- C) They are explicitly different growth patterns.
- D) Both terms apply to many infrastructure domains, not exclusively to one.

**The trap:** A. It's so easy to invert the definitions if you haven't really thought about which is which. Anchor: "scale-up = up the same controller, scale-out = out across more nodes."

---

**Q4.** A VMware administrator at a customer site says: "We tried HCI five years ago and the performance was bad. Why would we look again now?" What is the most effective SA response?
**Cert relevance:** sales-relevant (no cert weight, but this is the SA-chair question)

A) "Performance has improved a lot in five years"
B) "What workload, what generation of HCI hardware, and what specific performance metric? The answer is probably yes, things changed. But before I make claims, I want to know what you saw."
C) "All HCI vendors are basically the same now"
D) "Our solution is the fastest HCI on the market"

**Answer:** B

**Why this answer:** The first move on objection like this is *qualification*, not *defense*. You need to know what they saw, was it write latency on an OLTP database? Was it a small cluster underprovisioned for the workload? Was it an early-generation hybrid SSD/HDD platform? Each answer leads to a different (and credible) response. Defending HCI generically before you understand the specific objection is sales malpractice.

**Why not the others:**
- A) True but generic and unconvincing. The customer has heard this from every HCI vendor.
- C) Untrue and weakens your position. Vendors differ meaningfully.
- D) A claim like this without specifics is the kind of thing that costs you the room.

**The trap:** This is the question that separates SAs who close from SAs who don't. The instinct is to defend. The discipline is to qualify.

---

**Q5.** True or false: A Converged Infrastructure (CI) reference architecture like FlexPod is functionally equivalent to a Hyperconverged Infrastructure (HCI) deployment.
**Cert relevance:** NCA

True / False

**Answer:** False

**Why this answer:** CI is prepackaged three-tier. The storage array is still a separate device with its own controllers, separate fabric, separate management plane. HCI eliminates the separate array; storage software runs on the same nodes as compute. They share marketing language ("converged") but the architecture is fundamentally different.

**The trap:** CI marketing emphasizes "single point of support" and "unified solution," which sounds a lot like HCI talking points. Don't be fooled by surface-level vocabulary; look at the architecture.

---

**Q6.** Which of the following workloads is *least* suited to deployment on HCI?
**Cert relevance:** NCP-MCI · sales-relevant

A) A 50-VM general-purpose virtualization environment
B) A VDI deployment with 800 desktops
C) A 1.5 PB cold backup archive with no production compute requirements
D) A mixed enterprise environment with database, file, and web tiers

**Answer:** C

**Why this answer:** Same reasoning as Q2 in different clothing. Backup archives at PB scale with no compute need are the canonical HCI mismatch. The compute-to-storage ratio of an HCI node makes this uneconomic. Dedicated object storage, deduplication appliances, or tape are better tools.

**Why not the others:**
- A) General-purpose virtualization is exactly where HCI shines.
- B) VDI is one of the strongest HCI workloads (predictable boot storms, linked clones benefit from data locality).
- D) Mixed enterprise is the bread-and-butter HCI use case.

**The trap:** The answer requires you to remember that HCI's economics break when storage and compute requirements are wildly mismatched. Easy to forget under exam pressure.

---

**Q7.** What is the primary operational advantage of HCI over three-tier?
**Cert relevance:** NCA · sales-relevant

A) Lower hardware costs in all cases
B) Higher raw IOPS in all cases
C) Reduction in vendor count, refresh cycles, and management surface area
D) Elimination of the need for backup

**Answer:** C

**Why this answer:** The *primary* operational advantage is consolidation: one platform, one vendor relationship, one upgrade procedure, one management plane. This is the durable benefit. Cost and performance comparisons depend on workload and configuration.

**Why not the others:**
- A) HCI is sometimes cheaper, sometimes not. Depends on workload.
- B) Three-tier with a high-end array can absolutely beat HCI on raw IOPS for specific workloads.
- D) HCI does not eliminate the need for backup. (If a vendor tells a customer this, run.)

**The trap:** A and B sound like Nutanix sales talking points. They're not always true. Customers detect this kind of overclaim and trust suffers. The answer is the boring, durable one: operational simplification.

---

**Q8.** A discovery call reveals: customer has 12 ESXi hosts, a 200 TB Pure FlashArray, two storage admins who are "not going anywhere," and a Broadcom renewal in 14 months. Which is your strongest opening qualification question?
**Cert relevance:** sales-relevant (NCX-MCI design context)

A) "When can we schedule a Nutanix POC?"
B) "What is your storage admin team going to do if we collapse storage into the compute layer?"
C) "What workloads are running on the FlashArray today, and what is the operational pain point that has you looking at us?"
D) "How much do you spend on VMware licensing per year?"

**Answer:** C

**Why this answer:** Discovery before pitch. You need to know what's running, what's hurting, what they want different. The renewal is the *trigger*, not the *requirement*. The renewal tells you they're shopping; it does not tell you what they need. Workload + pain point reveals fit.

**Why not the others:**
- A) Asking for a POC before discovery is putting the cart before the horse. You will run a POC against the wrong workloads.
- B) A real question, but second. You ask it after you understand the workload, and the org-political dimension matters but isn't the lead.
- D) Money matters, but it's a closing-stage question, not a qualifying question. Asking it first reads as transactional.

**The trap:** A and D feel productive but they short-circuit discovery. The discipline is to slow down at the start and ask the workload + pain question first. Every BlueAlly deal that closes well got there because the SA did C before A.

---

**Q9.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / design discussion. Write your reasoning. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A customer environment: 8 ESXi hosts running roughly 200 VMs (mixed workloads, no Tier-0 OLTP), one 150 TB Pure FlashArray (4 years old, support renewal due in 9 months), two storage admins (one is the customer's most senior infra person and is openly hostile to the idea of "collapsing" storage), Broadcom renewal in 14 months. The customer has asked you to recommend whether they should move to Nutanix HCI, stay three-tier with a SAN refresh, or go hybrid.

**The challenge:**
Walk through your recommendation and your reasoning. Acknowledge the political dimension. Identify what you still need to know.

**A strong answer covers:**
- **The technology answer is clear:** the workload profile (no Tier-0, mixed general-purpose, ~200 VMs, 150 TB usable) is the bread-and-butter HCI use case. Eight nodes of modern Nutanix would handle this with margin.
- **The political answer is harder:** if the senior storage admin is openly hostile and is also the customer's most senior infrastructure person, a forced HCI migration is going to fail regardless of the technology being right. That is a real risk to name.
- **The honest recommendation is staged:** start with a contained Nutanix workload (e.g., a refresh of one workload tier) running alongside the existing FlashArray, prove the operational story over 6-12 months, then decide whether to converge or stay hybrid. This buys the storage admin a path to engage with the new platform on his terms rather than feeling displaced.
- **Name the renewals as the forcing function:** the Pure renewal in 9 months and the Broadcom renewal in 14 months together make the next 12 months the decision window. The cost of doing nothing is committing to another 3-5 years of three-tier plus VMware on Broadcom pricing.
- **What you still need to know:** the storage admin's actual concerns (skill obsolescence? control? real technical objections?), the workload growth forecast, the DR and backup architecture, the customer's appetite for hypervisor change (AHV) vs staying on ESXi.

**A weak answer misses:**
- Recommending "rip and replace" without acknowledging the political risk.
- Recommending "keep three-tier" without engaging with the renewal cost arithmetic.
- Treating the storage admin as an obstacle to overcome rather than a stakeholder to engage.
- Not staging the migration to de-risk the political dimension.

**Why this question matters for NCX:** NCX-MCI design defenses test architectural reasoning under real-world constraints. "The technology is right but the org isn't ready" is one of the most common situations and one of the things the panel will look for you to recognize and address. The right answer isn't always the most technically aggressive one.

---

**Q10.** *(NCX-style architectural defense, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / architectural defense. Write your response.

**The scenario:**
You are in front of a customer's lead infrastructure architect. He says:

> *"My read on HCI: you're consolidating fault domains. With three-tier, my array can fail and my compute keeps running; my compute can fail and my data is intact. With HCI, every node is a piece of both. A bad node is now a compute incident AND a storage incident at the same time. That sounds like worse availability to me, not better."*

**The challenge:**
Respond. He is technical, well-prepared, and his framing of the issue is fair. Address it without dismissing it.

**A strong answer covers:**
- **Acknowledge the framing is correct:** in HCI, compute and storage do share fate at the node level. He is not wrong about the architectural shift.
- **Reframe what "fault domain" actually means in practice:** in three-tier, the array is itself a fault domain you depend on. When the array has a controller failure, a path failure, a fabric issue, your entire compute layer can be affected. The fault domains are not as cleanly separated as the architecture diagram suggests.
- **Compare actual failure modes:** with HCI plus RF2, the cluster tolerates a single-node loss with automatic data re-replication and HA-driven VM restart. With three-tier and a controller failure, you may face I/O pause across the entire cluster while controllers fail over (or, in older arrays, you may face an outage). Both architectures handle their respective failure modes; the question is which mode is more common and which has greater blast radius.
- **Address the "same node is now compute + storage" concern directly:** that is true, but it also means the cluster's *self-healing is comprehensive*. Lose a node, AOS rebuilds the data on remaining nodes; AHV (or vSphere HA) restarts the VMs. The recovery is automatic. With three-tier, a node loss recovers compute, but a storage failure requires array-side recovery on a different timeline.
- **Name the workloads where his concern is most valid:** for a workload with single-VM resilience requirements (no application-level clustering, no second copy elsewhere), node failure on HCI is a brief outage. For workloads with application-level resilience (clustered databases, stateless app tiers), HCI's failure model is operationally identical to three-tier.
- **Close with the honest answer:** *"You're right that fault domains are different. They are not necessarily worse; they are differently shaped. RF3 plus availability zones gives you separate failure domains within the cluster. The right test is not architectural symmetry; it is failure mode probability times blast radius for your actual workload mix. Let us walk through your top five workloads and discuss their resilience model."*

**A weak answer misses:**
- Dismissing the architect's concern as outdated thinking.
- Defending HCI without conceding that the fault model is different.
- Skipping the comparison to three-tier failure modes (controller failures, fabric failures, dual-path issues).
- Not offering the workload-level analysis that turns the conversation into a productive design exercise.
- Treating "but RF2 protects you" as a complete answer. (It is part of the answer, not all of it.)

**Why this question matters for NCX:** This is a customer-architect challenge that comes up frequently in HCI evaluations. The skill being tested is your ability to engage with a fair technical critique, acknowledge the real tradeoff, and reframe the discussion productively. NCX panels are looking for this exact disposition.

---

## What You Now Have

You can now define HCI without saying "hyperconverged" twice and calling it done. You know it is fundamentally a software architecture, not a hardware product. You know it eliminates the separate storage array by distributing storage across the same nodes that run compute. You know it scales out, not up, and you know what that implies for procurement and capacity planning.

You can now name five scenarios where HCI is the wrong answer. That alone puts you ahead of most people who pitch it. It also gives you a sales posture customers will trust.

You have the fence: HCI is not CI, not just SDS, not composable, not cloud, not all-in-one. When the next customer's incumbent vendor uses one of those words interchangeably, you will catch it.

You have the SA-chair language for the most common HCI objection ("we tried it five years ago") and for the most common opening positioning question.

You have ten practice questions worth of HCI-category discrimination. Eight test recognition and scenario judgment in NCA and NCP-MCI style. Two are open-ended NCX-MCI-style design defenses, covering staged-migration recommendation under organizational constraint and architectural defense against a fault-domain critique. The first eight earn you points on NCA. The last two are practice for the kind of architectural reasoning the NCX-MCI panel actually evaluates, and for the kind of conversation that wins or loses customer trust in front of a sharp infrastructure architect.

You are now ready to look at a specific HCI implementation, Nutanix, and see what choices it made. That is Module 2.

---

## References

Authoritative sources verified during the technical review pass on this module. Use these to validate the version-sensitive claims (AOS, CE, Prism, NC2) before quoting them in front of customers.

- [Nutanix Community Edition v2.1 — Recommended Hardware](https://portal.nutanix.com/docs/Nutanix-Community-Edition-Getting-Started-v2_1:top-sysreqs-ce-r.html). Boot disk, hot tier, and RAM minimums for CE; the source for the lab specs in this module.
- [Nutanix AOS End-of-Life Calendar (endoflife.date)](https://endoflife.date/nutanix-aos). Current AOS release tracking. AOS 7.5 is the current major as of April 2026.
- [AOS 7.5.1.1 Support Window (eosl.date)](https://eosl.date/eol/product/nutanix-aos/). Confirms the April 2026 GA release and the December 2027 end-of-support date.
- [v4 API General Availability Announcement (Nutanix blog)](https://www.nutanix.com/blog/announcing-the-v4-api-and-sdk-general-availability-in-pc-2024-3-aos-7-0). v4 REST GA shipped with Prism Central 2024.3 / AOS 7.0; backs the "Nutanix v4 REST" reference in this module.
- [Nutanix Cloud Clusters (NC2) on AWS](https://www.nutanix.com/products/nutanix-cloud-clusters/aws). Official product page confirming the NC2 product name.
- [Prism Web Console Login Documentation](https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_7:wc-login-wc-t.html). Default `admin` / `nutanix/4u` first-login credentials, port 9440, and the forced password change on first login.

---

## Cross-References

- **Next:** [Module 2: The Nutanix Stack, Node, CVM, Cluster](./02-nutanix-architecture.md)
- **Glossary:** [HCI](./appendix-a-glossary.md#hci) · [Three-Tier](./appendix-a-glossary.md#three-tier) · [Scale-Out](./appendix-a-glossary.md#scale-out) · [Converged Infrastructure](./appendix-a-glossary.md#converged-infrastructure)
- **Comparison Matrix:** [Architecture Row](./appendix-b-comparison-matrix.md#architecture)
- **Objections:** [#1 "We tried HCI before"](./appendix-d-objections.md#obj-001) · [#7 "Why not just stay three-tier?"](./appendix-d-objections.md#obj-007) · [#14 "HCI can't handle our Tier-0"](./appendix-d-objections.md#obj-014)
- **Discovery Questions:** [Q-ARCH-01 Workload landscape](./appendix-e-discovery-questions.md#q-arch-01) · [Q-ARCH-02 Org structure](./appendix-e-discovery-questions.md#q-arch-02) · [Q-ARCH-03 Renewal trigger](./appendix-e-discovery-questions.md#q-arch-03)
- **Reference Architectures:** [Small enterprise (3-8 nodes)](./appendix-i-reference-architectures.md#ra-small)

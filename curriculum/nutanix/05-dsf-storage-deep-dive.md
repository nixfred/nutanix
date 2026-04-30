---
module: 05
title: DSF Deep Dive (How Storage Actually Works)
estimated_reading_time: 42 min
prerequisites:
  - Module 01 (HCI Foundations)
  - Module 02 (Nutanix Architecture)
  - Module 03 (AHV)
  - Module 04 (Prism)
  - Working CE cluster
  - Comfort with storage concepts (RAID, replication, IOPS, latency)
key_terms:
  - DSF (Distributed Storage Fabric)
  - Storage Pool
  - Storage Container
  - vDisk
  - Extent
  - Extent Group
  - OpLog
  - Extent Store
  - Content Cache
  - RF (Replication Factor)
  - EC-X (Erasure Coding)
  - ILM (Information Lifecycle Management)
  - Stargate / Curator / Cassandra
  - Reservation / Thin provisioning
diagrams:
  - storage-hierarchy
  - data-path
  - rf-vs-ec-comparison
  - node-failure-recovery
cert_coverage:
  NCA: ~15%
  NCP-MCI: ~28%
  NCM-MCI: ~30%
  NCP-US: ~20% (foundational)
sa_toolkit:
  related_objections: [obj-012, obj-013, obj-015, obj-019, obj-024]
  related_discovery: [q-stor-01, q-stor-02, q-stor-03, q-stor-04, q-stor-05]
---

# Module 5: DSF Deep Dive (How Storage Actually Works)

> **Cert coverage:** NCA (~15%) · NCP-MCI (~28%) · NCM-MCI (~30%) · NCP-US (~20% foundational)
> **SA toolkit:** Objections #12, #13, #15, #19, #24 · Discovery Q-STOR-01 through Q-STOR-05

---

## The Promise

By the end of this module you will:

1. **Trace a VM write through DSF end to end, naming every component it touches.** From the guest's `write()` call to acknowledged-on-stable-storage, you will know what happens, where it happens, and why. This is the single most-tested concept in NCP-MCI storage.
2. **Recommend RF2 vs RF3 vs erasure coding for a given workload with the math behind the choice.** Capacity overhead numbers, performance tradeoffs, failure-tolerance differences. No guessing.
3. **Pass roughly 28% of NCP-MCI and 30% of NCM-MCI.** This is the heaviest single-module weight in the curriculum because storage is the heart of the platform. NCM-MCI lab scenarios disproportionately involve storage troubleshooting.
4. **Defend the architecture against a senior storage admin who has been engineering arrays for 20 years.** That admin has real concerns: tail latency, write amplification, rebuild times, capacity efficiency, vendor-managed firmware. Each has a real answer. None requires hand-waving.
5. **Size a cluster correctly for given workload patterns.** Capacity reservation, thin provisioning behavior, OpLog sizing, Curator scan duration on large clusters. The math you need for actual customer designs.
6. **Talk about Stargate, Cassandra, and Curator** in their operational roles, not just as service names. When NCC reports "Stargate degraded on node 3" or "Curator scan stalled," you know what that means and what to look at first.

This is the technical core of the platform. The reader who internalizes this module has internalized Nutanix.

---

## Foundation: What You Already Know

You have managed shared storage. SAN or NAS, FC or iSCSI or NFS. You know the vocabulary: LUN, datastore, volume, RAID group, parity, hot spare, cache, write-back, write-through, controller failover, multipathing.

You have also been burned by it once or twice. Maybe a controller failed at 2am and the failover took 90 seconds longer than the customer's RTO allowed. Maybe a RAID rebuild ran for 18 hours during which performance was unusable. Maybe the array's compression algorithm was great in marketing but disappointing in reality. Real stuff.

Hold that experience. You are about to look at storage in a fundamentally different shape.

DSF (Distributed Storage Fabric) is the storage layer that runs across all the CVMs in a Nutanix cluster. There is no array. There is no controller in the traditional sense. There is software on every node that takes the local disks of every node, replicates data across them, deduplicates and compresses, presents one logical pool, and self-heals when nodes fail. The terminology is partly familiar (replication, compression) and partly new (OpLog, Extent Store, Curator, EC-X). The architecture, once you see it, is internally consistent.

The Foundation question for this module: how does a distributed storage layer running on the same hardware as your VMs match (and in some ways exceed) the performance and resilience of a dedicated array?

The honest short answer: by trading one set of architectural costs (extra hardware, separate controllers, FC fabric) for another (CVM resource tax, network round-trips for some operations, software complexity in distributed coordination). Whether the trade is favorable depends on the workload. For most enterprise workloads in 2026, it is. We will get to which workloads are which.

> [!FROM-THE-SA-CHAIR]
> When a senior storage admin pushes you on DSF performance ("can it really match my Pure array?"), the wrong move is to claim parity in all dimensions. The right move is precision: *"For most workloads, yes, sometimes better, sometimes within a few percent. For specific patterns (extreme-percentile tail latency on small-block random reads, sustained sequential bandwidth on a 4-node cluster), a dedicated all-flash array still wins on paper. The gap is smaller than it was three years ago, and for typical enterprise mixed workloads it is not material. For your specific workload, we run a POC and measure."* That sentence buys you the rest of the conversation.

---

## Core Content

### The Distributed Storage Fabric (DSF) Overview

DSF is the storage layer that runs across all CVMs in a Nutanix cluster. From a 30,000-foot view it does these things:

- **Pools the local disks** of every node into one logical capacity pool.
- **Replicates writes** across nodes for durability (RF2 or RF3).
- **Caches reads** in CVM RAM (the Content Cache) for performance.
- **Compresses, deduplicates, and (optionally) erasure-codes** data for efficiency.
- **Tiers data** between hot (NVMe / SSD) and cold (HDD, where present) based on access patterns.
- **Self-heals** when nodes or drives fail by re-replicating data to surviving copies.
- **Snapshots and replicates** for backup and DR (Module 7).

DSF is built around three key services from Module 2: Stargate (the data path), Cassandra (the metadata store), and Curator (the background scrubber). Pithos owns vDisk configuration. Zeus maintains cluster consensus. These five services together implement DSF.

### The Storage Hierarchy

Before tracing any I/O, you need the vocabulary. DSF uses a specific hierarchy:

1. **Storage Pool.** The aggregate of all physical disks in the cluster. Each cluster has one Storage Pool by default that includes all node-local disks. You almost never manipulate this directly.

2. **Storage Container.** A logical "datastore equivalent" carved from the Storage Pool. Containers are where you set storage-policy attributes: RF (2 or 3), compression on/off, deduplication on/off, erasure coding on/off, reservations, advertised capacity. A cluster typically has multiple containers (e.g., one for Production with RF3, one for Dev with RF2, one for a specific workload with EC enabled).

3. **vDisk.** The virtual disk attached to a VM. From the VM's perspective, a vDisk looks like a SCSI or virtio block device. Internally, a vDisk is a logical entity backed by extents.

4. **Extent.** A contiguous range of bytes within a vDisk, typically 1 MB. Extents are the unit of metadata in Cassandra. Each extent has a metadata record indicating where it is physically stored.

5. **Extent Group.** A 4 MB physical allocation unit on disk. Extent Groups contain multiple extents and are the unit of writing to the Extent Store. Multiple extents from one or more vDisks may share an Extent Group.

> [!ON-THE-EXAM] **NCA · NCP-MCI · NCM-MCI**
> The hierarchy is testable in multiple forms. Common question patterns: which level is the policy boundary (Storage Container)? What is an Extent (1 MB metadata unit)? What is an Extent Group (4 MB physical allocation unit)? Trap distractor: questions that swap "extent" and "extent group" definitions, or that imply the Storage Pool is the policy boundary (it is not; the Container is). Memorize the hierarchy in order: **Pool → Container → vDisk → Extent → Extent Group.**

> [!FAMILIAR]
> The hierarchy maps roughly onto traditional array concepts. Storage Pool is analogous to an array's aggregate or storage pool. Storage Container is analogous to a LUN or volume that carries policy attributes (block size, deduplication, compression). vDisk is analogous to a virtual disk file (`.vmdk`) backed by the volume. The mapping is not exact, but if you understand SAN concepts, the DSF hierarchy will feel familiar after a short orientation.

> [!DIFFERENT]
> Storage policy is software-defined and operates at the Container level. Changing a vSAN policy (or moving a VM between datastores in a traditional SAN) often requires a Storage vMotion. In DSF, you can change container-level policies (compression, EC) and Curator applies them in the background. The vDisk does not need to physically move. Policy is metadata; data movement is asynchronous. This is operationally simpler than the array equivalent and is one of the durable wins of software-defined storage at this layer.

---

### Diagram: The Storage Hierarchy

**id:** `storage-hierarchy`
**type:** layered
**caption:** From physical disks to VM block I/O. Each level is a different abstraction with different operational meaning.
**exam_relevance:** [NCA, NCP-MCI, NCM-MCI]
**whiteboard_ready:** true

**Elements (top to bottom):**
- Top: a row of "VM" boxes (light gray). Each has one or more vDisks attached as block devices (`/dev/sda`, `/dev/vda`, etc.)
- Below VMs: a band labeled **vDisk** (rust) showing virtual disks as logical entities. Sub-label: "Each vDisk has a unique ID; backed by extents."
- Below vDisks: a band labeled **Extent** (rust, narrower stripes). Sub-label: "1 MB metadata unit. Cassandra tracks every extent's location."
- Below extents: a band labeled **Extent Group** (rust, wider blocks). Sub-label: "4 MB physical allocation unit. Multiple extents may share one Extent Group."
- Below Extent Groups: a band labeled **Storage Container** (gold) with attributes shown: "RF=2, Compression=on, EC=off, Reservation=20 TB". Sub-label: "Policy boundary. RF, compression, dedup, EC are set here."
- Below Container: a band labeled **Storage Pool** (gold, wider). Sub-label: "All physical disks across all nodes. One pool per cluster typically."
- Bottom: row of physical node boxes (gray) showing local NVMe + SSD (and HDD on hybrid platforms).

**Connections:**
- VMs to vDisks: "Block I/O"
- vDisks to extents: "Logical mapping"
- Extents to extent groups: "Physical allocation"
- Extent groups to container: "Policy applied here"
- Container to pool: "Carved from"
- Pool to physical disks: "Aggregated across nodes"

**Annotations:**
- Beside Container: "This is where you set RF, compression, EC. Different containers get different policies."
- Beside Extent Group: "Compression and erasure coding operate at this level."
- Beside Cassandra (drawn off to the side): "Cassandra tracks every extent's physical location across the cluster."

**Why this diagram exists:** The hierarchy is one of those things that, once seen, makes everything else make sense. New BlueAlly SAs frequently confuse Storage Pool with Storage Container; the diagram fixes that. Whiteboard it when a customer asks "where do I configure compression?" The answer is: at the Container level, and the diagram shows why.

---

### The Data Path: A VM Write, End to End

This is the single most important concept in the module. Read it carefully. We will walk a VM write from `write()` syscall to acknowledged-durable.

**Step 1. The guest VM issues a write.** A Linux or Windows VM running on a Nutanix node calls into its storage stack: `write(fd, buf, len)`. The OS turns this into a SCSI or virtio block command directed at the VM's vDisk.

**Step 2. The hypervisor presents the I/O to the local CVM.** On AHV, the VM's vDisk is exposed via a paravirtualized interface that routes to the local CVM. On ESXi-on-Nutanix, the CVM presents an NFS datastore that ESXi mounts; writes traverse to the CVM via NFS. Either way: the I/O lands at the local CVM's Stargate process.

**Step 3. Stargate accepts the write.** Stargate, the data-path service, receives the write. Its first decision: is this a sequential streaming write (likely large and benefits from going straight to Extent Store) or a random write (benefits from the OpLog write buffer)? Most database, VDI, and general-purpose workloads are random; Stargate routes them to the OpLog.

**Step 4. The write goes to the OpLog (local copy).** The **OpLog** is a persistent write buffer on the local node's hot tier (NVMe or SSD). It is essentially a write-ahead log for incoming writes. Writes are appended to the OpLog and acknowledged once durable. The OpLog is where DSF gets its low-latency write characteristic.

**Step 5. The write is replicated to remote OpLog (RF copies).** Simultaneously with the local OpLog write, Stargate replicates the write to one or two other nodes' OpLogs (RF2 or RF3 respectively). The remote OpLog write must complete before the original VM write is acknowledged. This is the cost of strong consistency: every write traverses the network synchronously to at least one peer.

**Step 6. Acknowledgment to the VM.** Once the local OpLog and the required number of remote OpLog replicas have all confirmed durable write, Stargate acknowledges to the hypervisor, which acknowledges to the guest VM. The VM's `write()` call returns success.

**Step 7. OpLog drains to Extent Store (asynchronous).** The OpLog is a buffer, not the permanent home for the data. As the OpLog fills past thresholds (and during quiet periods), Stargate drains writes from the OpLog to the **Extent Store**, the persistent backing storage on the local node. The drain is sequential and efficient, which lets the underlying SSD/HDD perform optimally.

**Step 8. Compression and (optionally) deduplication apply during drain.** Inline compression compresses data as it moves from OpLog to Extent Store. If dedup is enabled on the container, fingerprints are computed during drain and dedup pointers are created where applicable.

**Step 9. Erasure Coding (if enabled) is applied later.** EC-X is a post-process operation. Curator runs periodic scans and converts qualifying data from RF replication to erasure-coded form, reducing capacity overhead. (More on EC below.)

**Step 10. Reads.** When the VM later reads the data:
- **First check: Content Cache.** Stargate maintains an in-memory cache of recently-accessed extents in CVM RAM. Cache hits return immediately.
- **Second: local hot tier (Extent Store on NVMe/SSD).** Local reads are fast.
- **Third: remote node** (over the network) if the data is not local. Curator may eventually migrate the data to be local for repeat reads.

That is the full path. Read it again. The next section (the diagram) makes it visual.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI**
> The data path is the heaviest single topic on NCP-MCI storage and on NCM-MCI labs. Memorize: **OpLog is the persistent write buffer (hot tier). Extent Store is the persistent backing storage. Content Cache is the in-memory read cache.** Trap distractor patterns: questions that imply OpLog is volatile (false; it is on durable hot-tier media), or that imply writes wait for Extent Store drain before acknowledgment (false; OpLog persistence is sufficient for ack), or that imply Content Cache is on disk (false; it is in CVM RAM).

---

### Diagram: The Data Path

**id:** `data-path`
**type:** flow
**caption:** A VM write from guest syscall to durable acknowledgment, with replication and asynchronous drain.
**exam_relevance:** [NCP-MCI, NCM-MCI]
**whiteboard_ready:** true

**Elements:**
- Top: "VM" (light gray) issuing a write
- Below VM: "Hypervisor" (muted blue), passes I/O to local CVM
- Below hypervisor: "Local CVM: Stargate" (rust), accepts the write
- Branching from Stargate, two parallel paths:
  - Down to "Local OpLog" on local NVMe/SSD (rust, hot tier)
  - Sideways to "Remote CVM Stargate" → "Remote OpLog" on a peer node (rust, hot tier)
- After both OpLog writes complete: "Acknowledgment" arrow back up to VM
- Asynchronous arrow from Local OpLog to "Extent Store" on local hot/cold tier (rust): "Drain (async, with compression and dedup)"
- Side annotations: "Content Cache (CVM RAM)" connected to read path
- Read flow shown lightly: VM → Stargate → Content Cache (hit) or Extent Store (local) or Remote CVM (network)

**Connections:**
- VM → Hypervisor → Stargate: "Write request"
- Stargate → Local OpLog: "Local persistence (NVMe)"
- Stargate → Remote CVM → Remote OpLog: "Network-replicated persistence (RF=2)"
- Remote and local OpLog ack → Stargate → Hypervisor → VM: "Write acknowledged"
- OpLog → Extent Store: "Async drain, with compression / dedup applied here"

**Annotations:**
- "Write is acknowledged after local + remote OpLog persistence. NOT after Extent Store."
- "Sequential bulk writes can bypass OpLog and go straight to Extent Store (Stargate routing decision)."
- "RF=3 adds a second remote OpLog write, increasing latency slightly."
- "Curator runs in the background. It does not appear in the synchronous write path."

**Why this diagram exists:** This is the diagram the BlueAlly SA draws when a senior storage admin asks "what is the actual write path?" Answering with words alone loses the room; drawing the path with explicit acknowledge-points wins it. Whiteboard-ready and high-priority. Memorize for NCP-MCI exams.

---

### The Cycle, Frame Two: DSF as a SAN, Distributed

Step back. In a traditional all-flash array, an incoming write hits the array's controller, lands in NVRAM (write cache, battery-backed), is acknowledged to the host, and is later destaged to flash. The controller's NVRAM is the latency-critical buffer. Replication for redundancy happens between dual controllers in the same chassis.

DSF replaces the array controller with the CVM, replaces NVRAM with the OpLog (persistent on the local node's hot tier), and replaces dual-controller redundancy with cross-node replication (RF2 or RF3). The write path looks similar in shape: incoming write, durable buffer, acknowledgment, asynchronous destage. The major architectural shift is that the durable buffer and its peer copy are on different physical nodes connected by the network.

This is why network quality matters more in DSF than in a traditional array. Within a SAN array, controller-to-NVRAM is a backplane operation. In DSF, OpLog-to-OpLog crosses the cluster network. A 25 or 100 GbE switching fabric with low jitter is not optional; it is the substrate of write performance.

### The Cycle, Frame Three: DSF as Cassandra-Plus-Stargate

Storage architects who appreciate distributed-systems internals will gravitate to this frame.

DSF is, in essence, two coordinated systems:

- **Cassandra (the metadata store).** Tracks every extent: which vDisks reference it, what extent group it lives in, where physically (which nodes, which disks). Every extent operation involves Cassandra metadata reads or writes. Cassandra is built on a fork of Apache Cassandra, optimized for the NoSQL key-value access patterns DSF requires. It runs in a ring across all CVMs.

- **Stargate (the data path).** Processes the actual I/O. Reads and writes flow through Stargate. Stargate consults Cassandra for "where does extent X live?" and operates on the data accordingly.

The architectural separation is deliberate: metadata is a small, hot, frequently-accessed problem (good fit for a distributed NoSQL store). Data is a large, less-frequently-randomly-accessed problem (good fit for distributed object storage with caching). Separating them lets each scale its own way.

This is the same pattern that powers Ceph, S3, and other distributed storage systems. The names are different but the design is canonical.

### The Cycle, Frame Four: DSF as the Value Layer

For an operations leader or CIO, the durable DSF story is not the data path. It is what DSF enables.

- **Snapshots that don't degrade performance.** Already covered in Module 3. DSF native snapshots are metadata operations.
- **Replication as a built-in primitive.** Async, NearSync, and Metro replication (Module 7) all leverage DSF's primitives. There is no separate replication appliance.
- **Compression and deduplication without an array refresh.** Capacity efficiency is built in. Customers used to paying separately for array-side compression often appreciate this.
- **Erasure coding as a capacity optimization.** Configurable per container.
- **Capacity portability.** vDisks can move between containers and clusters. Storage policy is a software attribute, not a hardware one.
- **Self-healing.** When a node or drive fails, DSF rebuilds without human intervention.
- **API-first storage.** Storage operations are JSON, not vendor-specific CLI.

The CIO-level pitch for DSF: *"You are not buying storage. You are buying a software-defined storage layer that grows when your compute grows, self-heals when hardware fails, and has every storage feature your team currently buys separately built in."*

---

### Replication Factor (RF) 2 and 3

RF is the per-container setting that determines how many copies of every write are stored across the cluster.

**RF2.** Two copies of every write. One local, one on a peer node. Survives any single node failure or single drive failure with no data loss.

**RF3.** Three copies of every write. One local, two on peer nodes. Survives any two simultaneous node failures (or two drive failures across nodes) with no data loss.

**Capacity overhead:**

| RF | Raw capacity multiplier | Effective usable capacity |
|---|---|---|
| RF2 | 2x | 50% of raw (before compression/dedup/EC) |
| RF3 | 3x | 33% of raw (before compression/dedup/EC) |

Capacity reservation (the cluster reserves headroom for self-healing) further reduces effective usable. A typical "rule of thumb" planning number is 50% effective for RF2 and 33% for RF3, with another 10-15% set aside for reservation. Compression and EC recover meaningful capacity back; we cover those below.

**Performance implications:**

- RF2 has slightly lower write latency than RF3 (one fewer network round-trip per write).
- RF3 has more network traffic during normal writes and during rebuilds.
- For most workloads the latency difference is small (single-digit percent on modern 25/100 GbE networks).

**When to choose which:**

| Workload pattern | Typical recommendation |
|---|---|
| General-purpose VMs, dev/test, file/print | RF2 |
| Tier-1 production with strong backup story | RF2 (with backup) |
| Tier-1 production without external backup | RF3 |
| VDI | RF2 (boot storms benefit from less write amplification) |
| Mission-critical databases without application HA | RF3 |
| Mission-critical databases with application HA (Always On, replica sets) | RF2 (application provides additional redundancy) |
| Compliance-driven workloads requiring multiple data copies | RF3 |

> [!FROM-THE-SA-CHAIR]
> The RF2 vs RF3 question comes up in nearly every customer design conversation. The wrong move is to default to RF3 ("more is better"). The right move is to ask: *"What is your RTO and RPO? What is your backup strategy? Are your applications doing redundancy at the application layer? How much capacity headroom can you afford?"* RF3 costs you 33% more raw capacity than RF2. On a 200 TB cluster that is 66 TB of raw capacity. That is real money. Recommend RF3 only when the customer's resilience requirements actually need it.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI**
> RF behavior under failure is heavily tested. Memorize: RF2 survives any single failure; RF3 survives any two simultaneous failures. After a failure, the cluster automatically re-replicates (Curator-driven) to restore the configured RF. Trap distractor: questions implying RF2 can survive two simultaneous node failures (false), or implying RF3 has no capacity overhead vs RF2 (false; RF3 stores 50% more data).

---

### Erasure Coding (EC-X): The Capacity Optimization

**Erasure coding** is an alternative to replication for redundancy. Instead of storing two or three full copies, EC stores data plus parity in a way that survives node failures with substantially less capacity overhead.

DSF's implementation is called **EC-X.** It is configurable per Storage Container.

**How EC-X works (simplified):** Take a stripe of N data blocks plus K parity blocks across N+K nodes. Any K nodes can fail and the data is still recoverable from the remaining nodes. The capacity overhead is K/(N+K).

**Common EC configurations on DSF:**

| Configuration | Stripe (data + parity) | Capacity overhead | Failure tolerance | Minimum cluster size |
|---|---|---|---|---|
| EC equivalent of RF2 | 4+1 (4 data, 1 parity) | 25% (vs RF2's 100%) | 1 node | 5 nodes |
| EC equivalent of RF3 | 4+2 (4 data, 2 parity) | 50% (vs RF3's 200%) | 2 nodes | 7 nodes |

The capacity savings are substantial. Going from RF2 to EC 4+1 reduces capacity overhead from 100% (50% effective) to 25% (75% effective). On a 200 TB raw cluster, that is the difference between 100 TB usable and 150 TB usable: a 50% increase in effective capacity, with the same failure tolerance.

**The catch:** EC has performance tradeoffs.

1. **Write amplification on small writes.** Writing a small block requires reading the existing stripe, modifying it, and rewriting parity. This is expensive for random write workloads.
2. **Curator overhead during conversion.** EC is applied as a post-process operation by Curator. New writes start as RF2 or RF3 and are converted to EC later when data has cooled. The conversion consumes background CPU and network.
3. **Recovery is more expensive.** Rebuilding from EC parity requires reading from multiple surviving nodes and recomputing. Single-replica rebuilds (from RF) are simpler.

**When to enable EC-X:**

| Workload type | EC recommendation |
|---|---|
| Cold archives, file servers, infrequently-modified data | EC strongly recommended (capacity wins outweigh modest performance cost) |
| Backup repositories on Nutanix | EC recommended |
| VDI persistent profiles, general-purpose VMs | EC often beneficial, test in your environment |
| OLTP databases, latency-sensitive workloads | EC not recommended (performance cost) |
| Workloads with high write churn and tight latency budgets | EC not recommended |

EC is a powerful capacity tool for the right workloads. Misapplied to write-heavy databases, it hurts. Know your workload before recommending it.

> [!ON-THE-EXAM] **NCP-MCI · NCP-US**
> EC topics: configuration (per Storage Container), minimum cluster size requirements, capacity-overhead calculations, performance tradeoffs. Memorize the math: EC 4+1 has 25% overhead and tolerates 1 failure (RF2-equivalent). EC 4+2 has 50% overhead and tolerates 2 failures (RF3-equivalent). Trap distractors: questions implying EC has zero performance impact (false), or that EC requires the same minimum cluster size as RF (false; EC requires more nodes).

---

### Diagram: RF vs EC Capacity and Failure Domains

**id:** `rf-vs-ec-comparison`
**type:** comparison
**caption:** Side-by-side: RF2, RF3, EC 4+1, EC 4+2. Capacity overhead and failure tolerance compared.
**exam_relevance:** [NCP-MCI, NCM-MCI, NCP-US]
**whiteboard_ready:** true

**Elements:**
- Four panels arranged in a 2x2 grid:

**Panel 1: RF2**
- 6 nodes shown as boxes
- A 1 GB block of data shown stored on Node 1 (local) and Node 2 (replica)
- Capacity used: 2 GB for 1 GB of data
- Overhead label: "100% overhead (50% effective)"
- Failure tolerance label: "1 node loss"

**Panel 2: RF3**
- 6 nodes shown as boxes
- Same 1 GB block stored on Nodes 1, 2, and 3
- Capacity used: 3 GB for 1 GB of data
- Overhead label: "200% overhead (33% effective)"
- Failure tolerance label: "2 simultaneous node losses"

**Panel 3: EC 4+1**
- 6 nodes shown
- 1 GB data striped across 4 nodes (250 MB each) with 1 parity block on the 5th node
- Capacity used: 1.25 GB for 1 GB of data
- Overhead label: "25% overhead (80% effective)"
- Failure tolerance label: "1 node loss"
- Note: "Min cluster: 5 nodes"

**Panel 4: EC 4+2**
- 8 nodes shown
- 1 GB data striped across 4 nodes (250 MB each) with 2 parity blocks on 2 other nodes
- Capacity used: 1.5 GB for 1 GB of data
- Overhead label: "50% overhead (67% effective)"
- Failure tolerance label: "2 simultaneous node losses"
- Note: "Min cluster: 7 nodes"

**Annotations:**
- Below grid: "EC trades CPU and network during writes for substantial capacity savings."
- "Failure tolerance is comparable to the RF equivalent. Capacity efficiency is dramatically better."
- "Workload sensitivity to small random writes determines whether EC is the right choice."

**Why this diagram exists:** The capacity and failure-tolerance comparison is one of the most-requested visualizations from customers evaluating Nutanix vs other HCI platforms or arrays. Whiteboard-ready. Use it whenever the conversation turns to "what does this actually cost in raw capacity?"

---

### Compression

DSF supports compression at the Storage Container level. It happens during the OpLog drain to Extent Store (so writes are acknowledged before compression runs; the compression cost is in background drain throughput, not in write latency).

**Two flavors:**

- **Inline compression.** Applied during drain. The default in modern AOS for most workloads.
- **Post-process compression.** Applied later by Curator on data that wasn't compressed during drain. Older AOS versions defaulted to this; current AOS uses inline as default.

**Compression algorithm.** DSF uses LZ4 by default (fast, modest ratios) with optional zstd for higher ratios at higher CPU cost. The actual ratio depends on data type: text, logs, and OS images compress 2-3x; already-compressed data (encrypted, JPEG, video) compresses minimally.

**Real-world ratios:** Mixed enterprise workloads typically achieve 1.5x to 2.5x compression ratios on DSF. Marketing numbers of 4-6x reflect best-case scenarios. Plan capacity using 2x for general workloads, 1.2x for already-compressed data, and let actual performance guide adjustment.

### Deduplication

DSF supports deduplication at two levels:

- **Cache deduplication.** In-CVM-RAM deduplication for hot data. Always on for compatible workloads. Helps with Content Cache efficiency.
- **On-disk deduplication.** Configured per Storage Container. Computes fingerprints during drain, deduplicates blocks across the container.

**When dedup is valuable:**

- VDI workloads with persistent profiles (multiple VMs, similar OS).
- Server VMs with similar operating systems.
- Test/dev environments with cloned VMs.

**When dedup is expensive without benefit:**

- Already-deduplicated data sources.
- Small clusters with low data uniqueness.
- Workloads with high write churn (dedup tracking adds overhead).

**Sizing implication:** dedup is metadata-heavy. Enabling on-disk dedup increases the CVM's metadata footprint, which means larger CVMs (more RAM) on dedup-enabled clusters. Size accordingly.

> [!FROM-THE-SA-CHAIR]
> Compression and dedup are common customer-facing topics. The discipline: never quote a specific data-reduction ratio without seeing the customer's data. Use ranges (1.5-2.5x for general workloads). Recommend dedup only for VDI or workloads with high data similarity. Misapplied dedup costs CVM resources without commensurate benefit. The customer's storage admin will respect honesty here more than aggressive marketing numbers.

---

### Tiering and Information Lifecycle Management (ILM)

On hybrid platforms (NVMe/SSD + HDD), DSF tiers data automatically. Hot data lives on the NVMe/SSD tier; cold data drifts to the HDD tier. **Curator** drives this via ILM (Information Lifecycle Management) scans.

In 2026, all-flash (or all-NVMe) Nutanix deployments dominate, and tiering is less operationally relevant for most customers. On hybrid platforms (which still ship for cost-optimized capacity tiers like backup or large file workloads), tiering is significant.

**ILM operations include:**

- Promoting hot data to the fastest tier.
- Demoting cold data to capacity tiers.
- Migrating data for locality after VM moves (already mentioned in Module 2).
- Rebalancing data when nodes are added or removed.
- Converting data from RF to EC (the EC conversion process).

ILM runs continuously in the background. It does not appear in the synchronous I/O path.

### Curator: The Background Worker

**Curator** is the background scrubbing and rebalancing service. It runs on every CVM. One Curator is master; others are followers. Curator does:

1. **Periodic scans** (full and partial). Full scans every 6-24 hours; partial scans more frequently. Scans walk metadata to identify operations to perform.
2. **Re-replication after failures.** When a node or drive is lost, Curator orchestrates re-replication of the affected data to restore configured RF or EC.
3. **EC conversion.** Converts qualifying data from RF to EC.
4. **Compression and dedup post-process** (in some configurations).
5. **Capacity reclamation.** Garbage collection of deleted vDisk space.
6. **Tiering / ILM operations.**
7. **Rebalancing when nodes are added or capacity is uneven.**

Curator is a workhorse. On a healthy cluster you barely notice it. When something is wrong (a stalled scan, a long-running re-replication, a backlog of pending operations), Curator status is one of the first things you check. The CLI command `curator_cli` and the NCC checks expose Curator state.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI**
> Curator's role is heavily tested, especially on NCM-MCI lab scenarios. Memorize: Curator is the background scrubber. It re-replicates data after failures. It converts RF to EC. It drives ILM and tiering. It does not handle the synchronous data path (that is Stargate). Trap distractor: questions that imply Curator is part of the synchronous write acknowledgment (false).

---

### Failure Recovery: When a Node Goes Down

Nutanix's resilience story is built around graceful failure handling. Walk through what happens when a node fails:

**At the moment of failure:**
- The cluster heartbeat (Cassandra and Zeus mechanisms) detects the node loss within seconds.
- Stargate on the failed node stops responding. Other CVMs route I/O around the failed node.
- VMs that were running on the failed node are restarted by Acropolis (AHV) or vSphere HA (ESXi-on-Nutanix). Module 3 covered this.
- Cassandra removes the failed node from the active ring. Operations continue with the surviving nodes.

**Curator's response:**
- Curator detects the missing replica copies (data that was on the failed node now has fewer than RF copies remaining).
- Curator orchestrates re-replication: reads surviving copies, writes to other nodes, restores configured RF.
- The duration depends on cluster size, data volume, and network bandwidth. For a 4-node cluster with 50 TB of unique data on the failed node, re-replication takes hours. For a 20-node cluster with the same data spread across more nodes, it takes much less.
- During re-replication, the cluster runs at reduced redundancy. A second failure during this window can result in data loss for RF2 clusters.

**For RF3 and EC clusters:**
- The cluster tolerates the failure plus one more (RF3) or one more on a subset of nodes (EC 4+2).
- Re-replication still occurs to restore configured redundancy.

**Drive failures:**
- Same general flow but at the drive level. Curator re-replicates data that was on the failed drive.
- Modern Nutanix nodes have multiple drives per node; a single drive failure does not lose the node's data, just the affected drive's data, which is replicated elsewhere.

> [!FROM-THE-SA-CHAIR]
> Customers ask "how long does rebuild take?" The honest answer depends on cluster size and data volume. The pattern: bigger clusters rebuild faster (more nodes share the work). For a typical 8-12 node cluster carrying 100-200 TB, expect rebuild times in the range of 2-8 hours after a node loss. Always frame this with "for your specific cluster size and data volume, we model it during the design phase." Don't quote a single number; the dependency is real.

---

### Diagram: Node Failure Recovery (Curator Scan)

**id:** `node-failure-recovery`
**type:** flow
**caption:** A node fails. Stargate routes around it; VMs restart on survivors; Curator re-replicates lost copies. The cluster self-heals.
**exam_relevance:** [NCP-MCI, NCM-MCI]
**whiteboard_ready:** false

**Elements:**
- A 5-node cluster shown across the top, with Node 3 grayed out and labeled "Failed"
- Below the failed node, the data it held: a set of "extents" with replicas marked as "Remaining: 1 of 2" (indicating RF=2 with one copy lost)
- Surviving nodes 1, 2, 4, 5 shown carrying their existing data
- Acropolis box (rust) shown invoking VM restarts on surviving hosts
- Curator box (rust) shown driving re-replication: reads surviving copies, writes new replicas to other surviving nodes
- Time axis below the diagram showing phases:
  - T+0: Node fails
  - T+5-30s: Heartbeat detects failure
  - T+30-90s: Acropolis restarts affected VMs
  - T+1-15m: Curator scan identifies missing replicas
  - T+15m to T+hours: Re-replication restores RF

**Connections:**
- Surviving nodes to "Curator master" (drawn on Node 1, e.g.): "Curator coordinates work"
- Curator to surviving nodes: "Re-replicate this extent to that node"
- Acropolis to surviving nodes: "Restart VMs"

**Annotations:**
- "VMs restart automatically. No human intervention."
- "Cluster operates at reduced redundancy during re-replication. RF2 clusters are at single-failure tolerance during this window."
- "Re-replication time depends on cluster size and data volume on the failed node."
- "RF3 and EC 4+2 clusters tolerate one more failure during this window."

**Why this diagram exists:** Customers worry about what happens when something fails. This diagram shows the end-to-end self-healing path with a time axis. Use it in design conversations when the resilience story is on the table. Particularly valuable for the customer's senior storage admin who has spent decades managing array-side failure procedures and wants to know what changes.

---

### Capacity Planning: The Math Customers Need

This section is for design conversations. When a customer asks "how much usable capacity will I get?", the math is more nuanced than just dividing raw by RF. Walk through it.

**Step 1. Raw cluster capacity.** Sum of all drives across all nodes. Example: 8 nodes × 4 drives × 7.68 TB NVMe = 245.76 TB raw.

**Step 2. Apply RF or EC.**
- RF2: divide by 2 → 122.88 TB (50%).
- RF3: divide by 3 → 81.92 TB (33%).
- EC 4+1: 80% of raw → 196.6 TB.
- EC 4+2: 67% of raw → 164.7 TB.

**Step 3. Apply capacity reservation.** AOS reserves headroom for self-healing on node loss. Default reservations are roughly:
- For an N-node cluster: reserve enough capacity to absorb the data of one node (or two for RF3).
- This typically reduces effective by 10-15% on smaller clusters and less on larger clusters.

**Step 4. Apply compression and dedup gains.** These add capacity back:
- General compression: 1.5-2.5x effective gain (so divide step 3 result by 1.5-2.5).
- Dedup: variable; conservative planning is 1.0x (no gain) unless workload-specific data supports a higher number.

**Worked example:** 8-node cluster, 4 × 7.68 TB NVMe per node, RF2, compression on, no EC, no dedup, general workload.

- Raw: 245.76 TB
- After RF2: 122.88 TB
- After 12% reservation: ~108 TB
- After 2x compression: ~216 TB effective usable for general data

The customer who hears "raw 245 TB, usable 216 TB" in a marketing slide is being told a real number, but only after compression. State all the assumptions when you walk through this.

> [!ON-THE-EXAM] **NCM-MCI · NCP-MCI**
> Capacity planning math is testable. Common scenarios: given raw capacity and RF, calculate effective usable. Given an EC configuration, calculate capacity overhead. Given compression assumptions, project final usable. Memorize the math: RF2 = divide by 2, RF3 = divide by 3, EC 4+1 = 80% of raw, EC 4+2 = 67% of raw. Apply reservation. Apply compression. State the assumptions.

---

### Storage-Only Nodes (Compute-Light Configurations)

For workloads that need a lot of capacity but not much CPU (some backup repositories, some file workloads, some object storage backing), Nutanix offers **storage-heavy nodes** and **storage-only nodes**.

A storage-only node has minimal compute (just enough for the CVM and DSF responsibilities) but large storage capacity. They contribute to the cluster's storage pool without contributing meaningful compute. They allow asymmetric clusters where a few storage-only nodes provide capacity for storage-heavy workloads while compute-balanced nodes handle the active VMs.

**When to use storage-only nodes:**
- Backup repositories on Nutanix (especially with Mine).
- Some Files / Objects workloads where the storage-to-compute ratio is high.
- Clusters that have outgrown their compute-balanced node sizing on the storage axis.

**Tradeoffs:**
- Erodes some of HCI's operational simplicity (you now have asymmetric node configurations).
- Still simpler than maintaining a separate dedicated storage tier.

This is a real workaround for the Module 1 honest gap (HCI's economics break for storage-imbalanced workloads). Storage-only nodes do not eliminate the issue; they soften it.

---

## Lab Exercise: Storage Container Manipulation and Failure Simulation

> [!LAB] **Time:** ~3 hours · **Platform:** Your 3-node CE cluster from Module 02

This lab gives you hands-on with the storage hierarchy, container policies, and a controlled failure simulation. You will create different containers, observe their behavior, and run an actual failure scenario.

**Steps:**

1. **Inventory the existing storage hierarchy.** SSH into a CVM and run:
   ```
   ncli storage-pool list
   ncli container list
   ncli vdisk list
   ```
   Note the default Storage Pool and the default container created at cluster install.

2. **Create a new Storage Container with non-default policies.** From Prism Element, navigate to Storage > Storage Container > Create Container:
   - Name: `lab-rf2-compressed`
   - Replication Factor: 2
   - Compression: enabled (default inline)
   - Erasure Coding: disabled (need 5+ nodes for EC)
   - Deduplication: cache only (on-disk dedup needs more nodes typically)
   - Reservation: leave unreserved
   - Save.

3. **Create a second container** with different settings:
   - Name: `lab-rf2-no-features`
   - All advanced features off.

4. **Provision a VM into each container.** Use Prism's Create VM, select the appropriate container for the VM's vDisk. Power them on with a small Linux OS.

5. **Generate I/O on each VM** to populate data. SSH in and run something like `dd if=/dev/urandom of=/tmp/test.dat bs=1M count=2048` to write 2 GB of random data, then `dd if=/dev/zero of=/tmp/zero.dat bs=1M count=2048` to write 2 GB of zeros (zeros compress to nearly nothing).

6. **Observe storage usage in Prism.** Compare the two containers. The `lab-rf2-compressed` container should show meaningfully less physical space used for the same logical data, especially the zeros file.

7. **Examine the data path components.** From a CVM, run:
   ```
   nodetool -h localhost ring     # Cassandra ring status
   curator_cli get_curator_state  # Curator status
   stargate_status                # Stargate health
   ```

8. **Look at vDisk metadata.** Find one of your VM's vDisks:
   ```
   vdisk_config_printer            # Lists vDisks and their config
   vdisk_usage_printer -vdisk_id <id>  # Usage statistics
   ```

9. **Failure simulation (CAREFUL: lab cluster only).** Power off one of the three CVMs (not all of them):
   - From Prism: select a node, put it in maintenance mode, then shut down the CVM via SSH (`sudo shutdown -h now` from the CVM).
   - Observe Prism's reaction: alerts fire, Data Resiliency status drops, VMs running on the failed node are auto-restarted on survivors.
   - Watch Curator begin re-replication. (This will be brief on a small lab cluster.)
   - Power the CVM back on. Bring it out of maintenance mode. Watch the cluster restore full redundancy.

10. **Run NCC after recovery.** `ncc health_checks run_all`. Verify the cluster reports healthy. Read through the output to see the categories of checks Curator and the storage layer expose.

**What this teaches you:**
- The storage hierarchy in operational practice: Pool, Container, vDisk.
- How container policies (compression, RF) affect actual capacity.
- The CLI surface for storage diagnostics.
- A real failure-and-recovery cycle.
- NCC's storage-related checks, which are exam-relevant for NCM-MCI.

**Customer-demo angle:** The container creation step and the failure simulation are both customer-demo flows. Show a customer the policy options on a container; show a customer a controlled failure and the auto-recovery. Both are powerful demonstrations.

---

## Practice Questions

Twelve questions. Six knowledge MCQ, four scenario MCQ, two NCX-style design questions. Read each, answer in your head, then read the explanation.

---

**Q1.** What is the OpLog?
**Cert relevance:** NCP-MCI · NCM-MCI

A) An in-memory cache that stores recently-read data
B) A persistent write buffer on the local node's hot tier (NVMe/SSD), used to acknowledge writes durably before they are drained to the Extent Store
C) A log file generated by the Acropolis service for VM lifecycle events
D) A backup of vDisk metadata stored in Cassandra

**Answer:** B

**Why this answer:** OpLog is the persistent, hot-tier write buffer. Writes land in OpLog (locally and on a peer node for RF2/RF3), are acknowledged once durable, and are later drained to the Extent Store. OpLog is the source of DSF's low-latency write characteristic.

**Why not the others:**
- A) That describes the Content Cache (in CVM RAM), not OpLog.
- C) Acropolis logs are unrelated; OpLog is a storage construct.
- D) Cassandra holds metadata, not OpLog data.

**The trap:** A is the seductive distractor: customers and learners hear "Op-Log" and think "it's a log of operations." That mental model conflates it with audit logs. OpLog is specifically the write buffer.

---

**Q2.** Which of the following correctly describes the relationship between Storage Pool, Storage Container, and vDisk in DSF?
**Cert relevance:** NCA · NCP-MCI

A) Storage Pool is carved from Storage Containers, which are carved from vDisks
B) Storage Pool is the aggregate of all physical disks; Storage Containers are logical carvings within the pool with policy attributes (RF, compression, EC); vDisks are virtual disks attached to VMs and are backed by extents within a container
C) Storage Pool, Storage Container, and vDisk are interchangeable terms for the same thing
D) vDisks are physical disks; Storage Containers are RAID groups; Storage Pools are LUNs

**Answer:** B

**Why this answer:** Pool is the physical aggregate. Container is the logical, policy-bearing layer. vDisk is the VM-facing virtual disk. The hierarchy is Pool → Container → vDisk → (extents → extent groups).

**Why not the others:**
- A) Inverts the hierarchy.
- C) They are distinct constructs with different purposes.
- D) Mixes Nutanix and traditional-array terminology incorrectly.

**The trap:** A and D both reflect partial mental models that get the hierarchy wrong. Memorize the order.

---

**Q3.** Which statement about RF (Replication Factor) is correct?
**Cert relevance:** NCA · NCP-MCI

A) RF is set at the cluster level and applies uniformly to all data
B) RF is set per Storage Container; different containers in the same cluster can have different RF settings (RF2 vs RF3)
C) RF must be the same as the cluster's node count
D) RF only applies to AHV; ESXi-on-Nutanix uses VMware's native replication

**Answer:** B

**Why this answer:** RF is a per-container setting. A cluster commonly has multiple containers with different RF: an RF3 container for Tier-1 production, an RF2 container for general-purpose, etc.

**Why not the others:**
- A) Cluster-level RF is incorrect; the granularity is per-container.
- C) RF and node count are unrelated. RF2 works on any cluster of 3+ nodes; RF3 works on any cluster of 5+ nodes.
- D) DSF and its RF apply uniformly across hypervisors. It is platform-level, not hypervisor-specific.

**The trap:** A is intuitive ("RF must be a cluster-wide setting"). The per-container granularity is one of DSF's design strengths.

---

**Q4.** A customer has an 8-node cluster with 245 TB raw capacity. They configure all data with RF=2 and enable compression. Mixed enterprise workloads are expected to compress at roughly 2x. Approximately what is the effective usable capacity (after RF, ~12% reservation, and compression)?
**Cert relevance:** NCP-MCI · NCM-MCI

A) 122 TB
B) 215 TB
C) 245 TB
D) 81 TB

**Answer:** B

**Why this answer:** Walk the math: 245 TB raw / 2 (RF2) = 122.5 TB. Apply ~12% reservation: ~108 TB. Apply 2x compression: ~216 TB effective usable. The closest answer is 215 TB.

**Why not the others:**
- A) That's the result of RF only, before compression.
- C) That's raw, before any overhead.
- D) That's RF3 math, not RF2.

**The trap:** A is tempting if you stop at "divide by 2." The full calculation includes compression gain, which often returns more capacity than RF takes.

---

**Q5.** Which of the following workloads is least suited for Erasure Coding (EC-X)?
**Cert relevance:** NCP-MCI · NCP-US

A) A backup repository with infrequent writes and high capacity needs
B) A cold-tier file archive with rare modifications
C) A latency-sensitive OLTP database with high random write churn
D) A general-purpose virtual file server with mixed read/write workloads

**Answer:** C

**Why this answer:** EC has higher write amplification on small random writes (read-modify-write of stripe parity). Latency-sensitive databases with random write workloads are the canonical anti-pattern for EC. They benefit more from RF replication.

**Why not the others:**
- A) Backup repositories are EC's sweet spot: high capacity, low write churn, large sequential operations.
- B) Cold archives are similarly well-suited.
- D) General-purpose file servers can benefit from EC, especially if writes are not extreme.

**The trap:** This question rewards knowing the tradeoff: EC is great for capacity-bound, low-write-churn workloads; bad for high-random-write or latency-critical workloads.

---

**Q6.** Which DSF service is responsible for re-replicating data after a node failure?
**Cert relevance:** NCP-MCI · NCM-MCI

A) Stargate
B) Cassandra
C) Curator
D) Acropolis

**Answer:** C

**Why this answer:** Curator is the background scrubber and orchestrator. After a node failure, Curator detects missing replicas and orchestrates re-replication to restore configured RF.

**Why not the others:**
- A) Stargate is the data path. It routes I/O, not background re-replication.
- B) Cassandra holds metadata. It does not move data.
- D) Acropolis manages VM lifecycle (HA, Live Migration), not DSF data operations.

**The trap:** D is plausible if you forgot that Acropolis's scope is VM lifecycle, not storage. Curator owns background storage work.

---

**Q7.** A 4-node Nutanix cluster has one node fail. The cluster is configured for RF=2 across all containers. What is the cluster's state immediately after the failure?
**Cert relevance:** NCP-MCI · NCM-MCI

A) The cluster is offline until the failed node is replaced
B) The cluster is online; VMs that were on the failed node are restarted on surviving nodes; Curator begins re-replication of data that lost a replica; during the re-replication window the cluster runs at reduced redundancy and a second node failure could cause data loss
C) The cluster is online but in read-only mode
D) The cluster requires manual intervention to restart VMs and to re-replicate data

**Answer:** B

**Why this answer:** This is the canonical RF2 single-node-failure scenario. Self-healing is automatic. The re-replication window is real and the second-failure exposure is real.

**Why not the others:**
- A) RF2 specifically tolerates single-node loss. Cluster stays online.
- C) Read-only is not a normal AOS state.
- D) Automation handles VM restart (HA) and re-replication (Curator). No manual intervention required.

**The trap:** A is a mental model from older redundancy systems. AOS's self-healing is fully automated.

---

**Q8.** A customer's storage architect says: "I'm worried about distributed storage tail latency. Our database has a 5ms p99 read latency requirement. Can DSF meet that?" What is the strongest SA response?
**Cert relevance:** sales-relevant · NCX-MCI prep

A) "Of course! DSF is fast; you have nothing to worry about."
B) "DSF on all-NVMe nodes typically delivers sub-millisecond p99 reads for cached data and low-single-digit-ms p99 for cold reads. Whether 5ms p99 is achievable depends on your specific access patterns, working set size, network topology, and cluster scale. The right answer is to run a POC with your actual workload and measure. Let me propose a POC scope."
C) "DSF can't match a Pure FlashArray for tail latency."
D) "Tail latency isn't really a concern with DSF."

**Answer:** B

**Why this answer:** Specific, honest, ends with a concrete proposal. Acknowledges the metric, gives realistic numbers, identifies the variables that affect the answer, and offers the right next step (POC).

**Why not the others:**
- A) Overconfident and generic. The architect has heard this from every vendor.
- C) Concedes too much. DSF on all-NVMe with proper sizing competes with all-flash arrays on most metrics.
- D) Dismissive of a legitimate concern. Costs you credibility.

**The trap:** A and D are confident-defensive answers. C is conceding-defensive. B is the honest, specific, productive answer that wins customer trust.

---

**Q9.** Which of the following is true about the relationship between Stargate and the OpLog?
**Cert relevance:** NCP-MCI · NCM-MCI

A) Stargate writes to the OpLog after the Extent Store has confirmed durability
B) Stargate writes to the OpLog (local and remote) and acknowledges the VM write only after the OpLog writes are durable; the Extent Store drain happens asynchronously afterward
C) Stargate bypasses the OpLog for all writes; OpLog is only used for reads
D) Stargate is unrelated to the OpLog; OpLog is owned by Curator

**Answer:** B

**Why this answer:** The data path covered earlier in the module: Stargate writes to local OpLog, replicates to remote OpLog, acknowledges the VM after both succeed. Extent Store drain is asynchronous and out of the synchronous write path.

**Why not the others:**
- A) Reverses the order. Extent Store comes after OpLog.
- C) OpLog is the write buffer; reads are served by Content Cache and Extent Store.
- D) Curator scans periodically; it does not own the OpLog. Stargate writes to OpLog.

**The trap:** A is a misremembering of the order. The exam tests whether you actually internalized the data path or just memorized the names.

---

**Q10.** A customer has an 8-node cluster with mixed compute-balanced and storage-only nodes. They want to migrate a backup repository workload (heavy storage, light compute) to this cluster. Which approach is correct?
**Cert relevance:** NCP-MCI · sales-relevant

A) Storage-only nodes cannot host data; they only provide quorum
B) Storage-only nodes contribute their disks to the Storage Pool, expanding cluster capacity; the backup repository workload's data will be distributed across all nodes (compute-balanced and storage-only) according to cluster policy
C) Storage-only nodes can only be used for snapshots, not for primary data
D) The customer must create a separate cluster for storage-heavy workloads

**Answer:** B

**Why this answer:** Storage-only nodes are full DSF participants from a data perspective. They contribute capacity to the Storage Pool. Data is distributed across all nodes per the cluster's RF/EC policies. Storage-only nodes have minimal compute (for the CVM and DSF), which is why they are optimized for capacity-heavy workloads.

**Why not the others:**
- A) Storage-only nodes do hold data; they are not just quorum participants.
- C) No such restriction.
- D) Mixing compute-balanced and storage-only in one cluster is the canonical answer for capacity-imbalanced workloads.

**The trap:** A reflects a partial understanding ("storage-only" sounds like it might mean "supporting role only").

---

**Q11.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep · NCM-MCI prep
**Format:** Short-answer / design discussion. Write your reasoning. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A customer wants to design a 12-node Nutanix cluster for a mixed workload: a 25 TB SQL Server database (random read-heavy, latency-sensitive, with Always On HA across two database VMs), 600 VDI desktops with persistent profiles (boot storms at 8am, ~50 GB per profile), and 250 general-purpose VMs (mixed read/write, no extreme latency requirements). They have a separate backup target (existing dedup appliance off-cluster). Network is 25 GbE per node with redundant uplinks. Hardware is all-NVMe, 7.68 TB drives, 4 drives per node.

**The challenge:**
Design the storage container topology. Choose RF and EC settings per workload. Calculate capacity. Justify your choices. Identify what you still need to know.

**A strong answer covers:**
- **Three storage containers, one per workload.** Containers are the policy boundary; different workloads benefit from different policies.
- **Container 1: SQL Server.** RF2 (because Always On provides application-level HA, eliminating the need for RF3 for resilience). Compression on (databases compress well, often 2x). Dedup off (low data uniqueness for transactional data). EC off (random write churn makes EC's write amplification a poor fit).
- **Container 2: VDI.** RF2 (boot storms are write-heavy and benefit from less write amplification). Compression on (OS pages compress well). On-disk dedup on (VDI persistent profiles share OS pages). EC off (write-heavy profile churn).
- **Container 3: General-purpose.** RF2. Compression on. Dedup cache-only (mixed data, not enough uniqueness for on-disk dedup gains). EC: candidate for EC 4+1 if the workload skews toward read-heavy and the customer values capacity efficiency. Document the tradeoff.
- **Capacity math.** 12 nodes × 4 drives × 7.68 TB = 368.64 TB raw. Apply RF2: 184.32 TB. Apply ~12% reservation: ~162 TB. Apply 2x compression on the workload mix: ~324 TB effective usable. Note: VDI dedup may add another 20-30% capacity gain, pushing usable higher for VDI workload.
- **Network considerations.** 25 GbE is sufficient for the workload. Verify redundant uplinks for cluster-network resilience.
- **Resilience.** RF2 across all three containers tolerates single-node loss. Document the 2-3 hour re-replication window for the data volumes involved.
- **Workload placement.** Use VM-host affinity to keep SQL Server primary and secondary on different nodes (they are already protected by Always On at the application layer; this is belt-and-suspenders). VDI scheduler: leverage AHV's default behavior; ADS rebalances during boot storms naturally.
- **What you still need to know:** SQL Server's specific p99 latency requirement, VDI golden image / linked clone strategy, growth forecast for the database, customer's appetite for EC on the general-purpose container, backup window requirements (for snapshot scheduling).

**A weak answer misses:**
- Defaulting to RF3 globally without considering Always On for SQL.
- Enabling EC across all containers without considering write profile.
- Forgetting compression and dedup in capacity math.
- Not naming the re-replication window as a real concern for RF2.
- Treating VDI as similar to SQL (the dedup case is very different).

**Why this question matters for NCX:** NCX-MCI panels expect storage design that integrates application architecture (Always On), workload patterns (random vs sequential, read vs write), and capacity efficiency choices (compression, dedup, EC). A strong answer reads like a senior architect's design document, not like a feature list.

---

**Q12.** *(NCX-style architectural defense, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / architectural defense. Write your response.

**The scenario:**
You are in front of a customer's senior storage architect, who has spent the last 18 years engineering all-flash arrays for the same enterprise. He says:

> *"Software-defined storage running on shared compute hosts means write amplification I can't audit, network round-trips I can't tune, and a vendor that updates the storage stack as part of a hypervisor upgrade. My all-flash array has dedicated controllers, NVRAM I can characterize, replication I can configure precisely, and a firmware cadence I control. Why would I trade that for distributed storage?"*

**The challenge:**
Respond. He is making a serious argument. Address it.

**A strong answer covers:**
- **Acknowledge what is real.** Distributed storage does involve write amplification, network round-trips, and a software-update cadence that ties to the platform. Pretending otherwise loses credibility.
- **Reframe each concern specifically:**
  - **Write amplification.** DSF's write path is well-characterized. OpLog is on local NVMe; replication adds one or two network round-trips. The amplification is bounded and measurable. Curator background work (compression, EC, ILM) is auditable through Curator metrics. Offer to share the actual numbers from a customer reference of similar profile.
  - **Network round-trips.** Yes, they exist. They are also bounded: local OpLog is sub-millisecond, remote OpLog over 25/100 GbE adds modest single-digit microseconds in normal operation. Network quality matters; that is why design includes proper switching fabric and topology. Offer a network reference architecture.
  - **Storage stack updates with the platform.** This is a real architectural choice. The benefit: storage and compute upgrade together, no version-skew incidents. The cost: less granular control over storage firmware. Most enterprises find this a net positive once they see the LCM workflow. Offer to demo LCM in a controlled change window.
  - **Auditability.** DSF exposes more telemetry per workload than a typical array. Per-VM IOPS, latency, read/write split, cache hit rate, compression ratio, dedup ratio. Show him Prism's analytics dashboards on the architect's actual workload during POC.
- **Reframe the "dedicated controller" comparison.** Dedicated controllers are real, and they have real downsides too. The controller is a single chassis with two CPUs that becomes the cluster's bottleneck at scale. DSF distributes that work across N CVMs, scaling horizontally. The controllers also fail, take dual-controller paths through firmware versions, and consume rack space and power. The tradeoff is real in both directions.
- **Address the "I trade what I have" concern directly.** The right answer is that he doesn't have to trade. Run his existing all-flash array workloads on Nutanix-on-ESXi for an evaluation period. Use Move (Module 3) to migrate selected workloads to DSF. Measure. Decide. The architecture supports parallel running.
- **Close with a concrete proposal.** *"You're right that distributed storage is different in shape from what you've been engineering. The right way to evaluate it is not to argue the architectures abstractly but to run your specific workloads on it and compare metrics on your data, on your scale, with your tools. Let me propose a POC scope: bring three of your workloads (one OLTP, one VSI, one capacity-heavy) onto a Nutanix cluster for 60 days, instrument both sides, and let the data drive the decision. If DSF doesn't meet your metrics, we'll know specifically where, and we'll have a conversation about whether to address those gaps or stay with arrays for those workloads. Can we agree on the POC scope today?"*

**A weak answer misses:**
- Dismissing the architect's experience.
- Claiming DSF is universally better than all-flash arrays.
- Skipping the auditability point (his concern about telemetry is real and answerable).
- Not naming the firmware-cadence tradeoff honestly.
- Not closing with a POC proposal that turns the conversation into a measurable evaluation.

**Why this question matters for NCX:** NCX-MCI panels test the disposition required to win over a senior incumbent architect. The right disposition is: engage their experience seriously, acknowledge tradeoffs precisely, reframe concerns into measurable test scenarios, propose a concrete next step. This is also the disposition that wins enterprise deals where the storage architect is the gatekeeper.

---

## What You Now Have

You can now trace a VM write through DSF end to end: guest syscall, hypervisor passthrough, local CVM Stargate, local OpLog, remote OpLog (RF=2 or RF=3), acknowledgment, asynchronous Extent Store drain, eventual compression and dedup, eventual EC conversion. Every component named. The write path is in your hands.

You know the storage hierarchy: Pool → Container → vDisk → Extent → Extent Group. You know the Container is the policy boundary. You can answer "where do I configure RF and compression?" without thinking.

You know RF2 and RF3 in detail: capacity overhead, failure tolerance, when to choose which, the math of the choice. You can recommend RF based on workload, application HA, and capacity budget, not by default.

You know Erasure Coding (EC-X) intimately: 4+1 vs 4+2, capacity savings, performance tradeoffs, the specific workloads where EC wins and where it costs. You can size a cluster correctly for EC requirements (5+ nodes for 4+1, 7+ for 4+2).

You know compression and dedup in their honest forms: what they cost, what they deliver, when to enable them, when to leave them off. You quote ranges, not marketing peaks.

You know how Curator works: background scans, re-replication, EC conversion, ILM, capacity reclamation. When NCC reports Curator stalled, you know what that means.

You can walk a customer through a node-failure scenario: detection, VM restart, Curator re-replication, the re-replication window, the second-failure exposure, the duration math.

You can do capacity planning math: raw, RF, reservation, compression, dedup, effective usable. Step by step. With the assumptions called out.

You have twelve practice questions worth of storage discrimination: OpLog vs Content Cache, hierarchy navigation, RF mechanics, EC tradeoffs, capacity calculation, failure recovery, Curator role, storage-only nodes, and two NCX-style design defenses (workload-aware container design, and the architectural defense against a senior incumbent storage architect).

You are now ready for networking. Module 6 takes the same depth treatment to AHV networking, Open vSwitch, virtual switches, VLANs, bridges, and Flow microsegmentation. The shape of the network matters; for some customers, it is the dimension that decides the deal.

---

## Cross-References

- **Previous:** [Module 4: Prism (Element and Central)](./04-prism-management.md)
- **Next:** [Module 6: Networking and Microsegmentation](./06-networking-flow.md)
- **Glossary:** [DSF](./appendix-a-glossary.md#dsf) · [Storage Pool](./appendix-a-glossary.md#storage-pool) · [Storage Container](./appendix-a-glossary.md#storage-container) · [vDisk](./appendix-a-glossary.md#vdisk) · [Extent](./appendix-a-glossary.md#extent) · [Extent Group](./appendix-a-glossary.md#extent-group) · [OpLog](./appendix-a-glossary.md#oplog) · [Extent Store](./appendix-a-glossary.md#extent-store) · [Content Cache](./appendix-a-glossary.md#content-cache) · [RF](./appendix-a-glossary.md#rf) · [EC-X](./appendix-a-glossary.md#ec-x) · [ILM](./appendix-a-glossary.md#ilm)
- **Comparison Matrix:** [Storage Row](./appendix-b-comparison-matrix.md#storage) · [Replication Row](./appendix-b-comparison-matrix.md#replication) · [Capacity Efficiency Row](./appendix-b-comparison-matrix.md#capacity-efficiency)
- **Objections:** [#12 "Tail latency vs all-flash arrays"](./appendix-d-objections.md#obj-012) · [#13 "Software-defined storage trust"](./appendix-d-objections.md#obj-013) · [#15 "Rebuild times"](./appendix-d-objections.md#obj-015) · [#19 "Capacity efficiency vs my array"](./appendix-d-objections.md#obj-019) · [#24 "Storage admin role obsolescence"](./appendix-d-objections.md#obj-024)
- **Discovery Questions:** [Q-STOR-01 Workload IOPS / latency profile](./appendix-e-discovery-questions.md#q-stor-01) · [Q-STOR-02 Capacity targets and growth](./appendix-e-discovery-questions.md#q-stor-02) · [Q-STOR-03 Existing array dependencies](./appendix-e-discovery-questions.md#q-stor-03) · [Q-STOR-04 Backup integration](./appendix-e-discovery-questions.md#q-stor-04) · [Q-STOR-05 Compliance / data placement](./appendix-e-discovery-questions.md#q-stor-05)
- **Sizing Rules:** [RF capacity math](./appendix-f-sizing-rules.md#rf-overhead) · [EC sizing](./appendix-f-sizing-rules.md#ec-sizing) · [Compression / dedup planning](./appendix-f-sizing-rules.md#compression-dedup)
- **CLI Reference:** [`ncli` storage commands](./appendix-g-cli-reference.md#ncli-storage) · [Curator CLI](./appendix-g-cli-reference.md#curator-cli) · [vDisk diagnostic tools](./appendix-g-cli-reference.md#vdisk-tools)
- **Reference Architectures:** [Mixed-workload mid-market](./appendix-i-reference-architectures.md#ra-mid) · [VDI-optimized](./appendix-i-reference-architectures.md#ra-vdi)
- **POC Playbook:** [Storage performance demo](./appendix-j-poc-playbook.md#storage-perf) · [Failure simulation demo](./appendix-j-poc-playbook.md#failure-sim)

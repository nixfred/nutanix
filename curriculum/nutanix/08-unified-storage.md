---
module: 08
title: Unified Storage (Files, Objects, Volumes)
estimated_reading_time: 32 min
prerequisites:
  - Modules 01 through 07
  - DSF concepts from Module 05
  - Familiarity with file shares (SMB / NFS), object storage (S3), and iSCSI
  - Working CE cluster
key_terms:
  - Nutanix Files
  - FSVM (File Server VM)
  - File Server
  - Nutanix Objects
  - Object Store
  - Bucket
  - Versioning
  - WORM (Write Once Read Many)
  - Nutanix Volumes
  - Volume Group
  - iSCSI Initiator / Target
  - SMB / NFS / S3
  - Files Analytics
  - Self-Service Restore (SSR)
  - Anti-ransomware
diagrams:
  - files-architecture
  - objects-architecture
  - unified-storage-consolidation
cert_coverage:
  NCA: ~5%
  NCP-MCI: ~10%
  NCM-MCI: ~5%
  NCP-US: ~80% (heaviest cert weight in the curriculum for this specialty)
sa_toolkit:
  related_objections: [obj-031, obj-032, obj-033, obj-034]
  related_discovery: [q-stor-06, q-stor-07, q-stor-08, q-stor-09]
---

# Module 8: Unified Storage (Files, Objects, Volumes)

> **Cert coverage:** NCA (~5%) · NCP-MCI (~10%) · NCM-MCI (~5%) · NCP-US (~80% heavy)
> **SA toolkit:** Objections #31 through #34 · Discovery Q-STOR-06 through Q-STOR-09

---

## The Promise

By the end of this module you will:

1. **Make the unified-storage consolidation pitch on a whiteboard in 5 minutes.** Replace separate filers (NetApp, Isilon), separate object storage (Cloudian, Scality, AWS S3 on-prem), and separate iSCSI arrays with one Nutanix cluster. Three vendor relationships consolidate to one. Three refresh cycles consolidate to one. The customer walks out understanding the consolidation case.
2. **Pass roughly 80% of NCP-US, the Unified Storage specialty cert.** This module is the foundation for that cert. NCP-US tests Files, Objects, and Volumes architecture, configuration, and operations.
3. **Position Files, Objects, and Volumes against their incumbent competitors.** NetApp ONTAP for files. AWS S3 or on-prem object stores for objects. Pure or Dell arrays for iSCSI block. Each comparison is honest: Nutanix is competitive in 90% of cases, behind in specific niches, with the integration and platform consolidation as the durable win.
4. **Recognize when unified storage is the wrong answer.** Some workloads (extreme-throughput HPC filers, hyperscale object workloads, very specialized SAN block scenarios) are still better served by purpose-built storage. Naming the gap honestly buys you customer trust.
5. **Make the backup-target consolidation case.** Nutanix Objects as a Veeam, Commvault, or Rubrik target replaces a separate Data Domain, Quantum, or NetApp StorageGRID appliance. This is one of the more concrete economic wins in 2026 and one of the easier-to-prove ROI cases.
6. **Identify which storage service to recommend for which use case.** Files for user shares and SMB-aware applications. Objects for backup targets and cloud-native applications. Volumes for non-Nutanix consumers (physical hosts, Oracle RAC on bare metal, legacy apps requiring iSCSI). The choices are clean once you know the use cases.

This module is the answer to "what other storage problems can Nutanix solve?" The answer, more often than customers expect, is "most of them."

---

## Foundation: What You Already Know

Your customer has multiple storage appliances. Walk into any enterprise datacenter and you find:

- A **NetApp filer** (or Isilon, Pure FlashBlade, Dell PowerScale) handling user shares, application file storage, and backup targets that need NFS or SMB.
- An **object storage tier** somewhere: AWS S3 for cloud-resident data, possibly Cloudian or Scality or MinIO on-prem for archive or backup.
- An **iSCSI array** for workloads that need block storage but aren't on the VMware cluster: physical database servers, legacy applications, bare-metal Linux for specific use cases, sometimes backup targets.
- Possibly a **Data Domain or similar dedup appliance** for backup repositories.

Each of those is a separate appliance with its own controller hardware, its own software upgrade cycle, its own support contract, its own management UI, its own replication story, and its own capacity-planning surface. Your customer's storage admin has spent years getting good at this. They are not necessarily happy about it.

The unified-storage story for Nutanix is: those four appliances consolidate to one cluster. **Files** replaces the filer. **Objects** replaces the object store and often the dedup appliance. **Volumes** replaces the iSCSI array. All three run on the same DSF substrate, in the same management plane, with the same lifecycle.

This is consolidation at the storage tier. It is the same operational argument that drove HCI adoption for compute, applied to storage services. The customer who saved one team's effort by collapsing compute-storage-network into HCI saves another team's effort by collapsing file-object-block onto unified storage.

> [!FROM-THE-SA-CHAIR]
> The unified-storage pitch lands hardest with operations leaders and CIOs, less so with individual storage admins (who reasonably value their domain expertise). The right framing for the storage admin: *"Your expertise stays valuable. The consolidation removes the appliances you spend time maintaining, not the storage knowledge you bring to the conversation. You become the architect for one platform instead of the operator of four. Your team gets back the hours that go into vendor escalations, firmware compatibility matrices, and quarterly upgrade dances."* That sentence respects their expertise and reframes the change as career advancement, not displacement.

---

## Core Content

### Nutanix Files: SMB and NFS at HCI Scale

**Nutanix Files** is a scale-out file storage service that runs on top of a Nutanix cluster. It provides SMB shares (for Windows clients and applications) and NFS exports (for Linux clients and applications). The architecture:

- A **File Server** is a logical SMB/NFS service. You can have multiple File Servers per cluster (e.g., one for Production, one for Engineering, one for VDI profiles).
- Each File Server is implemented as a cluster of **FSVMs** (File Server VMs). FSVMs are dedicated VMs that run the file-services stack on top of Nutanix. Three FSVMs per File Server is the typical minimum for HA and scale-out.
- FSVMs distribute the file-services workload across the underlying cluster. Adding FSVMs scales capacity and performance horizontally.
- File data sits on DSF with all of DSF's properties: RF, compression, deduplication, snapshots, replication.

**Capabilities:**

- **SMB 2.x and 3.x** with full Active Directory integration. Kerberos, ACLs, ABE (Access-Based Enumeration), DFS-N integration. Standard for Windows shares and applications.
- **NFS v3 and v4** for Linux clients and applications. Kerberos for v4.
- **Multi-protocol shares** (SMB and NFS on the same data, with appropriate ACL translation).
- **Snapshots** at the File Server, share, or path level. Native to DSF; instant; no I/O penalty.
- **Self-Service Restore (SSR).** End users can restore deleted files from snapshot via the Windows "Previous Versions" tab, without involving IT.
- **Files Analytics.** Built-in analytics on file usage, access patterns, hot/cold data identification, anomaly detection (a file pattern looks like ransomware). Comes with the Files product.
- **Anti-ransomware.** Real-time pattern detection on writes; alerts and (optionally) blocks suspicious activity. This has matured significantly in recent AOS releases.
- **Replication** to another Nutanix cluster (DR for file data).
- **Quotas** at the share or directory level.
- **DFS-N integration** for namespace-based access.

> [!FAMILIAR]
> The protocol surface maps cleanly onto NetApp ONTAP, EMC Isilon, or any enterprise filer your customer runs today. SMB 3.x, NFS v3/v4, AD integration, ACLs, quotas, snapshots, replication. The vocabulary is the vocabulary your customer's storage team already speaks. The architectural difference is what runs underneath: instead of a purpose-built filer appliance with proprietary controllers, Files runs as FSVMs on the same Nutanix cluster as the customer's compute. The user-facing experience is the same; the platform underneath is consolidated.

**When Files is the right tool:**

- User home directories and department shares (the canonical SMB use case).
- Application file storage: Windows-based applications that need an SMB share, Linux apps that need NFS.
- VDI persistent profiles (combined with Citrix or Horizon profile management).
- Web content stores, software distribution shares.
- Backup target for VM-level backups (where SMB/NFS is the protocol).

**When Files is not the right tool:**

- Extreme-throughput HPC scenarios where you need genuinely thousands of clients hitting one filer at saturation. Dedicated parallel filers (Lustre, GPFS / IBM Spectrum Scale, dedicated Isilon clusters) still win at the very top end.
- Workloads requiring features specific to a particular filer (NetApp FlexClone for cloning at the file level, FlexCache for distributed caching, certain ONTAP-specific snapshot policies). Files is competitive but does not match every NetApp feature.

> [!ON-THE-EXAM] **NCP-US**
> Files architecture is the largest topic on NCP-US. Memorize: a File Server is the logical SMB/NFS service; it is implemented by a cluster of FSVMs (typically 3+); FSVMs are dedicated VMs running on the underlying Nutanix cluster; data lives on DSF. Trap distractor: "Files is delivered as a feature of AHV with no dedicated VMs" (false; FSVMs are real, dedicated VMs).

---

### Diagram: Files Architecture

**id:** `files-architecture`
**type:** architecture
**caption:** A Nutanix File Server is implemented as a cluster of FSVMs running on top of Nutanix. Clients connect via SMB or NFS to a load-balanced endpoint.
**exam_relevance:** [NCP-US, NCP-MCI]
**whiteboard_ready:** true

**Elements:**
- Top: SMB and NFS clients (Windows desktops, Linux servers) connecting to a "File Server endpoint" (DNS round-robin or VIP)
- Below the endpoint: 3 FSVMs (rust/orange) labeled "FSVM-01, FSVM-02, FSVM-03"
- Below the FSVMs: a horizontal bar labeled "DSF (the underlying Nutanix cluster)" containing the file data
- A separate small box at the side labeled "Files Analytics" connected to FSVMs
- Sub-labels on FSVMs: "SMB 3.x · NFS v4 · Multi-protocol · ACL translation"

**Connections:**
- Clients to File Server endpoint: standard SMB or NFS over TCP
- Endpoint to FSVMs: DNS round-robin or VIP-based load balancing
- FSVMs to DSF: file data persisted in DSF with full RF, compression, dedup, snapshots, replication

**Annotations:**
- "Adding an FSVM scales capacity and performance horizontally."
- "All DSF benefits apply: snapshots, replication, compression, dedup, EC."
- "AD integration: Kerberos, ACLs, DFS-N. Same standards your customer's Windows team uses today."
- "Anti-ransomware monitoring runs on FSVMs in real time."

**Why this diagram exists:** Customers and BlueAlly SAs alike treat Files as "magic on top of Nutanix." The reality (dedicated FSVMs, real architecture, real scaling pattern) is more grounded. This diagram makes the implementation visible. Whiteboard it when a customer compares Files to NetApp.

---

### Files Analytics: A Real Differentiator

Most filers have analytics. **Files Analytics** is genuinely good. It runs as a small integrated service within Files and provides:

- **File-system aging.** What percentage of data hasn't been accessed in 12+ months? (Often surprising and budget-relevant.)
- **Top users by capacity.** Who owns the largest file footprints?
- **Top users by I/O activity.** Who is actively reading/writing the most?
- **File type breakdown.** PDF, video, log, database files: where is your storage actually going?
- **Anomaly detection.** A user account that suddenly accesses 10,000x normal volume is flagged. This is the foundation of the anti-ransomware capability.
- **Permission audit.** Which users have access to which shares; who has elevated permissions; what changed recently.
- **Compliance reporting.** Standard reports for various compliance frameworks.

> [!FROM-THE-SA-CHAIR]
> Files Analytics is one of the cleaner customer demos. The 3-minute version: pull up Files Analytics on a populated environment, show the file-aging dashboard ("70% of your data hasn't been accessed in over a year"), show the top-users-by-capacity report, show the access anomaly detection. Customers respond to seeing real, actionable information that their current filer doesn't expose this clearly. The aging-data insight in particular often unlocks tiering or archival decisions the customer has been deferring for years.

### Anti-Ransomware: The Security Story for Files

Files includes real-time ransomware detection that watches file write patterns. Encrypted-at-write patterns, mass-rename patterns, and suspicious extension changes trigger:

- Alerts (immediate notification to administrators).
- Optional blocking (refusing the suspicious writes, preventing further damage).
- Snapshot creation at detection time (preserving the pre-attack state).

For customers concerned about ransomware (which by 2026 is essentially every customer), this is a meaningful security story. It is not a complete anti-ransomware strategy by itself; it is a layer that complements endpoint protection, network segmentation, backup hygiene, and user awareness training.

---

### Nutanix Objects: S3 at Cluster Scale

**Nutanix Objects** is an S3-compatible object storage service running on top of a Nutanix cluster. It provides:

- **S3 API compatibility.** Standard S3 endpoints, signatures, requests. Most S3-compatible tools work without modification.
- **Buckets** as the unit of organization, with access policies, IAM-style users, and versioning.
- **Object versioning.** Multiple versions of an object retained per bucket configuration.
- **WORM (Write Once Read Many).** Compliance-driven immutability. Useful for regulatory archives and some backup retention scenarios.
- **Lifecycle policies.** Auto-tier or auto-delete objects based on age.
- **Replication** to other Nutanix Objects deployments or to AWS S3.
- **Multi-tenancy.** Separate object stores for different tenants, each with their own users and policies.

> [!DIFFERENT]
> Objects' S3 API is the same API your customer uses against AWS S3. The architectural difference is where the data lives and who pays for what. AWS S3 is consumption-priced, ingress-free, egress-billed; it is operationally hands-off and economically well-suited for some patterns (variable workloads, cloud-resident apps). Nutanix Objects is capex-or-subscription priced on customer-owned cluster capacity; it is operationally consolidated with the rest of the platform and economically well-suited for other patterns (steady-state backup repositories, latency-sensitive on-prem workloads, compliance-driven data sovereignty). For most customer environments by 2026, the right answer is some-of-each: Objects on-prem for the steady high-volume workloads, AWS S3 for the truly cloud-native and elastic patterns. Frame it as complementary, not competitive.

The architecture is similar in spirit to Files: dedicated VMs (the Objects "Object Service") run the S3 stack on top of DSF. Capacity scales by adding cluster capacity; throughput scales by adding object service VMs.

**When Objects is the right tool:**

- **Backup targets.** Veeam, Commvault, Rubrik, Cohesity all support S3 as a target. Objects becomes a Veeam repository, replacing a separate Data Domain or NetApp StorageGRID. This is one of the most concrete consolidation wins.
- **Cloud-native applications.** Apps designed against the S3 API (which is most modern apps) work natively against Objects.
- **Archives and long-term retention.** Use lifecycle policies to age data toward longer retention.
- **Compliance archives.** WORM mode for regulatory data (financial, healthcare, legal).
- **Big data and analytics intermediate storage.** Spark, Hadoop, modern data-warehouse staging often use S3-compatible storage.

**When Objects is not the right tool:**

- Hyperscale workloads at the AWS S3 scale (tens of petabytes, millions of TPS). Objects scales well but is not in the same league as AWS-scale infrastructure.
- Workloads that require AWS-specific features beyond S3 API (S3 Glacier deep archive economics, S3 Select compute pushdown for some specific patterns).
- Workloads where the cloud-economics arbitrage (capex vs cloud-opex) clearly favors public cloud.

> [!ON-THE-EXAM] **NCP-US**
> Objects topics: bucket configuration, object versioning, WORM, lifecycle policies, S3 access (key pair, signature), multi-tenancy. Memorize: Objects is S3-compatible (Object Service VMs running on DSF). Trap distractor: "Objects requires a dedicated cluster separate from VM workloads" (false; Objects runs on the same cluster).

---

### Diagram: Objects Architecture

**id:** `objects-architecture`
**type:** architecture
**caption:** S3-compatible storage running on Nutanix. Clients hit the S3 endpoint; data lives in DSF with full platform features.
**exam_relevance:** [NCP-US]
**whiteboard_ready:** true

**Elements:**
- Top: S3 clients (backup software like Veeam, cloud-native applications, S3 SDKs in user code) hitting an "S3 endpoint" (gold)
- Below the endpoint: 3 Object Service VMs (rust/orange) labeled "Object Service-01, -02, -03"
- Below: Object Stores (gold), each containing buckets, versioning, lifecycle policies
- Bottom: DSF (rust) with the actual object data persisted as DSF objects
- Side: WORM lock indicator on selected buckets; lifecycle policy indicators on others; replication arrow to another Objects deployment or AWS

**Connections:**
- Clients to S3 endpoint: standard S3 API over HTTPS
- Endpoint to Object Service VMs: load-balanced
- Object Service to DSF: data persisted with full DSF properties

**Annotations:**
- "Standard S3 API. Tools that work with AWS S3 work with Objects with endpoint-only changes."
- "WORM compliance for regulatory archives."
- "Replication to another Objects deployment or to AWS S3."
- "Often the answer for backup targets: replaces dedicated Data Domain or NetApp StorageGRID."

**Why this diagram exists:** Customers conflate "Objects" with "S3 in cloud" or with separate object-storage appliances. This diagram shows that Objects is the cloud's S3 model running on cluster infrastructure with platform-grade durability. Use it when a customer asks "wait, you have S3?"

---

### Nutanix Volumes: iSCSI for Non-Nutanix Consumers

**Nutanix Volumes** is the iSCSI block-storage service. It exposes LUNs (called Virtual Volumes in Nutanix vocabulary) over iSCSI to external consumers: physical servers, non-Nutanix VMs, certain bare-metal database deployments, legacy applications.

The architecture:

- **Volume Group:** a logical grouping of one or more LUNs that are presented together. Useful for application-aware grouping (e.g., "all the disks for SQL Server cluster instance ABC").
- **iSCSI Target:** the cluster acts as an iSCSI target. Standard initiator software on Linux, Windows, ESXi, or other consumers connects.
- **Multi-pathing:** the cluster exposes multiple iSCSI portal IPs for HA. Standard iSCSI multipath software on the consumer handles failover.
- **Data resides in DSF:** with all DSF properties (RF, compression, snapshots, replication, EC if applicable).

**When Volumes is the right tool:**

- **Physical Linux or Windows servers** that need block storage but aren't running on Nutanix as VMs. Examples: standalone database hosts, legacy physical applications, specialized hardware that runs bare metal.
- **Oracle RAC** or other clustered databases that prefer raw block storage shared across nodes (RAC's ASM works well over iSCSI).
- **Legacy iSCSI consumers** that the customer hasn't yet migrated.
- **VMware ESXi clusters not on Nutanix** that want to use Nutanix storage. ESXi can mount Volumes as iSCSI datastores. (This is a niche but real use case during migrations.)
- **Backup targets** for backup software that prefers iSCSI over SMB/NFS/S3.

**When Volumes is not the right tool:**

- Standard VM workloads on AHV. Just use AHV's normal vDisk presentation; you don't need iSCSI on top.
- Workloads where the consumer can use SMB, NFS, or S3 instead. Block protocols are more complex than file or object; only use them where genuinely required.

> [!ON-THE-EXAM] **NCP-US**
> Volumes topics: Volume Groups, iSCSI initiator/target configuration, multi-pathing, attaching to AHV/ESXi/physical hosts. Memorize: Volume Groups are the management unit; LUNs are exposed via iSCSI; multi-pathing is provided via multiple portal IPs. Trap distractor: "Volumes requires AHV" (false; Volumes is consumed by external initiators, often physical or non-Nutanix consumers).

---

### The Cycle, Frame Two: Unified Storage as Consolidation

For an operations leader or CIO, the durable Unified Storage story is consolidation. Specifically:

| Customer Has Today | Unified Storage Replaces With |
|---|---|
| NetApp / Isilon / FlashBlade for file shares | Nutanix Files (FSVMs on DSF) |
| Cloudian / Scality / MinIO / on-prem S3 | Nutanix Objects (Object Services on DSF) |
| iSCSI array (Pure, NetApp, Dell) for non-Nutanix consumers | Nutanix Volumes (Volume Groups on DSF) |
| Data Domain / Quantum / dedicated dedup appliance | Nutanix Objects with appropriate retention (often paired with backup software's native dedup) |

Four appliances consolidate to one cluster. The customer's BOM gets simpler. Their refresh cycles align. Their support contracts consolidate. Their team's effort goes into one platform instead of four.

The economic argument is real. Run the actual numbers for the specific customer; the savings depend on their deployment size and current depreciation schedule. Typical mid-market deployments save 30-50% over five years compared to maintaining four separate storage tiers.

### The Cycle, Frame Three: Unified Storage as DSF Made Consumable

For a technical architect, the architectural frame is more interesting: unified storage is DSF's distributed-storage primitives made consumable through industry-standard protocols.

DSF already does the hard parts: distributed metadata, replication, erasure coding, compression, dedup, snapshots, geographic replication. Files, Objects, and Volumes are protocol layers on top of DSF that translate user requests (SMB/NFS/S3/iSCSI) into DSF operations.

This is why all three services automatically benefit from DSF improvements. When DSF gets better at compression in a future release, Files, Objects, and Volumes all get better at compression. When DSF improves replication, all three services benefit.

The architecture is a single platform with multiple consumption protocols. Compare this to the NetApp world where the filer is purpose-built for files and bolting on object or block requires different products (StorageGRID for object, ONTAP block features for iSCSI), each with their own software lifecycles.

### The Cycle, Frame Four: Unified Storage as the Backup-Target Win

For backup teams, the durable frame is specifically about backup repositories. Most enterprise backup deployments today use:

- **Primary backup tier:** a deduplication appliance (Data Domain, Quantum, NetApp StorageGRID-with-dedup-software).
- **Secondary tier:** sometimes tape; increasingly cloud (AWS S3, Azure Blob).
- **Cloud archive:** AWS S3 Glacier or equivalent.

This stack costs real money. The dedup appliance alone is often a six-figure capex item. Software licensing on top of the appliance is more.

Unified Storage's pitch: Nutanix Objects becomes the primary backup repository. The major backup vendors (Veeam, Commvault, Rubrik, Cohesity, HYCU) all support S3-compatible storage as a target. The customer's backup workflow stays the same; the destination changes from Data Domain to Nutanix Objects. Replication to Objects in another datacenter (or to NC2 in cloud) handles geographic redundancy. Lifecycle policies handle long-term retention.

The consolidation: dedup appliance gone, separate cloud-archive billing simplified (depending on retention strategy), one less vendor relationship.

> [!FROM-THE-SA-CHAIR]
> The backup-target consolidation is one of the easier-to-prove ROI cases in 2026. *"Pull your Data Domain quote from your last refresh. Pull your annual Data Domain support cost. Pull the cost of cloud-archive egress and re-ingest you've experienced. Now compare that five-year run-rate to running Veeam (or your backup product) against Nutanix Objects on the same cluster you're already buying. The math is usually obvious within 24-36 months."* Bring this conversation up early when discovery surfaces a backup vendor relationship; it gives you a concrete consolidation conversation that the customer can model on their own data.

---

### Diagram: Unified Storage Consolidation

**id:** `unified-storage-consolidation`
**type:** comparison
**caption:** Before: four storage tiers from four vendors. After: one Nutanix cluster providing all four functions.
**exam_relevance:** [NCP-US, sales-relevant]
**whiteboard_ready:** true

**Elements (left side, "Before"):**
- Top: VMware compute environment (blue)
- Below, 4 separate storage appliances stacked:
  - NetApp filer (gray, labeled "File / SMB / NFS")
  - Cloudian / Scality (gray, labeled "Object / S3")
  - Pure iSCSI array (gray, labeled "Block / iSCSI for non-VM consumers")
  - Data Domain (gray, labeled "Backup target / dedup")
- Side annotations: "4 vendors. 4 refresh cycles. 4 management UIs. 4 support contracts."

**Elements (right side, "After"):**
- Top: Nutanix cluster (rust) with VMs (compute layer), labeled "Compute"
- Inside the cluster, three storage-service stacks:
  - Files (FSVMs)
  - Objects (Object Service VMs)
  - Volumes (iSCSI service)
- DSF underneath all of them
- A single line label below: "1 platform. 1 vendor. 1 refresh cycle. 1 management plane."

**Connections:**
- Left: VMware compute connecting to each of the four separate appliances
- Right: clients (SMB clients, S3 clients, iSCSI initiators) connecting to the appropriate service on the unified cluster
- Backup workflow shown on right going to Objects: "Veeam / Commvault / Rubrik → Objects bucket"

**Annotations:**
- "All four storage functions on one platform."
- "DSF underneath: same RF, compression, dedup, snapshots, replication for every storage type."
- "Compute and storage scale together. Add a node, add capacity for VMs, files, objects, and block simultaneously."

**Why this diagram exists:** This is the consolidation pitch in one diagram. Whiteboard it during executive customer conversations. The "before" and "after" make the consolidation visible and the licensing/operational savings tangible. One of the most-used customer-facing diagrams in the curriculum.

---

### What Unified Storage Genuinely Lacks

Honest gap list. Read it carefully.

1. **Files vs NetApp ONTAP at the high end.** ONTAP has 30+ years of file-services maturity: FlexClone for file-level cloning, FlexCache for distributed caching, advanced quotas, ABE policy refinement, ONTAP-specific snapshot policies, deep volume-management semantics. Files is competitive for the bulk of enterprise file workloads. For customers running deep ONTAP workflows, the comparison favors ONTAP.

2. **Objects vs AWS S3 ecosystem maturity.** AWS S3 has the broadest tooling ecosystem and the most niche-feature depth (S3 Select, Glacier Deep Archive economics, S3 Outposts, dozens of S3-native AWS services). Nutanix Objects is S3-compatible at the API level, which works for nearly every common S3 tool, but AWS-specific features beyond S3 are not available.

3. **Hyperscale object workloads.** AWS-scale or Google-scale object workloads (multi-petabyte, millions of operations per second) are out of scope for Nutanix Objects. Use cloud for cloud-scale.

4. **Specialized HPC filers.** Lustre, IBM Spectrum Scale (GPFS), and other parallel filers serve workloads where extreme throughput across thousands of clients is the requirement. Files does not target this segment.

5. **Volumes vs purpose-built block arrays at the very top end.** For block workloads requiring sub-millisecond p99 latency at extreme IOPS, dedicated block arrays still have an edge in some scenarios. Most enterprise block workloads are well-served by Volumes; the very top end is the exception.

These are real. None are deal-breakers for typical mid-market or enterprise workloads. Customers in those niches need targeted purpose-built storage.

### What Unified Storage Has That Separates It From Pieces-Bought-Separately

1. **Single platform.** One cluster, one vendor, one upgrade cycle, one support contract.
2. **Shared DSF foundation.** All three services inherit DSF's snapshots, replication, compression, dedup, EC.
3. **Shared management plane.** Files, Objects, and Volumes all configured and monitored from Prism Central.
4. **Shared identity integration.** AD or SAML configured once, applies across all three services.
5. **Shared replication topology.** Files Replication, Objects Replication, and DSF Async all use compatible network paths and policies.
6. **Shared scaling.** Adding cluster capacity adds capacity for VMs, files, objects, and block simultaneously.
7. **Shared licensing.** Bundled into AOS subscriptions; specific tier requirements vary but no separate appliance licensing.

---

## Lab Exercise: Deploy Files, Create a Share, Test Snapshot Recovery

> [!LAB] **Time:** ~3 hours · **Platform:** 3-node CE cluster + Prism Central (from earlier labs)
> Note: Files deployment on CE has limitations; some advanced features may not be available. Objects deployment is typically not supported on CE. Volumes can be tested on CE.

**Steps:**

1. **Verify Files availability on your CE cluster.** From Prism Central, navigate to Services > Files. If Files is available for deployment on your CE version, proceed. If not, walk through the deployment workflow conceptually using documentation.

2. **Deploy a File Server.** Use the deployment wizard:
   - Name: `lab-fs-01`
   - Number of FSVMs: 3 (the minimum for HA)
   - Storage container: use your default container or create a new one
   - Network: place FSVMs on an existing virtual network
   - Domain: integrate with AD if you have a lab AD; otherwise use a workgroup configuration

3. **Create an SMB share.** Once the File Server is up, create a share:
   - Name: `lab-share-01`
   - Type: SMB (or NFS depending on your client capability)
   - Path: `/lab-data` (or similar)
   - Permissions: appropriate for your test environment

4. **Mount the share from a client.** From a Windows VM, map a network drive: `\\<file-server-name>\lab-share-01`. From a Linux VM, mount via NFS or SMB.

5. **Generate test data on the share.** Copy some files. Create directories. Make the share look "real."

6. **Take a snapshot of the share.** From Prism Central, Files > Snapshots > Create. Note that this is an instant operation regardless of share size.

7. **Test Self-Service Restore.** From a Windows client with the share mapped:
   - Right-click on a file > Properties > Previous Versions
   - You should see the snapshot you just took
   - Restore a previous version of a file
   - This is the user-facing recovery experience and is genuinely valuable for customers

8. **Configure a snapshot schedule.** Define a hourly snapshot schedule with retention. Confirm the schedule activates and snapshots are taken on schedule.

9. **(Optional) Test Volumes.** Create a Volume Group with one or more LUNs. From a Linux VM (one not on Nutanix, ideally; can be on Nutanix for the lab), use `iscsiadm` to discover and connect:
   ```
   iscsiadm -m discovery -t sendtargets -p <cluster-data-services-ip>
   iscsiadm -m node -T <iqn> -p <portal> --login
   ```
   The new block device should appear; format and mount it.

10. **Inspect Files via CLI.** From a CVM:
    ```
    afs                          # Files CLI (may need ssh to FSVM)
    ncli filesserver list        # List file servers
    ncli filesserver status name=<name>
    ```

**What this teaches you:**
- Files deployment workflow and architecture in practice.
- The user-facing Self-Service Restore experience (a key customer demo moment).
- iSCSI / Volumes consumption pattern from an external-style consumer.

**Customer-demo angle:** Steps 6-7 are the customer demo for Files SSR. Show a customer how an end user can restore their own deleted files without an IT ticket. Help-desk reductions land emotionally. Time it: 90 seconds total.

---

## Practice Questions

Twelve questions. Six knowledge MCQ, four scenario MCQ, two NCX-style design questions. Read each, answer in your head, then read the explanation.

---

**Q1.** What is an FSVM in Nutanix Files architecture?
**Cert relevance:** NCP-US · NCP-MCI

A) A user account with elevated file-server permissions
B) A dedicated VM that runs the file-services stack as part of a File Server cluster
C) A storage pool reserved for file-share usage
D) A snapshot type used for file recovery

**Answer:** B

**Why this answer:** FSVMs (File Server VMs) are dedicated VMs that run the SMB/NFS file-services stack on top of Nutanix. Three FSVMs typically form a File Server (the logical share-serving entity). They are real VMs, not abstractions.

**Why not the others:**
- A) FSVM is an architectural component, not a user account.
- C) Storage pools are DSF-level constructs, unrelated to FSVMs.
- D) Snapshots are a DSF feature; FSVMs do not refer to snapshots.

**The trap:** A and C reflect partial understanding. FSVMs are the implementation of Files. Memorize: File Server = logical service; FSVMs = the VMs implementing it.

---

**Q2.** Which of the following is the appropriate Unified Storage service for replacing a dedicated S3-compatible object storage appliance (such as Cloudian or MinIO) used as a backup target?
**Cert relevance:** NCP-US · sales-relevant

A) Nutanix Files
B) Nutanix Objects
C) Nutanix Volumes
D) DSF storage container directly

**Answer:** B

**Why this answer:** Nutanix Objects provides S3-compatible object storage. Backup software (Veeam, Commvault, Rubrik, Cohesity) supports S3 as a target, so Objects is the direct replacement for Cloudian/MinIO/StorageGRID in this role.

**Why not the others:**
- A) Files is for SMB/NFS, not object storage.
- C) Volumes is for iSCSI block, not object.
- D) DSF storage containers are the underlying layer; you don't expose them directly to S3 clients.

**The trap:** D is technically clever ("just use the underlying storage") but operationally wrong. Customers consume DSF through the appropriate service layer.

---

**Q3.** What is the primary use case for Nutanix Volumes?
**Cert relevance:** NCP-US

A) Replacing the AHV hypervisor's vDisk presentation
B) Providing iSCSI block storage to non-Nutanix consumers (physical hosts, bare-metal databases, legacy applications)
C) Storing snapshots for VM-level backup
D) Hosting Nutanix Files data

**Answer:** B

**Why this answer:** Volumes exposes iSCSI LUNs to external consumers: physical Linux/Windows servers, bare-metal database hosts (Oracle RAC), legacy applications that require block storage and are not running as Nutanix VMs.

**Why not the others:**
- A) AHV's vDisk presentation works natively for VMs on Nutanix; Volumes is for external consumers.
- C) Snapshots are a DSF feature; Volumes is not specifically a snapshot mechanism.
- D) Files data lives on DSF, not on Volumes.

**The trap:** A reflects misunderstanding the consumer of Volumes. Memorize: Volumes is for clients outside the Nutanix VM context (typically physical or non-Nutanix hosts).

---

**Q4.** A customer wants to enable end users to recover deleted files from a Files share without involving IT. Which capability provides this?
**Cert relevance:** NCP-US · sales-relevant

A) Self-Service Restore (SSR), accessible via Windows "Previous Versions"
B) IT-managed snapshot restoration only
C) Recovery Plan / Leap orchestration
D) Files Analytics

**Answer:** A

**Why this answer:** SSR exposes Files snapshots to end users via the Windows "Previous Versions" tab. Users can restore deleted files or earlier versions without IT involvement. This is one of Files' most operationally valuable features.

**Why not the others:**
- B) IT-managed restore is the "old way" without SSR; SSR specifically removes IT from the loop for routine recovery.
- C) Recovery Plans are for VM-level DR orchestration, not user-level file recovery.
- D) Files Analytics provides reporting, not recovery.

**The trap:** B is intuitive ("of course, only IT can restore") and misses the SSR feature. Memorize SSR as the user-facing recovery capability.

---

**Q5.** Which of the following correctly describes Files Analytics?
**Cert relevance:** NCP-US

A) An external monitoring tool that must be deployed separately on a Windows server
B) An integrated analytics service within Files that provides file aging, top users, file type breakdown, anomaly detection (anti-ransomware foundation), and permission auditing
C) A reporting export to Splunk only
D) A licensed add-on requiring NCM Ultimate

**Answer:** B

**Why this answer:** Files Analytics is integrated with the Files product. It provides the analytics described, runs as part of the Files infrastructure, and forms the foundation for the anti-ransomware capability.

**Why not the others:**
- A) Files Analytics is integrated, not external.
- C) Files Analytics has its own UI; it is not just a Splunk export (though it can integrate).
- D) Files Analytics is part of Files, not a separate licensed add-on.

**The trap:** D reflects the fear that "useful features cost extra." Files Analytics is included with Files.

---

**Q6.** What is WORM in the context of Nutanix Objects?
**Cert relevance:** NCP-US

A) A diagnostic tool for tracing S3 API calls
B) Write Once Read Many: an immutability feature that prevents modification of objects in a bucket for a defined retention period, used for compliance archives
C) A programming language SDK for Objects
D) A bucket-replication mechanism

**Answer:** B

**Why this answer:** WORM (Write Once Read Many) provides regulatory-grade immutability. Once written, objects cannot be modified or deleted until the retention period expires. Common compliance use cases: financial records, healthcare data, legal archives.

**Why not the others:**
- A) Not a diagnostic tool.
- C) Not a programming language.
- D) Not a replication mechanism.

**The trap:** Test-takers unfamiliar with compliance vocabulary may guess on this question. The retention/immutability use case is specific and important for regulated industries.

---

**Q7.** A customer with a 6-node Nutanix cluster wants to consolidate their NetApp filer (50 TB used), their Data Domain backup target (200 TB usable after dedup), and their Pure iSCSI array (30 TB used) onto Nutanix. They are concerned about whether one cluster can serve all four roles (compute + Files + Objects + Volumes). What is the recommended approach?
**Cert relevance:** NCP-US · NCP-MCI · sales-relevant

A) Tell the customer to stay with their existing storage; Nutanix cannot replace all three
B) Size the cluster appropriately for the consolidated workload (compute + storage capacity for all three storage roles); deploy Files for the filer replacement, Objects for the backup target replacement, and Volumes for the iSCSI consumer; verify that compute and storage capacity are both adequate
C) Recommend separate Nutanix clusters for each storage type
D) Recommend keeping the Data Domain because Objects can't replace it

**Answer:** B

**Why this answer:** This is exactly the unified-storage use case. One cluster, sized for the combined workload, runs Files + Objects + Volumes alongside the customer's compute. The sizing exercise must account for the additional capacity (50+200+30 TB = 280 TB net needed, before RF and overhead).

**Why not the others:**
- A) The customer's workload is well within Nutanix's consolidation sweet spot.
- C) Separate clusters defeat the consolidation purpose.
- D) Objects with appropriate sizing and combined with backup-software dedup is a credible Data Domain replacement.

**The trap:** A and D reflect undue conservatism. The sizing is non-trivial but achievable.

---

**Q8.** A customer's senior storage admin says: "We've spent 12 years building NetApp expertise. Why would we walk away from FlexClone, FlexCache, and ONTAP's snapshot maturity?" What is the strongest SA response?
**Cert relevance:** sales-relevant

A) "Files is a perfect replacement for NetApp; you'll get all the same features."
B) "You don't have to walk away. Files is competitive for the bulk of enterprise file workloads, but for the specific ONTAP capabilities you mentioned, NetApp retains advantages. The right pattern for many customers is to consolidate the bulk of the file estate onto Files (user shares, application file storage, backup repositories) and keep NetApp for the workloads that genuinely need ONTAP-specific features. Let's map your shares against the feature matrix together."
C) "NetApp is overpriced; you should switch immediately."
D) "FlexClone is unnecessary; nobody uses it anymore."

**Answer:** B

**Why this answer:** Honest about the gap, respects the customer's expertise, names a concrete coexistence pattern, and proposes a workload-mapping exercise as the next step. This is the durable enterprise SA response.

**Why not the others:**
- A) Untrue. NetApp retains real ONTAP-specific advantages; pretending otherwise loses credibility.
- C) Bashing the incumbent vendor loses the room.
- D) Dismissive of a real ONTAP capability that customers genuinely use.

**The trap:** A and D are confident-defensive. Honesty about the gap is what wins customer trust.

---

**Q9.** Which of the following backup software products commonly support Nutanix Objects as a backup repository target?
**Cert relevance:** NCP-US · sales-relevant

A) Only Veeam
B) Veeam, Commvault, Rubrik, Cohesity, and HYCU all support S3-compatible storage and can use Nutanix Objects
C) Only Nutanix's own backup product
D) None; Objects is not a supported backup target

**Answer:** B

**Why this answer:** All major enterprise backup vendors support S3-compatible storage as a target. Nutanix Objects is S3-compatible, so it works with the full ecosystem of S3-aware backup tools.

**Why not the others:**
- A) Veeam works, but the support extends well beyond.
- C) Customers use third-party backup with Objects; Nutanix has its own backup integration paths but does not exclude others.
- D) Objects is explicitly designed to be a backup target.

**The trap:** A is a common mental model from heavily Veeam-shop customers. The reality is broader.

---

**Q10.** A customer wants to deploy a new file share for a department of 200 users. Their existing NetApp filer is at end-of-life and they are deciding between buying new NetApp or consolidating onto their existing 12-node Nutanix cluster. What is the strongest recommendation?
**Cert relevance:** NCP-US · sales-relevant

A) Buy new NetApp; Files isn't ready for primary file workloads
B) Consolidate onto Nutanix Files: deploy a File Server with 3 FSVMs on the existing cluster, integrate with AD, configure quotas and snapshot schedules, train users on Self-Service Restore, decommission the NetApp at the end of its lifecycle
C) Buy a separate small filer for this department
D) Use Objects instead of Files because it's newer

**Answer:** B

**Why this answer:** This is the textbook unified-storage consolidation scenario. Department of 200 users with standard SMB workloads is squarely within Files' capability. Consolidation onto the existing cluster eliminates a future refresh cycle and simplifies management.

**Why not the others:**
- A) Files handles standard departmental file workloads cleanly.
- C) Buying separate small storage defeats the consolidation premise.
- D) Objects is for object workloads (S3), not file shares (SMB/NFS).

**The trap:** A reflects undue conservatism; D reflects misunderstanding of the protocol differences.

---

**Q11.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep · NCP-US prep · sales-relevant
**Format:** Short-answer / design discussion. Write your reasoning. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A customer's environment:
- 8 ESXi hosts, ~250 VMs
- A 200 TB NetApp filer (4 years old, support ends in 8 months) serving SMB shares for ~800 users plus application file storage for several Java/Tomcat apps
- A 100 TB Data Domain (5 years old) serving as Veeam backup target
- A 50 TB Pure FlashArray serving iSCSI to a small bare-metal Oracle environment
- Annual storage spend: roughly $400K across the three appliances (refresh amortization + support + power/space)

The customer is evaluating whether to consolidate onto a new Nutanix cluster. They want a 5-year cost projection, an architecture proposal, and a phased migration plan.

**The challenge:**
Walk through your design. Cover the consolidated cluster sizing, the Files / Objects / Volumes deployment, the backup repository transition, the iSCSI consumer migration, and the phased rollout. Identify what you still need to know.

**A strong answer covers:**
- **Consolidated cluster sizing.** Total capacity need: ~250 VMs (existing compute load) + 200 TB Files + 100 TB Objects + 50 TB Volumes + headroom for RF, reservation, and growth. Estimate: a 12-node Nutanix cluster sized for compute with all-NVMe nodes, accounting for the storage role consolidation. Verify networking is 25 or 100 GbE.
- **Files deployment.** Deploy a File Server with 3-5 FSVMs initially (scale-out as data grows). Integrate with customer's AD. Configure SMB shares to mirror existing NetApp share structure. Snapshot schedules per the customer's existing recovery requirements. Files Analytics for reporting and anti-ransomware.
- **Objects deployment.** Deploy Object Service and configure as a Veeam target. Migrate Veeam backup repositories from Data Domain to Objects in stages: start with non-critical workloads, validate, then move critical workloads. Lifecycle policies for retention.
- **Volumes deployment.** Configure Volume Groups for the Oracle bare-metal environment. Use multipath iSCSI for HA. Migrate Oracle data via standard Oracle techniques (RMAN restore from backup, or careful storage-level copy with downtime window).
- **Phased migration plan:**
  - Month 1-2: deploy Nutanix cluster, validate platform
  - Month 2-3: migrate Veeam backup target to Objects (lowest risk, highest immediate value)
  - Month 3-6: migrate file shares from NetApp to Files (use Robocopy or Files Migration tool); decommission NetApp at end of support
  - Month 6-9: migrate Oracle from Pure to Volumes (planned downtime window)
  - Month 9-12: validate, optimize, decommission Pure
- **5-year cost projection.** Run the math:
  - Status quo: $400K/year × 5 = $2M, plus three refresh cycles in the period
  - Nutanix: cluster capex (estimate $700-900K for the 12-node cluster) + 5 years of subscription + Files / Objects / Volumes licensing + reduced support overhead = typically $1.0-1.4M for 5 years
  - Net savings: $600K-$1M over 5 years, with operational simplification not quantified
- **What you still need to know:** specific application file-storage requirements (some apps may have ONTAP-specific dependencies), Oracle's licensing posture (per-CPU vs per-core for the Oracle hosts that will consume Volumes), Veeam's specific S3-target version compatibility, the customer's RPO/RTO requirements for the file workloads, growth forecast for each storage tier.

**A weak answer misses:**
- Defaulting to "rip and replace" without phased migration.
- Skipping the cost projection (the customer asked for it).
- Not naming the Veeam migration first (lowest-risk consolidation).
- Forgetting to plan the Oracle migration window (real downtime).
- Treating all three storage tiers as equivalent migration risks.

**Why this question matters for NCX:** This is the kind of multi-tier storage consolidation that NCX panels evaluate. Pure-feature answers fail. The right answer integrates platform architecture, migration sequencing, cost analysis, and risk assessment.

---

**Q12.** *(NCX-style architectural defense, open-ended)*
**Cert relevance:** NCX-MCI prep · NCP-US prep · sales-relevant
**Format:** Short-answer / architectural defense. Write your response.

**The scenario:**
You are in front of a customer's lead storage architect, who has been a NetApp NCDA for 12 years. He says:

> *"Files is fine for general SMB shares, but we have ONTAP-specific workflows: FlexClone for instant SQL test environments from production data, FlexCache for distributed branch-office reads, ABE refinements that took years to tune, custom SnapMirror policies, and integration with Veritas NetBackup that uses ONTAP-specific APIs. Walking away from this means rebuilding workflows that took us a decade. Why is Files the right answer for us?"*

**The challenge:**
Respond. He has named specific ONTAP capabilities. Address each.

**A strong answer covers:**
- **Acknowledge the ONTAP capability set is real and the workflow investment is significant.** This is a 12-year specialist. Pretending Files matches every ONTAP feature loses the conversation.
- **Reframe each capability:**
  - **FlexClone.** Files snapshots are space-efficient and instant; clone-equivalent workflows can use snapshot-based provisioning with some workflow adaptation. For SQL Server specifically, Nutanix has database-aware cloning patterns (NDB / Era for database lifecycle) that handle the test-from-prod use case. Map his specific FlexClone workflows; many translate, some require workflow change.
  - **FlexCache.** Distributed-branch caching is a real ONTAP capability. Files does not have a direct equivalent. For branch-office read patterns, alternatives include deploying small Nutanix Files clusters at branches with replication, or using a third-party WAN acceleration. Acknowledge this gap.
  - **ABE.** Files supports ABE; the years of refinement on his ONTAP ABE policies translate as workflow-import-and-test rather than feature-loss.
  - **Custom SnapMirror policies.** Files Replication has its own policy model. Migrating SnapMirror policies is migration work, not feature-loss; the destination capabilities are comparable for most use cases.
  - **NetBackup with ONTAP-specific APIs.** Some backup integrations use ONTAP-specific APIs (NDMP variants, snapshot integration). Veritas supports Files at the SMB/NFS protocol level; ONTAP-specific integrations would require switching to standard backup approaches. This is a real cost in some workflows.
- **The honest reframe:** "You have legitimate ONTAP investment. The question isn't whether to throw it away. The question is whether the consolidation benefits over 5 years justify the workflow migration costs. For workflows like FlexClone and SnapMirror, the cost is real but bounded; for FlexCache, the gap is real and may favor keeping NetApp for specific branch use cases. The right answer is workflow-by-workflow."
- **Offer a concrete next step:** "Let me build a workflow inventory with you. We mark each workflow as: (1) translates cleanly to Files, (2) translates with workflow change, (3) requires keeping NetApp for that workload. The map will tell us whether full consolidation makes sense or whether the right answer is hybrid (Files for the bulk, NetApp for the workflows that don't translate). Either path saves money compared to refreshing all NetApp; we just need to know which path is right for you."

**A weak answer misses:**
- Claiming Files matches every ONTAP feature.
- Dismissing the architect's 12 years of expertise.
- Not naming FlexCache as a real gap.
- Forcing a full migration when hybrid is the better answer.
- Not closing with the workflow-mapping exercise as a concrete proposal.

**Why this question matters for NCX:** Storage architects with deep ONTAP investment are common in enterprise customers. The disposition being tested is acknowledging the real expertise, naming the real gaps, and reframing to a workflow-by-workflow mapping rather than a binary migration decision.

---

## What You Now Have

You can articulate the unified storage consolidation pitch in 5 minutes on a whiteboard: NetApp + Cloudian + Pure-iSCSI + Data-Domain consolidate to one Nutanix cluster running Files + Objects + Volumes on shared DSF.

You know each service's purpose: Files for SMB/NFS file workloads, Objects for S3-compatible object storage (especially backup targets and cloud-native apps), Volumes for iSCSI block to non-Nutanix consumers.

You know each service's architecture: Files uses dedicated FSVMs; Objects uses Object Service VMs; Volumes is the cluster acting as iSCSI target. All three sit on DSF and inherit DSF's properties.

You know Files' operational features: Self-Service Restore, Files Analytics, anti-ransomware, multi-protocol shares, replication. You can demo SSR in 90 seconds.

You know Objects' compliance and lifecycle features: versioning, WORM, lifecycle policies, replication. You know which backup vendors support it (Veeam, Commvault, Rubrik, Cohesity, HYCU).

You know Volumes' use cases: physical hosts, bare-metal databases (Oracle RAC), legacy iSCSI consumers, ESXi clusters not on Nutanix that want to consume Nutanix storage.

You have the honest gap list: Files vs ONTAP at the high end (FlexClone, FlexCache, advanced policies); Objects vs AWS S3 ecosystem maturity; hyperscale workloads still cloud-native; specialized HPC filers still purpose-built.

You have the consolidation economics: typical mid-market deployments save 30-50% over 5 years compared to maintaining four separate storage tiers. The backup-target consolidation alone (Objects replacing Data Domain) is one of the easier ROI wins.

You have twelve practice questions worth of unified-storage discrimination, including two NCX-style design defenses (multi-tier storage consolidation with cost projection, and the architectural defense against a 12-year NetApp specialist).

You are now ready for licensing. Module 9 covers Nutanix licensing, NCM tiers, AOS subscription models, hardware vs software pricing, and the financial conversation that frequently decides deals. After all the technical depth, this is the dimension that gets things signed.

---

## Cross-References

- **Previous:** [Module 7: Data Protection and DR](./07-data-protection.md)
- **Next:** [Module 9: Licensing and Real Costs](./09-licensing-economics.md)
- **Glossary:** [Nutanix Files](./appendix-a-glossary.md#nutanix-files) · [FSVM](./appendix-a-glossary.md#fsvm) · [File Server](./appendix-a-glossary.md#file-server) · [Nutanix Objects](./appendix-a-glossary.md#nutanix-objects) · [Object Store](./appendix-a-glossary.md#object-store) · [Bucket](./appendix-a-glossary.md#bucket) · [Versioning](./appendix-a-glossary.md#versioning) · [WORM](./appendix-a-glossary.md#worm) · [Nutanix Volumes](./appendix-a-glossary.md#nutanix-volumes) · [Volume Group](./appendix-a-glossary.md#volume-group) · [SMB](./appendix-a-glossary.md#smb) · [NFS](./appendix-a-glossary.md#nfs) · [S3](./appendix-a-glossary.md#s3) · [iSCSI](./appendix-a-glossary.md#iscsi) · [Files Analytics](./appendix-a-glossary.md#files-analytics) · [Self-Service Restore](./appendix-a-glossary.md#self-service-restore)
- **Comparison Matrix:** [Files vs NetApp Row](./appendix-b-comparison-matrix.md#files-vs-netapp) · [Objects vs S3 Row](./appendix-b-comparison-matrix.md#objects-vs-s3) · [Volumes vs Block Arrays Row](./appendix-b-comparison-matrix.md#volumes-vs-block)
- **Objections:** [#31 "We have NetApp; why add Files?"](./appendix-d-objections.md#obj-031) · [#32 "Objects vs AWS S3"](./appendix-d-objections.md#obj-032) · [#33 "Backup-target consolidation"](./appendix-d-objections.md#obj-033) · [#34 "iSCSI consumers and Volumes"](./appendix-d-objections.md#obj-034)
- **Discovery Questions:** [Q-STOR-06 File workload inventory](./appendix-e-discovery-questions.md#q-stor-06) · [Q-STOR-07 Object/S3 use cases](./appendix-e-discovery-questions.md#q-stor-07) · [Q-STOR-08 iSCSI consumer inventory](./appendix-e-discovery-questions.md#q-stor-08) · [Q-STOR-09 Backup target architecture](./appendix-e-discovery-questions.md#q-stor-09)
- **Sizing Rules:** [Files sizing](./appendix-f-sizing-rules.md#files-sizing) · [Objects sizing](./appendix-f-sizing-rules.md#objects-sizing) · [Volumes sizing](./appendix-f-sizing-rules.md#volumes-sizing)
- **CLI Reference:** [Files CLI (`afs`)](./appendix-g-cli-reference.md#files-cli) · [Objects management](./appendix-g-cli-reference.md#objects-cli) · [`ncli` volume commands](./appendix-g-cli-reference.md#ncli-volume)
- **Competitive Matrix:** [Files vs NetApp ONTAP](./appendix-h-competitive-matrix.md#files-vs-ontap) · [Objects vs Cloudian/MinIO/StorageGRID](./appendix-h-competitive-matrix.md#objects-vs-competitors)
- **Reference Architectures:** [Backup-target consolidation](./appendix-i-reference-architectures.md#ra-backup-consolidation) · [File-services consolidation](./appendix-i-reference-architectures.md#ra-files-consolidation)
- **POC Playbook:** [Files SSR demo](./appendix-j-poc-playbook.md#files-ssr) · [Objects backup-target demo](./appendix-j-poc-playbook.md#objects-veeam)

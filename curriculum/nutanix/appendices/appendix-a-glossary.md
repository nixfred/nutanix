---
appendix: A
title: Glossary
type: reference
purpose: Alphabetical reference for every key term used across the curriculum.
usage: |
  Read entries on demand during customer calls or while studying. Each entry is
  self-contained. The "Module N" reference points to where the term is taught
  in depth. The "See also" cross-references walk you to related entries.
last_updated: 2026-04-30
covers_letters: A through M
---

# Appendix A: Glossary

The working vocabulary of a Nutanix Solutions Architect at BlueAlly. Every term used across the 10 modules is here. Definitions are tight and operationally relevant.

Use Ctrl-F or the site search. Letters N through Z continue in the second half of this appendix.

---

## A

### Acropolis

The service stack on AHV that handles VM lifecycle: create, clone, migrate, snapshot, plus HA (host failure response), Live Migration, and ADS (load balancing). Acropolis is what makes AHV more than just KVM. Functional analog to vCenter's VM-control responsibilities.

*Module 3 (AHV).*
*See also:* [AHV](#ahv), [Live Migration](#live-migration), [HA](#ha-high-availability), [ADS](#ads-acropolis-distributed-scheduler).

### ADS (Acropolis Dynamic Scheduling)

The AHV equivalent of VMware DRS. Continuously monitors cluster load and rebalances VMs across hosts. Default polling interval 15 minutes (with a 30-minute backoff after a rebalance). Runs automatically; admin-tunable thresholds. Triggered by CPU contention, memory pressure, or storage hotspots. The internal service that runs ADS is named **Lazan**; you will see the name in logs and `acli` output. (Older docs and the curriculum's earlier draft used "Distributed Scheduler"; the authoritative name is "Dynamic Scheduling.")

*Module 3 (AHV).*
*See also:* [Acropolis](#acropolis), [Live Migration](#live-migration).

### AHV (Acropolis Hypervisor)

Nutanix's KVM-based hypervisor. Included with AOS at no additional licensing cost. Runs on every Nutanix node by default. Operationally similar to ESXi for VM admins; technically a Linux+KVM stack underneath. Acropolis services provide the management plane.

*Module 3 (AHV).*
*See also:* [Acropolis](#acropolis), [KVM](#kvm), [ESXi-on-Nutanix](#esxi-on-nutanix).

### Anti-ransomware

Real-time ransomware detection capability in Nutanix Files (via on-cluster Files Analytics) and the broader Data Lens product. Detects suspicious write patterns (mass encryption, mass rename, suspicious extension changes) using a 65,000+ signature library plus behavior-based anomaly detection. Alerts, optionally blocks, and snapshots the pre-attack state. One layer in a broader anti-ransomware strategy (complement with endpoint protection, network segmentation, backup hygiene).

*Module 8 (Unified Storage).*
*See also:* [Files Analytics](#files-analytics), [Data Lens](#data-lens), [Nutanix Files](#nutanix-files).

### AOS (Acropolis Operating System)

The Nutanix platform's core software stack: AHV + DSF + Prism Element. The AOS *software* is unchanged; the *licensing SKU name* AOS has been replaced by **NCI (Nutanix Cloud Infrastructure)**. Legacy AOS Pro / AOS Ultimate licenses are no longer available for new sale or renewal; existing AOS customers are being converted to NCI Pro / NCI Ultimate. Per-core subscription. AHV is included at every tier at no extra fee.

*Module 9 (Licensing).*
*See also:* [NCI](#nci-nutanix-cloud-infrastructure), [NCI Pro](#nci-pro), [NCI Ultimate](#nci-ultimate), [DSF](#dsf-distributed-storage-fabric).

### AOS Pro

**Legacy** subscription tier name. Replaced by **NCI Pro**; no longer available for new sale or renewal. See [NCI Pro](#nci-pro).

### AOS Ultimate

**Legacy** subscription tier name. Replaced by **NCI Ultimate**; no longer available for new sale or renewal. See [NCI Ultimate](#nci-ultimate).

### Application affinity group

A logical grouping of VMs that should be considered together during migration planning, DR orchestration, or policy enforcement. Often defined by application architecture: web tier + app tier + database tier of a single application. Used in Recovery Plans for startup-order definition.

*Module 10 (Migration), Module 7 (DR).*
*See also:* [Recovery Plan](#recovery-plan-leap), [Categories](#categories).

### Application-consistent snapshot

Snapshot taken after the application has flushed in-memory state to disk. On Windows, requires VSS (provided by NGT). On Linux, application-level quiesce coordinated through NGT. Compare to crash-consistent (default), which captures disk state without app coordination. Preferred for direct database restoration.

*Module 7 (Data Protection).*
*See also:* [Crash-consistent snapshot](#crash-consistent-snapshot), [NGT](#ngt-nutanix-guest-tools), [VSS](#vss-volume-shadow-copy-service).

### Async Replication

Periodic snapshot-based replication between Nutanix clusters. RPO typically 1 hour, configurable down to 15 minutes. Bandwidth-efficient (delta-based). Works over WAN, no strict latency requirements. Default for general-purpose DR. Compare to NearSync (sub-15-min RPO, low-latency) and Metro (zero RPO, metro-area only).

*Module 7 (Data Protection).*
*See also:* [NearSync](#nearsync), [Metro Availability](#metro-availability), [RPO](#rpo-recovery-point-objective).

---

## B

### Bond

A Linux/OVS bonding interface aggregating multiple physical NICs into a single logical link. Provides redundancy and (some modes) load balancing. Modes: active-backup (default), balance-slb (no switch config required), balance-tcp / LACP (requires switch coordination). Maps to vSphere NIC teaming policies.

*Module 6 (Networking).*
*See also:* [Open vSwitch](#open-vswitch-ovs), [LACP](#lacp).

### BoM (Bill of Materials)

The complete list of items in a customer proposal. A complete BoM has six sections: hardware, software, services, support, one-time items, recurring items. Hardware-and-license-only quotes are not BoMs; they cause customer pain six months in. BlueAlly discipline: name every item.

*Module 9 (Licensing).*
*See also:* [TCO](#tco-total-cost-of-ownership).

### Bridge (br0, br0.local)

OVS bridges in AHV. `br0` is the data bridge connected to physical NICs (via a bond), carrying user VM traffic. `br0.local` is the management bridge for CVM-hypervisor communication. The two-bridge separation is intentional and operationally meaningful.

*Module 6 (Networking).*
*See also:* [Open vSwitch](#open-vswitch-ovs).

### Bucket

The unit of organization in Nutanix Objects (and S3 generally). A bucket holds objects and has access policies, versioning, lifecycle policies, and optionally WORM compliance settings. Backup software targets buckets; cloud-native applications read and write to buckets.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Objects](#nutanix-objects), [WORM](#worm), [S3](#s3).

---

## C

### Capex / Opex

Capex (capital expenditure) is hardware, depreciated over 3-5 years, balance-sheet item. Opex (operating expenditure) is subscriptions and services, expensed in period, income-statement impact. Customer accounting preferences vary; ask early. Nutanix subscriptions typically opex; hardware typically capex.

*Module 9 (Licensing).*
*See also:* [TCO](#tco-total-cost-of-ownership), [BoM](#bom-bill-of-materials).

### Cassandra

Distributed metadata store at the heart of DSF. A fork of Apache Cassandra optimized for the access patterns DSF needs. Tracks every extent's location, every vDisk's configuration, every cluster-wide piece of data placement. Runs in a ring across all CVMs. Stargate consults Cassandra for "where does extent X live?"

*Module 2 (Architecture), Module 5 (DSF).*
*See also:* [Stargate](#stargate), [Curator](#curator), [Extent](#extent).

### Categories

Key-value tags assigned to VMs (and other entities) in Prism Central. Drive policy enforcement: backup policies, DR plans, microsegmentation rules, quotas, reporting. More powerful than vSphere tags because they are first-class policy keys. The integration point for Flow, Protection Policies, and Self-Service automation.

*Module 4 (Prism), Module 6 (Networking), Module 7 (DR).*
*See also:* [Projects](#projects), [Protection Policy](#protection-policy), [Flow Network Security](#flow-network-security).

### Cluster Virtual IP (VIP)

The IP address that Prism Element listens on. Floats between CVMs for HA. Customers and admins access Prism Element at `https://<VIP>:9440`. Distinct from the Data Services IP (used for iSCSI target presentation by Volumes).

*Module 4 (Prism).*
*See also:* [Prism Element](#prism-element), [Data Services IP](#data-services-ip).

### Compression (DSF)

Inline compression applied during OpLog drain to Extent Store using **LZ4** (fast, modest ratios, latency-friendly). Cold-data and post-process compression uses **LZ4HC** (higher compression at higher CPU cost). Inline compression is selective: applies to sequential streams and large I/Os (>64K) to avoid impacting random write performance. Real-world ratios for mixed enterprise workloads: 1.5-2.5x. Configured per Storage Container. Quote ranges, not marketing peaks.

*Module 5 (DSF).*
*See also:* [DSF](#dsf-distributed-storage-fabric), [OpLog](#oplog), [Storage Container](#storage-container).

### Content Cache

The in-memory read cache in CVM RAM. Stargate caches recently-accessed extents here. Cache hits return immediately. The first stop in the read path before checking the local Extent Store or remote nodes.

*Module 5 (DSF).*
*See also:* [Stargate](#stargate), [Extent Store](#extent-store), [OpLog](#oplog).

### Crash-consistent snapshot

The default Nutanix snapshot type. Captures vDisk state at a moment without application-level quiesce. Equivalent to pulling the power cord and rebooting: file systems may need recovery, in-memory data is lost, transactions may roll back. Sufficient for most workloads.

*Module 7 (Data Protection).*
*See also:* [Application-consistent snapshot](#application-consistent-snapshot).

### Curator

The background scrubbing and rebalancing service in DSF. Runs on every CVM (one master, others followers). Periodic scans walk metadata to identify operations: re-replication after failures, EC conversion, ILM/tiering, compression/dedup post-processing, capacity reclamation. Does not appear in the synchronous data path.

*Module 2 (Architecture), Module 5 (DSF).*
*See also:* [Stargate](#stargate), [Cassandra](#cassandra), [ILM](#ilm-information-lifecycle-management).

### Cutover

The moment when a workload transitions from old platform to new during migration. For Move: source VM shut down, final delta synced, target VM started. Typical cutover downtime per VM: 5-30 minutes. Scheduled in maintenance windows. The brief moment after months of preparation.

*Module 10 (Migration).*
*See also:* [Move](#move), [Parallel-running](#parallel-running), [Pilot wave](#pilot-wave).

### CVM (Controller VM)

The Nutanix-managed VM that runs on every node, hosting the storage services (Stargate, Cassandra, Curator, Pithos, Zeus) plus management services. Receives I/O from co-located user VMs, replicates across the cluster, manages metadata. The "tax" of HCI: typically 8-16 vCPU and 32-64GB RAM per node consumed by the CVM. The price of a distributed storage layer running on the same hardware as your VMs.

*Module 2 (Architecture).*
*See also:* [Stargate](#stargate), [Cassandra](#cassandra), [Curator](#curator), [DSF](#dsf-distributed-storage-fabric).

---

## D

### Data Lens

Nutanix's cloud-based (and, with v2.0 GA in 2026, fully on-premises including air-gapped) data governance and ransomware-detection service for unified storage. Evolved from on-cluster Files Analytics into a broader product covering Files, Objects, and (increasingly) other unified-storage targets. Carries a 65,000+ ransomware signature library plus behavior-based anomaly detection. Detect-and-block flow watches for encrypt-at-write, mass-rename, and suspicious-extension patterns.

*Module 8 (Unified Storage).*
*See also:* [Files Analytics](#files-analytics), [Anti-ransomware](#anti-ransomware), [Nutanix Files](#nutanix-files).

### Data Services IP

The cluster-level IP used for external iSCSI initiators connecting to Volumes. Distinct from the Cluster VIP (which serves Prism Element). External hosts connect to the Data Services IP for iSCSI target discovery.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Volumes](#nutanix-volumes), [iSCSI](#iscsi).

### Deduplication

DSF capability to detect duplicate data blocks and store one copy with references. Two flavors: cache dedup (in-CVM-RAM, always on for compatible workloads) and on-disk dedup (per-Storage-Container, requires more CVM metadata). Most useful for VDI and similar high-data-similarity workloads. Wrong tool for low-uniqueness workloads.

*Module 5 (DSF).*
*See also:* [Compression](#compression-dsf), [DSF](#dsf-distributed-storage-fabric), [Storage Container](#storage-container).

### Dependency mapping

Phase 0 migration deliverable identifying which VMs talk to which other VMs, which applications depend on which services, where the firewall and load-balancer dependencies live. Active discovery tools, application owner workshops, network flow analysis. Always takes longer than budgeted.

*Module 10 (Migration).*
*See also:* [Pilot wave](#pilot-wave), [Risk register](#risk-register).

### Decommissioning

The process of removing old infrastructure after migration. Physical removal, contract termination, asset disposal, license reconciliation. Often the customer's responsibility but worth naming in the BoM. Real money in licenses you stop paying and rack space you reclaim.

*Module 10 (Migration), Module 9 (Licensing).*
*See also:* [Hybrid steady-state](#hybrid-steady-state), [BoM](#bom-bill-of-materials).

### DSF (Distributed Storage Fabric)

The Nutanix software-defined storage layer. Runs across all CVMs in a cluster. Pools local disks of every node, replicates writes (RF2/RF3), caches reads, compresses/dedupes/erasure-codes, tiers data, self-heals. Built on Stargate (data path), Cassandra (metadata), Curator (background work), with Pithos and Zeus completing the service set.

*Module 2 (Architecture), Module 5 (DSF Deep Dive).*
*See also:* [Stargate](#stargate), [Cassandra](#cassandra), [Curator](#curator), [RF](#rf-replication-factor), [EC-X](#ec-x-erasure-coding).

---

## E

### EC-X (Erasure Coding)

DSF's erasure coding implementation. Configurable per Storage Container. Common configs: 4+1 (1-failure tolerance, 25% overhead vs RF2's 100%, requires 5+ nodes); 4+2 (2-failure tolerance, 50% overhead vs RF3's 200%, requires 7+ nodes). Trade-off: capacity savings vs write amplification on small random writes. Best for cold/archive workloads; bad for OLTP.

*Module 5 (DSF).*
*See also:* [RF](#rf-replication-factor), [Storage Container](#storage-container), [Curator](#curator).

### ESXi-on-Nutanix

Running VMware ESXi as the hypervisor on Nutanix hardware, instead of AHV. The CVM still runs DSF; vCenter still manages ESXi; Prism manages the Nutanix-native features. The migration-friendly path: keep ESXi mental model, get DSF benefits, decide on AHV later. Common starting point for VMware-mature customers.

*Module 3 (AHV).*
*See also:* [AHV](#ahv), [Move](#move).

### Extent

The 1 MB metadata unit in DSF. Each extent has a Cassandra metadata record indicating where it physically lives. vDisks are logical entities backed by extents. Multiple extents can share an Extent Group (the 4 MB physical allocation unit).

*Module 5 (DSF).*
*See also:* [Extent Group](#extent-group), [vDisk](#vdisk), [Cassandra](#cassandra).

### Extent Group

The physical allocation unit on disk in DSF holding extents. **1 MB on non-deduplicated containers; 4 MB on deduplicated containers.** The unit Stargate writes to the Extent Store. Compression and erasure coding operate at this level.

*Module 5 (DSF).*
*See also:* [Extent](#extent), [Extent Store](#extent-store).

### Extent Store

The persistent backing storage on each node where DSF writes go after draining from OpLog. The "cold" tier in the data path (though on all-flash nodes everything is fast). Reads check Content Cache first, then local Extent Store, then remote.

*Module 5 (DSF).*
*See also:* [OpLog](#oplog), [Content Cache](#content-cache), [Stargate](#stargate).

---

## F

### File Server

The logical SMB/NFS service in Nutanix Files. Implemented as a cluster of FSVMs (typically 3+). Customers can have multiple File Servers per cluster (e.g., one for Production, one for Engineering). Integrates with AD via Kerberos and ACLs.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Files](#nutanix-files), [FSVM](#fsvm-file-server-vm).

### Files Analytics

The on-cluster analytics service that ships inside Nutanix Files. Provides file aging, top users by capacity and I/O, file type breakdown, anomaly detection (anti-ransomware foundation), permission auditing. The 3-minute customer demo that often unlocks tiering decisions. The broader Nutanix product that evolved from Files Analytics is **Data Lens** (cloud-based or, with v2.0 GA in 2026, fully on-prem); the two coexist but Data Lens has the wider scope.

*Module 8 (Unified Storage).*
*See also:* [Data Lens](#data-lens), [Nutanix Files](#nutanix-files), [Anti-ransomware](#anti-ransomware).

### Flow Network Security

Nutanix's distributed firewall and microsegmentation product. Category-driven policy (not IP-based). Stateful rules. Distributed enforcement at OVS flow-rule level on each AHV host. Licensed via **NCI Ultimate** or via the **Security Add-On for NCI Pro** (per usable TiB; bundles Flow microsegmentation with Data-at-Rest Encryption). Not an NCM tier feature. Functional comparison to NSX-T's distributed firewall.

*Module 6 (Networking).*
*See also:* [Microsegmentation](#microsegmentation), [Categories](#categories), [NSX-T](#nsx-t), [Flow Virtual Networking](#flow-virtual-networking-fvn), [NCI Ultimate](#nci-ultimate).

### Flow Virtual Networking (FVN)

Nutanix's overlay networking product. VPC-style virtual networks with internal routing, NAT, BGP integration, service insertion. Younger than NSX-T; less mature for advanced routing patterns; sufficient for most multi-tenant use cases. Increasingly capable in 2024-2026.

*Module 6 (Networking).*
*See also:* [Flow Network Security](#flow-network-security), [VPC](#vpc-virtual-private-cloud), [Service Insertion](#service-insertion).

### Foundation

Nutanix's bare-metal cluster bootstrapping tool. Takes a set of new nodes and turns them into a working Nutanix cluster: imaging hypervisor, configuring CVMs, forming the cluster, validating networking. Run once per new cluster.

*Module 2 (Architecture).*
*See also:* [LCM](#lcm-life-cycle-manager), [NCC](#ncc-nutanix-cluster-check).

### FSVM (File Server VM)

The dedicated VM running the file-services stack as part of a File Server cluster. Three FSVMs typically form a File Server (the logical share-serving entity). Real VMs running on the underlying Nutanix cluster, not abstractions.

*Module 8 (Unified Storage).*
*See also:* [File Server](#file-server), [Nutanix Files](#nutanix-files).

---

## H

### HA (High Availability)

Acropolis-driven automatic VM restart on surviving hosts when a host fails. The AHV equivalent of vSphere HA. Triggered by host loss; affected VMs restart on cluster nodes with capacity. No additional licensing; built into AOS. Typical recovery time: 30-90 seconds for VM restart after failure detection.

*Module 3 (AHV).*
*See also:* [Acropolis](#acropolis), [ADS](#ads-acropolis-distributed-scheduler).

### HCI (Hyperconverged Infrastructure)

Architecture that runs compute and software-defined storage on the same x86 nodes, eliminating the separate storage array. The category Nutanix invented commercially. Compare to three-tier (separate compute/storage/network) and CI (preconfigured but still tiered). HCI's wins: simplification, scaling unit, operational consolidation. HCI's gaps: storage-imbalanced workloads, compute-imbalanced workloads.

*Module 1 (HCI Foundations).*
*See also:* [DSF](#dsf-distributed-storage-fabric), [CVM](#cvm-controller-vm).

### HCIR (Hyperconverged Infrastructure Ready)

Commodity hardware certified to run Nutanix software. Hardware sourcing option for customers who buy servers separately and license Nutanix software (software-only deployment). Most flexible; multi-vendor support boundaries.

*Module 9 (Licensing).*
*See also:* [NX appliance](#nx-appliance), [OEM partner](#oem-partner).

### Hybrid steady-state

A successful end-state where 70-90% of workloads run on Nutanix and specific workloads remain on VMware for legitimate reasons (NSX-T routing complexity, NetApp ONTAP-specific workflows, vendor-certified-only-on-ESXi applications, regulatory constraints). Not a project failure; often the architecturally correct outcome.

*Module 10 (Migration).*
*See also:* [Cutover](#cutover), [Production wave](#production-wave).

---

## I

### ILM (Information Lifecycle Management)

DSF's automated data movement: promoting hot data to fast tiers, demoting cold to capacity tiers, migrating for locality after VM moves, rebalancing on cluster expansion. Driven by Curator scans. Continuous background work; not in the synchronous I/O path.

*Module 5 (DSF).*
*See also:* [Curator](#curator), [DSF](#dsf-distributed-storage-fabric).

### Intelligent Operations

The NCM Pro feature set covering capacity analytics, anomaly detection, what-if planning, runway analysis, advanced reporting. Aria Operations (vROps) functional equivalent. Available at NCM Pro tier and above.

*Module 4 (Prism), Module 9 (Licensing).*
*See also:* [NCM](#ncm-nutanix-cloud-manager), [Prism Central](#prism-central).

### IPAM (IP Address Management)

Optional per-virtual-network DHCP service in AHV. When enabled, the cluster acts as DHCP server for VMs on that network. Useful for self-contained tenant networks (test/dev, VDI floating pools). Typically disabled for production networks served by corporate DHCP.

*Module 6 (Networking).*
*See also:* [Virtual Network](#virtual-network).

### iSCSI

The block-storage protocol used by Nutanix Volumes to expose LUNs to external initiators (physical hosts, non-Nutanix VMs, bare-metal databases). Standard initiator software on consumers; Nutanix cluster acts as iSCSI target. Multi-pathing via multiple portal IPs.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Volumes](#nutanix-volumes), [Volume Group](#volume-group).

---

## K

### KVM (Kernel-based Virtual Machine)

The Linux kernel hypervisor underlying AHV. Open-source, widely used, well-understood. AHV is KVM with the Acropolis service stack on top providing the management plane. Customers comfortable with KVM transfer their mental model directly to AHV.

*Module 3 (AHV).*
*See also:* [AHV](#ahv), [Acropolis](#acropolis).

---

## L

### LACP (Link Aggregation Control Protocol)

The 802.3ad standard for link aggregation. Used in some AHV bond modes (active-active LACP, balance-tcp). Requires switch-side configuration. Provides the highest throughput and load-balancing options when properly configured.

*Module 6 (Networking).*
*See also:* [Bond](#bond), [Open vSwitch](#open-vswitch-ovs).

### LCM (Life Cycle Manager)

Prism's coordinated upgrade tool for AOS, AHV, BIOS, BMC, NIC firmware, drive firmware. One-click rolling upgrades that respect cluster availability. The integrated equivalent of vSphere LCM but with broader scope (firmware coordination across the stack).

*Module 2 (Architecture), Module 4 (Prism).*
*See also:* [Foundation](#foundation), [NCC](#ncc-nutanix-cluster-check).

### Live Migration

The AHV equivalent of VMware vMotion. Move a running VM between AHV hosts in a cluster with no perceptible downtime. Triggered manually or by ADS rebalancing. Memory pre-copy plus brief switchover.

*Module 3 (AHV).*
*See also:* [Acropolis](#acropolis), [ADS](#ads-acropolis-distributed-scheduler), [Move](#move).

### LWS (Light-Weight Snapshots)

The technical mechanism underlying NearSync replication. Frequent (sub-minute) micro-snapshots at the source cluster that flow continuously to the destination. Achieves 20-second-to-15-minute RPO. Higher cluster overhead than Async snapshots.

*Module 7 (Data Protection).*
*See also:* [NearSync](#nearsync), [Async Replication](#async-replication).

---

## M

### Metro Availability

Synchronous replication between two Nutanix clusters at metro-area distance. Zero RPO. Requires <5ms RTT between sites. Witness VM at a third site for split-brain protection. Active-standby (typical) or active-active. Highest cost mode; only viable within metro distance. For long-distance DR, layer Async or NearSync to a remote third site.

*Module 7 (Data Protection).*
*See also:* [Witness VM](#witness-vm), [Async Replication](#async-replication), [NearSync](#nearsync), [RPO](#rpo-recovery-point-objective).

### Microsegmentation

Network security pattern that enforces firewall rules between individual VMs (or VM groups) rather than just at network perimeters. Implemented by Flow Network Security via category-driven policies and OVS flow-rule enforcement. Blocks lateral movement (compromised Web tier reaching DB tier directly).

*Module 6 (Networking).*
*See also:* [Flow Network Security](#flow-network-security), [Categories](#categories).

### Move

Nutanix's cross-platform VM migration tool. Supports ESXi-to-AHV, Hyper-V-to-AHV, AWS-to-NC2, Azure-to-NC2, AHV-to-AHV. Initial replication + incremental sync + brief planned cutover (5-30 min per VM). Not vMotion (no zero-downtime). Each migration scheduled in maintenance windows.

*Module 3 (AHV), Module 10 (Migration).*
*See also:* [Cutover](#cutover), [Pilot wave](#pilot-wave), [Production wave](#production-wave).

### Multi-tenancy

The capability to run multiple isolated tenants on shared infrastructure. In Prism Central: Projects with quotas and RBAC. In Flow Virtual Networking: separate VPCs per tenant. In Objects: separate object stores per tenant. The integration of these gives Nutanix multi-tenant capabilities equivalent to dedicated infrastructure for each tenant.

*Module 4 (Prism), Module 6 (Networking), Module 8 (Unified Storage).*
*See also:* [Projects](#projects), [Flow Virtual Networking](#flow-virtual-networking-fvn).

---

*Letters N through Z continue in the second half of this glossary.*

[Continue to N-Z →](./appendix-a-glossary-nz.md)

---

## References

The glossary entries are derived from and verified against the per-module References sections in each curriculum module; see those for primary sources (Nutanix Bible, portal tech notes, product datasheets, Nutanix.dev, etc.). The most cross-cutting authorities used in this glossary:

- [Nutanix Bible](https://www.nutanixbible.com/). The single most useful Nutanix-architecture reference; cited from nearly every glossary entry.
- [Nutanix Cloud Platform Software Options](https://www.nutanix.com/products/cloud-platform/software-options). Authoritative source for the NCI / NCM / NCP licensing structure that replaced AOS Pro / Ultimate.
- [Nutanix Portal Tech Notes (TN series)](https://portal.nutanix.com/page/documents/solutions/list). TN-2027 (data protection), TN-2032 (data efficiency), TN-2041 (Files architecture), TN-2106 (Objects), TN-2094 (Flow) are referenced from many glossary entries.
- [Nutanix Developer Portal](https://www.nutanix.dev/). v4 API reference, SDKs, automation tooling.

---

## Cross-References

- **Modules:** Each entry's "Module N" reference links to the defining module.
- **See also:** Internal links walk between related glossary entries.
- **Comparison Matrix:** [Appendix B](./appendix-b-comparison-matrix.md) has feature-by-feature comparisons (Nutanix vs VMware, Nutanix vs NetApp, etc.).
- **Objections:** [Appendix D](./appendix-d-objections.md) has the response scripts for common customer pushbacks on these topics.

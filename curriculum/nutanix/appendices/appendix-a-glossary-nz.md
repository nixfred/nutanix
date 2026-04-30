---
appendix: A
title: Glossary (N-Z)
type: reference
purpose: Continuation of the alphabetical glossary, letters N through Z.
usage: Pair with the A-M file for the complete vocabulary.
last_updated: 2026-04-30
covers_letters: N through Z
---

# Appendix A: Glossary (N-Z)

Continuation of the curriculum vocabulary. See [A-M](./appendix-a-glossary.md) for the first half.

---

## N

### NC2 (Nutanix Cloud Clusters)

Nutanix software running on AWS or Azure bare-metal infrastructure. From the platform's perspective, an NC2 cluster looks like any other Nutanix cluster. Enables cloud DR without a second physical datacenter, cloud-burst capacity, hybrid-cloud parity. Cloud bare-metal pricing varies; egress fees apply.

*Module 7 (DR), Module 9 (Licensing).*
*See also:* [Async Replication](#async-replication), [NearSync](#nearsync).

### NCC (Nutanix Cluster Check)

The cluster's built-in health-check tool. Runs automated checks across hardware, software, configuration, performance. Run on demand (`ncc health_checks run_all`) or on schedule. The first place to look when something is wrong.

*Module 2 (Architecture).*
*See also:* [LCM](#lcm-life-cycle-manager), [Foundation](#foundation).

### NCM (Nutanix Cloud Manager)

The umbrella name for advanced multi-cluster management features that sit on top of AOS in Prism Central. Tier structure: Starter (with PC, baseline), Pro (Intelligent Operations / capacity analytics), Ultimate (Self-Service / X-Play / cost governance). Tier contents shift between releases.

*Module 4 (Prism), Module 9 (Licensing).*
*See also:* [Prism Central](#prism-central), [Intelligent Operations](#intelligent-operations), [Self-Service](#self-service-formerly-calm).

### NearSync

Nutanix's near-synchronous replication mode. Uses Light-Weight Snapshots (LWS) to achieve 20-second-to-15-minute RPO. Requires lower-latency networking than Async (typically <5ms RTT) and adds cluster overhead at the source. For Tier-1 production with sub-15-minute RPO requirements.

*Module 7 (Data Protection).*
*See also:* [LWS](#lws-light-weight-snapshots), [Async Replication](#async-replication), [Metro Availability](#metro-availability).

### NetApp

Major storage incumbent. Customers commonly run NetApp filers (ONTAP), often alongside their VMware compute. Nutanix Files competes with ONTAP for general SMB/NFS workloads; ONTAP retains advantages in specific advanced workflows (FlexClone, FlexCache, advanced quotas, mature snapshot policies). Coexistence is often the right answer for established NetApp shops.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Files](#nutanix-files).

### NFS (Network File System)

Linux-native file-sharing protocol. Supported by Nutanix Files (v3 and v4). Used by Linux clients and applications, NFS-aware backup tools, and some hypervisor-level integrations.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Files](#nutanix-files), [SMB](#smb-server-message-block).

### NGT (Nutanix Guest Tools)

Guest-OS agent installed in VMs running on Nutanix. Enables application-consistent snapshots (VSS coordination on Windows, application quiesce on Linux), self-service file restore, and VM mobility between hypervisors. The Nutanix equivalent of VMware Tools.

*Module 3 (AHV), Module 7 (DR).*
*See also:* [Application-consistent snapshot](#application-consistent-snapshot), [VSS](#vss-volume-shadow-copy-service).

### NSX-T

VMware's overlay networking and security product. Comparison anchor for Flow Network Security (microsegmentation) and Flow Virtual Networking (overlays). NSX-T is more mature for advanced routing patterns, edge services, L2VPN, and third-party ecosystem. Coexistence pattern (NSX-T-on-Nutanix-on-ESXi) is common for established NSX-T customers.

*Module 6 (Networking).*
*See also:* [Flow Network Security](#flow-network-security), [Flow Virtual Networking](#flow-virtual-networking-fvn).

### Nutanix Files

Scale-out SMB/NFS file storage running on Nutanix. Implemented as a cluster of FSVMs on top of DSF. Replaces dedicated filers (NetApp, Isilon, Pure FlashBlade) for typical enterprise workloads. Includes Files Analytics, anti-ransomware, Self-Service Restore.

*Module 8 (Unified Storage).*
*See also:* [FSVM](#fsvm-file-server-vm), [File Server](#file-server), [Files Analytics](#files-analytics), [Self-Service Restore](#self-service-restore-ssr).

### Nutanix Objects

S3-compatible object storage running on Nutanix. Implemented as Object Service VMs on top of DSF. Replaces dedicated object storage appliances (Cloudian, Scality, MinIO, NetApp StorageGRID). Common backup-target consolidation: replaces Data Domain or similar for Veeam/Commvault/Rubrik repositories.

*Module 8 (Unified Storage).*
*See also:* [Bucket](#bucket), [WORM](#worm), [S3](#s3).

### Nutanix Volumes

iSCSI block storage service. Exposes LUNs (Virtual Volumes) over iSCSI to external initiators: physical Linux/Windows servers, bare-metal databases (Oracle RAC), legacy iSCSI consumers. Not for AHV VMs (use vDisks instead).

*Module 8 (Unified Storage).*
*See also:* [Volume Group](#volume-group), [iSCSI](#iscsi), [Data Services IP](#data-services-ip).

### NX appliance

Nutanix-branded hardware. Manufactured by Super Micro under the Nutanix brand. Single-vendor support: Nutanix is the throat to choke for both hardware and software. Tightly validated. The choice for customers wanting one vendor relationship.

*Module 9 (Licensing).*
*See also:* [OEM partner](#oem-partner), [HCIR](#hcir-hyperconverged-infrastructure-ready).

---

## O

### Object Store

A multi-tenant container in Nutanix Objects. Holds buckets, has IAM-style users and access policies. Multiple Object Stores enable hard tenant isolation in service-provider scenarios.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Objects](#nutanix-objects), [Bucket](#bucket).

### OEM partner

Hardware vendors that sell Nutanix-validated hardware: Dell EMC XC, Lenovo HX, HPE DX, Cisco UCS. Customers preserve their existing server-vendor relationship while running Nutanix software. Joint Nutanix + OEM support model. Common path for established server-vendor preferences.

*Module 9 (Licensing).*
*See also:* [NX appliance](#nx-appliance), [HCIR](#hcir-hyperconverged-infrastructure-ready).

### Open vSwitch (OVS)

The open-source kernel-level software switch in AHV. Same OVS used in OpenStack and many production Linux platforms. Provides the bridge, bonding, and VLAN functionality. Auditable, well-understood, not proprietary.

*Module 6 (Networking).*
*See also:* [Bridge](#bridge-br0-br0local), [Bond](#bond), [Flow Network Security](#flow-network-security).

### OpLog

The persistent write buffer in DSF. On each node's hot tier (NVMe/SSD). Receives writes from Stargate, replicates to peer node OpLogs (RF2 or RF3), acknowledges to VM after durable. Drains asynchronously to Extent Store. The source of DSF's low-latency write characteristic.

*Module 5 (DSF).*
*See also:* [Stargate](#stargate), [Extent Store](#extent-store), [RF](#rf-replication-factor).

---

## P

### Parallel-running

Period during migration when source and target platforms both operate. Common for risk-managed cutovers (run new platform alongside old, validate, switch). Costs both platforms' operational expenses simultaneously. Plan 1-3 months for typical migrations; longer for complex environments.

*Module 10 (Migration).*
*See also:* [Cutover](#cutover), [Hybrid steady-state](#hybrid-steady-state).

### Per-core licensing

Subscription model where customers pay per CPU core in their cluster (not per-VM, not per-CPU socket). Used by AOS, NCM, and post-Broadcom VMware. Multi-year terms with discount tiers. True-up provisions for mid-term core additions.

*Module 9 (Licensing).*
*See also:* [True-up](#true-up), [VCF](#vcf-vmware-cloud-foundation).

### Pilot wave

The first migration wave: 5-20 low-risk VMs (typically internal IT-team-owned, low criticality, representative of common patterns). Validates platform, tooling, processes. Establishes runbook for subsequent waves. Skipping pilot is the common project-failure pattern.

*Module 10 (Migration).*
*See also:* [Production wave](#production-wave), [Move](#move).

### Pithos

The DSF service that owns vDisk configuration metadata. Tracks which vDisks exist, their attributes, their associations with VMs and Storage Containers. Smaller than Cassandra but architecturally distinct.

*Module 2 (Architecture), Module 5 (DSF).*
*See also:* [Cassandra](#cassandra), [vDisk](#vdisk).

### Prism

The umbrella name for the Nutanix management plane. Two products: Prism Element (per-cluster) and Prism Central (multi-cluster). Replaces vCenter, vSphere LCM, Aria Operations, and Aria Automation in functional scope, integrated into one management surface.

*Module 4 (Prism).*
*See also:* [Prism Element](#prism-element), [Prism Central](#prism-central), [NCM](#ncm-nutanix-cloud-manager).

### Prism Central (PC)

The multi-cluster Nutanix management product. Deployed as a VM (single or scale-out three-VM). Aggregates multiple Prism Element clusters. Hosts Categories, Projects, Self-Service, X-Play, Intelligent Operations. The platform-wide management surface. Required for advanced features beyond per-cluster operations.

*Module 4 (Prism).*
*See also:* [Prism Element](#prism-element), [Categories](#categories), [Projects](#projects), [NCM](#ncm-nutanix-cloud-manager).

### Prism Element (PE)

The per-cluster Nutanix management UI. Runs in-cluster (on the CVMs), no separate appliance to deploy. Accessible at the Cluster VIP on port 9440. Handles VM lifecycle, host management, storage configuration, networking, monitoring for that cluster.

*Module 4 (Prism).*
*See also:* [Prism Central](#prism-central), [Cluster Virtual IP](#cluster-virtual-ip-vip).

### Production wave

A migration wave moving production workloads. Three typical waves: Wave 1 (low-tier, Tier-2/Tier-3, 50-200 VMs), Wave 2 (Tier-1, 200-600 VMs), Wave 3 (mission-critical, Tier-0, smallest count, highest scrutiny). Risk-sequenced to build confidence wave by wave.

*Module 10 (Migration).*
*See also:* [Pilot wave](#pilot-wave), [Cutover](#cutover).

### Projects

First-class multi-tenancy construct in Prism Central. Group of VMs, networks, images, and quotas with assigned users and roles. Richer than vCenter's resource pools. Used for environment separation (Dev/Test/Prod), business-unit isolation, service-provider tenancy.

*Module 4 (Prism).*
*See also:* [Categories](#categories), [Multi-tenancy](#multi-tenancy).

### Protection Domain (PD)

Legacy DSF construct for protecting groups of VMs/vDisks together. Configured in Prism Element. Manual VM membership. Single replication and snapshot schedule per PD. Continues to work; superseded by Protection Policies for new deployments.

*Module 7 (Data Protection).*
*See also:* [Protection Policy](#protection-policy).

### Protection Policy

Modern protection construct in Prism Central. Category-driven membership (auto-enrollment). Multiple schedules. Tied to Recovery Plans for orchestrated failover. The recommended protection construct for new deployments.

*Module 7 (Data Protection).*
*See also:* [Protection Domain](#protection-domain-pd), [Recovery Plan](#recovery-plan-leap), [Categories](#categories).

---

## R

### Recovery Plan (Leap)

The Nutanix DR orchestration product in Prism Central. Defines what VMs are protected (via category match), startup order (groups), network mapping, IP remapping, pre/post checks, test failover capability. The SRM functional equivalent on Nutanix. Available with appropriate NCM tier licensing.

*Module 7 (Data Protection).*
*See also:* [SRM](#srm-site-recovery-manager), [Protection Policy](#protection-policy), [Async Replication](#async-replication).

### RF (Replication Factor)

The per-Storage-Container setting determining how many copies of every write DSF stores. RF2 (two copies, 50% effective capacity, single-failure tolerance), RF3 (three copies, 33% effective capacity, two-failure tolerance). Most workloads run RF2 with backup; mission-critical or compliance-driven workloads may run RF3.

*Module 5 (DSF).*
*See also:* [Storage Container](#storage-container), [EC-X](#ec-x-erasure-coding).

### Risk register

Living document tracking identified migration risks, mitigations, ownership, and status. Reviewed weekly with project lead, monthly with executive sponsor. New risks added as discovered. Categories: technical, operational, commercial, relationship.

*Module 10 (Migration).*
*See also:* [Pilot wave](#pilot-wave), [Dependency mapping](#dependency-mapping).

### RPO (Recovery Point Objective)

The maximum acceptable data loss measured in time. RPO 1-hour means up to 1 hour of data may be lost in a disaster. Drives replication mode choice: zero RPO (Metro Availability), sub-15-minute (NearSync), 1-hour-plus (Async).

*Module 7 (Data Protection).*
*See also:* [RTO](#rto-recovery-time-objective), [Async Replication](#async-replication), [NearSync](#nearsync), [Metro Availability](#metro-availability).

### RTO (Recovery Time Objective)

The maximum acceptable downtime during recovery. RTO 30-minute means service must be restored within 30 minutes of disaster. Drives orchestration design: Recovery Plan automation, startup order, validation checks. Works alongside RPO; the two together define DR requirements.

*Module 7 (Data Protection).*
*See also:* [RPO](#rpo-recovery-point-objective), [Recovery Plan](#recovery-plan-leap).

---

## S

### S3

The de-facto standard object storage API, originated by AWS. Nutanix Objects is S3-compatible: standard S3 endpoints, signatures, requests. Tools that work with AWS S3 work with Objects with endpoint-only changes (Veeam, Commvault, custom applications, AWS SDKs).

*Module 8 (Unified Storage).*
*See also:* [Nutanix Objects](#nutanix-objects), [Bucket](#bucket).

### SAML

Security Assertion Markup Language, the standard for identity federation. Prism Central supports SAML 2.0 for SSO with major identity providers (Azure AD/Entra ID, Okta, Ping, etc.). Configured once in Prism Central; applies across the platform.

*Module 4 (Prism).*
*See also:* [Prism Central](#prism-central).

### Self-Service (formerly Calm)

The application blueprint and provisioning automation in NCM Ultimate. Define application stacks as blueprints with parameters; users self-provision through a service catalog. Aria Automation (vRA) functional equivalent. Brand renamed from "Calm" to "Self-Service."

*Module 4 (Prism), Module 9 (Licensing).*
*See also:* [NCM](#ncm-nutanix-cloud-manager), [X-Play](#x-play).

### Self-Service Restore (SSR)

The end-user-facing recovery feature in Nutanix Files. Snapshots exposed via the Windows "Previous Versions" tab. Users restore deleted files or earlier versions without IT involvement. Eliminates routine help-desk tickets for file recovery.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Files](#nutanix-files), [Application-consistent snapshot](#application-consistent-snapshot).

### Service Insertion

The Flow Virtual Networking capability to insert third-party network services (firewalls, load balancers, IDS/IPS) into traffic flow. Common pattern: Palo Alto VM-Series, Check Point, Fortinet inserted for deep packet inspection while Flow Network Security handles VM-tier microsegmentation.

*Module 6 (Networking).*
*See also:* [Flow Virtual Networking](#flow-virtual-networking-fvn), [Flow Network Security](#flow-network-security).

### SMB (Server Message Block)

The Windows-native file-sharing protocol. Nutanix Files supports SMB 2.x and 3.x with full Active Directory integration (Kerberos, ACLs, ABE, DFS-N).

*Module 8 (Unified Storage).*
*See also:* [Nutanix Files](#nutanix-files), [NFS](#nfs-network-file-system).

### SRM (Site Recovery Manager)

VMware's DR orchestration product. Comparison anchor for Recovery Plans. SRM has 15+ years of maturity with deep VMware integration. Recovery Plans is younger, integrated, and capable for typical use. Coexistence pattern (SRM on ESXi-on-Nutanix) is common for established SRM deployments.

*Module 7 (Data Protection).*
*See also:* [Recovery Plan](#recovery-plan-leap), [ESXi-on-Nutanix](#esxi-on-nutanix).

### Stargate

The DSF data-path service. Runs on every CVM. Receives I/O from local hypervisor, writes to local OpLog, replicates to peer OpLogs (RF2/RF3), acknowledges to VM. Manages reads from Content Cache, Extent Store, or remote nodes. The "controller" of the distributed storage layer.

*Module 2 (Architecture), Module 5 (DSF).*
*See also:* [Cassandra](#cassandra), [Curator](#curator), [OpLog](#oplog), [Content Cache](#content-cache).

### Storage Container

The logical "datastore equivalent" carved from the Storage Pool. The policy boundary in DSF: RF, compression, deduplication, erasure coding, advertised capacity, reservations are configured per-container. A cluster typically has multiple containers with different policies.

*Module 5 (DSF).*
*See also:* [Storage Pool](#storage-pool), [vDisk](#vdisk), [RF](#rf-replication-factor).

### Storage Pool

The aggregate of all physical disks in a cluster. Each cluster has one Storage Pool by default. Storage Containers are carved from the pool. Almost never manipulated directly.

*Module 5 (DSF).*
*See also:* [Storage Container](#storage-container), [DSF](#dsf-distributed-storage-fabric).

---

## T

### TCO (Total Cost of Ownership)

The complete cost of an infrastructure decision over a time horizon (typically 5 years). Categories: hardware, software, services, support, training, migration, operations, decommissioning. A real TCO has all categories, year-by-year breakdown, stated assumptions, and sensitivity analysis. CFOs evaluate models on this standard.

*Module 9 (Licensing).*
*See also:* [BoM](#bom-bill-of-materials), [Capex / Opex](#capex--opex).

### True-up

Contractual provision in multi-year subscriptions allowing customers to add cores during the term at agreed pricing. Protects customers from being locked at year-1 sizing for the entire term. Always negotiate true-up clauses for multi-year deals.

*Module 9 (Licensing).*
*See also:* [Per-core licensing](#per-core-licensing).

---

## V

### v4 API

The unified Nutanix REST API. JSON-based. Single base URL, single authentication, consistent schema across compute, storage, networking, automation. Supports PowerShell module, Python SDK, Terraform provider, Ansible collection, Pulumi provider. The automation foundation; first-class through every Nutanix feature.

*Module 4 (Prism).*
*See also:* [Prism Central](#prism-central), [X-Play](#x-play).

### vDisk

The virtual disk attached to a VM in Nutanix. Logical entity backed by extents (1 MB units). Cassandra tracks every extent's location. From the VM's perspective, vDisk looks like a SCSI or virtio block device.

*Module 5 (DSF).*
*See also:* [Extent](#extent), [Extent Group](#extent-group), [Cassandra](#cassandra).

### VCF (VMware Cloud Foundation)

VMware's bundled subscription including vSphere, vSAN, NSX-T, and Aria. Post-Broadcom licensing tier that customers must often buy to get vSphere. Per-core subscription. Comparison anchor for the Broadcom math.

*Module 9 (Licensing).*
*See also:* [VVF](#vvf-vmware-vsphere-foundation), [Per-core licensing](#per-core-licensing), [NSX-T](#nsx-t).

### Virtual Network

The Nutanix equivalent of a vSphere port group. Carries a VLAN tag (or none), optional IPAM, connection to a bridge (typically `br0`). VMs attach to virtual networks. Configured in Prism Element or Prism Central.

*Module 6 (Networking).*
*See also:* [VLAN](#vlan), [IPAM](#ipam-ip-address-management), [Bridge](#bridge-br0-br0local).

### VLAN

Standard 802.1Q virtual LAN tagging. Configured per Nutanix Virtual Network. AHV's OVS handles the tagging on `br0`. Inter-VLAN traffic exits via the bond uplink to the physical network for routing.

*Module 6 (Networking).*
*See also:* [Virtual Network](#virtual-network), [Open vSwitch](#open-vswitch-ovs).

### Volume Group

The management unit in Nutanix Volumes. Logical grouping of one or more LUNs presented together over iSCSI. Useful for application-aware grouping (e.g., all the disks for a SQL Server cluster instance).

*Module 8 (Unified Storage).*
*See also:* [Nutanix Volumes](#nutanix-volumes), [iSCSI](#iscsi).

### VPC (Virtual Private Cloud)

Isolated multi-subnet network environment with its own IP space and policies. Provided by Flow Virtual Networking. Concept similar to AWS VPC. Multiple VPCs per cluster enable hard tenant isolation; routing rules enable controlled inter-VPC communication.

*Module 6 (Networking).*
*See also:* [Flow Virtual Networking](#flow-virtual-networking-fvn), [Multi-tenancy](#multi-tenancy).

### VSS (Volume Shadow Copy Service)

Microsoft Windows feature for application-consistent snapshots. Coordinates application quiesce (databases, Exchange, AD) so the snapshot captures a consistent state. NGT provides the integration; Nutanix snapshots leverage VSS when application-consistent mode is requested.

*Module 7 (Data Protection).*
*See also:* [Application-consistent snapshot](#application-consistent-snapshot), [NGT](#ngt-nutanix-guest-tools).

### VVF (VMware vSphere Foundation)

VMware subscription tier including vSphere and vSAN. The level below VCF (which adds NSX-T and Aria). Post-Broadcom packaging change. Per-core subscription.

*Module 9 (Licensing).*
*See also:* [VCF](#vcf-vmware-cloud-foundation), [Per-core licensing](#per-core-licensing).

---

## W

### Witness VM

Small VM at a third site that provides quorum during partition events for Metro Availability deployments. When the two metro sites lose connectivity, the witness arbitrates which site continues operation. Critical for Metro deployments; design for resilience.

*Module 7 (Data Protection).*
*See also:* [Metro Availability](#metro-availability).

### WORM (Write Once Read Many)

Immutability feature in Nutanix Objects. Once written, objects cannot be modified or deleted until the retention period expires. Used for compliance archives: financial records, healthcare data, legal archives. Regulatory-grade protection.

*Module 8 (Unified Storage).*
*See also:* [Nutanix Objects](#nutanix-objects), [Bucket](#bucket).

---

## X

### X-Play

Event-driven automation engine in Prism Central (NCM Ultimate tier typically). Playbooks have triggers (alerts, events, schedules) and actions (notifications, API calls, snapshots, external scripts). Common patterns: alert-to-Slack, threshold-to-snapshot, event-to-ServiceNow ticket.

*Module 4 (Prism).*
*See also:* [Self-Service](#self-service-formerly-calm), [v4 API](#v4-api), [NCM](#ncm-nutanix-cloud-manager).

---

## Z

### Zeus

The cluster-consensus service in DSF. Uses Apache Zookeeper underneath. Maintains cluster state, coordinates cluster-wide decisions (leader election, configuration consensus). Lightweight but architecturally critical.

*Module 2 (Architecture).*
*See also:* [Cassandra](#cassandra), [Curator](#curator).

---

[← Back to A-M](./appendix-a-glossary.md)

---

## Cross-References

- **Modules:** Each entry's "Module N" reference links to the defining module.
- **See also:** Internal links walk between related glossary entries.
- **Comparison Matrix:** [Appendix B](./appendix-b-comparison-matrix.md) has feature-by-feature comparisons.
- **Objections:** [Appendix D](./appendix-d-objections.md) has response scripts for customer pushbacks on these topics.
- **CLI Reference:** [Appendix G](./appendix-g-cli-reference.md) has command syntax for the operational terms (`ncli`, `acli`, `manage_ovs`, etc.).

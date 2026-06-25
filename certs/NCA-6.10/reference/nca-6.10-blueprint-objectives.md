# NCA 6.10: Blueprint Objective Map

Nutanix Certified Associate (NCA) 6.10. This is the table of contents for study:
the four exam sections, what each covers, and the curriculum module that teaches
it. Every fact in the study guides and the High-Yield card was verified against
Nutanix documentation. **Always confirm the official blueprint at
[university.nutanix.com](https://university.nutanix.com) before exam day**: Nutanix
revises blueprints and the weightings shift between versions.

## Exam at a glance

| | |
|---|---|
| Exam | Nutanix Certified Associate (NCA) 6.10 |
| Questions | 50 (multiple choice + multiple response) |
| Time | 90 minutes |
| Passing score | 3000 (scaled 1000-6000) |
| Cost | $99 USD (discounts often available via Nutanix University) |
| Experience | ~6-12 months IT infrastructure, preferably with Nutanix |
| Platform baseline | AOS 6.10 · Prism Central pc2024.2 |
| Delivery | Online proctored or Pearson VUE test center |
| Validity | 3 years |
| Recommended free training | Nutanix Hybrid Cloud Fundamentals (NHCF); Enterprise Cloud Administration (ECA) |

## Section 1: Describe Lifecycle Management

Updating cluster software and firmware with **LCM (Life Cycle Manager)**.

- What LCM is: the unified one-click updater for **both software and firmware**, with dependency resolution and sequencing.
- **Inventory first**, then review available updates, then update.
- What LCM updates: AOS, Foundation, Prism, NCC/Licensing, plus firmware (BIOS/BMC, disk, HBA, NIC) on Nutanix NX and qualified OEMs (Dell, HPE, Cisco, Lenovo, Fujitsu) and NC2.
- **Dark-site** updates: host the LCM dark-site bundle on a local web server.

→ Teaches in: [Module 4: Prism Management](../../../modules/04-prism-management/index.html). Study guide: [01-lifecycle-management](../study-guides/01-lifecycle-management.html).

## Section 2: Describe Nutanix Basic Administration

VM management, cluster management, and licensing.

- **AHV** is the built-in, KVM-based hypervisor, included free in NCI.
- **Live Migration** (planned, no downtime) vs **HA** (restart after host failure) vs **ADS** (automatic load balancing).
- **Prism Element** (built into AOS, one cluster) vs **Prism Central** (separate VM, many clusters).
- **AOS licensing tiers**: Starter, Pro, Ultimate: align use case to tier.

→ Teaches in: [Module 3: AHV](../../../modules/03-ahv-hypervisor/index.html), [Module 4: Prism](../../../modules/04-prism-management/index.html), [Module 9: Licensing](../../../modules/09-licensing-economics/index.html). Study guide: [02-basic-administration](../study-guides/02-basic-administration.html).

## Section 3: Maintain Environmental Health

Where to find statistics, errors, and status.

- **Health dashboard** for at-a-glance status and data resiliency.
- **Health checks (NCC)** are scheduled; **alerts** are event-driven.
- **NCC (Nutanix Cluster Check)** is the health-check framework.
- **Pulse** sends scheduled telemetry to Nutanix for proactive support; it feeds Insights.
- **Support Portal**: downloads, license management, support cases, KB.

→ Teaches in: [Module 4: Prism](../../../modules/04-prism-management/index.html), [Module 2: Architecture](../../../modules/02-nutanix-architecture/index.html). Study guide: [03-environmental-health](../study-guides/03-environmental-health.html).

## Section 4: Describe Cluster Configuration Options

The densest section.

- **Storage components**: storage pool (raw disks) → container (policy: RF, compression, dedup, EC) → vDisk. Served by the CVM / DSF.
- **Replication Factor / Redundancy Factor**: RF2/FT1 (3 nodes, tolerate 1 failure) vs RF3/FT2 (5 nodes, tolerate 2). Default RF2/FT1.
- **Storage optimization**: compression (default), deduplication (VDI/clones), erasure coding EC-X (cold data, post-process >7 days).
- **Supported hypervisors**: AHV, ESXi, Hyper-V. **NC2**: full stack on AWS/Azure bare metal.
- **Cluster sizes**: 3-node standard; 2-node ROBO (needs a Witness VM); 1-node ROBO (selected HW).
- **Networking**: AHV uses Open vSwitch (OVS), default bridge br0 / bond br0-up; CVM+host on native VLAN; bonds ≥2 NICs.
- **Foundation** images bare-metal nodes into a cluster (~3 IPs per node + cluster IP + Data Services IP).

→ Teaches in: [Module 1: HCI Foundations](../../../modules/01-hci-foundations/index.html), [Module 2: Architecture](../../../modules/02-nutanix-architecture/index.html), [Module 5: DSF Storage](../../../modules/05-dsf-storage/index.html), [Module 6: Networking](../../../modules/06-networking-flow/index.html). Study guide: [04-cluster-configuration](../study-guides/04-cluster-configuration.html).

## Sources

Verified against the official **Nutanix NCA 6.10 Exam Blueprint Guide** plus Nutanix
documentation: AOS Storage / Data Efficiency (compression, deduplication, erasure
coding), Redundancy Factor vs Replication Factor, Prism architecture (Element vs
Central), Life Cycle Manager Guide (inventory, dark site, update specifications),
Nutanix Cluster Check (NCC) and Pulse / Insights, Data Protection with Async DR and
NearSync, AHV Networking best practices, AOS Software Options / Licensing, Foundation
Field Installation Guide, and Nutanix Cloud Clusters (NC2) on AWS/Azure.

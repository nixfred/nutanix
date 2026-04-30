---
appendix: I
title: Reference Architectures
type: reference
purpose: |
  Fully-sized reference designs for the most common BlueAlly deployment patterns.
  Each architecture has a profile (target customer), sizing, hardware
  recommendations, network topology, software stack, design rationale, and the
  trade-offs the architecture explicitly accepts.
usage: |
  Use as a starting point for proposal architecture. Adjust to the customer's
  specific situation. The reference architectures are calibrated to typical
  patterns; the actual proposal architecture comes from Sizer plus customer
  requirements.
discipline: |
  Three principles:
  1. Reference architectures are starting points, not deliverables. Customize
     per customer.
  2. Always run Sizer for the binding numbers in the proposal.
  3. Name the trade-offs explicitly. Every architecture sacrifices something.
last_updated: 2026-04-30
covers:
  - Small (mid-market mixed workloads, single site)
  - Medium (enterprise consolidation, single site with DR)
  - Large (multi-site enterprise, mixed compute and storage)
  - VDI (Citrix-driven, healthcare or financial profile)
  - Hybrid Nutanix + NC2 (cloud DR-extended)
  - ROBO fleet (retail / distributed sites at scale)
  - Greenfield startup (cost-sensitive, growth-oriented)
  - Compliance-heavy (financial services with HSM, audit, encryption)
---

# Appendix I: Reference Architectures

Eight reference architectures covering the most common BlueAlly deployment patterns. Each is a starting point; customize per customer. Always run Sizer for the binding numbers in the proposal.

The format for each architecture:

- **Customer profile** the architecture targets
- **Sizing** with cluster shape, hardware recommendations
- **Network topology** with switch and link recommendations
- **Software stack** with AOS / NCM / Files / Objects / Volumes / Flow choices
- **DR approach**
- **Design rationale** explaining why this shape
- **Trade-offs accepted** naming what this architecture chooses not to do
- **Variations** for common customer requests
- **Cross-references** to scenarios, comparison points, sizing rules

---

## 1. Small Reference Architecture

### Profile

Mid-market, ~150-300 employees in IT-relevant roles, single primary datacenter, secondary site for DR (often a colo or smaller office).

- VM count: 100-250
- Storage: 30-100 TB usable
- Workloads: general-purpose VMs, file shares, perhaps a small VDI deployment
- Compliance: standard (no PCI Level 1, no FedRAMP)
- Team: 3-6 person infrastructure team
- Annual run-rate today: $150-400K across hardware, software, support

### Sizing

**Production cluster:**

- 6-8 nodes, Dell XC or HPE DX (OEM partner) for established hardware-vendor relationships, or NX appliances for single-vendor preference
- All-NVMe configuration (768 GB-1 TB RAM, 32-48 cores per node)
- Storage capacity: 60-120 TB raw, providing 30-60 TB usable after RF2 + compression + headroom
- 25 GbE networking, 4 NICs per node minimum

**DR cluster:**

- 4 nodes at secondary site
- Sized for failover capacity (typically 70-80% of primary)
- Same node spec as primary for operational consistency

### Network Topology

- Top-of-rack switches: customer's preferred vendor (Cisco Nexus, Arista, Juniper); 2x ToR per cluster for redundancy
- Link speeds: 25 GbE for production data; 10 GbE acceptable for management
- Bond mode: balance-slb or LACP depending on switch capability
- Replication WAN: 200-500 Mbps between sites for typical Async DR

### Software Stack

- **Hypervisor:** AHV (default) or ESXi-on-Nutanix (if customer is mid-VMware-migration)
- **AOS:** Pro tier
- **NCM:** Pro tier (provides Intelligent Operations and Flow Network Security)
- **Files:** sized for existing file workloads (typically 20-50 TB)
- **Objects:** sized for backup target consolidation (typically 30-80 TB)
- **Volumes:** as needed for any iSCSI consumers
- **Flow Network Security:** included with NCM Pro; baseline microsegmentation policies

### DR Approach

- Async replication to DR site (1-hour RPO typical)
- Recovery Plans for orchestrated failover
- Quarterly test failover into isolated network at DR site
- Backup target replication via Veeam (or equivalent) to Objects at primary, replicated to DR

### Design Rationale

Calibrated for the BlueAlly mid-market sweet spot. Captures the consolidation story across compute, storage tiers, DR, and management without overprovisioning for enterprise-scale features the customer doesn't need.

8-node primary provides N+1 redundancy with margin during rebuild; supports growth to ~200 VMs comfortably; allows 150-VM workload to run with healthy headroom.

NCM Pro tier is the practical sweet spot: includes Intelligent Operations (capacity analytics) and Flow Network Security (microsegmentation), which most mid-market customers want. NCM Ultimate is overkill unless the customer specifically needs Self-Service or X-Play depth.

### Trade-offs Accepted

- No NearSync or Metro (Async-only DR; sub-15-min RPO not in scope)
- No NCM Ultimate (no Self-Service blueprints, no X-Play automation depth)
- Single-cluster production (no multi-cluster blast-radius management)
- 25 GbE rather than 100 GbE (sufficient for the workload; 100 GbE is overprovisioning)

### Variations

- **If customer is heavily Linux/Containers:** add Kubernetes-on-Nutanix consideration; cluster shape unchanged
- **If customer wants higher RPO:** upgrade to AOS Ultimate for NearSync on Tier-1 workloads
- **If customer is starting smaller (50-100 VMs):** scale down to 4-5 node primary

### Cross-References

- [Scenario 1: Mid-Market Full Consolidation](./appendix-c-scenarios.md)
- [Sizing Rules § Cluster Sizing Fundamentals](./appendix-f-sizing-rules.md)
- [Module 9 § Pricing Construction](./09-licensing-economics.md)

---

## 2. Medium Reference Architecture

### Profile

Larger mid-market or smaller enterprise, ~500-1,000 employees in IT-relevant roles, primary datacenter plus dedicated DR site, possibly some ROBO sites.

- VM count: 400-800
- Storage: 100-300 TB usable
- Workloads: general-purpose VMs, file shares, VDI (~500-1,000 sessions), databases, possibly small object storage needs
- Compliance: industry-standard (HIPAA, PCI DSS Level 2, SOC 2)
- Team: 6-12 person infrastructure team
- Annual run-rate today: $400K-1M

### Sizing

**Production cluster:**

- 12-16 nodes, OEM partner hardware (Dell XC, HPE DX, Lenovo HX, or Cisco UCS)
- All-NVMe, 1-1.5 TB RAM per node, 48-64 cores per node
- Storage capacity: 200-400 TB raw, providing 100-200 TB usable
- 25 GbE production, 100 GbE optional for east-west bandwidth
- 4-6 NICs per node

**DR cluster:**

- 8-10 nodes
- Sized for ~75% of primary capacity
- Same hardware family

**Optional Files/Objects cluster:**

- For larger file or object workloads, dedicate a 4-6 node cluster sized for file/object specifically
- Allows tuning the cluster for file/object I/O patterns separately from general VM workload

### Network Topology

- 2x ToR per cluster (redundant)
- 25 GbE production, 100 GbE for spine if east-west traffic justifies
- Bond mode: LACP with switch coordination
- Replication WAN: 500 Mbps-2 Gbps depending on change rate
- Dedicated replication VLAN or QoS lane

### Software Stack

- **Hypervisor:** AHV with possible ESXi-on-Nutanix subset
- **AOS:** Pro tier (Ultimate for tier-1 workloads needing NearSync)
- **NCM:** Pro tier (Ultimate if Self-Service / X-Play needed)
- **Files:** dedicated File Server for user shares + application file storage; 50-150 TB
- **Objects:** dedicated Object Store for backup targets and on-prem analytics; 100-200 TB
- **Volumes:** as needed
- **Flow Network Security:** category-based microsegmentation; PCI scope boundary enforcement
- **Flow Virtual Networking:** if multi-tenancy is in scope

### DR Approach

- Tier-1 workloads (~50-100 VMs): NearSync replication, 15-minute RPO, Recovery Plans orchestration
- Tier-2 workloads (~200-300 VMs): Async hourly, 1-hour RPO
- Tier-3 workloads (~150-300 VMs): Async daily or redeploy from gold image
- Quarterly test failover; SRM coexistence if customer has deep SRM investment

### Design Rationale

Sized for the consolidation story across compute, file/object/block storage, DR with tiered SLAs, and microsegmentation. The split between general-purpose cluster and dedicated Files/Objects cluster (when warranted) lets each cluster tune for its workload pattern; this is one of Nutanix's strengths over single-cluster-with-everything designs.

NCM Pro is the typical choice; upgrade to Ultimate when the customer needs Self-Service blueprints or X-Play event-driven automation for ServiceNow/SIEM integration.

Tiered DR matches infrastructure investment to workload criticality; not every workload needs NearSync.

### Trade-offs Accepted

- Multi-cluster operational complexity (more clusters = more upgrade coordination, more management surface)
- Some workloads stay on hybrid (ESXi-on-Nutanix subset for NSX-T-deep or SRM-deep workloads)
- Not pursuing extreme-scale features (Metro Availability typically not in scope unless metro-area DR is required)

### Variations

- **If multi-site Metro Availability needed:** add witness VM at third site, tune for synchronous replication, network <5ms RTT
- **If heavy NSX-T retention:** plan permanent ESXi-on-Nutanix subset; map workloads to AHV vs ESXi-on-Nutanix
- **If SRM customization is deep:** keep SRM for SRM-orchestrated workloads, Recovery Plans for new

### Cross-References

- [Scenario 1: Mid-Market Full Consolidation](./appendix-c-scenarios.md)
- [Scenario 2: Enterprise Multi-Site](./appendix-c-scenarios.md)
- [Module 7: Data Protection](./07-data-protection.md)

---

## 3. Large Reference Architecture

### Profile

Enterprise customer, 2,000+ employees in IT-relevant roles, multiple datacenters (often 2+ production, 1+ DR, possibly cloud-extended), significant footprint of mixed workloads.

- VM count: 1,500-5,000+
- Storage: 500 TB-2 PB usable
- Workloads: full enterprise mix; tier-0 mission-critical, multi-tenant business units, VDI at scale, large databases, significant file and object storage
- Compliance: heavy (SOX, PCI DSS Level 1, HIPAA, FedRAMP, industry-specific)
- Team: 15-30+ person infrastructure team
- Annual run-rate today: $1M-5M+

### Sizing

**Multiple production clusters** (workload-aligned):

- General-purpose cluster(s): 16-32 nodes each; multiple if workload justifies
- Database cluster: 8-12 nodes, all-NVMe, sized for database performance specifically
- VDI cluster: 12-24 nodes (if VDI is significant), tuned for boot-storm patterns
- Files cluster: 6-12 nodes, dedicated for file workloads
- Objects cluster: 6-12 nodes, dedicated for object storage

**Cluster splitting rationale:** blast-radius management. A 32-node cluster failing causes more impact than 4x 8-node clusters failing one. Multiple smaller clusters also enable independent upgrade cadences.

**DR clusters:**

- Mirror the production cluster topology at DR site
- Sized for full failover capacity for tier-1 and tier-2; tier-3 redeploys

**NC2 cloud DR (optional):**

- 8-12 NC2 nodes for additional cloud-extended DR
- Used for compliance-acceptable workloads needing third-site failover capability

### Network Topology

- Spine-leaf architecture with 100 GbE spine
- 25 GbE leaf-to-server (or 100 GbE for high-density)
- Dedicated network fabric for Nutanix replication if change rate justifies
- WAN: 5-20+ Gbps for replication between sites
- Multi-tenant network isolation via VLAN or VPC overlays (Flow Virtual Networking)

### Software Stack

- **Hypervisor:** mixed AHV + ESXi-on-Nutanix; AHV for new workloads; ESXi for NSX-T-deep / SRM-deep
- **AOS:** Pro for general; Ultimate for tier-1 with NearSync requirements
- **NCM:** Ultimate (Self-Service blueprints, X-Play, Cost Governance)
- **Prism Central:** scale-out (3 VMs) for HA and >10K VM management
- **Files:** multiple File Servers for tenant or workload separation
- **Objects:** multiple Object Stores for tenant or use-case separation; WORM-enabled buckets for regulatory archives
- **Volumes:** as needed for bare-metal databases, legacy iSCSI consumers
- **Flow Network Security:** comprehensive category-based microsegmentation; PCI / SOX scope boundary enforcement
- **Flow Virtual Networking:** multi-tenant VPC overlays; service insertion for advanced security

### DR Approach

- Tier-0 mission-critical: Metro Availability between primary datacenters where applicable; NearSync to remote DR
- Tier-1 production: NearSync replication, 15-min RPO
- Tier-2 production: Async hourly, 1-hour RPO
- Tier-3 / non-production: Async daily or redeploy
- Quarterly test failover at minimum, with audit attestation
- SRM coexistence indefinite; Recovery Plans for new
- Compliance-driven WORM archives in Objects with multi-year retention

### Design Rationale

Multi-cluster architecture is the differentiator for large enterprise. Single-cluster-with-everything fails at this scale due to blast radius, upgrade coordination, and workload-specific tuning needs. Workload-aligned clusters allow each to be tuned for its pattern (database, VDI, file/object) while the central Prism Central provides the unified management view.

NCM Ultimate enables the platform-level value (Self-Service for tenant onboarding, X-Play for event-driven automation, Cost Governance for chargeback) that large enterprise environments require.

Mixed-hypervisor architecture is the honest answer at this scale. Pure AHV migration is rare for enterprise customers with deep NSX-T / SRM / specific-application investments. Hybrid steady-state is the typical successful end state, possibly indefinite.

### Trade-offs Accepted

- Multi-cluster operational complexity is unavoidable; addressed via Prism Central centralization and disciplined upgrade-coordination
- Multi-vendor (Nutanix + remaining VMware) is the steady state; full consolidation rarely achieved
- Highest licensing cost tier (Ultimate everywhere); justified by enterprise feature needs
- Significant network investment (spine-leaf, 100 GbE)

### Variations

- **If NC2 cloud DR is in scope:** add cloud cluster as third site; align replication topology
- **If financial-services compliance:** add HSM integration, more rigorous audit logging, more frequent test failover
- **If multi-region (international):** plan cross-region replication carefully; bandwidth and latency become critical design constraints

### Cross-References

- [Scenario 2: Enterprise Multi-Site with Deep Incumbents](./appendix-c-scenarios.md)
- [Scenario 4: Compliance-Heavy Financial Services](./appendix-c-scenarios.md)
- [Module 9: Licensing Tier Selection](./09-licensing-economics.md)

---

## 4. VDI Reference Architecture

### Profile

VDI-centric deployment for healthcare, financial services, education, or contact centers. VDI is the primary workload (not a side use case).

- Sessions: 1,000-3,000 typical scale; can extend to 5,000+
- VDI broker: Citrix CVAD or VMware Horizon
- Profile: persistent or non-persistent depending on use case
- Compliance: typically HIPAA (healthcare), PCI (financial), or FERPA (education)
- Team: dedicated VDI team plus infrastructure support

### Sizing

**VDI cluster:**

- 12-24 nodes depending on session count
- All-NVMe required (boot-storm I/O pattern demands it)
- High RAM density (1-1.5 TB per node) for VM density
- 25 GbE production minimum; 100 GbE for >2,000 sessions
- Boot capacity: plan for 7am login storm with sustained high IOPS

**Capacity planning:**

- Persistent profiles: ~30-80 GB per user before dedup; on-disk dedup ratio typically 2-3x for similar-OS profiles
- Non-persistent: smaller per-user, higher density

**DR cluster:**

- Smaller than VDI primary (fail over broker tier first; profiles second)
- VDI DR is often "rebuild from gold image plus profile replication" rather than full session-state preservation

### Network Topology

- Dedicated VDI VLAN with appropriate microsegmentation
- 25 GbE minimum; 100 GbE recommended for boot storm
- Multi-pathing for storage traffic
- WAN to DR sized for profile replication (typically modest)

### Software Stack

- **Hypervisor:** AHV (Citrix CVAD has good AHV support)
- **AOS:** Pro tier
- **NCM:** Pro tier
- **Files:** dedicated File Server for persistent profile storage if not using broker-managed profile management
- **Volumes:** as needed for VDI infrastructure components
- **Flow Network Security:** category-based microsegmentation isolating VDI tier from back-end services

### DR Approach

- Broker tier: Async replication, fast recovery (RTO 30-60 min)
- Persistent profiles: Async replication daily; restore from backup if lost
- VDI sessions are not preserved across DR; users re-authenticate on DR site

### Design Rationale

VDI is one of Nutanix's strongest sweet spots. The architectural advantages of distributed I/O (boot storms handled gracefully across all nodes vs centralized array bottleneck) plus on-disk dedup (significant capacity savings on similar OS images) plus AHV included in AOS (no separate hypervisor licensing) combine to win VDI deals consistently.

All-NVMe is non-negotiable. Spinning-disk VDI causes boot-storm pain that no amount of caching fully solves.

### Trade-offs Accepted

- Higher per-node hardware cost (all-NVMe, high RAM density) than general-purpose clusters
- Dedicated VDI cluster vs general-purpose blend (purposeful, for tuning)
- VDI DR is "infrastructure DR" not "session DR"; users restart sessions

### Variations

- **If non-persistent / pooled deployment:** lower capacity needs; higher density per node
- **If extreme scale (5,000+ sessions):** multi-cluster VDI; partition by department or region
- **If GPU-enabled (engineering / design):** add GPU-equipped nodes; AHV supports GPU passthrough

### Cross-References

- [Scenario 3: VDI Deployment](./appendix-c-scenarios.md)
- [Sizing Rules § VDI](./appendix-f-sizing-rules.md)
- [Module 5 § DSF Performance for Write-Heavy](./05-dsf-storage-deep-dive.md)

---

## 5. Hybrid Nutanix + NC2 Reference Architecture

### Profile

Customer with on-prem Nutanix and cloud-extended capabilities via NC2. Use cases: cloud DR (no second physical datacenter), cloud burst capacity, hybrid-cloud workload mobility.

- On-prem footprint: any size
- Cloud workload: typically 20-30% of on-prem at steady state; can scale up dramatically during failover or burst
- Cloud platform: AWS or Azure
- Compliance: must allow for data in chosen cloud region

### Sizing

**On-prem cluster:**

- Sized per general reference architecture (Small, Medium, or Large above)

**NC2 cluster:**

- 4-8 nodes for steady-state workloads
- Scales up to full failover capacity at DR time
- Hibernation strategy if cloud bare-metal pricing supports

**Network:**

- AWS Direct Connect or Azure ExpressRoute for guaranteed bandwidth
- 1-10 Gbps depending on replication and workload volume
- IPSec VPN as backup connectivity

### Software Stack

- **Same software stack on both sides** (this is NC2's value: same platform on-prem and cloud)
- **AOS:** Pro on both
- **NCM:** Pro or Ultimate; Prism Central manages both clusters as one fleet
- **Files / Objects:** can run in either location depending on workload locality

### DR Approach

- Async replication (1-hour RPO) for tier-1 and tier-2 from on-prem to NC2
- NearSync for tier-1 if bandwidth supports
- Recovery Plans orchestration for failover
- Quarterly test failover into isolated network at NC2
- Replicate backup target (Objects) for off-cloud archive copies

### Design Rationale

NC2 makes cloud DR practical for customers without a second datacenter. The platform parity (same Nutanix on both sides) means runbooks transfer; failover doesn't require translation between platforms; the operational model is consistent.

Hibernation strategies (where supported) reduce steady-state cloud cost; the cluster scales up only when needed for test or actual failover.

### Trade-offs Accepted

- Cloud bare-metal cost is higher than on-prem at steady state; the value is in elimination of second-datacenter capex and the on-demand failover capability
- Cloud egress fees apply for replication direction
- Compliance constraints around data residency must be satisfied; cloud region selection critical

### Variations

- **If primary use case is burst capacity (not DR):** size NC2 cluster for typical burst rather than full failover
- **If multi-cloud (AWS + Azure):** complexity increases; NC2 in both with separate replication topologies
- **If true cloud-native hybrid (Kubernetes cross-cloud):** broader cloud strategy beyond NC2

### Cross-References

- [Scenario 6: Cloud DR with NC2](./appendix-c-scenarios.md)
- [Module 7 § NC2](./07-data-protection.md)
- [Comparison Matrix § NC2 vs VMware Cloud](./appendix-b-comparison-matrix.md#nutanix-nc2-vs-vmware-cloud-vmc-on-aws)

---

## 6. ROBO Fleet Reference Architecture

### Profile

Customer with a primary datacenter plus many distributed sites (retail stores, branch offices, manufacturing plants, oil-and-gas remote sites).

- Sites: 50-1,000+
- Per-site footprint: small (1-3 servers per site)
- Per-site workloads: POS, inventory, surveillance, basic application services
- WAN: variable per site; some on cable internet, some on dedicated MPLS or SD-WAN
- Critical data: replicated to central; non-critical data local-only

### Sizing

**Per-site cluster:**

- 1-node or 2-node Nutanix cluster (small-footprint configurations)
- Sized for the site's local workload (typically 5-20 VMs per site)
- Hardware via OEM partner for unified support; ruggedized options for harsh environments

**Central cluster (datacenter):**

- Sized per Small or Medium reference architecture above
- Acts as central management hub, central backup target, central replication destination

**Network:**

- Per-site WAN sized for change rate (typical: 50-200 Mbps depending on workload volume)
- SD-WAN for multi-site routing if applicable
- Central WAN aggregation sized for sum of sites

### Software Stack

- **Hypervisor:** AHV everywhere
- **AOS:** Pro on edge sites; Pro or Ultimate on central based on workload
- **Prism Central at central datacenter:** manages all sites as one fleet
- **Categories:** `Site: Store-001` through `Site: Store-NNN` for fleet-wide policy
- **Async replication:** critical data from each site to central
- **Objects (central):** backup target for fleet, with appropriate retention
- **Flow Network Security:** consistent policy across fleet via category-based rules

### DR Approach

- Per-site failure: critical data is at central; redeploy from gold image on replacement hardware; restore site-specific state from central
- Central failure: business continuity for HQ; remote sites continue operating with cached data; degraded mode for cross-site workflows
- No site-to-site replication (would exponentially complicate the topology)

### Design Rationale

The central Prism Central pattern is the differentiator. Managing 200+ sites individually is operationally impossible; managing them as a fleet through one Prism Central is workable. Categories-based policy enforcement applies the same security and operational rules across the fleet automatically.

Local-only workloads run on local cluster (POS, inventory cache); critical data replicates to central (transaction logs, customer data); the split optimizes for WAN bandwidth.

Per-site small footprint (1-2 nodes) is cost-efficient; full 3-node clusters at every site is cost-prohibitive at fleet scale.

### Trade-offs Accepted

- Single-node sites have no local redundancy (per-site failure means restore from central)
- 2-node sites have limited redundancy
- Central WAN aggregation is a single point of concentration
- LCM upgrade coordination across hundreds of sites is a real operational discipline

### Variations

- **If sites need higher per-site availability:** scale to 3-node per-site clusters (higher cost)
- **If extreme distributed (1,000+ sites):** consider sub-aggregation by region; multiple central hubs

### Cross-References

- [Scenario 5: ROBO and Distributed Sites](./appendix-c-scenarios.md)
- [Sizing Rules § Cluster Sizing Fundamentals](./appendix-f-sizing-rules.md)

---

## 7. Greenfield Reference Architecture

### Profile

Newly-funded company, no existing infrastructure, hybrid-cloud-native development model.

- Employees: 50-500 in IT-relevant roles, scaling rapidly
- VM workload: SaaS application backend, internal tools, dev/test environments
- Cloud presence: significant AWS or Azure footprint
- Steady-state target: hybrid (on-prem for predictable, cloud for variable)
- Team: small, often 2-5 infrastructure engineers
- Cost-sensitivity: high

### Sizing

**Production cluster:**

- 4-node cluster initially (HCIR commodity hardware for cost flexibility, or OEM partner)
- All-NVMe, balanced compute and capacity
- Scales by adding nodes as growth demands

**Dev/test cluster:**

- 2-3 node cluster for development environments
- Smaller node spec acceptable

**Network:**

- 25 GbE production
- VPN or cloud direct-connect to AWS/Azure VPC for hybrid integration
- Modest WAN (cloud burst is the elasticity, not on-prem-to-on-prem replication)

### Software Stack

- **Hypervisor:** AHV
- **AOS:** Pro
- **NCM:** Pro (Ultimate optional based on growth)
- **Files:** sized for application file storage and development shares (smaller than enterprise scale)
- **Objects:** for steady-state object workloads on-prem complementing cloud S3
- **Volumes:** as needed
- **Flow Network Security:** baseline microsegmentation

### DR Approach

- Initial: rely on AWS for cloud-resident data; on-prem critical data replicated to cloud Objects via Async
- As scale grows: consider NC2 cloud DR for full Nutanix-platform DR
- Avoid building out a second physical datacenter; cloud is the elasticity and the DR

### Design Rationale

Greenfield deployments should optimize for cost flexibility, growth, and avoiding premature commitments. HCIR or OEM hardware (rather than NX) preserves sourcing flexibility. Initial 4-node cluster is sized for current needs plus 6-month growth; scale by adding nodes as needed.

3-year subscription term (rather than 5-year) preserves flexibility given growth uncertainty. Subscription growth provisions ensure mid-term core additions are pre-priced.

Hybrid-cloud-native is the architecture, not on-prem replacing cloud. Steady-state predictable workloads on-prem; variable / elastic workloads in cloud. The split optimizes both cost and elasticity.

### Trade-offs Accepted

- Smaller initial cluster means less headroom for spikes (acceptable for greenfield)
- Cloud dependency for elasticity (intentional)
- Less mature operational tooling than enterprise environments (small team)

### Variations

- **If purely on-prem strategy:** scale on-prem cluster sooner; no cloud burst plan
- **If primarily cloud strategy:** smaller on-prem footprint; on-prem only for steady-state

### Cross-References

- [Scenario 9: Greenfield Deployment](./appendix-c-scenarios.md)
- [Sizing Rules § Cluster Sizing Fundamentals](./appendix-f-sizing-rules.md)
- [Module 9 § Subscription Terms](./09-licensing-economics.md)

---

## 8. Compliance-Heavy Reference Architecture

### Profile

Financial services, healthcare provider, or government contractor. Compliance is the central design driver.

- Compliance: PCI DSS Level 1, HIPAA, FedRAMP, SOX, FFIEC, or similar
- Workload sensitivity: regulated data alongside general-purpose
- Audit cadence: frequent (quarterly external, monthly internal)
- Identity rigor: SSO, MFA, separation of duties, JIT elevation
- Encryption requirements: at-rest with HSM-backed keys, in-transit
- Audit retention: 7-year typical for financial; 6-year HIPAA; varies by jurisdiction

### Sizing

**Production clusters:**

- Sized per Large or Medium reference architecture above
- May be split into compliance-scoped vs general clusters for blast-radius and audit-scope management

**Encryption infrastructure:**

- HSM (HashiCorp Vault, customer's existing HSM, or cloud HSM) for key management
- KMIP integration with Nutanix cluster encryption

**Audit infrastructure:**

- SIEM (Splunk, Elastic, Sentinel, etc.) receiving Prism audit logs
- 7-year log retention configured at SIEM tier

**WORM storage (Objects):**

- Dedicated buckets with WORM enabled for regulatory archives
- Retention policies aligned with regulatory requirements

### Network Topology

- Strict network segmentation with VLAN or VPC isolation per compliance scope
- Flow Network Security enforcing PCI / SOX / HIPAA scope boundaries
- Service insertion of customer's IDS/IPS for deep packet inspection
- Audit-grade network logging
- Encrypted in-transit for replication and management

### Software Stack

- **Hypervisor:** AHV
- **AOS:** Ultimate (for advanced security features and NearSync where required)
- **NCM:** Ultimate (Self-Service with approval workflows, X-Play for compliance-driven automation)
- **Files:** with anti-ransomware enabled; SMB encryption mandatory
- **Objects:** WORM-enabled buckets for archives
- **Volumes:** encrypted; HSM-backed keys
- **Flow Network Security:** comprehensive category-driven enforcement; default-deny posture
- **Cluster encryption:** enabled with HSM key management

### DR Approach

- RF3 for tier-0 financial systems (compliance often mandates)
- Metro Availability between primary datacenters where applicable
- NearSync to remote DR (typically same compliance zone)
- Recovery Plans with audit-attested test failover quarterly
- DR runbook documentation maintained per compliance requirements

### Identity and Access

- SAML to customer's IdP (typically Entra ID, Okta, or Ping)
- Separation of duties: distinct operational and audit roles
- JIT (Just-In-Time) elevation for sensitive operations
- MFA mandatory for all administrative access
- Audit log of every administrative action

### Design Rationale

Compliance is the central design driver, not an overlay. Architecture choices (cluster splitting for audit scope, network segmentation, encryption everywhere, audit logging end-to-end) all flow from compliance requirements.

NCM Ultimate is the typical license tier; the Self-Service approval workflows and X-Play automation are operationally valuable for compliance regimes that require change-control evidence.

WORM Objects buckets are the durable answer for regulatory archives; the multi-year immutability requirement is satisfied by Objects' Object Lock equivalent.

### Trade-offs Accepted

- Highest licensing cost tier
- Operational complexity from compliance-scoped cluster splitting
- HSM integration adds vendor relationship and operational dependency
- Documentation overhead for compliance evidence is real

### Variations

- **If FedRAMP:** validate Nutanix's specific FedRAMP authorization status; some features may have different availability in FedRAMP environments
- **If multi-region with sovereignty:** plan replication topology carefully; cloud DR may not be available if data residency is strict

### Cross-References

- [Scenario 4: Compliance-Heavy Financial Services](./appendix-c-scenarios.md)
- [Module 7 § Encryption and Compliance](./07-data-protection.md)
- [Module 8 § WORM Objects](./08-unified-storage.md)

---

## How to Use These Reference Architectures

**For initial proposal sketching:**

1. Identify which reference matches the customer's profile (Small / Medium / Large / VDI / Hybrid+NC2 / ROBO / Greenfield / Compliance-Heavy)
2. Use it as the starting-point architecture
3. Adjust per customer specifics
4. Run Sizer for binding numbers before submitting

**For architecture review with the customer:**

1. Walk through the reference's design rationale ("we typically design like this for customers in your profile because...")
2. Engage the customer on where their requirements deviate
3. Document the deviations explicitly in the proposal

**For competitive engagements:**

1. Use the reference to anchor the conversation in concrete numbers
2. Explain the trade-offs accepted; this signals you've done this before
3. Compare to the competitor's likely architecture for the same profile

**For cert prep:**

1. NCP-MCI tests sizing-and-design knowledge similar to the Small / Medium architectures
2. NCM-MCI extends to design depth that the Medium / Large / VDI architectures embody
3. NCX-MCI panel defense often involves architectures like the Compliance-Heavy or Hybrid+NC2 patterns

---

## Common Mistakes with Reference Architectures

1. **Treating reference architectures as the proposal.** They're starting points; the actual proposal is customized.
2. **Skipping Sizer.** The reference gets you within ~25%; Sizer gets you to the binding number.
3. **Not naming the trade-offs.** Every architecture sacrifices something; customers respect explicit acknowledgment.
4. **Forcing a customer into the wrong reference.** If the customer profile genuinely doesn't match any of the eight, design from first principles.
5. **Neglecting variations.** Each reference has variations for common deviations; use them rather than re-inventing.
6. **Underestimating compliance complexity.** Compliance-heavy customers genuinely need the Compliance-Heavy reference; don't try to make a Medium architecture work.

---

## Cross-References

- **Modules:** Each reference architecture pulls from multiple modules; the cross-references in each section point to specific modules
- **Glossary:** [Appendix A](./appendix-a-glossary.md) defines the terms used
- **Sizing Rules:** [Appendix F](./appendix-f-sizing-rules.md) provides the sizing math behind these architectures
- **Scenarios:** [Appendix C](./appendix-c-scenarios.md) has the design-exercise versions of these architectures
- **POC Playbook:** [Appendix J](./appendix-j-poc-playbook.md) has the validation steps for these architectures
- **Nutanix Sizer:** the official tool for binding sizing

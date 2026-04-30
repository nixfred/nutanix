---
module: 07
title: Data Protection and DR
estimated_reading_time: 38 min
prerequisites:
  - Modules 01 through 06
  - vSphere DR experience helpful (VR, SRM, snapshots)
  - Understanding of RPO and RTO concepts
  - Familiarity with categories from Module 04
key_terms:
  - Crash-consistent vs application-consistent snapshot
  - Protection Domain (PD)
  - Protection Policy
  - Async Replication
  - NearSync Replication
  - Metro Availability
  - LWS (Light-Weight Snapshots)
  - Recovery Plan (Leap)
  - Recovery Point Objective (RPO)
  - Recovery Time Objective (RTO)
  - NC2 (Nutanix Cloud Clusters)
  - Site Recovery Manager (SRM)
  - Witness VM
diagrams:
  - replication-topologies
  - protection-policies-categories
  - recovery-plan-failover
cert_coverage:
  NCA: ~10%
  NCP-MCI: ~22%
  NCM-MCI: ~25%
  NCP-NS: ~8%
sa_toolkit:
  related_objections: [obj-026, obj-027, obj-028, obj-029, obj-030]
  related_discovery: [q-dr-01, q-dr-02, q-dr-03, q-dr-04, q-dr-05]
---

# Module 7: Data Protection and DR

> **Cert coverage:** NCA (~10%) · NCP-MCI (~22%) · NCM-MCI (~25%) · NCP-NS (~8%)
> **SA toolkit:** Objections #26 through #30 · Discovery Q-DR-01 through Q-DR-05

---

## The Promise

By the end of this module you will:

1. **Design a multi-site DR architecture for a customer in 30 minutes.** Pick the replication mode (Async, NearSync, Metro), define RPO and RTO targets, choose between Protection Domains and Protection Policies, and produce a sized bandwidth requirement. This is one of the most-requested customer deliverables and one of the highest-leverage SA conversations.
2. **Pass roughly 22% of NCP-MCI and 25% of NCM-MCI.** Data protection is one of the heaviest single-topic weights on these exams. NCM-MCI labs frequently include DR scenarios: configure a Protection Policy, set up replication, test a failover, troubleshoot a stalled replication job.
3. **Defend Recovery Plans against an SRM-loyal customer.** SRM has 15+ years of polish. Recovery Plans is younger. The honest comparison is that for AHV-native deployments, Recovery Plans is integrated and capable; for established SRM shops, the migration is real work and the coexistence pattern is often the right answer.
4. **Make the cloud-DR case** using NC2 (Nutanix Cloud Clusters on AWS or Azure) for customers who want a DR site without a second datacenter. By 2026, this is one of the more compelling Nutanix-specific stories and BlueAlly SAs need to be able to walk through the math.
5. **Walk through the snapshot, replication, and recovery sequence end-to-end.** When something goes wrong in DR, the customer's first call is to you. You should know where to look: snapshot schedule status, replication queue depth, Recovery Plan validation, network path between sites.
6. **Make the RPO/RTO/cost tradeoffs explicit.** The customer who wants 1-minute RPO and 30-minute RTO and zero cost increase is asking for something that does not exist. Your job is to translate their business requirements into the right mix of products at a real budget.

DR is the dimension that decides enterprise deals. Customers who survived a real outage have opinions; customers who haven't are about to. Either way, when DR comes up, you need to know this material cold.

---

## Foundation: What You Already Know

You have built or maintained DR for VMware. The pieces:

- **VMware snapshots** for point-in-time recovery (with the consolidation pain Module 3 already covered).
- **vSphere Replication** for VM-level replication to a secondary site.
- **Storage array replication** (NetApp SnapMirror, EMC RecoverPoint, Pure ActiveCluster, etc.) for storage-level replication, often with better RPO than vSphere Replication.
- **Site Recovery Manager (SRM)** for orchestrating failover: VM boot order, IP remapping, runbook automation, test recovery without disrupting production.
- **Backup tooling** (Veeam, Commvault, Rubrik, Cohesity) layered on top for longer-retention backup and granular recovery.

You know the operational realities. RPO depends on the replication mechanism. RTO depends on how fast you can boot VMs at the DR site, with what network reconfiguration, with what application checks. SRM runbook tests are the only thing that proves DR actually works, and most customers do not run them as often as they should.

Hold that experience. Nutanix's data protection stack reorganizes those pieces. The replication is built into the platform (no separate replication appliance). The orchestration is integrated (no separate SRM purchase). The snapshots are DSF-native (no consolidation pain). The cloud-DR option is NC2 (no need to build a second datacenter). The pieces are different but the operational concepts transfer.

> [!FROM-THE-SA-CHAIR]
> When a customer brings up DR, the first move is not feature comparison. The first move is qualifying their actual requirements. *"What is your RPO target by application tier? What is your RTO? Do you have a contractual or regulatory requirement that drives those numbers? Have you tested a real failover in the last 12 months? What broke when you did?"* Most customers cannot answer all five questions cleanly. The conversation that follows is where the SA earns trust by translating their actual needs into the right Nutanix mix. Skip the qualifying step and you are pitching products against requirements you do not actually understand.

---

## Core Content

### Snapshots Revisited: Crash-Consistent vs Application-Consistent

Module 3 covered DSF snapshots as a DSF-native, instant, no-I/O-penalty operation. Layer the consistency dimension on top.

**Crash-consistent snapshots.** Captures the state of the vDisk at a moment in time. From the perspective of a guest, this is equivalent to pulling the power cord and the system coming back up: file systems may need recovery on first boot, in-memory data is lost, transactional databases may roll back recent transactions. For most workloads, crash-consistent recovery is sufficient.

**Application-consistent snapshots.** The snapshot is taken after the application has flushed its in-memory state to disk. On Windows, this requires VSS (Volume Shadow Copy Service) coordination, which Nutanix Guest Tools (NGT) provides. On Linux, application-consistent snapshots typically rely on application-level mechanisms (a database's quiesce command) coordinated through NGT.

When does the distinction matter?

- **Database VMs (SQL Server, Oracle, PostgreSQL).** Application-consistent snapshots are strongly preferred for direct restoration. Crash-consistent recovery requires the database's startup recovery to succeed, which usually works but is slower and occasionally messy.
- **File servers.** Crash-consistent is fine for general SMB/NFS workloads.
- **Application servers (web, app tier).** Crash-consistent is fine if the application is stateless or has its state in a backed-up database.
- **Active Directory domain controllers.** AD has its own consistency protocols; crash-consistent is acceptable but Microsoft's preferred approach involves database-aware snapshots.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI**
> Snapshot consistency types are testable. Memorize: crash-consistent (default, captures vDisk state at a moment), application-consistent (requires NGT and VSS coordination on Windows or application-level quiesce on Linux). Trap distractor: questions that imply application-consistent is the same as crash-consistent (false), or that NGT is unrelated to snapshot consistency (false; NGT provides the VSS integration).

---

### Protection Domains vs Protection Policies

This is the architectural shift that matters operationally and exam-wise. Read carefully.

**Protection Domains (PDs).** The legacy construct, configured in Prism Element. A PD is a group of VMs (and/or vDisks) that share a single replication and snapshot schedule. You create a PD, add VMs to it, attach a remote site, configure the schedule (e.g., snapshot every hour, replicate every 4 hours), and the PD enforces it. PDs have been the workhorse of Nutanix DR for years.

**Protection Policies.** The modern construct, configured in Prism Central. Policies are category-driven (Module 4). A Protection Policy says: "All VMs with `Environment: Production` get hourly snapshots retained for 7 days, with NearSync replication to the DR site." VMs are assigned to the policy by their categories, not by manual addition. Policies are tied to **Recovery Plans** (Leap) for orchestrated failover.

**Why the shift matters:**

| Dimension | Protection Domain | Protection Policy |
|---|---|---|
| Where configured | Prism Element | Prism Central |
| Membership model | Manual VM addition | Category-driven (auto) |
| Failover orchestration | Native PD failover (basic) | Recovery Plans (rich orchestration) |
| Multi-site management | Per-PD per-cluster | Centralized in PC |
| Recommended for new deployments | No | Yes |

**Existing customers:** PDs continue to work. There is no forced migration. Many production environments still run PDs because the migration to Policies is a project that requires planning. New deployments should start with Policies.

> [!FROM-THE-SA-CHAIR]
> Customers with established Protection Domains are common. The wrong move is to demand they migrate to Policies. The right move is: *"Your existing PDs continue to work. We recommend Policies for any new workload, and we can plan a migration of existing PDs at your pace, on your timeline. There's no pressure."* That respects their installed base. For greenfield deployments, recommend Policies from day one.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI**
> Both PDs and Policies are testable. Memorize the distinction: PDs are PE-based, manual-membership, legacy. Policies are PC-based, category-driven, modern. Trap distractor: "Protection Policies replaced Protection Domains and PDs are no longer supported" (false; both coexist).

---

### Async Replication: The Default Workhorse

**Async replication** is the bread-and-butter Nutanix DR mechanism. The pattern:

1. Take a snapshot at the source cluster on schedule (e.g., hourly).
2. Compute the delta between this snapshot and the previous replicated one.
3. Send the delta over the network to the destination cluster.
4. The destination cluster materializes the new snapshot.
5. Retain snapshots per the configured retention policy.

**Characteristics:**

- **Minimum RPO:** typically 1 hour, but can be configured down to 15 minutes in some scenarios (with some operational care). RPO is governed by the snapshot/replication interval, not by inherent technology limits.
- **Bandwidth efficiency:** delta-based, so steady-state bandwidth scales with change rate (the "delta" of the workload), not with total data size.
- **Network requirements:** any IP-routable connection between sites. Latency requirements are forgiving (works fine over WAN with hundreds of milliseconds RTT).
- **Cost:** lowest of the three replication modes. Suitable for most workloads.

**Topology options:**
- **One-to-one.** Single primary, single DR site.
- **One-to-many.** Single primary replicating to multiple DR sites (different RPOs to each).
- **Many-to-one.** Multiple production clusters replicating to a consolidated DR site (common for ROBO consolidation).
- **Bi-directional.** Two clusters protecting each other.

Most customers start here. Most stay here for the bulk of their workloads.

> [!FAMILIAR]
> Async replication maps cleanly onto vSphere Replication mental models. Periodic snapshot capture, delta computation, network transfer, target-side materialization. The mechanics are familiar: VR works the same way at a different layer. The differences are operational: Nutanix Async is configured in Prism (no separate VR appliance), runs natively in the platform (no separate licensing), and is part of the same management plane as compute and storage. If you have configured VR before, you will configure Nutanix Async in less time on your first try.

### NearSync Replication: When 1-Hour RPO Isn't Enough

**NearSync** is Nutanix's "almost-synchronous" replication mode. It uses **LWS (Light-Weight Snapshots)** under the hood, taking very frequent (sub-minute) micro-snapshots and continuously replicating them.

**Characteristics:**

- **RPO:** as low as 20 seconds in optimal configurations; commonly designed for 1-15 minute RPO.
- **Bandwidth requirements:** higher than Async. The replication is more continuous and less coalesced. Plan for sustained bandwidth roughly proportional to the workload's write rate.
- **Network requirements:** typically <5ms RTT to the destination, though specific platform versions vary. Some configurations relax this.
- **Cluster overhead:** higher than Async. NearSync places more load on Stargate and Curator at the source cluster.
- **Cost:** higher than Async, lower than Metro.

**When to use NearSync:**

- Tier-1 production databases where 1-hour RPO is too long.
- Compliance-driven workloads with sub-15-minute RPO mandates.
- Customers with adequate bandwidth and acceptable latency between sites.

**The honest gotcha:** NearSync's resource cost on the cluster is real. Customers who try to NearSync-protect their entire estate often find they need bigger CVMs or face cluster headroom issues. Use NearSync for the workloads that genuinely need it; leave the rest on Async.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI · NCP-NS**
> NearSync's specifics are tested. Memorize: NearSync uses Light-Weight Snapshots (LWS), achieves sub-15-minute RPO commonly, requires lower-latency networking than Async, and has higher cluster resource overhead. Trap distractor: "NearSync is synchronous" (false; sync is Metro). NearSync is "near" sync but not zero RPO.

### Metro Availability: Synchronous, Zero RPO

**Metro Availability** is true synchronous replication: every write at the source is replicated to the destination before it is acknowledged. RPO is zero (no data loss on any single-site failure).

**Characteristics:**

- **RPO:** zero. Synchronous replication.
- **Latency requirement:** <5ms RTT typically. Real-world deployments are usually metro-area distances (campus, dual-datacenter within a city).
- **Topology:** active-standby (typical) or active-active (advanced configurations).
- **Witness VM:** required for split-brain protection. Witness runs on a third site (a small cluster, a separate Nutanix instance, or sometimes a public-cloud VM) and provides quorum during partition events.
- **Cost:** highest of the three modes. Bandwidth + tightly-coupled networking + witness infrastructure.

**When to use Metro:**

- Mission-critical workloads with zero-data-loss requirements.
- Active-active datacenter architectures where workloads run in both sites simultaneously.
- Compliance frameworks that mandate synchronous replication (less common but exists).

**Important constraint:** Metro is only useful within metro-area latency (typically <100km). It is not a long-distance DR solution. For wide-area DR, you combine Metro (between two close sites) with Async or NearSync to a third remote site for full geographic resilience.

> [!DIFFERENT]
> Metro Availability resembles VMware's Stretched Cluster (often deployed with vSAN Stretched Cluster or NetApp MetroCluster) but with different operational characteristics. The Witness requirement is similar in concept. The latency budget is similar. The deployment workflow is integrated into Prism rather than requiring vSphere + storage-array-specific tooling. Customers familiar with stretched clusters in the VMware world transfer their mental model directly.

---

### Diagram: Replication Topologies (Async / NearSync / Metro)

**id:** `replication-topologies`
**type:** comparison
**caption:** Three replication modes side by side. RPO, latency budget, bandwidth, and resource cost characteristics shown.
**exam_relevance:** [NCP-MCI, NCM-MCI, NCP-NS]
**whiteboard_ready:** true

**Elements:**
- Three side-by-side panels:

**Panel 1: Async**
- Two cluster icons: Site A (Primary) and Site B (DR), separated by a long arrow labeled "WAN, any latency"
- Replication mechanism shown as periodic snapshot transfers (clock icons every hour)
- Characteristics box:
  - RPO: 1+ hour typical (15 min minimum in some configs)
  - Latency: any (works over WAN)
  - Bandwidth: change-rate proportional, low steady-state
  - Cluster overhead: low
  - Use case: General-purpose DR, ROBO, geographic distance

**Panel 2: NearSync**
- Two cluster icons: Site A and Site B, separated by a shorter arrow labeled "Low-latency network, <5ms typical"
- Replication mechanism shown as frequent LWS transfers (clock icons every minute)
- Characteristics box:
  - RPO: 20s to 15 min commonly
  - Latency: typically <5ms RTT
  - Bandwidth: continuous, scales with write rate
  - Cluster overhead: moderate to high (Stargate + Curator)
  - Use case: Tier-1 databases, sub-15-minute RPO requirements

**Panel 3: Metro Availability**
- Two cluster icons: Site A and Site B, in a tight loop labeled "Synchronous, <5ms RTT, metro distance"
- A third small icon labeled "Witness VM (third site)" connected to both clusters
- Replication mechanism shown as synchronous write commit
- Characteristics box:
  - RPO: 0 (zero data loss)
  - Latency: <5ms RTT, metro distance only
  - Bandwidth: full write throughput
  - Cluster overhead: high
  - Use case: Mission-critical, active-active, zero-RPO compliance

**Annotations:**
- Below diagram: "Most customers run a mix: Async for the bulk of workloads, NearSync for Tier-1 databases, Metro for the genuine zero-RPO use cases."
- Side note: "Metro requires metro-area distances. For long-distance DR, layer Async or NearSync on top to a remote third site."

**Why this diagram exists:** Customers comparing replication options need this side-by-side view to make informed RPO-vs-cost decisions. Whiteboard it during DR design conversations. Memorize the latency, RPO, and overhead characteristics for the cert exams.

---

### The Cycle, Frame Two: DR as RPO/RTO Mapped to Products

For an operations leader, the durable DR frame is mapping business requirements to technology choices.

| Application Tier | Business RPO | Business RTO | Recommended Approach |
|---|---|---|---|
| Mission-critical, zero-data-loss | 0 | <30 min | Metro Availability (campus) + Async to third site |
| Tier-1 production DB | 1-15 min | 1-2 hours | NearSync to DR cluster + Recovery Plan |
| Production general-purpose | 1-4 hours | 2-4 hours | Async + Recovery Plan |
| Test/Dev | 8-24 hours | 4-8 hours | Async with longer interval, or no DR |
| Ephemeral / stateless | n/a | redeploy | No replication; redeploy from source-of-truth |

This is the design conversation in 15 minutes. Walk the customer through their tiers, agree on RPO/RTO targets, map to the technology, and the architecture writes itself.

### The Cycle, Frame Three: Recovery Plans (Leap) as the SRM Replacement

**Recovery Plans** (sometimes called Leap, the original product brand) are Nutanix's DR orchestration product. They live in Prism Central and define:

- **What VMs are protected** (via category membership).
- **The startup order** (which VMs come up first, second, third).
- **Network mapping** (production VLAN 100 maps to DR VLAN 200).
- **IP address remapping** (or DHCP-based reassignment at the DR site).
- **Pre-checks and post-checks** (run a script before failover, run a script after).
- **Test failover capability** (run a failover into an isolated network, validate, tear down).
- **Manual or automated failover triggers.**

This is the SRM equivalent. The functional comparison:

| Feature | Site Recovery Manager (SRM) | Recovery Plans (Leap) |
|---|---|---|
| Hypervisor support | ESXi only | AHV native; ESXi-on-Nutanix supported via integration |
| Replication source | vSphere Replication or array-based | Native Nutanix (Async, NearSync, Metro) |
| Licensing | Separate VMware product | Bundled with Nutanix Cloud Manager (varies by tier) |
| Test failover | Yes, mature | Yes |
| Runbook orchestration | Mature, deeply customizable | Capable, less customizable in advanced edge cases |
| IP remapping / DHCP | Yes | Yes |
| Pre-/post-failover scripts | Yes | Yes |
| Multi-site planning | Yes | Yes |
| Maturity | 15+ years | 5+ years, rapidly improving |
| Cross-site management | vCenter-driven | Prism Central |

**The honest comparison:** SRM is more mature and has more advanced runbook customization for complex scenarios. Recovery Plans is integrated, free or bundled, and simpler to operate. For most enterprise DR requirements, Recovery Plans is sufficient. For customers with established SRM deployments, the migration is real work and the coexistence pattern (SRM on ESXi-on-Nutanix, Recovery Plans for AHV workloads) is often the right answer.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI**
> Recovery Plans are testable. Memorize: Recovery Plans live in Prism Central, define VM startup order and network/IP mappings, support test failover without disrupting production, and are the orchestration layer on top of Nutanix replication. Trap distractor: "Recovery Plans require Site Recovery Manager" (false; they are independent of SRM).

---

### Diagram: Protection Policies and Categories

**id:** `protection-policies-categories`
**type:** flow
**caption:** A category-driven protection policy automatically protects new VMs as they are tagged. No manual addition required.
**exam_relevance:** [NCP-MCI, NCM-MCI]
**whiteboard_ready:** false

**Elements:**
- Left side: A pool of VMs (light gray) showing categories assigned: some with `Environment: Production`, some with `Environment: Development`, some without categories yet
- Center: A "Protection Policy" box (gold) with rules:
  - Match: `Environment: Production`
  - Snapshot schedule: hourly, retain 7 days
  - Replication mode: NearSync
  - Destination: Site B
- Right side: Automatic actions
  - VMs matching the policy are auto-included
  - New VMs created with `Environment: Production` are auto-protected at category-assignment time
  - VMs that lose the category are auto-removed from protection
- Below: Recovery Plan "Production-Failover-Plan" (gold), referencing the Protection Policy and defining startup order, network mappings, and pre-/post-checks

**Connections:**
- Categories on VMs to Policy match-rule
- Policy to snapshot schedule and replication mode
- Replication arrows from local cluster to Site B
- Recovery Plan referencing the policy

**Annotations:**
- "Category assignment is the policy enrollment moment. Tag a VM Production, it is protected within minutes."
- "No manual addition. No forgotten VMs. Compare to manually-managed Protection Domains."
- "Recovery Plans orchestrate the failover of category-protected VMs."

**Why this diagram exists:** The category-driven policy model is the operational improvement over manually-managed PDs. This diagram shows why customers shift to Policies once they understand the automatic-enrollment behavior. Use it in design discussions for new deployments.

---

### NC2: DR to the Cloud Without a Second Datacenter

**NC2 (Nutanix Cloud Clusters)** runs the Nutanix platform on AWS or Azure bare-metal hosts. From the platform's perspective, an NC2 cluster looks like any other Nutanix cluster: AOS, AHV, CVMs, DSF, Prism. From the customer's perspective, it is Nutanix infrastructure they pay for as a cloud-consumption model rather than as on-premises hardware.

**For DR specifically, NC2 enables:**

- **Replicate from on-prem Nutanix to NC2 in cloud.** Use the same Async, NearSync (where supported), or Metro mechanisms.
- **Failover to cloud without a second datacenter.** When primary fails, VMs come up on NC2 in the cloud region.
- **Variable cost.** Pay for cloud capacity continuously (for active replication target) or with hibernation patterns (cluster spun down most of the time, spun up on failover or DR test).
- **Fast scaling.** Add NC2 capacity in cloud at the rate of cloud provisioning, not at the rate of physical hardware procurement.

**The economics:** NC2 is meaningfully cheaper than building and maintaining a second physical datacenter for many mid-market customers. For large customers, the math depends on workload size, retention requirements, and whether they have existing colo space they would otherwise consolidate.

**The honest constraints:**

- Cloud bare-metal pricing varies. Some workloads are cheap in cloud; some are expensive.
- Replication bandwidth from on-prem to cloud is real cost (egress fees apply on failback in some models).
- Failover RTO depends on whether the NC2 cluster is hot, warm, or cold.
- Some workloads have data-sovereignty constraints that prevent cloud DR.

> [!FROM-THE-SA-CHAIR]
> NC2 is one of the most compelling Nutanix-specific value propositions in 2026. For mid-market customers without a second datacenter, the conversation goes: *"You don't need to build a DR datacenter. We can replicate to NC2 in AWS or Azure, run your DR site as cloud capacity, and only pay for it during DR testing or actual failover. The economics often beat building physical DR by a wide margin, and the architecture is the same Nutanix platform you're already running on-prem."* When discovery surfaces "we don't have a second datacenter and our DR is paper-only," NC2 is the answer that turns that conversation around.

---

### Diagram: Recovery Plan Failover Flow

**id:** `recovery-plan-failover`
**type:** flow
**caption:** A primary-site loss triggers Recovery Plan execution at the DR site. VMs come up in the right order with the right network mappings.
**exam_relevance:** [NCP-MCI, NCM-MCI]
**whiteboard_ready:** true

**Elements:**
- Top half: Site A (Primary) before failure
  - 4 nodes, multiple VMs running, replicating snapshots to Site B continuously (rust arrow downward)
- Failure event: a red X over Site A: "Primary site lost"
- Recovery Plan execution timeline shown left to right:
  - T+0: Primary loss detected
  - T+1-5min: Recovery Plan triggered (manual or automatic depending on configuration)
  - T+5-10min: Pre-checks run (DNS reachable, network online, etc.)
  - T+10-20min: VMs powered on at Site B in startup-order groups (DB tier first, App tier second, Web tier third)
  - T+20-30min: IP remapping applied; DNS updated; post-checks run (application health, etc.)
  - T+30 min: Service restored at DR site
- Bottom half: Site B (DR) after failover
  - VMs running at DR site with appropriate IP / network mapping
  - Application reachable from end users via updated DNS or load balancer

**Connections:**
- Replication: Site A to Site B continuous prior to failure
- Failover trigger: from monitoring/manual decision to Recovery Plan
- Recovery Plan to: DR site VM-power-on, IP-remap, DNS-update, post-check
- End users to DR site: "Service restored"

**Annotations:**
- "Startup order matters. Database before application, application before web."
- "Test failovers can run this whole flow into an isolated network, without disrupting production."
- "RTO depends on VM count, application checks, and post-failover validation. Typical 15-60 minutes for medium environments."

**Why this diagram exists:** Customers want to see the failover flow visualized. This diagram shows what Recovery Plans actually do, including the time axis. Use it in DR architecture conversations to set realistic RTO expectations. Whiteboard it.

---

### Test Failover: The Feature That Customers Actually Use

The most important DR feature is the one customers neglect: testing.

Recovery Plans support **test failover**: run the entire failover sequence into an isolated network at the DR site, validate that everything comes up correctly, and tear down without affecting production. The test creates an isolated VLAN at the DR site, brings VMs up there, runs the configured checks, and reports.

**Why this matters:** DR runbooks that have not been tested in 12+ months frequently do not work when a real failover comes. The state of the world changes: VMs are added, network configurations drift, IP allocations change, application dependencies shift. The runbook decays.

**Recommended customer cadence:** quarterly test failovers minimum, monthly for critical workloads. The test takes 1-2 hours typically. The customer's DR program is real if and only if they actually run these.

> [!FROM-THE-SA-CHAIR]
> Test failover is one of the durable BlueAlly value propositions. *"Building DR is necessary; testing DR is what proves it works. Recovery Plans make test failover a 1-2-hour quarterly exercise instead of a multi-day fire drill that disrupts production. When you run your first quarterly test and find three things that don't work the way the runbook said, you'll know the platform paid for itself."* That sentence wins customers who have been burned by untested DR.

---

### What DR Genuinely Lacks vs Mature SRM Deployments

Honest gap list. Read it twice.

1. **Some advanced runbook customization.** SRM has 15+ years of accumulated capabilities for very complex orchestration patterns (cross-vendor app integration, complex pre-checks, vendor-specific scripted callouts). Recovery Plans handles the typical cases well; some advanced edge cases require scripted extensions.
2. **Cross-vendor replication source flexibility.** SRM can use array-based replication from a wide range of arrays. Recovery Plans is tied to Nutanix's native replication. For customers who want to keep their array's replication and orchestrate failover via SRM, that's a reason to keep SRM-on-ESXi-on-Nutanix.
3. **Reporting depth.** SRM's reporting around DR readiness, test history, and compliance posture has had more time to mature. Prism's reporting is increasingly capable but younger.
4. **Cross-environment scope.** SRM deployments often span heterogeneous infrastructure that includes non-Nutanix elements; Recovery Plans is Nutanix-centric.

For typical mid-market and enterprise general-purpose DR requirements, none of these are deal-breakers. For customers with established SRM and complex multi-vendor DR, the coexistence pattern is the durable answer.

### What Nutanix DR Has That SRM Does Not

1. **Integrated platform.** DR is part of the platform, not a separate purchased product.
2. **DSF-native snapshots underneath.** No I/O penalty, no consolidation, instant.
3. **Category-driven protection policies.** New VMs auto-enroll based on tagging.
4. **NC2 cloud DR option.** SRM has cloud-DR capabilities via VMware Cloud, but the integration is more recent and the licensing is separate.
5. **Single management plane.** Replication, recovery plans, and DR test in the same UI as compute and storage.
6. **Bundled licensing for basic capabilities.** Recovery Plans and basic replication included; advanced features in NCM tiers.

---

## Lab Exercise: Build a Protection Policy and Recovery Plan

> [!LAB] **Time:** ~3 hours · **Platform:** 3-node CE cluster + Prism Central from Module 04. Multi-cluster DR scenarios require two clusters; for CE, this lab focuses on single-cluster snapshot/policy operations and conceptual coverage of multi-cluster.

**Steps:**

1. **Take a manual VM snapshot.** From Prism Central, select a VM, choose "Take Snapshot." Note the type options: crash-consistent (default) or application-consistent (requires NGT installed in the guest, which the lab VMs may not have).

2. **Install NGT on a Linux VM.** SSH into one of your lab VMs, then mount and install NGT:
   ```
   sudo mount /dev/cdrom /mnt/cdrom
   sudo /mnt/cdrom/installer/linux/install_ngt.py
   ```
   (Specifics vary by NGT version; Prism's "Install NGT" wizard does this for you in production deployments.)

3. **Take an application-consistent snapshot.** With NGT installed, the snapshot UI now offers application-consistent. Take one. Verify the snapshot succeeds.

4. **Create categories** if you haven't already (from Module 04 lab):
   - Key: `Environment`, Values: `Production`, `Development`, `Test`
   - Key: `BackupTier`, Values: `Gold`, `Silver`, `Bronze`

5. **Tag VMs with categories.** Apply `Environment: Production` and `BackupTier: Gold` to your lab VM.

6. **Create a Protection Policy** in Prism Central:
   - Name: `Lab-Production-Policy`
   - Match: VMs with category `Environment: Production`
   - Snapshot schedule: every 1 hour, retain for 7 days
   - Replication: leave disabled for now (single-cluster lab) or configure to a second cluster if you have one
   - Save and enable

7. **Verify policy enrollment.** Navigate to the policy detail page and confirm your VM is automatically included based on its category.

8. **Tag a second VM with Environment: Production.** Confirm it auto-enrolls in the policy without manual addition.

9. **(Multi-cluster lab, if available)** Pair two clusters. Configure replication. Validate snapshots transfer.

10. **Create a Recovery Plan.** Navigate to Recovery Plans > Create. Define:
    - Name: `Lab-Production-Recovery`
    - Source / target sites
    - VMs included (via category match or explicit list)
    - Startup order (groups: DB first, App second, Web third)
    - Network mapping (production VLAN to DR VLAN)
    - Pre/post scripts (optional)
    - Save

11. **Run a test failover** (multi-cluster, optional). The platform spins up VMs at the DR site in an isolated network, runs your defined checks, and reports.

12. **Inspect Curator's role in protection.** From a CVM, run:
    ```
    curator_cli get_curator_state
    ```
    Note the protection-related background tasks: snapshot reclamation, replication queue management, retention enforcement.

**What this teaches you:**
- The snapshot-consistency distinction in practice.
- Category-driven protection policy enrollment.
- Recovery Plan structure and configuration.
- The CLI surface for protection diagnostics.

**Customer-demo angle:** Steps 4-7 are the customer-demo flow for category-driven protection. Show a customer how tagging a VM with `Environment: Production` automatically enrolls it in the configured policy. The "no manual addition" insight lands viscerally.

---

## Practice Questions

Twelve questions. Six knowledge MCQ, four scenario MCQ, two NCX-style design questions. Read each, answer in your head, then read the explanation.

---

**Q1.** What is required for an application-consistent snapshot of a Windows VM running on AHV?
**Cert relevance:** NCP-MCI · NCM-MCI

A) Nothing additional; all snapshots are application-consistent by default
B) NGT (Nutanix Guest Tools) must be installed in the guest, and the application must support VSS
C) The VM must be powered off during the snapshot
D) An external backup product must orchestrate the snapshot

**Answer:** B

**Why this answer:** Application-consistent snapshots on Windows require VSS coordination, which NGT provides. The application must be VSS-aware (most major applications including SQL Server, Exchange, AD are).

**Why not the others:**
- A) Default snapshots are crash-consistent; application-consistent requires NGT.
- C) Power-off would be cold backup, not application-consistent in the live sense.
- D) NGT and the platform handle this natively; external products may use the same NGT integration but are not required.

**The trap:** A is the default-mental-model trap. Snapshot consistency is a configuration, not a default behavior.

---

**Q2.** Which of the following correctly describes the relationship between Protection Domains and Protection Policies?
**Cert relevance:** NCP-MCI

A) Protection Policies replaced Protection Domains; PDs are no longer supported
B) Protection Domains are configured in Prism Element with manual VM membership; Protection Policies are configured in Prism Central with category-driven membership; both are supported and interoperate
C) They are the same construct under different names
D) Protection Domains are for snapshots; Protection Policies are for replication

**Answer:** B

**Why this answer:** PDs (PE-based, manual) and Policies (PC-based, category-driven) are both supported. Policies are recommended for new deployments; PDs continue to work for existing deployments.

**Why not the others:**
- A) PDs continue to be supported.
- C) They are distinct constructs with different membership models and management surfaces.
- D) Both can do snapshots and replication.

**The trap:** A is tempting if you assume "newer must have replaced older." Nutanix maintains both for compatibility with existing deployments.

---

**Q3.** What is the minimum typical RPO for Async replication?
**Cert relevance:** NCP-MCI · NCM-MCI

A) 0 (synchronous)
B) 20 seconds
C) 1 hour, with 15-minute possible in some configurations
D) 24 hours

**Answer:** C

**Why this answer:** Async replication's typical minimum RPO is 1 hour, configurable down to 15 minutes in some scenarios. Async is the default for general-purpose DR.

**Why not the others:**
- A) That is Metro Availability.
- B) That is NearSync's territory.
- D) Async can certainly do 24 hours, but the minimum is much shorter.

**The trap:** B is the seductive answer for someone who confuses Async and NearSync. Memorize: Async = 1 hour typical (15 min minimum); NearSync = 20 seconds to 15 minutes; Metro = 0.

---

**Q4.** Metro Availability requires which of the following?
**Cert relevance:** NCP-MCI · NCP-NS

A) WAN-distance separation between sites for geographic resilience
B) Sub-5ms RTT between sites and a Witness VM at a third site
C) Identical hardware at both sites
D) NC2 (Nutanix Cloud Clusters) deployment

**Answer:** B

**Why this answer:** Metro is synchronous, so it requires very low latency (typically <5ms RTT, metro-area distance). Witness VM at a third site provides quorum during partition events.

**Why not the others:**
- A) WAN distance is incompatible with synchronous replication; Metro is metro-area only.
- C) Hardware compatibility is general but not specifically a Metro requirement.
- D) NC2 is unrelated to the Metro requirement.

**The trap:** A reflects a misunderstanding of Metro's purpose. Metro is for short-distance, zero-RPO requirements. Long-distance DR uses Async or NearSync.

---

**Q5.** A customer needs DR with 1-minute RPO for a critical SQL Server. Which replication mode should you recommend?
**Cert relevance:** NCP-MCI · sales-relevant

A) Async with frequent intervals
B) NearSync replication
C) Metro Availability
D) Manual snapshot copying every minute

**Answer:** B

**Why this answer:** NearSync's RPO range (20 seconds to 15 minutes) fits the 1-minute target. It does not require Metro's strict latency budget but provides much tighter RPO than Async.

**Why not the others:**
- A) Async typically tops out at 15-minute minimum RPO; 1-minute is below its operational range.
- C) Metro provides zero RPO but requires <5ms latency, which is not specified or required for 1-minute RPO. Metro is overkill and constrains the deployment.
- D) Not a real option for any production deployment.

**The trap:** C is the temptation to "use the strongest option." Metro's constraints (latency, witness, cost) are real and unwarranted for a 1-minute RPO that NearSync can meet.

---

**Q6.** A customer running SRM on ESXi for DR is moving to Nutanix. What is the recommended approach?
**Cert relevance:** sales-relevant

A) Force migration to Recovery Plans on day one
B) Continue running SRM on ESXi-on-Nutanix; over time, evaluate migration to Recovery Plans for AHV-targeted workloads or new deployments; coexist for the foreseeable future if SRM functionality is integral
C) Replace SRM with PowerShell scripts
D) Use Recovery Plans to manage SRM remotely

**Answer:** B

**Why this answer:** This is the operationally correct approach. SRM continues to work on ESXi-on-Nutanix. Recovery Plans is recommended for AHV deployments and new workloads. Migration is a project, not a single event.

**Why not the others:**
- A) Forced migration ignores the customer's installed base and operational continuity.
- C) Scripts are not a replacement for an orchestration product.
- D) Recovery Plans does not manage SRM.

**The trap:** A reflects the impulse to consolidate immediately. The right answer respects existing investment and migrates at the customer's pace.

---

**Q7.** What does NC2 provide for DR scenarios?
**Cert relevance:** NCP-MCI · sales-relevant

A) A free DR site with no operational cost
B) The ability to run Nutanix on AWS or Azure bare-metal infrastructure, replicate from on-premises Nutanix to NC2, and fail over to cloud capacity without building a second datacenter
C) A backup software product
D) A vSphere Replication alternative

**Answer:** B

**Why this answer:** NC2 (Nutanix Cloud Clusters) runs the Nutanix platform on cloud bare-metal hardware (AWS or Azure). It enables DR-to-cloud without a second physical datacenter, with the same operational platform on both sides.

**Why not the others:**
- A) NC2 has real costs (cloud bare-metal pricing, bandwidth, etc.); not free.
- C) NC2 is not a backup product; it is infrastructure.
- D) NC2 is the destination for replication, not a replication product.

**The trap:** A reflects a marketing-flavored mental model. NC2 has costs but the economics often beat building physical DR.

---

**Q8.** A customer has a 200-VM environment, mixed Tier-1 (databases, 30 VMs), Tier-2 (apps, 70 VMs), and Tier-3 (general-purpose, 100 VMs). Their DR requirements are:
- Tier-1: 5-minute RPO, 30-minute RTO
- Tier-2: 1-hour RPO, 2-hour RTO
- Tier-3: 4-hour RPO, 4-hour RTO

What replication architecture do you recommend?
**Cert relevance:** NCP-MCI · NCM-MCI · sales-relevant

A) Metro for everything
B) NearSync for Tier-1, Async (1-hour interval) for Tier-2, Async (4-hour interval) for Tier-3, with appropriate Recovery Plans for each tier
C) Manual snapshots for all tiers
D) NC2 only

**Answer:** B

**Why this answer:** Match the replication mode to the RPO target. NearSync handles 5-minute RPO for Tier-1. Async handles the longer RPOs for Tier-2 and Tier-3 with appropriate intervals. Recovery Plans orchestrate failover for each tier.

**Why not the others:**
- A) Metro is overkill (and infeasible at WAN distances) for most workloads. Tier-3's 4-hour RPO does not need Metro.
- C) Manual snapshots are not a DR strategy.
- D) NC2 is a destination option, not a replication mode answer; the question asks about replication architecture.

**The trap:** A demonstrates the "use the strongest option" failure mode. Right-sizing the protection mode to RPO/RTO requirements is the SA-chair discipline.

---

**Q9.** Which of the following is correct about test failover in Recovery Plans?
**Cert relevance:** NCP-MCI · NCM-MCI

A) Test failover is impossible in Nutanix; only real failovers can be performed
B) Test failover runs the failover sequence into an isolated network at the DR site, allowing validation without disrupting production; teardown returns the cluster to normal protection state
C) Test failover requires powering down the production VMs
D) Test failover is the same as a real failover from a system perspective

**Answer:** B

**Why this answer:** This is exactly what test failover is designed for: validate the runbook, in an isolated network, without affecting production. The platform isolates DR-site VMs from the production network during the test.

**Why not the others:**
- A) Test failover is a core platform feature.
- C) Production VMs continue running normally; the test happens at the DR site.
- D) Test and real failovers differ specifically in network isolation and the post-test teardown step.

**The trap:** C is the misconception that any DR action affects production. Recovery Plans were specifically designed to enable testing without disruption.

---

**Q10.** Which DR feature is NOT included in Nutanix's baseline platform (without Pro/Ultimate NCM tiers)?
**Cert relevance:** NCP-MCI · sales-relevant

A) Native DSF snapshots
B) Async replication via Protection Domains
C) Recovery Plans / Leap orchestration with advanced runbook features
D) Crash-consistent snapshot scheduling

**Answer:** C

**Why this answer:** Recovery Plans / Leap with advanced orchestration features sits at NCM Pro tier or higher. Basic replication, snapshots, and Protection Domains are part of the platform baseline.

**Why not the others:**
- A) DSF snapshots are core to the platform.
- B) Async replication via PDs has been included for many AOS versions.
- D) Snapshot scheduling is included.

**The trap:** A and B sound like they should be tier-gated, but they are platform baseline. Recovery Plans' advanced features are the licensed add-on.

---

**Q11.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep · NCM-MCI prep · sales-relevant
**Format:** Short-answer / design discussion. Write your reasoning. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A customer is consolidating their VMware environment onto Nutanix. They have:
- Two physical datacenters in the same metro area (50km apart, 2-3ms RTT, 10 GbE link between them)
- One co-located DR datacenter 800km away (35-40ms RTT, 1 Gbps WAN)
- 1,200 VMs total: ~50 mission-critical (zero-RPO requirement, financial systems), ~250 Tier-1 production (5-15 min RPO target, mostly databases), ~900 general-purpose (4-hour RPO acceptable)
- Existing investment in SRM with about 300 VMs orchestrated through it
- Compliance mandate for monthly DR test attestations

The customer asks you to design the data protection architecture, including replication mode selection per tier, Protection Policy structure, Recovery Plan design, the SRM transition approach, and DR test cadence.

**The challenge:**
Walk through your design. Cover replication architecture, policy design, orchestration, transition, and operational rhythm. Identify what you still need to know.

**A strong answer covers:**
- **Multi-site architecture:**
  - Datacenter A and B (metro pair): Metro Availability between them for the 50 mission-critical VMs (zero RPO, 2-3ms RTT well within Metro's 5ms budget)
  - Witness VM hosted at the third (DR) datacenter to provide quorum for the metro pair
  - DR datacenter (800km): Async replication target for all tiers, NearSync for Tier-1 if bandwidth permits (verify 1 Gbps WAN can sustain Tier-1 change rate)
- **Protection Policy structure (in Prism Central):**
  - Policy 1 "Mission-Critical-Metro": match VMs with `Tier: Mission-Critical`, Metro replication between A and B, hourly snapshots retained 14 days, replication to remote DR via Async
  - Policy 2 "Tier1-NearSync": match VMs with `Tier: Tier1`, NearSync to DR datacenter (verify capacity), 5-minute RPO target
  - Policy 3 "GeneralPurpose-Async": match VMs with `Tier: General`, Async replication to DR datacenter at 4-hour interval
- **Recovery Plans:**
  - One Recovery Plan per tier, defining startup order (DB → App → Web), network mappings (DC A VLANs to DR DC VLANs), IP remapping or DHCP reassignment, pre/post checks
  - Test failover schedule: monthly for mission-critical (compliance-driven), quarterly for Tier-1, semi-annual for general-purpose
- **SRM transition:**
  - SRM continues to run on ESXi-on-Nutanix for the 300 currently-orchestrated VMs (no forced migration)
  - As workloads migrate to AHV (Module 3 / 10 territory), they move to Recovery Plans
  - Plan a 12-18 month phased transition, prioritizing simpler workloads first
- **Bandwidth math:** verify 1 Gbps WAN supports sustained replication for the workload mix. NearSync for 250 VMs requires careful sizing; if bandwidth is constrained, drop to Async with shorter intervals (15 min) for Tier-1
- **DR test cadence:**
  - Monthly: mission-critical full failover test (compliance attestation)
  - Quarterly: Tier-1 test
  - Semi-annual: general-purpose test (rotating subset)
  - Document each test result in the audit log and customer's compliance documentation
- **Operational considerations:**
  - Witness VM availability is critical; design it for resilience
  - Categories drive policy enrollment, so category hygiene matters; new VMs must be properly tagged at creation
  - Use Prism Central RBAC to give the DR team appropriate access without giving them broader admin rights
- **What you still need to know:**
  - WAN bandwidth utilization profile: is 1 Gbps fully available for replication, or shared?
  - Specific compliance framework for the monthly attestation (PCI DSS? SOX?)
  - Application dependencies for startup order (which databases must come up before which apps?)
  - SRM's current runbook complexity: are there scripted callouts that don't translate cleanly to Recovery Plans?
  - Does the customer want NC2 considered as an additional or alternative DR option?

**A weak answer misses:**
- Defaulting to Metro for all mission-critical (correct) without acknowledging Witness VM placement
- Forgetting to plan Tier-1 NearSync bandwidth feasibility on a 1 Gbps WAN
- Forced SRM migration timeline rather than coexistence
- Missing the test cadence (compliance-driven monthly is the customer's specific requirement)
- Not naming category hygiene as an operational requirement

**Why this question matters for NCX:** This is the kind of multi-site DR design that NCX panels probe. The right answer integrates replication topology, orchestration, transition strategy, operational rhythm, and identifies the constraints that need to be validated. Pure-feature answers fail.

---

**Q12.** *(NCX-style architectural defense, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / architectural defense. Write your response.

**The scenario:**
You are in front of a customer's senior DR architect, who has run SRM-based DR for 14 years. He says:

> *"SRM has 14 years of runbook customization, deep VMware integration, mature reporting, and a clear escalation path with VMware. Recovery Plans is younger, less feature-rich, and tied to Nutanix. Why would I move my proven DR practice to a less mature product?"*

**The challenge:**
Respond. He is making a real argument. Address it.

**A strong answer covers:**
- **Acknowledge SRM's maturity directly.** SRM is mature. Recovery Plans is younger. Pretending otherwise loses credibility.
- **Reframe the comparison precisely:**
  - **Maturity vs integration.** SRM's maturity is real. Recovery Plans' integration is also real: it is part of the platform you are evaluating, with no separate purchase or deployment. The comparison is not "mature vs less mature in isolation" but "mature standalone vs integrated platform feature."
  - **Runbook customization.** SRM has more advanced scripted-callout customization. For typical enterprise DR, this is rarely the differentiator. For your specific runbook complexity, we need to walk through what customization you actually use; if it is simple ordering and IP remapping, Recovery Plans handles it; if you have complex scripted callouts, we map those carefully.
  - **VMware integration.** SRM's tight VMware integration is a feature when the entire stack is VMware. As you move workloads to AHV, the integration value diminishes (the workloads are no longer VMware). For ESXi-on-Nutanix, SRM continues to work; you don't lose the integration where it matters.
  - **Reporting maturity.** SRM has more reporting depth. Prism's DR reporting is improving. For compliance-driven reporting, both can typically meet requirements; for nuanced operational dashboards, SRM has more polish today.
  - **Escalation path with vendor.** This is real. VMware's DR support is mature. Nutanix's DR support is also mature; the support quality conversation is a separate one and can be evaluated through reference customers.
- **Reframe the migration question:**
  - "You don't have to migrate. SRM continues to work on ESXi-on-Nutanix; your existing investment is preserved. New workloads on AHV use Recovery Plans. You evaluate Recovery Plans on its merits over time, not under migration pressure."
  - "The question is not 'rip and replace.' The question is 'what do you do for new AHV workloads?' For those, Recovery Plans is the integrated, included answer. SRM continues to do what it does well."
- **Offer a concrete validation step:**
  - "Run a test failover with Recovery Plans on a non-critical AHV workload. Compare the experience to SRM. The comparison will be informed by your hands-on experience, not by feature comparison decks. We can scope a 60-day Recovery Plans evaluation alongside your existing SRM."
- **Close with the durable framing:**
  - "I am not here to replace what works. I am here to integrate Recovery Plans where it makes sense and respect SRM where it makes sense. The right answer is probably hybrid for the next 12-24 months, with SRM continuing to handle the workloads it orchestrates today and Recovery Plans handling new AHV workloads."

**A weak answer misses:**
- Claiming Recovery Plans matches SRM in every dimension.
- Dismissing the architect's 14 years as outdated.
- Forcing a migration timeline.
- Not naming the coexistence pattern as the durable answer.
- Not offering the hands-on evaluation step.

**Why this question matters for NCX:** Senior DR architects with deep SRM history are common in enterprise customers. The skill being tested is acknowledging the real expertise, naming the real gaps, and reframing to a coexistence pattern rather than a forced migration. This is also the disposition that wins enterprise DR conversations.

---

## What You Now Have

You can distinguish crash-consistent from application-consistent snapshots and know when each is appropriate. You know NGT's role in providing VSS coordination on Windows.

You know the difference between Protection Domains (legacy, Prism Element, manual membership) and Protection Policies (modern, Prism Central, category-driven). You can recommend Policies for new deployments while respecting existing PDs.

You have the three replication modes mapped to RPO and operational characteristics: Async (1-hour typical, WAN-friendly, low overhead), NearSync (sub-15-minute RPO, low-latency required, moderate-high overhead), Metro (zero RPO, <5ms RTT, witness required, highest cost).

You can map application tiers to the right replication mode in 15 minutes. The matrix is in your hands: Tier-0 mission-critical → Metro + Async to remote, Tier-1 → NearSync, Tier-2 → Async hourly, Tier-3 → Async less-frequent.

You have Recovery Plans (Leap) as the SRM equivalent: orchestrated failover, network mapping, IP remapping, startup order, test failover capability, all integrated into Prism Central.

You can compare Recovery Plans to SRM honestly: SRM is more mature for advanced runbook customization; Recovery Plans is integrated and capable for typical use; coexistence is the durable answer for established SRM customers.

You have NC2 as the cloud DR option that eliminates the need for a second datacenter. The economics often beat physical DR for mid-market customers.

You know test failover is the feature customers neglect and the durable BlueAlly value: quarterly test failovers in 1-2 hours each, instead of multi-day fire drills with disrupted production.

You have twelve practice questions worth of DR discrimination, including two NCX-style design defenses (multi-site DR architecture with mixed tiers and SRM transition, and the architectural defense against a 14-year SRM veteran).

You are now ready for the unified storage layer. Module 8 covers Files, Objects, and Volumes: the storage services that sit on top of DSF and replace separate file storage, object storage, and iSCSI block targets the customer is currently buying as separate appliances.

---

## Cross-References

- **Previous:** [Module 6: Networking and Microsegmentation](./06-networking-flow.md)
- **Next:** [Module 8: Unified Storage (Files, Objects, Volumes)](./08-unified-storage.md)
- **Glossary:** [Crash-consistent snapshot](./appendix-a-glossary.md#crash-consistent-snapshot) · [Application-consistent snapshot](./appendix-a-glossary.md#application-consistent-snapshot) · [Protection Domain](./appendix-a-glossary.md#protection-domain) · [Protection Policy](./appendix-a-glossary.md#protection-policy) · [Async Replication](./appendix-a-glossary.md#async-replication) · [NearSync](./appendix-a-glossary.md#nearsync) · [Metro Availability](./appendix-a-glossary.md#metro-availability) · [LWS](./appendix-a-glossary.md#lws) · [Recovery Plan](./appendix-a-glossary.md#recovery-plan) · [Witness VM](./appendix-a-glossary.md#witness-vm) · [NC2](./appendix-a-glossary.md#nc2) · [SRM](./appendix-a-glossary.md#srm) · [RPO](./appendix-a-glossary.md#rpo) · [RTO](./appendix-a-glossary.md#rto)
- **Comparison Matrix:** [Replication Modes Row](./appendix-b-comparison-matrix.md#replication-modes) · [DR Orchestration Row](./appendix-b-comparison-matrix.md#dr-orchestration) · [Cloud DR Row](./appendix-b-comparison-matrix.md#cloud-dr)
- **Objections:** [#26 "What about SRM?"](./appendix-d-objections.md#obj-026) · [#27 "We have array-based replication"](./appendix-d-objections.md#obj-027) · [#28 "DR is too complex to migrate"](./appendix-d-objections.md#obj-028) · [#29 "Cloud DR isn't for us"](./appendix-d-objections.md#obj-029) · [#30 "Test failover is too disruptive"](./appendix-d-objections.md#obj-030)
- **Discovery Questions:** [Q-DR-01 RPO/RTO targets per tier](./appendix-e-discovery-questions.md#q-dr-01) · [Q-DR-02 Existing DR infrastructure](./appendix-e-discovery-questions.md#q-dr-02) · [Q-DR-03 SRM footprint](./appendix-e-discovery-questions.md#q-dr-03) · [Q-DR-04 DR test cadence and history](./appendix-e-discovery-questions.md#q-dr-04) · [Q-DR-05 Compliance / regulatory drivers](./appendix-e-discovery-questions.md#q-dr-05)
- **Sizing Rules:** [Replication bandwidth math](./appendix-f-sizing-rules.md#replication-bandwidth) · [NearSync cluster overhead](./appendix-f-sizing-rules.md#nearsync-overhead) · [Metro witness placement](./appendix-f-sizing-rules.md#metro-witness)
- **CLI Reference:** [Protection commands](./appendix-g-cli-reference.md#protection) · [Recovery Plan commands](./appendix-g-cli-reference.md#recovery-plans)
- **Competitive Matrix:** [Recovery Plans vs SRM](./appendix-h-competitive-matrix.md#dr-orchestration)
- **Reference Architectures:** [Multi-site DR design](./appendix-i-reference-architectures.md#ra-dr) · [Cloud-DR with NC2](./appendix-i-reference-architectures.md#ra-nc2-dr)
- **POC Playbook:** [DR demo flow](./appendix-j-poc-playbook.md#dr-demo) · [Test failover demo](./appendix-j-poc-playbook.md#test-failover)

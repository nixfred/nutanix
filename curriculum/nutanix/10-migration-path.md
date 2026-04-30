---
module: 10
title: Migration Path (The Customer-Running Playbook)
estimated_reading_time: 36 min
prerequisites:
  - All prior modules (01 through 09)
  - Real-world VMware migration experience (or comparable cross-platform migration experience)
  - Comfort leading multi-month customer engagements
  - Familiarity with project management vocabulary (waves, milestones, risk register, RACI)
key_terms:
  - Move (Nutanix migration tool)
  - Pilot wave
  - Production wave (Wave 1, 2, 3)
  - Cutover
  - Parallel-running
  - Hybrid steady-state
  - Risk register
  - Dependency mapping
  - Application affinity group
  - Validation criteria
  - Rollback plan
  - Decommissioning
diagrams:
  - migration-framework-phases
  - move-tool-architecture
  - hybrid-steady-state
cert_coverage:
  NCA: ~5%
  NCP-MCI: ~10% (Move tool, migration concepts)
  NCM-MCI: ~5%
  note: Synthesis module; sales- and engagement-relevant; lighter direct cert weight
sa_toolkit:
  related_objections: [obj-041, obj-042, obj-043, obj-044, obj-045]
  related_discovery: [q-mig-01, q-mig-02, q-mig-03, q-mig-04, q-mig-05, q-mig-06]
---

# Module 10: Migration Path (The Customer-Running Playbook)

> **Cert coverage:** NCA (~5%) · NCP-MCI (~10%) · NCM-MCI (~5%)
> **SA toolkit:** Objections #41 through #45 · Discovery Q-MIG-01 through Q-MIG-06
> **This is the synthesis module.** Modules 1-9 give you the technical and economic depth. This module is what BlueAlly SAs actually do for 12-18 months: lead the customer's migration from VMware to Nutanix.

---

## The Promise

By the end of this module you will:

1. **Run a 12-18 month customer migration engagement end to end.** Discovery, dependency mapping, platform build, pilot wave, production waves, cutover, decommissioning. The shape is consistent across customers; the specifics vary. You should be able to walk into a customer kickoff and lead the conversation.
2. **Use Move (the Nutanix migration tool) confidently.** What it migrates, what it doesn't, what to plan for, what tends to require manual cleanup.
3. **Sequence migrations by risk, not by convenience.** Pilot first (low-risk, internal), then low-tier production, then Tier-1, then mission-critical. Many customer projects fail because they invert this sequence; BlueAlly SAs lead customers to the right order.
4. **Design the hybrid steady-state honestly.** Some workloads will stay on VMware for legitimate reasons (NSX-T routing complexity, NetApp ONTAP-specific workflows, SRM-orchestrated complex DR, third-party-vendor dependencies). The right answer for many customers is hybrid permanently. Position this as a successful outcome, not a failure.
5. **Communicate with executives, IT teams, and end users at the right level for each.** Migration is as much a communication exercise as a technical one. The cadence and framing differ for each audience; you should know all three.
6. **Maintain a working risk register and act on it.** Migrations have predictable risks (dependency surprises, application performance regressions, network reconfiguration issues, team capacity constraints). The risk register is how you stay ahead of them.
7. **Know what year-2 stable state looks like** and have the conversation about BlueAlly's continuing role: managed services, upgrades, expansion, periodic optimization. Migrations end; relationships continue.

This is the final module. After this, the curriculum hands you the appendices (glossary, comparison matrix, objections, discovery questions, sizing rules, CLI reference, competitive matrix, reference architectures, POC playbook, certification tracker). The technical and commercial depth is in your hands. What remains is the engagement skill, which this module gives you a framework for.

---

## Foundation: What You Already Know

You have done migrations. Maybe vCenter version upgrades. Maybe storage-array swaps with Storage vMotion. Maybe a datacenter consolidation with cross-site moves. Possibly a hypervisor migration (Hyper-V to ESXi, or vice versa). You know the patterns:

- **Discovery is more time than you budget for.** Customers don't have current inventories. Dependency surprises are the rule, not the exception.
- **Pilot waves expose the real problems.** Theory dies in contact with the customer's actual environment.
- **Mission-critical workloads come last.** Practice on dev/test, build confidence, then move toward Tier-1.
- **Communication failures cause more pain than technical failures.** End users blindsided by an outage will not forgive you for being technically right.
- **The team gets tired.** Migrations are marathons. Plan for fatigue.
- **Cutover is the moment, not the project.** The 18 months around cutover are the project. The cutover itself is brief if everything else is right.

You also know what the VMware-to-Nutanix migration is not:

- **It is not Storage vMotion.** That works between VMFS datastores; it does not cross hypervisor boundaries.
- **It is not vMotion.** vMotion crosses ESXi hosts within a vSphere cluster, not between vSphere and AHV.
- **It is not a backup-and-restore operation.** Backup-and-restore is a fallback path, not the primary tool.
- **It is not a single weekend's work.** Realistic migrations take 6-18 months for typical enterprise environments.

The migration tool is **Move**. The framework is the phased-wave approach. The discipline is sequencing by risk. The communication is to multiple audiences with different framings. Hold all of those in mind. We'll build them out.

> [!FROM-THE-SA-CHAIR]
> The single most common BlueAlly SA mistake on migrations is letting the customer dictate the sequence. A customer who says "let's move our most critical workload first to prove value" is asking for a project failure. The right response: *"I understand the impulse, and I respect the urgency to demonstrate value. The discipline that produces successful migrations is to start with low-risk workloads, validate the platform and our processes, then move toward Tier-1. We demonstrate value within 60 days through the pilot results. Mission-critical comes after we've done it 100 times in your environment, not as the proof point."* That sentence saves projects.

---

## Core Content

### Move: The Migration Tool

**Move** is Nutanix's cross-platform VM migration tool. It supports several source-to-target paths:

- **VMware ESXi to AHV.** The primary use case for customers consolidating from VMware. Move connects to vCenter, accesses source VMs, migrates them to a target Nutanix AHV cluster.
- **Hyper-V to AHV.** Less common but supported. Useful for customers consolidating mixed VMware + Hyper-V environments.
- **AWS EC2 to NC2 (or to on-prem AHV).** For customers moving cloud workloads.
- **AWS EC2 to AHV on-prem.** Cloud repatriation.
- **Azure to NC2.**
- **AHV to AHV** across clusters (utility for cross-cluster migrations).

**The architecture:**

- A **Move appliance** (a VM, deployed on the target Nutanix cluster typically) coordinates the migration.
- An agent (often agentless on ESXi) connects to the source environment.
- The Move appliance reads source VM data (disks, configuration, network attachments).
- During migration, Move performs initial replication of VM data to the target cluster, then incremental sync of changes, and finally a brief cutover (the source VM is shut down, final delta is replicated, target VM starts).
- Cutover downtime is typically 5-30 minutes per VM, depending on size and final delta. For most workloads, this fits within scheduled maintenance windows.

**What Move handles automatically:**

- VM disk migration (the bulk of the data).
- VM configuration translation (CPU, RAM, NICs in best-effort).
- Power state preservation (VM running on target after migration).
- Some network mapping (Move asks you to map source port groups to target virtual networks).
- VMware Tools removal and NGT (Nutanix Guest Tools) installation in some scenarios.

**What requires manual handling:**

- **Complex networking.** Distributed switches with advanced port-group policies often require manual reconfiguration.
- **Application-specific configurations.** SQL Server cluster configurations, Oracle ASM disk layouts, anything that depends on storage paths or hypervisor-specific features.
- **NSX-T microsegmentation.** Requires re-implementing on Flow (or coexistence with NSX-T).
- **Custom guest OS modifications.** Customizations that depend on VMware Tools or VMware-specific drivers may need attention.
- **Snapshots and backups.** Old snapshot chains in VMware do not migrate cleanly; consolidate or accept that historical snapshots do not transfer.
- **Storage policies.** vSAN-specific storage policies (FTT, stripe count) don't have direct equivalents in DSF. The DSF replacement (RF, EC, compression, dedup) is configured at the Storage Container level.

> [!FAMILIAR]
> Move is conceptually similar to VMware Site Recovery Manager (with vSphere Replication) for cross-site VM migration, or to other cross-platform migration tools like Zerto, RackWare, or AWS Application Migration Service. The pattern (source replication, delta sync, brief cutover) is industry-standard. The implementation is Nutanix-specific. If you have run cross-platform migrations before, the muscle memory transfers.

> [!DIFFERENT]
> Move is not vMotion. vMotion is live VM migration between ESXi hosts within a vSphere cluster: no perceptible downtime, milliseconds of pause at most. Move handles cross-platform migration with a brief planned cutover (typically 5-30 minutes per VM). The cutover is scheduled within a maintenance window, not transparent to applications. Customers (and SAs) sometimes assume any migration tool works like vMotion; correcting this expectation early prevents disappointment. The right framing: *"Move is closer to backup-and-restore-with-careful-coordination than to vMotion. Each VM has a brief planned cutover. We schedule them in waves."*

> [!ON-THE-EXAM] **NCP-MCI**
> Move is testable at a conceptual level. Memorize: Move handles VM disk migration, configuration translation, and brief cutover; it does not handle NSX-T policies, complex application-specific configurations, or vSAN-specific storage policies (those require manual or scripted handling on the target). Trap distractor: "Move migrates the entire environment in one operation" (false; Move migrates per-VM or per-batch).

---

### Diagram: Move Tool Architecture

**id:** `move-tool-architecture`
**type:** flow
**caption:** Move appliance reads source VMs, replicates to target Nutanix cluster, syncs deltas, cuts over with minimal downtime.
**exam_relevance:** [NCP-MCI]
**whiteboard_ready:** true

**Elements:**
- Left side: Source environment
  - vCenter (gray) connected to ESXi hosts
  - VMs running on ESXi hosts
- Center: Move appliance (gold) running on the target Nutanix cluster
  - Connects to source vCenter for VM inventory and access
  - Has a queue of VMs to migrate
- Right side: Target Nutanix cluster
  - AHV hosts (rust)
  - Migrated VMs running on AHV
- Flow shown by arrows:
  - Discovery: Move queries vCenter for VM list, sizing, dependencies
  - Initial replication: VM disk data flows from source to target (large data transfer, takes time)
  - Incremental sync: changes since initial replication flow continuously
  - Cutover: source VM shut down, final delta synced, target VM started (5-30 minutes typical)

**Connections:**
- Move appliance to vCenter: API connection
- Move appliance to source ESXi: VM data access
- Move appliance to target Nutanix cluster: native (Move runs on target)
- Source VM (during cutover) to target VM: brief data sync, then handoff

**Annotations:**
- "Move handles VM-level migration. Network reconfiguration and application-specific items often require manual planning."
- "Initial replication takes hours to days for large environments; incremental syncs are quick."
- "Cutover downtime: 5-30 minutes per VM typical. Schedule per maintenance windows."
- "Migrations are batched into waves. Don't try to migrate everything at once."

**Why this diagram exists:** Customers and BlueAlly SAs alike treat Move as "magic." This diagram shows the actual mechanics. Whiteboard it during the migration kickoff. Customers who see the architecture trust the process.

---

### The 7-Phase Migration Framework

Successful migrations follow a phased structure. Each phase has clear entry and exit criteria.

**Phase 0: Discovery and Dependency Mapping (4-8 weeks)**

Inventory everything. Map dependencies. Classify workloads by risk and tier. Identify integration points (AD, network, monitoring, backup, DNS, load balancers, ITSM).

Deliverables:
- VM inventory with: name, OS, vCPU/RAM, storage, network attachments, dependencies, owning team, business tier (Tier-0 / Tier-1 / Tier-2 / Tier-3 / Dev), RPO/RTO requirements, compliance flags
- Application affinity groups (which VMs talk to which other VMs)
- Network dependency map (firewalls, load balancers, external services)
- Storage classification (block / file / object usage)
- Customer's existing tooling inventory (backup, monitoring, ITSM)

Common discovery techniques:
- Active discovery tools (Nutanix's discovery utility, Lansweeper, RVTools for VMware)
- Application Performance Monitoring data (if available)
- Workshops with application owners (slow but high-value)
- Network flow analysis (NetFlow, application dependency mapping tools)

This phase often takes longer than expected. Budget aggressively.

**Phase 1: Platform Build and Validation (4-6 weeks)**

Deploy the Nutanix cluster. Validate it. Integrate with customer's environment.

Deliverables:
- Nutanix cluster deployed and validated against POC criteria
- AD / SAML / SSO integration
- Network integration (VLANs, routing, firewalls)
- Backup integration (Veeam / Commvault / Rubrik against Nutanix Objects or vendor-specific)
- Monitoring integration (Prism Central forwarding to customer's SIEM, ITSM)
- DR site configured if applicable (Module 7)
- Categories defined (from Module 4 - drives policies in subsequent waves)

**Phase 2: Pilot Wave (4-8 weeks)**

Migrate 5-20 low-risk VMs. Validate the migration tooling, process, and team capacity. This is where theory meets reality.

Pilot VM selection criteria:
- Internal IT-team-owned (so the customer's team is the audience for any issue)
- Low criticality (no customer-facing impact if something goes wrong)
- Representative of common patterns (Linux + Windows mix, typical applications)
- Ideally including one VM from each major tier of complexity

Pilot wave establishes:
- Move tool installation and operation in the customer's environment
- Network mapping table (source port group to target virtual network)
- Cutover runbook
- Communication template
- Rollback procedures (validated, not theoretical)
- The customer's team's hands-on familiarity with the new platform

**Phase 3: Production Wave 1 (Low-Tier, 8-16 weeks)**

50-200 VMs. Dev/test/UAT environments. General-purpose Tier-3 production. Web tiers behind load balancers (with redundancy that absorbs cutover blips).

This wave is where you scale the team's capacity and validate the operational rhythm. By the end:
- The customer's team can do migrations without BlueAlly hand-holding
- The runbook is mature
- Common issues are identified and have known mitigations
- Confidence is built for the next tier

**Phase 4: Production Wave 2 (Tier-1, 12-20 weeks)**

200-600 VMs typically. Tier-1 production: customer-facing applications, internal-critical applications, databases (with care), core business workflows.

Tier-1 migrations require:
- Application-owner coordination per migration batch
- Clear validation criteria per application
- Rollback plans validated per application
- Cutover scheduled in maintenance windows
- Application-aware migration patterns (e.g., for SQL Server: use Always On to minimize downtime; for Oracle: planned-failover patterns; for clustered apps: failover to standby, migrate primary, fail back)

**Phase 5: Production Wave 3 (Mission-Critical, 8-16 weeks)**

The remaining workloads: Tier-0 mission-critical, often with zero-downtime requirements, complex application dependencies, regulated workloads, integration-intensive systems.

Often this wave includes:
- Workloads that genuinely require Metro Availability or NearSync (Module 7)
- Mission-critical applications with regulatory compliance constraints
- Applications with complex external integrations
- Workloads that may stay on VMware permanently if business case justifies (the hybrid steady-state question)

This wave gets the most engineering scrutiny per workload. Per-VM time investment is highest.

**Phase 6: Stabilization, Decommissioning, and Steady-State (4-8 weeks)**

Decommission old VMware infrastructure (or formalize the hybrid steady-state). Hand off operational ownership to customer team. Document final state. Conduct retrospective.

Deliverables:
- Decommissioning plan executed: physical removal, contract termination, asset disposal
- Hybrid steady-state documentation if applicable
- Final BoM reconciliation
- Operational runbook handed off
- Retrospective with all stakeholders
- BlueAlly's continuing role defined (managed services, upgrades, future expansion)

> [!ON-THE-EXAM] **NCP-MCI · sales-relevant**
> Migration phasing is testable at a conceptual level. Memorize: pilot → low-tier production → Tier-1 → mission-critical → stabilization. The sequence is risk-managed: validate the platform on low-risk workloads, build confidence, scale toward higher-risk last. Trap distractor: "Mission-critical workloads should migrate first to prove value" (false; this is the failure pattern, not the success pattern).

---

### Diagram: The 7-Phase Migration Framework

**id:** `migration-framework-phases`
**type:** timeline
**caption:** A 12-18 month migration broken into seven phases with clear entry/exit criteria. The shape is consistent across customers.
**exam_relevance:** [NCP-MCI, sales-relevant]
**whiteboard_ready:** true

**Elements:**
- Horizontal timeline showing months 0 through 18
- Seven phase blocks with phase numbers, durations, and key deliverables:
  - Phase 0: Discovery & Dependency Mapping (months 0-2)
  - Phase 1: Platform Build & Validation (months 1-3, can overlap with Phase 0)
  - Phase 2: Pilot Wave (months 3-5)
  - Phase 3: Production Wave 1 - Low-Tier (months 4-8)
  - Phase 4: Production Wave 2 - Tier-1 (months 7-13)
  - Phase 5: Production Wave 3 - Mission-Critical (months 12-16)
  - Phase 6: Stabilization & Decommissioning (months 15-18)

- Above each phase: percentage of VMs migrated by phase end (cumulative)
- Below each phase: key risks and dependencies
- Side annotations: communication cadence (kickoff, weekly steering, monthly executive review, milestone sign-offs)

**Annotations:**
- "Phases overlap intentionally. Discovery continues throughout; later waves benefit from earlier wave learnings."
- "Pilot wave is the most important risk-reduction phase. Don't skip it."
- "Communication cadence is independent of phase: weekly steering, monthly executive review, regular application-owner check-ins."
- "Budget 50% more time than you initially estimate for Phase 0 and Phase 4."

**Why this diagram exists:** This is the customer-facing project timeline. Walk through it in the kickoff. Refer to it weekly. Use it to set expectations and to identify when a phase is at risk of slipping. Whiteboard-ready and high-priority for executive presentations.

---

### The Cycle, Frame Two: Migration as Phased Risk Management

For a customer's IT operations leader, the durable migration frame is risk management. Each phase has a specific risk being reduced.

| Phase | Primary Risk Reduced |
|---|---|
| Phase 0: Discovery | "We don't actually know what we have." |
| Phase 1: Platform Build | "The new platform doesn't fit our environment." |
| Phase 2: Pilot | "We don't know what we don't know about migrating." |
| Phase 3: Wave 1 | "We can't migrate at scale." |
| Phase 4: Wave 2 | "Tier-1 workloads might break in ways we can't predict." |
| Phase 5: Wave 3 | "Mission-critical workloads have unique constraints." |
| Phase 6: Stabilization | "We never finish; the project never ends." |

Frame each phase as risk reduction. Customers who think in risk-management terms (which is most operations leaders) respond to this framing. The migration is not about VM movement; it's about systematically reducing risk.

### The Cycle, Frame Three: Migration as the SA's 12-18 Month Engagement

For BlueAlly, the migration is the engagement. The shape:

- **Month 0-2:** Discovery + platform build. Heavy SA engagement. BlueAlly visible weekly.
- **Month 3-5:** Pilot wave. SA is hands-on; the customer's team is learning.
- **Month 6-10:** Production waves. SA shifts to oversight + coaching; the customer's team is executing.
- **Month 11-15:** Tier-1 + mission-critical waves. SA is back hands-on for the highest-risk migrations.
- **Month 16-18:** Stabilization. SA hands off to managed services or to the customer's steady-state team.

The SA's role evolves from technical leader (early) to coach (middle) to specialist consultant (late) to relationship maintainer (after). Recognize the role shift. The customer's team's growing competence is a sign of success, not a threat to your engagement.

> [!FROM-THE-SA-CHAIR]
> The customer's team becoming self-sufficient is the goal, not the threat. The BlueAlly relationship continues post-migration through expansion, optimization, training, additional projects, and managed services. Customers who feel BlueAlly tried to keep them dependent are customers who switch reseller relationships at next refresh. Customers who feel BlueAlly built their team's capability are customers who refer you to peers and consolidate more business with you. The professional discipline is to coach yourself out of the day-to-day role.

### The Cycle, Frame Four: Migration as Platform Consolidation

For an executive sponsor (CIO, CTO, sometimes CFO), the durable frame is platform consolidation. The migration is one part of a broader simplification.

| What Customer Has Today | What Customer Has Year-2 Stable State |
|---|---|
| 3-4 separate storage tiers (filer, object, iSCSI, dedup) | 1 unified-storage cluster |
| 4-6 management products (vCenter, Aria Operations, Aria Automation, vSphere LCM, etc.) | Prism Central |
| 2-3 networking products (vSS/vDS, NSX-T, third-party security) | Prism + Flow + select third-party |
| Multiple refresh cycles | Aligned to cluster refresh |
| Multiple support contracts | 1-2 (Nutanix + hardware OEM) |
| Multiple vendor relationships | Consolidated to 2-3 |

Frame the migration as consolidation, not as VM movement. Executive sponsors care about portfolio simplification, not about Move tool mechanics.

---

### Hybrid Steady-State: The Permanent Coexistence

Some workloads will not migrate. Legitimate reasons:

- **NSX-T routing complexity.** Customers with deep NSX-T deployments (BGP integration, edge services, L2VPN, advanced routing) often keep those workloads on VMware-on-Nutanix or on traditional VMware infrastructure.
- **NetApp ONTAP-specific workflows.** Customers using FlexClone, FlexCache, or other ONTAP-specific features at high volume may keep file workloads on NetApp.
- **SRM-orchestrated complex DR.** Customers with mature SRM deployments and complex runbook customization may continue running SRM-on-ESXi-on-Nutanix.
- **Vendor-specific certifications.** Some applications are vendor-certified only on specific hypervisors. Until the vendor certifies AHV, those workloads stay where they are.
- **Regulatory or compliance constraints.** Some compliance frameworks audit specific platform configurations; changing introduces audit risk that the customer chooses not to take.
- **Application performance characteristics.** Some workloads (rare, but real) perform meaningfully better on specific platforms.

**Hybrid steady-state is not a project failure.** It is often the right architectural answer. The customer migrates the bulk (often 70-90%) to Nutanix, captures most of the consolidation value, and accepts permanent coexistence for the remaining workloads.

The hybrid steady-state has its own design decisions:

- **Network connectivity** between platforms (typically continuous, since applications often span both).
- **Identity integration** (single AD / SAML applied to both).
- **Backup strategy** (often unified through backup vendor; both platforms protected).
- **DR strategy** (potentially separate per platform, or unified if backup-and-restore-based).
- **Operational team structure** (Nutanix-team vs VMware-team, or unified-team that operates both).
- **License optimization** (reduce VMware footprint to actual remaining workload, not legacy quotas).

---

### Diagram: Hybrid Steady-State Architecture

**id:** `hybrid-steady-state`
**type:** architecture
**caption:** A 2-year stable state where 70-90% of workloads run on Nutanix, with specific workloads remaining on VMware for legitimate reasons. Both platforms share identity, networking, backup, and DR.
**exam_relevance:** [sales-relevant]
**whiteboard_ready:** false

**Elements:**
- Left side: Nutanix cluster (rust)
  - Hosts the bulk of VMs (e.g., 800 of 1,000 total VMs)
  - Files for general user shares
  - Objects for backup target
  - Volumes for non-Nutanix consumers
- Right side: VMware cluster (blue)
  - Hosts specific workloads (e.g., 200 of 1,000 VMs) with reasons noted:
    - "NSX-T routing complexity (50 VMs)"
    - "NetApp ONTAP FlexClone workflows (30 VMs)"
    - "SRM-orchestrated complex DR (40 VMs)"
    - "Vendor-certified only on ESXi (80 VMs)"
- Center: Shared services connecting both platforms
  - AD / SAML identity (single source)
  - Network fabric (both platforms on shared VLAN topology with cross-platform routing)
  - Backup vendor (Veeam / Commvault) protecting both
  - DR strategy (per-platform or unified)
  - Monitoring / SIEM (both platforms forwarding)
- Top: Customer's IT operations team
  - Possibly two sub-teams (Nutanix-focused and VMware-focused) or unified
- Bottom: Customer's user and application community
  - Don't see the platform difference; see services

**Connections:**
- Both platforms to AD / SAML
- Both platforms to network fabric
- Both platforms to backup vendor
- Both platforms to monitoring / SIEM
- IT team to both platforms
- Users / applications consume services that may run on either platform transparently

**Annotations:**
- "Hybrid steady-state is a successful outcome, not a project failure."
- "Customer captures consolidation value on the bulk of workloads while keeping VMware where it provides specific value."
- "Operational complexity is real but manageable; some customers run hybrid permanently."
- "License both platforms appropriately to actual workload, not legacy quotas."

**Why this diagram exists:** Customers worry that "incomplete" migration is failure. This diagram normalizes hybrid steady-state as a viable, often-correct end state. Use it in executive conversations early in the engagement to set realistic expectations.

---

### Risk Register: The Predictable Migration Risks

A working risk register tracks identified risks, mitigations, and ownership. Common risks across migrations:

**Technical risks:**
- **Dependency surprises.** Discovery missed an application dependency; the migration breaks something downstream. Mitigation: thorough Phase 0 + pilot wave validation + application-owner coordination per batch.
- **Application performance regression.** A workload runs slower on the new platform than expected. Mitigation: pre-migration baseline metrics + post-migration validation criteria + rollback plan.
- **Network reconfiguration issues.** Subnets, VLANs, firewall rules don't translate cleanly. Mitigation: detailed network mapping in Phase 0 + dry runs.
- **Storage migration delays.** Initial replication takes longer than estimated; cutover slips. Mitigation: realistic bandwidth assessment + parallel-running periods that absorb delays.
- **Tool failures.** Move encounters edge cases; specific VMs don't migrate cleanly. Mitigation: per-VM rollback plans + manual migration alternatives + support escalation paths.

**Operational risks:**
- **Team capacity constraints.** Customer's team is too thin to absorb migration work alongside operations. Mitigation: realistic capacity planning + BlueAlly augmentation + phasing that respects team bandwidth.
- **Runbook drift.** Migration runbook becomes stale as environment changes during migration. Mitigation: regular runbook reviews + version control + dedicated runbook owner.
- **Communication failures.** Stakeholders blindsided by migrations or outages. Mitigation: communication cadence + named communication owner + structured templates.

**Commercial risks:**
- **Scope creep.** Customer adds workloads or requirements mid-engagement. Mitigation: change-control process + scope reviews per phase + clear definitions of "in scope" vs "follow-on engagement."
- **Timeline slippage.** Phases run long; customer renegotiates contract. Mitigation: realistic phase budgets + early escalation + transparent status reporting.
- **License optimization missed.** Customer continues paying for VMware footprint that isn't being used. Mitigation: license consumption tracking + decommissioning aligned to migration progress.

**Relationship risks:**
- **Executive sponsor turnover.** The CIO who approved the project leaves; the new sponsor questions the rationale. Mitigation: strong written justification + executive briefings throughout + tangible mid-project value demonstration.
- **Customer team morale.** Migration fatigue sets in around month 9-12. Mitigation: celebrate wins + visible progress milestones + manage workload pacing.

The risk register is reviewed weekly with the customer's project lead and monthly with the executive sponsor. New risks are added as discovered. Closed risks are documented. The register is a living document.

---

### Communication Playbook by Audience

Migrations have multiple audiences, each with different information needs and frequencies.

**Executive sponsor (monthly executive review):**
- Format: 30-minute meeting with a one-page status summary
- Content: progress against milestones, top 3 risks, key decisions needed, financial update (capex/opex tracking against budget)
- Tone: business outcomes, not technical detail
- Goal: maintain executive support, surface decisions early

**Project steering committee (weekly):**
- Format: 60-minute working session with detailed status
- Content: phase progress, wave-by-wave status, risk register review, change requests, resource needs
- Tone: project management, action-oriented
- Goal: maintain project momentum, resolve blockers, coordinate cross-team work

**Application owners (per migration batch):**
- Format: pre-migration coordination, day-of-cutover communication, post-migration validation
- Content: what's migrating, when, what to expect, what to validate, who to call if issues
- Tone: respectful of their expertise, clear about responsibilities
- Goal: smooth cutover, validated outcomes, confident application owner

**End users (per migration with user impact):**
- Format: notification email + status page + helpdesk readiness
- Content: when (planned downtime if any), what changes (likely nothing visible), what to do if issues
- Tone: minimize concern, clear instructions
- Goal: avoid surprise, manage expectations

**IT operations team (continuous):**
- Format: shared documentation, daily standup during active migration windows, real-time chat
- Content: technical status, runbook execution, issue resolution
- Tone: collaborative, hands-on
- Goal: smooth execution, knowledge transfer, building team confidence

> [!FROM-THE-SA-CHAIR]
> Migration communication is more often the cause of project drama than migration technology. End users blindsided by an outage will not forgive technical correctness. Application owners not consulted before their workload migrates feel disrespected. Executive sponsors who learn about a major risk on their second monthly review feel out of the loop. Communication discipline matters as much as Move-tool fluency.

---

### Year-2 Stable State and the Continuing Relationship

Migrations end. The customer's environment stabilizes. What does year 2 look like and what is BlueAlly's continuing role?

**Customer's year-2 reality:**
- The Nutanix cluster is the operational platform.
- The customer's team is competent at routine operations.
- Periodic upgrades (LCM-driven) happen on a quarterly or semi-annual cadence.
- Capacity planning is part of routine work.
- Hybrid steady-state (if applicable) is being managed.
- The original BoM and TCO assumptions are tracking (or they aren't, and there are conversations about why).

**BlueAlly's continuing role options:**

- **Managed services.** BlueAlly operates the platform on the customer's behalf (or co-managed with the customer's team). Common for customers with thin internal teams.
- **Quarterly health checks and optimization reviews.** BlueAlly reviews the environment quarterly, identifies optimization opportunities, recommends adjustments.
- **Upgrade support.** BlueAlly leads major version upgrades; the customer team handles minor.
- **Expansion projects.** Adding clusters, additional sites, NC2 cloud presence, NKE Kubernetes deployments, additional Nutanix products.
- **Training and certification programs.** BlueAlly delivers ongoing training as the customer's team grows or rotates.
- **Strategic advisory.** BlueAlly's SA participates in the customer's annual infrastructure planning.

The continuing relationship matters commercially (recurring revenue) and reputationally (reference customers, peer referrals). Plan for it from the beginning of the engagement.

---

## Lab Exercise: Build a Migration Plan for a Realistic Customer

> [!LAB] **Time:** ~4 hours · **Platform:** Notebook + spreadsheet for the planning artifacts

This is the capstone exercise. You build a complete migration plan for a representative customer scenario. The output is the plan you would walk into a customer kickoff with.

**The scenario:**
- 16 ESXi hosts (Dell PowerEdge R750) running ~600 VMs
- Mix of workloads: ~50 mission-critical (Tier-0), ~150 Tier-1 (databases, customer-facing apps), ~250 Tier-2 (general-purpose), ~100 Tier-3 (dev/test), ~50 ROBO sites
- NetApp filer (200 TB), Pure FlashArray for iSCSI to bare-metal Oracle (50 TB), Data Domain for Veeam (100 TB)
- VCF subscription, NSX-T (30 VMs use deep microsegmentation)
- SRM with 200 VMs orchestrated
- 4 datacenters (2 production, 1 DR, 1 ROBO consolidation site)
- Customer has 8-person infrastructure team
- Decision to consolidate to Nutanix on Dell XC hardware made; refresh window is 12 months out
- BlueAlly engaged to lead the migration

**Build the following deliverables:**

1. **Phase plan** with timeline (months 0-18) showing each of the 7 phases, their duration, and key deliverables.

2. **Discovery checklist for Phase 0:** what you need to inventory, who you talk to, what tools you use.

3. **Pilot wave VM selection:** propose 15 specific VMs (by category) that would be in the pilot wave. Justify each selection.

4. **Wave 1 plan:** which 100-200 VMs go in Wave 1 (low-tier production). What's the cutover strategy? What are the validation criteria?

5. **Wave 2 plan:** Tier-1 production. Which 200-300 VMs? What application-specific patterns are needed (e.g., SQL Always On, Oracle planned-failover)?

6. **Wave 3 plan:** Mission-critical. Which 50 VMs? What special handling do they need?

7. **Hybrid steady-state assessment:** which workloads (if any) might stay on VMware permanently? Why? What's the design implication?

8. **Risk register:** top 10 risks for this engagement, with mitigations and ownership.

9. **Communication plan:** cadence and format for executive sponsor, steering committee, application owners, end users, IT team.

10. **Success criteria:** how does the customer know the migration succeeded? Define quantitative and qualitative criteria.

11. **BlueAlly engagement model:** what's the resource plan from BlueAlly side? Lead SA, supporting engineers, project manager. Hours per phase.

12. **Year-2 stable state proposal:** what does steady state look like? What's BlueAlly's continuing role?

**What this teaches you:**
- The shape of a real migration plan.
- The interconnection between technical sequencing, communication, risk management, and commercial structure.
- The discipline to produce complete artifacts before kickoff.
- The mental model of a 12-18 month engagement.

**Customer-demo angle:** Plans 1-3 above (phase plan, discovery checklist, pilot wave selection) are what you walk into a customer kickoff with. Practice presenting them. The customer's project lead and executive sponsor are the audience. Time it: a 60-minute kickoff covers items 1-12 at appropriate depth.

---

## Practice Questions

Twelve questions. Six knowledge MCQ, four scenario MCQ, two NCX-style design questions. Read each, answer in your head, then read the explanation.

---

**Q1.** What is Move?
**Cert relevance:** NCP-MCI

A) A vSphere feature for live VM migration between ESXi hosts
B) A Nutanix tool for cross-platform VM migration (ESXi to AHV, Hyper-V to AHV, AWS to NC2, etc.) that handles initial replication, incremental sync, and brief cutover
C) A backup product
D) A storage migration product like Storage vMotion

**Answer:** B

**Why this answer:** Move is the cross-platform VM migration tool. It handles the source-to-target VM migration with brief cutover.

**Why not the others:**
- A) That describes vMotion, a VMware feature within vSphere clusters.
- C) Move is for migration, not backup.
- D) Storage vMotion is VMware's intra-vSphere storage migration; Move handles cross-hypervisor migration.

**The trap:** A and D reflect VMware mental models for live migration. Move serves a different purpose: cross-platform migration with planned cutover.

---

**Q2.** What is the recommended sequence for migrating workloads from VMware to Nutanix?
**Cert relevance:** NCP-MCI · sales-relevant

A) Migrate mission-critical workloads first to demonstrate value, then move backwards through tiers
B) Pilot with low-risk internal workloads, then low-tier production, then Tier-1, then mission-critical, then stabilization
C) Migrate randomly to avoid pattern bias
D) Migrate all workloads simultaneously in a single weekend

**Answer:** B

**Why this answer:** This is the risk-managed sequence. Pilot validates platform and process; subsequent waves scale capacity and confidence; mission-critical comes last because it has the highest risk and the most learning depends on prior wave experience.

**Why not the others:**
- A) Migrating mission-critical first is the failure pattern. Insufficient learning, highest risk, highest visibility for failure.
- C) Random sequencing introduces unnecessary risk.
- D) Single-weekend migration of large environments is unrealistic.

**The trap:** A is intuitive ("prove value with the highest-stakes win") but it inverts the risk discipline. Memorize the correct sequence.

---

**Q3.** Which of the following requires manual handling during a VMware-to-Nutanix migration (not automated by Move)?
**Cert relevance:** NCP-MCI

A) VM disk migration
B) NSX-T microsegmentation policies
C) VM CPU / RAM configuration translation
D) Power state preservation

**Answer:** B

**Why this answer:** NSX-T policies don't translate to Flow automatically. The customer must either re-implement on Flow or maintain NSX-T (NSX-T-on-Nutanix-on-ESXi pattern). Move handles the VM but not the security policy associated with NSX-T.

**Why not the others:**
- A) Move automates VM disk migration.
- C) Move translates VM configuration in best-effort.
- D) Move preserves power state.

**The trap:** A and C may seem like they require manual intervention; in fact Move automates them well. NSX-T is a known manual category.

---

**Q4.** What is "hybrid steady-state" in the context of a Nutanix migration?
**Cert relevance:** sales-relevant

A) A failed migration where the customer couldn't move all workloads
B) A successful end-state where most workloads run on Nutanix and specific workloads remain on VMware for legitimate reasons (NSX-T routing complexity, NetApp ONTAP-specific workflows, vendor-certified-only-on-ESXi applications, etc.)
C) The pre-cutover state where both platforms are running
D) A temporary state lasting only a few weeks

**Answer:** B

**Why this answer:** Hybrid steady-state is a permanent end-state where 70-90% of workloads have moved to Nutanix and specific workloads remain on VMware. This is often the architecturally correct outcome.

**Why not the others:**
- A) Hybrid is not failure; it can be the right answer.
- C) Pre-cutover state is parallel-running, not hybrid steady-state.
- D) Hybrid steady-state is by definition long-term, not temporary.

**The trap:** A reflects the misconception that any incomplete migration is failure. The right framing is hybrid as a viable end-state.

---

**Q5.** What is the typical duration for a full enterprise migration from VMware to Nutanix?
**Cert relevance:** sales-relevant

A) 2-4 weeks
B) 6-12 months
C) 12-18 months for typical mid-to-large enterprise environments, with significant variation based on environment size and complexity
D) 3+ years

**Answer:** C

**Why this answer:** 12-18 months is the typical range for mid-to-large enterprise environments running 500-2000+ VMs with multiple storage tiers, NSX-T, SRM, and complex application portfolios.

**Why not the others:**
- A) Far too short for enterprise scale.
- B) Possible for smaller environments (under 200-300 VMs) without complex integrations.
- D) Only for very large or very complex environments; not the typical range.

**The trap:** A reflects unrealistic timeline expectations. C reflects the realistic range that BlueAlly SAs should set for customer expectations.

---

**Q6.** During a migration, an application owner asks who will validate that their application works correctly post-migration. What is the correct answer?
**Cert relevance:** sales-relevant

A) BlueAlly does all the validation
B) The application owner is responsible for application-level validation; BlueAlly and the customer's IT team validate platform-level correctness (VM running, network reachable, storage accessible)
C) Validation is automated and requires no human verification
D) The application owner is not involved

**Answer:** B

**Why this answer:** The application owner has domain knowledge of what "working correctly" means for their application. Platform-level validation (VM up, network OK) is BlueAlly + IT. Application-level validation (transactions complete, integrations work, performance acceptable) is the application owner.

**Why not the others:**
- A) BlueAlly doesn't have application domain knowledge.
- C) Automation helps but doesn't replace owner validation.
- D) Application owner involvement is essential.

**The trap:** A is the impulse to take full responsibility; that costs you on the application-validation side.

---

**Q7.** A customer's CIO says: "I want all 600 VMs migrated in 3 months. We're under pressure to demonstrate cloud-readiness this fiscal year." What is the strongest BlueAlly SA response?
**Cert relevance:** sales-relevant

A) "Sure, we can do that. We'll move fast and figure out issues as we go."
B) "I appreciate the urgency. A 3-month timeline for 600 VMs across the workload mix you've described is unrealistic and would carry serious risk. Realistic timelines for environments of your size are 12-18 months. What I can offer in 3 months: pilot wave complete, Wave 1 well underway (100-200 VMs migrated), and clear progress to demonstrate. Let me walk through what's realistic for the fiscal-year pressure point and we'll find a story you can tell that's both ambitious and credible."
C) "Migrations always take 18 months. There's no flexibility."
D) "If you want to move fast, you should consider a different vendor."

**Answer:** B

**Why this answer:** Honest about the timeline, respectful of the urgency, offers achievable progress in 3 months, and reframes to a realistic outcome the CIO can defend internally. This is the SA-chair conversation that builds long-term trust.

**Why not the others:**
- A) Sets up project failure. Customer trust evaporates when reality hits in month 4.
- C) Inflexible and unhelpful.
- D) Sends the customer to a competitor.

**The trap:** A and C are extremes (overcommit and under-engage). B is the disciplined middle.

---

**Q8.** Which of the following workloads is the worst candidate for a pilot wave?
**Cert relevance:** sales-relevant

A) An internal IT-team-owned development VM
B) A small departmental file server (5 users, low criticality)
C) The customer's primary customer-facing e-commerce application (Tier-0)
D) A handful of Linux test VMs running standard Apache

**Answer:** C

**Why this answer:** Tier-0 customer-facing applications are the worst pilot candidates. Pilot waves are for validating platform and process with minimal blast radius; Tier-0 has maximum blast radius. Customer expectations get inverted from "learn together, fix issues" to "perfection on day one."

**Why not the others:**
- A) Internal IT-owned: ideal pilot candidate.
- B) Departmental, low criticality: good pilot candidate.
- D) Test/dev: ideal pilot candidate.

**The trap:** Customers (and SAs) sometimes propose Tier-0 as the pilot ("prove it works on the real thing"). The discipline is to push back; Tier-0 is for Wave 5, not for pilot.

---

**Q9.** A customer asks: "What happens if a migration goes wrong during cutover? Can we roll back?" What is the correct response?
**Cert relevance:** sales-relevant

A) "Migrations don't go wrong; rollback isn't needed."
B) "Yes. Every migration batch has a documented rollback plan. If post-migration validation fails, we revert: VM is brought back up on source platform, traffic is redirected back, root cause is investigated. The migration is rescheduled after the issue is understood."
C) "Rollback isn't possible once you're on Nutanix."
D) "Rollback is possible but takes weeks."

**Answer:** B

**Why this answer:** Rollback plans are part of disciplined migration management. Every batch has one. They are tested in Phase 2 (pilot wave) and used as needed.

**Why not the others:**
- A) Migrations do go wrong; pretending otherwise costs trust.
- C) Rollback is possible (return to source).
- D) Rollback for a single VM is typically minutes, not weeks.

**The trap:** A reflects overconfidence. C reflects misunderstanding (Move keeps source intact during initial migration; rollback is a re-cutover to source).

---

**Q10.** A customer's IT operations manager is concerned that "the team is going to be overwhelmed by 18 months of migration on top of normal operations." What is the strongest SA response?
**Cert relevance:** sales-relevant

A) "Just have your team work harder."
B) "That concern is exactly right. Migrations layered on top of normal operations cause team burnout and project failure. The realistic answer is one or more of: BlueAlly augmentation for migration work; phasing that respects team bandwidth (slower migration if team is thin); temporary contract help; reassigning some operations work; or accepting longer timelines. Let's plan capacity honestly. What's your team's available bandwidth for migration work per week?"
C) "Migrations are easy; your team can handle it."
D) "If your team can't handle it, you shouldn't migrate."

**Answer:** B

**Why this answer:** Respects the operations manager's concern, names the real options, and asks for the data needed to plan capacity. This is the disciplined SA-chair conversation.

**Why not the others:**
- A) Dismissive. Causes burnout and project failure.
- C) Underestimates the workload.
- D) Combative and unhelpful.

**The trap:** A and C are common. B is the right discipline.

---

**Q11.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / migration design discussion. Write your reasoning. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A customer's environment:
- 24 ESXi hosts across 3 datacenters (2 production + 1 DR), ~1,500 VMs
- Mix: 100 Tier-0 mission-critical (financial systems, regulatory), 400 Tier-1 production (databases, customer-facing apps), 700 Tier-2 general-purpose, 300 dev/test
- VCF + NSX-T + Aria + SRM (heavily customized)
- NetApp filer (300 TB) + Pure FlashArray (100 TB) + Data Domain (200 TB)
- 12-person infrastructure team (mix of senior and junior)
- Hardware refresh timing: production hosts in 9 months, DR hosts in 18 months
- Compliance: SOX (financial), PCI DSS (payment processing), some HIPAA (one application)
- Customer commitment to consolidate onto Nutanix; BlueAlly engaged

The customer asks BlueAlly to design the 24-month migration plan. Cover sequencing, parallel-running, NSX-T strategy, SRM strategy, hybrid steady-state assessment, team capacity, and key risk areas.

**The challenge:**
Walk through your plan. Identify what you still need to know.

**A strong answer covers:**
- **Phasing across 24 months:**
  - Months 0-3: Discovery + platform build (parallel tracks). Production cluster deployed in primary DC; DR cluster planning underway.
  - Months 3-5: Pilot wave (15-20 internal VMs). Validate Move, processes, network mapping.
  - Months 5-9: Production Wave 1 (Tier-2 + dev/test, ~700 VMs). Establish operational rhythm. Aligns with production hardware refresh window.
  - Months 9-15: Production Wave 2 (Tier-1, ~400 VMs). Application-aware patterns. Database migrations with care.
  - Months 15-19: Production Wave 3 (Tier-0, ~100 VMs). Mission-critical with maximum scrutiny.
  - Months 19-22: DR migration + SRM transition. Aligns with DR hardware refresh.
  - Months 22-24: Stabilization, decommissioning, hybrid steady-state finalization.
- **NSX-T strategy:**
  - Discover specific NSX-T features in use. Most VMs likely use only basic distributed firewall (translatable to Flow).
  - For VMs using advanced NSX-T features (BGP routing, edge services, L2VPN), evaluate keeping NSX-T-on-Nutanix-on-ESXi or accepting the migration cost of re-implementing.
  - Estimate: 80-90% of NSX-T policies translate to Flow; remaining 10-20% may justify NSX-T retention for those workloads.
- **SRM strategy:**
  - Inventory SRM runbook customizations. Many translate to Recovery Plans (Leap); some require careful re-implementation.
  - Plan SRM-to-Recovery-Plans migration in months 12-22 as Tier-1 / Tier-0 workloads move.
  - Coexistence pattern: SRM continues to orchestrate VMware workloads; Recovery Plans orchestrate AHV workloads. Some hybrid customers maintain both indefinitely.
- **Hybrid steady-state assessment:**
  - Likely outcome: 90-95% of workloads on Nutanix; remaining 5-10% on VMware.
  - Probable VMware-resident workloads: applications certified only on ESXi, NSX-T-routing-dependent applications, certain regulatory workloads if customer's audit team prefers stability.
  - Document the hybrid design from month 6 onward; revisit each phase.
- **Team capacity:**
  - 12-person team is reasonable for this scale, but migration consumes significant bandwidth.
  - BlueAlly augmentation likely needed in Phase 0 (discovery), Phase 4-5 (Tier-1 + Tier-0 waves), and Phase 6 (decommissioning).
  - Estimate: BlueAlly committed 1-2 lead SAs + 2-3 supporting engineers + 1 project manager for the duration.
- **Compliance considerations:**
  - SOX: change-control documentation, audit trail for financial-system migrations, separation-of-duties enforcement.
  - PCI DSS: PCI-scope migration with appropriate validation; possibly QSA involvement for re-attestation.
  - HIPAA: one application, validate covered-entity / business-associate-agreement coverage with Nutanix and BlueAlly.
- **Top risks:**
  - SRM customization complexity (mitigation: detailed SRM inventory in Phase 0)
  - Tier-1 database migration (mitigation: application-aware patterns, application-owner coordination, validated rollback)
  - Compliance audit timing (mitigation: align migration windows with audit cycles)
  - Team burnout (mitigation: BlueAlly augmentation, phasing)
  - Hardware refresh dependency (mitigation: detailed sequencing aligned to refresh)
- **What you still need to know:**
  - Exact NSX-T feature inventory
  - Specific SRM runbook customizations
  - Application owner inventory and engagement model
  - Compliance audit calendar
  - Customer's executive sponsor and reporting cadence preference
  - Existing vendor contracts and termination provisions

**A weak answer misses:**
- Linear sequencing without phase overlaps
- No NSX-T strategy (this is a real customer challenge)
- No SRM transition plan
- No compliance acknowledgment
- No team capacity discussion
- No hybrid steady-state assessment
- No risk register

**Why this question matters for NCX:** This is the kind of multi-quarter migration design that NCX panels evaluate. Pure-feature answers fail. The right answer integrates technical sequencing, regulatory awareness, team capacity, vendor strategy, and explicit risk management.

---

**Q12.** *(NCX-style architectural defense, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / engagement defense. Write your response.

**The scenario:**
You are 11 months into a customer migration. 600 of 1,500 VMs have moved. The customer's CIO calls a meeting. He says:

> *"This is taking too long, costing more than projected, and the team is exhausted. I'm questioning whether we should finish. Maybe we declare partial success at 600 VMs and stay hybrid permanently with the remaining 900 on VMware. Talk me out of it, or talk me into it. Either way, give me a real recommendation, not a sales pitch."*

**The challenge:**
Respond. The CIO has a real concern. A real recommendation is needed. Address it honestly.

**A strong answer covers:**
- **Acknowledge the concerns are real.** "You're right that we're behind original projections. The team is exhausted. The cost has run higher than initial estimates. None of this is in dispute."
- **Be specific about why:** "The original timeline assumed [specific assumption]. Reality has been [specific reality]. The two main drivers are [specific drivers, e.g., NSX-T complexity and Tier-1 application coordination]."
- **Walk through the choice honestly:**
  - **Option A: Continue to full migration.** Remaining 900 VMs in 8-10 months at current pace. Continued team load. Continued cost. Final outcome: full consolidation, simpler steady-state, full TCO benefits.
  - **Option B: Hybrid steady-state at 600 migrated.** Stop migrating. Stabilize. The 600 VMs run on Nutanix; the 900 stay on VMware. Operational complexity is permanent. Some TCO benefits captured, not all. License both platforms appropriately.
  - **Option C: Hybrid steady-state at higher count.** Pick a subset of the remaining 900 that are easy/low-risk. Migrate those (maybe 300-400 more in 4-6 months). Stop. The remainder stays on VMware permanently.
- **Make a real recommendation:**
  - Likely Option C in many real cases. "Looking at the remaining 900 VMs: ~400 are straightforward general-purpose workloads that would migrate in 4-6 months at current pace with acceptable team load. ~300 are Tier-1 with moderate complexity. ~200 are Tier-0 or NSX-T/SRM-dependent and would be the slowest. Recommend: continue the easy 400 (months 12-16), then make the Tier-1 + Tier-0 decision at month 16 with fresh data."
  - "This captures most of the consolidation value, respects team load, and gives you a defensible win to report."
- **Address the cost overrun:** "We need to be transparent about why costs ran higher and what continuing costs would be. Let me bring an updated TCO based on actuals to the next meeting. Decisions like this should be informed by current numbers, not original estimates."
- **Address the team exhaustion:** "Team load is non-negotiable in this conversation. Whatever path we choose has to respect what the team can sustainably absorb. Option C with a 4-6 month easy-migrations push, then a pause, gives the team a recovery window before the next decision."
- **Honor his framing:** "You asked for a real recommendation, not a sales pitch. My honest recommendation is Option C with the breakpoint at month 16. Some BlueAlly accounts would push for Option A because more migration is more BlueAlly engagement. That's the wrong answer for you. The right answer for you is the one that maximizes captured value while respecting team capacity and producing a result you can defend internally."
- **Close with a concrete next step:** "Let me come back next week with: (1) updated actuals-based TCO for Options A, B, C; (2) detailed phase plan for Option C; (3) explicit team-load projection for each option. You make the decision after seeing real numbers."

**A weak answer misses:**
- Defending the original plan instead of acknowledging the concerns
- Pushing Option A (full migration) without addressing the real costs
- Not naming hybrid steady-state as a legitimate outcome
- Not offering a real recommendation when asked
- Not respecting the CIO's "real recommendation, not a sales pitch" framing

**Why this question matters for NCX:** Mid-engagement crises are real. The disposition being tested is honest assessment, willingness to recommend against your own short-term commercial interest, and producing actionable analysis when the customer asks for it. This is the senior-SA conversation that builds 10-year customer relationships.

---

## What You Now Have

You can run a 12-18 month customer migration engagement end to end. Discovery, platform build, pilot, three production waves, stabilization, decommissioning. The shape is consistent; the specifics adapt to each customer.

You know Move's mechanics: cross-platform migration with brief cutover, what it handles automatically, what requires manual planning (NSX-T policies, complex application configurations, vSAN-specific storage policies, snapshot chains).

You know the 7-phase framework with clear entry and exit criteria per phase. You can produce a phased plan in your sleep and adapt it to the customer's environment.

You know the four mental frames for migration: phased risk management, the SA's 12-18 month engagement, platform consolidation, and what customers actually pay for and want.

You can position hybrid steady-state honestly. 70-90% migration with the rest remaining on VMware is often the right architectural answer. You frame it as a successful outcome, not a failure.

You can communicate at the right level for each audience: executive sponsor (monthly, business outcomes), steering committee (weekly, project management), application owners (per batch, technical and respectful), end users (notifications, minimize concern), IT team (continuous, collaborative).

You maintain a working risk register and act on it. Technical, operational, commercial, relationship risks. Weekly review with project lead, monthly with executive sponsor.

You know what year-2 stable state looks like and the continuing relationship: managed services, quarterly health checks, upgrade support, expansion projects, training, strategic advisory. Migrations end; relationships continue.

You have twelve practice questions worth of migration discrimination, including two NCX-style design defenses (24-month migration design with NSX-T + SRM strategy + compliance + team capacity, and the architectural defense of an honest mid-engagement recommendation when the customer is questioning whether to finish).

---

## Curriculum Capstone: What You Have After Ten Modules

Ten modules. Roughly 6,000-7,000 lines of curriculum. The complete picture:

**Technical depth (Modules 1-8):**
- HCI category and where it's right vs wrong
- Nutanix platform architecture (CVM, DSF, AOS, AHV, Prism)
- Hypervisor (AHV with the honest Broadcom-driven adoption story)
- Management plane (Prism Element, Prism Central, NCM tiers, categories, X-Play, v4 API)
- Storage fabric (DSF data path, RF, EC, compression, dedup, Curator, capacity planning)
- Networking and microsegmentation (OVS, virtual networks, Flow Network Security, Flow Virtual Networking, the NSX-T comparison)
- Data protection and DR (snapshots, Protection Domains/Policies, Async/NearSync/Metro, Recovery Plans, the SRM comparison, NC2 cloud DR)
- Unified storage (Files, Objects, Volumes, the NetApp/Cloudian/iSCSI consolidation pitch)

**Commercial depth (Module 9):**
- AOS and NCM tier feature mapping
- Hardware sourcing (NX, OEM, software-only)
- The Broadcom math
- Complete BoM construction with all six categories
- 5-year TCO methodology with sensitivity analysis
- Capex vs opex for CFO conversations
- Multi-year subscription mechanics

**Engagement depth (Module 10):**
- Move tool and migration tooling
- 7-phase framework
- Pilot wave through mission-critical production through hybrid steady-state
- Communication playbook by audience
- Risk register discipline
- Year-2 stable state and continuing relationship

**Cert preparation embedded throughout:**
- NCA: foundational topics across all modules
- NCP-MCI: ~25-30% of the curriculum directly tested
- NCM-MCI: heavy on storage, DR, and operational scenarios
- NCP-NS: networking and security specialty (Module 6 plus Module 7 replication networking)
- NCP-US: unified storage specialty (Module 8 with foundational coverage in Module 5)
- NCP-MCA: automation specialty (Module 4 with deeper applications throughout)
- NCX-MCI: oral defense preparation via the 20+ NCX-style design questions across modules

**SA-chair vocabulary embedded throughout:**
- Objections handled directly with honest framings
- Discovery questions positioned at the right places in the conversation
- Customer-demo flows for the highest-impact features
- Honest gaps named alongside honest strengths
- Coexistence patterns named for every incumbent comparison

**Appendices A through K** (built next): complete the reference toolkit.

You are now ready for real customer engagements. The curriculum's job is done. Yours begins.

---

## References

Authoritative sources verified during the technical review pass on this module. Most of Module 10's content is engagement methodology and project framework, which is BlueAlly-and-industry practice rather than Nutanix-product specification; the Move-tool specifics are tied to Nutanix sources verified in Module 03's review.

- [Nutanix Move Product Page](https://www.nutanix.com/products/move). Current product positioning, supported source/target paths, free-tool status.
- [Nutanix Move 5.5 Documentation Hub](https://portal.nutanix.com/docs/Nutanix-Move-v5_5:top-hyperv-vm-migration-c.html). Authoritative reference for Hyper-V, ESXi, cloud-source migration paths.
- [Nutanix Bible — VM Migration Architecture](https://www.nutanixbible.com/21b-vm-migration-arch.html). Move's CBT-equivalent change-tracking, cutover semantics, downtime envelope.
- [Nutanix Move on AHV (Tech Note TN-2072)](https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2072-AHV-Migration:nutanix-move.html). Detailed Move architecture for AHV migrations.
- [Accelerate VMware Migrations to AWS with Nutanix NC2 (AWS APN Blog)](https://aws.amazon.com/blogs/apn/accelerate-vmware-migrations-to-aws-with-nutanix-nc2/). VMware → NC2 migration use case.
- [RVTools (Dell)](https://www.dell.com/support/kbdoc/en-us/000202418/rvtools). The standard third-party VMware inventory tool referenced in the discovery phase.
- [Lansweeper Network Discovery](https://www.lansweeper.com/). Asset and dependency discovery tool also referenced for migration discovery.
- [Nutanix Bible — Disaster Recovery Services (Recovery Plans referenced for hybrid coexistence)](https://www.nutanixbible.com/13a-book-of-dr-services.html). Cross-cluster orchestration patterns relevant to hybrid steady-state.
- [Nutanix Cloud Clusters (NC2) on AWS](https://www.nutanix.com/products/nutanix-cloud-clusters/aws). NC2 as a migration target for cloud-bound workloads.

---

## Cross-References

- **Previous:** [Module 9: Licensing and Real Costs](./09-licensing-economics.md)
- **Next:** [Appendix A: Glossary](./appendix-a-glossary.md) (or any appendix)
- **Glossary:** [Move](./appendix-a-glossary.md#move) · [Pilot wave](./appendix-a-glossary.md#pilot-wave) · [Production wave](./appendix-a-glossary.md#production-wave) · [Cutover](./appendix-a-glossary.md#cutover) · [Parallel-running](./appendix-a-glossary.md#parallel-running) · [Hybrid steady-state](./appendix-a-glossary.md#hybrid-steady-state) · [Risk register](./appendix-a-glossary.md#risk-register) · [Dependency mapping](./appendix-a-glossary.md#dependency-mapping)
- **Comparison Matrix:** [Move vs other migration tools](./appendix-b-comparison-matrix.md#migration-tools)
- **Objections:** [#41 "Migration is too risky"](./appendix-d-objections.md#obj-041) · [#42 "We don't have time for 18 months"](./appendix-d-objections.md#obj-042) · [#43 "Our team can't take on a migration"](./appendix-d-objections.md#obj-043) · [#44 "What if Move fails on our complex VMs?"](./appendix-d-objections.md#obj-044) · [#45 "Can we just do hybrid permanently?"](./appendix-d-objections.md#obj-045)
- **Discovery Questions:** [Q-MIG-01 Workload tier inventory](./appendix-e-discovery-questions.md#q-mig-01) · [Q-MIG-02 Application dependency awareness](./appendix-e-discovery-questions.md#q-mig-02) · [Q-MIG-03 NSX-T / SRM footprint](./appendix-e-discovery-questions.md#q-mig-03) · [Q-MIG-04 Team capacity for migration work](./appendix-e-discovery-questions.md#q-mig-04) · [Q-MIG-05 Compliance / audit timing](./appendix-e-discovery-questions.md#q-mig-05) · [Q-MIG-06 Hardware refresh windows](./appendix-e-discovery-questions.md#q-mig-06)
- **Sizing Rules:** [Migration wave sizing](./appendix-f-sizing-rules.md#wave-sizing) · [Move concurrency limits](./appendix-f-sizing-rules.md#move-concurrency)
- **CLI Reference:** [Move CLI](./appendix-g-cli-reference.md#move-cli)
- **Reference Architectures:** [Customer-running migration playbook](./appendix-i-reference-architectures.md#ra-migration-playbook) · [Hybrid steady-state design](./appendix-i-reference-architectures.md#ra-hybrid)
- **POC Playbook:** [Move tool demo](./appendix-j-poc-playbook.md#move-demo) · [Pilot wave templates](./appendix-j-poc-playbook.md#pilot-templates)
- **Cert Tracker:** [Migration topics in cert blueprints](./appendix-k-cert-tracker.md#migration-coverage)

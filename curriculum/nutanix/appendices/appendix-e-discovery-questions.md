---
appendix: E
title: Discovery Questions
type: reference
purpose: |
  The kickoff-meeting and ongoing-discovery question bank. Use during initial
  customer meetings, technical deep-dives, and commercial negotiations. Each
  question is designed so the answer changes the recommendation; if the answer
  doesn't matter, it's not the right question.
usage: |
  Read the relevant section before a customer call. Pick 5-15 questions
  appropriate to the meeting type. Don't try to ask all 46 in one session.
  Discovery is iterative; questions deepen as the relationship deepens.
discipline: |
  Five principles across all discovery:
  1. Open-ended over closed. "What does your DR look like?" beats "Do you do DR?"
  2. Don't lead toward Nutanix. Surface the real situation, not the answer you want.
  3. Listen for what's not said. The unspoken concerns are often the most important.
  4. Ask follow-up questions. The first answer is rarely the complete picture.
  5. Take notes. The discovery you do today informs the proposal in 4 weeks.
last_updated: 2026-04-30
covers:
  - General discovery (6 questions)
  - Workload inventory (6 questions)
  - Management and automation (4 questions)
  - Storage (9 questions)
  - Networking and security (5 questions)
  - Data protection / DR (5 questions)
  - Migration (6 questions)
  - Economics (5 questions)
---

# Appendix E: Discovery Questions

Forty-six discovery questions across eight topic areas. Each is designed so the answer changes what BlueAlly recommends. The questions are starting points; the customer-specific follow-ups are where real discovery happens.

**The senior-SA mindset on discovery:** customers don't trust vendors who pitch before they understand. Discovery is how trust is built. Spending the first meeting (or the first three) on discovery rather than presentation is often what distinguishes the BlueAlly engagement from competing vendors who lead with slides.

**The four meeting types:**

- **Kickoff (1-2 hours):** general discovery, broad picture. Use Q-GEN, Q-WL, Q-ECON questions.
- **Technical deep-dive (2-4 hours):** specific to a domain. Use Q-STOR, Q-NET, Q-DR, Q-MGMT depending on focus.
- **Commercial review (1 hour):** budget, timing, decision-making. Use Q-ECON, Q-GEN-03 questions.
- **Architecture review (full day):** full cross-domain review for complex environments. Mix from all sections.

Don't ask all 46 in one session. Pick the 5-15 most relevant. Discovery is iterative.

---

## General Discovery (Q-GEN)

### Q-GEN-01: Current Infrastructure Baseline

**Ask:** *"Walk me through your current infrastructure at a high level. How many sites, how many hosts, what's the hypervisor and storage stack, what's the rough VM count?"*

**Listen for:** Scale, complexity, vendor mix, age of the deployment.

**If they have a clean inventory:** Great; ask for specifics. Reasonable assumption: this is a mature ops team.
**If they don't:** That itself is information. Discovery will take longer; offer to use RVTools or similar to help inventory.

**Related objections:** Sets context for nearly all other objections.
**Module reference:** N/A (foundational)

---

### Q-GEN-02: Pain Points and Triggers

**Ask:** *"What's prompting this conversation? What's not working today, or what's about to change?"*

**Listen for:** The actual reason they're considering Nutanix. Refresh timing. Specific pain (cost, performance, complexity, vendor relationship). Sometimes a recent incident.

**If they say "VMware renewal pricing":** Licensing-driven. Use Q-ECON questions and frame around 5-year TCO comparison.
**If they say "DR doesn't work":** DR-driven. Lead with NC2 conversation and Recovery Plans demo.
**If they say "Storage refresh":** Storage-led. Lead with consolidation story (Files / Objects / Volumes).
**If they say "Just exploring":** No urgency yet. Discovery-only mode for now; build relationship for next refresh.

**Related objections:** All; understanding the trigger reframes everything.
**Module reference:** N/A (foundational)

---

### Q-GEN-03: Budget Cycle and Decision-Makers

**Ask:** *"Help me understand your decision process. Who needs to be involved, what's your fiscal year, when's the next budget cycle, and what's the approval threshold for a deal of this size?"*

**Listen for:** Buying committee structure, timeline pressure, approval bureaucracy.

**If decision is concentrated:** Faster cycle; engage the decision-maker directly.
**If decision is distributed:** Plan stakeholder map; expect 6-12 month sales cycle for enterprise deals.

**Related objections:** [#36 Subscription preference](./appendix-d-objections.md#36-we-prefer-perpetual-licensing-subscription-doesnt-work-for-us), [#38 Recent VMware renewal](./appendix-d-objections.md#38-we-just-renewed-vmware-for-3-years-bad-timing).
**Module reference:** [Module 9: Licensing](./09-licensing-economics.md).

---

### Q-GEN-04: Timeline and Refresh Windows

**Ask:** *"What's coming due in the next 12-24 months? Hardware refresh, software renewals, contract end dates, datacenter lease, anything?"*

**Listen for:** Concrete timing anchors. The migration plan should align with these windows.

**If hardware refresh in 6-9 months:** Natural conversation timing; aligns capex flow.
**If VMware renewal in 12+ months:** Pilot now; main migration aligned with renewal end.
**If nothing in 18+ months:** Discovery-only; revisit closer to a refresh window.

**Related objections:** [#38 Recent VMware renewal](./appendix-d-objections.md#38-we-just-renewed-vmware-for-3-years-bad-timing), [#42 Timeline pressure](./appendix-d-objections.md#42-we-dont-have-time-for-an-18-month-project).
**Module reference:** [Module 10: Migration](./10-migration-path.md).

---

### Q-GEN-05: Team Size and Skills

**Ask:** *"Tell me about your infrastructure team. How many people, what skill mix, who owns what?"*

**Listen for:** Capacity for migration work, existing platform expertise, role concentration vs distribution.

**If team is large and specialized:** Migration capacity is good; coordination is harder.
**If team is small and generalist:** Migration capacity is constrained; BlueAlly augmentation likely needed.
**If team has Nutanix experience already:** Faster ramp. Acknowledge and adapt.
**If team has no Nutanix experience:** Plan training (NCA, NCP-MCI) into the engagement.

**Related objections:** [#43 Team capacity](./appendix-d-objections.md#43-our-team-is-already-overloaded-they-cant-take-on-a-migration).
**Module reference:** [Module 10: Migration](./10-migration-path.md).

---

### Q-GEN-06: Compliance Frameworks

**Ask:** *"What compliance or regulatory frameworks apply to your environment? PCI, HIPAA, SOX, FedRAMP, state regulations, industry-specific?"*

**Listen for:** Specific frameworks, audit timing, scope of regulated data.

**If heavy compliance:** Plan for compliance review early; identify Nutanix's relevant certifications; expect documentation requests.
**If specific data sovereignty:** May affect cloud DR (NC2) decisions.
**If audit calendar is tight:** Migration timing must respect audit windows.

**Related objections:** [#21 Compliance approval](./appendix-d-objections.md#21-our-compliance-team-wont-approve-a-new-platform), [#29 Cloud DR compliance](./appendix-d-objections.md#29-cloud-dr-isnt-for-us-compliance-requires-data-on-prem).
**Module reference:** [Scenario 4: Compliance-heavy](./appendix-c-scenarios.md), [Module 7: DR](./07-data-protection.md).

---

## Workload Inventory (Q-WL)

### Q-WL-01: VM Count and Distribution

**Ask:** *"What's the rough VM count, and how is it distributed: dev/test, general production, Tier-1, mission-critical?"*

**Listen for:** Total scale, tier distribution, the criticality mix.

**If dev/test is heavy (>40%):** Easy migration starting point. Wave 1 candidates.
**If Tier-0 / mission-critical is heavy:** Migration risk is higher; phasing more careful.

**Related objections:** [#41 Migration risk](./appendix-d-objections.md#41-migration-is-too-risky-we-cant-afford-an-outage).
**Module reference:** [Module 10: Migration](./10-migration-path.md).

---

### Q-WL-02: Critical Applications

**Ask:** *"What are your top 5 most-critical applications? What does each one do, what's the technology stack, what's the SLA?"*

**Listen for:** Specific dependencies, technology constraints, application owner names.

**Use the answers to:** identify the workloads that need the most careful migration handling, name application owners for later coordination, surface technology dependencies (Oracle RAC, SQL Always On, specific vendor certifications).

**Related objections:** [#3 Database performance](./appendix-d-objections.md#3-we-need-dedicated-storage-for-our-database-hci-cant-handle-it), [#41 Migration risk](./appendix-d-objections.md#41-migration-is-too-risky-we-cant-afford-an-outage).
**Module reference:** [Module 10 § Wave Planning](./10-migration-path.md).

---

### Q-WL-03: Workload Tier Distribution

**Ask:** *"How do you tier your workloads? What's the SLA, RPO, and RTO requirement for Tier-0 vs Tier-1 vs Tier-2 vs general-purpose?"*

**Listen for:** Whether tiering is formal, what the actual numbers are, whether SLAs are documented vs aspirational.

**If they have formal tiering:** Use the tiers to drive replication mode and Recovery Plan design.
**If they don't:** Help them define tiers as part of discovery; this becomes a migration planning artifact.

**Related objections:** [#27 Array-based replication](./appendix-d-objections.md#27-we-have-array-based-replication-why-would-we-use-nutanix), [#28 DR migration](./appendix-d-objections.md#28-migrating-dr-is-too-complex-we-cant-risk-it).
**Module reference:** [Module 7: Data Protection](./07-data-protection.md).

---

### Q-WL-04: Database Footprint

**Ask:** *"Tell me about your database environment. SQL Server, Oracle, PostgreSQL, others? Scale of each, latency requirements, HA architecture?"*

**Listen for:** Database engine mix, performance requirements, application-level HA (Always On, RAC, replica sets).

**If application-level HA is in place:** RF2 may be sufficient for storage replication; the application provides additional layer.
**If no application HA:** RF3 may be appropriate for the database tier.
**If extreme latency requirements:** Plan a POC with the actual workload; don't promise without measuring.

**Related objections:** [#3 Database performance](./appendix-d-objections.md#3-we-need-dedicated-storage-for-our-database-hci-cant-handle-it), [#12 Tail latency](./appendix-d-objections.md#12-tail-latency-on-distributed-storage-will-hurt-our-database-we-need-an-all-flash-array).
**Module reference:** [Module 5: DSF](./05-dsf-storage-deep-dive.md).

---

### Q-WL-05: VDI Footprint

**Ask:** *"Are you running VDI? If so, how many sessions, what broker (Citrix / VMware Horizon / RDS), persistent or non-persistent profiles?"*

**Listen for:** Scale, broker choice, profile pattern, boot-storm timing.

**If VDI is significant:** Strong Nutanix sweet spot; lead with VDI consolidation conversation. On-disk dedup makes sense for persistent profiles.
**If VDI is light:** Less specialized design needed.

**Related objections:** N/A specific; VDI tends to surface as positive value rather than objection.
**Module reference:** [Scenario 3: VDI Deployment](./appendix-c-scenarios.md).

---

### Q-WL-06: Specialty Workloads

**Ask:** *"Are there any specialty workloads I should know about? AI/ML training, HPC, video processing, real-time analytics, anything unusual?"*

**Listen for:** Workloads outside typical enterprise patterns.

**If HPC at scale:** Warning flag. Specialty parallel filesystems (Lustre, GPFS) may still be needed; not a Nutanix sweet spot.
**If AI/ML training:** GPU passthrough on AHV is supported; workload-specific design.
**If video processing or media:** Possibly an Isilon / FlashBlade workload; evaluate carefully.

**Related objections:** [#1 Scale skepticism](./appendix-d-objections.md#1-hci-is-just-for-small-workloads-were-an-enterprise) (sometimes triggered by specialty workloads).
**Module reference:** [Module 1 § Where HCI is Wrong](./01-hci-foundations.md), [Module 8 § Files vs Isilon](./08-unified-storage.md).

---

## Management and Automation (Q-MGMT, Q-AUT)

### Q-MGMT-01: Current Management Stack

**Ask:** *"Walk me through your management plane today. vCenter version, Aria components, vSphere Lifecycle Manager, anything else?"*

**Listen for:** Depth of Aria adoption, integration complexity, management-team size.

**If Aria is light (vCenter + vROps only):** Simpler migration to NCM Pro.
**If Aria is deep (vCenter + Aria Ops + Aria Automation + custom blueprints):** Complex migration; likely NCM Ultimate; coexistence pattern with Aria for cross-vendor analytics.

**Related objections:** [#10 Aria investment](./appendix-d-objections.md#10-what-about-my-aria-investment-we-have-5-years-of-dashboards-and-policies).
**Module reference:** [Module 4: Prism](./04-prism-management.md).

---

### Q-MGMT-02: Aria Footprint

**Ask:** *"How heavily are you using Aria? Specifically: how many vROps reports do you actually use, how many Aria Automation blueprints are in active use, what custom integrations exist?"*

**Listen for:** Depth of actual usage vs license entitlement. Often customers have Aria licensed but use only a fraction.

**If actual usage is light:** NCM tier replacement is straightforward.
**If actual usage is deep:** Migration is real engineering work; plan parallel-running.

**Related objections:** [#10 Aria investment](./appendix-d-objections.md#10-what-about-my-aria-investment-we-have-5-years-of-dashboards-and-policies).
**Module reference:** [Module 4: Prism](./04-prism-management.md).

---

### Q-MGMT-03: Identity and SSO Requirements

**Ask:** *"How do you handle identity for infrastructure admin access? AD, SAML to a specific IdP, MFA requirements, role separation?"*

**Listen for:** Specific identity provider, role-separation requirements (often compliance-driven), MFA/JIT requirements.

**Use the answer to:** plan Prism Central identity integration, RBAC role design, audit logging configuration.

**Related objections:** [#21 Compliance approval](./appendix-d-objections.md#21-our-compliance-team-wont-approve-a-new-platform).
**Module reference:** [Module 4: Prism](./04-prism-management.md), [Scenario 4: Compliance-heavy](./appendix-c-scenarios.md).

---

### Q-AUT-01: Automation Tooling Inventory

**Ask:** *"What automation tooling does your team use? PowerCLI, Terraform, Ansible, custom Python scripts, ServiceNow integration, anything else?"*

**Listen for:** Tooling preferences, depth of automation, maturity of the practice.

**If PowerCLI-heavy:** Nutanix PowerShell module is the natural transition; emphasize compatibility.
**If Terraform-mature:** Nutanix Terraform provider; emphasize the IaC alignment.
**If ServiceNow-integrated:** Plan webhook / X-Play integration carefully; this is often the highest-risk integration.

**Related objections:** [#9 PowerCLI investment](./appendix-d-objections.md#9-what-about-our-powercli-scripts-weve-built-years-of-automation), [#22 PowerCLI standardization](./appendix-d-objections.md#22-our-team-standardized-on-powercli-we-cant-switch), [#23 ServiceNow integration](./appendix-d-objections.md#23-our-servicenow-integration-will-break-thats-not-acceptable).
**Module reference:** [Module 4 § v4 API](./04-prism-management.md).

---

## Storage (Q-STOR)

### Q-STOR-01: Workload IOPS / Latency Profile

**Ask:** *"What's the I/O profile of your workloads? Specifically: what's your peak IOPS, what's the read/write split, and what's the p99 latency requirement on your most demanding workload?"*

**Listen for:** Specific numbers vs vague "high performance" claims. Customers who can answer with numbers have measured; those who can't have rough estimates.

**If they have specific p99 numbers:** Validate against DSF capabilities; if 5ms or higher, DSF is comfortable. If <2ms p99, plan POC carefully.
**If they don't know:** Offer to help measure during POC.

**Related objections:** [#3 Database performance](./appendix-d-objections.md#3-we-need-dedicated-storage-for-our-database-hci-cant-handle-it), [#12 Tail latency](./appendix-d-objections.md#12-tail-latency-on-distributed-storage-will-hurt-our-database-we-need-an-all-flash-array).
**Module reference:** [Module 5: DSF](./05-dsf-storage-deep-dive.md).

---

### Q-STOR-02: Capacity Targets and Growth

**Ask:** *"What's your current storage footprint, what's the growth rate, and what's the 3-year capacity target?"*

**Listen for:** Current capacity, growth trajectory, refresh-cycle expectations.

**Use the answers to:** size cluster appropriately with growth headroom, plan multi-year subscription with growth provisions, validate consolidation case.

**Related objections:** [#19 Capacity efficiency](./appendix-d-objections.md#19-capacity-efficiency-on-my-array-is-better-we-dont-need-hci).
**Module reference:** [Module 5: DSF](./05-dsf-storage-deep-dive.md), [Module 9: Licensing](./09-licensing-economics.md).

---

### Q-STOR-03: Existing Array Dependencies

**Ask:** *"Walk me through your existing storage tiers. What's running where, why was it chosen, what specific features do you depend on?"*

**Listen for:** ONTAP-specific features (FlexClone, FlexCache), Pure data services, Dell PowerStore features, Data Domain dedup, anything platform-specific that doesn't translate cleanly.

**If they depend on specific incumbent features:** Plan workflow mapping exercise; identify what migrates cleanly vs what stays.

**Related objections:** [#31 NetApp investment](./appendix-d-objections.md#31-we-have-netapp-why-would-we-add-files), [#19 Capacity efficiency](./appendix-d-objections.md#19-capacity-efficiency-on-my-array-is-better-we-dont-need-hci).
**Module reference:** [Module 8: Unified Storage](./08-unified-storage.md), [Comparison Matrix § Files vs ONTAP](./appendix-b-comparison-matrix.md#nutanix-files-vs-netapp-ontap).

---

### Q-STOR-04: Backup Integration

**Ask:** *"What backup product are you using, where does the backup data land, and what's the cost of that backup infrastructure annually?"*

**Listen for:** Backup vendor (Veeam / Commvault / Rubrik / Cohesity / HYCU), repository type (Data Domain, NetApp StorageGRID, etc.), annual cost.

**If using S3-capable backup product with separate dedup appliance:** Strong Objects consolidation conversation; one of the easier ROI wins.

**Related objections:** [#33 Backup target consolidation](./appendix-d-objections.md#33-our-backup-target-is-fine-why-consolidate).
**Module reference:** [Module 8 § Backup-Target Consolidation](./08-unified-storage.md).

---

### Q-STOR-05: Compliance and Data Placement

**Ask:** *"Are there compliance or contractual requirements around where specific data must live? Encryption, geo-restrictions, audit retention?"*

**Listen for:** Encryption-at-rest requirements, key-management requirements (HSM integration), audit-log retention duration, geo-locality constraints.

**Use the answers to:** validate Nutanix's encryption capabilities against requirements, plan KMIP integration with HSM, configure audit-log forwarding.

**Related objections:** [#21 Compliance approval](./appendix-d-objections.md#21-our-compliance-team-wont-approve-a-new-platform), [#29 Cloud DR compliance](./appendix-d-objections.md#29-cloud-dr-isnt-for-us-compliance-requires-data-on-prem).
**Module reference:** [Scenario 4: Compliance-heavy](./appendix-c-scenarios.md).

---

### Q-STOR-06: File Workload Inventory

**Ask:** *"What file workloads are you running? User home directories, application file shares, VDI profiles, backup repositories, anything else?"*

**Listen for:** Diversity of file workloads, total capacity, AD integration depth, current filer's role.

**Use the answers to:** size Files appropriately, identify SMB vs NFS mix, plan any specialty workloads (Isilon-class workloads stay on Isilon).

**Related objections:** [#31 NetApp investment](./appendix-d-objections.md#31-we-have-netapp-why-would-we-add-files).
**Module reference:** [Module 8: Unified Storage](./08-unified-storage.md).

---

### Q-STOR-07: Object/S3 Use Cases

**Ask:** *"Are you using object storage anywhere? Backup targets, application data, archive, cloud-native workloads?"*

**Listen for:** Whether they have any S3 footprint (AWS or on-prem), what it's used for, scale.

**If they have on-prem object (Cloudian, Scality, MinIO, StorageGRID):** Direct consolidation conversation.
**If they only use AWS S3:** Discuss complementary on-prem Objects for steady-state workloads.
**If they don't use object storage:** Less Objects conversation, focus on Files and Volumes.

**Related objections:** [#32 Objects vs S3](./appendix-d-objections.md#32-why-use-objects-instead-of-aws-s3), [#33 Backup-target consolidation](./appendix-d-objections.md#33-our-backup-target-is-fine-why-consolidate).
**Module reference:** [Module 8 § Objects](./08-unified-storage.md).

---

### Q-STOR-08: iSCSI Consumer Inventory

**Ask:** *"Do you have any iSCSI consumers that aren't running on the VMware cluster? Bare-metal servers, Oracle RAC, legacy applications, anything?"*

**Listen for:** Specific consumer types, performance requirements, vendor dependencies.

**If significant iSCSI footprint exists:** Volumes consolidation conversation. Map specific consumers; some translate cleanly, some have specific requirements.

**Related objections:** [#34 iSCSI consumers](./appendix-d-objections.md#34-we-have-iscsi-consumers-that-arent-moving-they-need-a-real-array).
**Module reference:** [Module 8 § Volumes](./08-unified-storage.md).

---

### Q-STOR-09: Backup Target Architecture

**Ask:** *"Specifically about your backup architecture: what's the primary backup target, secondary tier, cloud archive if any? Total capacity? Retention policy?"*

**Listen for:** Multi-tier backup architecture, total cost of the backup infrastructure, retention complexity.

**Use the answers to:** propose specific backup-target consolidation onto Objects with appropriate sizing for capacity and retention.

**Related objections:** [#33 Backup-target consolidation](./appendix-d-objections.md#33-our-backup-target-is-fine-why-consolidate).
**Module reference:** [Module 8 § Backup-Target Story](./08-unified-storage.md), [Scenario 6: Cloud DR with NC2](./appendix-c-scenarios.md).

---

## Networking and Security (Q-NET, Q-SEC)

### Q-NET-01: Physical Network Topology

**Ask:** *"What's your physical network architecture? Top-of-rack switches, core, fabric vendor, link speeds, any specific topology choices?"*

**Listen for:** Vendor (Cisco, Arista, Juniper), link speeds (10/25/40/100 GbE), redundancy patterns, any unusual topology.

**Use the answers to:** validate Nutanix networking requirements (25 GbE recommended for production, 100 GbE for high-density), plan cluster placement, identify any pre-migration network upgrades.

**Related objections:** [#17 Cisco fabric tuned](./appendix-d-objections.md#17-our-cisco-fabric-is-tuned-for-our-network-software-defined-networking-wont-match), [#25 SDN trust](./appendix-d-objections.md#25-we-dont-trust-software-defined-networking-hardware-is-more-reliable).
**Module reference:** [Module 6: Networking](./06-networking-flow.md).

---

### Q-NET-02: NSX-T Footprint

**Ask:** *"Tell me about your NSX-T deployment. What are you using it for: distributed firewall, routing, edge services, L2VPN, federation?"*

**Listen for:** Depth of NSX-T usage. Light usage (basic distributed firewall on a subset of VMs) translates cleanly to Flow. Deep usage (BGP integration, edge services, L2VPN) often requires NSX-T retention.

**If NSX-T is light:** Flow Network Security replacement is feasible.
**If NSX-T is deep:** Hybrid is the likely answer; map workloads to "translate to Flow" vs "keep NSX-T."

**Related objections:** [#16 NSX-T retention](./appendix-d-objections.md#16-what-about-nsx-t-we-have-it-deployed-and-tuned), [#20 Existing microsegmentation](./appendix-d-objections.md#20-weve-already-invested-in-microsegmentation-were-not-starting-over).
**Module reference:** [Module 6: Networking](./06-networking-flow.md), [Comparison Matrix § Flow vs NSX-T](./appendix-b-comparison-matrix.md#flow-network-security-vs-vmware-nsx-t-distributed-firewall).

---

### Q-NET-03: Microsegmentation Requirements

**Ask:** *"What's your microsegmentation strategy? Are you doing application-tier segmentation today, planning to, or treating it as future?"*

**Listen for:** Current state, drivers (compliance, zero-trust, recent incident), maturity of the practice.

**If they have mature microsegmentation:** Address NSX-T retention or Flow translation per Q-NET-02.
**If they're planning microsegmentation:** Strong Flow Network Security positioning.
**If it's future:** Lower priority for current discussion; mention as a future capability.

**Related objections:** [#20 Existing microsegmentation](./appendix-d-objections.md#20-weve-already-invested-in-microsegmentation-were-not-starting-over).
**Module reference:** [Module 6 § Flow Network Security](./06-networking-flow.md).

---

### Q-NET-04: Multi-Tenancy and VPC Needs

**Ask:** *"Do you have multi-tenant requirements? Different business units needing isolation, service-provider tenants, projects with separate quotas?"*

**Listen for:** Tenant isolation requirements, scale (a few business units vs many tenants), strictness of isolation (compliance-driven vs convention).

**If significant multi-tenancy:** Flow Virtual Networking conversation; Projects in Prism Central; consider service-insertion patterns for security.
**If single-tenant or simple BU separation:** FVN may be overkill; Categories + Projects in Prism Central often sufficient.

**Related objections:** N/A specific; multi-tenancy tends to surface as positive value.
**Module reference:** [Module 6 § Flow Virtual Networking](./06-networking-flow.md).

---

### Q-SEC-01: Compliance Frameworks (Detail)

**Ask:** *"For each compliance framework you mentioned: what's the current scope, when's the next audit, and what specifically gets audited?"*

**Listen for:** Specific scope (which workloads, which data), audit timing (don't migrate during audit windows), audit depth.

**Use the answers to:** plan migration timing around audit cycles, identify documentation requirements, ensure Nutanix's certifications cover the customer's scope.

**Related objections:** [#21 Compliance approval](./appendix-d-objections.md#21-our-compliance-team-wont-approve-a-new-platform).
**Module reference:** [Scenario 4: Compliance-heavy](./appendix-c-scenarios.md).

---

## Data Protection / DR (Q-DR)

### Q-DR-01: RPO/RTO Targets per Tier

**Ask:** *"What's your RPO and RTO target by application tier? What do you currently achieve, and is there a gap between target and reality?"*

**Listen for:** Whether targets are formal or aspirational, current vs target gap, whether they've ever tested.

**Use the answers to:** map tiers to replication mode (Async / NearSync / Metro), design Recovery Plans, identify tiers where current DR is inadequate.

**Related objections:** [#28 DR migration risk](./appendix-d-objections.md#28-migrating-dr-is-too-complex-we-cant-risk-it), [#30 Test failover disruption](./appendix-d-objections.md#30-test-failover-is-too-disruptive-we-cant-do-it-quarterly).
**Module reference:** [Module 7: Data Protection](./07-data-protection.md).

---

### Q-DR-02: Existing DR Infrastructure

**Ask:** *"What's your DR architecture today? Second site, replication mechanism, runbook, how often have you tested?"*

**Listen for:** Whether DR is real (tested, validated) or paper-only.

**If DR is paper-only:** Strong NC2 / Recovery Plans pitch; the test-failover capability is genuine value.
**If DR is mature with regular testing:** Migration is delicate; respect the working system; coexistence may be right.

**Related objections:** [#26 SRM investment](./appendix-d-objections.md#26-what-about-srm-weve-spent-10-years-on-the-runbooks), [#27 Array-based replication](./appendix-d-objections.md#27-we-have-array-based-replication-why-would-we-use-nutanix), [#28 DR migration](./appendix-d-objections.md#28-migrating-dr-is-too-complex-we-cant-risk-it).
**Module reference:** [Module 7: Data Protection](./07-data-protection.md), [Scenario 6: Cloud DR](./appendix-c-scenarios.md).

---

### Q-DR-03: SRM Footprint

**Ask:** *"If you're using SRM: how many VMs are orchestrated, how customized are the runbooks, when did you last run a real failover?"*

**Listen for:** SRM scale and depth of customization. The deeper the customization, the harder Recovery Plans migration.

**If SRM is simple (basic runbooks, default IP remapping):** Recovery Plans translation is feasible.
**If SRM is heavily customized:** Coexistence pattern likely right; SRM stays for the workloads it orchestrates well.

**Related objections:** [#26 SRM investment](./appendix-d-objections.md#26-what-about-srm-weve-spent-10-years-on-the-runbooks).
**Module reference:** [Module 7 § Recovery Plans vs SRM](./07-data-protection.md).

---

### Q-DR-04: DR Test Cadence and History

**Ask:** *"How often do you test DR, and what was the result of the last test?"*

**Listen for:** Honest answer (often "we should test more"), specific issues found, whether tests are real or theatrical.

**Use the answers to:** position Recovery Plans test failover capability, identify where current DR has known gaps.

**Related objections:** [#30 Test failover disruption](./appendix-d-objections.md#30-test-failover-is-too-disruptive-we-cant-do-it-quarterly).
**Module reference:** [Module 7 § Test Failover](./07-data-protection.md).

---

### Q-DR-05: Compliance and Regulatory DR Drivers

**Ask:** *"Are any of your DR requirements compliance-driven? RPO mandates, audit requirements for DR testing, specific recovery validation?"*

**Listen for:** Compliance-mandated specific RPO/RTO numbers, audit attestation requirements for DR testing.

**Use the answers to:** ensure replication mode meets compliance RPO, plan DR test cadence to satisfy audit requirements, configure WORM-protected archives where required.

**Related objections:** [#21 Compliance approval](./appendix-d-objections.md#21-our-compliance-team-wont-approve-a-new-platform), [#29 Cloud DR compliance](./appendix-d-objections.md#29-cloud-dr-isnt-for-us-compliance-requires-data-on-prem).
**Module reference:** [Scenario 4: Compliance-heavy](./appendix-c-scenarios.md).

---

## Migration (Q-MIG)

### Q-MIG-01: Workload Tier Inventory

**Ask:** *"For migration planning: how would you tier your workloads from a migration-risk perspective? Easy / moderate / complex / mission-critical?"*

**Listen for:** The customer's own risk assessment of their environment. They often know which workloads are problem children.

**Use the answers to:** plan wave sequencing (easy first, complex last), identify the hand-holding workloads.

**Related objections:** [#41 Migration risk](./appendix-d-objections.md#41-migration-is-too-risky-we-cant-afford-an-outage), [#44 Move tool](./appendix-d-objections.md#44-what-if-move-fails-on-our-complex-vms).
**Module reference:** [Module 10: Migration](./10-migration-path.md).

---

### Q-MIG-02: Application Dependency Awareness

**Ask:** *"Do you have current application dependency maps? Which VMs talk to which other VMs, what external services they depend on?"*

**Listen for:** Whether dependency mapping is current, partial, or non-existent. Most customers have partial.

**If dependency maps are current:** Faster Phase 0; migrations sequenced well.
**If they're not:** Phase 0 includes dependency mapping work (active discovery, application owner workshops, network flow analysis).

**Related objections:** [#41 Migration risk](./appendix-d-objections.md#41-migration-is-too-risky-we-cant-afford-an-outage).
**Module reference:** [Module 10 § Discovery and Dependency Mapping](./10-migration-path.md).

---

### Q-MIG-03: NSX-T and SRM Footprint

**Ask:** *"For migration planning: what's the inventory of workloads using NSX-T microsegmentation and workloads orchestrated by SRM?"*

**Listen for:** Specific counts; deep usage means hybrid steady-state likely.

**Use the answers to:** scope which workloads are migration candidates vs likely-permanent-VMware.

**Related objections:** [#16 NSX-T retention](./appendix-d-objections.md#16-what-about-nsx-t-we-have-it-deployed-and-tuned), [#26 SRM investment](./appendix-d-objections.md#26-what-about-srm-weve-spent-10-years-on-the-runbooks), [#45 Hybrid permanence](./appendix-d-objections.md#45-can-we-just-do-hybrid-permanently).
**Module reference:** [Scenario 2: Enterprise Multi-Site](./appendix-c-scenarios.md).

---

### Q-MIG-04: Team Capacity for Migration Work

**Ask:** *"Realistically, how much of your team's time can go to migration work per week? What other major projects are running in parallel?"*

**Listen for:** Honest capacity assessment, competing priorities.

**Use the answers to:** size BlueAlly augmentation, set realistic timeline expectations, identify where parallel projects might cause conflicts.

**Related objections:** [#42 Timeline pressure](./appendix-d-objections.md#42-we-dont-have-time-for-an-18-month-project), [#43 Team overload](./appendix-d-objections.md#43-our-team-is-already-overloaded-they-cant-take-on-a-migration).
**Module reference:** [Module 10 § Team Capacity](./10-migration-path.md).

---

### Q-MIG-05: Compliance and Audit Timing

**Ask:** *"What's your audit calendar over the next 18-24 months? When are SOX, PCI, internal audits scheduled?"*

**Listen for:** Specific dates that constrain migration timing.

**Use the answers to:** schedule compliance-relevant migrations between audits, not during.

**Related objections:** [#21 Compliance approval](./appendix-d-objections.md#21-our-compliance-team-wont-approve-a-new-platform).
**Module reference:** [Scenario 4: Compliance-heavy](./appendix-c-scenarios.md).

---

### Q-MIG-06: Hardware Refresh Windows

**Ask:** *"What hardware is coming due for refresh in the next 12-24 months? Compute, storage, network gear?"*

**Listen for:** Refresh dates per asset category.

**Use the answers to:** align Nutanix migration with refresh windows; capex flow becomes natural.

**Related objections:** [#38 Recent VMware renewal](./appendix-d-objections.md#38-we-just-renewed-vmware-for-3-years-bad-timing).
**Module reference:** [Module 9: Licensing](./09-licensing-economics.md), [Module 10: Migration](./10-migration-path.md).

---

## Economics (Q-ECON)

### Q-ECON-01: Current Annual Run-Rate

**Ask:** *"What's your current annual run-rate across infrastructure? Software subscriptions, hardware support, services, power, cooling, datacenter space?"*

**Listen for:** Whether they have a clean number or have to assemble it. Most customers have to assemble.

**Use the answers to:** build the apples-to-apples 5-year TCO comparison; have a real baseline.

**Related objections:** [#35 Sticker comparison](./appendix-d-objections.md#35-sticker-price-comparison-is-roughly-the-same-why-migrate), [#40 TCO skepticism](./appendix-d-objections.md#40-tco-claims-always-feel-inflated-i-dont-trust-the-numbers).
**Module reference:** [Module 9: Licensing](./09-licensing-economics.md).

---

### Q-ECON-02: Refresh Timing

**Ask:** *"What's the refresh schedule for your current infrastructure? When does each component come due?"*

**Listen for:** Same as Q-MIG-06 but with the financial framing.

**Use the answers to:** align proposal capex / opex flow with the customer's existing refresh financial plan.

**Related objections:** [#38 Recent VMware renewal](./appendix-d-objections.md#38-we-just-renewed-vmware-for-3-years-bad-timing).
**Module reference:** [Module 9: Licensing](./09-licensing-economics.md).

---

### Q-ECON-03: Capex vs Opex Preference

**Ask:** *"How does your finance team prefer to account for infrastructure investments? Capex-heavy on hardware, opex on software, or a different model?"*

**Listen for:** CFO preference, accounting policy specifics, multi-year subscription comfort level.

**Use the answers to:** structure the proposal appropriately; subscription terms aligned to accounting cycles.

**Related objections:** [#36 Subscription preference](./appendix-d-objections.md#36-we-prefer-perpetual-licensing-subscription-doesnt-work-for-us).
**Module reference:** [Module 9 § Capex vs Opex](./09-licensing-economics.md).

---

### Q-ECON-04: Hardware Sourcing Preference

**Ask:** *"Do you have an existing server-vendor relationship you'd like to preserve, or are you open to NX appliances or commodity hardware?"*

**Listen for:** Existing Dell / HPE / Lenovo / Cisco contracts, preference for single-throat-to-choke vs multi-vendor flexibility.

**Use the answers to:** choose the right sourcing path (NX vs OEM vs HCIR).

**Related objections:** [#39 Hardware lock-in](./appendix-d-objections.md#39-hardware-vendor-lock-in-is-the-real-cost-we-need-flexibility).
**Module reference:** [Module 9 § Hardware Sourcing](./09-licensing-economics.md).

---

### Q-ECON-05: Growth Projection

**Ask:** *"What's your growth projection for compute and storage over the next 3-5 years? Linear, accelerating, declining, M&A-driven volatility?"*

**Listen for:** Realistic growth assumptions, planned acquisitions, business growth that drives infrastructure.

**Use the answers to:** size cluster with appropriate headroom, recommend subscription term length (multi-year for stable; shorter with growth provisions for volatile).

**Related objections:** [#36 Subscription preference](./appendix-d-objections.md#36-we-prefer-perpetual-licensing-subscription-doesnt-work-for-us).
**Module reference:** [Module 9 § Multi-Year Subscription Mechanics](./09-licensing-economics.md).

---

## How to Sequence Discovery

**First customer meeting (kickoff, 60-90 min):**

Cover the foundation. Pick from:
- Q-GEN-01 (current infrastructure)
- Q-GEN-02 (pain points and triggers)
- Q-GEN-04 (timeline and refresh)
- Q-WL-01 (VM count and distribution)
- Q-ECON-01 (annual run-rate, light touch)
- Q-MGMT-01 (current management stack)

This establishes the situation. Don't dive deep yet.

**Second meeting (technical deep-dive on the customer's biggest concern, 2-3 hours):**

Tailor to what surfaced in kickoff. If storage was the trigger, pick from Q-STOR. If DR is the trigger, pick from Q-DR. If networking, Q-NET. If economics, Q-ECON.

**Third meeting (architecture review, 4-6 hours or full day):**

Comprehensive cross-domain. Pick 15-20 questions across all categories. By this meeting, you have rapport; the customer expects depth; the answers inform the proposal.

**Fourth meeting (commercial / decision):**

Q-ECON depth. Q-GEN-03 (decision-makers). Refine the financial story. Surface any remaining objections.

---

## What to Do With the Answers

After every discovery session:

1. **Update the customer notes.** Specific quotes are valuable; capture them.
2. **Update the risk register.** What did this session surface?
3. **Update the proposal scope.** What changed about what we're recommending?
4. **Identify follow-up questions.** What new questions does this answer raise?
5. **Identify follow-up actions.** Documents to send, references to share, technical validations to schedule.

Discovery is a continuous process, not a one-time intake. The discovery you do in week 1 informs the proposal in month 3, the design in month 6, and the operational handoff in month 18.

---

## Cross-References

- **Modules:** Each question links to the module where the topic is taught in depth.
- **Glossary:** [Appendix A](./appendix-a-glossary.md) defines the terms used in the questions.
- **Comparison Matrix:** [Appendix B](./appendix-b-comparison-matrix.md) supports the answers customers give.
- **Scenarios:** [Appendix C](./appendix-c-scenarios.md) shows how discovery answers feed into design exercises.
- **Objections:** [Appendix D](./appendix-d-objections.md) has the responses to objections that emerge from discovery answers.
- **POC Playbook:** [Appendix J](./appendix-j-poc-playbook.md) has the demo flows that often resolve technical questions faster than discussion.

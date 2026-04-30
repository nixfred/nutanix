# todo.md — Phased Build Plan

> **Read first:** `CLAUDE.md`. This file is the execution plan; CLAUDE.md is the spec. Do not skip phases. Do not one-shot. Stop at every gate and confirm with Fred before advancing.

---

## How to Use This File

- Phases run in order. A phase begins only after the previous phase's **Gate** is signed off.
- Each phase has: **Goal**, **Tasks**, **Acceptance Gate**, **Commit message**.
- The Acceptance Gate is a binary check. If any item fails, the phase is not done.
- Mark tasks complete by changing `- [ ]` to `- [x]`. Keep this file in git.
- If a task reveals that CLAUDE.md is wrong, update CLAUDE.md first, then continue.
- The **Extensibility Test** (add a file, it appears) is re-run at Phase 4, Phase 7, and Phase 13. It is the load-bearing property of this build.

---

## Phase 0 — Pre-flight: resolve open questions

**Goal:** clear blockers that affect schema and routing before any code is written. Fixing these later is expensive.

**Tasks:**

- [ ] Resolve the duplicate glossary file. `appendix-a-glossary.md` and `appendix-a-glossary-nz.md` both exist. Decide: which is canonical, what the `-nz` variant is for, whether both ship.
- [ ] Confirm appendix routing under the track. Decision: appendices live at `/nutanix/appendices/<slug>` (per-track, scales to multiple tracks). Document this in CLAUDE.md if not already.
- [ ] Decide source-of-truth layout for markdown. Recommendation: author files live in `/curriculum/nutanix/`, get copied (not symlinked) into `src/content/tracks/nutanix/` by a build script. Symlinks break on Windows and confuse Astro's content loader. Confirm with Fred.
- [ ] Confirm BlueAlly brand colors. Fetch `https://blueally.com` once; record extracted hex values, or fall back to defaults in CLAUDE.md if fetch fails. Pin them.
- [ ] Confirm hero-image strategy: AI generation (which provider? Replicate / Fal / DALL-E) or programmatic SVG. AI is faster to iterate; SVG is zero-cost and deterministic. Default to AI with caching, but get explicit confirmation.
- [ ] Confirm the missing appendices (h, i, j, k). They are referenced in CLAUDE.md as "coming." Are they in scope for v1, or v1 ships with a–g and they land later?
- [ ] Confirm the two PDFs (`03-ahv-hypervisor.pdf`, `04-prism-management.pdf`) at the project root are scratch and can be ignored or moved to `/_drafts/`.

**Gate:**
- [ ] Each open question has a recorded decision in this file or in CLAUDE.md.
- [ ] No follow-up question requires speculation in later phases.

**Commit:** `chore: phase 0 pre-flight decisions recorded; spec gaps closed`

---

## Phase TR — Technical Review of the Curriculum (runs before Phase 1)

**Goal:** every technical claim in the curriculum holds up against authoritative sources as of April 2026. The site rests on this. Wrong facts here compound everywhere.

**Method:** read each file with a fine tooth comb playing the role of a senior Nutanix engineer. For every claim about Nutanix architecture, product names, version numbers, default credentials, port numbers, exam blueprints, performance numbers, sizing rules, CLI syntax, partner-stack names (VMware, Pure, NetApp, HPE, Cisco), and competitive comparisons, verify against an authoritative source (Nutanix portal docs, Nutanix Bible, Nutanix University blueprints, vendor sites, RFCs, etc.). Record findings inline in this todo.md per file. **Add a `## References` section to the source markdown of every reviewed module/appendix**, before its `## Cross-References` block, listing the authoritative sources verified during the pass with descriptive titles and links. This is what lets a reader (or a future SA defending a claim in front of a customer) follow the chain back to the source.

**Per-file findings format:**
```
- [ ] NN-name.md — STATUS: pending | in-review | findings-recorded | revised | confirmed
  - Confirmed: ...
  - Suspect: ... (claim, source consulted, recommended revision)
  - Wrong: ... (claim, evidence, required fix)
```

**Files to review (23 total):**

Modules:
- [ ] 00-framework.md (meta-spec; check pedagogy claims, blueprint % numbers, callout taxonomy)
- [x] 01-hci-foundations.md — STATUS: findings-recorded, one correction made, References section added
  - Confirmed:
    - Prism Element URL pattern `https://<cluster-IP>:9440` and port 9440. Source: Nutanix portal docs, multiple community references.
    - Default first-login credentials `admin` / `nutanix/4u` for Prism Element (must be changed on first login). Source: Nutanix Prism 6.7 web-console docs, ervik.as/nutanix-default-credentials. Still current as of AOS 7.x.
    - CE 2.1 is the current Community Edition release. Source: portal.nutanix.com `Nutanix-Community-Edition-Getting-Started-v2_1`.
    - AOS 7.5 is current as of April 2026 (7.5.1.1 supported through Dec 31 2027). Source: endoflife.date/nutanix-aos, eosl.date.
    - Nutanix v4 REST API is GA (released with PC 2024.3 / AOS 7.0), so the curriculum's "Nutanix v4 REST" reference is correct. Source: nutanix.com/blog/announcing-the-v4-api-and-sdk-general-availability.
    - "NC2 (Nutanix on AWS bare metal)" naming matches official "Nutanix Cloud Clusters (NC2) on AWS." Source: nutanix.com/products/nutanix-cloud-clusters/aws.
    - 2-node ROBO Nutanix offering exists (real product line).
  - Corrections applied:
    - Lab Path A: changed "32 GB RAM minimum" to "32 GB RAM recommended (16 GB is the documented absolute minimum, but CE with Prism Central is realistically 32 GB+)." The portal docs list 16 GB as minimum; 32 GB was the curriculum's practical recommendation mislabeled as the "minimum."
  - Suspect (kept as-is, flagged for cross-file pass):
    - "AOS 7.5 + all-NVMe routinely delivers sub-millisecond latency for OLTP" — directional claim, no citation. Curriculum already advises the SA to bring concrete benchmark data, so framing is acceptable. Re-evaluate against the Module 05 storage-deep-dive treatment of latency claims.
    - Cert blueprint estimates "NCA ~12% / NCP-MCI ~5% / NCM-MCI ~2%" for HCI fundamentals are plausible but unsourced. Real fix belongs in 00-framework.md and appendix-k-cert-tracker.md once the official current Nutanix blueprint PDFs are pulled and reconciled. Flag for the framework pass.
    - "50-70% reduction in infrastructure operational hours" already self-flagged in the curriculum text ("verify against your own customer base before quoting specific percentages"). Acceptable as-is.
    - "Position Nutanix Files / Mine for backup" — Files (file storage) and Mine (backup-integration platform) are different products; the slash reads ambiguous but not technically wrong. Optional clarity edit on a future copy pass.
- [x] 02-nutanix-architecture.md — STATUS: findings-recorded, no corrections required, References section added
  - Confirmed:
    - CVM service list (Stargate = data path; Cassandra = distributed metadata; Curator = background scrub/rebalance/ILM; Zeus = cluster config + ZooKeeper interface; Pithos = vDisk config; Acropolis = AHV VM lifecycle). Source: nutanixbible.com/2f-book-of-basics-cluster-components, next.nutanix.com community knowledge base.
    - Cassandra characterization "forked, optimized version of Apache Cassandra" matches Nutanix Bible's "heavily modified Apache Cassandra."
    - Pithos still current as a per-node vDisk config manager built on top of Cassandra.
    - Three-node minimum production cluster confirmed (Zeus/ZooKeeper quorum requirement).
    - Two-node ROBO cluster requires external Witness VM in separate failure domain. Witness VM minimum spec confirmed: 2 vCPU / 6 GB RAM / 25 GB disk; latency to cluster ≤ 500 ms; cannot run on AWS or Azure. Source: portal.nutanix.com BP-2083 ROBO Deployment best practices.
    - Cluster create command syntax `cluster -s <ip1>,<ip2>,<ip3> create` exact match to current Nutanix nCLI docs (AOS 6.8+ nCLI reference).
    - OEM partner mapping: Dell XC, HPE DX, Cisco UCS, Lenovo HX, plus Fujitsu and Supermicro. Curriculum's list (Dell XC, HPE DX, Cisco UCS, Lenovo HX, Supermicro, plus NX) is accurate; Fujitsu missing but optional.
    - Foundation Standalone (laptop) vs Foundation Central (Prism Central, fleet scale) split is correct.
    - LCM as the day-two coordinated upgrade tool (AOS, AHV, BIOS, BMC, NIC firmware, drive firmware, NCC, Foundation, Files, Objects, Volumes, NKE) confirmed.
    - NCC `ncc health_checks run_all` command syntax confirmed.
    - Default CVM SSH user `nutanix` with default password `nutanix/4u` (forced change on first login) confirmed.
    - Cluster external IP via `ncli cluster set-external-ip-address external-ip-address=<vip>` is valid AOS 6.8+ syntax.
    - Block awareness as fault-domain feature for chassis-level redundancy confirmed.
    - RF2/RF3 semantics, single-node failure recovery flow (HA restart + Curator-driven re-replication) confirmed.
    - Data locality semantics (VM moves but data follows over time via background services) confirmed.
  - Suspect (kept as-is, flagged for cross-file pass):
    - CVM resource sizing table is labeled "AOS 7.5 generation" with specific vCPU/RAM numbers (8/32 minimum, 12/48 typical, 14-16/64+ heavy). The Nutanix Support Portal page for AOS 7.5 CVM specifications is gated; could not verify exact numbers from public sources. Historical defaults were 8 vCPU / 16 GB with 24 GB floor at AOS 5.0; the curriculum's higher numbers are plausible for AOS 7.5 with production features but should be cross-checked with portal access during Phase TR finalization.
    - Service list omits Medusa, the abstraction layer that fronts Cassandra. Pedagogically reasonable (treating Cassandra as "the metadata service" hides implementation detail), but worth flagging for Module 05 (DSF deep dive) consistency check; if Module 05 invokes Medusa by name the abstraction needs to be introduced here too.
  - No source-markdown corrections applied; module's technical claims hold.
- [x] 03-ahv-hypervisor.md — STATUS: findings-recorded, four corrections made, References section added
  - Corrections applied:
    - **ADS expansion fixed.** Curriculum had "ADS (Acropolis Distributed Scheduling)" in two places (front matter and the ADS section heading). Authoritative Nutanix sources (Nutanix Bible, AHV Admin Guide v6.8, BP-2029) call it **Acropolis Dynamic Scheduling** (or "Acropolis Dynamic Scheduler"). Fixed both. Added a sentence noting the internal service name is Lazan and that it appears in logs and `acli` output.
    - **vSphere Foundation pricing rewritten.** The original text said "vSphere Foundation pricing is now in the range of $350 to $500 per core per year" which conflated vSphere Foundation MSRP (~$190/core) with VMware Cloud Foundation (~$350/core). Replaced with a SKU-aware paragraph: vSphere Foundation $190/core, VCF $350/core (down from $700), 16-core CPU minimum, 72-core order minimum. Recomputed the 256-core example correctly: $48k/yr (vSphere Foundation) or $90k/yr (VCF) before uplift.
    - **Q7 explanation updated.** The math in the explanation no longer matches the answer choice C cleanly without the SKU clarification. Rewrote the explanation to specify VCF tier with support uplift gets you to the C range, and to call out that vSphere Foundation alone would land at answer B. Preserves the order-of-magnitude pedagogical intent.
    - **Added 72-core minimum order note.** Broadcom's post-2025 minimum order constraint is a meaningful gotcha in customer pricing conversations and was missing.
  - Confirmed:
    - AHV is KVM-based with QEMU device emulator, libvirt management primitives, Open vSwitch networking, and Acropolis as the Nutanix-specific control plane.
    - Acropolis runs as a service inside every CVM with master/follower election via Zeus/ZooKeeper.
    - HA recovery time order of 30-90 seconds (in the same range as vSphere HA).
    - ADS default polling interval 15 minutes; rebalance triggers 30-minute backoff (not in curriculum but consistent with curriculum's "evaluates persistent pressure, not transient spikes" framing).
    - Nutanix Move is free, deployed as a VM, supports ESXi/Hyper-V/AWS/Azure/GCP/other-Nutanix as sources, performs change-tracked bulk copy + brief cutover.
    - Move cutover downtime envelope (5-15 minutes per VM) accurate per Nutanix Move 5.5 docs and Bible.
    - NGT capability list (time sync, IP detection, VirtIO drivers, VSS-aware Windows snapshots, FLR, self-service file restore) accurate.
    - Calm renamed to NCM Self-Service in 2022; curriculum's dual naming is appropriate.
    - NKE (Nutanix Kubernetes Engine) branding still current; the user-facing label is shifting to "Kubernetes Management" starting NKE 2.8 / PC 2023.1.0.1 but the curriculum's NKE reference remains accurate at the product level.
    - FT (Fault Tolerance / lockstep CPU) gap on AHV confirmed.
    - DSF-native snapshot semantics (instant, redirect-on-write, no consolidation) confirmed.
  - Suspect (kept as-is):
    - Acropolis status page on port 2030 — could not verify from public sources; likely correct (internal status pages on per-CVM ports are common in Nutanix services). Re-validate during Phase TR finalization.
    - "ADS recommends or executes Live Migrations" — accurate; ADS can run in advisory or enforcing mode depending on cluster config. Curriculum doesn't distinguish; pedagogically OK.
- [x] 04-prism-management.md — STATUS: findings-recorded, three corrections made, References section added
  - Corrections applied:
    - **Ansible collection name fixed.** Curriculum had `nutanix.ncp` (not a real collection). Correct name is `nutanix.ansible` per the official Nutanix Ansible repo and Nutanix.dev docs. Fixed.
    - **v4 API URL pattern fixed.** Curriculum's curl examples used `https://<pc-ip>:9440/api/nutanix/v4/<namespace>/<version>/<path>` with an extra `/nutanix/v4/` prefix. The actual pattern per the Nutanix v4 API User Guide is `/api/<namespace>/<version>/<path>`, e.g., `/api/vmm/v4.0/ahv/config/vms`. Rewrote the three curl examples and added a one-line URL-pattern comment so a reader sees the canonical shape.
    - **NCM tier description rewritten.** Curriculum claimed "NCM Starter. Included with PC." That is wrong; NCM Starter is a paid SKU separate from NCI. Rewrote to clarify the NCI / NCM / NCP split: basic Prism Central (multi-cluster, Categories, Projects, RBAC, v4 API) is included with NCI; NCM tiers (Starter / Pro / Ultimate) are separate paid licenses; NCP bundles combine them. Updated the capability table to reflect what is actually included with NCI vs each NCM tier.
  - Confirmed:
    - Prism Element is per-cluster, in-cluster, included with NCI.
    - Prism Central is a separately-deployed VM for multi-cluster management.
    - PC has multiple sizing tiers including X-Small (5 clusters / 50 hosts / 500 VMs), Small, and scale-out 3-VM. Curriculum's "12,500 VMs / 100 clusters" small-tier number could not be confirmed from public sources; flagged for portal cross-check.
    - Categories drive policy enforcement (backup, DR, microsegmentation, quotas) — not just metadata. The vSphere tags comparison is accurate.
    - X-Play is event-driven automation tied to NCM Self-Service / Calm (not standalone). Pedagogical framing is consistent with current Nutanix product structure.
    - SAML 2.0 SSO supported with Azure AD / Entra ID, Okta, Ping Identity. AD via LDAP also supported.
    - v4 REST API URL pattern corrected; namespaces (VMM, ClusterMgmt, IAM, etc.) confirmed via developers.nutanix.com.
    - Terraform provider `nutanix/nutanix` confirmed (latest v2.4.2 as of 2026).
    - Python SDK packages namespaced as `ntnx-<namespace>-py-client` (e.g., `ntnx-vmm-py-client`, `ntnx-clustermgmt-py-client`) confirmed.
    - v3 (and earlier) APIs deprecating starting Q4 2026 — worth surfacing in Module 04's "What Prism has that VMware doesn't" section in a future copy pass.
  - Suspect (kept as-is):
    - PC lab specs "vCPU: 6, RAM: 26 GB" do not match either current public source (small instance is documented as 4 vCPU / 18 GB; 3-VM scale-out is 6 vCPU / 28 GB per VM). Curriculum's numbers may be a mid-version data point or an aggregate; flagged for portal cross-check during Phase TR finalization.
    - PowerShell module names `Nutanix.Prism.Common`, `Nutanix.Prism.PS.Module` could not be confirmed from public sources; the official PowerShell SDK lives at github.com/nutanix/nutanix-powershell-sdk. Flag for cross-check.
    - "Pulumi provider" mentioned; current Pulumi support for Nutanix appears to be community-maintained, not official Nutanix. Worth a clarifying note in a future copy pass.
- [x] 05-dsf-storage-deep-dive.md — STATUS: findings-recorded, three corrections made, References section added
  - Corrections applied:
    - **Extent Group size is 1 MB or 4 MB, not just 4 MB.** Curriculum claimed "Extent Group: a 4 MB physical allocation unit." Per the Nutanix Bible and the Polar Clouds DSF walkthrough, an extent group is **1 MB on non-deduplicated containers** and **4 MB on deduplicated containers**. Updated the hierarchy definition and the related ON-THE-EXAM callout.
    - **EC 4+1 minimum cluster size is 6 nodes, not 5.** Curriculum's EC table and the rf-vs-ec-comparison diagram both said "Min cluster: 5 nodes" for 4+1. Per Nutanix's TN-2032 erasure-coding solution brief, the optimal 4+1 stripe is supported on clusters of ≥ 6 nodes (the math is 5 nodes for the stripe + 1 for rebuild headroom). Updated the table, added an explanatory paragraph, and updated the diagram annotation.
    - **Compression algorithm is LZ4 + LZ4HC, not LZ4 + zstd.** Curriculum claimed "LZ4 by default with optional zstd for higher ratios at higher CPU cost." Per the Nutanix Bible AOS Storage chapter and the TN-2032 compression brief, DSF uses **LZ4 for inline compression** (latency-critical path) and **LZ4HC for cold-data / post-process compression** (better ratio at higher CPU). Zstd is not used in standard DSF. Rewrote the algorithm paragraph and added the inline-selectivity detail (>64K sequential streams only).
  - Confirmed:
    - Storage hierarchy Pool → Container → vDisk → Extent → Extent Group is accurate.
    - Extent (logical) is 1 MB; Cassandra tracks each extent's physical location.
    - OpLog is persistent, on local hot tier (NVMe/SSD), serves as write buffer; writes acknowledged after local + remote OpLog persist (RF2 = 1 remote, RF3 = 2 remote); Extent Store drain is asynchronous.
    - Stargate write characterizer routes random/bursty writes to OpLog and sustained sequential writes directly to Extent Store. Curriculum has this correctly.
    - Content Cache is in CVM RAM. Confirmed.
    - Cassandra metadata replication is 3 copies by default (configurable up to 5). Curriculum doesn't quantify this but the architectural framing is correct.
    - RF2 = 50% effective, RF3 = 33% effective. Confirmed.
    - EC 4+2 minimum cluster size is 7 nodes. Confirmed (curriculum was correct on this row).
    - Curator handles re-replication, EC conversion, ILM, capacity reclamation, rebalancing. Confirmed.
    - Curator runs full scans every 6-24 hours and partial scans more frequently. Order-of-magnitude correct per Nutanix Bible.
    - Storage-only nodes contribute capacity to the pool but minimal compute. Confirmed.
    - LZ4HC mention adds value: cold-tier data benefits from the higher-compression variant.
  - Suspect (kept as-is):
    - Capacity reservation "10-15%" is a planning rule of thumb. Real reservations vary with cluster size and configuration; the curriculum's range is reasonable but not precisely sourced.
    - "Bigger clusters rebuild faster (more nodes share the work)" — directionally true; specific time estimates are workload-dependent.
    - The DSF-internal "Cassandra" terminology versus the customer-workload "Cassandra-on-Nutanix" reference (BP-2007) could trip a careful reader; pedagogically OK as the curriculum is clear that DSF's Cassandra is the metadata service.
- [x] 06-networking-flow.md — STATUS: findings-recorded, four corrections made, References section added
  - Corrections applied:
    - **Flow Network Security licensing was wrong (2 places).** Curriculum claimed Flow Network Security "requires NCM Pro or higher" and "Flow Network Security at NCM Pro is included." Per the Nutanix Cloud Platform software-options page and Flow product page, Flow Network Security is licensed via **NCI Ultimate** or via the **Security Add-On for NCI Pro** (the Security Add-On is per-usable-TiB and bundles Flow microsegmentation with Data-at-Rest Encryption, software and SED). Flow is **not** an NCM tier feature. Rewrote both places to make this explicit and to warn against confusing NCM management tiers with Flow's NCI-side licensing. Also softened the "licensing change" framing to acknowledge per-TiB pricing and the encryption bundling.
    - **LACP recommendation tightened.** Curriculum's bond-mode discussion described LACP as "Best throughput with proper switch coordination" and "Production deployments with proper switch configuration." Per Nutanix's official posture (community thread + Nutanix Bible), Nutanix has historically leaned **away from LACP** because misconfigured upstream switches can disable cluster connectivity in hard-to-recover ways. Added a sentence calling this out and emphasizing the documentation-and-validation discipline if a customer chooses LACP.
    - **NCP-NS introduction date refined.** Curriculum said "introduced in 2025." Public scheduling actually opened April 4, 2026 for NCP-NS 7.5. Updated to the exact name (NCP-NS = Nutanix Certified Professional – Network and Security) and the April 2026 launch date.
  - Confirmed:
    - OVS bridges br0 (data) and br0.local (management) confirmed.
    - Bond modes (active-backup, balance-slb, balance-tcp, LACP) and their semantics confirmed.
    - Default new-cluster bond mode is active-backup. Confirmed.
    - balance-slb operates without switch-side LACP; balance-tcp and LACP do require LACP-configured switches. Confirmed.
    - Flow Virtual Networking VPC overlay confirmed (PC 2022.x and forward); BGP integration, NAT, service insertion all verified.
    - manage_ovs CLI tool, ovs-vsctl, ovs-ofctl as troubleshooting tools confirmed.
    - Flow rules enforced at OVS bridge level on each host (distributed enforcement, not central).
    - Categories drive Flow policy (vs IP-based ACLs); vSphere tag comparison accurate.
    - Stateful firewall rules with explicit-allow / explicit-deny default policy.
    - Service insertion for Palo Alto VM-Series, Check Point, Fortinet pattern confirmed.
    - NSX-T Federation comparison framing accurate (NSX-T Federation is a real multi-site product; FVN is younger and catching up).
  - Suspect (kept as-is):
    - "vlan.<id>" naming convention is a curriculum-style suggestion; Nutanix doesn't enforce a specific naming convention for Virtual Networks. Pedagogically OK.
    - Lab notes "Flow Network Security on Community Edition is limited; some features may not be available." Acceptable; CE Flow capabilities are version-dependent and the curriculum's caveat is appropriate.
- [x] 07-data-protection.md — STATUS: findings-recorded, three refinements made, References section added
  - Refinements applied:
    - **Recovery Plans / Leap naming clarified.** Curriculum said "Recovery Plans (sometimes called Leap, the original product brand)." The current product name is **Nutanix Disaster Recovery** (which was renamed from Leap a few years back); Recovery Plans is the runbook construct *inside* Nutanix DR. Updated the framing to make this explicit while preserving the Leap name for SAs who still use it in customer conversations.
    - **LWS storage location added.** Curriculum mentioned LWS but didn't say where the LWS store lives. Per TN-2027, the LWS store is allocated on the cluster's SSD tier; that is where NearSync-protected changes land first. Added the detail.
    - **Metro latency design guidance added.** Curriculum said "<5ms RTT typically." The 5 ms number is the documented hard ceiling, not a typical design target. Production architectures aim for ≤ 3.5 ms RTT under load (30% headroom), and P99.9 latency under concurrent I/O matters more than the average. Rewrote the latency-requirement bullet to capture this distinction.
  - Confirmed:
    - Crash-consistent vs application-consistent snapshot distinction; NGT/VSS coordination on Windows.
    - Protection Domains in Prism Element (manual VM membership, legacy) vs Protection Policies in Prism Central (category-driven, modern); both coexist; migration is supported but disruptive.
    - Async replication minimum RPO 1 hour typical (15 min in some configs); delta-based; bandwidth-efficient; works over WAN at any latency.
    - NearSync RPO 20 seconds achievable in optimal configurations (AOS 5.17+); 1-15 minute RPO is the typical configured range; LWS metadata-level micro-snapshots; <5ms RTT typical; higher Stargate/Curator overhead than Async.
    - Metro Availability: synchronous, RPO zero, witness VM in separate failure domain for split-brain protection, metro distance only (typically <100km).
    - Recovery Plans: VM startup order, network mapping, IP remapping, pre/post scripts, test failover into isolated network.
    - SRM comparison framing accurate: SRM more mature for complex orchestration; Recovery Plans integrated and capable for typical enterprise DR.
    - NC2 cloud DR target on AWS or Azure bare-metal; spin-up patterns for cost optimization.
    - Witness VM specs and constraints (covered in Module 02 review) consistent.
  - Suspect (kept as-is):
    - "Recovery Plans bundled with Nutanix Cloud Manager (varies by tier)" in the SRM comparison table — the licensing is actually NCI-tier-driven (basic Async with NCI; advanced DR features tied to NCI Pro/Ultimate). Worth a future copy pass to align with the corrected NCM/NCI terminology established in Modules 04 and 06.
    - Quarterly test failover cadence is a recommendation, not a Nutanix-mandated number. Pedagogically OK.
- [x] 08-unified-storage.md — STATUS: findings-recorded, four corrections made, References section added
  - Corrections applied:
    - **FSVM minimum count clarified.** Curriculum said "Three FSVMs per File Server is the typical minimum for HA and scale-out." Per TN-2041, **single-FSVM File Servers are supported for small clusters** (one- and two-node Nutanix clusters, or AOS 5.10.1+ small-environment scenarios). Updated to call out the exception while preserving "three FSVMs is typical for HA + distributed shares."
    - **Files Analytics → Data Lens product evolution surfaced.** Curriculum framed analytics entirely as "Files Analytics." The current Nutanix product is **Data Lens** (cloud-based and, with v2.0 GA in 2026, fully on-premises including air-gapped). Files Analytics persists as the on-cluster service but Data Lens is the broader unified-storage governance / ransomware-detection product. Updated three places: the Files capabilities bullet, the analytics section heading and intro, and the anti-ransomware section to mention the 65,000+ ransomware signature library Data Lens carries.
    - **Objects WORM detail added.** Curriculum had a one-liner on WORM. Per TN-2106 and the Nutanix Community WORM blog, WORM uses the S3 Object Lock specification with specific semantics worth knowing: 24-hour grace period after enabling, retention can be extended but never reduced, versioning auto-enabled and cannot be suspended on a WORM bucket. Added all of this for NCP-US and customer-compliance conversations.
    - **Anti-ransomware tied to Data Lens.** Curriculum's anti-ransomware section was framed as a Files-only feature. Updated to reflect that Data Lens is the broader product driving ransomware detection across Files (and increasingly across Unified Storage).
  - Confirmed:
    - File Server is the logical SMB/NFS service; FSVMs are the dedicated VMs implementing it; data lives on DSF.
    - SMB 2.x / 3.x and NFS v3 / v4 with Kerberos and AD integration confirmed.
    - Multi-protocol shares (SMB and NFS on the same data with ACL translation) confirmed.
    - Self-Service Restore via Windows "Previous Versions" tab confirmed.
    - Nutanix Objects: S3-compatible API, buckets, versioning, WORM (S3 Object Lock), lifecycle policies, replication to other Objects deployments or AWS S3.
    - Object Service VMs as the implementation of Objects (architecture parallel to FSVMs for Files).
    - Nutanix Volumes: iSCSI block storage, Volume Groups as the management unit, multi-pathing via multiple portal IPs.
    - Veeam, Commvault, Rubrik, Cohesity, HYCU all support S3-compatible storage as backup targets.
    - NCP-US covers Files, Objects, Volumes (curriculum's ~80% claim is plausible for this module's coverage of NCP-US blueprint).
  - Suspect (kept as-is):
    - "Distributed shares only available on three or more FSVMs" — TN-2041 confirms this; curriculum's three-FSVM-typical framing aligns once the single-FSVM exception is noted (now done).
    - `afs` CLI command for Files-specific operations — could not verify exact command name from public sources; flag for portal cross-check.
- [x] 09-licensing-economics.md — STATUS: findings-recorded, two structural corrections made, References section added
  - Corrections applied:
    - **AOS Pro / AOS Ultimate → NCI Pro / NCI Ultimate.** The single biggest finding. Per the Nutanix Cloud Platform software-options page and the License Manager conversion docs, **AOS Pro and AOS Ultimate are legacy and no longer available for new sale or renewal**; existing AOS customers are being converted to NCI Pro / NCI Ultimate. Curriculum was using the legacy "AOS Pro / Ultimate" naming throughout. Rewrote the main "AOS Subscription Tiers" section as "NCI Subscription Tiers (formerly AOS)" with explicit explanation of the rename, the conversion requirement (AOS 6.1.1+ / PC 2022.4+ / NCC 4.5.0+), the dual-naming reality during transition, and the existence of NCI-Compute (NCI-C) as a compute-only SKU. Updated front matter key_terms, the ON-THE-EXAM callout, and the tier-feature-matrix diagram column names. Added NCP bundles (NCI + NCM combined SKUs) which the original module didn't surface.
    - **NCM Starter "included with PC" wrong.** Same error I already corrected in Module 04 — propagated here. NCM Starter is a paid SKU; what's included with NCI baseline is basic Prism Central (multi-cluster, Categories, Projects, RBAC, v4 API). Rewrote the NCM tier section to match Module 04's correction and added the NCI/NCM/NCP packaging structure for clarity.
  - Confirmed:
    - AHV is included with NCI / AOS at every tier (no separate hypervisor licensing). Confirmed.
    - Per-core licensing model. Confirmed.
    - Multi-year subscription terms (1, 3, 5 year) with discount tiers. Confirmed.
    - Hardware sourcing options: NX (Nutanix-branded, manufactured by Super Micro), OEM (Dell XC, Lenovo HX, HPE DX, Cisco UCS), software-only on HCIR commodity hardware. Confirmed.
    - Capex vs opex framing accurate.
    - 5-year TCO methodology, BoM categories (hardware, software, services, support, training, migration, operations, decommissioning) accurate.
    - Broadcom comparison framing accurate (already corrected in Module 03 with the precise vSphere Foundation $190 / VCF $350 numbers; Module 09's general framing here aligns with that).
  - Suspect (kept as-is):
    - "Files licensing typically per-TB or per-FSVM model" — packaging varies; flag for verification against current pricing tools.
    - "Objects licensing per-TB capacity" — same caveat.
    - Practice questions Q1-Q12 use AOS terminology rather than NCI; pedagogically OK with the rename note added to the ON-THE-EXAM callout, since exam material is currently transitioning. Future copy pass could update the QA wording.
    - Lab Exercise references "AOS Pro per-core" in steps; update in a future pass.
- [x] 10-migration-path.md — STATUS: findings-recorded, no source corrections required, References section added
  - Confirmed:
    - Move tool source/target paths (ESXi, Hyper-V, AWS, Azure, AHV-to-AHV, NC2 paths) consistent with Module 03's verification.
    - Cutover-downtime envelope of 5-30 min per VM is acceptable as the wider honest range; Module 03 used 5-15 min as the typical envelope. Both are accurate within their framing.
    - Move appliance runs on the target Nutanix cluster, connects to source vCenter for ESXi migration via API, performs initial replication + incremental sync + brief cutover.
    - VMware Tools removal and NGT installation handled "in some scenarios" hedge is appropriate (Move does swap drivers in many cases but specific guest configurations may require manual handling).
    - Move does NOT handle: NSX-T microsegmentation policies, complex application-specific configurations, vSAN-specific storage policies, old snapshot chains. Consistent with TN-2072 documentation.
    - 7-phase migration framework is industry-standard project methodology, not a Nutanix product. Acceptable as BlueAlly's recommended engagement structure.
    - Discovery tools mentioned (Nutanix's discovery utility, RVTools, Lansweeper, NetFlow / application dependency mapping) are real tools commonly used in migration projects.
    - Risk register categories, communication playbook, year-2 stable state are project-management practice rather than verifiable technical claims.
    - Hybrid steady-state framing accurate (some workloads remain on VMware permanently for legitimate reasons; NSX-T routing complexity, NetApp ONTAP-specific workflows, SRM-orchestrated complex DR, third-party dependencies all genuine).
  - No corrections required. Module 10 is mostly synthesis / engagement methodology with thin technical surface, and the technical claims that exist are consistent with verifications already performed in earlier modules.

**ALL 10 MODULES COMPLETE.** Total corrections across the modules:
- Module 01: 1 correction (CE RAM minimum)
- Module 02: 0 corrections (clean)
- Module 03: 4 corrections (ADS rename, Broadcom pricing, Q7 explanation, 72-core minimum)
- Module 04: 3 corrections (Ansible collection name, v4 API URL pattern, NCM tier description)
- Module 05: 3 corrections (extent group size, EC 4+1 minimum, compression algorithm)
- Module 06: 4 corrections (Flow licensing × 2, LACP recommendation, NCP-NS date)
- Module 07: 3 refinements (Leap rename, LWS storage tier, Metro latency headroom)
- Module 08: 4 corrections (FSVM minimum exception, Files Analytics → Data Lens, WORM detail, anti-ransomware → Data Lens)
- Module 09: 2 structural corrections (AOS → NCI tier rename, NCM Starter packaging)
- Module 10: 0 corrections (clean synthesis module)

**Total: 24 substantive corrections + 10 References sections added across 10 modules.** Twelve appendices remain.

Appendices:
- [x] appendix-a-glossary.md (A-M) — STATUS: findings-recorded, eight corrections + new entry, References section added
  - Phase 0 duplicate question resolved: `appendix-a-glossary.md` covers A-M and `appendix-a-glossary-nz.md` covers N-Z. They are companion halves, not duplicates. Front matter and intros confirm the split. No merge needed; the file pair is intentional.
  - Corrections applied:
    - **ADS heading: "Distributed Scheduler" → "Dynamic Scheduling"** (matches Module 03). Added Lazan internal-service-name note and 15-minute polling default.
    - **AOS / AOS Pro / AOS Ultimate**: rewritten to reflect the NCI rename. AOS entry now points to NCI; AOS Pro and AOS Ultimate entries marked as legacy with redirects to NCI Pro / NCI Ultimate (matches Module 09).
    - **Anti-ransomware**: tied to Data Lens with the 65,000+ signature library detail (matches Module 08).
    - **Compression (DSF)**: corrected algorithm pair from "LZ4 + zstd" to "LZ4 + LZ4HC" with selectivity detail (matches Module 05).
    - **Extent Group**: corrected from fixed "4 MB" to "1 MB or 4 MB depending on dedup" (matches Module 05).
    - **Files Analytics**: surfaced the Data Lens product evolution (matches Module 08).
    - **Flow Network Security**: corrected licensing from "NCM Pro typically" to "NCI Ultimate or Security Add-On for NCI Pro" (matches Module 06).
  - Added: **Data Lens** entry (new in glossary; tied to Files Analytics evolution and Module 08 corrections).
- [x] appendix-a-glossary-nz.md (N-Z) — STATUS: findings-recorded, four corrections + three new entries, References section added
  - Corrections applied:
    - **NCM**: rewrote to clarify NCM is paid-and-separate-from-NCI, NCM Starter is not bundled with PC (matches Modules 04, 09).
    - **Recovery Plan (Leap)**: heading expanded to "Recovery Plan (Nutanix Disaster Recovery / formerly Leap)" with the rename history (matches Module 07).
    - **Witness VM**: added the authoritative spec (2 vCPU / 6 GB / 25 GB / ≤500 ms / not on AWS or Azure) from BP-2083 (matches Module 02).
    - **v4 API**: corrected URL pattern, added v4 deprecation Q4 2026 note, fixed Ansible collection name to `nutanix.ansible`, flagged Pulumi as community not official (matches Modules 04, 06).
  - Added entries: **NCI**, **NCI Pro**, **NCI Ultimate**, **NCP** (Nutanix Cloud Platform bundles). These four entries make the new licensing structure searchable in the glossary.
- [x] appendix-b-comparison-matrix.md — STATUS: findings-recorded, six corrections made, References section added
  - Corrections applied:
    - **Front matter `covers:` field**: "AOS subscription vs VCF" → "NCI subscription, formerly AOS, vs VCF" (alignment with Module 09 NCI rename).
    - **Flow vs NSX-T licensing row**: "NCM Pro tier" → "NCI Ultimate, or Security Add-On for NCI Pro (per usable TiB)" (matches Module 06 fix). Updated the honest-assessment paragraph with the accurate licensing framing.
    - **Recovery Plans (Leap) heading**: expanded to "Nutanix Recovery Plans (Nutanix Disaster Recovery / formerly Leap)" (matches Module 07 rename).
    - **NCM vs Aria Suite table**: rewrote the "Multi-cluster mgmt | Starter (with PC)" row to clarify that basic multi-cluster, Categories, Projects, RBAC, and v4 API are included with NCI baseline (no NCM required); paid NCM tiers add features on top (matches Modules 04 and 09 corrections).
    - **HyperFlex EOL row**: replaced vague "EOL announced in 2024" with authoritative dates from Cisco's official EOL announcement: last order Sep 11, 2024; last bug-fix support Sep 11, 2025; last subscription renewal Feb 28, 2029. Updated the honest-assessment paragraph to name the Cisco-Nutanix partnership product **Cisco Compute Hyperconverged with Nutanix** as the recommended migration path on qualifying M6 hardware.
    - **AOS Subscription vs VCF Subscription** licensing comparison rewritten as "NCI Subscription (formerly AOS) vs VCF Subscription" with current pricing references: vSphere Foundation ~$190/core, VCF ~$350/core (down from $700), 16-core CPU minimum, 72-core order minimum (matches Modules 03 and 09 corrections).
  - Confirmed:
    - DSF vs vSAN architectural framing accurate; CVM 8-16 vCPU / 32-64 GB consistent with Module 02.
    - DSF vs traditional arrays trade-offs accurate.
    - Files vs NetApp ONTAP trade-offs accurate (FlexClone, FlexCache, ABE, SnapMirror).
    - Files vs Isilon, Files vs FlashBlade comparisons accurate.
    - Objects vs AWS S3 trade-offs accurate.
    - Volumes vs purpose-built arrays accurate.
    - Replication modes comparison (Async / NearSync / Metro) consistent with Module 07.
    - NC2 vs VMC on AWS comparison accurate.
    - VxRail comparison accurate.
    - Nutanix vs Azure Stack HCI comparison accurate.
    - Move vs alternatives (HCX, Zerto, RackWare) comparison accurate.
    - Hardware sourcing (NX / OEM / HCIR) accurate.
  - No content gaps identified beyond corrections above.
- [x] appendix-c-scenarios.md — STATUS: findings-recorded, two corrections made, References section added
  - Corrections applied:
    - **Scenario 1 TCO line**: "AOS Pro + NCM Pro" → "NCI Pro (formerly AOS Pro) + NCM Pro" (matches Module 09 NCI rename).
    - **Scenario 9 subscription line**: "3-year AOS Pro + NCM Pro term" → "3-year NCI Pro (formerly AOS Pro) + NCM Pro term" (same fix).
  - Confirmed:
    - Ten scenarios are synthesis / engagement exercises with thin direct-technical surface; the technical claims they reference (RF/EC math, replication modes, NCI/NCM tiers, Flow licensing, HyperFlex EOL, Move tool, hybrid steady-state) all derive from modules where corrections have already been applied.
    - HyperFlex EOL reference in Scenario 4 is consistent with Appendix B's authoritative dates (Sep 2024 last order).
    - NSX-T migration framing in Scenarios 1 and 2 consistent with Module 06's accurate Flow vs NSX-T comparison.
    - SRM and Recovery Plans framing in Scenario 2 consistent with Module 07's Leap → Nutanix DR rename.
    - Cisco UCS-based Nutanix migration path in Scenario 4 aligns with the OEM partnership product (Cisco Compute Hyperconverged with Nutanix) surfaced in Appendix B.
    - Greenfield hardware sourcing recommendations (HCIR / OEM / NX) in Scenario 9 consistent with Module 09.
  - References section added linking back to the per-module References sections that source the technical specifics.
- [x] appendix-d-objections.md — STATUS: findings-recorded, two corrections made, References section added
  - Corrections applied:
    - **Objection #16 (NSX-T)**: Flow Network Security licensing claim "included licensing in NCM Pro" → "licensed via NCI Ultimate or as a Security Add-On for NCI Pro (per usable TiB; bundles Data-at-Rest Encryption)" (matches Module 06 Flow licensing fix).
    - **Objection #29 (SRM)**: "Recovery Plans (Leap)" → "Recovery Plans (the runbook construct inside Nutanix Disaster Recovery, formerly branded Leap)" + replaced vague "bundled with appropriate NCM tier" with the accurate breakdown (Async with NCI baseline, NearSync/Metro with NCI Ultimate, Recovery Plans included with Nutanix Disaster Recovery in Prism Central).
  - Confirmed:
    - The objection-handling appendix is mostly engagement scripts (response language, what NOT to say, see-also pointers). The technical claims it does make are consistent with the modules where they're sourced.
    - HyperFlex past-experience reference (#1) consistent with Appendix B's authoritative EOL framing.
    - Aria-vs-NCM coexistence framing (#10) consistent with Module 04's accurate parallel-running pattern.
    - Array-based-replication coexistence framing accurate.
    - Test-failover description accurate (consistent with Module 07).
  - References section added linking back to the per-module References sections that source the technical specifics (Modules 03, 06, 07, 09, Appendix B).
- [x] appendix-e-discovery-questions.md — STATUS: findings-recorded, no corrections needed, References section added
  - Confirmed: Appendix E is pure engagement methodology (questions to ask, what to listen for, what to do with answers across Q-GEN, Q-WL, Q-MGMT, Q-AUT, Q-STOR, Q-NET, Q-SEC, Q-DR, Q-MIG, Q-ECON families). It deliberately avoids restating technical specifications.
  - Scanned for known error patterns (AOS Pro/Ultimate, NCM Starter packaging, Distributed Scheduler, Files Analytics-only, Flow-on-NCM-Pro, /api/nutanix/v4 prefix, extent-group 4 MB, zstd, Karbon, nutanix.ncp): zero matches.
  - References section added linking each question family to its underlying module's References section so readers can trace any answer back to authoritative sources.
- [x] appendix-f-sizing-rules.md — STATUS: findings-recorded, two corrections made, References section added
  - Corrections applied:
    - **EC-X 4+1 minimum cluster size**: "requires 5+ nodes minimum" corrected to "stripe technically requires 5 (4+1) but Nutanix's recommended minimum is 6 nodes for rebuild headroom" (matches Module 05 fix per TN-2032).
    - **Prism Central sizing tiers**: rewrote the PC sizing table to reflect (a) the X-Small tier (5 clusters / 50 hosts / 500 VMs), which the curriculum was missing, and (b) the actual public-source vCPU / RAM / disk numbers (X-Small 4 vCPU / 18 GB / 100 GB; Small 6 vCPU / 26 GB / 500 GB; 3-VM scale-out 6 vCPU / 28 GB per VM / 500 GB per VM). The curriculum's earlier numbers (4/16, 8/32, 8/32×3) didn't match the Nutanix community resource-requirements thread or the X-Small tech blog. Added an explicit "exceeding documented limits is unsupported" warning per the Prism Central upgrade-limitations doc.
  - Confirmed:
    - Cluster minimum / sweet-spot / max framing accurate.
    - CVM tax table (8/32, 12/48, 16/64) consistent with Module 02.
    - RF / EC capacity overhead math accurate.
    - Compression ranges (1.5-2.5×) and dedup ranges (workload-specific) accurate.
    - Link-speed recommendations (10/25/100 GbE) accurate.
    - Bond mode selection guidance accurate.
    - Replication bandwidth estimation methodology accurate.
    - VDI density rules (50-200 sessions/node by profile) reasonable rules-of-thumb.
    - Database sizing guidance reasonable.
    - FSVM count "minimum 3" is the typical default; small-cluster single-FSVM exception flagged in the References pointer back to Module 08.
    - Object Service VM minimum 3 confirmed.
  - Suspect (kept as-is):
    - "Per LUN: up to 64 TB practical" — Nutanix Volumes per-LUN limits evolve; couldn't quickly verify the 64 TB number against current docs. Curriculum already adds "verify against current Nutanix limits at design time" caveat. Acceptable.
- [x] appendix-g-cli-reference.md — STATUS: findings-recorded, four corrections made, References section added
  - Corrections applied:
    - **`ncli cluster get-domain-fault-tolerance-status` requires `type=` parameter** (`type=node` for node-level FT, `type=rackable_unit` for block-level). Curriculum had the command without the parameter; current AOS requires it. Fixed in three places (main listing + two diagnostic recipes). Added a note about the per-component output (STATIC_CONFIGURATION, ERASURE_CODE_STRIP_SIZE, METADATA, ZOOKEEPER, EXTENT_GROUPS, OPLOG).
    - **`ncli health-check run-all` replaced with `ncc health_checks run_all`** as the canonical syntax. The ncli surface used to expose a thin passthrough to NCC, but the canonical command is the `ncc` form covered in detail later in the appendix.
    - **Stargate URL recipe corrected**: `links http://0:2009/h/stargate` → `links http://127.0.0.1:2009`. The 2009 page is the vdisk_stats page and is locked down by iptables to the local CVM (not cross-subnet), which is why the `links` text browser is run from inside the CVM. Added the iptables-restriction context.
    - **Page name**: clarified the 2009 page is the **vdisk_stats** page with histogram-style per-vDisk metrics, matching the public Nutanix Bible AOS Administration chapter.
  - Confirmed:
    - SSH connection patterns for CVM (`nutanix` user) and AHV host (`root` user) accurate.
    - `allssh` and `hostssh` cluster-wide command wrappers confirmed.
    - `ncli` object types (cluster, host, storage-container, storage-pool, network, protection-domain, data-services-vip, vm) consistent with portal command reference.
    - `acli` VM, storage, and network command syntax accurate.
    - `acli` dotted-method syntax (`vm.list`, `storage_container.create`, `net.add_dhcp_pool`) confirmed.
    - NCC categories (hardware_checks, network_checks, cluster_checks, data_protection_checks, system_checks, metadata_checks) accurate.
    - `ovs-vsctl`, `ovs-ofctl`, `ovs-appctl` syntax accurate.
    - `manage_ovs` wrapper usage accurate (Module 06 already verified bond modes).
    - Move CLI command shape (move version / status / env list / plan list / plan start / plan cutover) reasonable; verified against Move product positioning in Module 03 review.
    - `cluster status` for service-state-per-CVM, `upgrade_status` for component versions accurate.
    - `ncc log_collector run_all` for support log bundles accurate.
  - Suspect (kept as-is):
    - Move log file path `/opt/xtract-vm/logs/` — Move's history includes the legacy "Xtract" branding; the path persists in some Move versions. Acceptable as-is.
- [x] appendix-h-competitive-matrix.md — STATUS: findings-recorded, one refinement made, References section added
  - Refinement applied:
    - **HyperFlex section**: replaced vague "Cisco announced end-of-development in 2024" with the authoritative dates from Cisco's official EOL announcement (last order Sep 11, 2024; last bug-fix maintenance Sep 11, 2025; last subscription renewal Feb 28, 2029) and named the Cisco-Nutanix partnership migration product **Cisco Compute Hyperconverged with Nutanix** running on qualifying UCS M6 hardware (consistent with Appendix B's correction).
  - Confirmed:
    - The competitive playbook is engagement methodology (positioning, discovery questions, kill questions, win/loss conditions, what-NOT-to-say). Thin technical surface; the technical claims it does make derive from Appendix B and the modules.
    - VMware (VxRail / vSAN ReadyNodes) positioning consistent with Appendix B and Module 09.
    - HPE positioning (SimpliVity, Alletra dHCI, GreenLake) accurate; OEM partnership (Nutanix on HPE DX) consistent with Module 02.
    - Dell APEX positioning consistent with Module 09 consumption-model framing.
    - Azure Stack HCI positioning consistent with Appendix B.
    - Scale Computing HC3, StorMagic positioning reasonable for SMB / edge segments.
    - Ceph / SDS-on-commodity positioning honest; matches the "operational complexity vs licensing savings" trade-off framing in Module 09.
    - Status quo / "do nothing" framing accurate.
  - References section added linking to Cisco's official HyperFlex EOL announcement, Futurum's industry analysis, and the Appendix B / Module 09 / Module 06 References sections that source the feature-and-pricing tables behind the positioning playbooks.
- [ ] appendix-i-reference-architectures.md
- [ ] appendix-j-poc-playbook.md
- [ ] appendix-k-cert-tracker.md

**Discipline:**
- One file per pass. Do not batch. Do not skim.
- Each pass produces a commit with the findings recorded in this todo.md and any required corrections to the source markdown.
- Web research for every version number, product name, port, credential, and blueprint percentage. Do not trust pre-2026 memory; the Nutanix product line moves fast.
- Findings get the same epistemic discipline as commits: cite the source URL, quote the relevant claim, propose the revision.
- A file is **confirmed** only when every technical claim has been checked or the residual unchecked claims are explicitly listed.

**Gate (whole phase):**
- [ ] Every file marked confirmed or revised.
- [ ] Source markdown updated where the curriculum was wrong, with a single commit per module/appendix capturing the changes plus rationale.
- [ ] A summary at the end of this phase listing the categories of corrections made (e.g., "AOS version drift in 4 modules", "default-cred changes in 2 modules", "blueprint percentages adjusted to current NCA blueprint").

**Commit style for this phase:** `review: <file> technical pass; <N> findings recorded; <M> corrections made` with a body listing the specific changes.

---

## Phase 1 — Scaffold: Astro + content collections + schemas

**Goal:** bare-bones Astro site builds and runs. Content collections defined with strict Zod schemas. No UI, no styling, no real pages. The skeleton.

**Tasks:**

- [ ] `bun init`, install Astro, MDX integration, Tailwind, TypeScript strict.
- [ ] `astro.config.mjs` with `base: '/nutanix'`, `site: 'https://nixfred.com'`, MDX integration, Tailwind integration.
- [ ] `tsconfig.json` with strict mode and path aliases (`@/lib`, `@/components`, `@/content`).
- [ ] `tailwind.config.ts` configured with the CSS custom properties from CLAUDE.md (color tokens, typography stack, fonts).
- [ ] Move the curriculum markdown into `/curriculum/nutanix/` (do not modify the files; just relocate).
- [ ] Write a `scripts/sync-curriculum.ts` that copies `/curriculum/<track>/*.md` into `src/content/tracks/<track>/` on `prebuild` and on `dev` start. Idempotent.
- [ ] Create `src/content/config.ts` with Zod schemas for: `track` (the `_track.yaml`), `module`, `appendix`. Use `.strict()` so unknown keys fail the build.
- [ ] Create `_track.yaml` for the Nutanix track with values from CLAUDE.md.
- [ ] Run `astro sync` and `bun run build`. Build must succeed. No real pages yet; just the collection sync.
- [ ] Write `scripts/verify-frontmatter.ts` that loads every collection entry and reports schema violations with line numbers. Wire into `prebuild`.
- [ ] Set up `.github/workflows/deploy.yml` with the build steps from CLAUDE.md. Do not enable Pages yet.

**Gate:**
- [ ] `bun run dev` starts without errors.
- [ ] `bun run build` succeeds with zero warnings.
- [ ] `verify-frontmatter` passes against all 10 modules and all available appendices.
- [ ] Adding an unknown key to a module's front matter fails the build with a clear error message (test this manually).

**Commit:** `feat: phase 1 scaffold; astro + content collections + strict zod schemas`

---

## Phase 2 — Parser layer: callouts, diagrams, questions

**Goal:** a tested parsing layer that extracts every structured element from module markdown into typed objects. No UI yet. This is the foundation everything renders against.

**Tasks:**

- [ ] Create `src/test/fixtures/` with one minimal sample of each: callout (5 types), diagram block, MCQ practice question, NCX-style practice question. Hand-authored, deliberately small.
- [ ] Install a lightweight test runner (`vitest` or Astro's built-in test setup).
- [ ] `src/lib/parseCallouts.ts`: detects `> [!FAMILIAR]` etc. blocks and returns `{ type, body }`. Tests against fixtures.
- [ ] `src/lib/parseDiagram.ts`: extracts diagram blocks (id, type, caption, exam_relevance, whiteboard_ready, elements, connections, annotations, why-this-diagram). Returns typed object. Tests against fixtures.
- [ ] `src/lib/parseQuestions.ts`: extracts MCQ and NCX-style questions with their structured answer parts. Tests against fixtures.
- [ ] `src/lib/glossaryIndex.ts`: builds a `Map<term, { definition, slug }>` from the glossary appendix. Tests.
- [ ] Run all parsers across the real curriculum. Record the count of callouts, diagrams, questions, glossary terms in a build log. This is your baseline.
- [ ] Add cross-reference validation to `scripts/verify-links.ts`: every `diagrams:` entry in front matter must have a matching diagram block in body and vice versa; every `prerequisites:` slug must resolve; every `cert_coverage:` exam ID must be in the track's `cert_ladder`.

**Gate:**
- [ ] All parsers have tests; tests pass.
- [ ] Running parsers on the real curriculum produces zero errors and the expected element counts (10 modules × 12 questions = 120 MCQ + NCX entries; ~31 diagrams; 5 callout types present).
- [ ] `verify-links` runs clean across the real curriculum.

**Commit:** `feat: phase 2 parser layer; callouts, diagrams, questions, glossary index, link verification`

---

## Phase 3 — Core components: callout, diagram, question, glossary term

**Goal:** the renderable pieces of a module page exist as components, styled to spec, demo-able in isolation.

**Tasks:**

- [ ] `Callout.astro`: renders all 5 types with the translucent backgrounds from CLAUDE.md.
- [ ] `Diagram.tsx`: renders parsed diagram object. Use rough.js. Lazy-render on scroll (IntersectionObserver). Whiteboard-ready badge if flag set. Cert relevance badges. Accessible figcaption.
- [ ] `PracticeQuestion.tsx`: MCQ variant with reveal-answer, green/red feedback, expanded "why this answer / why not the others / the trap." NCX variant with text area + "show strong answer." LocalStorage with version key.
- [ ] `GlossaryTerm.tsx`: hover/tap popover. Debounced. Wraps first occurrence per page only (avoid noise).
- [ ] `Wordmark.astro`: NutaNIX with cyan NIX. No animation.
- [ ] `PathTreatment.astro`: `/nix/nutanix` mono treatment.
- [ ] Build a `/_dev/components` page (only in dev) that renders one of each component with sample data. This is your visual regression surface for the rest of the build.

**Gate:**
- [ ] Visit `/_dev/components` in dev. Every component renders correctly, matches spec colors, is keyboard-navigable.
- [ ] PracticeQuestion state survives page reload (localStorage works).
- [ ] Diagram does not render until scrolled into view (verify in devtools).
- [ ] GlossaryTerm popover does not appear on body text outside a `<dfn>` wrapper.

**Commit:** `feat: phase 3 core components; callout, diagram, question, glossary term, brand chrome`

---

## Phase 4 — Routing + nav builder + first end-to-end module page

**Goal:** one module page renders end-to-end at `/nutanix/<module>`. The sidebar is generated from `src/lib/nav.ts`. The Extensibility Test passes for the first time.

**Tasks:**

- [ ] `src/lib/nav.ts`: pure function from collections to nav tree. Sorts by `order`. Filters `status: archived`. Computes prev/next.
- [ ] `src/lib/links.ts`: resolves relative `.md` links by consulting collections.
- [ ] `src/pages/[track]/[module].astro`: dynamic route. Loads the module entry, renders MDX with custom components (Callout, Diagram, PracticeQuestion, GlossaryTerm). Pulls the sidebar from `nav.ts`.
- [ ] `src/components/Sidebar.astro`: renders the nav tree. Active-section highlighting on scroll. Collapsible groups.
- [ ] `src/components/PrevNext.astro`: derived from nav tree, rendered in module footer.
- [ ] `src/pages/[track]/index.astro`: track landing page. Lists modules from the nav tree, plus a hero with the wordmark and path treatment.
- [ ] `src/pages/index.astro`: homepage. Lists all tracks (currently just Nutanix) with cards from `_track.yaml`.

**Extensibility Test (run now):**
- [ ] Add `src/content/tracks/nutanix/99-zzz-test.md` with valid front matter (`order: 999`, `status: published`, body says "test"). Run `bun run dev`. Module appears in sidebar at the bottom, in prev/next on module 10, and is reachable at `/nutanix/zzz-test`. Delete the file, confirm it disappears. **If this fails, do not proceed.**

**Gate:**
- [ ] Module 5 (DSF) renders end-to-end with all callouts, all diagrams, all 12 questions.
- [ ] Sidebar is generated from `nav.ts`; no hardcoded module list anywhere.
- [ ] Cross-references in module bodies resolve to clean URLs (no `.md` links).
- [ ] Extensibility Test passes.
- [ ] Lighthouse on a single module page: Performance ≥ 85 (will tighten later), Accessibility ≥ 95.

**Commit:** `feat: phase 4 dynamic routes + nav builder + first e2e module page; extensibility verified`

---

## Phase 5 — All modules + appendices + cross-references

**Goal:** every module renders. Every appendix renders. Every cross-reference resolves. Site is content-complete.

**Tasks:**

- [ ] Verify modules 01–10 all render without warnings.
- [ ] `src/pages/[track]/appendices/[slug].astro`: dynamic route for appendices.
- [ ] Sidebar groups: "Modules" and "Appendices" within each track.
- [ ] Run `verify-links` across all rendered pages. Zero broken links.
- [ ] Glossary term wrapping in body text: build the matcher, apply only to first occurrence per page, wire into the MDX render pipeline.

**Gate:**
- [ ] All 10 modules + all 7 appendices render with zero console errors.
- [ ] Every cross-reference in the curriculum resolves.
- [ ] Glossary popover triggers on hover for at least 5 randomly-selected terms.
- [ ] Sidebar reflects the full module + appendix list, generated from nav.ts.

**Commit:** `feat: phase 5 full content render; all modules, appendices, glossary popovers, cross-refs resolved`

---

## Phase 6 — Search

**Goal:** Cmd-K search palette over all modules and appendices. Faceted by track. Index built at build time.

**Tasks:**

- [ ] `src/lib/search.ts`: walks collections, builds FlexSearch index per track plus a global index.
- [ ] `SearchPalette.tsx`: Cmd-K trigger, results grouped by section (modules / appendices), keyboard navigable, default-scoped to current track with a "search all tracks" toggle.
- [ ] Index ships as a JSON asset, loaded on first Cmd-K open (not on initial page load).
- [ ] Highlight matched terms in result snippets.

**Gate:**
- [ ] Cmd-K opens the palette; query "NearSync" returns results from Module 7.
- [ ] Closing the palette does not break scroll position.
- [ ] Mobile: a search button in the header opens the same palette.

**Commit:** `feat: phase 6 search; flexsearch index, cmd-k palette, track-scoped`

---

## Phase 7 — Practice mode + cert tracker

**Goal:** `/practice` aggregates all 120 MCQs with filters and a Random-20 quiz. `/certs` shows progress per exam.

**Tasks:**

- [ ] `src/pages/practice.astro`: aggregates questions across modules. Filters by cert, module, type, status. Random 20 generator. Time-per-question tracking. Final summary screen.
- [ ] `src/pages/certs.astro`: card per exam, derived from `_track.yaml.cert_ladder`. Coverage from front matter. User performance from localStorage.
- [ ] Both pages are derived from collections; new modules adding cert_coverage data flow in automatically.

**Extensibility Test (re-run):**
- [ ] Add a temporary 11th module with `cert_coverage: { NCA: 5 }`. Confirm `/practice` and `/certs` reflect it without code changes.

**Gate:**
- [ ] `/practice` filters work end-to-end. Random 20 produces a varied set across runs.
- [ ] `/certs` shows accurate coverage percentages.
- [ ] LocalStorage state for question attempts persists across page navigation.

**Commit:** `feat: phase 7 practice mode + cert tracker; both derived from collections`

---

## Phase 8 — Brand chrome polish + 404 + footer

**Goal:** the site looks finished. Wordmark, path treatment, 404, footer attribution all in place.

**Tasks:**

- [ ] Header: sticky, contains wordmark + track switcher + search button.
- [ ] Homepage hero: large path treatment (`/nix/nutanix`), subtitle, optional cursor blink.
- [ ] Footer on every page: "Written by Fred Nix · Built for BlueAlly Solutions Architects · Source on GitHub."
- [ ] `404.astro`: terminal-style miss with `cd: no such file or directory`, recovery links.
- [ ] Browser tab title format: `NutaNIX · Module 5: DSF Deep Dive`.
- [ ] Open Graph + Twitter card images for share previews.

**Gate:**
- [ ] Visit `/nutanix/does-not-exist`; the 404 renders correctly with the terminal aesthetic.
- [ ] Tab title updates correctly across navigation.
- [ ] No em-dashes, no emoji in chrome (grep the build output to verify).

**Commit:** `feat: phase 8 brand chrome; wordmark, path, 404, footer, og images`

---

## Phase 9 — Hero images

**Goal:** every module, every appendix, the homepage have on-brand hero images.

**Tasks:**

- [ ] `scripts/generate-heroes.ts`: takes `_track.yaml.hero_prompt` plus per-module topic, calls AI image API, caches result in `public/images/heroes/<track>/<slug>.webp`.
- [ ] Run for all 10 modules + 7 appendices + homepage.
- [ ] Manual review of every output. Regenerate on-brand misses.
- [ ] `HeroImage.astro`: lazy-loaded, AVIF with WebP fallback, max 120KB target.
- [ ] Document the prompt strategy in `scripts/generate-heroes.ts` so re-generation is reproducible.

**Gate:**
- [ ] Every page has a hero. Every hero is on-brand.
- [ ] Page weight per module ≤ 500KB excluding hero. Hero ≤ 150KB.

**Commit:** `feat: phase 9 hero images; generated, cached, on-brand for all pages`

---

## Phase 10 — Print/PDF stylesheet

**Goal:** "Print this module" produces a clean PDF.

**Tasks:**

- [ ] Print stylesheet: hide sidebar, header, search, hero. Expand all practice-question answers. Replace rough.js canvases with cached SVG snapshots or hide them with a "see online" note.
- [ ] "Print this module" link in module footer triggers `window.print()`.

**Gate:**
- [ ] Cmd-P on Module 5 produces a readable PDF with all content and no UI chrome.

**Commit:** `feat: phase 10 print stylesheet; modules export to clean pdf`

---

## Phase 11 — Performance + accessibility audit

**Goal:** Lighthouse Performance ≥ 95, Accessibility = 100. JS-disabled readability verified.

**Tasks:**

- [ ] Run Lighthouse on homepage, one module, `/practice`, `/certs`. Record scores.
- [ ] Fix any image-size, render-blocking, or unused-JS issues.
- [ ] Verify focus states on every interactive element. Real focus rings, not browser defaults.
- [ ] Verify keyboard navigation: tab through a full module page, including practice questions.
- [ ] Disable JS in devtools. Visit a module. Content must still be readable. Sidebar/search/practice are allowed to be non-functional.
- [ ] Add ARIA labels where needed (search palette, callouts, diagram figures).

**Gate:**
- [ ] Lighthouse Performance ≥ 95 on all four target pages.
- [ ] Lighthouse Accessibility = 100 on all four.
- [ ] Module content readable with JS disabled.

**Commit:** `chore: phase 11 perf + a11y audit; lighthouse 95/100, js-disabled readable`

---

## Phase 12 — Deploy

**Goal:** site live at `nixfred.com/nutanix`.

**Tasks:**

- [ ] Create the GitHub repo `nixfred/nutanix` (public). Confirm with Fred before pushing.
- [ ] Push the project to `main`.
- [ ] Enable GitHub Pages: Settings → Pages → Source: GitHub Actions.
- [ ] Push triggers deploy. Verify the action succeeds.
- [ ] Visit `https://nixfred.com/nutanix`. Click through every module. Hit Cmd-K. Submit a practice question.
- [ ] Test on mobile (real device, not just devtools emulation).
- [ ] Test on a flight scenario: load the site once, then put the browser offline. Cached pages should still render.

**Gate:**
- [ ] Site is live and functional at `nixfred.com/nutanix`.
- [ ] Mobile works.
- [ ] No console errors in production.

**Commit:** `chore: phase 12 deploy; live at nixfred.com/nutanix`

---

## Phase 13 — Validate the zero-touch property end-to-end

**Goal:** prove that the extensibility design holds in production, not just in dev.

**Tasks:**

- [ ] Add a throwaway second track: `src/content/tracks/_test/_track.yaml` plus one tiny module `_test/01-hello.md` with valid front matter.
- [ ] Run `bun run dev`. Confirm the new track shows on the homepage, has its own sidebar group, has its own search scope, has prev/next within itself.
- [ ] Build and deploy. Confirm same on production.
- [ ] Delete the throwaway track. Confirm it disappears cleanly.
- [ ] Document the "how to add a track" and "how to add a module" recipes in `README.md`.

**Gate:**
- [ ] The Extensibility Test passes against the live production site.
- [ ] README documents the recipes; a newcomer can add a module without reading any other file.

**Commit:** `chore: phase 13 extensibility validated end-to-end; recipes documented`

---

## Phase 14 (optional, Phase 2 of build) — Bookmarks + notes

**Goal:** per-paragraph bookmarks and notes in localStorage. CLAUDE.md flags this as optional. Skip if Phase 13 has shipped and Fred is satisfied.

**Tasks:**

- [ ] Bookmark icon per paragraph (visible on hover).
- [ ] `/bookmarks` page lists saved bookmarks.
- [ ] Notes input on each bookmark.
- [ ] Export-to-markdown for the bookmarks list.

**Gate:**
- [ ] Bookmarks survive page reload and navigation.

**Commit:** `feat: phase 14 bookmarks + notes (optional v2 feature)`

---

## Risk Register (kept honest as we go)

- **Schema drift over time.** Mitigated by Zod strict + CI verify-frontmatter. Re-check at Phase 13.
- **Glossary popover noise.** First-occurrence-only rule must hold. Re-verify at Phase 5.
- **rough.js print failure.** Addressed in Phase 10. Don't forget the SVG snapshot fallback.
- **Search index size growth across tracks.** Per-track indexes load on demand from Phase 6 forward. Re-measure at Phase 13 with the test track.
- **Hero image budget creep.** Hard 150KB ceiling per hero. Audit at Phase 9 and Phase 11.
- **Hardcoded "Nutanix" strings sneaking into chrome.** Track-aware components only. Grep at Phase 13.

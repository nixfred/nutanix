---
appendix: F
title: Sizing Rules
type: reference
purpose: |
  Back-of-envelope sizing rules for initial customer conversations. Use these
  during discovery, in whiteboard sessions, and for first-pass capacity planning.
  Final sizing always requires Nutanix Sizer with the customer's actual workload
  data; these rules get you within ~25% for early conversations.
usage: |
  Read the relevant section before a sizing conversation. Quote ranges, not point
  estimates. Always say "rough sizing" or "pre-sizer estimate" when sharing these
  numbers; commit to formal Sizer output for the proposal.
discipline: |
  Three principles:
  1. Quote ranges, not single numbers. Real workloads have variance.
  2. Always plan for failure tolerance. RF2 needs N+1; RF3 needs N+2.
  3. Reserve headroom. 70-80% utilization is the practical ceiling, not 100%.
last_updated: 2026-04-30
covers:
  - Cluster sizing fundamentals
  - Compute sizing (VM density, CVM tax, headroom)
  - Storage sizing (raw to usable, RF, EC, compression, dedup)
  - Network sizing (link speeds, bond modes, replication bandwidth)
  - Workload-specific rules (VDI, databases, containers, AI/ML)
  - Files, Objects, Volumes sizing
  - DR cluster sizing
  - Prism Central sizing
  - Replication bandwidth estimation
---

# Appendix F: Sizing Rules

These are rules of thumb for initial sizing conversations. They get you within roughly 25% for early planning. Final sizing requires Nutanix Sizer with actual workload data, and that's the number that goes into the proposal. When in doubt, run Sizer.

The discipline at a glance:

- **Quote ranges.** "8-12 nodes" is honest; "10 nodes" implies precision you don't have.
- **Plan for failure.** RF2 means you need at least N+1 nodes; RF3 needs N+2. Don't size for the happy path.
- **Reserve headroom.** Practical utilization ceiling is 70-80%. Above that, performance degrades and growth headroom evaporates.
- **Sizer is the source of truth.** These rules accelerate the conversation; they don't replace formal sizing.

---

## Cluster Sizing Fundamentals

### Minimum and Recommended Cluster Sizes

**Production minimum:** 3 nodes. RF2 is supported; rebuild after a node loss runs at degraded redundancy.

**Production recommended baseline:** 4-5 nodes. Provides N+1 redundancy with margin during rebuild.

**Production sweet spot:** 6-12 nodes for typical mid-market workloads. Performance scales linearly; rebuild times stay reasonable; failure tolerance is solid.

**Larger clusters:** 16-32 nodes for enterprise. Beyond ~32 nodes, evaluate splitting into multiple clusters for blast-radius management.

**ROBO / edge:** 1-node and 2-node clusters supported with specific licensing and feature limitations. Used for retail stores, branch offices, edge sites where datacenter-class redundancy isn't required.

**For final sizing:** Nutanix Sizer factors workload mix, growth, and failure-tolerance targets into the recommended cluster size.

---

### Failure-Tolerance Math

**RF2:** One node failure tolerated. Cluster must have at least N+1 nodes where N is the count needed to run all VMs at full performance. During rebuild, second failure causes data loss for affected vDisks.

**RF3:** Two simultaneous failures tolerated. Cluster needs N+2 minimum. Rebuild can survive one more failure during the window.

**EC-X 4+1:** One node failure tolerated. The stripe technically requires 5 nodes (4 data + 1 parity), but Nutanix's recommended minimum is **6 nodes** so the cluster has rebuild headroom after a node loss. EC can be enabled on 4-node clusters with smaller stripes, but the optimal 4+1 stripe needs 6+ nodes per TN-2032.

**EC-X 4+2:** Two failures tolerated; requires 7+ nodes minimum.

**Rule of thumb:** Don't run RF3 unless you genuinely need it. Most enterprise workloads run RF2 with proper backup. RF3 is for compliance-mandated, mission-critical, or workloads without external backup.

**For final sizing:** Sizer adjusts capacity calculations for the chosen RF.

---

## Compute Sizing

### VM Density per Node

**Typical mid-market mixed VMs (2 vCPU, 8 GB RAM average):**

- All-NVMe nodes with 32-48 cores, 768 GB-1 TB RAM: 50-100 VMs per node depending on density assumptions
- Account for the CVM tax (8-16 vCPU, 32-64 GB RAM consumed per node)
- Practical ceiling: target 70% CPU utilization at peak; leave 30% for spike absorption and rebalancing

**Heavier VMs (4-8 vCPU, 16-32 GB RAM):**

- 20-40 VMs per node typical
- More memory-bound than CPU-bound for typical mixed workloads

**VDI density:**

- Persistent: 80-150 desktops per node depending on user profile
- Non-persistent / pooled: 100-200 per node
- Driver: RAM, not CPU, for typical knowledge-worker desktops

**For final sizing:** Sizer takes per-VM specs as input and produces node count.

---

### CVM Tax Planning

Every Nutanix node runs a Controller VM. Reserve compute for it:

| Workload Profile | CVM vCPU | CVM RAM |
|---|---|---|
| Light (low IOPS, small cluster) | 8 | 32 GB |
| Standard (typical enterprise) | 12 | 48 GB |
| Heavy (high IOPS, larger cluster) | 16 | 64 GB |
| Very heavy (dense IOPS, large cluster) | 20+ | 96+ GB |

**Rule of thumb:** Reserve 10-15% of node compute for the CVM. Sizer is more precise.

**For final sizing:** CVM specs are part of platform recommendations, not a customer-tunable.

---

### Headroom Recommendations

| Resource | Practical Ceiling | Reasoning |
|---|---|---|
| CPU | 70% peak utilization | Spike absorption, rebalancing, growth |
| RAM | 75-85% | Memory ballooning is degraded performance |
| Storage | 75% | Capacity for snapshots, growth, rebalancing |
| Network bandwidth | 60-70% peak | Replication and rebuild traffic compete |

**Rule of thumb:** Size for 18-24 months of growth at the projected rate. Beyond that, plan for cluster expansion (add nodes).

---

## Storage Sizing

### Raw to Usable Conversion

The path from "how much disk are we buying" to "how much can the customer actually store":

| Step | Multiplier | Example (100 TB raw) |
|---|---|---|
| Raw drive capacity | 1.00 | 100 TB |
| Replication Factor 2 | 0.50 | 50 TB |
| Replication Factor 3 | 0.33 | 33 TB |
| EC-X 4+1 (instead of RF2) | 0.80 | 80 TB |
| EC-X 4+2 (instead of RF3) | 0.67 | 67 TB |
| Compression (typical 1.5-2.0x effective) | 1.5-2.0× | 75-100 TB after RF2 |
| Dedup (workload-dependent, 1.0-1.5×) | 1.0-1.5× | additional savings on high-similarity data |
| Free-space reservation | 0.85 | reduce by 15% for headroom |

**Rule of thumb for mixed enterprise workloads:**
- RF2 + compression + 15% headroom → ~70-80% effective usable capacity vs raw before dedup
- RF2 + compression + dedup on VDI → ~110-150% effective vs raw (higher than 100% due to dedup)
- RF3 + compression → ~50-55% effective vs raw

**For final sizing:** Sizer factors customer-specific compression and dedup ratios from workload analysis.

---

### Compression Reduction Ratios

Quote ranges, not marketing peaks:

| Workload Type | Typical Compression Ratio |
|---|---|
| OS images (Windows, Linux) | 1.5-2.5× |
| General application data | 1.2-1.8× |
| Already-compressed (video, images, encrypted) | 1.0-1.1× |
| Database (varies) | 1.3-2.0× |
| Backup data | already-compressed by backup software, 1.0-1.2× |
| Logs | 2-4× |

**Rule of thumb for mixed enterprise:** plan on 1.5-2.0× effective compression. Anything above 2× is a bonus.

**Watch out for:** customer expectations primed by marketing 4-6× claims. Reset early; quote 1.5-2.5×.

**For final sizing:** Sizer uses workload-appropriate ratios; refine after POC.

---

### Deduplication Ratios

Workload-dependent. Be conservative:

| Workload Type | Typical Dedup Ratio |
|---|---|
| VDI persistent profiles | 2-3× |
| VDI non-persistent (similar OS images) | 3-5× |
| General-purpose VMs | 1.0-1.3× |
| Application data | 1.0-1.2× |
| Backup data | 1.0× (already deduped by backup software) |

**Rule of thumb:** Enable on-disk dedup for VDI; leave disabled for general workloads unless POC shows ratio above 1.3×. Dedup adds metadata overhead; not free.

**For final sizing:** Sizer evaluates dedup benefit for the specific workload mix.

---

## Network Sizing

### Link Speed Recommendations

| Cluster Profile | Recommended Link |
|---|---|
| Small ROBO (1-3 nodes, light workload) | 10 GbE |
| Mid-market production (4-8 nodes) | 25 GbE |
| Enterprise production (8+ nodes) | 25 GbE minimum, 100 GbE for high-density |
| All-NVMe / very-high-IOPS | 100 GbE |
| VDI at scale | 25 GbE acceptable, 100 GbE for >2,000 sessions |

**Rule of thumb:** 25 GbE is the sweet spot for new deployments. 10 GbE is sufficient for small clusters and many ROBO sites. 100 GbE matters when you're saturating 25 GbE on east-west traffic.

**Per-node count:** 2 NICs minimum for redundancy. 4 NICs typical for separating data, management, and replication traffic.

**For final sizing:** Validate against actual workload throughput requirements.

---

### Bond Mode Selection

| Bond Mode | Use Case | Switch Coordination |
|---|---|---|
| active-backup | Simplest; good for most ROBO and small production | None |
| balance-slb | Active-active without LACP; good throughput, no switch config | None |
| balance-tcp / LACP | Best throughput and load distribution | Required (LACP on switch side) |
| active-active LACP | Maximum performance, MC-LAG capable | Required, advanced switch features |

**Rule of thumb:** active-backup for ROBO and small clusters where simplicity matters. balance-slb for production where switch coordination is hard. LACP for production where the network team controls the switch and can configure it.

**For final sizing:** Discuss with customer's network team; mode choice depends on switch capabilities and ops preferences.

---

### Replication Bandwidth Estimation

For DR planning, estimate WAN bandwidth needed for replication:

**Async replication (1-hour RPO):**

- Sustained bandwidth ~= daily change rate / 86,400 seconds × 8 (for bits)
- Rule of thumb: typical mid-market environment changes 2-5% of total capacity per day
- Example: 50 TB cluster, 3% daily change = 1.5 TB/day = ~150 Mbps sustained
- Plus 50% headroom for catch-up after WAN outages

**NearSync (sub-15-min RPO):**

- Bandwidth must support continuous change rate plus burst handling
- Typically 2-3× the Async sustained number
- Plus low-latency requirement (<5ms RTT typical)

**Metro Availability:**

- Synchronous: must support write throughput in both directions
- Plan for full peak write bandwidth, not average

**For final sizing:** Use customer's actual change-rate data from existing replication or measurement during POC.

---

## Workload-Specific Sizing

### VDI

**Baseline density assumption:** 100 sessions per node for typical knowledge workers, all-NVMe configuration with 768+ GB RAM.

**By user profile:**

| Profile | Sessions per Node |
|---|---|
| Task worker (light apps, browser) | 120-180 |
| Knowledge worker (Office, browser, light specialty) | 80-120 |
| Power user (specialty apps, multiple tools) | 50-80 |
| Developer / heavy workload | 30-50 |

**Storage:** persistent profiles 30-80 GB per user before dedup; non-persistent share base image plus delta. Plan capacity for the full population, not the concurrent count, if persistent.

**Boot-storm planning:** all-NVMe handles boot storms gracefully; spinning-disk configurations don't. For VDI at scale, all-NVMe is non-negotiable.

**For final sizing:** Sizer has VDI-specific profiles; refine with POC against actual login storm.

---

### Databases

**SQL Server / Oracle (typical OLTP):**

- All-NVMe required for predictable performance
- Plan p99 read latency under 2-3 ms; validate during POC
- RF2 typical; RF3 for compliance-mandated tier-0
- Storage IOPS: 5,000-50,000 per database typical, validate against current array metrics

**Database scale rule of thumb:**

| DB Size | Cluster Size Suggestion |
|---|---|
| Up to 5 TB | Fits in any production cluster |
| 5-20 TB | 6+ nodes for I/O distribution |
| 20-50 TB | 8+ nodes; consider dedicated database cluster |
| 50+ TB | Dedicated database cluster; specialty design |

**For final sizing:** Database-specific Sizer profiles plus validation POC.

---

### Containers and Kubernetes

**Per-cluster baseline:**

- 3-5 worker nodes for small Kubernetes clusters
- Scales to 100+ worker nodes for large platforms
- Storage via Nutanix CSI driver (persistent volumes)

**Density:** containers vary widely; size by total CPU/RAM/storage demand from the workload, then add 20-30% for Kubernetes overhead.

**For final sizing:** Sizer with the specific Kubernetes platform (Rancher, OpenShift, vanilla K8s) and the workload profile.

---

### AI / ML Training

**GPU-equipped nodes:**

- AHV supports GPU passthrough
- Density: 1-8 GPUs per node typical
- Network bandwidth becomes critical for distributed training (100 GbE recommended)
- Storage: balance fast scratch (local NVMe) with capacity (DSF for datasets)

**Rule of thumb:** AI training is rarely a Nutanix-only workload at scale; customers often combine on-prem (steady-state datasets) with cloud GPU (burst training). Hybrid is the typical pattern.

**For final sizing:** Specialty Sizer configurations exist; engage Nutanix solution architects for large GPU deployments.

---

## Files Sizing

### FSVM Count and Capacity

**Minimum:** 3 FSVMs per File Server.

**Capacity rule of thumb:**

| File Server Size | FSVM Count | Notes |
|---|---|---|
| Up to 50 TB | 3 FSVMs | Default baseline |
| 50-200 TB | 3-5 FSVMs | Scale per workload |
| 200 TB+ | 5+ FSVMs, possibly multiple File Servers | Scale and isolation |

**Performance:** each FSVM handles a portion of the file load; more FSVMs = more concurrent throughput.

**Rule of thumb:** start with 3 FSVMs; scale up if performance metrics show saturation.

**For final sizing:** Sizer has Files-specific recommendations.

---

### Files Capacity Planning

**Reservations and overhead:**

- DSF storage container backing Files: same RF and compression rules as general DSF
- Plan 20% overhead for snapshots, file system metadata, growth headroom

**Use case differentiation:**

- User home directories: many small files; dedup may not help; size by capacity
- Application file shares: varies by application
- Backup repositories: large sequential writes; compression helps; consider Objects instead

**For final sizing:** Match sizing to workload; Files sizer profiles refine.

---

## Objects Sizing

### Object Service VM Footprint

**Minimum:** 3 Object Service VMs per Object Store for HA.

**Throughput scaling:** more Object Service VMs = more concurrent S3 throughput. Plan based on the application's expected concurrent connection count.

**Capacity:** Objects sits on DSF; capacity is bounded by the underlying cluster's available storage container space.

**Rule of thumb:**

- Backup target use case: capacity-driven sizing; throughput is moderate
- Active application data (logs, intermediate results): throughput-driven; more Object Service VMs
- Archive: capacity-driven; minimal Object Service VMs sufficient

**For final sizing:** Objects has its own Sizer profile.

---

## Volumes Sizing

### LUN and Volume Group Limits

**Per LUN:** up to 64 TB practical (verify against current Nutanix limits at design time; product capabilities evolve).

**Per Volume Group:** multiple LUNs grouped; limits are generous for typical use.

**Per cluster:** thousands of LUNs supported.

**Multi-pathing:** plan multiple iSCSI portal IPs (one per node typical) for path redundancy.

**For final sizing:** Volumes is bounded by cluster storage container capacity; size for the consumers' needs plus overhead.

---

## DR Cluster Sizing

### Primary-to-DR Sizing Relationship

**Common patterns:**

| DR Pattern | DR Cluster Size |
|---|---|
| Cold DR (rebuild from backup) | None / minimal |
| Warm DR (Async replication, restart on failover) | 50-100% of primary capacity |
| Hot DR (active-passive Metro) | 100% of primary capacity |
| Active-Active Metro | Both sites size for full load if other fails |

**NC2 cloud DR:** size for failover capacity (Tier-1 + Tier-2 workloads typically); use cloud's elastic provisioning to scale up only at failover time.

**Rule of thumb:** DR cluster typically sized for 60-80% of primary if running steady-state DR; 100% for active-active.

**For final sizing:** Sizer has DR-specific configurations factoring replication mode and failover capacity targets.

---

## Prism Central Sizing

### Single PC vs Scale-Out

**X-Small PC VM** (introduced for smaller environments):

- Suitable for up to **5 clusters, 50 hosts, 500 VMs**
- Lowest footprint option; appropriate for ROBO, edge, and small-environment central management
- Strict scale ceiling: exceeding the X-Small limits results in an unsupported configuration

**Single PC VM (Small / Standard):**

- Suitable for typical mid-market through enterprise deployments
- Single VM; simpler operations
- Single point of failure for management plane (clusters continue running if PC is down)

**Scale-out PC (3 VMs):**

- Required for the largest deployments and production-grade PC HA
- Distributed across hosts for HA
- Recommended for enterprise deployments at scale

**PC VM resource specs (verify against the current Prism Central sizing guide; numbers evolve):**

| PC Mode | vCPU | RAM | Storage |
|---|---|---|---|
| X-Small | 4 | 18 GB | 100 GB |
| Small | 6 | 26 GB | 500 GB |
| Large (3-VM scale-out) | 6 per VM (× 3) | 28 GB per VM (× 3) | 500 GB per VM |

**For final sizing:** verify against the current Nutanix Prism Central sizing guide at design time; the size tiers and their VM-count ceilings shift between PC versions, and exceeding the documented limits results in an unsupported configuration.

---

## Replication Bandwidth Estimation

### Quick Calculation Method

**Step 1:** Estimate daily change rate. Typical mid-market: 2-5% of total capacity per day.

**Step 2:** Convert to per-second sustained bandwidth.

```
Daily change (TB) × 8,000 (Gbits per TB) / 86,400 (seconds per day) = sustained Gbps
```

**Step 3:** Apply replication-mode multiplier:

| Mode | Multiplier on Sustained |
|---|---|
| Async (1+ hour RPO) | 1.0× plus 50% catchup headroom |
| NearSync | 2.0-3.0× for burst handling |
| Metro | Sized for peak write throughput, not average |

**Step 4:** Add WAN compression benefit if applicable. Native compression in Nutanix replication often provides 1.3-1.7× effective bandwidth multiplier.

**Worked example:**

- 50 TB cluster, 3% daily change = 1.5 TB/day
- Sustained: 1.5 × 8,000 / 86,400 = 0.139 Gbps = 139 Mbps
- Async with catchup headroom: 200 Mbps
- After native compression: ~130-150 Mbps effective

**Rule of thumb:** for mid-market, plan 200-500 Mbps WAN for typical Async DR; more for NearSync; full peak write rate for Metro.

**For final sizing:** measure customer's actual change rate before committing to WAN sizing.

---

## When to Use These Rules vs Sizer

**Use these rules:**

- Discovery and early proposal conversations
- Whiteboard sessions
- Rough TCO estimates for first-pass evaluation
- "Will this fit on 8 nodes or 12?" judgment calls

**Switch to formal Sizer when:**

- Customer wants a binding proposal
- Specific performance targets need validation
- Workload has unusual patterns (specialty databases, AI training, HPC-adjacent)
- Multi-cluster or multi-site designs need coordination
- DR sizing where bandwidth matters

**The Sizer workflow:**

1. Customer provides workload data: VM list, resource consumption, growth assumptions
2. Sizer produces node count, model recommendations, capacity calculations
3. Output is the basis for the proposal BoM
4. Sizer output gets validated during POC and refined before order

The rules in this appendix accelerate the conversation; Sizer produces the number that goes in the contract. Don't conflate the two.

---

## Common Sizing Mistakes to Avoid

1. **Sizing for the average, not the peak.** Workloads have spikes; size for peak with headroom.
2. **Ignoring the CVM tax.** Reserve compute for the CVM; it's not free.
3. **Forgetting failure-tolerance math.** RF2 needs N+1; RF3 needs N+2.
4. **Quoting marketing compression ratios.** 1.5-2.5× is the honest range for mixed workloads.
5. **Skipping the headroom reservation.** 100% utilization is theoretical; 70-80% is the practical ceiling.
6. **Underestimating replication bandwidth.** Async DR is rarely the smallest WAN bill.
7. **Missing growth headroom.** Size for 18-24 months; expand cluster after.
8. **Trusting customer's "typical" without validation.** Their typical is often a peak; their peak is often catastrophic. Measure during POC.
9. **Forgetting that Files, Objects, Volumes consume the same DSF capacity.** Size the cluster for total demand, not per-service.
10. **Skipping PC sizing.** Prism Central is part of the design, not an afterthought.

---

## References

The sizing rules in this appendix are heuristics for early conversations; the authoritative source is Nutanix Sizer for binding sizing, and the public sources below for tier-by-tier specifics:

- [Module 02 References — CVM resource sizing](../02-nutanix-architecture.md#references). Backs the CVM tax table.
- [Module 05 References — RF / EC / compression / dedup math](../05-dsf-storage-deep-dive.md#references). Backs the storage-sizing section. Note: TN-2032 confirms the EC 4+1 recommended minimum of 6 nodes (the curriculum's earlier 5-node claim was wrong and is now corrected).
- [Module 06 References — Bond modes, link-speed recommendations](../06-networking-flow.md#references). Backs the network-sizing section, including Nutanix's official caution on LACP coordination.
- [Module 07 References — Async / NearSync / Metro replication, change-rate planning](../07-data-protection.md#references). Backs the replication-bandwidth section.
- [Module 08 References — Files, Objects, Volumes sizing constraints](../08-unified-storage.md#references). Backs the FSVM count guidance (note the single-FSVM exception for small clusters per TN-2041).
- [Resource Requirements for Prism Central (Nutanix Community)](https://next.nutanix.com/intelligent-operations-26/resource-requirements-for-prism-central-38237). Authoritative current PC vCPU / RAM / disk per tier.
- [Introducing X-Small Prism Central (Nutanix Tech Center)](https://www.nutanix.com/tech-center/blog/introducing-x-small-prism-central-a-low-footprint-option-for-smaller-environments). X-Small PC ceiling: 5 clusters / 50 hosts / 500 VMs.
- [Limitations of Prism Central Deployment (Nutanix Portal v7.3)](https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_7_3:upg-pc-upgrade-limitations-c.html). Strict-ceiling-equals-unsupported framing.

---

## Cross-References

- **Modules:** Each section links back to the module where the topic is taught in depth.
- **Glossary:** [Appendix A](./appendix-a-glossary.md) defines the terms used.
- **Comparison Matrix:** [Appendix B](./appendix-b-comparison-matrix.md) helps with competitive sizing comparisons.
- **Reference Architectures:** [Appendix I](./appendix-i-reference-architectures.md) has fully-sized reference designs.
- **POC Playbook:** [Appendix J](./appendix-j-poc-playbook.md) has the validation steps that confirm sizing during proof-of-concept.
- **Nutanix Sizer:** the official tool for binding sizing. Always run Sizer for the proposal.

---
module: 03
title: AHV (The Hypervisor Question)
estimated_reading_time: 35 min
prerequisites:
  - Module 01 (HCI Foundations)
  - Module 02 (Nutanix Architecture)
  - ESXi and vCenter operational experience
  - Working CE cluster from Module 02
key_terms:
  - AHV (Acropolis Hypervisor)
  - KVM (Kernel-based Virtual Machine)
  - QEMU
  - libvirt
  - Acropolis (the service)
  - Live Migration
  - ADS (Acropolis Distributed Scheduling)
  - NGT (Nutanix Guest Tools)
  - Nutanix Move
  - Open vSwitch (OVS)
  - VirtIO drivers
diagrams:
  - ahv-vs-esxi-stack
  - acropolis-control-plane
  - mixed-hypervisor-cluster
cert_coverage:
  NCA: ~10%
  NCP-MCI: ~28%
  NCM-MCI: ~15%
sa_toolkit:
  related_objections: [obj-004, obj-005, obj-009, obj-021]
  related_discovery: [q-hyp-01, q-hyp-02, q-hyp-03, q-hyp-04, q-hyp-05]
---

# Module 3: AHV (The Hypervisor Question)

> **Cert coverage:** NCA (~10%) · NCP-MCI (~28%) · NCM-MCI (~15%)
> **SA toolkit:** Objections #4, #5, #9, #21 · Discovery Q-HYP-01 through Q-HYP-05

---

## The Promise

By the end of this module you will:

1. **Explain AHV to a VMware admin in 60 seconds without sounding defensive.** AHV is the most emotionally loaded comparison in the Nutanix conversation. The customer's entire career is built on ESXi. You need a frame for AHV that lands as honest, not as conversion.
2. **Know exactly what AHV does well, what it lacks, and the comparison to ESXi at a sentence level.** No marketing. Specific gaps named. Specific strengths quantified.
3. **Pass the AHV portions of NCP-MCI.** Roughly 28% of the NCP-MCI blueprint touches AHV directly: VM configuration, networking on AHV, HA, Live Migration, snapshots, and AHV-specific troubleshooting. Master this module and a quarter of NCP-MCI is in your pocket.
4. **Handle the four most common AHV objections** in customer conversations: "why would I switch hypervisors?", "what about FT?", "my backup vendor doesn't support AHV," and "my team has 10 years of ESXi expertise."
5. **Make the Broadcom math case** with specific dollar comparisons. By April 2026, this is the single strongest economic argument in your bag. You should be able to walk through it on a whiteboard.
6. **Position the mixed-hypervisor option correctly.** The most common deal pattern in 2026 is not "switch from ESXi to AHV." It is "run ESXi on Nutanix today, drift to AHV over time, or stay mixed forever." You should be able to recommend this without sounding like you're hedging.

This module is where the customer conversation gets real. The technology answer is the easy part. The harder part is the conversation about change, expertise, and money.

---

## Foundation: What You Already Know

You know ESXi cold. The VMkernel, the vSphere stack, vCenter as a separate piece of management infrastructure, the licensing tiers (Standard, Enterprise Plus, vSphere Foundation, vCloud Foundation post-Broadcom), VMware Tools, the snapshot mechanics that everyone has been burned by once.

You know what vCenter does. It is the brain. Without it, you have unmanaged ESXi hosts. With it, you have HA, DRS, vMotion, Storage vMotion, distributed switches, content libraries, vRealize integrations, the Update Manager, the whole thing. vCenter is software you have to install, patch, monitor, and license. If vCenter falls over, your infrastructure does not stop running, but your management plane does.

Hold that picture. Now switch hypervisors.

AHV reorganizes those vCenter responsibilities. There is no separate management product to install. The control plane lives inside the CVM as a service called Acropolis. HA, Live Migration, scheduling, and VM lifecycle are all handled by Acropolis without any external coordinator. The result is operationally simpler. It is also genuinely different from what you are used to.

That is the change you are about to walk a customer through. It is a real change. Pretending it is not is how SAs lose deals.

> [!FROM-THE-SA-CHAIR]
> The first time a VMware admin asks you "why would I switch hypervisors?", the worst answer is a feature comparison. The right answer starts with: *"You don't have to. You can run ESXi on Nutanix today, get all the HCI operational benefits, and never touch AHV. Most of our customers start there. The hypervisor decision is a separate decision from the platform decision, and you can make it later, on your terms."* That sentence buys you the rest of the meeting. Once the admin knows their ESXi expertise is not at risk, they listen to the AHV story differently.

---

## Core Content

### What AHV Actually Is

AHV is a Type-1 hypervisor based on KVM (Kernel-based Virtual Machine) with QEMU as the device emulator and libvirt for management primitives. It is bundled into every Nutanix cluster. It is free in the sense that there is no separate AHV license; the cost is part of the AOS subscription.

Read carefully: KVM is a Linux kernel module that turns Linux into a hypervisor. QEMU is the user-space process that emulates devices for guest VMs. libvirt is a management API. AHV is what Nutanix built on top of those open-source pieces: a hardened, opinionated, enterprise-managed hypervisor with the upstream complexity hidden behind Prism and Acropolis.

The lineage matters because customers will ask. The honest framing: *"AHV is to KVM what RHEL is to upstream Linux. Nutanix took the open-source kernel-mode hypervisor that Red Hat, IBM, AWS, and Google all use in some form, hardened it for enterprise HCI, integrated it with the rest of the platform, and made it manageable for people who don't want to think about KVM internals."*

> [!ON-THE-EXAM] **NCA · NCP-MCI**
> Both exams expect you to know that AHV is KVM-based. Trap distractor: "AHV is a fork of Xen" (false), or "AHV is a Nutanix-proprietary hypervisor with no open-source heritage" (false), or "AHV requires a separate Linux installation on each node" (false; AHV is a complete hypervisor stack, not a layer added to a separate Linux install). Anchor: AHV is built on KVM, QEMU, libvirt, and Open vSwitch, with Acropolis as the Nutanix-specific control plane.

### Acropolis: The Service That Replaces vCenter (For AHV)

This is the architectural insight that changes everything for a VMware admin.

In the VMware world, vCenter is a separate appliance (or VM) running an entire management stack: vCenter Server, the Web Client, the Platform Services Controller, the database. It is software you deploy, patch, license, and protect. If vCenter fails, your AHV-equivalent (vSphere) keeps running, but your management plane is gone. HA still works because ESXi hosts run their own HA agents. DRS does not work without vCenter. vMotion does not work without vCenter.

AHV does not have a vCenter. The control-plane responsibilities (VM lifecycle, HA, scheduling, Live Migration, host management) are handled by a service called **Acropolis**, which runs inside every CVM. One Acropolis instance is elected master at any time; the others are followers, ready to take over if the master CVM is lost. The election uses the Zeus / ZooKeeper consensus mechanism we covered in Module 2.

This means three things:

1. **There is no external management appliance to deploy.** AHV is functionally complete out of the box.
2. **HA, Live Migration, scheduling, and VM lifecycle have no external dependency.** They survive the loss of any single CVM (or even multiple) because Acropolis is replicated across CVMs.
3. **The management surface lives at Prism Element (per-cluster) and Prism Central (multi-cluster).** Module 4 goes deep on Prism. For now, treat Prism as the AHV equivalent of the vSphere Web Client, with Acropolis underneath as the equivalent of vCenter Server.

> [!FAMILIAR]
> If you understand vCenter's responsibilities, you understand Acropolis's responsibilities. The functions are the same: lifecycle, HA, scheduling, migration. The packaging is different: Acropolis is a distributed service inside the cluster's CVMs, not a separate appliance you deploy.

> [!DIFFERENT]
> No external management appliance is the durable architectural advantage. You do not maintain, patch, license, or protect a vCenter. You do not have a vCenter outage that disables vMotion. You do not have a vCenter that needs its own DR plan. This is real operational simplification, not marketing. Customers who internalize this also internalize most of why AHV exists.

---

### Diagram: AHV Stack vs ESXi Stack

**id:** `ahv-vs-esxi-stack`
**type:** comparison
**caption:** Same hardware, two hypervisor stacks. Note where the management plane lives in each.
**exam_relevance:** [NCA, NCP-MCI]
**whiteboard_ready:** true

**Elements (left side, "ESXi on Nutanix"):**
- Bottom: physical hardware (gray)
- Above: "ESXi (VMkernel)" (muted blue)
- Above ESXi: two boxes side by side, "CVM" (rust) and "User VMs" (light gray)
- Above the cluster (off-cluster): a separate box "vCenter Server Appliance (VCSA)" (gold/teal, drawn floating outside the cluster) labeled "External management; required for HA / DRS / vMotion"

**Elements (right side, "AHV"):**
- Bottom: identical physical hardware (gray)
- Above: "AHV (KVM + QEMU + libvirt + Open vSwitch)" (muted blue)
- Above AHV: two boxes side by side, "CVM (with Acropolis service inside)" (rust, with internal sub-label "Acropolis: VM lifecycle, HA, Live Migration, ADS"), and "User VMs" (light gray)
- Above the cluster, a single connector to "Prism Element" (gold) labeled "In-cluster management. No external appliance."

**Connections:**
- Left side: arrows from ESXi up to vCenter labeled "Out-of-cluster management dependency"
- Right side: arrows from AHV/CVM up to Prism Element labeled "In-cluster management; survives any single CVM loss"

**Annotations:**
- Center label between the two stacks: "Same workload, two operating models."
- Below ESXi side: "vCenter is software you deploy, patch, license, protect."
- Below AHV side: "Acropolis is a service that lives inside the cluster. No external appliance."
- A red dotted box around the vCenter on the left: "External dependency. If this fails: no DRS, no vMotion, no centralized management."

**Why this diagram exists:** The single most important architectural insight about AHV is that the management plane is not external. This diagram makes that visible. It is also a whiteboard diagram for customer meetings. Draw it; let them sit with it for a beat.

---

### The Cycle, Frame Two: AHV as the Hypervisor Without a Tax

In the ESXi world, you pay for the hypervisor (now subscription-only post-Broadcom) and you pay for vCenter (bundled into the same subscription, but it is its own deployment). On a 100-VM customer, vSphere Foundation pricing is now in the range of $350 to $500 per core per year, with a 16-core minimum per CPU. For a typical 4-node cluster with 64 cores total per node (256 cores cluster-wide), that lands at roughly $90,000 to $128,000 per year just in hypervisor licensing. Plus vCenter (bundled but not free in any meaningful sense). Plus VMware support add-ons.

AHV's hypervisor license is bundled into the AOS subscription. Functionally, the hypervisor is free. The customer pays for AOS, which they would pay for anyway if they buy Nutanix. The hypervisor cost line on the BOM goes to zero.

This is the Broadcom argument made concrete. Switching from ESXi to AHV on the same hardware, in 2026, can save a typical mid-market customer six figures per year. Not in efficiency. In line-item licensing.

> [!FROM-THE-SA-CHAIR]
> The Broadcom math is the strongest economic argument you have right now and you should be able to recite it from memory. Practice this on a whiteboard. *"At your current core count, you're paying roughly $X per year for vSphere licensing. On AHV, that line goes to zero. Over five years, that's $5X. Apply that against the cost of platform migration and team retraining; the math is usually clear within 18-24 months."* Always offer to run the actual math on the customer's specific environment. The conversation goes from emotional ("I don't want to switch hypervisors") to financial ("show me the math") quickly, and once it is financial, you have a path.

### The Cycle, Frame Three: AHV as KVM Hardened for HCI

For a customer with a Linux-savvy team, the technical lineage matters. AHV is not a science project. KVM is the kernel virtualization layer in Red Hat OpenStack, AWS Nitro (in part), Google Compute Engine (KVM-derived), Oracle Cloud, and many others. It is one of the two hypervisors (alongside ESXi and Hyper-V, three if you count Xen's diminishing presence) that powers production at hyperscale.

What Nutanix added: hardening (security profiles, restricted access, controlled package set), opinionation (one supported configuration per AOS version, no manual KVM tuning), enterprise management (Prism, Acropolis, NGT, NCC integration), and the integration with DSF that makes AHV's snapshot, replication, and DR experience genuinely better than running stock KVM with external storage.

The customer who asks "is AHV stable enough" usually has not made the connection that they are already trusting KVM in their cloud workloads. AHV is not a riskier KVM. It is a more managed KVM.

### The Cycle, Frame Four: AHV as One Less Thing to Manage

For an operations leader, the durable AHV pitch is not lower licensing or KVM lineage. It is: one less management product to deploy, patch, license, protect, and integrate.

vCenter is software your team manages. It has a database. It has its own upgrade cadence. It has a backup story. It has its own HA story (vCenter HA, which is its own thing). It has a Tomcat instance and a Postgres database and an LDAP integration and a certificate refresh cycle. None of this is hard, individually. All of it adds up to a thing your team owns.

AHV does not have any of that. The Acropolis service auto-starts on every CVM. The master election is automatic. There is no database to back up because state lives in Cassandra and Zeus. There is no certificate cycle to manage because Prism handles it. The customer does not get a free hypervisor; the customer gets a hypervisor that does not require a parallel management infrastructure.

For some customers this is the durable argument. For others, the licensing math is the durable argument. The good SA reads which one matters more in any given meeting.

---

### Live Migration (the vMotion Equivalent)

AHV's Live Migration is functionally equivalent to vMotion. It moves a running VM from one host to another with sub-second user-visible disruption. The mechanism is the same kind of memory-copy plus iterative dirty-page tracking that vMotion uses, with a final stun-and-switch when the dirty rate is low enough.

What it does:
- Live migration of running VMs between hosts in the same cluster.
- No interruption visible to the guest OS or the application (for typical workloads).
- Storage stays in DSF; only memory and execution state move.

What it does not do:
- Cross-cluster live migration. (For this, you use Nutanix Move, which is async, not live.)
- Cross-hypervisor live migration. (You cannot live-migrate from ESXi to AHV; you migrate VMs cold or use Move.)

> [!ON-THE-EXAM] **NCP-MCI**
> Live Migration is a high-frequency NCP-MCI topic. Common questions: which workloads are eligible (most), what triggers an automatic live migration (ADS rebalancing, host maintenance mode, planned upgrades), what happens during the migration. Trap pattern: questions that imply data is migrated during Live Migration. **Memorize:** AHV Live Migration moves memory and execution state, not storage. Storage stays in DSF and is accessible from any node. Data locality is a separate, eventually-consistent optimization.

### HA in AHV (No vCenter Required)

When a node fails in an AHV cluster, here is what happens:

1. The remaining CVMs detect the loss via cluster heartbeat (Zeus / Cassandra mechanisms).
2. Acropolis (the master, on a surviving CVM) inventories the VMs that were on the failed node.
3. Acropolis selects target hosts for those VMs based on resource availability and any affinity rules in effect.
4. The VMs are restarted on surviving hosts.

End to end this is on the order of 30-90 seconds, similar to vSphere HA. The key difference: there is no vCenter dependency. If you lost vCenter at the same time as a host in the VMware world, vSphere HA still restarts the VMs (because it runs locally on each host) but you have no centralized recovery picture and you cannot invoke DRS to load-balance the recovered VMs. AHV does not have this dependency because the equivalent of vCenter (Acropolis) lives in the surviving cluster.

> [!DIFFERENT]
> There is no FT (Fault Tolerance / lockstep CPU) equivalent in AHV. ESXi's FT mirrors a VM's CPU state to a secondary host in real time, providing zero-downtime failover for the protected VM. AHV does not offer this. Customers who genuinely require FT (a small subset, typically real-time financial systems and some medical devices) cannot achieve it on AHV. The honest answer in a customer conversation: *"FT is one of a small number of features ESXi has that AHV does not. If you have workloads that genuinely require lockstep failover, those workloads stay on ESXi. Most workloads don't actually need FT; they need application-level HA, which AHV's HA handles equivalently to vSphere's."* Don't oversell. Name the gap.

> [!ON-THE-EXAM] **NCP-MCI · NCM-MCI**
> AHV HA is a high-weight exam topic. The mechanics (heartbeat detection, Acropolis-driven recovery, no external coordinator) are testable. So is the configuration: HA reservation policies, host failure tolerance settings, and the affinity rules that constrain VM placement during recovery. Memorize: AHV HA does not require vCenter (because there is no vCenter). It uses Acropolis, which runs inside every CVM. Quorum is maintained by Zeus.

### ADS: The DRS Equivalent

ADS (Acropolis Distributed Scheduling) is the AHV equivalent of vSphere DRS. It periodically rebalances VMs across hosts to address resource pressure.

ADS is genuinely simpler than DRS. DRS has decades of tuning for aggressive placement, pre-emptive load balancing, and granular policy options. ADS does the practical 80% of what DRS does with a fraction of the configuration surface. Specifically:

- ADS runs every 15 minutes by default and evaluates host CPU and memory pressure.
- When pressure exceeds thresholds, ADS recommends or executes Live Migrations to rebalance.
- VM-host affinity, VM-VM affinity, and anti-affinity rules are honored.
- ADS does not micromanage at the granularity DRS does; it acts on persistent pressure, not transient spikes.

For most customers this is fine. For customers with extreme density requirements or workloads that benefit from aggressive DRS-style optimization, ADS may feel less responsive. This is the honest gap and you name it if asked.

> [!FAMILIAR]
> If you understand DRS, you understand ADS. The intent is the same. The implementation is simpler.

> [!DIFFERENT]
> ADS is less aggressive and less configurable than DRS. For most customers this is operationally a benefit (less micromanagement). For density-obsessed customers it can feel like a regression. Know your customer.

---

### Diagram: Acropolis Control Plane

**id:** `acropolis-control-plane`
**type:** architecture
**caption:** Where Acropolis lives, who is master, what it controls, and how it survives a CVM loss.
**exam_relevance:** [NCP-MCI, NCM-MCI]
**whiteboard_ready:** false

**Elements:**
- Four CVM boxes laid out left to right (rust/orange).
- Inside each CVM: standard service set (Stargate, Cassandra, Curator, Zeus, Pithos), and an additional box labeled "Acropolis"
- One CVM (e.g., the leftmost) has its Acropolis box outlined heavily and labeled "Master"
- The other three Acropolis boxes are outlined lightly and labeled "Follower"
- Above the CVMs: a horizontal bar labeled "Acropolis distributed control plane (master + followers)"
- To the side: VM management actions (Create VM, Power On, Live Migrate, HA Restart) shown flowing into the master Acropolis

**Connections:**
- Master Acropolis to each AHV host: "Issues VM lifecycle commands"
- Followers to master: "Watch state; ready to take over"
- Zeus consensus shown linking all four CVMs: "Master election runs here"

**Annotations:**
- "If the master CVM is lost, a follower is elected master in <30 seconds. Cluster operations resume."
- "There is no separate management appliance. Acropolis is in the cluster."
- "VM lifecycle, HA, ADS, and Live Migration all run through the Acropolis master."

**Why this diagram exists:** To make concrete the mechanism by which AHV does not need a vCenter. Customers who have been burned by vCenter outages internalize this diagram quickly. Use it when the customer specifically asks "what happens if the management plane fails?"

---

### Memory Management: KSM, Overcommit, Ballooning

AHV uses **KSM** (Kernel Same-page Merging), a Linux kernel feature that finds identical memory pages across VMs and deduplicates them. Functionally, KSM is similar to ESXi's Transparent Page Sharing (TPS), with two caveats:

1. KSM operates at the kernel level via a periodic scanner (`ksmd`), which is configurable on AHV but not exposed for tuning to typical operators.
2. ESXi disabled inter-VM TPS by default years ago for security reasons (page-sharing side channels). KSM's security posture is similar: deduplication is opportunistic, and aggressive cross-tenant deduplication is not the default.

Memory overcommit is supported. Ballooning works similarly to ESXi's balloon driver. NGT installs the balloon driver on Windows and Linux guests. As with ESXi, the practical advice is: don't rely on aggressive memory overcommit for production workloads; size for the actual working set.

> [!ON-THE-EXAM] **NCP-MCI**
> Memory features (KSM, ballooning, overcommit) are testable on NCP-MCI. The trap distractor pattern: questions that conflate KSM with TPS without distinguishing them, or that imply AHV cannot overcommit. Both can. KSM is the AHV mechanism for page sharing. Ballooning is the AHV (and ESXi) mechanism for memory reclamation under pressure.

### Snapshots: The Genuine AHV Win

Here is where AHV (in fact, AOS, since it works on ESXi-on-Nutanix too) has a real advantage: snapshots are native to DSF, not the hypervisor.

In the ESXi world, a VM snapshot creates a delta vmdk. The hypervisor redirects writes to the delta. Read performance can degrade as the snapshot tree grows. Removing a snapshot triggers a consolidation operation that can be slow and can actually pause I/O for noticeable periods on large VMs. Anyone who has accidentally run with a months-old snapshot on a database VM has felt this pain.

In Nutanix, snapshots are a DSF operation. They are:

- **Instant.** Snapshot creation takes milliseconds, regardless of VM size.
- **Space-efficient.** DSF uses redirect-on-write with metadata pointers; the snapshot consumes only the changed blocks.
- **No I/O penalty.** Reads and writes to the active VM continue at native speed; the snapshot is a metadata reference, not a delta file in the VM's I/O path.
- **Consolidation-free.** Removing a snapshot is a metadata operation. No multi-hour consolidation that pauses your VM.

This is one of the cleanest advantages Nutanix has over running on a SAN. It applies to both AHV and ESXi-on-Nutanix, but it is most visible in customer demos when an AHV VM has hundreds of snapshots and the cluster is not breathing hard.

> [!FROM-THE-SA-CHAIR]
> When a customer has been burned by ESXi snapshot consolidation outages (and most customers with database admins have been burned at least once), the AHV/Nutanix snapshot story is genuinely emotional. Lead with: *"Snapshots in Nutanix are a DSF operation, not a hypervisor operation. They are instant, they don't degrade VM performance, and there is no consolidation step. If your DBA has ever paged you because a snapshot delete pegged the disk, you know why this matters."* That sentence is gold. It pulls a real, felt operational pain into the conversation. Your customer's senior infrastructure people will hear it.

### Console Access and NGT

Console access on AHV is browser-based, served through Prism. Behind the scenes it is VNC. There is no equivalent of the VMware Remote Console (VMRC) standalone app. For most administrative tasks, the in-Prism console is fine. For tasks that need a desktop-class console (heavy console-driven installations, certain legacy workflows), you can SSH to the hypervisor and use VNC clients directly, but in practice this is rare.

**NGT (Nutanix Guest Tools)** is the analog of VMware Tools. It provides:
- Time synchronization between host and guest
- IP address detection and reporting (so Prism shows guest IPs)
- VirtIO drivers for paravirtualized I/O (storage, network)
- VSS-aware snapshots for Windows (application-consistent snapshots)
- File-Level Recovery (FLR) for guest file recovery from a Nutanix snapshot
- Self-service file restore utilities

NGT is required for VSS-consistent snapshots on Windows guests, for accurate IP reporting in Prism, and for FLR. Install it on every production VM as a matter of course, the same way you install VMware Tools on every ESXi-resident VM.

> [!ON-THE-EXAM] **NCA · NCP-MCI**
> NGT installation, upgrade, and capabilities are tested. Trap distractor: "NGT is required for AHV operation" (false; AHV does not require NGT, but VSS-aware snapshots, FLR, and accurate IP reporting in Prism do). Memorize the NGT capability list above.

### What AHV Genuinely Lacks (The Honest Gap List)

Read this twice. As a BlueAlly SA, naming the gap honestly is what wins customer trust. Hand-waving the gap is what loses deals.

1. **FT (Fault Tolerance, lockstep CPU mirroring).** No equivalent in AHV. If a customer has a workload that requires lockstep failover, that workload stays on ESXi.
2. **Mature third-party ISV ecosystem.** ESXi has 20+ years of ISV support. AHV's ecosystem is meaningful and growing (Veeam, Commvault, Rubrik, HYCU, Cohesity, others all support AHV; major monitoring tools support AHV via APIs), but ESXi is broader. Specific niche ISVs may not yet certify AHV. Always check.
3. **Some advanced networking features that NSX-T provides on ESXi.** Distributed firewall sophistication, advanced routing patterns, gateway services. Module 6 covers this in detail and discusses where Flow Network Security closes the gap and where it does not.
4. **DRS-level scheduling sophistication.** ADS does the practical work; it does not match DRS's tuning surface.
5. **vSphere with Tanzu / native Kubernetes integration in the way VMware ships it.** AHV has NKE (Nutanix Kubernetes Engine), which is genuinely good, but it is a different product with a different operational model.
6. **Hot-add CPU and memory limits.** AHV supports hot-add but with tighter limits than ESXi in some scenarios. Check current AOS version specifics.
7. **Console UX parity with VMRC.** Prism's in-browser VNC console is functional but not as feature-rich as the VMware Remote Console for certain heavy desktop-driven scenarios.

That is the honest list. None of these are deal-breakers for the typical mid-market or enterprise general-purpose virtualization customer. All of them are real and you should not pretend otherwise.

### What AHV Has That ESXi Does Not (The Honest Strength List)

1. **No separate hypervisor licensing line item.** Bundled with AOS.
2. **No vCenter dependency.** Acropolis lives in the cluster.
3. **DSF-native snapshots.** Instant, space-efficient, no I/O penalty, no consolidation.
4. **One-click rolling upgrades via LCM.** AOS, AHV, BIOS, BMC, NIC firmware, drive firmware in one orchestrated workflow. ESXi's Update Manager / Lifecycle Manager is good but not in the same league for cross-component coordination.
5. **Tight integration with Nutanix DR (Leap, Metro, NearSync).** Module 7 goes deep.
6. **Open Virtual Switch (OVS) underneath networking.** Module 6 covers this; for SDN-friendly customers, OVS is a meaningful technical asset.
7. **Single API surface (Nutanix v4).** No need to glue together vSphere API + array API + network API. Automation is genuinely simpler.

### The Mixed-Hypervisor Reality (Read This Carefully)

This is the section that wins more BlueAlly deals than any feature comparison.

The customer does not have to choose AHV or ESXi. They can run both. Specifically:

- **ESXi on Nutanix** is a fully supported, common deployment pattern. The customer keeps vCenter, keeps ESXi, keeps every piece of their VMware tooling, and gets the HCI platform benefits underneath: distributed storage, scale-out, one-click upgrades, integrated DR, the Prism management experience. They still pay VMware for vSphere licensing.

- **AHV on Nutanix** is the alternative. Same hardware, different hypervisor, no VMware bill.

- **Mixed clusters within a single Prism Central** are supported. You can have an ESXi cluster and an AHV cluster, both Nutanix, both managed from the same Prism Central. They are separate clusters at the cluster level (a single cluster runs one hypervisor), but the management experience is unified.

- **Migration from ESXi to AHV** is a controlled process using Nutanix Move (free tool). Module 10 walks the migration end to end. The headline: VMs go cold for the cutover but the rest is automated.

The most common 2026 deal pattern at BlueAlly is not "switch from ESXi to AHV." It is:

1. Deploy Nutanix with ESXi on it. Customer keeps everything they know.
2. Get HCI benefits immediately (snapshots, DR, scale-out, lifecycle management).
3. Over 12-36 months, drift workloads to AHV at the customer's pace, starting with low-risk workloads (dev/test, monitoring, file/print, internal apps).
4. Maybe converge fully to AHV. Maybe stay mixed forever. Either is fine.

> [!FROM-THE-SA-CHAIR]
> The mixed-hypervisor option is your strongest opening positioning for any VMware shop right now. *"You don't have to switch hypervisors to start. Run ESXi on Nutanix, get the HCI benefits today, and decide on AHV later, on your terms. Most customers stay on ESXi for the first 12-18 months. After that, the licensing math usually decides for them."* Memorize that. It defuses the hypervisor objection in three sentences.

---

### Diagram: Mixed-Hypervisor Cluster Topology

**id:** `mixed-hypervisor-cluster`
**type:** architecture
**caption:** A single Prism Central managing both an ESXi-on-Nutanix cluster and an AHV cluster. This is the most common 2026 deal pattern.
**exam_relevance:** [NCP-MCI]
**whiteboard_ready:** true

**Elements:**
- Top center: Prism Central instance (gold) labeled "Single management plane"
- Below Prism Central, two cluster blocks side by side:
  - **Cluster A (ESXi):** four nodes, each with ESXi (muted blue) + CVM (rust) + user VMs. Above the cluster: "vCenter Server" (gray, drawn as external dependency, with red dotted outline)
  - **Cluster B (AHV):** four nodes, each with AHV (muted blue) + CVM (rust, with Acropolis sub-label) + user VMs. No external management dependency.
- A "Nutanix Move" arrow between the two clusters (orange) labeled "VM-by-VM migration path; no time pressure"

**Connections:**
- Prism Central down to both clusters
- Each cluster's CVMs to their hypervisor
- ESXi cluster up to vCenter (with the red-dashed external-dependency callout)
- Nutanix Move arrow shown as bidirectional but biased rightward (toward AHV) over time

**Annotations:**
- "Both clusters are Nutanix. Both get HCI benefits. One pays VMware; one does not."
- "Migration is workload-by-workload, on the customer's timeline."
- "vCenter still required for the ESXi cluster. AHV cluster has no equivalent dependency."

**Why this diagram exists:** Customers want to know they can preserve their VMware investment while exploring AHV. This diagram makes the hybrid path visual. Use it when the conversation moves from "should we" to "how would we." Whiteboard it.

---

### Nutanix Move: The Migration Tool

**Move** is Nutanix's free VM migration tool. It is delivered as a small VM you deploy on your Nutanix cluster. It supports migration from:
- VMware ESXi (most common)
- Microsoft Hyper-V
- AWS EC2
- Microsoft Azure
- Google Cloud Platform
- Other Nutanix clusters (rare; Live Migration is preferred within a cluster)

The Move workflow:

1. Deploy Move (download an OVA / qcow2; run it on your Nutanix cluster).
2. Add source environment (e.g., point Move at your vCenter and provide credentials).
3. Add target (your Nutanix cluster's Prism Element or Central).
4. Create a migration plan: select VMs, set network mappings, set scheduling.
5. Run the plan. Move performs the bulk of the data copy *while the VM continues running on the source* (using CBT-equivalent change tracking).
6. At the cutover, the VM is briefly powered off, the final delta is copied, drivers are swapped (VMware Tools out, NGT in), and the VM powers up on AHV.

Total downtime per VM is typically 5-15 minutes for the cutover. Bulk data copy happens in the background and can take hours per terabyte depending on network and source array performance.

> [!ON-THE-EXAM] **NCP-MCI**
> Nutanix Move shows up on NCP-MCI. Memorize: Move is free, deployed as a VM on the target Nutanix cluster, supports ESXi/Hyper-V/AWS/Azure/GCP as sources, and performs change-tracked bulk copy plus a brief cutover. Trap distractor: "Move requires a separate licensing fee" (false), or "Move does live migration cross-cluster" (false; the cutover is brief but not live in the vMotion sense).

---

## Lab Exercise: Live Migration and a VM on AHV

> [!LAB] **Time:** ~90 min · **Platform:** Your 3-node CE cluster from Module 02

This lab puts AHV concepts into your hands. You deploy a VM, observe its placement, live-migrate it, snapshot it, and explore the Acropolis control plane.

**Steps:**

1. **Log into Prism Element** at `https://<cluster-vip>:9440`.
2. **Navigate to VM > Create VM.** Provision a small Linux VM:
   - Name: `lab-vm-01`
   - vCPU: 2, RAM: 4 GB
   - Disk: 20 GB (new vDisk on the default storage container)
   - NIC: default network
   - Boot from a Linux ISO you upload to the cluster (Ubuntu Server or similar)
3. Power on, install the OS, then power off. Take note of which physical host the VM landed on (Prism shows this in the VM detail pane).
4. **Power the VM back on.** Note the host placement again. ADS may have moved it; or it may stay.
5. **Live Migrate the VM.** Right-click the VM in Prism, choose "Migrate," select a different host. Watch the migration. Note the duration. Confirm the guest OS is unaffected.
6. **Take a snapshot.** Right-click VM > "Take Snapshot." Note that this is instant. Take three more in quick succession to build a snapshot tree.
7. **Restore the VM from a snapshot.** Use the snapshot management UI to revert to an earlier snapshot. Note that this is also fast.
8. **Open an SSH session to the cluster** (`ssh nutanix@<any-cvm-ip>`). Run `acli vm.list` to see all VMs from the CLI. Run `acli vm.get lab-vm-01` to see detailed VM properties.
9. **Optional: simulate a host failure.** If you have a non-production cluster, you can power off the host running your VM. Watch HA restart the VM on a surviving host. (Do not do this on a production cluster.)
10. **Check Acropolis master.** Run `acli` interactive mode and look at cluster state, or via SSH: `links http://0:2030` to see the Acropolis master's status page (Acropolis runs an internal status page on port 2030 of each CVM).

**What this teaches you:**
- The VM creation experience in Prism (compare it to vCenter's New-VM wizard mentally).
- Live Migration in practice: how it looks, how long it takes, what is and is not interrupted.
- Snapshot speed (instant) and behavior (no I/O penalty).
- The CLI surface (`acli`) for AHV-specific operations.
- The Acropolis master's role and how to peek at it.

**Customer-demo angle:** This lab is also the foundation for a customer POC. When you walk a customer through Nutanix, this is roughly the demo flow: provision a VM, migrate it, snapshot it, recover it. It takes 20 minutes and covers the AHV core experience.

---

## Practice Questions

Twelve questions. Six knowledge MCQ (NCA / NCP-MCI), four scenario MCQ (NCP-MCI / NCM-MCI), two NCX-style design questions. Read each, answer in your head, then read the explanation.

---

**Q1.** What is AHV?
**Cert relevance:** NCA · NCP-MCI

A) A proprietary Nutanix hypervisor with no open-source heritage
B) A Type-1 hypervisor based on KVM, QEMU, libvirt, and Open vSwitch, with Acropolis as the Nutanix-specific control plane
C) A Type-2 hypervisor that runs on top of Linux
D) A modified version of Xen developed by Nutanix

**Answer:** B

**Why this answer:** AHV is built on KVM (a Type-1 hypervisor at the kernel level), uses QEMU for device emulation, libvirt for management primitives, Open vSwitch for networking, and adds Acropolis as the distributed control plane. This is the canonical description.

**Why not the others:**
- A) AHV has clear open-source heritage. KVM, QEMU, libvirt, OVS are all open-source.
- C) KVM (and therefore AHV) is Type-1: the hypervisor is the kernel itself, not an application running on top of an OS.
- D) Xen has nothing to do with AHV. This is a common mis-conflation among VMware-focused engineers.

**The trap:** D is the seductive distractor for someone who knows VMware history. Xen was an early enterprise hypervisor (Citrix XenServer, Amazon's early EC2). AHV has no Xen heritage; it is KVM-based.

---

**Q2.** Which Nutanix service is responsible for VM lifecycle, HA, and Live Migration on an AHV cluster?
**Cert relevance:** NCP-MCI

A) Stargate
B) Acropolis
C) Cassandra
D) Curator

**Answer:** B

**Why this answer:** Acropolis is the distributed control plane that handles VM lifecycle, HA, ADS scheduling, and Live Migration. It runs as a service inside every CVM. One Acropolis is master; the others are followers.

**Why not the others:**
- A) Stargate is the data path. It handles I/O, not VM lifecycle.
- C) Cassandra is the metadata store. It tracks where data lives, not what VMs are doing.
- D) Curator is the background scrubber. It rebalances data, not VMs.

**The trap:** This question rewards memorizing the CVM service responsibilities from Module 2 plus knowing that Acropolis is the AHV-specific addition. If you confused "Acropolis" with "AOS" (Acropolis Operating System, the platform name), you might over-think this. Acropolis here means the service, not the platform.

---

**Q3.** True or false: AHV requires an external management appliance similar to vCenter for HA, Live Migration, and VM lifecycle management to function.
**Cert relevance:** NCA · NCP-MCI

True / False

**Answer:** False

**Why this answer:** AHV does not require any external management appliance. The control plane (Acropolis) lives inside the CVMs as a distributed service. HA, Live Migration, scheduling, and VM lifecycle all work without an external coordinator. Prism Element provides the user-facing management UI and runs in-cluster as well.

**The trap:** A VMware admin's intuition is "of course there's a vCenter equivalent." AHV's architecture deliberately eliminates the external management dependency. This is one of the key differentiators.

---

**Q4.** A VM running on AHV node-A is live-migrated to node-B. What happens to the VM's storage during the migration?
**Cert relevance:** NCP-MCI

A) Storage is migrated synchronously with the memory and execution state; the VM cannot resume on node-B until storage is fully transferred
B) Storage remains in DSF and is accessible from node-B without any data movement; only memory and execution state are transferred during Live Migration
C) Storage must be manually migrated using Storage vMotion equivalent
D) Live Migration copies storage to node-B and deletes the original

**Answer:** B

**Why this answer:** DSF is a distributed storage fabric. The VM's storage is already accessible from any node in the cluster. Live Migration moves only the memory pages and execution state, not storage. Data locality is a separate, eventually-consistent optimization that happens after migration.

**Why not the others:**
- A) Storage migration is not part of Live Migration. The architecture explicitly separates them.
- C) There is no manual storage migration step in normal AHV operations. DSF makes storage cluster-wide already.
- D) Storage is not deleted from the original node; data locality is opportunistic, not destructive.

**The trap:** A is the classic VMware-mental-model trap. In a non-HCI world, moving a VM might involve moving its storage. In AHV (and ESXi-on-Nutanix), DSF makes that unnecessary by design.

---

**Q5.** Which of the following is NOT a feature provided by NGT (Nutanix Guest Tools)?
**Cert relevance:** NCA · NCP-MCI

A) VirtIO drivers for paravirtualized I/O
B) VSS-aware snapshots for Windows guests
C) Real-time CPU lockstep between primary and secondary VMs (Fault Tolerance)
D) IP address detection and reporting to Prism

**Answer:** C

**Why this answer:** Fault Tolerance (lockstep CPU mirroring) does not exist in AHV. It is an ESXi feature with no AHV equivalent. NGT cannot provide it.

**Why not the others:**
- A) VirtIO drivers are part of NGT; they enable paravirtualized network and storage performance.
- B) NGT provides VSS-aware snapshots on Windows for application-consistent backups.
- D) NGT reports the guest's IP back to Prism for visibility.

**The trap:** This question tests two things at once: NGT capabilities and the FT gap. If you know NGT well but didn't notice that FT is the trap, you might second-guess the right answer.

---

**Q6.** Approximately how long does it take to create a snapshot of a 1 TB VM on Nutanix?
**Cert relevance:** NCA · NCP-MCI · sales-relevant

A) Several minutes, proportional to VM size
B) Several seconds, with brief I/O quiescing
C) Milliseconds, regardless of VM size
D) Snapshots are not supported on VMs larger than 500 GB

**Answer:** C

**Why this answer:** DSF snapshots are metadata operations using redirect-on-write semantics. They are instant regardless of VM size. There is no data-copy step at snapshot creation.

**Why not the others:**
- A) Snapshot time is not proportional to VM size in DSF.
- B) Brief I/O quiescing is true for application-consistent (VSS) snapshots, but the snapshot operation itself is still milliseconds. The quiesce is for application state, not the underlying snapshot.
- D) There is no size limit for snapshots.

**The trap:** A and B reflect ESXi mental models (snapshot creation taking time, quiescing being lengthy). DSF snapshots are categorically different and you should be able to demo this in seconds in front of a customer.

---

**Q7.** A customer is running 4 ESXi nodes (16 cores per CPU, 2 CPUs per node, so 128 cores cluster-wide). They are evaluating Nutanix and considering whether to stay on ESXi-on-Nutanix or switch to AHV. With current Broadcom-era subscription pricing, approximately what is the annual VMware licensing they could eliminate by moving to AHV?
**Cert relevance:** sales-relevant

A) About $5,000-$10,000 per year
B) About $20,000-$30,000 per year
C) About $50,000-$70,000 per year
D) Nothing; AHV requires equivalent licensing

**Answer:** C

**Why this answer:** With vSphere Foundation (or equivalent) subscription pricing typically in the $400-$550 per core per year range as of 2026 (varies by tier and discounting), 128 cores at, say, $450/core = $57,600 per year. This is rough but in the right order of magnitude. The answer requires both knowing per-core pricing and doing the math.

**Why not the others:**
- A) An order of magnitude too low. Per-core pricing makes any modern cluster's bill substantial.
- B) Closer but still low for a 128-core cluster.
- D) AHV is bundled with AOS subscription; no separate hypervisor licensing fee.

**The trap:** D is the trap for someone who hasn't internalized the AHV pricing model. The economic argument is real and large; pricing math is the strongest single driver of AHV adoption in 2026.

---

**Q8.** A customer's VMware admin says: "I have ten years of ESXi expertise. I'm not retraining my team for AHV." What is the strongest SA response?
**Cert relevance:** sales-relevant

A) "AHV is so similar to ESXi that retraining isn't really needed."
B) "You don't have to. Run ESXi on Nutanix, keep your team's expertise intact, and decide on AHV later, on your terms. Most customers stay on ESXi for the first 12-18 months. After that, the licensing math usually drives the conversation."
C) "Your team will adapt quickly, AHV is much simpler."
D) "If your team can't learn AHV, that's a problem you need to address."

**Answer:** B

**Why this answer:** This response respects the admin's expertise, removes the false binary (HCI requires hypervisor switch), names the real path forward (mixed deployment, time-based decision), and points to the durable economic driver (licensing math).

**Why not the others:**
- A) Untrue and patronizing. AHV and ESXi are similar in concept but operationally different. Pretending there is no learning curve insults the admin's experience.
- C) Even if true (debatable), this doesn't address the admin's concern. He is not asking whether his team can learn; he is signaling that he doesn't want to.
- D) Combative and tone-deaf. Costs you the meeting.

**The trap:** A and C are the natural defensive moves. D is the bad-day move. B is the durable answer that wins deals. Memorize it.

---

**Q9.** Which of the following workloads is least likely to be a good fit for migration from ESXi to AHV?
**Cert relevance:** NCP-MCI · sales-relevant

A) A general-purpose Windows file server
B) A SQL Server database (single instance, application-level resilience)
C) A real-time financial trading system requiring lockstep failover (FT)
D) A Linux VDI deployment

**Answer:** C

**Why this answer:** AHV does not support Fault Tolerance (lockstep CPU mirroring). Workloads that genuinely require zero-downtime failover at the hypervisor level are the rare cases that should stay on ESXi.

**Why not the others:**
- A) File servers are bread-and-butter AHV workloads.
- B) SQL Server with application-level HA (Always On, log shipping, etc.) does not require FT and runs well on AHV. NGT provides VSS-aware snapshots.
- D) VDI is one of the strongest AHV use cases (predictable boot storms benefit from data locality and instant cloning via DSF).

**The trap:** This question requires you to remember the specific AHV gap (no FT) and connect it to the kind of workload that actually requires FT (narrow: real-time finance, some medical, some industrial). The trap is treating "Tier-1 database" as automatically FT-required, which it is not.

---

**Q10.** A customer's environment has 50 VMs on ESXi. They want to migrate to AHV. What tool do you use, and what is the typical per-VM downtime during cutover?
**Cert relevance:** NCP-MCI

A) Nutanix Move; downtime is typically 5-15 minutes per VM at cutover
B) vMotion; downtime is zero
C) PowerCLI scripts; downtime is hours
D) Manual export/import; downtime is variable but always significant

**Answer:** A

**Why this answer:** Nutanix Move is the supported, free tool for ESXi-to-AHV migration. It performs change-tracked bulk copy in the background while the VM continues running, then a brief cutover (5-15 minutes typical) to swap drivers and restart on AHV.

**Why not the others:**
- B) vMotion is VMware's tool and only moves VMs between ESXi hosts. It cannot cross hypervisors.
- C) PowerCLI cannot perform cross-hypervisor migration at the VM data level. It can orchestrate, but the actual migration tool is Move.
- D) Possible in theory but not the supported path; Move automates this.

**The trap:** B is intuitive ("I know vMotion"); C feels VMware-admin-clever ("I'll script it"). The right answer is the named, supported, free tool: Move.

---

**Q11.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / design discussion. Write your reasoning. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A customer environment: 12 ESXi hosts, 600 VMs, mixed workloads (no FT requirements anywhere, no real-time requirements, several SQL Server databases with Always On clusters, ~150 VDI desktops, the rest general-purpose). VMware Foundation renewal in 6 months. Annual VMware licensing currently $180,000. Storage refresh on the existing FlashArray due in 9 months ($350,000 quoted refresh). Two infrastructure engineers who are skeptical of AHV but open-minded if shown the numbers. CTO has asked for a five-year cost comparison and a risk-assessment recommendation.

**The challenge:**
Walk through your recommendation. Cover the financial case, the technical migration plan, the risk assessment, and the human/political dimension.

**A strong answer covers:**
- **Five-year licensing math:** at $180k/year today, conservatively $200-250k/year over 5 years with Broadcom escalations, total VMware spend ~$1.0-1.25M over 5 years if they stay on ESXi. AHV eliminates this line entirely. Net savings approximately $1M over 5 years.
- **Storage refresh comparison:** the $350k FlashArray refresh becomes Nutanix node spend instead. Apples-to-apples comparison is not 1-to-1 (you're getting compute + storage in HCI nodes), but the story is that the storage refresh budget alone covers a substantial portion of the Nutanix purchase.
- **Migration plan recommendation:** Phase 1 (months 1-6): deploy Nutanix with ESXi, migrate VMs onto Nutanix-on-ESXi with no hypervisor change, use the FlashArray refresh budget to fund this. Phase 2 (months 6-18): migrate low-risk workloads to AHV using Move (general-purpose VMs first, then VDI, then file/print). Phase 3 (months 18-36): migrate SQL Server VMs to AHV after the team has confidence and tooling is proven. Stop migrating when the customer wants; mixed steady-state is fine.
- **Risk assessment:** the platform risk is low (Nutanix is mature, AHV is mature). The team risk is the real risk: the two skeptical engineers need to be brought along, not overridden. Recommend dedicated training budget and lab time for them. Ideally one of them becomes the AHV champion.
- **Workload-specific notes:** SQL Server with Always On is fully supported on AHV; the Always On replicas provide application-level HA so FT is not required. VDI on AHV is a strong use case (Citrix and Horizon both work; Horizon support has improved post-Broadcom).
- **Honest gaps to flag:** if the customer has any niche ISV products that explicitly require ESXi certification, those workloads stay on ESXi. Always check the ISV list before promising AHV migration.
- **Recommended structure of the proposal:** financial case, phased migration plan, training and team-development plan, risk register, workload-by-workload disposition matrix.

**A weak answer misses:**
- Recommending immediate full migration to AHV without acknowledging team skepticism.
- Skipping the phased approach (Phase 1: ESXi-on-Nutanix as the de-risking step).
- Not naming the storage refresh as a funding mechanism for the Nutanix capex.
- Forgetting to check ISV requirements before committing.
- Treating skeptical engineers as obstacles rather than potential champions.

**Why this question matters for NCX:** The NCX panel will probe both your technical reasoning and your understanding of organizational dynamics. A pure-technology answer fails. A pure-organizational answer also fails. The right answer integrates both.

---

**Q12.** *(NCX-style architectural defense, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / architectural defense. Write your response.

**The scenario:**
You are in front of a customer's VMware-loyal architect at a strategy review. He says:

> *"AHV is fine for greenfield workloads, but my mature ESXi environment has 15 years of integration: vROps, Aria Automation, Aria Operations for Logs, our SOAR pipeline reading vCenter events, our backup vendor's vCenter integration, our chargeback system pulling from vCenter APIs, our PAM/identity integration with vCenter SSO, our compliance scanners reading vCenter inventory. Switching hypervisors means rebuilding all of that. The licensing savings don't cover the integration cost."*

**The challenge:**
Respond. He has a real point. Address it without dismissing it.

**A strong answer covers:**
- **Acknowledge the integration cost is real and not trivial.** A mature ESXi environment with 15 years of tooling has real switching cost. Pretending otherwise costs you credibility.
- **Reframe by integration category:**
  - **Infrastructure monitoring (vROps, Aria Operations):** Prism Central provides equivalent functionality natively. Many customers find Prism's experience cleaner than Aria's. There is migration work but the destination is functional.
  - **Automation (Aria Automation):** Nutanix v4 REST API is well-documented and clean; Calm / NCM Self-Service replaces blueprint-style automation. Migration of automation is real work but the platform is API-first.
  - **Backup vendor integration:** Veeam, Commvault, Rubrik, Cohesity, HYCU all support AHV. Confirm with the specific vendor and version; for Tier-1 backup vendors AHV support is mature.
  - **SOAR / event pipelines:** Nutanix exposes events via API and via syslog-equivalent forwarding. The existing event consumers can usually be repointed; it's not a rewrite, it's a re-pointing.
  - **Identity / PAM:** Nutanix supports SAML, LDAP, AD integration. Existing identity infrastructure usually plugs into Prism with reasonable effort.
  - **Compliance scanners:** if the scanner uses VMware-specific APIs, that's real rework. If the scanner uses inventory APIs at a higher abstraction (CMDB), the change is smaller.
- **Stage the answer:** the integration cost is bounded. It is not "rebuild the world." It is "migrate roughly 10-15 integration points, each of which is a few-day to few-week project." Total integration migration cost is real but typically in the $100-300k range for a mature enterprise environment, depending on scope.
- **Reframe against the licensing savings:** if VMware licensing is $200k/year, the integration migration cost pays back in 1-2 years. After that, it is pure savings. State this comparison directly.
- **Offer a hybrid approach:** the architect's environment can stay on ESXi-on-Nutanix indefinitely. Integration stays intact. New workloads go to AHV. Drift over time. This is the durable win that does not force a rewrite.
- **Close with a concrete next step:** *"Let me build a workload-by-workload integration map with you. We'll mark which integrations are Nutanix-equivalent ready, which need re-pointing, and which would require rework. The map will tell us whether the licensing savings cover the integration cost in your specific environment. If the math doesn't work, the answer is 'stay on ESXi on Nutanix and capture the platform benefits without changing the hypervisor.' If the math works, we have a phased plan."*

**A weak answer misses:**
- Dismissing the integration cost as "not that bad."
- Promising integration parity without acknowledging the work.
- Not offering the hybrid path as the de-risked option.
- Not closing with a concrete proposal (the integration mapping exercise) that turns the conversation into a follow-up meeting.
- Hand-waving on specific integrations (vROps, SOAR, etc.) the architect named. He named them specifically; respond specifically.

**Why this question matters for NCX:** NCX panels test whether you can defend an architecture against an informed customer-side architect with real history and real concerns. The right disposition is to engage the concern fully, acknowledge what is real, and reframe to a productive next step. This is also the disposition that wins enterprise deals.

---

## What You Now Have

You can now explain AHV to a VMware admin in 60 seconds without sounding defensive. You know it is KVM-based, with QEMU and libvirt and Open vSwitch underneath, and Acropolis as the Nutanix-specific control plane. You know the lineage and you know how to defend it.

You have four mental frames for AHV: the hypervisor without a vCenter, the hypervisor without a tax, KVM hardened for HCI, and one less management product to maintain. When a customer pushes from any of those angles, you have a frame ready.

You know what AHV genuinely lacks: FT, mature niche-ISV ecosystem in some categories, DRS-level scheduling sophistication, console UX parity in heavy desktop scenarios. You can name those gaps without flinching.

You know what AHV genuinely has that ESXi does not: no separate licensing, no vCenter dependency, DSF-native snapshots, one-click rolling upgrades, tight DR integration, OVS-based networking, single API surface.

You know the mixed-hypervisor story cold, which is the most important customer-facing positioning you have right now: *they don't have to switch hypervisors to get the platform benefits.* That sentence wins meetings.

You know the Broadcom math at the level required to walk through it on a whiteboard, with per-core pricing, cluster-level cost, and five-year savings comparison. You also know how to bracket the integration cost and reframe the comparison.

You have the migration tool: Nutanix Move, free, change-tracked bulk copy with brief cutover. You know what it does and what it does not (it is not live).

You have twelve practice questions worth of AHV-specific discrimination: the technology, the comparison, the migration tool, the workload fit, and two NCX-style design defenses covering five-year financial planning under organizational constraint and architectural defense against an integration-focused customer architect. Those are real points on NCP-MCI and real practice for NCX-MCI.

You are now ready for the management plane. AHV's control plane lives inside the cluster (Acropolis), but the user-facing experience lives in Prism Element and Prism Central. Module 4 makes that concrete.

---

## Cross-References

- **Previous:** [Module 2: The Nutanix Stack](./02-nutanix-architecture.md)
- **Next:** [Module 4: Prism (Element and Central)](./04-prism-management.md)
- **Glossary:** [AHV](./appendix-a-glossary.md#ahv) · [KVM](./appendix-a-glossary.md#kvm) · [Acropolis](./appendix-a-glossary.md#acropolis) · [Live Migration](./appendix-a-glossary.md#live-migration) · [ADS](./appendix-a-glossary.md#ads) · [NGT](./appendix-a-glossary.md#ngt) · [Nutanix Move](./appendix-a-glossary.md#nutanix-move) · [Open vSwitch](./appendix-a-glossary.md#open-vswitch) · [VirtIO](./appendix-a-glossary.md#virtio)
- **Comparison Matrix:** [Hypervisor Row](./appendix-b-comparison-matrix.md#hypervisor) · [Management Plane Row](./appendix-b-comparison-matrix.md#management-plane) · [Snapshots Row](./appendix-b-comparison-matrix.md#snapshots)
- **Objections:** [#4 "Why switch hypervisors?"](./appendix-d-objections.md#obj-004) · [#5 "What about FT?"](./appendix-d-objections.md#obj-005) · [#9 "My backup vendor doesn't support AHV"](./appendix-d-objections.md#obj-009) · [#21 "My team has 10 years of ESXi expertise"](./appendix-d-objections.md#obj-021)
- **Discovery Questions:** [Q-HYP-01 Current hypervisor footprint](./appendix-e-discovery-questions.md#q-hyp-01) · [Q-HYP-02 ISV requirements](./appendix-e-discovery-questions.md#q-hyp-02) · [Q-HYP-03 FT and lockstep workloads](./appendix-e-discovery-questions.md#q-hyp-03) · [Q-HYP-04 Integration tooling inventory](./appendix-e-discovery-questions.md#q-hyp-04) · [Q-HYP-05 Renewal timing and budget](./appendix-e-discovery-questions.md#q-hyp-05)
- **Sizing Rules:** [AHV CPU overcommit ratios](./appendix-f-sizing-rules.md#cpu-overcommit) · [Memory headroom for AHV](./appendix-f-sizing-rules.md#memory-headroom)
- **CLI Reference:** [`acli` for AHV operations](./appendix-g-cli-reference.md#acli) · [`virsh` for KVM diagnostics](./appendix-g-cli-reference.md#virsh)
- **Competitive Matrix:** [AHV vs ESXi line by line](./appendix-h-competitive-matrix.md#ahv-vs-esxi)
- **POC Playbook:** [AHV demo flow](./appendix-j-poc-playbook.md#ahv-demo)

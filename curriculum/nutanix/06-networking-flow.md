---
module: 06
title: Networking and Microsegmentation
estimated_reading_time: 38 min
prerequisites:
  - Modules 01 through 05
  - vSphere networking (vSS, vDS, port groups, VLAN trunking)
  - Working CE cluster
  - Familiarity with VLAN concepts and IP routing
key_terms:
  - Open vSwitch (OVS)
  - Bridge (br0, br0.local)
  - Bond
  - Virtual Network
  - VLAN
  - IPAM (IP Address Management)
  - Flow Network Security
  - Flow Virtual Networking (FVN)
  - VPC (Virtual Private Cloud)
  - Microsegmentation
  - Category-driven policy
  - Service Chain / Service Insertion
diagrams:
  - ahv-networking-stack
  - flow-microsegmentation
  - flow-virtual-networking
cert_coverage:
  NCA: ~10%
  NCP-MCI: ~18%
  NCP-NS: ~50% (heavy)
  NCM-MCI: ~12%
sa_toolkit:
  related_objections: [obj-016, obj-017, obj-020, obj-025]
  related_discovery: [q-net-01, q-net-02, q-net-03, q-net-04, q-sec-01]
---

# Module 6: Networking and Microsegmentation

> **Cert coverage:** NCA (~10%) · NCP-MCI (~18%) · NCP-NS (~50% heavy) · NCM-MCI (~12%)
> **SA toolkit:** Objections #16, #17, #20, #25 · Discovery Q-NET-01 through Q-NET-04, Q-SEC-01

---

## The Promise

By the end of this module you will:

1. **Configure AHV networking from vSphere muscle memory.** You know vSwitches, port groups, VLAN trunking, NIC teaming. AHV uses different vocabulary (Open vSwitch, bridges, bonds, virtual networks) for largely equivalent concepts. By the end of this module the mapping is automatic.
2. **Pass roughly half of NCP-NS, the newest specialty cert.** NCP-NS (Nutanix Certified Professional – Network and Security) is Nutanix's network-and-security cert; the 7.5 version opened for public scheduling in April 2026. Roughly half of its blueprint is the material in this module: AHV networking, Flow Network Security, Flow Virtual Networking. The other half is Module 7 territory (DR, replication networking) plus deeper Flow operations.
3. **Build a microsegmentation policy using categories** and explain why category-driven policy is operationally cleaner than the IP-based ACL world your customer has been living in.
4. **Defend Flow against an NSX-T-loyal architect.** The honest comparison: Flow Network Security is competitive with NSX-T's distributed firewall for VM microsegmentation, often simpler. Flow Virtual Networking is younger than NSX-T and behind it for advanced routing patterns. You should be able to draw the comparison without flinching.
5. **Make the network economics case.** Flow Network Security is included in NCM Pro (or higher); Flow Virtual Networking is also bundled. NSX-T is a separate VMware product with its own licensing. The licensing math is meaningful and you should be able to walk through it.
6. **Diagnose AHV networking issues from the CLI.** When NCC reports a network anomaly or a customer's VM has connectivity problems, you should know where to look: OVS bridges, bonds, virtual network configuration, Flow policy state.

The network is the dimension where customers diverge most. Some come from a Cisco-only world and don't trust software-defined networking. Some come from heavy NSX-T deployments and want to know what they lose. Some treat the network as commodity. Read the customer; pick the framing.

---

## Foundation: What You Already Know

Picture a vSphere host. It has physical NICs (vmnics in VMware vocabulary). Those NICs are aggregated into a vSwitch (vSS for standard, vDS for distributed). Port groups on the vSwitch define VLAN tagging and policy. Each VM connects to a port group. NIC teaming is configured at the vSwitch level (active-active, active-standby, route-based-on-IP-hash, etc.).

vCenter centralizes this on vDS: one configuration deployed across many hosts, with consistent port groups and policies.

You also probably know the limits. vSS configuration is per-host; you maintain it everywhere. vDS requires Enterprise Plus (post-Broadcom: bundled into Foundation/Cloud Foundation tiers). NIC teaming policies are well-defined but bonded across many configurations over the years.

Now switch underneath. AHV networking uses **Open vSwitch (OVS)**: an open-source production-grade software switch that runs in the Linux kernel of every AHV host. Rather than VMware's proprietary vSwitch, OVS is the same switch used in Red Hat OpenStack, OVN, and many Linux-based virtualization platforms. The vocabulary differs:

| vSphere Term | AHV / OVS Term |
|---|---|
| vSphere Distributed Switch (vDS) | (no exact equivalent; configuration is per-host but managed centrally via Prism Central) |
| Standard vSwitch (vSS) | Open vSwitch bridge (typically `br0`) |
| Port Group | Virtual Network (or "subnet" in Prism vocabulary) |
| VLAN trunk on uplinks | VLAN configuration on the bond / bridge |
| NIC Teaming Policy (active-standby, IP hash) | Bond Mode (active-backup, balance-tcp, balance-slb, LACP) |
| vmkernel ports (management, vMotion, etc.) | Internal interfaces on `br0.local` (CVM/host management) |

Hold that mapping. The rest of this module fills it in.

> [!FROM-THE-SA-CHAIR]
> When a vSphere admin asks "is this just vSwitches with different names?", the right answer is partly yes and partly no. *"OVS gives you the equivalent of a distributed switch managed via Prism Central, with bridges, bonds, and VLAN configuration that map cleanly onto your vDS knowledge. The vocabulary is different but the operational concepts transfer in an afternoon. The bigger story is what comes on top: Flow gives you NSX-style microsegmentation included in the NCM Pro licensing, with category-driven policy that's easier to operate than IP-based ACLs."* That sentence reframes the conversation from "another network stack to learn" to "you get distributed switching and microsegmentation included."

---

## Core Content

### Open vSwitch: The Substrate

OVS is the software switch underneath every AHV host. It runs in the Linux kernel. Each AHV host has at least two bridges by default:

- **`br0`** (the data bridge). The bridge that carries user VM traffic. Connected to the host's physical NICs (typically via a bond). This is where your VMs' NICs land.
- **`br0.local`** (the local management bridge). Used for CVM-to-hypervisor traffic and internal management. Not exposed to user VMs.

Inside `br0`, OVS handles VLAN tagging, MAC learning, broadcast/multicast, and (for Flow Network Security) flow rules that implement the firewall.

> [!ON-THE-EXAM] **NCP-MCI · NCP-NS**
> OVS bridges are testable. Memorize: `br0` is the user-traffic bridge connected to physical NICs via a bond. `br0.local` is the management bridge for CVM-hypervisor communication. Common trap: questions that imply a single bridge handles everything (false; the separation between data and management is intentional). On NCP-NS specifically, expect questions about how Flow rules attach to OVS at the bridge level.

### Bonds: NIC Teaming for AHV

A **bond** in AHV is a Linux bonding interface (or, more recently, OVS bond) that aggregates multiple physical NICs into a single logical link. Bonds provide redundancy and (for some modes) load balancing.

**Bond modes you should know:**

| Bond Mode | What It Does | When to Use |
|---|---|---|
| **active-backup** | One active NIC; others standby. Failover on link loss. | Default. Simplest. Works with any switch. |
| **balance-slb** (Source Load Balancing) | OVS balances traffic across NICs based on source MAC. No switch-side configuration required. | When you want better-than-active-backup throughput without configuring the switch for LACP. |
| **balance-tcp** | Balances per-flow based on TCP/IP headers. Requires LACP on the switch. | Best throughput with proper switch coordination. |
| **LACP (active-active)** | 802.3ad link aggregation. Requires LACP on the switch. | Production deployments with proper switch configuration. |

**The default for new clusters** is typically active-backup, which is robust but does not load-balance traffic across multiple NICs. Most production deployments upgrade to balance-slb (no switch-side change needed). LACP / balance-tcp is supported and gives the best throughput with proper switch coordination, but **Nutanix's official recommendation has historically leaned away from LACP** because misconfigured upstream switches can disable cluster connectivity in ways that are hard to recover from in the field. If you choose LACP for a customer, document the switch-side configuration carefully and validate before go-live. The choice depends on the network team's discipline and the switch capabilities.

> [!FAMILIAR]
> Bond modes map cleanly onto vSphere NIC teaming policies. Active-backup is "Use Explicit Failover Order." Balance-slb is roughly analogous to "Route Based on Source MAC Hash." LACP is "Route Based on IP Hash" with LACP enabled on the vDS. The concepts transfer; the configuration UI is different.

> [!DIFFERENT]
> AHV bonds are configured per host but managed centrally via Prism Central in newer AOS versions (the "Network Configuration" workflow, sometimes called Network Controller). For legacy configurations, bonds are managed via `manage_ovs` from the AHV CLI. Plan to use the Prism-driven workflow for new deployments; legacy CLI is for troubleshooting and edge cases. This central management gives you something close to vDS-style cluster-wide consistency.

### Virtual Networks (the Port Group Equivalent)

A **Virtual Network** in Nutanix vocabulary is the equivalent of a vSphere port group. It carries:

- **VLAN tag** (or none, for untagged traffic).
- **Optional IPAM** (IP Address Management): the cluster can act as DHCP for VMs on this network.
- **Connection to a bridge** (typically `br0`).

When you create a VM and attach it to a Virtual Network, the VM's vNIC lands on `br0` with the configured VLAN. Inter-VM traffic on the same VLAN flows at OVS speed (in-host, kernel-level). Inter-VLAN traffic exits via the bond uplink to the physical network and is routed there.

**Naming convention:** virtual networks in Prism are commonly named `vlan.<id>` (e.g., `vlan.100`, `vlan.200`) for clarity, though arbitrary names are allowed.

### IPAM: When Nutanix Acts as Your DHCP

For each Virtual Network, you can enable **IPAM** (IP Address Management). When IPAM is enabled, AHV provides DHCP services for VMs on that network. You configure:

- The subnet (e.g., `10.20.30.0/24`)
- The default gateway (e.g., `10.20.30.1`)
- The DNS servers
- The DHCP pool range (e.g., `10.20.30.100` through `10.20.30.200`)

This is useful for self-contained tenant networks (test/dev, isolated environments, VDI floating pools) where you don't want to rely on external DHCP. For production environments connected to the corporate network, you typically disable IPAM and let the physical network's DHCP serve VMs.

> [!ON-THE-EXAM] **NCP-MCI · NCP-NS**
> IPAM is testable. Memorize: IPAM provides DHCP services to VMs on a Virtual Network where it is enabled. The cluster acts as the DHCP server for that subnet. IPAM is per-virtual-network. Trap distractor: IPAM is required for any AHV virtual network (false; IPAM is optional and typically disabled when an external DHCP server exists).

---

### Diagram: AHV Networking Stack

**id:** `ahv-networking-stack`
**type:** layered
**caption:** From physical NICs up to VMs. Bonds aggregate NICs, bridges carry traffic, virtual networks provide VLAN-tagged port groups.
**exam_relevance:** [NCA, NCP-MCI, NCP-NS]
**whiteboard_ready:** true

**Elements (bottom to top):**
- Bottom: Physical NICs (gray): "eth0, eth1" with cable icons
- Above NICs: Bond (teal): "bond0 (active-backup or LACP)"
- Above the bond: OVS Bridge (teal): "br0 (data bridge)"
- Above br0: Multiple Virtual Networks (gold), shown as separate boxes:
  - "vlan.100 (Production, with IPAM disabled)"
  - "vlan.200 (Dev, with IPAM enabled, 10.20.30.0/24)"
  - "vlan.50 (DMZ, with IPAM disabled)"
- Above the virtual networks: VMs (light gray): "VM-Web-01, VM-DB-01, VM-Dev-01"
- A separate, smaller stack on the side: "br0.local (management bridge)" with internal interfaces for CVM and hypervisor management
- Side annotation: Prism Central / Prism Element manages this stack

**Connections:**
- Physical NICs to bond: "Aggregated for redundancy"
- Bond to br0: "Carries all VM traffic"
- Br0 to virtual networks: "VLAN-tagged or untagged"
- Virtual networks to VMs: "VM vNICs attach here"
- Br0.local to CVM: "Management traffic"
- Prism Element to bridges/bonds: "Configuration management"

**Annotations:**
- Beside the bond: "Bond mode determines failover and load-balancing behavior."
- Beside virtual networks: "Per-network IPAM optional. Per-network VLAN required."
- Beside br0.local: "Internal management. Not exposed to user VMs."
- Below: "Open vSwitch (OVS) is the kernel-level switch in AHV. It runs natively in Linux and is the same OVS used in OpenStack and many other production platforms."

**Why this diagram exists:** vSphere admins map this onto their vSS/vDS mental model in seconds when they see it laid out this way. Whiteboard it during AHV networking conversations. The IPAM annotation prevents the most-common confusion ("does AHV require its own DHCP?"). Memorize the layout.

---

### The Cycle, Frame Two: AHV Networking as Linux Plus OVS

For a network engineer who comes from the Linux world, this frame is more natural. AHV's networking is just Linux networking with Open vSwitch as the switch. The CLI tools you would use on any Linux host (`ip`, `ovs-vsctl`, `ovs-ofctl`, `ip link`) work on the AHV hypervisor.

This means:
- Standard troubleshooting tools work. `tcpdump` on `br0` shows you VM traffic. `ovs-vsctl show` lists bridges and ports. `ovs-appctl` queries OVS internals.
- Standard automation works. Ansible, Terraform, custom scripts can manipulate OVS configuration where the Nutanix abstraction is insufficient.
- The architecture is auditable. There is no proprietary kernel module hiding what the network does. OVS is open source and well-understood.

For network teams that have been frustrated by VMware's proprietary networking stack, this is a real attraction. For teams that prefer the abstraction, the Prism management plane provides equivalent simplicity.

### The Cycle, Frame Three: Networking as Policy, Not Topology

In traditional networking, you secure VMs by VLAN segmentation, ACLs on switch ports, and firewall rules at gateways. Topology drives policy: a VM is in `vlan.100`, and policy is whatever applies to `vlan.100`. Move the VM to a different VLAN, the policy changes.

Flow Network Security inverts this. Policy is defined by **categories** (Module 4) attached to VMs, not by topology. A VM tagged `Tier: Web` follows the Web-tier policy regardless of which VLAN it lives in. Move the VM to a different cluster, the policy follows.

This is the operational frame that wins customer hearts when the customer's security architect understands it. We will get to Flow specifically below; for now, hold the frame: in the AHV/Flow world, network policy follows the VM.

### The Cycle, Frame Four: What Comes Free vs What Costs

For an economics-minded customer, the durable network story is licensing.

In the VMware world, the network capabilities scale up by license tier:
- vSS (standard switch): included with the basic hypervisor.
- vDS (distributed switch): requires Enterprise Plus or higher (now bundled in Foundation tiers).
- NSX-T microsegmentation: separate NSX licensing, per-CPU or per-workload.
- NSX-T routing/edge services: even higher NSX tiers.

In the Nutanix world:
- AHV networking with OVS, virtual networks, IPAM: included with AOS / NCI baseline.
- **Flow Network Security (microsegmentation):** licensed via **NCI Ultimate**, or via the optional **Security Add-On** for NCI Pro (the Security Add-On bundles Flow microsegmentation with Data-at-Rest Encryption, both software and SED, and is licensed per usable TiB). It is **not** an NCM tier feature; do not confuse the NCM management tiers with Flow's NCI-side licensing.
- **Flow Virtual Networking (VPC overlay, advanced routing):** included with PC / AOS at the platform level; specific feature gating depends on PC and AOS version.

The customer who has been paying separately for NSX-T microsegmentation will find Flow Network Security at the NCI Ultimate tier (or as a Security Add-On) a meaningful licensing change. Always run the actual numbers; the answer depends on tier, workload size, and whether the Security Add-On's encryption capabilities also displace another vendor's tooling.

---

### Flow Network Security: Microsegmentation Done Differently

**Flow Network Security** is Nutanix's distributed firewall and microsegmentation product. It runs across all AHV hosts in the cluster (Prism Central drives policy distribution; OVS enforces rules at each host).

**The model:**
- Define **categories** (Module 4) on VMs (e.g., `Tier: Web`, `Tier: App`, `Tier: DB`, `Environment: Prod`).
- Define **rules** that apply to category groups: "VMs with `Tier: Web` may receive traffic from anywhere on port 443; may send traffic to VMs with `Tier: App` on port 8080; may not communicate with `Tier: DB` directly."
- Rules are **stateful** (return traffic is automatically permitted for established connections).
- Enforcement is **distributed**: each host enforces rules for VMs running on that host. There is no central firewall bottleneck.
- Default policy is configurable: explicit-allow (whitelist, the secure default) or explicit-deny.

**A typical microsegmentation policy:**

```
Web tier (VMs with Tier: Web):
  Inbound from any: 443/tcp, 80/tcp (allow)
  Outbound to App tier: 8080/tcp (allow)
  Default: deny

App tier (VMs with Tier: App):
  Inbound from Web tier: 8080/tcp (allow)
  Outbound to DB tier: 5432/tcp (allow)
  Default: deny

DB tier (VMs with Tier: DB):
  Inbound from App tier: 5432/tcp (allow)
  Outbound: deny
  Default: deny
```

The result: a three-tier application where each tier can only reach the next; lateral movement (e.g., a compromised Web VM directly reaching DB) is blocked at the OVS flow-rule level. No physical network changes required. No VLAN reorganization required. Categories assigned in Prism Central drive everything.

> [!FROM-THE-SA-CHAIR]
> Microsegmentation is the security demo that converts customers. The 90-second demo: "Watch me build a three-tier microsegmentation policy in front of you. Web can talk to App, App can talk to DB, no other lateral movement allowed. I will deploy three VMs with the right categories, deploy the policy, and show you the traffic blocked." Customers who have tried to build equivalent ACLs on physical switches respond viscerally to seeing this work in 90 seconds. Always offer the demo when discovery surfaces a security or compliance team.

> [!ON-THE-EXAM] **NCP-NS**
> Flow Network Security is the largest topic on NCP-NS. Memorize: Flow uses categories to scope rules; rules are stateful; enforcement is distributed across hosts via OVS flow rules; default policies can be allowlist or denylist; policy distribution comes from Prism Central. Trap distractors: questions implying Flow requires a central firewall appliance (false; enforcement is distributed), or that Flow rules are based on IP addresses (categories are the primary mechanism, though IP-based rules are also supported).

---

### Diagram: Flow Microsegmentation

**id:** `flow-microsegmentation`
**type:** flow
**caption:** Three-tier application secured with category-driven Flow Network Security. Lateral movement blocked at OVS flow-rule level on each host.
**exam_relevance:** [NCP-NS, NCP-MCI]
**whiteboard_ready:** true

**Elements:**
- Three category groups arranged left to right:
  - Web tier: 3 VMs labeled "VM with `Tier: Web`" (light gray)
  - App tier: 3 VMs labeled "VM with `Tier: App`" (light gray)
  - DB tier: 2 VMs labeled "VM with `Tier: DB`" (light gray)
- Above the VMs, policy boxes (gold):
  - "Web rule: inbound 443 from any; outbound 8080 to App"
  - "App rule: inbound 8080 from Web; outbound 5432 to DB"
  - "DB rule: inbound 5432 from App; default deny"
- Below the VMs, the OVS bridges on each host (teal): "OVS flow rules enforce policy at the host"
- An arrow from the user / internet (drawn at top) coming in: "443/tcp"
- Allowed flows shown with green arrows: Web → App on 8080, App → DB on 5432
- Blocked flows shown with red X: Web → DB direct (denied), DB → outbound (denied)

**Connections:**
- Internet → Web tier on 443: ALLOW
- Web → App on 8080: ALLOW
- App → DB on 5432: ALLOW
- Web → DB on 5432: DENY (blocked at OVS)
- DB → outbound: DENY

**Annotations:**
- "Categories drive policy, not VLANs."
- "Move a VM to another cluster: policy follows."
- "Enforcement is distributed across hosts; no central firewall bottleneck."
- "Compare to traditional ACL approach: configure switch ACL, configure each switch port, hope nothing changes."

**Why this diagram exists:** Microsegmentation is hard to explain without a picture. This diagram makes the policy-by-category approach visible and contrasts it implicitly with the traditional VLAN-and-ACL approach. Whiteboard it for security-team conversations.

---

### Flow Virtual Networking (FVN): The VPC Overlay

**Flow Virtual Networking** is Nutanix's overlay network product. It provides VPC-style virtual networks: isolated, multi-subnet networks with their own routing, NAT, and policy, decoupled from the underlying physical network topology.

**What it provides:**
- **Virtual Private Clouds (VPCs).** Create isolated network environments with their own IP space.
- **Routing between VPCs.** Internal routing rules, similar to AWS VPC peering.
- **NAT.** Inbound and outbound NAT for VPC-to-physical traffic.
- **External connectivity.** Connect VPCs to the physical network via gateway services.
- **BGP integration.** Some configurations support BGP with the upstream physical network.
- **Service insertion.** Insert third-party network services (firewalls, load balancers) into VPC traffic flow.

FVN is meaningfully newer than Flow Network Security (introduced in PC 2022+, with substantial maturity gains in 2024-2025). For customers with simple network requirements (flat or VLAN-segmented), FVN is overkill. For customers who want true overlay networking, multi-tenant VPCs, or hybrid-cloud network parity (where AWS VPCs and on-prem networks should look similar), FVN is the right tool.

**The honest comparison to NSX-T:** FVN is younger and has fewer features than NSX-T's overlay networking. NSX-T has years of routing-pattern depth, third-party ecosystem maturity, and edge-services breadth. FVN is catching up; it is not yet ahead. For customers with deep NSX-T investment and complex network requirements, NSX-T-on-Nutanix-on-ESXi is often the right pattern, with FVN as a future migration consideration.

> [!DIFFERENT]
> Flow Virtual Networking and NSX-T overlap in concept but differ in maturity. NSX-T is more mature for advanced routing patterns, edge services, and the third-party ecosystem. FVN is simpler to operate, included in the platform, and increasingly capable. For typical multi-tenant VPC use cases, FVN is sufficient and the licensing math favors it. For complex routing or established NSX-T deployments, NSX-T retains an edge. Be honest about which scenario your customer is in.

---

### Diagram: Flow Virtual Networking Topology

**id:** `flow-virtual-networking`
**type:** architecture
**caption:** A multi-VPC FVN deployment with internal routing, NAT, and physical-network integration.
**exam_relevance:** [NCP-NS]
**whiteboard_ready:** false

**Elements:**
- Center: Nutanix cluster (drawn as multiple nodes)
- Inside the cluster, three VPC blocks (gold):
  - VPC-Production (10.10.0.0/16)
  - VPC-Dev (10.20.0.0/16)
  - VPC-Shared-Services (10.30.0.0/16)
- Each VPC contains subnets with VMs (light gray)
- Routing rules shown between VPCs (rust arrows): "Production ↔ Shared (allowed)"; "Dev ↔ Shared (allowed)"; "Production ↔ Dev (denied)"
- North side: Physical network gateway (gray) labeled "Corporate Network"
- A VPC Gateway (gold) bridging VPCs to physical: NAT and routing
- BGP shown as a label on the gateway: "BGP peering with corporate routers"
- South side: Optional service insertion: "Third-party firewall (e.g., Palo Alto VM-Series)" with traffic redirected through it

**Connections:**
- VMs to VPC subnets: standard
- VPCs to each other via routing rules
- VPCs to physical network via VPC gateway
- BGP integration with upstream routers
- Service insertion: traffic redirected through third-party security VM

**Annotations:**
- "VPCs are isolated by default; routing rules enable controlled inter-VPC communication."
- "Gateway provides NAT and BGP integration with the physical network."
- "Service insertion lets you chain third-party security or load-balancing into VPC traffic."
- "Compare to AWS VPC concepts: very similar mental model."

**Why this diagram exists:** Customers with cloud-native mental models or multi-tenant requirements respond to VPC topology diagrams. This makes FVN's capabilities concrete. Use it when discovery surfaces multi-tenancy or hybrid-cloud-parity requirements.

---

### Service Insertion: Third-Party Security Integration

For customers who run third-party network security (Palo Alto VM-Series, Check Point, Fortinet, etc.) and want to insert those services into Flow's traffic flow, FVN supports **service insertion**. The pattern:

1. Deploy the third-party security VM (e.g., a Palo Alto VM-Series instance) on the cluster.
2. Configure FVN to redirect specific traffic flows through the security VM.
3. The security VM inspects, applies its own policy, and forwards (or drops).

This lets customers leverage their existing security tooling without giving up Flow Network Security for VM-level microsegmentation. The two layers complement: Flow does VM-tier microsegmentation; the third-party service does deep packet inspection, IDS/IPS, threat prevention.

**The honest framing for customers:** Flow Network Security is competitive for microsegmentation. Flow Virtual Networking provides VPC-style overlays. For deep packet inspection and advanced threat prevention, you typically still want a specialized security product (Palo Alto, Check Point, Cisco). Service insertion lets you have both.

---

### What Networking Genuinely Lacks Compared to NSX-T

Honest gap list. Read it carefully.

1. **Advanced routing patterns.** NSX-T has years of depth in distributed routing, BGP integration, OSPF, dynamic routing protocols. FVN handles common cases well but does not match NSX-T's full routing feature set.
2. **Edge services maturity.** NSX-T's edge services (load balancing, VPN, gateway firewall, IDS/IPS) are mature and well-integrated. Nutanix relies more on service insertion for advanced services.
3. **Third-party ecosystem depth.** NSX-T has had more time to develop integrations with third-party security and networking products. Flow's ecosystem is meaningful and growing but younger.
4. **Geographic / multi-cluster networking.** NSX-T Federation provides multi-site network policy with strong consistency. FVN is increasingly capable here but not yet at NSX-T Federation parity.
5. **Some performance-tuning depth at the network plane.** NSX-T has been optimized over many years for very large, very dense network deployments. Flow handles typical enterprise scale well; for hyperscale, NSX-T may still have an edge.

For typical mid-market and enterprise general-purpose deployments, none of these are deal-breakers. For customers running deep NSX-T deployments or with hyperscale requirements, the gaps are real.

### What Networking Has That Differentiates Nutanix

1. **Open vSwitch as the substrate.** Open standard, auditable, well-understood. Many engineers prefer this to proprietary kernel modules.
2. **Category-driven policy.** Flow's category model is operationally cleaner than IP-based ACLs for typical enterprise use. (NSX-T also supports tag-based policy; the comparison favors Flow on simplicity for typical use.)
3. **Bundle-friendly licensing for microsegmentation.** Flow Network Security ships with NCI Ultimate (or via the Security Add-On for NCI Pro); NSX-T microsegmentation is a separate VMware product with its own per-CPU or per-workload licensing. Run the actual customer numbers, but the typical comparison favors Flow.
4. **Single management plane.** Network configuration, microsegmentation, and policy all live in Prism Central alongside compute and storage.
5. **Native integration with categories from compute and storage.** A single category like `Environment: Production` drives backup policy, microsegmentation, quotas, and reporting.

---

## Lab Exercise: Build a Microsegmentation Policy

> [!LAB] **Time:** ~2.5 hours · **Platform:** 3-node CE cluster with Prism Central deployed (from Module 04)

This lab walks you through configuring AHV networking, deploying VMs onto a virtual network, building a Flow Network Security microsegmentation policy with categories, and verifying the policy enforcement.

**Note:** Flow Network Security on Community Edition is limited; some features may not be available. The lab still teaches the workflow and concepts.

**Steps:**

1. **Inventory existing networking.** From a CVM, run:
   ```
   manage_ovs show_uplinks
   manage_ovs show_bridges
   manage_ovs show_bonds
   ```
   Note the default bridge (`br0`) and bond configuration.

2. **Create three Virtual Networks** in Prism Element:
   - Network 1: `vlan.100` for Web tier (VLAN 100, IPAM disabled)
   - Network 2: `vlan.200` for App tier (VLAN 200, IPAM enabled, 10.20.30.0/24)
   - Network 3: `vlan.300` for DB tier (VLAN 300, IPAM enabled, 10.20.40.0/24)
   
   For lab purposes, all three can use the same VLAN if your CE switch infrastructure is limited; the policy concepts work the same.

3. **Deploy three VMs** (one for each tier). Use any small Linux image. Assign each VM to its respective virtual network. Boot them.

4. **Define categories in Prism Central** (from Module 4 lab; you may already have this set up):
   - Key: `Tier`, Values: `Web`, `App`, `DB`

5. **Apply categories to your VMs.** From Prism Central VM list, assign:
   - VM-Web-01: `Tier: Web`
   - VM-App-01: `Tier: App`
   - VM-DB-01: `Tier: DB`

6. **Create a Flow Network Security policy** in Prism Central:
   - Navigate to Policies > Security Policies > Create Security Policy.
   - Application Type: 3-tier (Web/App/DB)
   - Define rules:
     - Web tier: inbound from Any on 80/tcp and 443/tcp
     - App tier: inbound from Web tier on 8080/tcp
     - DB tier: inbound from App tier on 5432/tcp
     - Default: deny everything else
   - Apply to category-defined VMs. Save and enable the policy.

7. **Test the policy.**
   - From VM-Web-01, ping VM-App-01: should fail (ICMP not in rules).
   - From VM-Web-01, `nc -v VM-App-01 8080`: should succeed once a service is listening.
   - From VM-Web-01, attempt to reach VM-DB-01 directly: should fail.
   - From VM-App-01, reach VM-DB-01 on 5432: should succeed.

8. **Inspect the OVS flow rules** that Flow has installed. From an AHV host:
   ```
   ovs-ofctl dump-flows br0
   ```
   You should see flow entries that implement the firewall rules. This is what is doing the work at runtime.

9. **Optional: enable a "monitor mode"** policy first to capture traffic without enforcing, then enable enforcement. This is the recommended customer-deployment pattern: monitor for a period, validate the rules don't break legitimate traffic, then enforce.

**What this teaches you:**
- Virtual network and VLAN configuration in AHV.
- Category assignment and category-driven policy.
- Building a real microsegmentation policy from scratch.
- The CLI surface for OVS troubleshooting.
- The customer-recommended monitor-mode-then-enforce pattern.

**Customer-demo angle:** This lab is the microsegmentation demo. Steps 4-7 take roughly 15 minutes once you have the VMs running. Customers respond to seeing real traffic flow through and seeing the deny actually deny. Always offer this demo when security teams are in the conversation.

---

## Practice Questions

Twelve questions. Six knowledge MCQ, four scenario MCQ, two NCX-style design questions. Read each, answer in your head, then read the explanation.

---

**Q1.** What is the role of `br0` in AHV networking?
**Cert relevance:** NCA · NCP-MCI · NCP-NS

A) The local management bridge for CVM-hypervisor traffic
B) The default OVS bridge that connects to physical NICs (via a bond) and carries user VM traffic
C) A backup interface used only during failure scenarios
D) The DHCP server for AHV's IPAM functionality

**Answer:** B

**Why this answer:** `br0` is the default OVS data bridge. It connects to physical NICs (typically through a bond) and carries all user VM traffic. Virtual networks attach to `br0` with VLAN tags.

**Why not the others:**
- A) That describes `br0.local`, not `br0`. The two bridges have different purposes.
- C) `br0` is the active bridge in normal operation, not a backup.
- D) IPAM is a per-virtual-network feature, not a function of any bridge.

**The trap:** A is the seductive distractor for someone who confuses the data and management bridges. **Memorize:** `br0` is the data bridge for VM traffic. `br0.local` is for management.

---

**Q2.** Which of the following bond modes does NOT require corresponding configuration on the upstream physical switch?
**Cert relevance:** NCP-MCI · NCP-NS

A) LACP (active-active)
B) Balance-tcp
C) Active-backup
D) Both A and B

**Answer:** C

**Why this answer:** Active-backup uses one NIC at a time and switches over on link failure. It requires no special switch configuration; any switch supporting basic Ethernet works.

**Why not the others:**
- A) LACP requires the switch to support and be configured for 802.3ad link aggregation.
- B) Balance-tcp requires LACP-style link aggregation on the switch.
- D) Both require switch configuration.

**The trap:** Customers and test-takers who default to LACP as "the right answer" miss that LACP requires switch-side cooperation. Active-backup is the simpler, switch-agnostic default.

---

**Q3.** Which of the following describes Nutanix's IPAM (IP Address Management)?
**Cert relevance:** NCP-MCI · NCP-NS

A) A required service that handles all DHCP for any AHV VM
B) An optional per-virtual-network service where the cluster acts as a DHCP server for VMs on that network
C) A service that replaces the customer's existing AD-integrated DHCP server
D) A service that requires a separate licensing fee

**Answer:** B

**Why this answer:** IPAM is an optional per-virtual-network feature. When enabled, the cluster provides DHCP for that network's VMs. When disabled (typical for production networks), VMs use external DHCP from the corporate network.

**Why not the others:**
- A) IPAM is optional, not required.
- C) IPAM coexists with corporate DHCP; customers typically disable IPAM on networks where corporate DHCP serves.
- D) IPAM is included with AHV; no separate licensing.

**The trap:** A reflects a partial understanding. IPAM is a tool, not a requirement.

---

**Q4.** What does Flow Network Security primarily provide?
**Cert relevance:** NCP-MCI · NCP-NS

A) VPC-style overlay networking with multiple isolated subnets
B) Distributed firewall and microsegmentation, with rules driven by VM categories rather than IP addresses
C) Load balancing and reverse proxy services for inbound traffic
D) Deep packet inspection and IDS/IPS

**Answer:** B

**Why this answer:** Flow Network Security is the distributed firewall product. Rules are defined per category (e.g., `Tier: Web` can talk to `Tier: App` on 8080) and enforced by OVS flow rules on each AHV host.

**Why not the others:**
- A) That describes Flow Virtual Networking (FVN), a different product.
- C) Flow is not a load balancer; service insertion can chain third-party load balancers if needed.
- D) Flow has some intrusion-detection capabilities but is not primarily a deep-packet-inspection product.

**The trap:** Confusion between Flow Network Security (microsegmentation) and Flow Virtual Networking (overlays). Memorize: **Network Security = microsegmentation, Virtual Networking = overlays.**

---

**Q5.** Which of the following is true about Flow Network Security policy enforcement?
**Cert relevance:** NCP-NS

A) Policies are enforced at a central firewall appliance that all traffic must traverse
B) Policies are enforced at each AHV host via OVS flow rules; enforcement is distributed
C) Policies are enforced at the physical switch via ACLs configured by Nutanix
D) Policies are enforced only on inbound traffic; outbound is uncontrolled

**Answer:** B

**Why this answer:** Flow's distributed enforcement model means each host enforces rules for the VMs running on it, via OVS flow entries. This avoids the bottleneck of centralized firewalls and allows microsegmentation at scale.

**Why not the others:**
- A) Centralized enforcement is the older, traditional model. Flow specifically does not work this way.
- C) Flow operates at the OVS layer in AHV, not on physical switches.
- D) Stateful rules apply in both directions; outbound is fully controlled.

**The trap:** A is what older firewall architectures looked like. Test-writers know that this old mental model is sticky and use it as a distractor.

---

**Q6.** Which is the correct mapping of vSphere networking concepts to AHV/OVS concepts?
**Cert relevance:** NCA · NCP-MCI

A) vDS port group → AHV bridge; NIC team → bond; VLAN trunk → IPAM
B) vDS port group → Virtual Network; vSwitch → OVS bridge; NIC teaming policy → bond mode
C) vSS → Flow Network Security; vDS → Flow Virtual Networking; port group → category
D) vCenter → Open vSwitch; ESXi → AHV; port group → service insertion

**Answer:** B

**Why this answer:** vDS port groups map to AHV Virtual Networks. vSwitch maps to OVS bridge. NIC teaming policy maps to bond mode. This is the canonical translation.

**Why not the others:**
- A) Misaligns multiple terms; IPAM is unrelated to VLAN trunking.
- C) Conflates Flow products with vSphere base networking.
- D) Misaligns multiple concepts.

**The trap:** This question rewards practitioners who have actually built the mental mapping. Memorize the translation table.

---

**Q7.** A BlueAlly customer has 50 VMs running a three-tier application (Web, App, DB) on Nutanix AHV. They want to implement microsegmentation so Web cannot directly reach DB, even though all three tiers share the same VLAN. What is the recommended approach?
**Cert relevance:** NCP-NS · sales-relevant

A) Deploy NSX-T separately and configure microsegmentation through NSX-T
B) Reorganize the VMs into separate VLANs and configure ACLs on the physical switches
C) Define `Tier` categories on the VMs (Web, App, DB) and create Flow Network Security rules that allow Web → App → DB traffic flow but deny Web → DB direct
D) Deploy a virtual firewall and route all VM-to-VM traffic through it

**Answer:** C

**Why this answer:** This is exactly the use case Flow Network Security was designed for: VM-tier microsegmentation independent of VLAN topology, driven by categories. Distributed enforcement means no central firewall bottleneck.

**Why not the others:**
- A) NSX-T deployment is unnecessary for this; Flow handles it cleanly and is included with NCM Pro.
- B) VLAN reorganization is the old way; it requires switch reconfiguration and doesn't scale to many policies.
- D) Centralized firewall creates bottleneck; not the right architecture for VM microsegmentation at scale.

**The trap:** A and B are options that work but are operationally inferior. The exam (and the customer) reward you for picking the architecturally correct approach.

---

**Q8.** A customer's senior network architect says: "We've spent 4 years building NSX-T. Why would I look at Flow?" What is the strongest SA response?
**Cert relevance:** sales-relevant

A) "Flow is much simpler than NSX-T. You should switch."
B) "NSX-T continues to work on Nutanix-on-ESXi. You don't have to migrate. For new AHV workloads, Flow Network Security provides VM-tier microsegmentation that's competitive with NSX-T's distributed firewall, with included licensing. For your established NSX-T deployments, keep them; the platforms coexist. Let me understand your specific NSX-T use cases and we can decide where Flow fits."
C) "Flow has every NSX-T feature included for free."
D) "NSX-T is going away because of Broadcom; you should switch immediately."

**Answer:** B

**Why this answer:** Respects the architect's investment, names the coexistence pattern (NSX-T on ESXi-on-Nutanix continues to work), differentiates Flow Network Security from NSX-T's full feature set, and proposes a discovery conversation. This is the SA-chair answer that wins customer trust.

**Why not the others:**
- A) Dismissive of years of work. Loses credibility.
- C) Untrue. Flow Network Security is competitive for microsegmentation; NSX-T retains advantages for advanced routing and edge services.
- D) Speculative and aggressive. Costs you the relationship.

**The trap:** Confident overclaims (A, C) and competitive negativity (D) all read as sales pressure. The honest answer respects existing architecture and reframes to specific use cases.

---

**Q9.** Which of the following correctly distinguishes Flow Network Security from Flow Virtual Networking?
**Cert relevance:** NCP-NS

A) Flow Network Security provides overlay networks; Flow Virtual Networking provides microsegmentation
B) Flow Network Security provides distributed firewall and microsegmentation driven by categories; Flow Virtual Networking provides VPC-style overlay networks with internal routing, NAT, and physical-network integration
C) They are the same product, marketed under two names
D) Flow Network Security is for AHV; Flow Virtual Networking is for ESXi

**Answer:** B

**Why this answer:** This is the canonical distinction. Network Security = firewall/segmentation. Virtual Networking = overlays/VPCs/routing. Different products solving different problems, both part of the broader Flow family.

**Why not the others:**
- A) Reverses the products' purposes.
- C) Distinct products with different feature sets and use cases.
- D) Both are AHV-native; ESXi support varies and is not the dichotomy.

**The trap:** New learners often conflate the two Flow products because they share the brand name. Memorize the distinction precisely.

---

**Q10.** A customer has a 10-node Nutanix cluster running AHV with Flow Network Security enabled. They are considering enabling Flow Virtual Networking for a new multi-tenant project. Which statement is correct?
**Cert relevance:** NCP-NS · sales-relevant

A) Flow Virtual Networking and Flow Network Security cannot coexist on the same cluster
B) Flow Virtual Networking and Flow Network Security can coexist; FVN provides VPC-style isolation while Flow Network Security provides microsegmentation within or across VPCs
C) Enabling Flow Virtual Networking requires migrating to ESXi
D) Flow Virtual Networking replaces Flow Network Security

**Answer:** B

**Why this answer:** The two products are complementary. FVN provides VPC isolation and overlay networking; Flow Network Security provides VM-level microsegmentation. Many real deployments use both.

**Why not the others:**
- A) Not true; they coexist by design.
- C) FVN runs on AHV.
- D) Different products with different purposes; they do not replace each other.

**The trap:** B requires understanding that the products are layered, not competing. Customers who hear "Flow" and assume one product miss this nuance.

---

**Q11.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep · NCP-NS prep · sales-relevant
**Format:** Short-answer / design discussion. Write your reasoning. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A financial-services customer is deploying a new Nutanix cluster for their core banking application. The application is a three-tier system (Web, App, DB), with sensitive customer data in the DB tier. Compliance requires:
- VM-level microsegmentation (no lateral movement between tiers)
- Audit logging of all denied traffic
- Network isolation between Production and DR environments
- Service insertion of an existing Palo Alto VM-Series for deep packet inspection on incoming traffic
- Multi-tenant isolation between Banking, Wealth Management, and Internal Apps business units
- Integration with corporate identity for policy administration

The customer asks you to design the network and security architecture.

**The challenge:**
Walk through your design. Cover virtual networks, FVN VPCs (if applicable), Flow Network Security policy structure with categories, service insertion, audit logging, and identity integration. Identify what you still need to know.

**A strong answer covers:**
- **VPC topology with FVN.** Three VPCs, one per business unit (Banking, Wealth, Internal Apps). Each VPC has its own IP space and isolation by default. Inter-VPC routing rules defined for any required cross-business-unit shared services.
- **Per-VPC virtual networks.** Within each VPC, separate subnets for Web, App, DB tiers. VLAN tagging for routing semantics where physical-network integration is required.
- **Categories.** Define `BusinessUnit` (Banking/Wealth/InternalApps), `Tier` (Web/App/DB), `Environment` (Prod/DR), and `Compliance` (PCI/HIPAA/SOX as applicable). Apply to all VMs.
- **Flow Network Security policies.** Per-tier rules:
  - Web allows inbound 443 from internet (after Palo Alto inspection); outbound 8080 to App
  - App allows inbound 8080 from Web; outbound 5432 to DB
  - DB allows inbound 5432 from App only; outbound denied
  - Cross-business-unit denial by default; explicit rules for any shared-services exceptions
- **Service insertion.** Configure FVN service insertion to redirect all internet-facing inbound traffic through the Palo Alto VM-Series before it reaches the Web tier. This adds DPI without losing the Flow microsegmentation underneath.
- **Audit logging.** Configure Flow to log all denies. Forward logs to the customer's SIEM (Splunk, Elastic, etc.). This satisfies the compliance audit requirement.
- **Production/DR isolation.** Use Environment: Prod and Environment: DR categories. Cross-environment policies are denial-by-default; explicit replication-traffic rules permit Async / NearSync replication (Module 7).
- **Identity integration.** Prism Central with SAML to the customer's IdP (Azure AD, Okta, or similar). Map identity groups to Prism roles for policy administration. Audit policy changes via the platform's audit log.
- **What you still need to know:** specific compliance frameworks in play (PCI v4.0, GLBA, SOX), the Palo Alto VM-Series model and licensing the customer holds, whether the customer wants a separate management VPC for Prism Central, the specifics of the corporate-network BGP topology, and the audit-log retention requirements.

**A weak answer misses:**
- Defaulting to flat virtual networks without VPCs for multi-tenant isolation.
- Using IP-based ACLs instead of category-driven Flow rules.
- Forgetting service insertion for the existing Palo Alto VM-Series.
- Not addressing audit logging requirements.
- Treating Production/DR as the same security zone.
- Missing the Prism Central identity integration.

**Why this question matters for NCX:** NCX-MCI panels probe network and security design integration. A pure-feature answer fails. The right answer integrates VPC topology, microsegmentation policy, third-party service insertion, audit/compliance, and identity into one coherent design.

---

**Q12.** *(NCX-style architectural defense, open-ended)*
**Cert relevance:** NCX-MCI prep · NCP-NS prep · sales-relevant
**Format:** Short-answer / architectural defense. Write your response.

**The scenario:**
You are in front of a customer's senior network and security architect. He has 12 years of NSX-T experience. He says:

> *"Flow looks fine for basic microsegmentation, but our environment uses NSX-T's distributed routing, gateway services with stateful firewall, IDS/IPS at the edge, BGP integration with our SD-WAN, L2VPN to remote sites, and a third-party-integration ecosystem we've spent years tuning. Flow doesn't have most of that. Why would we walk away from NSX-T?"*

**The challenge:**
Respond. He has named real NSX-T capabilities. Address each.

**A strong answer covers:**
- **Acknowledge the NSX-T capability set is real and substantial.** You don't argue with 12 years of investment.
- **Reframe each capability:**
  - **Distributed routing.** FVN provides distributed routing for VPCs. NSX-T's routing is more mature for complex topologies. For simple-to-moderate routing patterns, FVN is sufficient. For your specific complexity, we walk through cases together.
  - **Gateway services / stateful firewall.** FVN gateways provide NAT and basic firewall. NSX-T's edge services (DLB, NAT, gateway firewall) are more feature-rich. For deep edge services, customers either retain NSX-T-on-ESXi or insert third-party services via FVN service insertion.
  - **IDS/IPS at the edge.** Service insertion in FVN is the equivalent path: insert a third-party IDS/IPS (Palo Alto, Check Point, Cisco) into the traffic flow. Flow's own IDS capabilities are growing but service insertion is the more capable answer for now.
  - **BGP / SD-WAN integration.** FVN supports BGP. SD-WAN integration depth varies by SD-WAN vendor; customers typically run SD-WAN at the corporate edge and FVN within the cluster, with BGP between. Confirm the specific SD-WAN integration the customer needs.
  - **L2VPN to remote sites.** Not natively in FVN; this is a legitimate gap for customers with that requirement. Customers with L2VPN needs typically retain NSX-T or use SD-WAN-overlay alternatives.
  - **Third-party ecosystem.** NSX-T has more depth here. Service insertion bridges some of this; absolute parity is not yet achieved.
- **The honest reframe:** "You don't have to walk away from NSX-T. NSX-T continues to run on Nutanix-on-ESXi, which means the cluster's HCI benefits don't require a network-architecture change. Flow Network Security is genuinely competitive for VM microsegmentation and is an option to evaluate for new workloads. For your full NSX-T feature set, NSX-T-on-Nutanix is the durable answer."
- **Close with a concrete proposal:** *"Let me build a feature-by-feature mapping with you. We'll mark which NSX-T capabilities Flow matches, which it partially matches, which require service insertion, and which require keeping NSX-T. The map will tell us where Nutanix-on-AHV-with-Flow makes sense, where Nutanix-on-ESXi-with-NSX-T makes sense, and whether a hybrid is the right destination. Can we agree on that mapping exercise as a follow-up?"*

**A weak answer misses:**
- Claiming Flow has all NSX-T features (it doesn't, and the architect knows it).
- Dismissing NSX-T as outdated.
- Not acknowledging the L2VPN gap.
- Missing the NSX-T-on-Nutanix-on-ESXi coexistence pattern.
- Not proposing the feature-mapping exercise as a concrete next step.

**Why this question matters for NCX:** This is a real customer conversation. NCX panels test whether you can defend Nutanix architecture against an informed incumbent network architect, acknowledging the gaps honestly while reframing to a productive next step. The customer's 12 years of NSX-T are a fact, not an obstacle to argue with.

---

## What You Now Have

You can now translate vSphere networking vocabulary to AHV/OVS terminology fluently. vDS port group becomes Virtual Network, vSwitch becomes OVS bridge, NIC teaming becomes bond mode. The mapping is in your hands.

You know the OVS substrate: `br0` for VM data traffic, `br0.local` for management. Bonds with active-backup, balance-slb, balance-tcp, and LACP modes. VLAN configuration on virtual networks. IPAM as an optional per-virtual-network DHCP service.

You have four mental frames for AHV networking: vDS-equivalent for VMware admins, Linux+OVS for network engineers, policy-not-topology for operations leaders, included-vs-licensed for economics-focused customers.

You know Flow Network Security: distributed firewall, category-driven policy, stateful rules, OVS-level enforcement, and the operational pattern of "monitor first, then enforce" for production deployments. You can build a three-tier microsegmentation policy in 90 seconds and demo it.

You know Flow Virtual Networking: VPC-style overlays, internal routing, NAT, BGP integration, service insertion. You know FVN is younger and less feature-complete than NSX-T but increasingly capable.

You have the honest comparison to NSX-T: Flow Network Security is competitive for microsegmentation; FVN is behind NSX-T on advanced routing, edge services, L2VPN, and third-party ecosystem maturity. You can position the coexistence pattern (NSX-T-on-Nutanix-on-ESXi) as the durable answer for customers with established NSX-T investment.

You know the licensing reality: Flow Network Security at NCM Pro is included; NSX-T microsegmentation is separately licensed. The economics matter and you can walk through them.

You have twelve practice questions worth of networking and security discrimination, including two NCX-style design defenses (financial-services compliance design, and the NSX-T architect defense conversation). The NCP-NS specialty cert is now substantially in your reach.

You are now ready for data protection and DR. Module 7 covers Protection Domains, Async/NearSync/Metro replication, Recovery Plans (Leap), Site Recovery Manager comparison, and the durable DR story that frequently decides enterprise deals.

---

## References

Authoritative sources verified during the technical review pass on this module. Flow licensing in particular is the most volatile area; reverify against the current Nutanix Cloud Platform software-options page before quoting tier-specific costs.

- [Nutanix Bible — AHV Architecture (networking)](https://www.nutanixbible.com/pdf/5a-book-of-ahv-architecture.pdf). Authoritative source for OVS bridges (br0, br0.local), bond modes, and the default networking topology.
- [Nutanix Bible — AHV Administration](https://www.nutanixbible.com/pdf/5c-book-of-ahv-administration.pdf). manage_ovs CLI reference, bond-mode configuration workflows.
- [AHV Bond Modes (Virtual Ramblings)](https://www.virtualramblings.com/define-and-differentiate-ahv-bond-modes/). Independent walkthrough of active-backup, balance-slb, balance-tcp, and LACP semantics.
- [Changing AHV Bonding Mode to LACP / Balance-TCP (Nutanix Community)](https://next.nutanix.com/ahv-virtualization-27/changing-bonding-mode-in-ahv-from-active-backup-to-active-active-lacp-and-balance-tcp-43521). Source for Nutanix's caution on LACP and the upstream-switch coordination requirement.
- [Flow Network Security Product Page](https://www.nutanix.com/products/flow-network-security). Current product positioning.
- [Nutanix Cloud Platform Software Options](https://www.nutanix.com/products/cloud-platform/software-options). Authoritative source for Flow Network Security licensing: included in NCI Ultimate, or via the Security Add-On for NCI Pro (per-usable-TiB pricing, bundles Data-at-Rest Encryption).
- [Nutanix Bible — Flow Network Security](https://www.nutanixbible.com/12a-book-of-network-services-flow-network-security.html). Distributed enforcement at OVS, category-driven policy, stateful rules.
- [Nutanix Bible — Flow Virtual Networking](https://www.nutanixbible.com/12c-book-of-network-services-flow-virtual-networking.html). VPC overlay architecture, NAT, BGP, service insertion.
- [Flow Virtual Networking Guide (PC 2024.2)](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Flow-Virtual-Networking-Guide-vpc_2024_2:Nutanix-Flow-Virtual-Networking-Guide-vpc_2024_2). Current FVN product documentation.
- [Exploring BGP Routing in FVN (Nutanix.dev, 2023)](https://www.nutanix.dev/2023/08/31/exploring-bgp-routing-inside-nutanix-flow-virtual-networking-fvn-vpc/). FVN BGP integration details.
- [NCP-NS 7.5 Open for Scheduling (Nutanix University)](https://en.vmik.net/2026/03/whats-new-at-nutanix-university-15/). Confirms the April 4, 2026 public exam launch date.
- [TN-2094 Flow Network Security Tech Note](https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2094-Flow:TN-2094-Flow). Detailed Flow technical reference.

---

## Cross-References

- **Previous:** [Module 5: DSF Deep Dive](./05-dsf-storage-deep-dive.md)
- **Next:** [Module 7: Data Protection and DR](./07-data-protection.md)
- **Glossary:** [Open vSwitch](./appendix-a-glossary.md#open-vswitch) · [Bridge](./appendix-a-glossary.md#bridge) · [Bond](./appendix-a-glossary.md#bond) · [Virtual Network](./appendix-a-glossary.md#virtual-network) · [VLAN](./appendix-a-glossary.md#vlan) · [IPAM](./appendix-a-glossary.md#ipam) · [Flow Network Security](./appendix-a-glossary.md#flow-network-security) · [Flow Virtual Networking](./appendix-a-glossary.md#flow-virtual-networking) · [Microsegmentation](./appendix-a-glossary.md#microsegmentation) · [Service Insertion](./appendix-a-glossary.md#service-insertion)
- **Comparison Matrix:** [Networking Row](./appendix-b-comparison-matrix.md#networking) · [Microsegmentation Row](./appendix-b-comparison-matrix.md#microsegmentation) · [Network Overlay Row](./appendix-b-comparison-matrix.md#overlay)
- **Objections:** [#16 "What about NSX-T?"](./appendix-d-objections.md#obj-016) · [#17 "Network performance vs my Cisco fabric"](./appendix-d-objections.md#obj-017) · [#20 "We've already invested in microsegmentation"](./appendix-d-objections.md#obj-020) · [#25 "Software-defined networking trust"](./appendix-d-objections.md#obj-025)
- **Discovery Questions:** [Q-NET-01 Physical network topology](./appendix-e-discovery-questions.md#q-net-01) · [Q-NET-02 NSX-T footprint](./appendix-e-discovery-questions.md#q-net-02) · [Q-NET-03 Microsegmentation requirements](./appendix-e-discovery-questions.md#q-net-03) · [Q-NET-04 Multi-tenancy / VPC needs](./appendix-e-discovery-questions.md#q-net-04) · [Q-SEC-01 Compliance frameworks](./appendix-e-discovery-questions.md#q-sec-01)
- **Sizing Rules:** [Network bandwidth per node](./appendix-f-sizing-rules.md#network-bandwidth) · [Bond / NIC teaming guidance](./appendix-f-sizing-rules.md#bond-mode-selection)
- **CLI Reference:** [`manage_ovs` commands](./appendix-g-cli-reference.md#manage-ovs) · [`ovs-vsctl` and `ovs-ofctl`](./appendix-g-cli-reference.md#ovs-tools)
- **Competitive Matrix:** [Flow vs NSX-T feature comparison](./appendix-h-competitive-matrix.md#flow-vs-nsx-t)
- **Reference Architectures:** [Multi-tenant VPC design](./appendix-i-reference-architectures.md#ra-multi-tenant) · [Microsegmentation reference](./appendix-i-reference-architectures.md#ra-microseg)
- **POC Playbook:** [Microsegmentation demo](./appendix-j-poc-playbook.md#microseg-demo) · [Flow vs NSX-T comparison demo](./appendix-j-poc-playbook.md#flow-nsx-comparison)

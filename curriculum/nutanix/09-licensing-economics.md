---
module: 09
title: Licensing and Real Costs
estimated_reading_time: 28 min
prerequisites:
  - Modules 01 through 08
  - Basic understanding of enterprise procurement and financial decision-making
  - Awareness of customer accounting concepts (capex, opex, depreciation)
key_terms:
  - NCI Pro / NCI Ultimate (current platform tiers)
  - AOS Pro / AOS Ultimate (legacy platform tier names; in conversion to NCI)
  - NCM Starter / Pro / Ultimate (paid management tiers)
  - NX appliance (Nutanix-branded hardware)
  - OEM partner (Dell XC, Lenovo HX, HPE DX, Cisco UCS)
  - Software-only / HCIR
  - BoM (Bill of Materials)
  - TCO (Total Cost of Ownership)
  - VCF (VMware Cloud Foundation)
  - Per-core subscription
  - Capex vs opex
  - Multi-year subscription term
caveat: |
  Specific pricing and tier contents change between AOS versions and quarterly Nutanix
  packaging updates. This module teaches the structure and methodology. For any
  customer-specific pricing, the BlueAlly internal pricing tools and current Nutanix
  price book are the authoritative sources. Never quote pricing from this curriculum
  in a customer conversation.
diagrams:
  - tier-feature-matrix
  - tco-components-over-time
  - broadcom-vs-nutanix-comparison
cert_coverage:
  NCA: ~5%
  NCP-MCI: ~5%
  NCM-MCI: ~5%
  NCP-US: ~5%
  note: Sales-relevant module; lighter cert weight, heavy customer-conversation weight
sa_toolkit:
  related_objections: [obj-035, obj-036, obj-037, obj-038, obj-039, obj-040]
  related_discovery: [q-econ-01, q-econ-02, q-econ-03, q-econ-04, q-econ-05]
---

# Module 9: Licensing and Real Costs

> **Cert coverage:** NCA (~5%) · NCP-MCI (~5%) · NCM-MCI (~5%) · NCP-US (~5%)
> **SA toolkit:** Objections #35 through #40 · Discovery Q-ECON-01 through Q-ECON-05
> **Important:** Specific pricing changes between AOS versions and quarterly Nutanix updates. The BlueAlly internal pricing tools are the authoritative source for any customer-facing number.

---

## The Promise

By the end of this module you will:

1. **Read the Nutanix licensing matrix in 60 seconds.** AOS subscription tiers, NCM tiers, what each includes, what falls below the line. The structure is consistent; the specific feature names shift between releases. Understand the structure, then verify the current details against the pricing guide.
2. **Build a complete BoM (Bill of Materials) for a customer scenario.** Hardware (nodes, network, cables), software (AOS subscription, NCM tier, additional features), services (implementation, training, migration), support (standard vs extended). The shape of a complete BoM is what BlueAlly SAs produce and what customers act on.
3. **Run a 5-year TCO model that holds up under CFO scrutiny.** Capex, opex, refresh cycles, migration costs, staff time, decommissioning. Show all the assumptions. Don't hide anything.
4. **Make the Broadcom comparison precisely.** VMware's post-Broadcom licensing reorganization (per-core subscription, larger minimum commitments, bundled tier structure) genuinely changed the economics for many customers. The comparison is real and BlueAlly SAs need to walk through it without overclaiming or oversimplifying.
5. **Talk to a CFO without flinching.** Capex vs opex preferences, multi-year subscription mechanics, depreciation interaction with the customer's accounting policy, when 3-year subscriptions make sense vs 5-year. If you can't have this conversation, the technical wins do not close.
6. **Recognize the hidden cost categories customers forget.** Migration parallel-running periods, training, third-party tooling that interacts with the platform, decommissioning costs for old gear, one-time professional services. These are real and they show up in customer satisfaction conversations 6 months in.

This is the module that turns technical wins into signed deals. After eight modules of platform depth, this is the dimension that gets things signed.

---

## Foundation: What You Already Know

You have priced VMware. The mental model:

- **Hypervisor licensing:** vSphere per-CPU (pre-Broadcom) or per-core subscription (post-Broadcom) bundled into Cloud Foundation tiers (VVF, VCF).
- **Storage licensing:** vSAN per-CPU (pre-Broadcom) or bundled into VCF (post-Broadcom).
- **Networking licensing:** NSX-T per-CPU or per-workload, separately licensed.
- **Management licensing:** Aria Suite separately, with its own per-CPU model historically.
- **Support contracts:** Production support, extended support, fees that scale with the licensing footprint.

Layer the hardware on top:

- **Server vendors:** Dell, HPE, Lenovo, Cisco. Refresh every 3-5 years.
- **Storage arrays:** NetApp, Pure, EMC, etc. Refresh on their own cycles.
- **Network gear:** Cisco, Arista, Juniper. Yet another refresh.
- **Cables, switches, rack power, cooling.** Easy to forget; real money.

Layer operations on top:

- **Power and cooling.** Datacenter space cost.
- **IT labor.** The team's time on operations, troubleshooting, vendor escalation.
- **Migration costs.** When something is replaced, the migration project.
- **Training.** Recurring as products change.

You have built BoMs and TCO models against this stack. You have probably been asked "show me the 5-year number" by a customer's CFO and produced a spreadsheet. The skill transfers cleanly to Nutanix; the specific cost categories are different but the methodology is the same.

> [!FROM-THE-SA-CHAIR]
> The single most common BlueAlly SA mistake on pricing conversations is producing a hardware-and-license-only quote and calling it "the BoM." Customers who sign that quote and then experience the unaccounted-for costs (migration, training, parallel-running, decommissioning) become unhappy customers who blame the platform when the real problem was incomplete scoping. A complete BoM names everything: hardware, software, services, support, training, migration, and the inevitable surprise category. Customers respect the honesty and budget appropriately.

---

## Core Content

### NCI Subscription Tiers: The Foundation (formerly AOS)

The platform itself is now licensed under **NCI (Nutanix Cloud Infrastructure)**. NCI replaces the legacy **AOS Pro / AOS Ultimate** SKU naming; **legacy AOS licenses are no longer available for new sale or renewal**, and existing AOS customers are being converted to NCI Pro / NCI Ultimate by their account teams. Both names will appear in the wild for a while: AOS in older customer contracts, current pricing tools, and customer vocabulary; NCI in net-new quotes and current Nutanix portal collateral. Internally to the platform, NCI = AOS at the version level; the rename is a packaging change, not a software change.

The current tier structure:

- **NCI Pro** (formerly AOS Pro). The foundational tier. Includes AHV (the hypervisor at no extra charge), DSF, Prism Element, baseline replication (Async), basic snapshots and backup, baseline network features. This is what most customers buy.
- **NCI Ultimate** (formerly AOS Ultimate). Adds advanced features: NearSync replication, Metro Availability, advanced data services, Flow Network Security (also available as a Security Add-On for NCI Pro on a per-usable-TiB model; see Module 6), additional storage features, advanced security.
- **NCI Compute (NCI-C)** also exists as a compute-only SKU for clusters that don't need DSF storage features (rare; mostly relevant for specific edge or compute-mostly scenarios).

NCI requires AOS 6.1.1 (LTS 6.5) or later, Prism Central 2022.4+, and NCC 4.5.0+ on the cluster. Customers running older AOS versions need to upgrade before converting from AOS to NCI licensing.

Licensing is **per-core**. The customer pays for the cores in their cluster, not for VM count. This is similar in shape to VMware's post-Broadcom per-core model.

**Multi-year subscription terms** (1, 3, or 5 years) are standard. Longer terms typically come with discount tiers. Customers who plan their compute footprint can save meaningfully on multi-year commitments.

> [!ON-THE-EXAM] **NCA · NCP-MCI**
> Tier structure is testable at a high level. Memorize: **NCI Pro** is the standard platform tier; **NCI Ultimate** adds NearSync, Metro, advanced features, and Flow Network Security. **NCM tiers** (Starter / Pro / Ultimate) are separately-licensed paid SKUs that add management capabilities on top of either NCI tier. Older exam material may still use the legacy "AOS Pro / Ultimate" naming; treat the two as functionally equivalent, but in current Nutanix collateral the term is NCI. Trap distractor: questions implying AHV requires a separate license fee (false; AHV is included with NCI / AOS at every tier).

### NCM Tiers: The Management Layer

**NCM (Nutanix Cloud Manager)** is the umbrella name for the multi-cluster management capabilities that sit on top of NCI. NCM is **separately licensed from NCI** (do not assume "NCM Starter is bundled with Prism Central" — it is not; basic Prism Central comes with NCI, and NCM tiers are paid add-ons). The tier structure:

- **NCM Starter.** A paid tier focused on infrastructure operations: monitoring, planning, rightsizing, basic Intelligent Operations, low-code automation.
- **NCM Pro.** Adds deeper Intelligent Operations (anomaly detection, what-if planning, runway analysis), advanced reporting, more aggressive alert correlation. Aria Operations equivalent functionality.
- **NCM Ultimate.** Adds Self-Service (formerly Calm) blueprint-driven provisioning, X-Play playbook automation, cost governance, advanced multi-cloud features. Aria Automation equivalent functionality.

Nutanix also packages **NCP (Nutanix Cloud Platform)** bundles that combine NCI and NCM at matching tiers (NCP Starter = NCI Pro + NCM Pro; NCP Pro = NCI Ultimate + NCM Pro; NCP Ultimate = NCI Ultimate + NCM Ultimate). Customers who want everything in one SKU often choose NCP rather than buying NCI and NCM separately.

For most enterprise customers, **NCM Pro is the right baseline** (provides the analytics that justify the investment). NCM Ultimate is for customers who plan to drive serious self-service or multi-cloud governance. **What is included with basic NCI without any NCM:** multi-cluster Prism Central management, Categories, Projects, baseline reporting and dashboards, RBAC, identity integration, the v4 REST API. That is the floor; NCM tiers add capability on top.

> [!FAMILIAR]
> The NCM tier structure maps roughly onto the VMware Aria Suite tiers. Aria Operations (formerly vROps) maps to NCM Pro's Intelligent Operations. Aria Automation (formerly vRA) maps to NCM Ultimate's Self-Service. Aria Suite Lifecycle (the bundle) is similar in spirit to NCM as the umbrella product. Customers who priced Aria Suite have a mental model that transfers.

### Hardware Sourcing: NX, OEM, or Software-Only

Nutanix software runs on three categories of hardware:

**1. Nutanix-branded NX appliances.** Hardware sold by Nutanix directly (manufactured by Super Micro under the Nutanix brand). Single-vendor support: Nutanix is the throat to choke for hardware and software. Tightly validated. Customers who want one vendor relationship typically choose NX.

**2. OEM partner hardware.** Dell EMC XC, Lenovo HX, HPE DX, Cisco UCS. The customer buys hardware from their existing server vendor (Dell, Lenovo, HPE, Cisco) with Nutanix software pre-installed and validated. The customer's server-vendor relationship continues; Nutanix provides software and joint support. This is the most common path for customers with established server-vendor preferences.

**3. Software-only on HCIR (Hyperconverged Infrastructure-Ready) commodity hardware.** Customer buys hardware separately from the HCL (Hardware Compatibility List) and licenses Nutanix software. Most flexible, requires more diligence on the hardware decision, more support boundaries to manage.

| Sourcing Option | Single-vendor support | Customer's existing relationship | Cost flexibility |
|---|---|---|---|
| NX | Yes (Nutanix) | New relationship if not already in place | Standardized |
| OEM (Dell, Lenovo, HPE, Cisco) | Joint Nutanix + OEM | Preserves existing vendor relationship | Standard with negotiated discounts |
| Software-only / HCIR | Multi-vendor (customer manages) | Maximum flexibility | Most flexible; customer handles hardware sourcing |

The choice depends on the customer's organizational preferences. There is no single "right" answer. A customer with deep Dell relationships typically chooses Dell XC. A customer who wants minimum complexity chooses NX. A customer with custom hardware preferences (or unusual workload requirements) chooses software-only.

> [!FROM-THE-SA-CHAIR]
> Always ask the hardware-sourcing question early in discovery. *"Do you have an existing server-vendor relationship you'd like to preserve, or are you open to NX appliances or commodity hardware?"* The answer significantly affects the BoM and the customer's procurement workflow. BlueAlly's relationships with Dell, Lenovo, HPE, and Cisco mean the OEM-partner path is typically smooth; verify the customer's current contract status with their preferred vendor before pricing.

> [!DIFFERENT]
> Unlike most enterprise infrastructure, Nutanix's hardware sourcing flexibility is real. The customer is not locked into a specific hardware vendor by the platform choice. This is operationally important: it preserves the customer's negotiating leverage with hardware vendors and lets them adapt the platform to their existing procurement patterns. Compare to platforms that bundle hardware tightly (some hyperconverged vendors require specific appliances). Nutanix's openness is a real differentiator.

---

### Diagram: AOS and NCM Tier Feature Matrix

**id:** `tier-feature-matrix`
**type:** matrix
**caption:** Feature mapping across AOS and NCM tiers. Useful for quoting against customer requirements.
**exam_relevance:** [NCA, NCP-MCI, sales-relevant]
**whiteboard_ready:** false

**Elements (matrix structure):**
- Rows: feature categories
  - Hypervisor (AHV)
  - DSF core (RF, snapshots, async replication)
  - NearSync replication
  - Metro Availability
  - Compression / dedup baseline
  - Erasure Coding (advanced configurations)
  - Prism Element
  - Prism Central baseline
  - Categories / Projects
  - Intelligent Operations (capacity analytics, anomaly detection)
  - Self-Service (blueprints, Calm)
  - X-Play playbook automation
  - Cost Governance
  - Files (separate licensing typically)
  - Objects (separate licensing typically)
  - Volumes (typically included in AOS)
  - Flow Network Security
  - Flow Virtual Networking

- Columns:
  - NCI Pro (formerly AOS Pro)
  - NCI Ultimate (formerly AOS Ultimate)
  - NCM Starter (paid)
  - NCM Pro
  - NCM Ultimate
  - NCP bundles (Starter / Pro / Ultimate, combine NCI + NCM)
  - Add-on (Files, Objects, Security Add-On for Flow + DARE, etc.)

- Cells: checkmark / dot / blank to indicate inclusion

**Annotations:**
- "Tier contents shift between AOS versions. This matrix is structural; verify specifics against the current pricing guide."
- "AHV is included with AOS. Always."
- "NCM Pro is the typical analytics tier for enterprise deployments."
- "Files and Objects are typically separately licensed; verify in the current price book."

**Why this diagram exists:** Customers ask "what's included in what?" constantly. A clean matrix answers it. BlueAlly SAs use this internally for first-pass quoting; verify specific feature inclusion against the current Nutanix price book before producing a customer-facing quote.

---

### The Cycle, Frame Two: The Broadcom Math

The post-Broadcom VMware licensing reorganization is the elephant in the 2026 enterprise infrastructure conversation. The honest summary:

- **Per-core subscription.** VMware moved from per-CPU (per-socket) to per-core licensing. Combined with minimum-cores-per-CPU floors (often 16 cores per CPU minimum, with rounding), this materially increased licensing for many configurations.
- **Bundled tier structure.** vSphere alone became harder to buy. VVF (VMware vSphere Foundation) and VCF (VMware Cloud Foundation) bundle vSphere with vSAN, sometimes NSX, and Aria. Customers who only wanted the hypervisor often found themselves paying for storage and management features they were not using.
- **Reduced perpetual licensing.** The transition pushed customers toward subscription. Existing perpetual licenses were honored but new perpetual was constrained or unavailable in some markets.
- **Larger minimum commitments.** Multi-year subscriptions became more common; the entry point for new customers became larger.
- **Reseller channel changes.** The reseller landscape simplified, with fewer authorized partners and larger minimum customer sizes for some programs.

For specific customers, the impact ranges from "modest increase, manageable" to "doubled licensing, re-evaluating." The variability depends on:
- Configuration density (many cores per VM = fewer cores per VM-cost; few cores per VM = more cores per VM-cost)
- Whether vSAN was already licensed (if yes, the bundling is closer to neutral)
- Whether Aria was already licensed (similar logic)
- Multi-year commitment status

**The Nutanix comparison is not "always cheaper."** It is "the comparison favors Nutanix more often than it did pre-Broadcom, especially for customers who are paying for VVF/VCF features they don't fully use."

When making the comparison:
- Get the customer's actual VMware quote (or recent renewal). Don't work from rumors.
- Build the equivalent Nutanix BoM honestly: AOS Pro plus the features the customer needs, equivalent NCM tier, hardware on the appropriate sourcing model.
- Compare 5-year TCO with all categories included. Don't compare "AOS subscription vs vSphere subscription" alone; that's not the actual decision the customer is making.
- Acknowledge what VMware does well and what customers might give up. Be honest.

> [!FROM-THE-SA-CHAIR]
> The Broadcom conversation requires discipline. The customer has heard the price-increase stories from peers and probably feels frustrated about it. The wrong move is to pile on with anti-Broadcom rhetoric; you sound like a vendor exploiting the moment. The right move is professional and specific: *"The licensing reorganization changed the economics for many customers. Whether it changes them favorably for Nutanix in your specific case depends on your configuration, your feature usage, and your refresh timing. Let me build the apples-to-apples comparison for your environment. We may find Nutanix is meaningfully cheaper; we may find the gap is smaller than you've heard. Either way, you'll know the real number."* That sentence wins customer trust by treating the comparison as factual rather than emotional.

---

### Diagram: Broadcom vs Nutanix 5-Year TCO Comparison Methodology

**id:** `broadcom-vs-nutanix-comparison`
**type:** comparison
**caption:** A structured methodology for the comparison. Walk every category. Don't skip anything.
**exam_relevance:** [sales-relevant]
**whiteboard_ready:** true

**Elements:**
- Two columns: VMware (post-Broadcom) and Nutanix
- Rows of cost categories:
  - Hypervisor licensing (per-core subscription)
  - Storage software licensing (vSAN bundled into VCF / DSF in AOS)
  - Networking software (NSX-T separate / Flow in NCM Pro)
  - Management software (Aria Suite / NCM tier)
  - Hardware (compute, storage, network) over 5 years including refresh
  - Support contracts
  - Power and cooling (similar for similar hardware footprint)
  - Datacenter space (similar; consolidation savings if applicable)
  - Migration costs (one-time, applies to whichever direction the migration goes)
  - Staff training (smaller for incumbent platform; larger for new platform)
  - Operational labor (5 years)
  - Decommissioning (one-time, applies to old gear)

- For each row, indicate:
  - VMware column: typical cost component
  - Nutanix column: typical cost component
  - Net difference: estimate range (e.g., "Nutanix typically 20-40% lower for typical configs; verify with actual customer numbers")

- Bottom: "Total 5-year TCO (real customer numbers required)"

**Annotations:**
- "Don't compare line items in isolation. Compare totals."
- "Migration costs apply once; recurring costs apply for 5 years."
- "Some categories are similar (power/cooling). Don't pretend they aren't."
- "The customer's actual VMware quote is the starting point. Without that, you're estimating."

**Why this diagram exists:** This is the methodology diagram. BlueAlly SAs use it as a checklist when building TCO models. The customer's CFO can follow the structure. Whiteboard it for executive conversations.

---

### Building a Complete BoM

A complete BoM has these sections. Hand a customer anything that's missing categories and you've underscoped the project.

**1. Hardware:**
- Nodes: count, model, CPU (cores), RAM, NVMe/SSD storage, network NICs (10/25/100 GbE)
- Network switching (if BlueAlly is providing): top-of-rack, distribution
- Cables, transceivers, power cords, rack rails (often ignored; real money on a multi-rack deployment)
- Sourcing source: NX, OEM (specific OEM), or software-only with customer hardware

**2. Software:**
- AOS subscription tier and term (typically 3 or 5 year)
- Total cores covered
- NCM tier and term
- Files licensing (if needed; per-TB or per-FSVM model depending on packaging)
- Objects licensing (if needed; per-TB capacity)
- Volumes (typically included in AOS)
- Flow licensing per applicable tier (often included with NCM Pro)
- Any add-ons (NDB / Era for database lifecycle, NKE for Kubernetes management, etc.)

**3. Services (BlueAlly):**
- Implementation: project hours for cluster deployment, configuration, integration with customer's environment (AD, network, storage, monitoring)
- Migration assistance: planning, dry runs, cutover support
- Training: BlueAlly-led sessions for the customer's team
- Optional: managed services post-deployment

**4. Support:**
- Standard production support (24x7 included with subscription)
- Extended support tiers (faster SLAs, dedicated TAM, etc.) if applicable
- Hardware support (handled by Nutanix for NX, by OEM for OEM, by customer for software-only)

**5. One-time items:**
- Network upgrades if the customer's network requires changes (e.g., 10 GbE to 25/100 GbE switching)
- Datacenter prep (rack space, power, cooling adjustments)
- Migration tooling licensing if needed for specific complex migrations
- Decommissioning costs for replaced gear (often the customer's responsibility but worth naming)

**6. Recurring items:**
- Subscription renewals (years 4, 5 if not on multi-year)
- Support renewals
- Operational tooling subscriptions (monitoring, backup, etc., if integrated)

A complete BoM names every item. The customer can then categorize as capex, opex, one-time, recurring, etc. according to their accounting policy.

> [!ON-THE-EXAM] **NCP-MCI · sales-relevant**
> BoM components are not commonly tested at deep specificity but the awareness that subscription, services, hardware, support, and migration are separate categories is fundamental.

### Capex vs Opex: The CFO Frame

Customers' accounting preferences vary:

- **Capex (capital expenditure)** is typically hardware. Depreciated over a useful life (3-5 years typically). Sits on the balance sheet. Some companies prefer this because it amortizes over time and matches the cash flow story their auditors expect.
- **Opex (operating expenditure)** is typically subscriptions and services. Expensed in the period incurred. Income-statement impact. Some companies prefer this for the simpler accounting and the avoidance of capex approval cycles.

Nutanix subscriptions are typically opex. Hardware is typically capex. Multi-year subscription commitments may have specific accounting treatments depending on the customer's auditor.

**Common customer preferences:**
- **Public companies and growth-mode SaaS companies:** opex, with multi-year commitments expensed appropriately.
- **Capital-rich enterprises:** capex preference for hardware, opex for software.
- **Government / regulated:** depends on agency; capex often preferred for certain budget cycles.

The right SA-chair move is to ask early: *"How does your finance team prefer to account for infrastructure investments? Are you typically capex-heavy on hardware, opex on software, or a different model?"* The answer changes how you structure the proposal.

### 5-Year TCO Modeling: The Methodology

A real TCO model has these properties:

1. **Time horizon: 5 years standard.** Some customers want 3 or 7; 5 is the common default.
2. **All cost categories present.** Hardware, software, services, support, training, migration, operations, decommissioning. Don't skip any.
3. **Year-by-year.** Show what cost falls in which year. Useful for cash-flow analysis.
4. **Assumptions called out.** What did you assume about growth, refresh cycles, headcount, training? List the assumptions; the customer's CFO will check them.
5. **Sensitivity analysis.** What happens if growth is higher than assumed? If migration takes longer? Show the ranges, not just point estimates.
6. **Comparison-ready.** If the customer is comparing VMware vs Nutanix, the categories should align so the customer can compare apples to apples.

Common categories per year:

| Year | Capex (hardware) | Software subscription | Services (one-time) | Support (annual) | Training | Migration | Operations |
|---|---|---|---|---|---|---|---|
| Year 1 | Cluster purchase | Year 1 sub | Implementation | Year 1 support | Initial training | Migration project | Year 1 ops labor |
| Year 2 | (refresh deferred) | Year 2 sub | (none) | Year 2 support | Refresher | (complete) | Year 2 ops |
| Year 3 | (refresh deferred) | Year 3 sub | (none) | Year 3 support | Light | (n/a) | Year 3 ops |
| Year 4 | (refresh deferred) | Year 4 sub or renewal | (none) | Year 4 support | Light | (n/a) | Year 4 ops |
| Year 5 | (refresh approaching) | Year 5 sub | (none) | Year 5 support | Light | (n/a) | Year 5 ops |
| Year 6 (planning) | New cluster planning | n/a | Refresh project | n/a | n/a | n/a | n/a |

The customer's actual TCO depends on their actual growth, their actual refresh policy, and their actual labor model. A real model uses customer-specific numbers.

---

### Diagram: TCO Components Over Time

**id:** `tco-components-over-time`
**type:** stacked-bar
**caption:** A 5-year stacked TCO view showing cost composition year by year. The shape of the cost is informative beyond the total.
**exam_relevance:** [sales-relevant]
**whiteboard_ready:** false

**Elements:**
- X-axis: Year 1 through Year 5
- Y-axis: total cost
- Stacked bars per year, with components:
  - Hardware capex (Year 1 large, subsequent years zero or small for additions)
  - Software subscription (relatively flat across years)
  - Support (relatively flat)
  - Services (Year 1 spike for implementation; Year 0 or 1 for migration)
  - Training (Year 1 spike, smaller Year 2-5)
  - Operations labor (relatively flat across years)
  - Decommissioning (one-time, in transition year)

- Side annotations: total over 5 years; cost per VM-year; cost per TB-year (useful normalization metrics)

**Annotations:**
- "Year 1 is the heaviest year due to capex and one-time services. Years 2-5 are subscription + support + ops."
- "When the customer refreshes hardware in Year 5-6, the capex spike repeats."
- "Subscription customers have a more predictable year-by-year pattern than capex-heavy hardware customers."

**Why this diagram exists:** Customers' CFOs want to see the cash-flow shape, not just the total. This diagram makes year-by-year cost composition visible. Use it in customer financial conversations.

---

### Hidden Costs Customers Forget

A complete TCO calls these out. Skip them and the customer gets surprised six months in.

1. **Migration parallel-running periods.** During migration, the customer often runs old and new platforms simultaneously. Both consume power, space, support, and team time. Plan for 1-3 months of parallel for typical migrations; longer for complex environments.

2. **Training that recurs.** Customers often budget initial training and forget recurring training as products evolve. Plan for some training budget every year.

3. **Third-party tooling integration.** Backup tools, monitoring tools, configuration management tools, ITSM (ServiceNow), CMDB integration. Each may require integration work; some require additional licensing.

4. **Decommissioning of old gear.** Removing old arrays, filers, and servers takes time, and the customer typically owes lease or maintenance fees on old contracts that don't expire on the migration timeline.

5. **Bandwidth costs for cloud-DR (if NC2).** Cloud egress fees on failback are real. Plan for them.

6. **Network upgrades if the new platform requires them.** Moving from 10 GbE to 25/100 GbE switching costs money in switches and cables. Plan for it.

7. **Datacenter infrastructure changes.** Power and cooling upgrades, rack space changes, KVM and console infrastructure for the new gear.

8. **Audit and compliance work.** Some compliance frameworks require validation when infrastructure changes. Plan for audit-team time and possibly third-party assessments.

9. **Staff role changes.** Existing staff may need new training; some roles may shift; some hiring may be required (e.g., a Nutanix specialist for a customer with no prior Nutanix experience).

The professional honest BoM names all of these as line items, even if they are estimates. The customer's CFO appreciates the honesty and budgets accordingly. Customers who have been blindsided by these costs from a previous vendor are particularly receptive to seeing them upfront.

---

### Multi-Year Subscription Mechanics

Multi-year terms come with discounts and trade-offs:

- **1-year:** Most flexibility, highest per-year cost, easy to walk away. Useful for evaluation periods or organizations with budget uncertainty.
- **3-year:** Common middle ground. Discount typically meaningful (5-15% off 1-year per-year cost). Reasonable commitment level.
- **5-year:** Largest discounts (10-25%+). Aligns with hardware refresh cycle. Fits customers with predictable footprint.

**Consider before recommending multi-year:**

- **Growth uncertainty.** If the customer plans to add many cores in year 2-3, the 5-year-at-current-cores commitment may need true-up provisions.
- **M&A activity.** If acquisition or divestiture is on the horizon, lock-in is a risk.
- **Budget cycle alignment.** Some customers have firm 3-year capex/opex cycles; others have annual reviews.

**Expansion provisions** (often called "true-up" or "ramp" provisions) let the customer add cores during the term at agreed pricing. Always negotiate these for multi-year deals; they protect both parties.

> [!FROM-THE-SA-CHAIR]
> Push for multi-year only when it actually serves the customer. A 5-year deal that the customer outgrows in year 2 is worse than a 3-year deal sized appropriately. The right framing: *"Multi-year saves you money if your footprint is stable. Let's size for your 18-24 month plan, then decide on term length based on your confidence in that plan and your growth trajectory."* That respects the customer's reality and avoids the painful renegotiation conversation in year 2.

---

## Lab Exercise: Build a Representative TCO Model

> [!LAB] **Time:** ~3 hours · **Platform:** Spreadsheet (Excel, Google Sheets, or equivalent)

This exercise builds a complete 5-year TCO model for a representative customer scenario. The numbers are illustrative; the methodology is the takeaway.

**The scenario:**
A customer has 8 ESXi hosts, ~250 VMs, NetApp filer (200 TB), Pure FlashArray for iSCSI (50 TB), Data Domain backup target (100 TB). Refresh is 12 months out. Annual run-rate: roughly $400K-500K across hardware, software subscriptions, and support for the existing environment.

**Build TCO models for two options:**

**Option A: VMware refresh (post-Broadcom).** New ESXi cluster on Cloud Foundation, refresh storage arrays, refresh backup target. Baseline.

**Option B: Nutanix consolidation.** 12-node cluster running compute + Files + Objects + Volumes. Decommission the three storage tiers over 6-9 months.

**Steps:**

1. **Build the BoM template.** Categories: hardware, software, services, support, training, migration, operations, decommissioning. Year columns: Year 1 through Year 5.

2. **Populate Option A (VMware refresh).** Use representative ranges (per-core subscription estimate, hardware estimate, support estimate). Document assumptions clearly. Sum to 5-year total.

3. **Populate Option B (Nutanix).** Same categories. AOS Pro per-core, NCM Pro, Files licensing, Objects licensing, hardware via OEM (Dell XC or Lenovo HX), services for migration. Document assumptions.

4. **Add recurring operational labor.** Estimate FTE hours saved by consolidation (3 vendors to 1, fewer firmware upgrade cycles, single management plane). Multiply by loaded labor cost.

5. **Add migration costs to Option B.** Implementation services, parallel-running for 3 months, training, decommissioning of old gear.

6. **Calculate 5-year totals.** Compare Options A and B.

7. **Sensitivity analysis.** Re-run with growth assumptions ±20%, migration time ±50%, labor cost ±15%. See how the comparison changes.

8. **Document the assumptions explicitly.** A model without stated assumptions is an unaudible model. List every assumption with the value used.

9. **Build a one-page summary.** What's the 5-year delta? What are the top 3 drivers of the difference? What are the top 3 risks?

**What this teaches you:**
- How to build a model that holds up under CFO scrutiny.
- The categories that matter and how to estimate them when specific quotes aren't yet available.
- The importance of assumptions documentation.
- Sensitivity analysis as a skill.

**Customer-demo angle:** The one-page summary from step 9 is the customer-facing deliverable. Practice producing it cleanly. CFOs respect concise, well-documented analyses.

---

## Practice Questions

Twelve questions. Six knowledge MCQ, four scenario MCQ, two NCX-style design questions. Read each, answer in your head, then read the explanation.

---

**Q1.** Which of the following is correct about AHV licensing?
**Cert relevance:** NCA · NCP-MCI

A) AHV requires a separate per-CPU license fee
B) AHV is included with AOS subscription at no additional licensing cost
C) AHV requires VMware vSphere licensing as a base
D) AHV is licensed only at the AOS Ultimate tier

**Answer:** B

**Why this answer:** AHV is included with AOS. There is no separate hypervisor licensing fee. This is one of the durable Nutanix economics points compared to VMware's separate hypervisor licensing.

**Why not the others:**
- A) AHV is included; no separate fee.
- C) AHV is its own hypervisor; it does not require vSphere.
- D) AHV is included at all AOS tiers.

**The trap:** Customers and test-takers who think in VMware-licensing models may default to "the hypervisor must be licensed separately." Memorize: AHV is included with AOS.

---

**Q2.** Which feature requires NCM Ultimate licensing?
**Cert relevance:** NCA · NCP-MCI · sales-relevant

A) Categories and Projects
B) Multi-cluster Prism Central management
C) Self-Service blueprint automation (formerly Calm)
D) Async replication

**Answer:** C

**Why this answer:** Self-Service / Calm requires NCM Ultimate (or equivalent licensing tier). It is the highest-tier NCM feature in the standard packaging.

**Why not the others:**
- A) Categories and Projects are baseline Prism Central features.
- B) Multi-cluster management is the core PC value, included.
- D) Async replication is part of AOS, not gated at NCM Ultimate.

**The trap:** Customers and test-takers who hear "Prism Central includes everything" miss the tier gating on advanced features.

---

**Q3.** Which statement about Nutanix hardware sourcing is correct?
**Cert relevance:** NCP-MCI · sales-relevant

A) Nutanix software runs only on Nutanix-branded NX appliances
B) Customers can run Nutanix on NX appliances, on OEM hardware (Dell XC, Lenovo HX, HPE DX, Cisco UCS), or on commodity hardware that meets the HCL (software-only deployment)
C) Nutanix supports only Dell hardware for non-NX deployments
D) Software-only deployments are unsupported

**Answer:** B

**Why this answer:** Nutanix's hardware sourcing flexibility includes NX (Nutanix-branded), OEM partners (Dell XC, Lenovo HX, HPE DX, Cisco UCS), and software-only on HCL-compliant commodity hardware. Customer choice based on relationship and preference.

**Why not the others:**
- A) Multiple hardware paths exist.
- C) Multiple OEM options.
- D) Software-only is fully supported.

**The trap:** A reflects a limited mental model that may have been true years ago. The current state is multi-vendor flexibility.

---

**Q4.** A customer has a 5-year subscription term with growth provisions. The first year, they license 256 cores; in year 3, they grow to 384 cores. What is the typical billing structure?
**Cert relevance:** sales-relevant

A) The customer pays a penalty for exceeding the original commitment
B) The customer trues up to 384 cores in year 3 at the agreed pricing per core, with the additional cores billed for the remaining contract term
C) The customer must wait until contract expiration to add cores
D) The customer is forced to renegotiate the entire contract

**Answer:** B

**Why this answer:** Standard multi-year contracts include true-up provisions that let customers add cores during the term at the agreed pricing. The new cores are billed for the remaining term.

**Why not the others:**
- A) Penalties are not the typical mechanism; true-up at agreed pricing is.
- C) Customers add cores during the term routinely.
- D) Renegotiation may happen at term end; mid-term true-up is normal.

**The trap:** A is the worst-case mental model. The actual contractual mechanic is true-up at agreed pricing.

---

**Q5.** Which of the following is the most accurate description of a complete BoM (Bill of Materials)?
**Cert relevance:** sales-relevant

A) Hardware list with prices
B) Hardware + software subscription
C) Hardware, software subscription, services (implementation, migration, training), support, one-time items, and recurring items
D) The customer's existing infrastructure inventory

**Answer:** C

**Why this answer:** A complete BoM names every category. Anything less leaves the customer underbudgeted and the project at risk.

**Why not the others:**
- A) Hardware-only is dramatically incomplete.
- B) Adds software but still misses services, support, migration, training, etc.
- D) That is a customer's current-state inventory, not a forward-looking BoM.

**The trap:** A and B are tempting because they are simpler. The discipline is to name everything; customers respect the rigor.

---

**Q6.** What is the typical structure of a 5-year TCO comparison between competing platforms?
**Cert relevance:** sales-relevant

A) Compare the highest-cost line item from each platform
B) Compare 5-year totals across all categories (hardware, software, services, support, operations, migration, decommissioning) with stated assumptions and sensitivity analysis
C) Compare year-1 capex only
D) Compare subscription pricing only

**Answer:** B

**Why this answer:** Real TCO comparisons require all categories, all years, stated assumptions, and sensitivity analysis. Customer CFOs evaluate models on this standard.

**Why not the others:**
- A) Cherry-picking line items doesn't represent total cost.
- C) Year 1 alone misses the durable cost story.
- D) Subscription alone is one cost category among many.

**The trap:** A, C, and D are common shortcuts that produce misleading results. The discipline is full-category, multi-year, with assumptions.

---

**Q7.** A customer's CFO asks: "Tell me how I should think about Nutanix subscription versus VMware Cloud Foundation subscription." What is the strongest SA response?
**Cert relevance:** sales-relevant

A) "Nutanix is always cheaper than VMware."
B) "It depends on your specific configuration. The comparison favors Nutanix more often than it did pre-Broadcom for customers paying for VVF/VCF features they don't fully use, but the gap depends on your core count, your feature usage, and your refresh timing. Let me build the apples-to-apples comparison for your environment over 5 years with all categories included."
C) "VMware is too complicated to understand."
D) "Nutanix is for customers who can't afford VMware."

**Answer:** B

**Why this answer:** Honest, specific, ends with a concrete proposal. Acknowledges variability, names the relevant variables, and offers to do the work.

**Why not the others:**
- A) Untrue and overconfident. CFOs see through this.
- C) Insulting. Costs you the room.
- D) Repugnant framing.

**The trap:** A is the seductive defensive answer. Honesty is the durable SA-chair posture.

---

**Q8.** A customer wants to consolidate their NetApp filer ($75K/year support), Data Domain ($60K/year), and Pure FlashArray ($55K/year) onto Nutanix. They are reviewing the Nutanix BoM. Which approach is correct?
**Cert relevance:** sales-relevant

A) Quote only the Nutanix hardware and AOS subscription; let the customer figure out the rest
B) Build a complete BoM including hardware, AOS subscription, Files licensing for filer replacement, Objects licensing for backup-target replacement, NCM tier, services for migration of all three storage tiers, parallel-running cost during migration, and decommissioning estimates for the old gear
C) Pretend the migration is free
D) Tell the customer their existing storage was overpriced and Nutanix will be much cheaper

**Answer:** B

**Why this answer:** This is the complete-BoM discipline. The customer needs to see all the cost categories so they can compare to the run-rate they are eliminating.

**Why not the others:**
- A) Underscoping causes customer pain six months in.
- C) Migration is real work with real cost.
- D) Comparative-trash-talk is unprofessional.

**The trap:** A and C are easier; B is correct. BlueAlly's reputation depends on the discipline.

---

**Q9.** Which of the following best describes the difference between an OEM partner deployment (e.g., Dell XC) and a software-only / HCIR deployment?
**Cert relevance:** sales-relevant

A) OEM is more expensive; software-only is always cheaper
B) OEM provides Nutanix software pre-installed on validated hardware from a major server vendor with joint Nutanix + OEM support; software-only means the customer buys hardware separately from the HCL and licenses Nutanix software, with multi-vendor support boundaries
C) OEM and software-only are the same
D) Software-only is unsupported

**Answer:** B

**Why this answer:** This is the distinction. OEM = pre-validated, joint support. Software-only = customer-sourced hardware, multi-vendor support model. Both are fully supported approaches; the choice depends on customer preferences.

**Why not the others:**
- A) Cost depends on configuration; not always cheaper one way or the other.
- C) Distinct sourcing models.
- D) Software-only is supported.

**The trap:** A is a common assumption that doesn't always hold. The right answer is "depends on the specific deal."

---

**Q10.** A BlueAlly customer is comparing 3-year vs 5-year subscription terms. Their growth forecast: 256 cores year 1, 320 cores year 2, 384 cores year 3, 384 cores year 4, 384 cores year 5. What's the right SA recommendation?
**Cert relevance:** sales-relevant

A) Always recommend 5-year for the discount
B) Recommend 5-year with growth provisions / true-up clauses, sized for year-3 stability, with an annual review cadence to monitor whether the assumptions hold
C) Recommend 1-year to avoid commitment
D) Recommend 3-year and then renew

**Answer:** B

**Why this answer:** The growth forecast suggests stability after year 3. A 5-year term with growth provisions captures the discount while protecting the customer from being locked in at year-1 sizing for the entire term. The annual review cadence catches any deviation early.

**Why not the others:**
- A) Without growth provisions, the customer is stuck at year-1 sizing.
- C) 1-year forfeits the discount unnecessarily for a stable trajectory.
- D) 3-year is reasonable but 5-year with provisions captures more value.

**The trap:** A and C are extremes; B is the disciplined SA-chair answer.

---

**Q11.** *(NCX-style design question, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / design discussion. Write your reasoning. There is no single correct answer; there are stronger and weaker frames.

**The scenario:**
A customer's environment:
- 16 ESXi hosts (Dell PowerEdge R750), ~500 VMs
- Mix of compute-balanced and storage-heavy workloads
- vSphere Cloud Foundation subscription (per-core, recently renewed for 1 year)
- vSAN included in VCF
- NSX-T separately licensed (per-CPU, 2 years remaining on contract)
- NetApp filer ($120K/year support)
- Veeam backup with Data Domain target ($95K/year support combined)
- 1 TB of Aria Operations data, 6 months of operational history

The customer asks BlueAlly to build a 5-year TCO comparison: continue with VMware refresh in year 2, or migrate to Nutanix on Dell XC hardware.

**The challenge:**
Walk through your BoM and TCO methodology. Cover both options. Identify what you still need to know. Recommend a path.

**A strong answer covers:**
- **Discovery first:** Get the customer's actual VCF renewal quote, NSX-T contract details, current NetApp/DD/Veeam refresh costs and timing, growth projections, and accounting policy preference (capex vs opex).
- **Option A: VMware refresh BoM and 5-year TCO:**
  - Hardware: refresh of 16-node Dell cluster in year 2-3 + storage refresh
  - Software: VCF renewal at current pricing + NSX-T renewal in year 2 + Aria continuation
  - Storage tier: NetApp refresh, Data Domain renewal or refresh
  - Services: minimal (continuation of current vendor relationships)
  - Operational labor: continuation of current model (4 vendors, 4 management UIs, 4 refresh cycles)
- **Option B: Nutanix on Dell XC consolidation:**
  - Hardware: 16-20 node Dell XC cluster (sized for compute + Files + Objects + Volumes consolidation)
  - Software: AOS Pro per-core, NCM Pro, Files licensing for NetApp replacement, Objects licensing for Data Domain replacement
  - Networking: Flow Network Security (replaces NSX-T microsegmentation; for advanced NSX-T routing patterns, evaluate need)
  - Services: migration project (12-15 months for full consolidation), parallel-running periods, training, decommissioning
  - One-time costs: NetApp/DD/Pure decommissioning costs, possible early-termination discussions on NSX-T contract
  - Operational labor: simplified model (1 platform, 1 management plane, 1 refresh cycle)
- **5-year totals (illustrative methodology):**
  - Option A: $X (sum of hardware + software + storage refresh + support + ops over 5 years)
  - Option B: $Y (sum of new cluster + AOS subscription + NCM + Files/Objects + services + reduced ops + decommissioning)
  - Net delta: typically $200K-800K over 5 years depending on configuration; verify with actual customer numbers
- **Sensitivity factors:**
  - If NSX-T routing complexity is high, the Flow gap may favor staying with NSX-T-on-VMware or NSX-T-on-Nutanix-on-ESXi pattern
  - If NetApp ONTAP-specific workflows are critical, Files migration cost is higher
  - If growth is +30% over 5 years, both options scale similarly with capex addition
- **Risk factors:**
  - Migration risk: 12-15 months of project work, with parallel-running and decommissioning
  - Vendor consolidation risk: more dependence on Nutanix (offset by Dell hardware relationship preserved)
  - Skills risk: team needs Nutanix training; offset by AHV's similarity to vSphere operationally
- **Recommendation framework:**
  - If TCO delta is $300K+ over 5 years and NSX-T/NetApp complexity is moderate: recommend Option B with phased migration
  - If TCO delta is small and incumbent complexity is high: recommend phased evaluation, maybe migrating subset of workloads to Nutanix while keeping incumbent for complex use cases
  - If customer's CFO heavily prefers opex: subscription-based Nutanix model fits better
  - If customer's CFO heavily prefers capex stability: consider ramifications of subscription model
- **What you still need to know:**
  - Customer's actual VCF renewal pricing
  - NSX-T contract remaining term and termination provisions
  - NetApp ONTAP-specific workflow inventory
  - Customer's growth projection
  - Customer's accounting policy preference
  - Specific compliance frameworks (PCI, HIPAA, etc.) that affect either choice

**A weak answer misses:**
- Skipping discovery and producing TCO with assumed numbers
- Comparing VCF subscription to AOS subscription without including hardware, services, support, operations
- Forgetting NSX-T contract remaining term (early termination has cost)
- Not naming the migration risk
- Producing a single-point estimate without sensitivity

**Why this question matters for NCX:** Real customer comparisons require complete BoMs and honest TCO methodology. NCX panels evaluate whether you can build a financially-credible analysis, name what you don't know, and produce an actionable recommendation.

---

**Q12.** *(NCX-style architectural defense, open-ended)*
**Cert relevance:** NCX-MCI prep · sales-relevant
**Format:** Short-answer / financial defense. Write your response.

**The scenario:**
You are in front of a customer's CFO. He has read the Nutanix proposal. He says:

> *"The hardware sticker price on this Nutanix proposal is roughly the same as VMware's. I save some money on the VCF subscription versus AOS subscription, sure, but only marginally. And then you've added a six-figure 'professional services' line for the migration, which is real cash. Why would I take on the disruption when the math is roughly the same?"*

**The challenge:**
Respond. He has done some math. He has an unfavorable framing. Address it.

**A strong answer covers:**
- **Acknowledge the math is more nuanced than sticker comparison.** "You're right that hardware-and-license sticker isn't dramatically different. The story is in the categories beyond hardware-and-license."
- **Walk through the categories he hasn't yet:**
  - **Storage consolidation:** "Your current environment has separate storage tiers (NetApp, Data Domain, Pure). On Nutanix that's consolidated. The storage-tier line items disappear from your run-rate. Over 5 years that's $X in support, plus a $Y refresh you don't have to do."
  - **Operational simplification:** "Four vendor relationships consolidate to two (Nutanix software, Dell hardware). Your team's time on firmware compatibility, vendor escalations, and quarterly upgrade dances reduces. We estimate $Z in labor savings over 5 years; it's not always quantifiable up front, but ask your team to estimate the time they spend coordinating across the four vendors today."
  - **Refresh cycles:** "Your NetApp refreshes in year X, your Data Domain in year Y, your Pure in year Z. On Nutanix, they all refresh together. Over 5 years that's one refresh cycle instead of three."
  - **Subscription shape:** "Your VCF subscription is 80%+ of your annual run-rate. On Nutanix the same shape exists but the licensing model is different. Look at year-by-year, not just totals."
- **Address the disruption concern directly:**
  - "Migration is real work. The professional services line covers the project: planning, dry runs, parallel-running, decommissioning. The disruption is real for 12-18 months. The disruption ends; the savings continue."
  - "Most customers find the operational simplification is what makes it worthwhile, even when the BoM-line-item math is closer than they expected. Ask your team if they want to spend the next 5 years coordinating across four vendors, or one platform."
- **Address the "sticker is roughly the same" framing:**
  - "Sticker is rarely the right comparison. Run-rate over 5 years with all categories included is the right comparison. If sticker was the only thing that mattered, every customer would buy the cheapest possible and we'd never have these conversations. The real decision is risk-adjusted run-rate over the planning horizon."
- **Offer a concrete next step:**
  - "Let me build the apples-to-apples 5-year TCO with your team, including all the categories I just named. If the math comes out roughly the same, you stay with VMware and we revisit at next refresh. If the math favors Nutanix by enough margin to justify the migration, you have a clean decision. Either way, you'll have the answer instead of guessing."
- **Close with respect for his role:**
  - "You're the right person to be skeptical here. CFOs who run the math carefully save their organizations real money. I want to do the math with you, not at you."

**A weak answer misses:**
- Arguing back without acknowledging the sticker observation
- Hand-waving on the operational simplification number ("it'll save lots of money, trust me")
- Not naming the migration risk honestly
- Failing to offer the concrete TCO modeling exercise as the next step
- Treating the CFO as an obstacle rather than a decision-maker doing his job

**Why this question matters for NCX:** Senior financial decision-makers test whether the SA understands the customer's accounting reality. The disposition is professional, specific, honest about disruption, and oriented toward producing a defensible analysis rather than winning an argument.

---

## What You Now Have

You can read the Nutanix licensing matrix in 60 seconds. AOS Pro vs Ultimate. NCM Starter vs Pro vs Ultimate. What's included with each tier and what falls below the line. The structure is in your hands.

You know the hardware sourcing options: NX (Nutanix-branded, single-vendor support), OEM (Dell XC, Lenovo HX, HPE DX, Cisco UCS - preserves customer relationships), and software-only / HCIR (most flexible, multi-vendor support boundaries). You ask the sourcing question early in discovery.

You can build a complete BoM with all six sections: hardware, software, services, support, one-time items, recurring items. You name every line, even the items that are estimates.

You can run a 5-year TCO model that holds up under CFO scrutiny: all categories, all years, stated assumptions, sensitivity analysis, comparable structure for incumbent and proposed options. You produce the one-page summary that CFOs read.

You have the Broadcom comparison framing: precise, professional, customer-specific. You don't pile on; you build the apples-to-apples analysis. You know the variables that determine whether the comparison favors Nutanix marginally, significantly, or only in specific scenarios.

You know the capex vs opex frame for CFO conversations. You ask the accounting policy preference early.

You have the multi-year subscription mechanics: 1, 3, 5 year terms; growth provisions and true-ups; when to recommend each based on customer growth and confidence.

You have the hidden cost categories that customers forget: parallel running, recurring training, third-party tooling, decommissioning, network upgrades, datacenter changes, audit work, staff role changes. You name them in the BoM.

You have twelve practice questions worth of pricing, BoM, and TCO discrimination, including two NCX-style design defenses (full multi-tier consolidation TCO comparison with sensitivity analysis, and the architectural defense of Nutanix economics against a skeptical CFO).

You are now ready for the synthesis module. Module 10 brings everything together: the customer-running migration playbook from VMware to Nutanix, the order of operations, the technical and commercial timing, the parallel-running patterns, the cutover decisions, and the year-2 stable state. After that, the appendices.

---

## References

Authoritative sources verified during the technical review pass on this module. Licensing structure changes more than any other dimension; reverify against the BlueAlly internal pricing tools and the current Nutanix software-options page before quoting any specific tier or number to a customer.

- [Nutanix Cloud Platform Software Options](https://www.nutanix.com/products/cloud-platform/software-options). Authoritative source for current NCI tiers (Pro / Ultimate), NCM tiers (Starter / Pro / Ultimate), and NCP bundle structure.
- [Nutanix Cloud Infrastructure (NCI) Datasheet](https://www.nutanix.com/library/datasheets/nci). Tier-by-tier feature comparison and the NCI vs NCI-Compute split.
- [Nutanix Cloud Manager (NCM) Datasheet](https://www.nutanix.com/library/datasheets/ncm). NCM tier structure and Self-Service / X-Play / cost-governance gating.
- [License Manager — Conversion Requirements (Nutanix Portal)](https://portal.nutanix.com/page/documents/details?targetId=License-Manager:lmg-licmgr-pnp-licensing-requirements-r.html). AOS Pro → NCI Pro conversion requirements (AOS 6.1.1+ / PC 2022.4+ / NCC 4.5.0+).
- [NCM/NCI/NUS Licensing (Nutanix Community)](https://next.nutanix.com/ncm-nci-nus-licensing-new-licensing-183). Active community thread on the licensing transition; useful for current customer-side experiences.
- [Impact of Not Converting AOS Pro to NCI Pro](https://next.nutanix.com/ncm-nci-nus-licensing-new-licensing-183/impact-of-not-converting-aos-pro-license-to-nci-pro-before-expiry-45466). What happens to legacy AOS licenses at renewal.
- [VCF Licensing Guide 2026 (Redress Compliance)](https://redresscompliance.com/vcf-licensing-guide-2026.html). Current Broadcom VMware licensing structure for the VMware comparison: VCF $350/core, vSphere Foundation $190/core, 16-core CPU minimum, 72-core order minimum.
- [vSphere Foundation vs Standard 2026](https://vmwaremadesimple.com/articles/vsphere-foundation-vs-standard-2026.html). Current vSphere Foundation MSRP and partner quote ranges.
- [Lenovo HX Nutanix Software Solution Product Guide](https://lenovopress.lenovo.com/lp1765-nutanix-software-solution-product-guide). OEM-partner pricing structure example.
- [Flow Network Security Licensing (from Module 6 References)](https://www.nutanix.com/products/cloud-platform/software-options). Flow ships with NCI Ultimate or as the Security Add-On for NCI Pro (per usable TiB; bundles Data-at-Rest Encryption).

---

## Cross-References

- **Previous:** [Module 8: Unified Storage](./08-unified-storage.md)
- **Next:** [Module 10: Migration Path](./10-migration-path.md)
- **Glossary:** [AOS Pro](./appendix-a-glossary.md#aos-pro) · [AOS Ultimate](./appendix-a-glossary.md#aos-ultimate) · [NCM tiers](./appendix-a-glossary.md#ncm-tiers) · [NX appliance](./appendix-a-glossary.md#nx-appliance) · [OEM partner](./appendix-a-glossary.md#oem-partner) · [HCIR](./appendix-a-glossary.md#hcir) · [BoM](./appendix-a-glossary.md#bom) · [TCO](./appendix-a-glossary.md#tco) · [VCF](./appendix-a-glossary.md#vcf) · [VVF](./appendix-a-glossary.md#vvf) · [Per-core licensing](./appendix-a-glossary.md#per-core-licensing) · [True-up](./appendix-a-glossary.md#true-up)
- **Comparison Matrix:** [Tier feature matrix](./appendix-b-comparison-matrix.md#tier-features) · [Hardware sourcing options](./appendix-b-comparison-matrix.md#hardware-sourcing) · [TCO category structure](./appendix-b-comparison-matrix.md#tco-categories)
- **Objections:** [#35 "Sticker price comparison"](./appendix-d-objections.md#obj-035) · [#36 "Subscription vs perpetual"](./appendix-d-objections.md#obj-036) · [#37 "Migration cost is too high"](./appendix-d-objections.md#obj-037) · [#38 "We just renewed VMware"](./appendix-d-objections.md#obj-038) · [#39 "Hardware vendor lock-in"](./appendix-d-objections.md#obj-039) · [#40 "TCO claims feel inflated"](./appendix-d-objections.md#obj-040)
- **Discovery Questions:** [Q-ECON-01 Current annual run-rate](./appendix-e-discovery-questions.md#q-econ-01) · [Q-ECON-02 Refresh timing](./appendix-e-discovery-questions.md#q-econ-02) · [Q-ECON-03 Capex vs opex preference](./appendix-e-discovery-questions.md#q-econ-03) · [Q-ECON-04 Hardware sourcing preference](./appendix-e-discovery-questions.md#q-econ-04) · [Q-ECON-05 Growth projection](./appendix-e-discovery-questions.md#q-econ-05)
- **Sizing Rules:** [Per-core sizing for AOS subscription](./appendix-f-sizing-rules.md#per-core) · [TCO sensitivity factors](./appendix-f-sizing-rules.md#tco-sensitivity)
- **Reference Architectures:** [Cost-optimized mid-market](./appendix-i-reference-architectures.md#ra-cost-mid) · [Enterprise consolidation TCO baseline](./appendix-i-reference-architectures.md#ra-tco-enterprise)
- **POC Playbook:** [Pricing scenario walkthrough](./appendix-j-poc-playbook.md#pricing-scenarios) · [TCO model template](./appendix-j-poc-playbook.md#tco-template)

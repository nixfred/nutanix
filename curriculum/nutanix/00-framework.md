---
type: framework
title: Nutanix for the VMware Mind. BlueAlly SA Enablement and Cert Prep Platform
audience: Claude Code (build agent)
primary_users: BlueAlly Solutions Architects (Nutanix practice)
version: 2.0
date: 2026-04-30
---

# Nutanix for the VMware Mind

## Project Framework and Build Specification (v2)

This document is the meta-guide for the build agent. It describes the curriculum, the teaching methodology, the file layout, the design system, and the rendering target. Read this in full before touching any module file. If a module conflicts with this framework, the framework wins.

---

## 1. What We're Building

A self-contained dual-purpose web platform for BlueAlly Solutions Architects (Nutanix practice). It does three jobs simultaneously, and every page must serve all three:

1. **Teach the platform** to someone whose mental model lives in VMware. Map every Nutanix concept to a vSphere/ESXi anchor.
2. **Drive certification progression.** NCA on day one, NCP-MCI within ~60 days of starting, NCM-MCI by month 12, NCX-MCI on the 12-to-24-month horizon, NPX as a multi-year goal. Every module page includes exam-aligned content with cert-specific tagging.
3. **Equip the SA in front of customers.** Positioning, discovery questions, objection handling, sizing math, POC playbook. The SA should be able to open this site in a coffee-shop fifteen minutes before a customer meeting and walk in sharper.

These three jobs are not in tension. They are reinforcing. The same conceptual mastery that wins exams wins customer trust.

---

## 2. Audience: Speak Directly to Them

**The reader is a BlueAlly Solutions Architect.** Not a generic VMware admin. Not "the reader." A BlueAlly SA. Address them as `you` throughout. They have:

- 5+ years presales experience minimum (some have 25+)
- Deep VMware/vSphere/ESXi knowledge
- A laptop, a customer Zoom, and a sales rep (Claire, Monica, Ginger, others) who just brought them into a deal
- Pressure to close: 3% of invoiced gross profit, with meaningful-contribution requirements
- A migration wave coming at them, VMware shops staring at Broadcom renewals, and a window to win

Where customer-facing context matters, write for the SA chair. "When the customer asks X, you say Y because Z." Not abstract. Specific.

What they do **not** know:
- Most Nutanix vocabulary (AHV, AOS, CVM, DSF, Prism, Flow, NCM, NDB, NKE, NC2)
- Advanced VMware products they may not have touched: NSX, vSAN, Horizon, SRM, vRealize/Aria. When a Nutanix concept maps to one of these, teach both sides.

---

## 3. Teaching Methodology: Winston's Rules, Applied

Every module is built on Patrick Winston's "How to Speak" framework. This is non-negotiable. The structure is the pedagogy.

| Winston Rule | How It Shows Up |
|---|---|
| Open with a promise, not a joke | `## The Promise` opens every module: explicit, specific empowerment statement, named cert outcomes |
| Curse of knowledge: explain the obvious | Anchor every Nutanix concept to ESXi/vSphere; never assume Nutanix-side context |
| The 5-minute rule | The Promise + Foundation sections are disproportionately polished, they earn the rest |
| Cycle: repeat in 3 different ways | Each major idea appears as: (1) prose, (2) diagram, (3) VMware contrast, (4) exam framing, that's actually four passes |
| Build a fence | Every key concept includes "what this is NOT" passages to prevent collision with similar ideas |
| Verbal punctuation | Section breaks are explicit. Transitions signposted. No idea blurs into the next |
| Inspire before inform | The Promise establishes stakes (career, customer credibility, commission) before content |
| End with contribution | Modules close with `## What You Now Have`, what was gained, not what was said |

---

## 4. Module Structure (Mandatory Template)

Every module file conforms to this skeleton. **Module 01 is the canonical template, when in doubt, mirror it.**

```markdown
---
module: NN
title: ...
estimated_reading_time: NN min
prerequisites: [list]
key_terms: [list]
diagrams: [list of diagram IDs]
cert_coverage:
  NCA: ~XX%
  NCP-MCI: ~XX%
  NCM-MCI: ~XX%
  (other certs: %)
---

# Module N: [Title]

## Cert Coverage Banner
[Visual badge bar showing which certs this module serves and at what weight]

## The Promise
What you will know by the end that you don't know now. Specific. Concrete. No hedging.
Includes the cert and customer-facing outcomes.

## Foundation: What You Already Know
Anchor to specific ESXi/vSphere knowledge. Build the fence: what it is NOT before what it IS.

## Core Content
The substance. Subsections as needed. Concrete before abstract. Cycle.

### [Subsection]
Prose...

> [!ON-THE-EXAM] (cert: NCA, NCP-MCI)
> Specific topic likely on the exam. Common distractor. Terminology distinction.

> [!FROM-THE-SA-CHAIR]
> When a customer says X, you say Y. The discovery question to ask back. The data that supports your answer.

> [!FAMILIAR]
> Where this maps cleanly to VMware mental models.

> [!DIFFERENT]
> Where Nutanix departs from VMware in a way that will trip up someone applying ESXi mental models.

### Diagram: [Title]
[Structured spec, see Section 7]

## Lab Exercise: [Name]
**Platform:** Nutanix Community Edition (CE 2.1)
**Time:** ~XX min
**What you'll build:**
[Step-by-step instructions tied to module concepts]

## Practice Questions
[8-12 exam-style questions with full explanations, see Section 8]

## What You Now Have
The Winston contribution close. Not a summary. A statement of gain.

## Cross-References
- Previous: [link]
- Next: [link]
- Glossary terms: [links to Appendix A]
- Discovery questions: [links to Appendix E]
- Objection handlers: [links to Appendix D]
```

---

## 5. Module Inventory and Sequencing

Ten core modules. Sequenced so each builds on the previous. Cert coverage roughly maps the NCA/NCP-MCI blueprint.

| # | File | Title | Primary Cert Weight |
|---|---|---|---|
| 01 | `01-hci-foundations.md` | What HCI Is and Why It Exists | NCA, NCP-MCI |
| 02 | `02-nutanix-architecture.md` | The Nutanix Stack: Node, CVM, Cluster | NCA, NCP-MCI |
| 03 | `03-ahv-hypervisor.md` | AHV: The Hypervisor Question | NCP-MCI |
| 04 | `04-prism-management.md` | Prism: Element and Central | NCA, NCP-MCI |
| 05 | `05-dsf-storage-deep-dive.md` | DSF Deep Dive: How Storage Actually Works | NCP-MCI, NCM-MCI |
| 06 | `06-networking-flow.md` | Networking and Microsegmentation | NCP-MCI, NCP-NS |
| 07 | `07-data-protection.md` | Data Protection and DR | NCP-MCI, NCM-MCI |
| 08 | `08-unified-storage.md` | Files, Objects, Volumes | NCP-US |
| 09 | `09-licensing-economics.md` | Licensing and Real Costs | (sales-critical) |
| 10 | `10-migration-path.md` | Migrating from VMware to Nutanix | NCP-MCI, NCM-MCI |

Plus eleven appendices (the SA toolkit):

| File | Content |
|---|---|
| `appendix-a-glossary.md` | Nutanix term ↔ VMware equivalent, alphabetical, both directions |
| `appendix-b-comparison-matrix.md` | Side-by-side Nutanix vs VMware feature matrix |
| `appendix-c-scenarios.md` | Final assessment: scenario-based decision exercises |
| `appendix-d-objections.md` | 30+ VMware-admin objections with responses, data, discovery follow-ups |
| `appendix-e-discovery-questions.md` | Qualifying and design questions per Nutanix area |
| `appendix-f-sizing-rules.md` | Node-count math, RF overhead, CPU/RAM ratios per workload type |
| `appendix-g-cli-reference.md` | `acli`, `ncli`, `cluster`, NCC, v4 API, printable cheat sheet |
| `appendix-h-competitive-matrix.md` | Nutanix vs VxRail, Cisco HX (EOL), Azure Stack HCI, Pure+vSphere |
| `appendix-i-reference-architectures.md` | Small/mid/large + VDI + hybrid (NC2) reference designs |
| `appendix-j-poc-playbook.md` | What to demo, in what order, common gotchas, the "aha" moments |
| `appendix-k-cert-tracker.md` | Cert blueprints, weighted topics, study-plan generator data |

---

## 6. Required Diagrams (Master List)

Every diagram is hand-drawn-style (Excalidraw aesthetic, rendered via rough.js). Every diagram has metadata:

```
exam_relevance: [list of certs]  # which exams test material in this diagram
whiteboard_ready: true | false   # whether the SA should be able to redraw from memory in front of a customer
```

Master list:

1. **Three-tier vs HCI side-by-side** (M1), `whiteboard_ready: true`, NCA, NCP-MCI
2. **Anatomy of a Nutanix node**, hypervisor, CVM, local disks (M2), `whiteboard_ready: true`, NCA, NCP-MCI
3. **A 4-node cluster with DSF distribution** (M2), `whiteboard_ready: true`, NCA, NCP-MCI
4. **The data path: VM write → CVM → local + remote replica** (M5), `whiteboard_ready: true`, NCP-MCI, NCM-MCI
5. **RF2 vs RF3 visual**, failure domain (M5), NCP-MCI
6. **Erasure coding (EC-X) layout** vs RF replication (M5), NCP-MCI, NCM-MCI
7. **AHV networking: bridges, bonds, VLANs** mapped to vSwitch (M6), `whiteboard_ready: true`, NCP-MCI, NCP-NS
8. **Flow microsegmentation: app-centric policy** (M6), NCP-NS
9. **Prism Element vs Prism Central hierarchy** (M4), `whiteboard_ready: true`, NCA, NCP-MCI
10. **Protection Domain + Async / NearSync / Metro** comparison (M7), NCP-MCI, NCM-MCI
11. **Migration architecture: VMware → hybrid → Nutanix** (M10), `whiteboard_ready: true`, sales-facing
12. **Node failure recovery (curator scan)** (M5), NCP-MCI, NCM-MCI
13. **Files / Objects / Volumes positioning** (M8), NCP-US

---

## 7. Diagram Specification Format

Each diagram block:

```
### Diagram: [Title]
**id:** `kebab-case-id`
**type:** architecture | flow | comparison | timeline | layered
**caption:** One-line caption shown below the diagram
**exam_relevance:** [NCA, NCP-MCI, ...]
**whiteboard_ready:** true | false

**Elements:**
- Component A: [shape] [position] [color] [label]
- Component B: ...

**Connections:**
- A → B: [label]

**Annotations:**
- Free-text callouts pointing at specific elements

**Why this diagram exists:**
One sentence on what the reader takes away.
```

Color conventions for the Excalidraw rendering:

- **Compute / Hypervisor:** muted blue (#5b7a99)
- **Storage / DSF:** rust (#b04a26), signals the differentiator
- **Network:** teal (#0d5e5e)
- **Management plane:** gold (#b8862c)
- **VMware reference:** desaturated gray-blue (#607d8b)
- **Failure / fault domain:** dotted red outline (#9e2424)

When `whiteboard_ready: true`, the diagram renders with an additional toggle: **"Practice mode"** hides the labels for memorization study.

---

## 8. Practice Question Structure

End-of-module questions follow exam style. Mix:

- 50% straight knowledge MCQ (NCA-style)
- 30% scenario MCQ (NCP-MCI-style, "A customer wants X with constraint Y, which option...")
- 20% scenario short-answer (NCX-prep, open-ended)

Each MCQ specifies:

```
**Q: [question]**
**Cert relevance:** [NCA / NCP-MCI / etc.]
A) ...
B) ...
C) ...
D) ...

**Answer:** [letter]

**Why this answer:** [explanation]
**Why not the others:**
- A) [why this is wrong]
- C) [why this is wrong]
- D) [why this is wrong]

**The trap in this question:** [what the test-writer is testing for; the gotcha]
```

The "Why not the others" and "trap" sections are not optional. Most exam-prep sites give you "B is correct, here's why." That is not enough. The reader needs to understand the full discrimination logic, because that is what the exam tests.

---

## 9. Callout Conventions

Five callout types. Render each as visually distinct boxed inserts:

| Callout | Color | Purpose |
|---|---|---|
| `> [!FAMILIAR]` | teal | Where Nutanix maps cleanly onto an existing VMware concept |
| `> [!DIFFERENT]` | rust | Where Nutanix departs in a way that trips up VMware mental models |
| `> [!ON-THE-EXAM]` | gold | Cert-tagged exam alert with topic, distractor, terminology |
| `> [!FROM-THE-SA-CHAIR]` | deep navy | Customer-facing positioning, what to say, the discovery question |
| `> [!LAB]` | green | Pointer to a CE hands-on exercise |

No `info`, `tip`, `warning`, or `note` callouts. Only these five. The contrast is the teaching tool.

---

## 10. Quality Bar (Read Twice)

The reader is a senior presales architect with 25+ years in infrastructure. They will detect:

- Hand-waving ("basically, it just...")
- VMware-bashing or Nutanix-fanboying
- Softened tradeoffs
- Marketing language masquerading as technical content
- Vague comparisons that avoid specifics
- Cert-prep filler ("memorize these acronyms!") with no understanding behind it

Avoid all of these. When something is genuinely a tradeoff, name it. When AHV is behind ESXi, say where. When Nutanix licensing is worse for a specific scenario, name the scenario. Where Nutanix is genuinely ahead, one-click upgrades, integrated lifecycle (LCM), cluster expansion, the management UX, free hypervisor, state that plainly with numbers.

**Hard rules:**

- No em-dashes. Use colons, commas, parens, or sentence breaks.
- No "hope this finds you well" energy. Direct, professional, dry.
- No emoji.
- No "exciting" or "revolutionary" or "game-changing."
- Numbers and specifics over adjectives. "AHV adds roughly 6 minutes to a host upgrade vs ESXi" beats "AHV is fast to upgrade."
- Cite specific terms (AOS, AHV, CVM, DSF, RF, NCM, Flow, NDB, NKE, NC2) and define on first use.
- Name versions: AOS 7.5, AHV 11.0, Prism Central 7.5 are current as of Q1 2026.

---

## 11. Site Build Instructions for Claude Code

When rendering these markdown files into a website:

**Output:** Single self-contained `index.html` (split if file size demands it). External runtime: Google Fonts, rough.js from CDN. No backend.

**Layout:**
- Sticky left sidebar: module list with cert-coverage badge per item, completion indicator (intersection-observer driven, persisted via localStorage)
- Main content area: serif body, monospace for technical terms
- Right rail (desktop only): in-module table of contents, cert-relevance chips, jump-to-practice-questions

**Required interactive features:**

1. **Cert progress dashboard.** Persistent header widget showing aggregate completion across NCA / NCP-MCI / NCM-MCI based on (a) modules read, (b) practice questions attempted, (c) practice questions correct. Stored in localStorage. The reader explicitly likes deterministic, local-only state.

2. **Search.** Client-side full-text index across all modules and appendices. No server. Use Lunr.js or FlexSearch from CDN.

3. **"10-minute brief" toggle per module.** Collapses the module to: Promise + The 3 SA-Chair callouts + Comparison-to-VMware summary + key diagram. For pre-meeting cram. Hide-able section markers in markdown using `<details data-brief>`.

4. **Print/PDF export per module.** Print-stylesheet that strips nav and renders for letter-size paper. Page breaks before headings.

5. **Bookmarks and notes per section.** Click a section heading, get a bookmark icon. Sidebar panel lists bookmarks with optional notes. localStorage.

6. **Diagram practice mode.** For diagrams with `whiteboard_ready: true`, a toggle hides labels. Click each element to reveal. For exam memorization.

**Typography:**
- Display: **Fraunces** (variable, opsz tuned high for headlines)
- Body: **Source Serif 4** (8..60 opsz)
- Mono: **JetBrains Mono**
- Avoid: Inter, Roboto, Space Grotesk, system fonts.

**Color palette:**
- Background: warm off-white `#f5f1e8` (paper)
- Ink: `#1a1d24`
- Rules: `#c9bfa5`
- Teal (FAMILIAR): `#0d5e5e`
- Rust (DIFFERENT, Nutanix orange): `#b04a26`
- Gold (ON-THE-EXAM): `#b8862c`
- Deep navy (FROM-THE-SA-CHAIR): `#1e3a5f`
- Green (LAB): `#3d6b3d`

**Diagrams:** Render with rough.js into inline SVG. One-time fade-in on first scroll into view. No looping animation.

**Navigation:** Smooth scroll between modules. URL hash updates as the reader scrolls. Sidebar tracks active section.

---

## 12. File Naming Conventions

- All files in this directory.
- Snake-case-with-numbers, lowercase, `.md`.
- Front matter is YAML, on every file.
- Internal links: `[Module 5](./05-dsf-storage-deep-dive.md)`.
- Glossary refs: `[CVM](./appendix-a-glossary.md#cvm)`.
- Objection refs: `[Objection #12](./appendix-d-objections.md#obj-012)`.
- Discovery refs: `[Q-LIC-03](./appendix-e-discovery-questions.md#q-lic-03)`.

---

## 13. The North Star

When a BlueAlly SA finishes the last scenario in Appendix C and closes the tab, they should think:

> *I could now sit across from a Nutanix SE, or, more importantly, across from a customer's VMware admin, and ask the right questions. I could now defend, on technical merit, whether their environment should move. I now know what they would lose, what they would gain, and what they would have to relearn. Nobody softened anything for me. And I've passed NCA, I'm halfway through NCP-MCI prep, and I've got a discovery question library and an objection handler ready before the next call.*

That is the bar. Build to it.

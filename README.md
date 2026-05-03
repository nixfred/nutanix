# Nuta**NIX**

```
$ cd /nix/nutanix
$ ls
01-hci-foundations/   06-networking-flow/      a-glossary/              g-cli-reference/
02-nutanix-architecture/  07-data-protection/  b-comparison-matrix/     h-competitive-matrix/
03-ahv-hypervisor/    08-unified-storage/      c-scenarios/             i-reference-architectures/
04-prism-management/  09-licensing-economics/  d-objections/            j-poc-playbook/
05-dsf-storage/       10-migration-path/       e-discovery-questions/   k-cert-tracker/
                                               f-sizing-rules/
$ _
```

A field guide to Nutanix, written for the people who have to explain it.

**Live site:** [nixfred.com/nutanix](https://nixfred.com/nutanix)
**Status:** Curriculum complete. Ten modules, eleven appendices, all shipped.

---

## What this is

A learning-and-reference site built from a complete Nutanix curriculum. Cert prep, customer-conversation language, whiteboard-ready diagrams, honest comparisons against the incumbent stacks. Dark mode by default. Hand-authored. Works on a plane.

It is also this repo. Each page is a single HTML file. Each hero is a single SVG. No build step, no framework, no JavaScript bundle to ship. Open `index.html` in a browser and it just works.

## Who it is for

Solutions Architects at BlueAlly Technology Solutions, and any SA-shaped engineer who has to walk into a room and defend a Nutanix design without hand-waving. Specifically:

- You came from VMware and you need the translation layer, not a 101.
- You have 30 seconds between meetings and you need the answer fast.
- You have two hours on a Saturday and you are studying for NCP-MCI.
- You will be sitting across from a competing architect next week and you need real comparisons, not marketing slides.

If you are looking for a vendor-glossy product tour, this is not that.

---

## The ten modules

Each module follows the same shape: cert coverage up top, four mental frames on every concept, a fence around what it is not, a lab exercise, twelve practice questions with traps explained, a hand-authored hero SVG, and 20 to 40 in-page rough.js diagrams.

| | Module | Focus |
|---|---|---|
| **01** | [HCI Foundations](https://nixfred.com/nutanix/modules/01-hci-foundations/) | Why hyperconverged, where it came from, what it replaces |
| **02** | [Nutanix Architecture](https://nixfred.com/nutanix/modules/02-nutanix-architecture/) | CVM, Stargate, Cassandra, Curator, Medusa, Pithos, Zeus |
| **03** | [AHV Hypervisor](https://nixfred.com/nutanix/modules/03-ahv-hypervisor/) | The hypervisor most VMware engineers underestimate |
| **04** | [Prism Management](https://nixfred.com/nutanix/modules/04-prism-management/) | Element vs Central, NCM Self-Service, X-Play, the v4 API |
| **05** | [DSF Deep Dive](https://nixfred.com/nutanix/modules/05-dsf-storage/) | The data path, tiering, locality, the parts customers grill you on |
| **06** | [Networking and Flow](https://nixfred.com/nutanix/modules/06-networking-flow/) | OVS, microsegmentation, VPC overlays without the firewall theater |
| **07** | [Data Protection](https://nixfred.com/nutanix/modules/07-data-protection/) | Snapshots, Async, NearSync, Metro, Recovery Plans, NC2 |
| **08** | [Unified Storage](https://nixfred.com/nutanix/modules/08-unified-storage/) | Files, Objects, Volumes; when each one is the right answer |
| **09** | [Licensing and Economics](https://nixfred.com/nutanix/modules/09-licensing-economics/) | NCI, NCM, NCP bundles. The conversation customers actually want to have |
| **10** | [Migration Path](https://nixfred.com/nutanix/modules/10-migration-path/) | Move, design tradeoffs, hybrid steady-state, the parts that go wrong |

## The eleven appendices

The reference toolkit. Read on demand between meetings; lean on during competitive engagements; quote from in proposals.

| | Appendix | What it gets you |
|---|---|---|
| **A** | [Glossary](https://nixfred.com/nutanix/appendices/a-glossary/) | 119 terms across 21 letter sections, alphabetic lookup |
| **B** | [Comparison Matrix](https://nixfred.com/nutanix/appendices/b-comparison-matrix/) | 24 head-to-head comparisons across 12 categories |
| **C** | [Scenarios](https://nixfred.com/nutanix/appendices/c-scenarios/) | 10 capstone design exercises with strong-vs-weak answers |
| **D** | [Objections Handbook](https://nixfred.com/nutanix/appendices/d-objections/) | 45 customer objections with SA-chair response scripts |
| **E** | [Discovery Questions](https://nixfred.com/nutanix/appendices/e-discovery-questions/) | 46 questions, what to listen for, where each branch goes |
| **F** | [Sizing Rules](https://nixfred.com/nutanix/appendices/f-sizing-rules/) | 13 rules of thumb, 14 sizing tables, raw-to-usable math |
| **G** | [CLI Reference](https://nixfred.com/nutanix/appendices/g-cli-reference/) | ncli, acli, ncc, ovs, manage_ovs, Move, 9 diagnostic recipes |
| **H** | [Competitive Matrix](https://nixfred.com/nutanix/appendices/h-competitive-matrix/) | 9 competitors with discovery questions, kill questions, win/loss conditions |
| **I** | [Reference Architectures](https://nixfred.com/nutanix/appendices/i-reference-architectures/) | 8 sized designs: Small, Medium, Large, VDI, Hybrid+NC2, ROBO, Greenfield, Compliance |
| **J** | [POC Playbook](https://nixfred.com/nutanix/appendices/j-poc-playbook/) | 10 demo scripts, 30-day cadence, scoping and final-report templates |
| **K** | [Cert Tracker](https://nixfred.com/nutanix/appendices/k-cert-tracker/) | Blueprint coverage map for the seven-cert ladder |

---

## Cert ladder coverage

```
NCX-MCI    Expert         live oral panel defense, ~50% pass rate
   |
NCM-MCI    Master         100-150 hr prep + 6 months production experience
   |
NCP-MCI    Professional   the core HCI cert, target first
NCP-NS     Professional   Network Security specialization
NCP-US     Professional   Unified Storage specialization
NCP-MCA    Professional   Multicloud Automation specialization
   |
NCA        Associate      foundational concepts, 20-40 hr prep
```

Each module declares per-exam coverage in its YAML front matter. Appendix K maps every blueprint domain to the modules and appendices that teach it, with study sequences, hands-on labs, and the specific pitfalls each cert tier hides.

## What makes it different

- **One file per page.** No framework, no bundler, no node_modules. Each module is a single self-contained HTML file. Each hero is a single SVG. View source and the whole thing is right there.
- **Hand-authored hero SVG per page** in the same isometric vocabulary, tuned per topic. The Module 5 hero is a storage waterfall. Module 6 is a microsegmented network. Appendix H is a competitive constellation. Appendix K is the cert ladder. Twenty-one heroes, no stock art.
- **rough.js diagrams in flow.** Whiteboard-style. Designed so an SA can redraw them in front of a customer.
- **Dark mode only.** No light-mode toggle. Designed for late-night prep and laptop screens on planes.
- **No em-dashes anywhere.** Not in body text, not in tables, not in metadata. Use commas and colons.
- **No emoji, no AI tells.** No "in conclusion." No "it's important to note that." No breathless adjectives. Direct, professional voice top to bottom.
- **Tight typography.** Inter for display, JetBrains Mono for code, generous line height, max 75ch line length, tabular numerals where they belong.
- **The wordmark is a play on the author's surname.** Nuta**NIX** with the trailing three letters in cyan. Once you see it, you do not unsee it.

---

## Run it locally

```bash
git clone https://github.com/nixfred/nutanix.git
cd nutanix
python3 -m http.server 8000
```

Open `http://localhost:8000`. That is the build.

If you want to read the source markdown without the site chrome, read `curriculum/nutanix/`. Each module and appendix is plain markdown with strict YAML front matter. The HTML pages under `modules/` and `appendices/` are the rendered output.

## Repo structure

```
nutanix/
├── README.md                    # this file
├── CLAUDE.md                    # build discipline, voice rules, design tokens
├── index.html                   # homepage
├── 404.html                     # terminal-aesthetic miss
├── modules/                     # 10 module pages, one HTML each
├── appendices/                  # 11 appendix pages, one HTML each
├── public/
│   ├── images/                  # 21 hand-authored hero SVGs + share-card.png
│   └── favicon.svg              # NX monogram
└── curriculum/                  # source markdown the site is rendered from
    └── nutanix/
        ├── 00-framework.md
        ├── 01-hci-foundations.md  ...  10-migration-path.md
        └── appendices/
            └── appendix-a-glossary.md  ...  appendix-k-cert-tracker.md
```

## Source materials

The curriculum is plain markdown with YAML front matter declaring `track`, `order`, `slug`, `status`, `prerequisites`, `key_terms`, `diagrams`, `cert_coverage`, and `sa_toolkit`. The HTML pages were hand-rendered against that source. The plan in `CLAUDE.md` describes a future Astro path that activates if the curriculum scales beyond the current single track; until then, hand-rendered HTML wins on simplicity, performance, and the absence of any build pipeline that could break.

---

## Built by

[Fred Nix](https://github.com/nixfred), Senior Solutions Architect at [BlueAlly Technology Solutions](https://blueally.com). Built for the BlueAlly SA bench. Public on the assumption that other people are trying to learn the same things.

The curriculum was developed over a multi-day collaboration with Claude (Anthropic) using the Patrick Winston "How to Speak" methodology: cycle each concept four ways, build a fence around what it is not, calibrate to the audience. Every module file, every appendix, every hero SVG, every line of CSS was reviewed by a working SA before shipping.

PRs welcome from anyone who finds something wrong, especially if you have been on the customer side of a Nutanix bake-off and have a war story worth turning into a discovery question.

## License

Curriculum content: **CC BY 4.0**. Code: **MIT**. Use it, fork it, ship it into your own training bench. If you teach from it, a credit back is appreciated, not required.

---

```
$ cd ..
$ pwd
/nix
$ _
```

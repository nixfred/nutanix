# Nuta**NIX**

```
$ cd /nix/nutanix
$ ls
01-hci-foundations/   06-networking-flow/      appendix-a-glossary/
02-architecture/      07-data-protection/      appendix-b-comparison-matrix/
03-ahv-hypervisor/    08-unified-storage/      appendix-c-scenarios/
04-prism-management/  09-licensing-economics/  appendix-d-objections/
05-dsf-storage/       10-migration-path/       appendix-e-discovery-questions/
```

A field guide to Nutanix, written for the people who have to explain it.

**Live site:** [nixfred.com/nutanix](https://nixfred.com/nutanix)

---

## What this is

A learning-and-reference site built from a complete 10-module Nutanix curriculum. Cert prep, customer-conversation language, whiteboard-ready diagrams, honest comparisons against the incumbent stacks. Dark mode by default. Searchable. Printable. Works on a plane.

It is also this repo. The curriculum is plain markdown with strict front matter. The site is a thin lens over those files. Read it on the web; clone it for the source.

## Who it is for

Solutions Architects at BlueAlly Technology Solutions, and any SA-shaped engineer who has to walk into a room and defend a Nutanix design without hand-waving. Specifically:

- You came from VMware and you need the translation layer, not a 101.
- You have 30 seconds between meetings and you need the answer fast.
- You have two hours on a Saturday and you are studying for NCP-MCI.
- You will be sitting across from a competing architect next week and you need real comparisons, not marketing slides.

If you are looking for a vendor-glossy product tour, this is not that.

## What is in the curriculum

Ten modules, roughly 600 pages of source. Each module follows the same shape: cert coverage up top, four mental frames on every concept, a fence around what it is not, a lab exercise, twelve practice questions with traps explained.

| | Module | Focus |
|---|---|---|
| 01 | HCI Foundations | Why hyperconverged, where it came from, what it replaces |
| 02 | Nutanix Architecture | CVM, Stargate, Cassandra, Curator, Medusa |
| 03 | AHV Hypervisor | The hypervisor most VMware engineers underestimate |
| 04 | Prism Management | Element vs Central, what each is for |
| 05 | DSF Storage Deep Dive | The data path, tiering, locality, the parts customers grill you on |
| 06 | Networking and Flow | Microsegmentation without the enterprise-firewall theater |
| 07 | Data Protection | Snapshots, replication, NearSync, Metro, Leap |
| 08 | Unified Storage | Files, Objects, Volumes, when each one is the right answer |
| 09 | Licensing and Economics | The conversation customers actually want to have |
| 10 | Migration Path | Move, design tradeoffs, the parts that go wrong |

Plus appendices: glossary, comparison matrix, design scenarios, objection handling, discovery questions, sizing rules, CLI reference.

## Cert ladder coverage

NCA. NCP-MCI. NCP-NS. NCP-US. NCP-MCA. NCM-MCI. NCX-MCI.

Each module declares per-exam coverage in front matter. The `/certs` page rolls those up so you can see what is ready to test on, what needs more work, and what is not started.

## The interactive bits

- **Cmd-K search** across every module and appendix. Lands you on the right section, not just the right page.
- **Practice questions** with the answer revealed inline, the wrong-answer traps explained, and progress saved locally.
- **Hand-drawn diagrams** rendered in rough.js so they look like something you would whiteboard, not something a marketing team produced.
- **Glossary hover-cards** on technical terms. No more tab-juggling to remember what Curator does.
- **Print-to-PDF** on every module, for the offline hours.

## Run it locally

```bash
git clone https://github.com/nixfred/nutanix.git
cd nutanix
bun install
bun run dev
```

Open `http://localhost:4321/nutanix`.

```bash
bun run build      # production build into ./dist
bun run preview    # serve the built site locally
```

## Architecture in three lines

1. **Astro + MDX + Tailwind + TypeScript strict.** Static site. No backend.
2. **Content collections with strict Zod schemas.** Drop a markdown file with valid front matter and it appears in the navigation, search, cert rollups, and prev/next. No registry to update.
3. **Designed for more than one track.** The site is structured so a future VMware-deprecation track, AWS track, or anything else lives next to Nutanix without touching navigation or routing code.

## Adding a module

```
src/content/tracks/nutanix/11-files-deep-dive.md
```

Valid front matter is the contract. The build fails loudly if anything is missing. The sidebar, search index, cert rollups, and homepage update themselves.

## Adding a track

```
src/content/tracks/<new-track>/_track.yaml
src/content/tracks/<new-track>/01-...md
```

Drop the folder. The track lands on the homepage, gets its own landing page, gets its own sidebar group, gets its own search scope. Nothing else changes.

## Source materials

The raw markdown lives under `curriculum/`. The site copies it into `src/content/` at build time. If you want to read the curriculum without the site chrome, read `curriculum/`. If you want to read it as a website, that is what `nixfred.com/nutanix` is for.

## Built by

Fred Nix, Senior Solutions Architect at [BlueAlly Technology Solutions](https://blueally.com). Built for the BlueAlly SA bench. Public on the assumption that other people are trying to learn the same things.

The curriculum and the site are evolving. PRs welcome from anyone who finds something wrong, especially if you have been on the customer side of a Nutanix bake-off and have a war story worth turning into a discovery question.

## License

Curriculum content: CC BY 4.0. Code: MIT. Use it, fork it, ship it into your own training bench. If you teach from it, a credit back is appreciated, not required.

---

```
/nix/nutanix
```

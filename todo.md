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
- [ ] 05-dsf-storage-deep-dive.md
- [ ] 06-networking-flow.md
- [ ] 07-data-protection.md
- [ ] 08-unified-storage.md
- [ ] 09-licensing-economics.md
- [ ] 10-migration-path.md

Appendices:
- [ ] appendix-a-glossary.md
- [ ] appendix-a-glossary-nz.md (resolve duplicate; pick canonical or merge)
- [ ] appendix-b-comparison-matrix.md
- [ ] appendix-c-scenarios.md
- [ ] appendix-d-objections.md
- [ ] appendix-e-discovery-questions.md
- [ ] appendix-f-sizing-rules.md
- [ ] appendix-g-cli-reference.md
- [ ] appendix-h-competitive-matrix.md
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

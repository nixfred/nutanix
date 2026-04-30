# CLAUDE.md

## Project Brief

You are building a learning-and-reference website from a complete Nutanix curriculum. The curriculum is already written. Your job is to turn it into a world-class web experience.

This is a real project. The site goes live at **nixfred.com/nutanix** as a GitHub Pages site under the `nixfred/nutanix` repository. The repository will be created during this build; nixfred.com is already configured as the custom domain at the user level (the `nixfred/nixfred.github.io` repo or equivalent), so any project repo with GitHub Pages enabled will be reachable at `nixfred.com/<repo-name>`.

## How We Got Here

Fred Nix is a Senior Solutions Architect at BlueAlly Technology Solutions, which is a Nutanix-strong reseller. He started the role on April 16, 2026. To accelerate his ramp and to build a teaching asset for the rest of the BlueAlly SA bench, an earlier Claude (working in the chat product) built a 10-module curriculum over a multi-day collaboration. The curriculum:

- Teaches Nutanix to engineers with VMware backgrounds
- Prepares readers for the Nutanix cert ladder (NCA, NCP-MCI, NCP-NS, NCP-US, NCP-MCA, NCM-MCI, NCX-MCI)
- Equips Solutions Architects with the language, framing, objection handling, and design judgment they need in front of customers
- Embeds the Patrick Winston "How to Speak" methodology: cycle each concept four ways, build a fence around what it isn't, calibrate to the audience

The curriculum is in `/curriculum/` (you'll move the source markdown there during scaffolding). Read every module before you start designing. The voice and content discipline are non-negotiable inputs to the design.

## Audience

**Other BlueAlly Solutions Architects.** Not customers. Not students. Working SAs who need:

- Reference material they can pull up between calls
- Cert prep with embedded practice questions
- Customer-conversation language they can rehearse
- Diagrams they can whiteboard
- Honest comparisons they can defend in front of incumbent-vendor architects

Design every screen for someone with 30 seconds between meetings, on a laptop, looking for a specific answer. Then design it again for someone with two hours on a Saturday studying for NCP-MCI. Both modes need to work.

## Source Materials

The 10 module files plus the framework spec live in the source. You will see:

- `00-framework.md` is the meta-spec describing the curriculum's structure, callout types, diagram metadata, cert blueprints, and the website-build instructions (this very file's content informed those instructions). Read this first.
- `01-hci-foundations.md` through `10-migration-path.md` are the 10 modules.
- `appendix-a-glossary.md` through `appendix-k-cert-tracker.md` are reference appendices (all eleven exist as of repo init; an extra `appendix-a-glossary-nz.md` variant lives next to `appendix-a-glossary.md` and needs Phase 0 resolution).

Each module file has:

- **YAML front matter** with `module`, `title`, `estimated_reading_time`, `prerequisites`, `key_terms`, `diagrams` (a list of diagram IDs declared in body), `cert_coverage` (per-exam percentages), and `sa_toolkit` (related objection and discovery question IDs).
- **A consistent body structure**: cert-coverage banner → "The Promise" (numbered learning goals) → "Foundation: What You Already Know" → "Core Content" (with cycling, fence-building, four mental frames) → "Lab Exercise" → "Practice Questions" (12 per module, structured) → "What You Now Have" → "Cross-References".
- **Five callout types** marked with `> [!FAMILIAR]`, `> [!DIFFERENT]`, `> [!ON-THE-EXAM]`, `> [!FROM-THE-SA-CHAIR]`, `> [!LAB]`. Each has distinct semantic meaning and should render with distinct visual treatment.
- **Diagram blocks** structured with `**id:**`, `**type:**`, `**caption:**`, `**exam_relevance:**`, `**whiteboard_ready:**`, then **Elements**, **Connections**, **Annotations**, **Why this diagram exists** sections.
- **Practice questions** with structured answers: `**Answer:**`, `**Why this answer:**`, `**Why not the others:**`, `**The trap:**` for MCQ; NCX-style questions have scenario, challenge, "strong answer covers" bullets, "weak answer misses" bullets, "why this matters for NCX" closing.

Parse all of this structure. Render each element as a first-class UI component. Don't flatten the markdown into generic prose; the curriculum's information architecture is the site's information architecture.

## Visual Design

**Dark mode. Permanent dark mode. No light-mode toggle.** This is for SAs working at night and on planes, and the high-tech aesthetic the audience expects.

### Color System

Base on BlueAlly's brand palette but darkened for night use. The actual BlueAlly site uses a deep navy and electric blue. Verify by fetching `https://blueally.com` during the build to extract their brand colors precisely; the values below are reasonable defaults if the fetch fails.

```
--bg-base:       #0a1628   /* very dark navy, primary background */
--bg-surface:    #102747   /* slightly lighter navy, cards/panels */
--bg-elevated:   #1a3258   /* hover states, popovers */
--border:        #1f3a66   /* subtle dividers */
--text-primary:  #e8f0ff   /* warm white, body copy */
--text-secondary:#a8b8d4   /* muted, metadata, captions */
--text-tertiary: #6b7a9a   /* deep muted, placeholder */
--accent-primary:#4a9eff   /* BlueAlly-derived electric blue */
--accent-cyan:   #00d4ff   /* high-tech accent, hover, links */
--accent-glow:   #4a9effbf /* used for shadows/glows */
--success:       #34d399   /* correct answer */
--warning:       #fbbf24   /* trap distractor revealed */
--danger:        #f87171   /* wrong answer */

/* Callout backgrounds (translucent over surface) */
--callout-familiar:  rgba(74, 158, 255, 0.08)  /* blue tint */
--callout-different: rgba(168, 85, 247, 0.08)  /* purple tint */
--callout-exam:      rgba(251, 191, 36, 0.08)  /* gold tint */
--callout-sa:        rgba(0, 212, 255, 0.08)   /* cyan tint */
--callout-lab:       rgba(52, 211, 153, 0.08)  /* green tint */
```

Use accent colors with restraint. The high-tech feel comes from typography, layout discipline, and subtle motion, not from saturated color everywhere. Most of the screen should be deep navy and warm white.

### Typography

- **Display & headings:** `Inter` or `Geist` at heavy weights (600-700). Tight letter-spacing on large sizes.
- **Body:** `Inter` 16px regular. Generous line-height (1.65). Max line-length 70-75ch. This is reading material; it has to be easy on the eyes.
- **Code & technical labels:** `JetBrains Mono` or `Geist Mono`. Use for: code blocks, CLI commands, diagram element labels, `key_terms` chips, filenames.
- **Numbers in metadata** (cert percentages, RPO numbers, etc.): tabular figures. Use `font-variant-numeric: tabular-nums` so numbers align in tables.

### Layout

- **Sticky sidebar navigation** on the left (240-280px wide). Module list with current-page indicator, expandable to show in-page sections, search input at top.
- **Centered content column** (max 760px for prose, allow wider for diagrams and tables).
- **Persistent right rail** on wide viewports (≥1280px) showing the current page's table of contents (module sections), cert-coverage badges for the current page, and a "next/previous module" footer.
- **Mobile**: sidebar collapses to a hamburger; right rail hides; content fills width with appropriate padding.

### Motion

Subtle and purposeful. Specifically:

- Smooth scroll between in-page anchors (CSS `scroll-behavior: smooth`).
- Sidebar item highlights on hover with a 150ms transition.
- Practice-question reveal: 250ms ease-out expand for the answer block.
- Page transitions: 200ms cross-fade if you use a SPA framework. Otherwise standard browser navigation is fine.
- No bouncing, no parallax, no hero-section shenanigans. This is a working tool.

### Aesthetic Touchstones

- **Linear.app's documentation site** for the dark-mode polish and typography.
- **Vercel docs** for the layout discipline and the way they handle code/diagrams in flow.
- **Stripe docs** for the in-page navigation patterns.
- **Anthropic docs** (claude.com/docs) for the dense-but-readable balance.

Do not copy these sites. Match the *quality bar* they set.

## Branding and Identity

The site brand pulls a play on the author's surname (Fred Nix) hiding inside the product name. Use both treatments together; they reinforce each other.

### Wordmark: **NutaNIX**

Render "Nutanix" with the trailing three letters (**NIX**) in the cyan accent color (`--accent-cyan: #00d4ff`). The leading "Nuta" stays in `--text-primary`. Use this wordmark in:

- The site header (left side, sticky)
- The favicon (`NX` monogram in cyan on dark navy, or just the highlighted NIX glyph)
- Open Graph / social-share images
- The browser tab title prefix
- Print/PDF export headers

Keep the wordmark a single-color play (white + cyan); avoid gradients or effects. The NIX highlight is the one design move; everything else stays clean.

CSS sketch:

```html
<span class="wordmark">Nuta<span class="wordmark-nix">NIX</span></span>
```

```css
.wordmark { font-family: var(--font-display); font-weight: 700; letter-spacing: -0.02em; }
.wordmark-nix { color: var(--accent-cyan); }
```

### Path Treatment: `/nix/nutanix`

Render the literal URL path of the site (`/nix/nutanix`) in JetBrains Mono on every page hero, styled as a Unix path or terminal prompt element. This treatment:

- Reinforces the high-tech, engineer-built aesthetic
- Is functionally accurate (it is the actual URL path)
- Echoes the wordmark play (NIX appears twice, once highlighted, once in the path)
- Works on the home page hero as the centerpiece text and on module pages as a smaller breadcrumb-style element

Example home-page hero treatment:

```
$ cd /nix/nutanix
$ ls
01-hci-foundations/  02-architecture/  03-ahv/  ...
```

Or as a clean static element rather than animated terminal:

```
/nix/nutanix
```

Rendered in mono, with `--accent-cyan` on the leading slash and a subtle cursor-blink character at the end if you want a touch of motion. Don't overdo the animation; subtlety wins.

### Subtitle

Under the wordmark on the home page and About surface:

> A Field Guide for BlueAlly Solutions Architects.

Optionally, a smaller author line:

> Notes from the BlueAlly SA bench. By Fred Nix.

### Browser Tab Title Format

`NutaNIX · Module 5: DSF Deep Dive`

Use a centered dot (`·`, U+00B7) as the separator, never an em-dash. The wordmark stays unstyled in the tab (browsers don't render color in tab titles), so this is the one place "Nutanix" renders without the highlight. Acceptable.

### 404 Page

Lean into the terminal aesthetic. The 404 should feel like a missed `cd` command:

```
$ cd /nix/nutanix/the-page-you-wanted
cd: no such file or directory

Try one of these:
  → Home
  → Modules
  → Glossary
```

Mono font, the `cd` prompt in `--text-secondary`, the error in `--danger`, the recovery links in `--accent-cyan`. Functional and on-brand.

### Author Attribution

Footer of every page:

> Written by Fred Nix · Built for BlueAlly Solutions Architects · Source on [GitHub](https://github.com/nixfred/nutanix)

Keep it short. The site doesn't need a long colophon.

### What NOT to Do With the Branding

- **No taglines or marketing language.** "Master Nutanix in 30 days" or "Your guide to Nutanix mastery" is not the voice. The site is a working tool, not a course landing page.
- **No animation on the wordmark itself.** The cyan highlight is enough. Don't add hover effects, glow, or motion to the wordmark.
- **No alternative logos or icon variations.** One wordmark, one favicon, one path treatment. Consistency over variety.
- **No taking the play too far.** Don't write "by Fred NIX" with the surname highlighted everywhere; once on the home page is enough. Subtle.

## Required Features

### 1. Navigation

- **Sidebar** lists all 10 modules in order plus the appendices section. Each module entry is collapsible to show its in-module sections (parsed from H2 headings).
- **Search** is full-text across all modules and appendices. Use `Lunr` or `FlexSearch` (FlexSearch is faster). Index at build time, search client-side. Keyboard shortcut: `Cmd/Ctrl-K` opens the search palette.
- **Breadcrumbs** at the top of each page: `Home / Module 5 / DSF Deep Dive`.
- **In-page table of contents** in the right rail, auto-generated from headings, with active-section highlighting on scroll.
- **Module footer** with `Previous: Module N` and `Next: Module N+1` links.
- **Cross-reference resolution**: the modules contain links like `./appendix-a-glossary.md#stargate`. Resolve all of these to clean URLs (`/nutanix/glossary#stargate` or similar). Don't leave broken `.md` links.

### 2. Interactive Practice Questions

The 12 practice questions per module are the highest-value interactive feature. Render them as:

- **Question card** with question number, cert-relevance badge, and the question text.
- **Answer choices** as tappable buttons (radio-button-like behavior).
- **"Reveal answer"** button (or auto-reveal on selection).
- After reveal:
  - The correct answer is highlighted green.
  - If the user picked wrong, their pick is highlighted red.
  - The "Why this answer" explanation expands inline.
  - The "Why not the others" expands as a list.
  - The "The trap" callout appears in the warning color.
- **Per-module progress** stored in `localStorage`: questions answered, percentage correct, last attempt timestamp.
- **NCX-style questions** (Q11 and Q12 in each module) get a different treatment: open-text-area for the user to write their reasoning, then a "Show strong answer" button that reveals the structured "strong answer covers / weak answer misses / why this matters" content. No auto-grading; this is a self-assessment tool.

### 3. Practice-Mode Dashboard

A `/practice` page that aggregates all 120 MCQ questions across modules. Filter by:

- Cert exam (NCA / NCP-MCI / NCP-NS / NCP-US / NCM-MCI / NCP-MCA)
- Module
- Question type (knowledge MCQ / scenario MCQ / NCX-style)
- Status (unattempted / answered / correct / incorrect)

A "Random 20" button generates a 20-question quiz from the user's selected filters. Track time-per-question. Show a final summary screen.

### 4. Cert Progress Tracker

A `/certs` page with one card per cert exam (NCA, NCP-MCI, NCP-NS, NCP-US, NCP-MCA, NCM-MCI, NCX-MCI). Each card shows:

- Exam name and link to Nutanix's official blueprint
- Coverage percentage from the modules (sum the `cert_coverage` percentages from front matter)
- The user's practice-question performance for that cert
- A "ready / needs work / not started" indicator based on completion + accuracy

### 5. Hero Image Per Page

Every module page (and the home page, and each appendix) gets one hero image at the top of the content. The image must be:

- **Contextually related** to the page's specific topic. Module 5 (DSF) gets storage-themed imagery. Module 6 (Networking) gets network-topology-themed imagery. Module 7 (DR) gets resilience-themed imagery. The home page gets a hero that signals "Nutanix curriculum for Solutions Architects."
- **High-tech aesthetic** matching the dark-mode site. Think: abstract geometric, neon-on-dark, data-visualization-inspired, isometric architectural. Avoid stock-photo "people in a meeting" cliché.
- **Generated** rather than sourced from stock libraries. Two acceptable approaches:
  1. **AI image generation** via Replicate, Fal, or DALL-E API at build time. Cache outputs in the repo so the build is deterministic and offline-capable.
  2. **Programmatic SVG art** using p5.js, generative-art techniques, or hand-crafted SVG illustrations. Slower to author but zero runtime cost and easy to theme.
- **Sized** appropriately: ~1600x900 hero, displayed at full content width with `aspect-ratio: 16/9`. Optimize for fast loading (WebP or AVIF).

If using AI generation, prompt with the module's topic plus style anchors: "isometric architectural diagram, deep navy and electric blue, high-tech, abstract, no text, no people, no logos." Iterate until the result is on-brand. Document the prompts so future regeneration is reproducible.

### 6. Diagrams

The modules contain ~31 diagrams declared in structured blocks (id, type, caption, elements, connections, annotations). Render each diagram as:

- **A figure block** with the diagram's `caption` as a figcaption.
- **Whiteboard-ready badge** if the front-matter flag is true ("Draw this in front of customers").
- **Cert relevance badges** (NCA / NCP-MCI / etc.).
- **The actual visual** rendered with one of these approaches (in order of preference):
  1. **rough.js** for the Excalidraw-style hand-drawn aesthetic. Parse the Elements list and lay out programmatically. This is the framework's stated preference for the "whiteboard" feel.
  2. **Mermaid** for flow-style diagrams where rough.js is overkill.
  3. **Hand-authored SVG** as a fallback for diagrams that need precise control.

Rough.js is preferred because it lets readers visualize the diagram the way they would whiteboard it. Don't make the diagrams look like polished marketing slides; they should look like an SA's notebook.

### 7. Glossary Term Hover-Cards

When the body text contains a term that's defined in Appendix A (Glossary), wrap it in a `<dfn>`-like component. On hover (or tap on mobile), show a small popover with the term's definition and a link to the full glossary entry. This is one of the highest-value features for fast cross-reference.

Implementation: build a glossary index at build time from `appendix-a-glossary.md`. During markdown rendering, scan body text for known terms and wrap them. Use a debounced popover so it doesn't fire constantly.

### 8. Print/PDF Export

Each module has a "Print this module" link in the footer. Hide the sidebar, hide interactive elements, render Q&A in expanded form, optimize for letter-paper-PDF. Useful for SAs who want to study on a plane or share a module with a customer.

### 9. Bookmarks and Notes (Optional, Phase 2)

If time permits, add per-paragraph bookmarking and notes. Stored in `localStorage`. Surface a "My Bookmarks" page. Skip this if it slows down the core experience.

## Implementation Approach

> **Pinned decision (2026-04-30):** the active build path is **static HTML, hand-rendered per page**, not Astro. This is "Option C — hybrid" from the Phase 0 decision in `todo.md`. The Astro plan below is the deferred target; it activates only when one of the trigger conditions fires (second track, manual-nav pain, ≥5 modules shipped). Until a trigger fires, follow the Module 1 page (`modules/01-hci-foundations/index.html`) as the structural template for the remaining modules and appendices: shared CSS custom properties, per-module hero SVG under `public/images/hero-NN-*.svg` in the isometric-cluster vocabulary, rough.js diagrams, practice-question reveal interaction, prev/next navigation. The Astro plan below is preserved as the eventual home of the curriculum once scale justifies the migration.

You have flexibility on framework choice. Recommended:

- **Astro** is the best-in-class static site generator with first-class markdown support, partial hydration for interactive components (the practice-question UI), built-in syntax highlighting, easy GitHub Pages deployment. Use `bun` as the package manager (Fred prefers Bun over npm/yarn).
- **MDX** for the markdown processing pipeline so you can embed React/Vue/Svelte components in modules where useful.
- **TypeScript strict mode** throughout. Fred's preference; it also catches errors before deployment.
- **Tailwind CSS** for styling (matches Fred's stack and accelerates the dark-theme build). Configure with the color system above.

If Astro is unavailable or unsuitable, alternatives in order: Next.js (heavier but well-known), VitePress (lighter, Vue-based), or plain HTML+CSS+JS with a build script (slowest to develop, no framework dependencies).

## Extensibility Architecture

The site must scale to many learning tracks (Nutanix today; potentially VMware-deprecation, AWS, Cisco, ServiceNow, internal-process tracks tomorrow) and to many modules per track without anyone editing navigation, search, or routing code.

The rule that makes this work: **navigation is a derived view of content, never a source of truth.** A markdown file with valid front matter exists; therefore it appears in the sidebar, in search, in cert rollups, in prev/next, in homepage cards. There is no registry to update.

### Acceptance test (build for this, validate against this)

Before any feature is "done," all three of these must hold:

1. **Add a module to an existing track.** Drop `src/content/tracks/nutanix/11-files-deep-dive.md` with valid front matter. Run `bun run dev`. The module appears in the sidebar in correct order, in the search index, in cert-coverage rollups, in prev/next, on the track landing page. Zero other files touched.
2. **Add a brand-new track.** Drop `src/content/tracks/vmware/_track.yaml` plus one or more module files. The track appears on the homepage, gets its own `/vmware/` landing page, gets its own sidebar group, gets its own search scope. Zero other files touched.
3. **Add a new content kind** (say, "runbooks"). Drop `src/content/runbooks/*.md` plus one entry in the collection registry. Pages, nav group, and search inclusion follow. Exactly one file touched outside of `src/content/`.

### Layers

**1. Content collections with strict schemas.** Astro's content-collections API defines what every track and module must declare. Validation runs at build time. A new module either has the metadata to be navigated to, or the build fails loudly. Use Zod with `.strict()` so unknown front-matter keys are caught, not silently ignored.

**2. Dynamic routes only.** No per-module file in `src/pages/`. Three dynamic routes cover everything:

```
src/pages/[track]/index.astro           # track landing
src/pages/[track]/[module].astro         # module detail
src/pages/[track]/appendices/[slug].astro  # per-track appendices
```

Plus a small number of static pages (`/`, `/practice`, `/certs`, `/404`).

**3. A single nav builder.** `src/lib/nav.ts` is a pure function from collections to nav tree. It sorts, groups, applies status filters, computes prev/next. The sidebar component is dumb: it renders whatever the tree says. The nav builder has no knowledge of any specific track.

**4. A single link resolver.** `src/lib/links.ts` rewrites every relative `.md` cross-reference at build time by consulting collections. New tracks and new appendices need no changes here.

**5. A single search index builder.** `src/lib/search.ts` walks every collection, builds the FlexSearch index, faceted by track. New track, new content kind, automatically indexed.

### Required front matter

Every module file:

```yaml
track: nutanix          # which track this belongs to
order: 50               # sparse integer (10, 20, 30...) so inserts don't renumber
slug: dsf-storage       # URL slug (defaults to filename stem if absent)
status: published       # draft | published | archived (filters at build)
title: "DSF Deep Dive"
estimated_reading_time: 45
prerequisites: [01-hci-foundations, 02-nutanix-architecture]
key_terms: [Stargate, Cassandra, Curator, Medusa]
diagrams: [dsf-data-path, dsf-tier-flow]
cert_coverage:
  NCA: 5
  NCP-MCI: 30
sa_toolkit:
  objections: [obj-storage-perf, obj-data-locality]
  discovery: [disc-iops-baseline]
audience: [sa, cert]    # optional facets for nav/search modes
```

`order` is sparse on purpose. Use 10, 20, 30 for the initial set so a new module slipped between two existing ones gets `15` without renumbering anything.

Every track folder has `_track.yaml`:

```yaml
id: nutanix
name: Nutanix
subtitle: A field guide for BlueAlly Solutions Architects.
order: 1                # display order on the homepage
default_audience: sa
accent: cyan            # which color from the palette this track uses
hero_prompt: "isometric architectural, deep navy, electric blue, abstract, no text, no people"
cert_ladder:            # optional, only if track has certs
  - { id: NCA, name: "Nutanix Certified Associate", url: "..." }
  - { id: NCP-MCI, name: "...", url: "..." }
```

Homepage track cards, per-track theming, hero generation, and the cert page all derive from `_track.yaml`. Adding a track means adding this file plus module files. Nothing else.

### Sidebar UX with multiple tracks

Default to **track switcher in the header + per-track sidebar**, with an "all tracks" view on the homepage. Cleaner past 3-4 tracks, matches the "30 seconds between meetings" use case, scales well. Avoid a single mega-sidebar that lists every module of every track; it gets noisy fast.

### Theming per track

Each track may set `accent` to one of the existing palette tokens (`cyan`, `purple`, `gold`, `green`). The site's CSS uses a track-scoped CSS custom property so a track can shift its accent without forking styles. Resist the temptation to give a track its own typography or layout; the abstraction collapses the moment you do.

### Build-time validation

CI must fail the build for any of these:

- Front matter missing required fields, or containing unknown fields (Zod strict).
- Two modules within a track sharing the same `order` (warn allowed, but log).
- A relative `.md` link that does not resolve to a known content entry.
- A `prerequisites` reference to a slug that does not exist.
- A `diagrams` ID declared in front matter but not found in the body, or vice versa.
- A `cert_coverage` exam ID not in the track's `cert_ladder`.

These are cheap checks. Each one prevents a category of silent rot.

### What this costs upfront

Roughly half a day extra versus a hardcoded sidebar. Pays back the first time a new module is added (5 minutes vs an hour of hunting through nav, search, and cert rollups), and pays back massively the first time a second track lands.

### Build Commands

The repo's `README.md` should document:

```bash
bun install
bun run dev      # local dev server
bun run build    # production build to ./dist
bun run preview  # preview the production build locally
```

GitHub Actions workflow at `.github/workflows/deploy.yml` should:

1. Trigger on push to `main`.
2. Install dependencies with `bun install`.
3. Build with `bun run build`.
4. Deploy `./dist` to GitHub Pages.

## Repository Structure

The structure below reflects the multi-track, content-collection-driven design from the Extensibility Architecture section. There is no per-module file under `src/pages/`; routes are dynamic.

```
nutanix-site/                         # repo name nixfred/nutanix
├── README.md
├── CLAUDE.md
├── todo.md                           # phased build plan (see Reading Order)
├── package.json
├── astro.config.mjs                  # base: '/nutanix', site: 'https://nixfred.com'
├── tailwind.config.ts
├── tsconfig.json
├── .github/
│   └── workflows/
│       └── deploy.yml
├── public/
│   ├── images/                       # hero images, cached AI outputs, favicon
│   └── favicon.svg
├── curriculum/                       # raw source markdown (authored by Fred)
│   └── nutanix/                      # one folder per track
│       ├── 00-framework.md
│       ├── 01-hci-foundations.md
│       ├── ...
│       └── appendix-a-glossary.md
├── src/
│   ├── pages/
│   │   ├── index.astro               # home: lists all tracks
│   │   ├── practice.astro            # practice-mode dashboard
│   │   ├── certs.astro               # cert progress tracker
│   │   ├── 404.astro
│   │   └── [track]/
│   │       ├── index.astro           # track landing page
│   │       ├── [module].astro        # module detail (dynamic)
│   │       └── appendices/
│   │           └── [slug].astro      # per-track appendices (dynamic)
│   ├── content/
│   │   ├── config.ts                 # Zod schemas + collection registry
│   │   └── tracks/
│   │       └── nutanix/
│   │           ├── _track.yaml       # track metadata
│   │           ├── 01-hci-foundations.md
│   │           ├── ...
│   │           └── appendices/
│   │               └── a-glossary.md
│   ├── components/
│   │   ├── Layout.astro
│   │   ├── Sidebar.astro             # renders nav tree from src/lib/nav.ts
│   │   ├── TrackSwitcher.astro
│   │   ├── Wordmark.astro            # NutaNIX, with cyan NIX
│   │   ├── PathTreatment.astro       # /nix/nutanix mono treatment
│   │   ├── SearchPalette.tsx         # Cmd-K, FlexSearch, faceted by track
│   │   ├── Callout.astro             # 5 callout types
│   │   ├── Diagram.tsx               # rough.js renderer, lazy on scroll
│   │   ├── PracticeQuestion.tsx      # MCQ + NCX-style variants
│   │   ├── GlossaryTerm.tsx          # hover-card
│   │   ├── HeroImage.astro
│   │   ├── PrevNext.astro            # derived from nav tree
│   │   └── CertBadge.astro
│   ├── lib/
│   │   ├── collections.ts            # collection registry; new kinds added here
│   │   ├── nav.ts                    # pure: collections -> nav tree
│   │   ├── links.ts                  # resolves relative .md links
│   │   ├── search.ts                 # FlexSearch index builder
│   │   ├── parseCallouts.ts
│   │   ├── parseDiagram.ts
│   │   ├── parseQuestions.ts
│   │   └── glossaryIndex.ts
│   └── styles/
│       └── global.css                # Tailwind + CSS custom properties
└── scripts/
    ├── generate-heroes.ts            # AI image generation, results cached
    ├── verify-links.ts               # build-time link/reference validation
    ├── verify-frontmatter.ts         # schema strictness check
    └── add-track.ts                  # scaffolds a new track folder + _track.yaml
```

## GitHub & Deployment

1. **Create the repo**: `nixfred/nutanix` on GitHub. Public.
2. **Push the project** to `main`.
3. **Enable GitHub Pages**: Settings → Pages → Source: "GitHub Actions" (the deploy workflow handles the publish).
4. **Configure base path**: Astro's `astro.config.mjs` needs `base: '/nutanix'` and `site: 'https://nixfred.com'`. All internal links must resolve correctly with this base path.
5. **Verify**: after the first successful deploy, the site should be live at `https://nixfred.com/nutanix`.

If Fred wants a custom domain at the project level later (e.g., `nutanix.nixfred.com`), that's a configuration change in the repo's Pages settings plus a CNAME record. Out of scope for the initial build.

## Quality Bar

This site is meant to be world-class. The standard:

- **No AI-generated tells.** No "in conclusion," no "it's important to note that," no breathless adjectives, no hedging. The curriculum is written in a direct, professional voice; the site chrome must match.
- **No em-dashes anywhere.** Not in body text, not in UI strings, not in metadata. Use commas, colons, parentheses, or new sentences. Fred catches every one and finds them annoying.
- **No emoji** in the UI chrome. The curriculum doesn't use them; neither should the site.
- **Tight typography.** Don't let line lengths get long, don't let sections get visually monotonous. Vary the rhythm with diagrams, callouts, and well-placed code blocks.
- **Performance budget.** Lighthouse Performance ≥ 95. Total page weight under 500KB excluding hero images. Hero images lazy-loaded, optimized.
- **Accessibility.** Lighthouse Accessibility = 100. Real keyboard navigation. Real focus states. Real ARIA where needed for the interactive components.
- **Readable with JavaScript disabled.** The content is the product. Interactive features are enhancements. Sidebar navigation, search, and practice questions are JS-required (acceptable); the actual curriculum content must be readable as static HTML.

## Out of Scope (For Now)

- User accounts. All progress in `localStorage`. If multiple users on the same browser, they share state. Acceptable.
- Server-side anything. Pure static site.
- Video or audio. Text + images + diagrams only.
- Comments or community features.
- Translations. English only.
- Analytics. Skip Google Analytics, Plausible, etc. Fred can add later if he wants.

## What "Done" Looks Like

The reviewer (Fred or another BlueAlly SA) opens `nixfred.com/nutanix` on a phone, on a laptop, with WiFi, on a plane (cached). The reviewer:

1. Sees a home page that immediately communicates what this is and who it's for.
2. Browses to Module 5 (DSF). Finds the data-path diagram. Whiteboard-ready badge is visible. The diagram looks like something they could redraw in a customer meeting.
3. Scrolls to a practice question. Picks an answer. Sees the green/red feedback. Reads the trap explanation.
4. Hits `Cmd-K`. Searches "NearSync." Lands on the right section of Module 7.
5. Hovers a glossary term in the body. Sees the definition popover.
6. Clicks "Print this module." Gets a clean PDF for the plane ride.
7. Closes the laptop. The progress they made is still there tomorrow.

Build for that reviewer.

---

## Reading Order for Claude Code

1. This file (`CLAUDE.md`).
2. `todo.md` (the phased build plan; this is how the work gets done, in order, with gates).
3. `curriculum/nutanix/00-framework.md` (curriculum meta-spec; informs design decisions).
4. `curriculum/nutanix/01-hci-foundations.md` (representative module; understand the structure).
5. Skim modules `02` through `10` to confirm structural consistency.
6. Skim available appendices.
7. Begin Phase 1 of `todo.md`. Do not skip ahead.

## Build Discipline: Never One-Shot This

Do not attempt to scaffold, parse, render, deploy, and polish in a single pass. The site has too many interlocking concerns (extensibility, parsing, components, search, theming, images, performance, accessibility, deployment) for a one-shot build to land cleanly. One-shotting produces a demo that looks complete and breaks under the first edit.

**The discipline:**

- Work strictly in phases as defined in `todo.md`. Each phase has an explicit acceptance gate.
- **At every phase gate, stop and confirm with Fred** before starting the next phase. Show what was built, what was skipped, what is broken. Do not auto-advance.
- Commit at the end of each phase with a descriptive commit message. Phases are review checkpoints; commits make them reviewable.
- If a phase's acceptance gate fails, fix it before moving on. Do not paper over it with a TODO and proceed.
- The extensibility acceptance test (drop a file, it appears) is a gate at multiple phases, not a one-time check at the end.
- Hero images, glossary popovers, and print-PDF stylesheets come late on purpose. They are easy to retrofit; routing and parsing are not.
- If a phase reveals a flaw in CLAUDE.md, update CLAUDE.md before continuing. The spec is allowed to evolve; silent drift is not.

Build something Fred is proud to share with his peers. Take it one phase at a time.

# cognix-docs — Domain Inventory

The on-demand domain reference for `cognix-docs` — the public Cognix
documentation site (Starlight + Astro → `docs.cognix.dev`). This file is the
single source for the locked v1 docs design: the 9-section information
architecture, the generated CLI-reference contract, the narrative content
model, the API-reference scope, and theming / search / versioning.

It mirrors KMCP entry `7b330fd0` (Cognix Docs Site Build, GTM-build campaign
thread C, locked 2026-05-22). **Everything here is locked.** Encode it; do not
re-derive it. If reality contradicts the design, surface the contradiction —
do not patch the design to fit. Design doc:
`docs/cognix/specs/2026-05-22-gtm-build-campaign/docs-site-build/design.md`.

## 1. The 9-section IA → Starlight structure

The locked v1 IA has **9 sections**. Each section becomes one Starlight sidebar
group and one content directory under `src/content/docs/`. A site-wide home
page (`index.mdx`) sits at the collection root. The sidebar is configured in the
`starlight()` integration block in `astro.config.mjs`.

| #   | Section       | Content directory  | Sidebar group style | v1 launch state             |
| --- | ------------- | ------------------ | ------------------- | --------------------------- |
| —   | Home          | `index.mdx` (root) | n/a — splash page   | Complete                    |
| 1   | Start Here    | `start-here/`      | Manual `items`      | Launch-blocking — complete  |
| 2   | Concepts      | `concepts/`        | Manual `items`      | Launch-blocking — complete  |
| 3   | Desktop App   | `desktop-app/`     | Manual `items`      | Launch-blocking — complete  |
| 4   | CLI Reference | `reference/cli/`   | `autogenerate`      | Launch-blocking — generated |
| 5   | Configuration | `configuration/`   | Manual `items`      | Launch-blocking — complete  |
| 6   | Integrations  | `integrations/`    | Manual `items`      | Launch-blocking — complete  |
| 7   | Guides        | `guides/`          | Manual `items`      | Seeded — 2-3 guides         |
| 8   | API Reference | `reference/api/`   | Manual `items`      | Thin — hand-written         |
| 9   | Contributing  | `contributing/`    | Manual `items`      | Launch-blocking — complete  |

Notes on directory layout:

- `reference/cli/` and `reference/api/` are **two directories nested under a
  shared `reference/` parent**. They are distinct sections (4 and 8) with
  different content models — CLI is generated, API is hand-written.
- Every section directory carries an `index.md` / `index.mdx` landing page.
- A page's URL is its path under `src/content/docs/` minus the extension.

### Launch gate — what ships at v1

**All 9 sections are present at launch** — none is omitted. The depth varies:

- **Launch-blocking (sections 1-6 and 9)** — ship complete enough to meet the
  v1 docs bar. These are the sections a v1 user must be able to rely on.
- **Guides (7)** — ships **seeded with 2-3 guides**, not exhaustive. More
  guides are added downstream over time.
- **API Reference (8)** — ships **thin**: a hand-written overview only (see
  § 4). Present, but deliberately shallow at v1.

## 2. CLI Reference — generated-content contract

Section 4 (`reference/cli/`) is **not hand-authored**. It is the consumer end
of a generated-content pipeline owned by thread A.

- **Producer** — thread A's `tools/docsgen/` generator emits **Starlight-ready
  markdown** as the `cli-docs` asset. Each generated page carries:
  - per-page `title` frontmatter (so Starlight renders the page and sidebar
    label without manual config), and
  - a **"generated — do not hand-edit" banner** at the top of the page body.
- **Consumer** — the `cognix-docs` `ingest` workflow drops the `cli-docs` asset
  into `src/content/docs/reference/cli/` as a **dumb file-drop**: no
  post-processing, no transformation, no patching. Files land as-emitted.
- **Sidebar** — the `reference/cli` subtree uses Starlight's **`autogenerate`**
  sidebar group. New generated pages appear in the sidebar with no config
  change; per-page order is controlled by `sidebar.order` frontmatter the
  generator may emit.
- **Marking** — generated files are marked `linguist-generated` in
  `.gitattributes` so GitHub treats them as generated (collapsed in diffs,
  excluded from language stats).

This contract **refines thread A**: thread A's original `docsgen` spec said only
"Markdown" — thread C, as the consumer, defines the concrete format
(Starlight-ready: `title` frontmatter + banner). A one-line pointer was added to
thread A's design doc to record the refinement.

**Hard rule:** never hand-edit a file under `reference/cli/`. Edits are
overwritten on the next ingest. To change CLI-reference content, fix the
generator (`tools/docsgen/`).

## 3. Narrative content model

The hand-written sections (everything except the generated CLI reference) use a
plain content model:

- **Format** — Markdown / MDX files inside the section directories.
  - **Plain Markdown (`.md`) by default** — prose, headings, lists, links,
    fenced code, tables.
  - **MDX (`.mdx`) only where a page needs a Starlight component** — asides,
    cards, tabs, steps, code blocks generated from a variable/file.
- **No custom interactive islands at v1** — **Starlight built-in components
  only** (`Aside`, `Card`, `CardGrid`, `Tabs`, `Steps`, `Code`, `LinkCard`,
  `FileTree`, `Badge`, `Icon`). No React / Vue / Svelte islands, no bespoke
  widgets. This keeps Starlight upgradeable.

### Scaffolded here vs authored downstream

Thread C (and the build steps that execute it) scaffold the **structure**, not
the prose:

| Scaffolded by the docs build (thread C / this skill) | Authored downstream (Tier-2 content stream) |
| ---------------------------------------------------- | ------------------------------------------- |
| The 9 section directories                            | The narrative body copy for each page       |
| Per-section `index` landing pages                    | Concept explanations, guide walkthroughs    |
| Frontmatter conventions (`title`, `description`, …)  | Desktop-app and configuration prose         |
| Sidebar IA in `astro.config.mjs`                     | —                                           |

The **narrative copy itself is a downstream Tier-2 content stream** — AI-drafted,
Felix-reviewed — and is distinct from thread F's marketing copy. When
scaffolding, leave page bodies as a stub or TODO; do not write the prose.

## 4. API Reference (section 8)

- **v1 scope** — a **hand-written thin overview** of the Cognix sidecar's
  HTTP + WebSocket API. It covers, at an overview level:
  - **auth** — how a client authenticates to the sidecar,
  - **endpoint shape** — the shape of the HTTP API surface,
  - **WS events** — the WebSocket event model.
- **Not in v1** — auto-generation from an OpenAPI spec. This is **deferred**.
- **Concrete deferral trigger** — auto-generation is reconsidered when, and only
  when, **the sidecar gains a maintained OpenAPI spec**. Until that exists, the
  API reference stays hand-written and thin.

The API Reference is one of the 9 launch sections — it is present at v1, just
deliberately shallow.

## 5. Theming, search, versioning

### Theming — themed, not forked

The brand is applied entirely through **Starlight's own theming surface**. There
is no theme fork and no bespoke redesign — Starlight stays upgradeable.

- **Brand colors** — `--sl-color-*` CSS custom properties, seeded from the
  **vendored Cognix brand palette**.
- **Fonts** — Inter (body) + JetBrains Mono (monospace).
- **Logo** — the Cognix logo, via Starlight's `logo` config.
- **Header** — a **light custom header** that links back to `cognix.dev`.
- **Dark mode** — Starlight's built-in light/dark support; no custom toggle.

#### Vendored brand-palette sync note

The brand palette is **vendored** — `cognix-docs` holds its **own copy** of the
same brand-color subset. There are **three vendored copies** of this subset
across the org: the original, the copy thread B vendors into `cognix-web`, and
this third copy in `cognix-docs`. All three are **hand-synced** on the rare
occasion the brand changes. There is no shared package — keep the copy in sync
manually when the brand palette is updated.

### Search — Pagefind

Search is **Starlight's built-in Pagefind**. No extra dependency, no custom
search config at v1. Pagefind indexes the built site at build time; exclude a
page with `pagefind: false` frontmatter.

### Versioning — unversioned at v1

The site is **unversioned at v1** — a single current version of the docs, no
version switcher. The `starlight-versions` plugin is **deferred on a concrete
trigger** (carried forward from the Website & Docs decision, `3fe3a07f`). Do not
add versioning machinery until that trigger fires.

## Build sequencing (context)

Thread C is the _plan_; its build steps run in this order — thread D's GTM
skill-layer (or a decomposition) executes it:

1. Scaffold the Starlight repo + brand theme + sidebar IA.
2. Scaffold the 9 section directories with index pages + frontmatter
   conventions.
3. Wire the CLI-reference ingest target + `autogenerate` sidebar group.
4. Narrative copy authored downstream (ongoing Tier-2 content stream).

Steps 1-3 are decomposition + execution work; step 4 is the downstream content
stream.

## Cross-thread relationships

- **C-1 — refines thread A** (`694cbc7d`): `docsgen` emits Starlight-ready
  markdown; the `cognix-docs` ingest stays a dumb file-drop. Thread A's design
  doc was updated with a one-line pointer to this refinement.
- **C-2 → thread D**: the GTM build skill-layer executes this plan.
- **C-3**: the narrative docs copy is a downstream Tier-2 content stream —
  distinct from thread F's marketing copy.

Related KMCP entries: Website & Docs (`3fe3a07f`, extended by thread C), thread B
(`08111a24`, vendored-palette sibling), thread A (`694cbc7d`, depended on),
sidecar HTTP+WS API (`66130134`, the API Reference subject).

---
name: gtm-docs
description: Use when working on the cognix-docs documentation site — building or editing Starlight sections, applying the brand theme, wiring the generated CLI reference, scaffolding narrative pages, or configuring the sidebar IA. Provides the Starlight + Astro tech patterns (content collections, sidebar config, built-in components, theming, search) and routes to the docs domain inventory. Not for the cognix-web marketing site, not for writing the narrative docs copy itself (downstream Tier-2 content), and not for the desktop app.
---

# gtm-docs — cognix-docs Domain + Tech Skill

The combined domain + tech skill for `cognix-docs` — the public Cognix
documentation site (Starlight + Astro → `docs.cognix.dev`). The
`cognix-workflow` pipeline injects this skill as context during decompose /
execute of docs build steps. It carries the Starlight/Astro tech patterns (this
file) and points to the docs domain inventory (`references/docs-domain.md`).

**Announce at start:** "Using gtm-docs to [build / edit / theme / wire] [the docs target]."

## MANDATORY — run BEFORE any other action this session

These are commands, not hints. Run them before editing any file, before
answering a structural question, before scaffolding anything.

1. **Read the domain inventory** — `Read references/docs-domain.md`. It holds
   the locked 9-section IA, the CLI-reference contract, the narrative content
   model, theming detail, the API-reference scope, and versioning. The tech
   patterns below are meaningless without the domain frame.
2. **Confirm the stack** — run `cat package.json` in the repo root. Verify the
   installed `@astrojs/starlight` and `astro` versions before applying any
   pattern below. If a major version differs from § Stack baseline, stop and
   re-check current Starlight docs — do not apply a stale pattern.
3. **Locate the work** — the build sequence and current step live in
   `docs/site-build-roadmap.md`; the `site-build` skill navigates it. This
   skill is the _how_, not the _what's next_.

## Hard rules

<HARD-GATE>
**Do NOT re-decide the locked design.** The 9-section IA, the themed-not-forked
brand approach, the CLI-reference file-drop contract, the narrative content
model, and unversioned-at-v1 are all locked in `references/docs-domain.md`
(KMCP `7b330fd0-b8f8-461b-bd70-c67b3fbbea4a`, GTM-build campaign thread C). Encode them; never re-derive
them. If reality contradicts the design, surface the contradiction — do not
patch the design to fit.

**Do NOT add custom interactive islands at v1.** Starlight built-in components
only (`Aside`, `Card`, `CardGrid`, `Tabs`, `Steps`, `Code`, `LinkCard`,
`FileTree`, `Badge`, `Icon`). No React/Vue/Svelte islands, no bespoke widgets.
This keeps Starlight upgradeable.

**Do NOT fork or hand-redesign Starlight.** Brand is applied through Starlight's
own theming surface — `--sl-color-*` custom properties, `customCss`, `logo`,
`components` config — never by overriding internal layout or patching the
theme package. Themed, not forked.

**Do NOT hand-edit the generated CLI reference.** `src/content/docs/reference/cli/`
is a dumb file-drop target for the `cli-docs` ingest. Files there carry a
"generated — do not hand-edit" banner and are marked `linguist-generated`.
Never edit them in place; fix the generator (`tools/docsgen/`) instead.

**Do NOT write the narrative docs copy.** This skill scaffolds section
directories, index pages, and frontmatter conventions. The prose for concepts /
guides / desktop-app pages is a downstream Tier-2 content stream, Felix-reviewed.
Scaffold the structure; leave the body copy as a stub or TODO.
</HARD-GATE>

## Why this skill exists

`cognix-docs` is genuinely new tech for the Cognix org — the main repo has no
Astro or Starlight skill, and Starlight has its own content model, config
surface, and component set that differ from a hand-rolled site. Without this
skill, a fresh decompose/execute agent would invent ad-hoc patterns: a custom
sidebar, hand-built cards, MDX everywhere, a forked theme. This skill encodes
the one correct set of Starlight conventions plus the locked `cognix-docs`
domain design so every docs build step lands consistently.

It is **combined domain + tech in one skill — per thread C design, do NOT
split.** The tech patterns (this file) and the domain inventory
(`references/docs-domain.md`) ship together because a docs build step always
needs both.

## Stack baseline

Verified against the `cognix-docs` repo, 2026-05-22. Re-check via step 2 of the
MANDATORY block before applying.

| Package                              | Version   | Notes                                                                                                  |
| ------------------------------------ | --------- | ------------------------------------------------------------------------------------------------------ |
| `astro`                              | `^6.3.1`  | Astro 6 — content layer is stable; `src/content.config.ts` is the collection entry point.              |
| `@astrojs/starlight`                 | `^0.39.2` | Starlight 0.39 — pre-1.0, minor releases can shift config; check release notes on bump.                |
| `@astrojs/check` + `typescript`      | —         | `npm run check` runs `astro check` (type + content-schema validation).                                 |
| `prettier` + `prettier-plugin-astro` | —         | `npm run format` / `npm run format:check`. There is **no lint script** — use `format:check` + `check`. |
| Node                                 | `>=22`    | `engines` field; `.nvmrc` pins it.                                                                     |

Primary sources (re-fetch before a major Starlight bump):

- Starlight docs — `https://starlight.astro.build/` (configuration, components,
  CSS-and-tailwind, site-search, frontmatter reference).
- Astro docs — `https://docs.astro.build/` (content collections / content layer).

## Starlight + Astro tech patterns

### Content collections — where pages live

All docs pages live in **one** content collection, `docs`, rooted at
`src/content/docs/`. The collection is declared once in `src/content.config.ts`:

```ts
// src/content.config.ts
import { defineCollection } from 'astro:content';
import { docsLoader } from '@astrojs/starlight/loaders';
import { docsSchema } from '@astrojs/starlight/schema';

export const collections = {
	docs: defineCollection({ loader: docsLoader(), schema: docsSchema() }),
};
```

- One **directory per IA section** under `src/content/docs/` (see
  `references/docs-domain.md` for the 9-section list and exact directory names).
- Each section directory gets an **`index.md` / `index.mdx`** landing page.
- A page's **URL** is its path under `src/content/docs/` minus the extension —
  `src/content/docs/concepts/pipelines.md` → `/concepts/pipelines/`.
- Every page carries frontmatter validated by `docsSchema()` — at minimum
  `title`; commonly `description`. `astro check` fails the build on a schema
  violation, so frontmatter is not optional.

### Page frontmatter conventions

```md
---
title: Pipelines
description: How Cognix pipelines orchestrate a build.
---
```

| Field              | Use                                                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------------------------------- |
| `title`            | Required. Drives the `<h1>`, sidebar label, and `<title>`.                                                          |
| `description`      | Strongly recommended — SEO meta + search snippet.                                                                   |
| `sidebar`          | Optional object — `order`, `label` override, `badge`. Use `order` to sequence pages within an `autogenerate` group. |
| `template: splash` | Landing-page layout (no sidebar/TOC). Used by `index.mdx` only.                                                     |
| `hero`             | Splash hero block — `title`, `tagline`, `image`, `actions`. Home page only.                                         |
| `pagefind: false`  | Excludes a page from search. Apply to thin redirect/stub pages.                                                     |
| `editUrl: false`   | Suppresses the edit link — apply on the generated CLI reference pages.                                              |

### Sidebar config — `astro.config.mjs`

The sidebar is the IA made navigable. It is configured in the `starlight()`
integration block in `astro.config.mjs`. Two group styles:

```js
// astro.config.mjs
starlight({
	title: 'Cognix Docs',
	sidebar: [
		// Manual group — explicit order, hand-curated narrative sections.
		{
			label: 'Start Here',
			items: [
				{ slug: 'start-here/installation' },
				{ slug: 'start-here/quickstart' },
			],
		},
		// Autogenerate group — Starlight builds entries from a directory.
		// REQUIRED for the generated CLI reference (file-drop target).
		{
			label: 'CLI Reference',
			autogenerate: { directory: 'reference/cli' },
		},
	],
});
```

- **Manual `items`** — for hand-authored narrative sections where order is
  meaningful. Use `{ slug: '...' }` (validated against the collection) over raw
  `link` strings for internal pages.
- **`autogenerate: { directory }`** — Starlight derives entries from the
  directory tree. **This is the locked pattern for `reference/cli/`** — the
  ingest drops files in, the sidebar updates with no config change. Order within
  an autogenerate group is controlled by per-page `sidebar.order` frontmatter.
- `collapsed: true` on a group ships it collapsed by default.
- The full 9-section sidebar shape is specified in `references/docs-domain.md` —
  build the `sidebar` array to match it.

### Markdown vs MDX — when to reach for `.mdx`

| Use                                                                    | Extension       |
| ---------------------------------------------------------------------- | --------------- |
| Prose, headings, lists, links, fenced code, tables — no component      | `.md` (default) |
| Page needs a Starlight component (`Aside`, `Card`, `Tabs`, `Steps`, …) | `.mdx`          |
| Home / landing page with `hero` + `CardGrid`                           | `.mdx`          |

**Default to `.md`.** Reach for `.mdx` only when a page actually renders a
component. Do not blanket-convert sections to MDX — plain Markdown is lighter
and the locked content model says "Markdown by default, MDX where a component
is needed."

### Starlight built-in components

Imported from `@astrojs/starlight/components` inside `.mdx` files. **These are
the only components allowed at v1** — no custom islands.

```mdx
---
title: Configuring a project
---

import {
	Aside,
	Card,
	CardGrid,
	Tabs,
	TabItem,
	Steps,
	Code,
	LinkCard,
} from '@astrojs/starlight/components';

<Aside type="tip">Asides come in note / tip / caution / danger.</Aside>

<Steps>
	1. Install the CLI. 2. Run `cognix init`. 3. Open the desktop app.
</Steps>

<CardGrid>
	<Card title="Pipelines" icon="rocket">
		Short feature blurb.
	</Card>
	<Card title="Knowledge" icon="open-book">
		Short feature blurb.
	</Card>
</CardGrid>

<LinkCard title="Quickstart" href="/start-here/quickstart/" />
```

| Component           | Use for                                                                                                                                 |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `Aside`             | Callouts — `note`, `tip`, `caution`, `danger`.                                                                                          |
| `Card` / `CardGrid` | Feature grids, section landing pages. `stagger` prop on the grid for offset layout.                                                     |
| `Tabs` / `TabItem`  | Alternative paths — OS, package manager. `syncKey` keeps tabs in sync across the page.                                                  |
| `Steps`             | Ordered procedures — wraps an `<ol>`, renders numbered step markers.                                                                    |
| `Code`              | Code blocks generated from a variable/file (when a fenced block is not enough). Normal code uses plain fenced blocks (Expressive Code). |
| `LinkCard`          | Prominent single navigational link.                                                                                                     |
| `FileTree`          | Directory-structure illustrations.                                                                                                      |
| `Badge` / `Icon`    | Inline status badges and icons.                                                                                                         |

### Theming — themed, not forked

Brand is applied entirely through Starlight's theming surface. **No layout
overrides, no theme fork.**

1. **Brand CSS** — a custom stylesheet (e.g. `src/styles/brand.css`) registered
   via `customCss` in `astro.config.mjs`. It overrides `--sl-color-*` custom
   properties from the vendored Cognix brand palette, and sets `--sl-font` /
   `--sl-font-mono` to Inter / JetBrains Mono.

   ```css
   /* src/styles/brand.css */
   :root {
   	--sl-color-accent-low: /* … */;
   	--sl-color-accent: /* … */;
   	--sl-color-accent-high: /* … */;
   	--sl-font: 'Inter', system-ui, sans-serif;
   	--sl-font-mono: 'JetBrains Mono', ui-monospace, monospace;
   }
   :root[data-theme='light'] {
   	/* light-mode overrides */
   }
   ```

2. **Dark mode** is built in — Starlight ships light + dark; theme via the
   `:root[data-theme='light']` / default-dark selector split. Do not build a
   custom toggle.
3. **Logo** — `logo: { src, alt, replacesTitle }` in the integration config.
4. **Header link back to `cognix.dev`** — a light header customization. Prefer
   the `social` array or a config-level link; only override the `Header`
   component slot via `components` config if a plain link cannot do it.
5. The brand palette is **vendored** (a hand-synced copy of the same subset
   `cognix-web` uses). On the rare brand change, the copy is updated by hand —
   see `references/docs-domain.md` for the sync note.

### Search — Pagefind

Search is **Starlight's built-in Pagefind** — `pagefind: true` (the default; no
extra dependency). It indexes the built site at build time. To exclude a page,
set `pagefind: false` in its frontmatter. No search config beyond that at v1.

### Validation — `astro check`

`npm run check` runs `astro check` — it type-checks and validates every page's
frontmatter against `docsSchema()`. Run it after any content or config change.
A schema violation (missing `title`, bad `sidebar` shape) fails it. There is no
`npm run lint`; the quality gate is `format:check` + `check`.

## Routing to the domain inventory

The **docs DOMAIN** — the 9-section IA with exact directory names, the
CLI-reference generated-content contract, the narrative content model, the
theming detail, the API-reference scope, and versioning — is in
`references/docs-domain.md`. It is the on-demand domain reference for this
skill.

**Read `references/docs-domain.md` for any of:**

- the exact 9 section directory names and their order;
- which sections are launch-blocking vs seeded-thin at v1;
- the CLI-reference file-drop contract and `linguist-generated` marking;
- the narrative content model (what is scaffolded here vs authored downstream);
- the vendored brand-palette sync note;
- the API-reference v1 scope and the OpenAPI deferral trigger;
- the versioning decision (unversioned at v1).

Do not re-derive any of the above — it is locked. This SKILL.md does not inline
the inventory; the reference file is the single source.

## Rationalization table — STOP signals

| Thought                                                             | Reality                                                                                                             |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| "I know Starlight, I'll skip reading the domain reference"          | The IA, directory names, and contracts are project-specific and locked. Read `references/docs-domain.md` first.     |
| "A custom Svelte card would look nicer than `<Card>`"               | v1 is Starlight built-ins only. A custom island breaks the no-fork rule and Starlight upgradeability. Use `<Card>`. |
| "I'll just tweak this generated CLI page, it's a one-line fix"      | The CLI reference is a file-drop; hand-edits are overwritten on next ingest. Fix `tools/docsgen/` instead.          |
| "MDX everywhere is simpler than deciding per page"                  | The locked model is Markdown by default, MDX only where a component renders. Blanket MDX violates it.               |
| "The theme would look better with a custom layout override"         | Themed, not forked. Brand goes through `--sl-color-*` + `customCss`. No layout overrides.                           |
| "I'll write the concept-page prose while I'm scaffolding"           | Narrative copy is downstream Tier-2, Felix-reviewed. Scaffold structure + frontmatter; stub the body.               |
| "Versioning would be good to add now"                               | Unversioned at v1 is locked. `starlight-versions` is deferred on a concrete trigger.                                |
| "I'll apply the pattern from memory, the versions are close enough" | Starlight is pre-1.0; minor releases shift config. Run the `package.json` check; re-fetch docs on a mismatch.       |

## Red flags — back up immediately

- About to create a `.svelte` / `.jsx` component for a docs page → STOP, use a
  Starlight built-in.
- About to edit a file under `src/content/docs/reference/cli/` → STOP, that is
  generated; fix the generator.
- About to override a Starlight layout/theme internal → STOP, theme via
  `--sl-color-*` + `customCss` only.
- About to write narrative body copy → STOP, scaffold structure only.
- About to apply a Starlight config pattern without checking `package.json` →
  STOP, verify the installed version first.
- About to skip the MANDATORY block (domain reference, version check) → STOP,
  run it.

## Common mistakes

| Mistake                                      | Fix                                                                                     |
| -------------------------------------------- | --------------------------------------------------------------------------------------- |
| New content collection per section           | One `docs` collection; one directory per section under `src/content/docs/`.             |
| Manual sidebar entries for the CLI reference | `autogenerate: { directory: 'reference/cli' }` — the ingest must not need config edits. |
| `.mdx` for plain-prose pages                 | `.md` is the default; `.mdx` only when a component renders.                             |
| Hand-built cards/callouts                    | `Card` / `CardGrid` / `Aside` from `@astrojs/starlight/components`.                     |
| Theme fork or layout override                | `--sl-color-*` + `customCss` + `logo` + `components` config.                            |
| Missing `title` frontmatter                  | `astro check` fails the build — every page needs `title`.                               |
| Editing generated CLI pages                  | Fix `tools/docsgen/`; the subtree is a dumb file-drop.                                  |
| Looking for `npm run lint`                   | There is none — gate on `npm run format:check` + `npm run check`.                       |

## Knowledge integration

This skill encodes KMCP entry `7b330fd0-b8f8-461b-bd70-c67b3fbbea4a` (Cognix Docs Site Build, GTM-build
campaign thread C) — the locked design. It does not query or write KMCP at
runtime; the locked design is mirrored into `references/docs-domain.md`. If the
design changes, the change flows through a thread-C re-open and KMCP update,
then into the reference file — not via ad-hoc edits to this skill.

## References

- `references/docs-domain.md` — the docs DOMAIN inventory: 9-section IA,
  CLI-reference contract, narrative model, theming detail, API reference,
  versioning. The on-demand domain reference; read it in the MANDATORY block.
- Starlight docs — `https://starlight.astro.build/` — primary spec for
  configuration, components, theming, and search. Re-fetch before a major bump.
- Astro docs — `https://docs.astro.build/` — content collections / content layer.

## Scope NOT in this skill

- The `cognix-web` marketing site (separate repo, separate skill).
- Writing the narrative docs copy itself — downstream Tier-2 content stream.
- The Cognix desktop app (Tauri/Svelte) and the Go core.
- The `cli-docs` generator `tools/docsgen/` internals — this skill consumes its
  output as a file-drop; it does not author the generator.
- Build-sequence navigation / "what's next" — that is the `site-build` skill.

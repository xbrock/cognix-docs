# cognix-docs Documentation Site — Build Roadmap

> Status: active (until launch-ready)
> Seeded: 2026-05-22 from GTM-build campaign thread C (KMCP `7b330fd0`)
> Maintained by: the `site-build` orchestration skill

This document is the single source of truth for `cognix-docs` documentation-site
build progress. It is durable and committed — the `site-build` skill reads it to
decide what to do next and writes status changes back as work completes. The
build sequence below was decided in campaign thread C (`docs-site-build`,
KMCP `7b330fd0`) and is seeded here verbatim; this roadmap tracks the sequence,
it does not re-decide it.

`cognix-docs` is the public Starlight/Astro documentation site
(→ `docs.cognix.dev`). It is a parallel doc to the `cognix-web` marketing-site
roadmap — same shape, distinct surface.

## Status legend

- `pending` — ready to start, no unmet dependency
- `in_progress` — actively being worked
- `blocked` — a dependency or manual gate is not yet done
- `done` — verified complete, with evidence

## Build sequence

The build walks one mostly-linear sequence.

### 1. Brand theme — `pending`

Apply the Cognix brand to Starlight via its theming surface — `--sl-color-*`
custom properties seeded from the vendored brand palette, Inter + JetBrains
Mono, the Cognix logo, a light custom header linking back to `cognix.dev`, and
dark mode. Themed, not forked — Starlight stays upgradeable.

### 2. 9-section IA scaffold — `blocked`

Depends on item 1. Starlight sidebar groups + one content directory per section
under `src/content/docs/`: `start-here/`, `concepts/`, `desktop-app/`,
`reference/cli/` (generated, fenced), `configuration/`, `integrations/`,
`guides/`, `reference/api/`, `contributing/`, plus `index.mdx`. The sidebar is
configured in `astro.config.mjs`. Includes section index pages + frontmatter
conventions.

### 3. Wire the CLI-reference ingest target + `autogenerate` sidebar — `blocked`

Depends on item 2. The `reference/cli/` subtree is a dumb file-drop target for
the `cli-docs` ingest; that subtree uses Starlight's `autogenerate` sidebar;
files are marked `linguist-generated`.

### 4. Seed 2–3 guides + thin API reference — `blocked`

Depends on item 2. Guides ships 2-3 seeded; API Reference is a thin hand-written
overview of the sidecar HTTP+WS API.

### 5. Verify & tune site-health automation — `blocked`

The site-health automation (PR-gate CI + scheduled workflows) is _delivered_ by
the GTM epic itself — this item is a verify-and-tune-against-the-real-site step,
not a build-from-scratch step.

### 6. Launch-ready — `blocked`

Final gate. The docs site is verified complete and ready to launch.

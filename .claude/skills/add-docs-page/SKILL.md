---
name: add-docs-page
description: Use when scaffolding a new page in an existing cognix-docs documentation section — placing the file in the correct section directory under src/content/docs/, applying that section's frontmatter conventions, and wiring it into the sidebar. A lightweight on-demand maintenance task — no pipeline ceremony. Not for writing the narrative docs copy itself (downstream Tier-2 content stream — this skill scaffolds structure + frontmatter, not prose), not for the generated reference/cli/ subtree (a do-not-hand-edit file-drop — fix tools/docsgen/ instead), and not for the cognix-web marketing site.
---

# Add a cognix-docs Section Page

A lightweight, on-demand maintenance task: scaffold one new page inside an
**existing** `cognix-docs` section, with correct IA placement, frontmatter, and
sidebar wiring. This is a quick task — no pipeline, no review gates. Scaffold
the structure and stop; the prose comes later.

**Announce at start:** "Using add-docs-page to scaffold [page] in [section]."

## Hard rules

<HARD-GATE>
**Do NOT write the narrative body copy.** This task scaffolds the file,
frontmatter, and sidebar entry. The prose for concept / guide / desktop-app
pages is a downstream Tier-2 content stream, Felix-reviewed. Leave the page body
as a short stub or `TODO` and stop.

**Do NOT add a page under `src/content/docs/reference/cli/`.** That subtree is a
generated-content file-drop — files there carry a "generated — do not hand-edit"
banner and are overwritten on the next ingest. To add CLI-reference content, fix
the generator (`tools/docsgen/`), not this directory.

**Do NOT create a new section.** This task adds a page to one of the **existing
9 sections**. A new section is an IA change — that is a `gtm-docs` / design
decision, not a maintenance task.

**Do NOT touch the cognix-web marketing site** — different repo, different skill.
</HARD-GATE>

## Before you start

1. **Read `../gtm-docs/references/docs-domain.md`** — it holds the locked
   9-section IA, the exact section directory names, which sidebar style each
   section uses (manual `items` vs `autogenerate`), and the frontmatter
   conventions. Do not guess directory names — read them.
2. **Confirm the target section exists** — run
   `ls src/content/docs/` and check the section directory is present. If it is
   not yet scaffolded, this is site-build work, not a page add — stop and say so.

## Steps

1. **Pick the target section.** Choose one of the 9 IA sections (`start-here/`,
   `concepts/`, `desktop-app/`, `configuration/`, `integrations/`, `guides/`,
   `reference/api/`, `contributing/` — and `reference/cli/`, which is
   off-limits per the HARD-GATE). If the section is ambiguous, ask Felix.

2. **Create the page file** in that section directory under
   `src/content/docs/<section>/`. Name it kebab-case; the filename (minus
   extension) becomes the URL slug — `concepts/pipelines.md` → `/concepts/pipelines/`.
   Use **`.md` by default**; use `.mdx` only if the page will render a Starlight
   built-in component (`Aside`, `Card`, `Tabs`, `Steps`, …).

3. **Apply the section's frontmatter conventions.** Every page needs `title`
   (drives the `<h1>`, sidebar label, and `<title>`); add `description` (SEO +
   search snippet). Match whatever extra conventions the section already uses —
   read a sibling page in the same directory and mirror it. Leave the body as a
   stub:

   ```md
   ---
   title: Pipelines
   description: How Cognix pipelines orchestrate a build.
   ---

   <!-- TODO: narrative copy — downstream Tier-2 content -->
   ```

4. **Wire the sidebar if the section uses a manual sidebar.** Check the
   section's group in the `sidebar` array in `astro.config.mjs`:
   - **Manual `items` group** → add a `{ slug: '<section>/<page>' }` entry in
     the intended order. Prefer `{ slug }` over a raw `link` for internal pages.
   - **`autogenerate` group** (`reference/cli/` only) → no edit needed; the page
     appears automatically. But note `reference/cli/` is off-limits anyway.

5. **Verify.** Run `npm run check` (`astro check` — type + frontmatter-schema
   validation; a missing `title` or bad `sidebar` shape fails it). Then run
   `npm run format` and `npm run format:check`. There is no `lint` script.

6. **Stop short of the prose.** The page exists with valid frontmatter and a
   sidebar entry. The narrative copy is downstream Tier-2 work — do not write it.

## Rationalization table — STOP signals

| Thought                                                         | Reality                                                                                              |
| --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| "I'll write the body copy too while I'm here"                   | Narrative prose is downstream Tier-2, Felix-reviewed. Stub the body and stop.                        |
| "This CLI command needs a reference page — I'll add one"        | `reference/cli/` is a generated file-drop. Add it via `tools/docsgen/`, never by hand.               |
| "I'll guess the section directory name"                         | Read `../gtm-docs/references/docs-domain.md` — directory names are locked, not guessable.            |
| "This content doesn't fit the 9 sections — I'll make a new one" | A new section is an IA change, not a maintenance task. Route through `gtm-docs` / a design call.     |
| "`.mdx` is safer, I'll use it everywhere"                       | `.md` is the default; `.mdx` only when the page renders a Starlight component.                       |
| "The sidebar updates itself"                                    | Only `autogenerate` groups do. Manual `items` sections need an explicit entry in `astro.config.mjs`. |

## Red flags — back up immediately

- About to write narrative prose → STOP, scaffold structure only; the body is a stub.
- About to create a file under `reference/cli/` → STOP, that subtree is generated.
- About to invent a section directory name → STOP, read `docs-domain.md`.
- About to add a 10th section → STOP, that is an IA design change, not this task.
- About to claim done without `npm run check` + `format:check` → STOP, run them.

## Scope NOT in this task

- Writing the narrative docs copy — downstream Tier-2 content stream.
- Adding pages to the generated `reference/cli/` subtree — fix `tools/docsgen/`.
- Creating a new IA section — a `gtm-docs` / design decision.
- The `cognix-web` marketing site.

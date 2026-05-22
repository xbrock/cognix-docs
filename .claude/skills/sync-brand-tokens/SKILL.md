---
name: sync-brand-tokens
description: Use when the upstream Cognix brand changed and cognix-docs's Starlight `--sl-color-*` theme needs re-syncing — hand-syncs the vendored brand subset (palette + Inter / JetBrains Mono) into the Starlight theme CSS via Starlight's own `--sl-color-*` theming surface. A lightweight on-demand maintenance task — no pipeline ceremony. Not for forking or redesigning the Starlight theme (themed-not-forked — Starlight stays upgradeable), and not for the cognix-web marketing site's Tailwind-v4 `tokens.css` (different repo, different transform).
---

# Sync Brand Tokens — docs variant

A lightweight, on-demand, **task-shaped** maintenance skill. When the upstream
Cognix brand changes, hand-sync the vendored brand subset into this repo's
Starlight theme CSS. No pipeline, no decomposition, no ceremony — read, edit,
verify, done.

**Announce at start:** "Using sync-brand-tokens to re-sync the docs Starlight theme."

## This is the docs variant

`sync-brand-tokens` exists in **both** GTM repos and they are **not identical**:

| Repo                       | Target                             | Transform                                        |
| -------------------------- | ---------------------------------- | ------------------------------------------------ |
| **cognix-docs (this one)** | Starlight `--sl-color-*` theme CSS | brand palette → `--sl-color-*` custom properties |
| cognix-web                 | Tailwind-v4 `tokens.css`           | brand palette → Tailwind `@theme` tokens         |

Only the **upstream brand source is shared**. The transform is different — never
copy the cognix-web `tokens.css` here, and never push this theme CSS there.

## Hard rule — themed, not forked

The brand is applied **entirely through Starlight's sanctioned theming surface**:
the `--sl-color-*` CSS custom properties, overridden in a custom CSS file wired
via `customCss` in `astro.config.mjs`. Do **NOT** fork Starlight, override
Starlight internals, eject components, or patch `node_modules`. Starlight must
stay upgradeable. If a brand value cannot be expressed as a `--sl-color-*`
override, stop and surface it — do not work around it by forking.

Reference: `.claude/skills/gtm-docs/references/docs-domain.md` § 5 (Theming —
themed, not forked) is the locked design this skill maintains.

## Steps

1. **Identify the upstream brand source** — the canonical Cognix brand palette
   (the same source the cognix-web variant reads). Note the current brand
   colors and the fonts: **Inter** (body) + **JetBrains Mono** (monospace).
2. **Identify the Starlight theme CSS file** — the custom CSS file holding the
   `--sl-color-*` overrides, the one wired via `customCss` in `astro.config.mjs`
   (commonly `src/styles/theme.css` or similar). If `customCss` is not yet
   wired, wire it — add the file and the `customCss` array entry. This is still
   the sanctioned surface, not a fork.
3. **Apply the transform** — seed Starlight's theme tokens from the vendored
   brand palette:
   - Map the brand palette onto the `--sl-color-*` accent and gray scales
     (e.g. `--sl-color-accent`, `--sl-color-accent-low/high`, `--sl-color-white`,
     `--sl-color-gray-*`), in both the light and dark `:root` blocks Starlight
     defines.
   - Set the fonts to Inter + JetBrains Mono.
   - Keep the vendored subset a faithful copy of upstream — this is a sync, not
     a redesign.
4. **Verify** — run `npm run check`, then `npm run build`. Both must pass.
   Spot-check the built site renders with the brand colors.
5. **Keep it themed-not-forked** — confirm the diff touches only the custom
   theme CSS file and (if newly wired) the `customCss` array. Nothing under
   `node_modules`, no Starlight component overrides.

## Rationalization table — STOP signals

| Thought                                                | Reality                                                                   |
| ------------------------------------------------------ | ------------------------------------------------------------------------- |
| "A quick component override gets the exact brand look" | That is a fork. Stay on `--sl-color-*`. Surface the gap instead.          |
| "I'll just copy cognix-web's `tokens.css` over"        | Different repo, different transform. Only the upstream palette is shared. |
| "This needs a decompose / pipeline pass"               | No. It is a task-shaped maintenance edit — read, edit, verify, done.      |
| "I'll redesign the palette while I'm here"             | This is a sync. Copy upstream faithfully; design changes happen upstream. |

## Red flags — back up immediately

- About to edit anything under `node_modules/` → STOP, that is a fork.
- About to override a Starlight component → STOP, use `--sl-color-*` only.
- About to touch `cognix-web` or a Tailwind `tokens.css` → STOP, wrong variant.
- About to skip `npm run check` / `npm run build` → STOP, verify before done.

## When this skill is done

- The custom theme CSS `--sl-color-*` overrides match the current upstream
  brand palette; fonts are Inter + JetBrains Mono.
- `customCss` in `astro.config.mjs` points at the theme CSS file.
- `npm run check` and `npm run build` pass.
- The diff is confined to the sanctioned theming surface — no fork.

## Scope NOT in this skill

- Forking or redesigning the Starlight theme (themed-not-forked is locked).
- The cognix-web marketing site's Tailwind-v4 `tokens.css` (different variant).
- Changing the upstream brand palette itself (that happens upstream).
- IA, sidebar, content, or CLI-reference ingest work.

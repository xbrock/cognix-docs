# cognix-docs

**Public** documentation website for Cognix — the Astro + Starlight static site
deployed to [`docs.cognix.dev`](https://docs.cognix.dev) via Cloudflare Pages.

## Project setup

`cognix-docs` is a self-contained `cognix-workflow` project: the `cognix-workflow`
plugin is installed at project scope, and it carries its own KMCP knowledge store.

The build is driven by the `site-build` skill, which walks
`docs/site-build-roadmap.md`.

## Development

Requires Node 22+ (see `.nvmrc`).

```sh
npm run dev      # start the dev server
npm run build    # build the static site to ./dist/
npm run check    # astro check (type-check)
npm run format   # prettier --write .
```

## Notes

Do not hand-edit the `.cognix/` knowledge store — it is local working state and
is gitignored (this is a public repo; the KMCP must never enter its history).
The `src/content/docs/reference/cli/` subtree, once scaffolded, holds
generated CLI-reference content ingested from the `cognix` release pipeline — do
not hand-edit it either.

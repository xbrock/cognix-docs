---
name: site-build
description: Use when checking or advancing the cognix-docs documentation-site build — "where is the docs site", "what's next on the docs site", picking up the next docs build step, or dispatching the next theme/IA/ingest/guides work unit. An evidence-verified build tracker that dispatches the next step into cognix-workflow or a docs maintenance skill. Not for building docs sections directly (this skill dispatches, it does not implement), not for the cognix-web marketing site, and not for writing narrative docs copy.
---

# cognix-docs Site Build

The runnable entry point for the `cognix-docs` documentation-site build. Shows
where the build stands, cross-checks the tracker against reality, and walks
Felix to the next step. A lighter analog of the Cognix `roadmap` skill: same
load→cross-check→dashboard→recommend→confirm→dispatch loop, but it walks one
mostly-linear build sequence and needs only file / section-directory / index-page
/ `autogenerate`-sidebar / ingest-target / workflow existence checks — no git or
KMCP archaeology.

**Announce at start:** "Using site-build to [show status / advance <step>]."

## MANDATORY — run BEFORE any other action this session

These steps are commands, not hints. Run them before answering any question,
before dispatching, before editing anything.

1. **Load the tracker** — read `docs/site-build-roadmap.md`. It is seeded; it
   always exists. If it is somehow absent, stop and tell Felix — do not
   reconstruct it.
2. **Check it is clean** — run `git status --short docs/site-build-roadmap.md`.
   If it shows uncommitted edits, surface that to Felix before any status
   change (a prior run may have been interrupted mid-update).
3. **Evidence cross-check** — run § Evidence cross-check and build the drift note.
4. **Show the dashboard** — present the dashboard — item 3 of
   § The fixed workflow (every site-build run) — before asking anything else or
   dispatching.

## Hard rules

<HARD-GATE>
**Do NOT re-decide the roadmap.** The 6-item build sequence in
`docs/site-build-roadmap.md` is seeded verbatim from GTM-build campaign thread C
(`docs-site-build`, KMCP `7b330fd0`). This skill navigates that sequence; it
never edits the item set or their order. If reality contradicts the roadmap,
surface the contradiction as a drift note and tell Felix — do not patch the
roadmap to fit.

**Do NOT mark a step `done` without evidence.** Every `done` requires a concrete
artifact signal (§ Evidence cross-check) — a file, section directory, index
page, `autogenerate` sidebar group, ingest target, or workflow file that
actually exists. No evidence → the step stays `in_progress` or `pending`.

**Do NOT skip the commit after a status change.** Every `site-build-roadmap.md`
edit is its own commit. That is how the tracker persists across sessions. No
batching two status changes into one commit.

**Do NOT dispatch a step before Felix confirms it.** Always show the dashboard,
recommend the next step, and get a yes first.
</HARD-GATE>

## Why this skill exists

The documentation-site build is a sequence of steps that span many sessions.
Without a live tracker, every fresh session has to reconstruct "what's done,
what's next, what's blocked" by inspecting the repo. `docs/site-build-roadmap.md`
plus this skill remove that friction: the doc is durable state, this skill reads
it, verifies it against the real repo, and confirm-then-dispatches the next step.

It is deliberately lighter than the `roadmap` skill. `roadmap` re-derives status
across git history, file existence, and KMCP for many heterogeneous item types.
`site-build` walks one mostly-linear sequence of homogeneous build steps, so the
evidence cross-check is just "do the files / section directories / index pages /
`autogenerate` sidebar / ingest target / workflow files this step claims
actually exist".

**The docs build has no external manual gate.** Unlike the `cognix-web` site
build — whose step 1 is an externally-run open-design visual pass — the
`cognix-docs` build sequence is fully dispatchable end to end (brand theme → IA
scaffold → CLI-reference ingest wiring → seed guides + API ref → verify
site-health → launch-ready). Do not invent or wait on a manual gate the docs
build does not have.

## On-disk layout

```
docs/site-build-roadmap.md             # the live state doc (this skill maintains it)
.claude/skills/site-build/SKILL.md      # this skill
.claude/skills/add-docs-page/           # maintenance skill — scaffold a new docs page (part of this GTM skill layer)
.claude/skills/sync-brand-tokens/       # maintenance skill — brand-token sync (part of this GTM skill layer)
```

`docs/site-build-roadmap.md` is the durable state. Everything flows from it.

## The build sequence

Six steps, mostly linear, seeded from thread C (see the roadmap doc for the full
text of each):

1. **Brand theme** — apply the Cognix brand to Starlight via `--sl-color-*`
   custom properties, fonts, logo, header, dark mode. Themed, not forked.
2. **9-section IA scaffold** — Starlight sidebar groups + one content directory
   per section under `src/content/docs/`, plus `index.mdx`. Sidebar configured
   in `astro.config.mjs`. (Depends on step 1.)
3. **CLI-reference ingest target + `autogenerate` sidebar** — wire
   `reference/cli/` as a dumb file-drop target for the `cli-docs` ingest, using
   Starlight's `autogenerate` sidebar; files marked `linguist-generated`.
   (Depends on step 2.)
4. **Seed 2–3 guides + thin API reference** — seed guides under `guides/`; a
   thin hand-written API Reference overview of the sidecar HTTP+WS API.
   (Depends on step 2.)
5. **Verify & tune site-health automation** — the site-health workflows are
   delivered by the GTM epic itself; this step verifies and tunes them against
   the real site.
6. **Launch-ready** — final gate. The docs site is verified complete.

## Status vocabulary

- `pending` — ready to start, no unmet dependency or gate.
- `blocked` — a dependency step is not `done`.
- `in_progress` — started; a sub-skill has been dispatched.
- `done` — verified complete, with an evidence signal (§ Evidence cross-check).

**Next step** = the first non-`done` item in sequence whose dependency is
`done`.

## The fixed workflow (every site-build run)

1. **Load** — MANDATORY step 1. Read `docs/site-build-roadmap.md`.
2. **Evidence cross-check** — MANDATORY step 3. Diff what the doc claims against
   what is actually in the repo (§ Evidence cross-check). Build the drift note.
3. **Dashboard** — present: each step with its status; the next step (one line);
   blocked steps with the dependency holding them; any drift warnings. If Felix
   only wanted status, stop here.
4. **Recommend + confirm** — name the next non-`done` step and ask "start it
   now?". Felix may override to any step — give a one-line soft dependency
   warning if it is out of order, then proceed on confirm.
5. **Dispatch** — on confirm: set the step `in_progress`, commit
   `chore(site-build): start <step>`, then invoke the routed sub-skill
   (§ Dispatch routing). Hand off.
6. **Close the step** — when Felix confirms the step is done: verify its
   evidence signal exists (§ Evidence cross-check). If it does — set `done`,
   flip any dependent step that is now unblocked to `pending`, commit
   `chore(site-build): complete <step>`. If it does not — keep `in_progress` and
   say why.

Steps 1–3 and 5–6 are low-freedom — exact, ordered, a commit per status change.
Step 4's recommendation is the first non-`done` item; Felix may still override
to any step — give a one-line soft dependency warning if it is out of order,
then proceed on confirm.

## Evidence cross-check

On every run, before trusting `docs/site-build-roadmap.md`, re-derive each
step's real status. This is lighter than the `roadmap` skill — file / section-
directory / index-page / `autogenerate`-sidebar / ingest-target / workflow
existence only, no git log or KMCP queries:

**No KMCP integration — by design.** Unlike the `roadmap` skill, this skill does
not read from or write to the KMCP knowledge base. Its evidence is file /
section-directory / index-page / `autogenerate`-sidebar / ingest-target /
workflow existence only; the durable state is `docs/site-build-roadmap.md`.

| Step                      | `done` evidence signal                                                                                                                                                                   |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Brand theme            | Cognix-brand `--sl-color-*` custom properties exist in the theme stylesheet, the logo/header config is set in `astro.config.mjs`, and dark mode is wired.                                |
| 2. 9-section IA scaffold  | a content directory exists under `src/content/docs/` for each of the 9 sections, each with an index page, plus `index.mdx`; the Starlight `sidebar` is configured in `astro.config.mjs`. |
| 3. CLI-reference ingest   | `src/content/docs/reference/cli/` exists as the ingest target, its sidebar group uses Starlight `autogenerate`, and the subtree is marked `linguist-generated`.                          |
| 4. Guides + API reference | `guides/` has 2–3 seeded guide pages and `reference/api/` has a hand-written API overview page.                                                                                          |
| 5. site-health automation | the PR-gate CI + scheduled site-health workflow files exist under `.github/workflows/`.                                                                                                  |
| 6. launch-ready           | steps 1–5 all `done` and the roadmap's launch gate is checked.                                                                                                                           |

**Step 5 has no hard upstream dependency.** "Verify & tune site-health
automation" carries no `Depends on` line in `docs/site-build-roadmap.md` — it
becomes reachable as steps 1–4 progress (there is a real site to verify against)
rather than blocking on any single prior step. Only "launch-ready" (step 6)
gates on all prior steps.

**Crash recovery.** A step left `in_progress` whose artifacts are absent (the
dispatch crashed mid-step) is flagged in the drift note and re-offered as the
next step — never assumed complete.

**Drift runs on `done` steps too.** A `done` step whose evidence has since
disappeared (a section directory deleted, a workflow removed) surfaces as a
drift warning — it never silently flips status.

**Ambiguity rule.** When a signal is inconclusive, the step keeps its
`site-build-roadmap.md` status and the drift note flags it `unverifiable —
confirm manually`. Never guess; never silently overwrite the doc.

## Dispatch routing

| Step kind                                                                                              | Routes to                                                               |
| ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| A genuinely-open step (a design decision thread C did not specify)                                     | `cognix-workflow` **brainstorm → decompose → execute → code-review**    |
| A step already specified by thread C (brand theme, IA scaffold, ingest wiring, guides — design locked) | `cognix-workflow` **decompose → execute → code-review** (no brainstorm) |
| Maintenance work — scaffold a new docs page                                                            | the `add-docs-page` skill (repo maintenance skill)                      |
| Maintenance work — brand-token sync                                                                    | the `sync-brand-tokens` skill (repo maintenance skill)                  |

Every step in the 6-step sequence is thread-C-specified, so the common path is
decompose → execute → code-review with no brainstorm. Reach for brainstorm only
when a step is genuinely open.

## Rationalization table — STOP signals

| Thought                                                            | Reality                                                                                                                 |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| "The doc says step 3 is pending, I'll just trust it"               | Run the evidence cross-check first — the doc drifts. Trust evidence, then the doc.                                      |
| "The section directories look mostly there, I'll mark step 2 done" | Partial is not done. All 9 section directories + their index pages + the sidebar config must exist. Else `in_progress`. |
| "I'll batch the status updates and commit once at the end"         | Every status change is its own commit. Batching kills cross-session persistence.                                        |
| "The roadmap step looks wrong — I'll just tweak it"                | This skill never re-decides the roadmap. It is seeded from thread C. Surface the contradiction as drift; do not patch.  |
| "Step 2 needs a manual visual-design gate first, like cognix-web"  | No. The docs build has no external manual gate — the whole sequence is dispatchable. Recommend step 2 directly.         |
| "I'll dispatch the next step to save a round-trip"                 | Always show the dashboard and get a confirm before dispatching.                                                         |
| "This step is thread-C-specified but I'll brainstorm it anyway"    | The design is locked. Thread-C steps go straight to decompose — no brainstorm.                                          |

## Red flags — back up immediately

- About to skip the evidence cross-check → STOP, the doc drifts; run it.
- About to mark a step `done` without a matching artifact signal → STOP, keep it `in_progress`.
- About to edit the step set or order in `site-build-roadmap.md` → STOP, that is re-deciding the roadmap; surface it as drift instead.
- About to wait on or invent a manual visual-design gate → STOP, the docs build has none.
- About to dispatch before Felix confirmed the step → STOP, show the dashboard first.
- About to make two status changes in one commit → STOP, one commit per change.

## Common mistakes

| Mistake                                                           | Fix                                                                    |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Trusting `site-build-roadmap.md` without the evidence cross-check | Always cross-check first; the doc is a claim, the repo is truth.       |
| Silently overwriting the doc when evidence disagrees              | Surface a drift warning and ask Felix; never overwrite silently.       |
| Marking a thread-C step `done` on partial artifacts               | All of the step's evidence signals must be present.                    |
| Skipping the per-change commit                                    | Commit after every status change — start, complete.                    |
| Brainstorming a step thread C already specified                   | Thread-C steps route decompose → execute → code-review, no brainstorm. |
| Treating a `blocked` step as the next step                        | The next step is the first non-`done` item whose dependency is `done`. |
| Waiting on a manual design gate                                   | The docs build has no external gate; recommend the next step directly. |

## Scope NOT in this skill

- Implementing the brand theme, IA scaffold, ingest wiring, or guides —
  dispatched sub-skills do that.
- Writing narrative docs copy or guide prose — that is content work, not build
  orchestration.
- Re-deciding the build sequence — it is seeded from thread C; route
  contradictions back to Felix as drift.
- The `cognix-web` marketing site — that has its own `site-build` skill.
- Scaffolding a new docs page or syncing brand tokens — the `add-docs-page` and
  `sync-brand-tokens` repo maintenance skills.

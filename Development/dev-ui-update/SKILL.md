---
name: dev-ui-update
description: Fold a visual design handoff into a project's planning docs — read the handoff bundle (HTML mockups, README, design tokens), create or update docs/design-system.html as the single source of truth for everything design, surface every conflict between the mockups and the existing plan, then sweep stale design lines out of docs/mvp-plan.html, docs/prd.html, docs/progress.html, the docs/feature-*.html briefs and docs/competitive-scan.html. Handles both the first handoff and every later one that revises or expands the UI, keeping a dated handoff log so nothing silently overwrites a locked decision. Never writes application code. Use whenever a design handoff, design package, mockup set, screen comps, style guide or design tokens arrive and must be reconciled with the plan — including "incorporate the design", "here are the new mockups", "the designer sent an update", "build the design system doc", "the UI changed", or "these screens are redesigned".
---

# Dev UI Update

Turn a visual design handoff into (a) one authoritative design document the implementer reads, and (b) plan docs that no longer contradict the design.

This skill runs twice in a project's life and then keeps running: once when the first handoff lands, and again every time the designer revises or expands the UI. The two cases share almost every step — what differs is whether `docs/design-system.html` is being *authored* or *amended*.

## Hard constraints

1. **No application code.** Not XAML, not React, not a "quick component to show the token in use." Token tables, measurements, state sketches and interaction contracts are the deliverable; implementation is not.
2. **`docs/` beats the handoff bundle's `reference/` folder.** A handoff bundle normally carries copies of the plan docs the designer worked against. Those copies are a historical record and are frozen. Never treat them as current, and never edit the current docs to match them.
3. **The handoff bundle's own README beats the mockups where they disagree**, unless the README says otherwise. Most handoff READMEs say this explicitly ("where a mock and this document disagree, this document wins") — find that line and honour it.
4. **Never delete a locked design decision.** Supersede it, dated, with the reason. A decision that vanishes is a decision that gets re-litigated.
5. **Never silently reconcile a conflict.** When the mockups and an existing plan doc disagree, the user decides. See Step 4.

## Step 0: Inventory

Before reading anything closely, list what exists. Do not skip this — the mode, the amount of work, and half the questions all fall out of the inventory.

1. **Handoff bundles.** Look for `docs/handoff/`, `docs/handoff-*/`, `docs/design/`, `design-handoff*/`, or whatever the user pointed at. A bundle usually contains:
   - a `README.md` — the written spec; the most important file in the bundle
   - one or more `*.dc.html` / `*.html` mockup canvases, plus a `support.js` runtime
   - `_ds/<system-name>-<uuid>/` — `styles.css` (authoritative token values), `readme.md` (the base design system's own guide), `_ds_manifest.json`, sometimes an adherence lint config
   - `screenshots/README.md` — the per-frame index, when the bundle ships one. Only this index is read. The image files themselves are never opened, linked or embedded.
   - `reference/` — frozen copies of the plan docs (see constraint 2)
2. **The design doc.** Does `docs/design-system.html` already exist?
3. **The plan docs.** Note which of these exist, by name: `docs/mvp-plan.html`, `docs/prd.html`, `docs/progress.html`, `docs/phases/*.html`, `docs/competitive-scan.html`, `docs/feature-*.html`, `README.md`, `CLAUDE.md`.
4. **Anything else in `docs/`** that looks like a decision record (`*-orig.html`, `review-*.html`, ADRs). Note them; do not assume they are current.

**Report the inventory before doing anything else**, and name every file that a normal project would have but this one does not — especially `docs/prd.html` and `docs/progress.html`. Do not create a missing doc. Say it is missing, say what will therefore not get updated, and move on.

If there is no handoff bundle at all, stop and ask where the design lives.

## Step 1: Read the handoff

Read, in this order:

1. The bundle `README.md`, whole. It carries the fidelity statement, the deviations from the base design system, the token tables, the per-screen specs, the interaction contracts, and — usually — a "not yet designed" list and an open-questions list. Everything downstream keys off it.
2. `_ds/*/styles.css` — the real token values. Where the README's table and the CSS disagree on a hex or a ramp, the CSS wins for values; the README wins for usage.
3. `_ds/*/readme.md` — the base system. You need this to state, in the design doc, which of the base system's rules the app deliberately breaks.
4. `screenshots/README.md` — the per-frame index, if the bundle has one. Cheaper than parsing the mockup canvas, and it names each frame's option ID. Skip this step when the file is absent and take frame IDs from the mockup canvas instead. Do not open the image files.
5. The mockup canvas itself — skim for structure and frame labels. Do **not** try to reconstruct measurements from its inline styles; the README already carries them, and the canvas is a static prototype with drawn-open popovers and faked carets.

While reading, collect three lists:

- **Locked** — decisions the handoff states as final.
- **Not drawn** — screens or states the handoff names as undesigned. These become the design doc's gap list and often the plan's open risks.
- **Suspect** — anything that contradicts the base system, the plan, an earlier handoff, or itself. The bundle's own internal inconsistencies belong here too (a frame count that disagrees with the frame index, a token named two ways).

## Step 2: Pick the mode, then confirm the grounding

**First handoff** — `docs/design-system.html` does not exist. You are authoring it.

**Update handoff** — `docs/design-system.html` exists. You are amending it. Before amending, read it whole, and read its **Handoff log** (see Step 3) so you know which bundle each existing statement came from.

Watch for a third case: `docs/design-system.html` exists but the incoming bundle explicitly supersedes an *earlier bundle* rather than the doc ("this document supersedes the `design_handoff_orrery_shell` package"). Treat that line as authoritative over everything the named package contributed, and record the supersession in the log.

Then open with a short grounding summary — a handful of lines, no more:

- what the app is, in one line
- which bundle is being folded in, its date, and how many screens it covers
- first handoff or update; if update, how many screens are new vs revised
- the two or three biggest conflicts or gaps you already see

Confirm it before writing anything.

## Step 3: `docs/design-system.html`

One self-contained HTML file. It holds everything design. After this skill runs, an implementer should be able to build from this document alone. The bundle stays available if someone wants to look at a picture.

### File contract

- Self-contained: one `<style>` block in `<head>`, no external CSS, no JS, no webfonts.
- Match the visual house style of the sibling plan docs (`docs/mvp-plan.html` etc.) — read one and reuse its palette, `.wrap` width, heading rhythm and `.note` / `.card` / `.tag` classes. The design doc is a *plan* document; it is not styled in the app's own design language, and confusing the two makes it unreadable.
- **No images.** The doc carries no screenshot links and no embedded pictures. Everything an implementer needs is written as text, tables and measurements. A screen that can only be explained by a picture is under-specified. Write the measurements instead.
- Every measurement, hex, ramp step and size stays transcribable: state values as values, in tables, so they can be pasted into a `ResourceDictionary`, a theme file, or a token JSON without arithmetic.

### Section contract

Use these sections, in this order. Omit a section only when the handoff genuinely has nothing for it, and say so rather than dropping the heading silently.

1. **Status and authority** — date, which bundles it consolidates, fidelity, and the standing rules: this doc wins over the handoff bundle; the bundle's `reference/` copies are frozen; `docs/` wins over them.
2. **Handoff log** — one dated row per bundle ever folded in: bundle path, date, what it added, what it changed, what it superseded. *This table is what makes the update mode auditable. It is not optional, and it exists from the first run with one row.*
3. **Base system and deliberate deviations** — the design system the handoff is built on, then the numbered list of rules the app breaks on purpose, each with its reason. Mark the list closed: keep all of them, extend none.
4. **Color** — light table, dark table, both full ramps, and the rules for accent text at paragraph sizes.
5. **Spacing, radius, elevation.**
6. **Typography** — family, fallback, and a role table (role · size / weight · notes). Include the muted-opacity ladder.
7. **Icons** — set, weight, the full used list, sizes by context, and any reserved icon (an AI-only marker, for example).
8. **The application shell** — window, titlebar, navigation in both expanded and collapsed form, any flyout, status bar, and any persistent side rail. Every region with its width and fill token.
9. **Screens** — one subsection per screen, in a stable order, each headed with its frame IDs, each carrying layout, measurements, anatomy and states. This is the bulk of the document.
10. **Cross-cutting interaction contracts** — the numbered rules that hold on every screen (how a Suggest/assistant surface behaves, how a modal resolves, what may never auto-commit).
11. **Motion and interaction states** — durations, hover/pressed/focus, keyboard map, reduced-motion behavior.
12. **Accessibility.**
13. **State sketch** — the view-model per screen. Keeps the design doc useful to the implementer without duplicating the data model.
14. **Assets** — fonts, icon bundles, licensing, what must be vendored vs. what is a system resource.
15. **Not yet designed** — the gap list, and the open questions for the implementer, numbered.
16. **Screen → phase map** — which build phase implements which screen, keyed to the phase numbers in `docs/prd.html` or `docs/mvp-plan.html`. This is the join between design and plan; keep it current in both directions.

### First handoff

Author all sections from the bundle. Where the bundle is silent, write the section heading and one line naming the gap — an empty heading is information.

### Update handoff

Amend in place. Rules:

- **Classify every incoming item before writing it**: `new` (a screen or token that did not exist), `revised` (a value or behavior that changed), `superseded` (an earlier decision explicitly overturned), `unchanged` (restated identically — write nothing).
- **Token changes are the highest blast radius.** A changed hex, size or spacing step touches every screen. Call every token change out separately in the report, and check whether any screen section quotes the old value inline.
- **Revising a screen rewrites that screen's subsection**, and appends a dated line to the Handoff log — not a strikethrough inside the section. Keep screen sections clean and current; keep the history in the log.
- **A superseded decision moves into the log with its reason**, and out of the body. Constraint 4 is satisfied by the log, not by clutter.
- **Never drop a screen the new bundle simply does not mention.** Silence is not deletion. If a screen looks abandoned, ask.
- **Items graduating off the "not yet designed" list** get removed from section 15 and added as real subsections in section 9. Say which ones graduated.
- **Bump the date and the bundle list in section 1**, always.

### Checkpoint

**Stop here and show the user the design doc before touching any plan doc.** The design document is large and it becomes the authority for everything after it; a wrong section 3 propagates into four other files. Offer exactly this choice: review the design doc first, or continue straight through to the plan sweep. Recommend reviewing first on a first handoff, and continuing through on a small update handoff.

## Step 4: The conflict sweep

Read the plan docs against the design doc you just wrote. Every place they disagree is a conflict. Build one table and bring it to the user:

| # | Where | Plan says | Design says | Kind |

Kinds, which determine what happens next:

- **Stale** — the plan describes a placeholder that the design has now settled ("theme to be supplied later", "visual design TBD"). No decision needed; fix it in Step 5.
- **Contradiction** — the plan and the design describe genuinely different behavior (a popover vs. a rail; four action types vs. three slots; one table vs. one table per chapter). **The user decides.** Do not assume the newer artifact wins — a designer can miss a requirement just as easily as a plan can go stale.
- **Naming** — same thing, different word ("fill empty fields" vs. "Fill in the Blanks", renamed traits, renamed columns). Cheap to fix, but pick one name and use it in every doc.
- **Sequencing** — the design implies work lands in a different phase than the plan schedules (a locked shell moves theming from a late polish phase into phase 1). Confirm the phase move before writing it.
- **Internal** — the plan contradicts itself, or the bundle contradicts itself, independent of the other. Report it; it is usually a one-word fix and it is always worth catching.

Present contradictions one at a time, in prose, with your recommendation and a one-line reason, so the user can reply "yes" and move on. Do not use tappable option widgets for these — the real answer is often a third thing.

Nothing in Step 5 is written until the contradictions are resolved.

## Step 5: Sweep the plan docs

Act only on docs that exist. Show each intended edit before writing it. Never restate the design in a plan doc — one or two lines and a link to `docs/design-system.html` is the correct density.

**`docs/mvp-plan.html`**
- Rewrite the interface/UI section to match the locked design, and link the design doc.
- Delete every "design to be supplied", "theme is a placeholder", "theming seam" line. That was true; it is not any more.
- Re-point the build phases: a locked visual design usually moves theme resources, shell chrome and icon bundling *earlier* (typically into phase 1) and empties a late "theme seam ready for design" line.
- Append a dated entry to the amendments/header meta line naming the bundle folded in and that `dev-ui-update` did it. Add an amendments section if there is none.

**`docs/prd.html`** — preserve its machine-readable shapes exactly; `dev-create-progress` parses them.
- Update UI/UX requirements and any interface specification section.
- Add design deliverables to the phase checklists they belong in, and extend those phases' validation criteria ("renders at the locked token values in light and dark").
- Never renumber existing phases. Use a sub-phase (`data-phase="3b"`) if design work slots between two planned phases.

**`docs/progress.html`** — do not hand-edit. Re-run `dev-create-progress` after the PRD changes. If the PRD does not exist, say that progress cannot be reindexed and leave it alone.

**`docs/feature-*.html` briefs** — each brief the design covers gets: its visual/UI passages reconciled with the design doc, and a line at the top pointing at `docs/design-system.html` as the visual authority. Where a brief's *model* (not its visuals) conflicts with the mockups, that is a Step 4 contradiction and must already be resolved before you touch the file.

**`docs/competitive-scan.html`** — usually untouched by a design handoff. Touch it only if the design newly ships something a scan row is badged as missing; then update that row's badge, its `.cov` note and its `data-coverage` / `data-phase` attributes together. Never delete a row, never re-rank.

**`CLAUDE.md` / repo README** — if either states UI conventions, point them at the design doc rather than duplicating tokens.

## Step 6: Mark the handoff bundle

The bundle stays frozen, with two exceptions:

1. Append a short block to the bundle's `README.md`: this bundle has been consolidated into `docs/design-system.html` on <date>; the design doc is now authoritative; the `reference/` copies are historical and `docs/` wins over them.
2. Fix outright factual errors inside the bundle README that would mislead a reader (a frame count that disagrees with the frame index, a broken relative path). Nothing else.

Do not refresh `reference/` from `docs/`. The record of what the designer actually worked against is worth more than a self-contained bundle.

## Step 7: Verify

Run these, and report the result of each:

1. Open `docs/design-system.html` in a browser (or render it) and confirm it loads with no broken layout.
2. The design doc contains no `<img>` tag and no link to an image file.
3. Every token value in the design doc matches `_ds/*/styles.css`. Any mismatch is a bug in the design doc, not in the CSS.
4. Every screen named in the handoff README has a subsection; every subsection names its frame IDs.
5. No plan doc still contains a phrase from the stale list ("design TBD", "theme placeholder", "visual design to be supplied", "mockups pending").
6. The Handoff log has a row for this bundle.
7. On an update run: every item you classified `revised` or `superseded` appears in the log, and no screen present before the run is missing after it.

## Step 8: Report

Close with:

- every file touched, one line each
- the token changes, listed separately, if any
- the conflicts resolved and how
- what did not get updated, and why (missing PRD, missing progress, declined change)
- the "not yet designed" list, as the standing gap
- the next command — normally `dev-plan-phase <N>` for the phase the design just changed, or `dev-create-progress` if the PRD moved

Still no code.

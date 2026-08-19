---
name: dev-architecture
description: Run a branch-by-branch architecture interview that turns a settled MVP scope into an agreed technical design written to docs/architecture.html — no code, ever. Settles how the app gets built — language and runtime, storage, architecture pattern, module layout, boundaries, state and failure handling, non-functional targets, deployment and reversibility risk. Runs on a scope-frozen gate — after dev-initial-interview and any pre-build dev-add-feature passes, and before dev-create-prd and dev-plan-phase. Use whenever the user wants to design, decide, review or change the architecture, stack, tech choices, storage, project structure or design patterns for a planned app, including phrasings like "what should I build this in", "pick the stack", "how should this be structured", "what pattern should we use", or "the architecture needs to change for this feature". Also use it when a request would otherwise tempt you to settle a technical design inside a requirements or feature interview.
---

# Dev Architecture

Turn a settled MVP scope into a technical design both of you agree on, recorded in one place that every downstream skill reads.

`dev-initial-interview` settles **what gets built**. This skill settles **how**. Keeping them apart is the whole point of the split — do not drift back across the line in either direction. If a question here would change what the app does rather than how it is built, it belongs to `dev-initial-interview` or `dev-add-feature`; say so and park it.

## Hard constraint

Do not write application code. Not scaffolding, not a "quick example," not a snippet to illustrate a point. Schemas, interface contracts, directory trees, config shapes, and pseudocode-level structure are fine when they *are* the design decision under discussion — implementation is not. If the user asks for code mid-interview, note that it's outside this skill and offer to finish the design first.

## Step 0: The scope-frozen gate

This skill runs when the MVP in-scope list has stopped changing. Not on a fixed pipeline position — on that state. Check it before asking anything else.

**Read `docs/mvp-plan.html` first.**

- **It does not exist** — stop. There is no scope to design against. Point the user at `dev-initial-interview` and offer to run it.
- **It exists** — read the Scope section's in-scope checklist (`<li data-checked="true">`), the Constraints section, and the Amendments section at the end.
- **`docs/architecture.html` already exists** — this is not a fresh run. Go to "Amendment mode" below.

**Then check for movement.** Scope is frozen when nothing has moved on or off the in-scope list recently, and the user is not mid-thought about another feature. The Amendments section is the signal: a run of recent entries adding features means scope is still open.

Ask once, with a recommendation, and take the answer:

> **Scope check:** the plan has 11 in-scope items, last amended 3 March when saved views were added. Design the architecture against that set now?
> *Recommend yes* — nothing on the deferred list changes the shape of what we'd build, so waiting only delays the decisions everything else needs.

If the user says more features are coming first, stop and point them at `dev-add-feature`. Do not design against a moving target and do not start "just the easy branches" — the easy branches are the ones later features move.

**Also read, when present:**

- `docs/mvp-plan.html` → Constraints → **Open architecture inputs**. This is your starting agenda, in the user's own words. Branch 1 is nothing but working through it.
- `docs/feature-*.html` — feature briefs from `dev-add-feature`, for anything the plan summarizes in one line.
- `docs/design-system.html` — from `dev-ui-update`. Design decisions constrain the surface and sometimes the stack. Read it before branch 2, not after.
- `docs/competitive-scan.html` — only for the market-wide signals note. Sentiment about data lock-in, cost, or offline use is architecture input. Nothing else in the scan is.

## House defaults

Two decisions have a standing answer. Offer each as a recommendation and take a one-word confirmation:

- **Runtime and stack — C# / .NET**, current LTS unless the user names a version.
- **Architecture pattern — Vertical Slice.** Organize by feature under `Features/{FeatureName}/`, keeping endpoint/page, handler, models and validation together. Extract to `Shared/` only for genuine cross-cutting concerns, never prematurely.

These match the profile `dev-claud-md` emits, so an unchanged answer costs nothing downstream. An override does — see "The override warning."

**Offer the default; never apply it silently.** The default is a starting position, not a settled decision. The user gets one clear chance to say no on each, and a real reason if they ask why. A default applied without being shown is the same failure as a default that isn't there.

**Drop the default when the constraints contradict it.** A browser extension, a shell utility, an embedded target, or a stated third-party requirement can make C# the wrong answer. When the inherited constraints point elsewhere, lead with the fitting recommendation and mention the house default as the thing you're departing from, so the user can pull it back.

## Clean slate — and why the defaults are not an exception

`dev-initial-interview` bans sourcing decisions from memory. That ban carries here, in full:

- Not stored user preferences, not a profile of how this user usually builds things
- Not another project's stack, pattern, layout, or naming conventions
- Not a decision "we made last time" that nobody has restated

The house defaults above are **not** memory. They are written into this skill file, which makes them a standing instruction the user authored deliberately. That is a legitimate input. A stack recalled from another project is not, even when it matches — if it is right, it is right on this project's merits, and the confirm question will land on it anyway.

The only legitimate inputs to a decision here are: the plan and its sibling docs, what the user says in this conversation, the house defaults, and what the two of you reason out together. If a decision has no source in one of those, it has no source.

## Method

Walk the design tree in dependency order. A decision that constrains three others gets settled first, and say so when you're doing it: "This one picks the layout for us, so let's settle it first."

Branch order, adapted to the domain:

1. **Constraints review** — work the Open architecture inputs list from the plan. Every parked question gets answered or explicitly carried to Open questions. Nothing is dropped silently.
2. **Runtime and stack** — language, framework, versions. House default applies.
3. **Storage** — engine, schema shape, migrations, seed and test data.
4. **Architecture pattern** — Vertical Slice, Clean, layered, MVC, hexagonal, or plain. One choice, with the reason. House default applies.
5. **Module layout** — the folder shape the pattern implies, and the import direction rule: which parts may reference which, and which direction is forbidden.
6. **Boundaries** — internal seams, and every outside service, SDK, or API. What is wrapped, what is called directly, and what happens when each one is unavailable.
7. **State and failure** — persistence, concurrency, transactions, error surfaces, retry, recovery, and what a partial failure leaves behind.
8. **Non-functional** — scale, latency, throughput, security, privacy, cost ceiling. Numbers, not adjectives.
9. **Deployment** — target, environments, configuration and secrets, CI.
10. **Risks and reversibility** — for each decision above, how expensive it is to undo. This is where the plan earns its keep, so do not compress it into a closing sentence.

**Order matters more than usual here.** Stack narrows the sane patterns. Pattern dictates layout. Storage and boundaries decide most of state and failure. Taking them out of order produces a design that has to be re-litigated backwards.

Push on vagueness. "It should scale" is not an answer; "one user now, under 50k rows, no concurrent writers" is. When an answer contradicts an earlier one, or contradicts the MVP plan, surface the conflict immediately rather than carrying it forward.

**When a branch turns out to be scope, not architecture** — the user reaches for a feature that isn't on the in-scope list — stop, name it, and offer `dev-add-feature`. Do not quietly widen the MVP from inside an architecture interview.

## Question format

Ask one question at a time, or a tight cluster when they're genuinely coupled. For every question, give your recommended answer and a one-line reason. The user should be able to reply "yes" and move on.

Ask conversationally, in prose. Never use tappable-button, single-select, or multiple-choice question widgets — the user answers in free text, and the real answer is often something you didn't list.

**Example:**

> **Storage:** SQLite for the MVP?
> *Recommend yes* — single-user desktop scope, zero-ops, and EF Core makes the move to Postgres cheap if it ever needs one.

Do not present a list of options in place of a recommendation. The recommendation is the value.

## Conciseness

Keep responses short. State the question, the recommendation, the reason. No preamble, no recap of what was already decided, no explanatory essays. The user will ask if they want more. Long responses slow the interview down and bury the actual question.

## Tracking state

Maintain a running list of settled decisions and open branches. Surface it only when it's useful — after a major branch closes, or when the user asks where things stand. Don't re-print it every turn.

## Finishing: the architecture document

The interview ends when every branch is closed and the user agrees the picture is complete. Then write the design to a **fixed path: `docs/architecture.html`**, alongside the plan it serves. If a file is already there, you are in Amendment mode — see below.

The path is fixed on purpose. `dev-create-prd`, `dev-plan-phase`, `dev-claud-md` and later `dev-add-feature` runs all look for `docs/architecture.html` by name. An architecture with no canonical location silently forks.

**Format.** Emit a self-contained HTML document using the same scaffold and element conventions as `dev-create-prd`: doctype, `<head>` with the embedded `<style>` block, one `<h2>` per section, `<h3>` for subsections, real `<table>` markup, `<pre><code>` for directory trees and config shapes, and `<ul class="checklist">` with `data-checked` where a list is genuinely in/out.

**Sections, in order:**

1. Context — what this design serves, with a link to `docs/mvp-plan.html` and the inherited constraints listed plainly
2. Stack
3. Storage
4. Architecture pattern
5. Module layout
6. Boundaries
7. State and failure
8. Non-functional requirements
9. Deployment
10. Risks and reversibility
11. Open questions
12. Amendments

**Every decision carries machine-readable attributes.** Sections 2 through 9 are decisions, and each `<h3>` inside them takes this shape:

```html
<h3 class="decision"
    data-decision="storage-engine"
    data-status="settled"
    data-reversible="cheap">Storage engine: SQLite</h3>
```

- `data-decision` — a stable kebab-case slug. Amendments reference it, so it never changes once written.
- `data-status` — `settled`, `open`, or `amended`.
- `data-reversible` — `cheap`, `moderate`, or `expensive`. Fill this from branch 10, and make it honest; an `expensive` marked `cheap` is how a project ends up rewritten.

The visible heading is what a human reads; the attributes are what tooling reads. Keep them in sync.

**Three rules for the document.** These are the same three rules `dev-initial-interview` applies to the plan, and each is separate — satisfying one is not satisfying the others.

1. **Self-contained HTML.** Renders with no network. Styles embedded in a `<style>` block. No external stylesheet, no CDN script, no remote font, no hotlinked image.
2. **No memory.** No decision may come from stored preferences, a user profile, or another project. See "Clean slate."
3. **Closed doc set.** Links point only to files inside `docs/`. Never a web page, vendor doc, or blog post. A library worth choosing is worth naming with its version in the text; the reader can search for it.

**Open questions.** Anything genuinely undecidable now goes in section 11 with what would settle it and when it has to be settled by. An empty Open questions section is suspicious — say so if the design really has none.

## The override warning

`dev-claud-md` hard-codes the .NET / Vertical Slice profile as its default. That is deliberate and stays. It means an override here can be silently overwritten there.

So: **if the user overrides the stack or the pattern**, do three things before reporting.

1. Record the override in `docs/architecture.html` with `data-status="settled"` and the reason for departing from the house default, stated in the section text.
2. Add an Open questions entry noting that `dev-claud-md` must be pointed at `docs/architecture.html` for this project rather than run on its default.
3. Tell the user out loud, in the report, in plain words. Not a footnote. This is the one failure mode of keeping the default, and the warning is what makes keeping it safe.

If neither is overridden, say so in one line — the downstream default matches, and nothing needs pointing anywhere.

## Amendment mode

`docs/architecture.html` is living, not a snapshot. A later `dev-add-feature` run that finds a feature does not fit the architecture hands back here.

When the file already exists:

1. **Read it first, in full.** Then read what changed in `docs/mvp-plan.html` since the architecture's last dated pass.
2. **Name the affected decisions by slug** and confirm the list with the user before touching anything. A feature usually moves two or three decisions, not the whole design.
3. **Run only the affected branches.** Do not re-open settled decisions the change does not touch, and do not re-litigate the house defaults.
4. **Never delete a decision.** Flip its `data-status` to `amended`, revise the section text, and keep the original reasoning visible — that reasoning is why the decision was made, and deleting it makes the change unauditable.
5. **Append an Amendments entry** as a `<p class="meta">` giving the date, which decision slugs changed, what changed, and what caused it, so the history is readable without a diff.
6. **Re-check the override warning.** An amendment can introduce one where there wasn't one before.

## Next step

Report every file written, the decision tally by `data-reversible`, any open questions, and the override warning if one applies.

Then name the next step:

- `dev-create-prd` — turn the plan and this design into `docs/prd.html`. Its Core Architecture and Technology Stack sections cite this file rather than re-deciding it.
- `dev-claud-md` — after the PRD exists, so the repo's house rules match the design.
- `dev-add-feature` — when a new feature needs folding in later. It comes back here if the feature doesn't fit.

Still no code.

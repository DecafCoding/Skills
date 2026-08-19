---
name: dev-add-feature
description: Run a relentless, branch-by-branch interview that turns a rough feature idea into a complete integration plan for an existing app — grounded in the current plan, the settled architecture at docs/architecture.html, the repo, and any prior competitive scan at docs/competitive-scan.html. Checks the feature against the architecture rather than redesigning it, and hands back to dev-architecture for an amendment when the feature does not fit. Then folds the agreed feature back into whichever plan docs exist (docs/mvp-plan.html, docs/prd.html, docs/progress.html, docs/competitive-scan.html). Never writes code. Use this whenever the user wants to add, extend, bolt on, "think through," or decide what to build next for something that already exists or is already planned, even if they never say the word "interview." Companion to dev-initial-interview, which covers net-new applications; use dev-add-feature instead once there is an existing plan or codebase to fit into.
---

# Dev Add Feature

Turn a rough feature idea into a complete, mutually understood plan for folding it into an existing application — by first understanding what already exists, then interviewing the user until nothing material is unresolved.

## Hard constraint

Do not write application code. Not scaffolding, not a "quick example," not a snippet to illustrate a point. Schemas, API shapes, data models, interface contracts, and pseudocode-level structure are fine when they *are* the design decision under discussion — implementation is not. If the user asks for code mid-interview, note that it's outside this skill and offer to finish the plan first.

## Step 0: Ground yourself in what exists

Before asking anything, read the ground truth. Do not skip this — the value of this skill over a generic brainstorm is that every question is informed by the actual system.

1. **The plan** — locate and read the existing planning docs: `docs/mvp-plan.html`, `docs/prd.html`, phase docs under `docs/phases/`, `docs/progress.html`, README, or whatever the user points at. If the app exists only as a plan, that plan *is* the ground truth. **Note which of these exist** — you will need that list again at the end, when the plan docs get updated to include this feature.
2. **The competitive scan** — check for `docs/competitive-scan.html`. If `dev-initial-interview` ran, this holds the Required / Top 10 / Other buckets with a coverage badge on every row. Read it; it's the backlog this feature may already be sitting in, and the badges say what is already built.
3. **The architecture** — check for `docs/architecture.html`. If `dev-architecture` ran, this is the settled technical design: stack, storage, pattern, module layout, boundaries, state and failure, non-functional, deployment. Read it in full. Every `<h3 class="decision">` in it is a constraint this feature must fit, and `data-reversible` tells you what a misfit would cost to undo. It is the authority for branch 7.
4. **The repo** — if there is code, survey it: entry points, module layout, data model / migrations, routing or command surface, config, tests. Read the parts the feature will plausibly touch. Don't read everything; read enough to ask sharp questions.
5. **Conventions** — how this codebase already does the thing the feature needs (persistence, auth, background work, error handling, UI composition). New features should follow existing patterns unless there's a reason not to. Where the repo and `docs/architecture.html` disagree, say so — one of them is wrong, and which one is a question for the user, not a guess for you.

Then open with a short grounding summary — no more than a handful of lines: what you understand the app to be, which parts this feature likely touches, and the two or three biggest unknowns. Confirm it before interviewing.

If there is no plan and no repo, say so and suggest `dev-initial-interview` instead.

## Step 0.5: The competitive scan, if there is one

When `docs/competitive-scan.html` exists, it changes how this interview opens. Handle it in one of two ways depending on how the user arrived:

**They named a feature.** Check whether it's already in the scan. If it is, lead with that in the grounding summary — the entry's user-sentiment notes and which comparables ship it are evidence you already have, and it sharpens branch 1 considerably. Say what the scan found and whether it agrees with their motivation. If the scan contradicts them — everyone ships it and users still complain about it — surface that now, not later.

**They're deciding what to build next.** Present the Top 10 as the candidate menu — but only the rows still badged `b-no` (Not in MVP). Rows badged `b-in` or `b-part` are already built or partly built, and `b-never` is ruled out; offering any of them as a fresh candidate is the exact failure the coverage badges exist to prevent. A `b-part` row is still fair game as a *widening* — say so explicitly rather than pitching it as new. Keep the scan's original ranking; do not re-rank to fill the gaps. Then recommend one and say why, in terms of the app's current state — what's now built, what the riskiest remaining assumption is, what the last phase exposed. Mention that Other exists and offer to show it if nothing in the ten fits.

Either way, once a feature is chosen, the scan's job as an input is done — but it still gets updated at the end (see the Finishing step). Run the normal branch order against it. Do not keep referring back to what competitors do — that's market evidence, and past this point the existing app and its users are the better authority.

If the scan is stale enough to matter — the app has changed a lot, or it's been a year — say so and offer to refresh it before choosing. Don't refresh silently.

## Method

Walk the design tree. Start at the root (what the feature is and why it's needed now), then descend branch by branch. Resolve decisions in dependency order — a decision that constrains three others gets settled before those three. Say so when you're doing this: "This one gates the data model, so let's settle it first."

Typical branch order, adapted to the domain:

1. **Motivation** — what's missing today, who feels it, what "done" changes for them
2. **Fit** — where this sits relative to the existing core loop; does it extend it, branch it, or replace part of it
3. **Scope line** — what's in this feature, what's explicitly deferred, what's never
4. **Blast radius** — every existing module, table, endpoint, screen, and job this touches; what breaks if it's wrong
5. **Data model changes** — new entities, changed entities, migrations, backfill, rollback
6. **Interfaces** — new or changed surfaces, inputs, outputs, and the contract with existing ones
7. **Architecture fit** — reuse vs. new component, dependencies added, conformance to existing conventions. When `docs/architecture.html` exists this branch *checks* against it rather than deciding — see "The architecture handoff" below
8. **State and failure** — persistence, concurrency, partial failure, recovery, interaction with existing state
9. **Compatibility** — backward compatibility, existing data, existing users, feature flagging, staged rollout
10. **Non-functional** — performance impact on existing paths, security, privacy, cost delta
11. **Testing and regression** — what proves this works, what proves the old behavior still works
12. **Build plan** — task ordering, what proves the riskiest assumption first, where it lands in the existing phase plan
13. **Done criteria** — how you both know the feature is finished

Push on vagueness. "It should be fast" is not an answer; "sub-200ms on a 10k-row table" is. When an answer contradicts an earlier one — or contradicts the existing plan or code — surface the conflict immediately rather than carrying it forward.

## The architecture handoff

When `docs/architecture.html` exists, this skill does not decide architecture. It decides whether the feature fits the architecture that is already agreed. Branch 7 has exactly three possible outcomes, and you must land on one of them out loud.

**1. It fits.** The feature uses the existing pattern, layout, storage and boundaries as they stand. Note which `data-decision` slugs it leans on and move to branch 8. Nothing else to do.

**2. It fits with a local exception.** The feature needs one narrow departure — an extra library, a new table shape, a background worker where none existed. Record it in the feature brief as an exception, name the decision slug it departs from, and say plainly what would need to change if the exception ever became the rule. Do not edit `docs/architecture.html` for a one-off.

**3. It does not fit.** The feature needs a decision in `docs/architecture.html` to change — a different storage engine, a new boundary, a pattern the layout can't express, a non-functional target the current design can't hit. **Stop the interview here.** Do not design the replacement yourself, and do not carry on to branches 8 through 13 as if the foundation were settled.

For outcome 3, do this:

- Name the decision slugs that would have to change, and their `data-reversible` values. `expensive` is the number that should make both of you slow down.
- Tell the user the feature requires an architecture change, and offer `dev-architecture` in amendment mode. It reads the existing file, changes only the affected decisions, and logs the amendment.
- Offer the alternative in the same breath: a narrower version of the feature that fits the current architecture. Often that is the better trade, and the user cannot weigh it unless you put it beside the amendment.
- Resume this interview once the amendment lands, re-reading `docs/architecture.html` first.

**Branches 8 and 10 stay feature-local.** State and failure, and non-functional, are asked here only about *this feature's* impact — what it adds to existing paths. The system-wide targets belong to `docs/architecture.html`. If answering one of these would change a system-wide target, that is outcome 3, not a branch 8 answer.

## Question format

Ask one question at a time, or a tight cluster when they're genuinely coupled. For every question, give your recommended answer and a one-line reason, anchored in what the code or plan already does where relevant. The user should be able to reply "yes" and move on.

Ask conversationally, in prose. Never use tappable-button, single-select, or multiple-choice question widgets — the user answers in free text, and the real answer is often something you didn't list.

**Example:**

> **Storage for saved filters:** new `saved_filters` table, or a JSON column on `users`?
> *Recommend the table* — the existing schema already normalizes per-user collections this way, and a JSON column blocks the "share a filter" case you mentioned.

Do not present a list of options in place of a recommendation. The recommendation is the value.

## Conciseness

Keep responses short. State the question, the recommendation, the reason. No preamble, no recap of what was already decided, no explanatory essays. The user will ask if they want more. Long responses slow the interview down and bury the actual question.

## Tracking state

Maintain a running list of settled decisions and open branches. Surface it only when it's useful — after a major branch closes, or when the user asks where things stand. Don't re-print it every turn.

## Finishing: the feature plan document

The interview ends when every branch is closed and the user agrees the picture is complete. Then write the plan as an **OKF-HTML concept** — follow the `okf-html-library` skill for the mechanics (library root, `okf-meta` block, relative `theme.css` link, `templates/concept.html`, index regeneration). Do not hand-write styling; styling lives in `theme.css`.

Metadata conventions for this document:

- `type`: `Feature Plan` (reuse the library's existing type if one already fits)
- `title`: the feature name
- `description`: one line — the feature and the app it extends
- `tags`: the app name, plus the areas touched
- `resource`: the repo or plan URI if there is one, otherwise `null`

Body sections, as semantic HTML:

1. Summary and motivation
2. Fit with the existing application (with links to related concepts where the library has them)
3. Scope — in, deferred, never
4. Blast radius — the concrete list of touched components
5. Data model changes and migration/rollback approach
6. Interfaces and contracts
7. Architecture fit — which of the three outcomes this landed on, the `data-decision` slugs the feature leans on, any local exception and what it departs from, and the reasoning kept. If an amendment to `docs/architecture.html` was needed, name it and the date it landed rather than restating the new decision here
8. Compatibility and rollout
9. Testing and regression plan
10. Build plan — ordered tasks and where they land in the existing phase plan
11. Done criteria
12. Open risks and assumptions

## Finishing: fold the feature back into the plan docs

Writing the feature plan is not the last step. A feature that exists only in the knowledge library, while the MVP plan, the PRD, and `progress.html` still describe an app without it, is a plan that has already drifted. Close the loop.

**This step is required, not an offer.** Do not ask whether the user wants the plan docs updated — they do; that is what "add a feature" means. Do ask for confirmation on the *substantive* choices called out below (phase number, in-scope vs. deferred), and show the user each edit you intend to make before writing it.

Check for each doc and act only on the ones that exist. If none exist, say so and stop — there is nothing to update.

**1. `docs/mvp-plan.html`, if it exists.**

- If the feature is in scope for the MVP: add it to the in-scope checklist (`<li data-checked="true">`), and update the sections it materially changes — data model, interfaces, milestones, done criteria. Do not restate the whole feature plan; one or two lines per section plus a link to the feature-plan concept.
- If the feature is deferred past the MVP: add it to the deferred checklist (`<li data-checked="false">`) with a link, and change nothing else. Deferred features do not get to edit the data model.
- Either way, append an entry to the `Amendments` section: the date, the feature name, what changed, and that `dev-add-feature` changed it. If the doc has no `Amendments` section, add one.

**2. `docs/prd.html`, if it exists.**

The PRD's HTML contract is machine-readable and `dev-create-progress` parses it — preserve the shapes exactly (see `dev-create-prd`).

- **MVP Scope** — add the feature to the in-scope or out-of-scope checklist, matching the scope line settled during the interview.
- **User Stories** — add the one or two stories the feature introduces, in the existing "As a … I want … so that …" format.
- **Implementation Phases** — this is the decision to confirm with the user before writing. Ask: does this land in an existing phase, or become a new one? Then:
  - *Existing phase* — add the feature's deliverables to that phase's checklist, and extend its validation criteria.
  - *New phase* — append a new heading in the exact required shape, `<h3 class="phase" data-phase="<N>">Phase <N>: <name></h3>`, with Goal, Deliverables checklist, and Validation criteria beneath it. Never put a time estimate inside the heading text; it goes in a following `<p class="meta">`. Pick the next unused phase number; use a sub-phase (`data-phase="3b"`) when the feature slots between two existing phases and renumbering would break already-planned work. **Never renumber existing phases** — phase docs, branch names, and `progress.html` all key off those numbers.
- Sections the feature genuinely changes (Technology Stack, API Specification, Security & Configuration, Risks) get updated too. Sections it doesn't touch stay untouched.

**3. `docs/progress.html`, if it exists.**

Do not hand-edit it. Re-run `dev-create-progress`, which regenerates the index from the phase docs and the PRD. A feature added to the PRD as a new phase with no phase doc yet correctly appears as `data-status="not-planned"` — that is the intended state, and it is what tells the autonomous routine to stop and ask for `dev-plan-phase <N>`.

Regeneration preserves existing task statuses and blockers. If `dev-create-progress` warns that it would clobber `done` / `skipped` marks or blocker blockquotes, stop and show the user the warning rather than forcing it.

If the PRD is absent but `progress.html` exists, say that the new feature cannot be indexed without a PRD phase and leave `progress.html` alone.

**4. `docs/competitive-scan.html`, if it exists.**

The scan is the standing backlog and the market evidence behind it. If it still shows this feature as unbuilt after the interview, the next run offers it again as if nothing happened.

**The badge is the contract.** Each row carries exactly one coverage badge — `b-in` (In MVP · P#), `b-part` (Partial · P#), `b-no` (Not in MVP), or `b-never` (Never) — plus a `.cov` note explaining the mark, and, on newer scans, mirrored `data-coverage` / `data-phase` attributes. Only `b-no` rows are offerable candidates; `b-never` is permanently off the menu. Update the badge, the note, and the attributes together.

- **The feature was already in the scan** — change its badge to `b-in` (or `b-part`, if this feature only covers part of what the row describes) with the phase number, and rewrite its `.cov` note to say what now ships and what still doesn't. Link the feature-plan concept from the note. Keep the row in its bucket at its existing rank.
- **The feature was not in the scan** — add a row in the bucket it belongs to, in that bucket's row shape, badged the same way, with a note that it came from this interview rather than from research. The scan is meant to be the complete inventory; a feature that bypassed it leaves a hole.
- **The scope line moved something out** — a capability this interview deferred keeps or gets `b-no`; one explicitly ruled out gets `b-never` with the reason in its note.
- **The scan is stale** — if you flagged it in Step 0.5 and the user declined a refresh, still record this feature's outcome and note in the header meta that the research predates it. Never refresh silently here; refreshing is research, this step is bookkeeping.

Three standing rules, inherited from `dev-initial-interview`:

- **Never delete a row.** Its sentiment and comparables are the evidence behind the decision; deleting them makes the decision unauditable.
- **Never re-rank the Top 10.** The ranking records what the market said at research time. When events overtake a row's rank, say so in its `.cov` note.
- **Update the tally and add a dated footer line** for this pass, so the next run knows when the marking was last true.

If a row you just marked sits in the Required bucket, check whether the MVP now meets the rest of that bucket. A table stake still carrying `b-part` or `b-no` is worth naming in the final report.

**5. `docs/architecture.html`, if it exists — do not edit it here.**

This skill never writes to the architecture doc. It is owned by `dev-architecture`, and a feature interview editing it is how the two skills end up disagreeing.

- **Outcome 1 or 2** (fits, or fits with a local exception) — nothing to do. The exception lives in the feature brief, not in the architecture.
- **Outcome 3** (needs an amendment) — the amendment was made by `dev-architecture` before this interview resumed. Confirm the Amendments entry is there and dated, and reference it from the feature brief. If it is missing, the amendment never landed; stop and say so.

**6. Report what changed.** Close with a short list: every file touched, the one-line change in each, which architecture-fit outcome the feature landed on, and the next command — normally `dev-plan-phase <N>` to turn the new phase into implementation tasks. If a doc was missing, say which one and what didn't get updated because of it.

Still no code.

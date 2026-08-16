---
name: dev-add-feature
description: Run a relentless, branch-by-branch interview that turns a rough feature idea into a complete integration plan for an existing app — grounded in the current plan, the repo, and any prior competitive scan at docs/competitive-scan.html. Never writes code. Use this whenever the user wants to add, extend, bolt on, "think through," or decide what to build next for something that already exists or is already planned, even if they never say the word "interview." Companion to dev-initial-interview, which covers net-new applications; use dev-add-feature instead once there is an existing plan or codebase to fit into.
---

# Dev Add Feature

Turn a rough feature idea into a complete, mutually understood plan for folding it into an existing application — by first understanding what already exists, then interviewing the user until nothing material is unresolved.

## Hard constraint

Do not write application code. Not scaffolding, not a "quick example," not a snippet to illustrate a point. Schemas, API shapes, data models, interface contracts, and pseudocode-level structure are fine when they *are* the design decision under discussion — implementation is not. If the user asks for code mid-interview, note that it's outside this skill and offer to finish the plan first.

## Step 0: Ground yourself in what exists

Before asking anything, read the ground truth. Do not skip this — the value of this skill over a generic brainstorm is that every question is informed by the actual system.

1. **The plan** — locate and read the existing planning docs: `docs/prd.html`, phase docs under `docs/phases/`, `docs/progress.html`, README, or whatever the user points at. If the app exists only as a plan, that plan *is* the ground truth.
2. **The competitive scan** — check for `docs/competitive-scan.html`. If `dev-initial-interview` ran, this holds the Top 10 and Other buckets that were deliberately parked out of the MVP. Read it; it's the backlog this feature may already be sitting in.
3. **The repo** — if there is code, survey it: entry points, module layout, data model / migrations, routing or command surface, config, tests. Read the parts the feature will plausibly touch. Don't read everything; read enough to ask sharp questions.
4. **Conventions** — how this codebase already does the thing the feature needs (persistence, auth, background work, error handling, UI composition). New features should follow existing patterns unless there's a reason not to.

Then open with a short grounding summary — no more than a handful of lines: what you understand the app to be, which parts this feature likely touches, and the two or three biggest unknowns. Confirm it before interviewing.

If there is no plan and no repo, say so and suggest `dev-initial-interview` instead.

## Step 0.5: The competitive scan, if there is one

When `docs/competitive-scan.html` exists, it changes how this interview opens. Handle it in one of two ways depending on how the user arrived:

**They named a feature.** Check whether it's already in the scan. If it is, lead with that in the grounding summary — the entry's user-sentiment notes and which comparables ship it are evidence you already have, and it sharpens branch 1 considerably. Say what the scan found and whether it agrees with their motivation. If the scan contradicts them — everyone ships it and users still complain about it — surface that now, not later.

**They're deciding what to build next.** Present the Top 10 as the candidate menu: a short numbered list, each with its one-line description and sentiment note, ranked as the scan ranked them. Then recommend one and say why, in terms of the app's current state — what's now built, what the riskiest remaining assumption is, what the last phase exposed. Mention that Other exists and offer to show it if nothing in the ten fits.

Either way, once a feature is chosen, the scan's job is done. Run the normal branch order against it. Do not keep referring back to what competitors do — that's market evidence, and past this point the existing app and its users are the better authority.

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
7. **Architecture fit** — reuse vs. new component, dependencies added, conformance to existing conventions
8. **State and failure** — persistence, concurrency, partial failure, recovery, interaction with existing state
9. **Compatibility** — backward compatibility, existing data, existing users, feature flagging, staged rollout
10. **Non-functional** — performance impact on existing paths, security, privacy, cost delta
11. **Testing and regression** — what proves this works, what proves the old behavior still works
12. **Build plan** — task ordering, what proves the riskiest assumption first, where it lands in the existing phase plan
13. **Done criteria** — how you both know the feature is finished

Push on vagueness. "It should be fast" is not an answer; "sub-200ms on a 10k-row table" is. When an answer contradicts an earlier one — or contradicts the existing plan or code — surface the conflict immediately rather than carrying it forward.

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
7. Architecture decisions, with the reasoning kept
8. Compatibility and rollout
9. Testing and regression plan
10. Build plan — ordered tasks and where they land in the existing phase plan
11. Done criteria
12. Open risks and assumptions

Ask before writing whether the user also wants the existing plan docs (PRD, phase docs) updated to reference this feature — that's a separate action, and `dev-plan-phase` is the right tool for turning this into implementation tasks.

If this feature came out of `docs/competitive-scan.html`, mark that entry as planned and link it to this concept, so the next run doesn't offer it again as a fresh candidate.

Still no code.

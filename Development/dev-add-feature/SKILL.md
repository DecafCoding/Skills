---
name: dev-add-feature
description: Run a relentless, branch-by-branch interview that turns a rough feature idea into a complete integration plan for an existing app. It is grounded in the current plan, the settled architecture at docs/architecture.html, the repo, and any prior competitive scan at docs/competitive-scan.html. It checks the feature against the architecture instead of redesigning it. It hands back to dev-architecture for an amendment when the feature does not fit. It then folds the agreed feature back into whichever plan docs exist (docs/mvp-plan.html, docs/prd.html, docs/progress.html, docs/competitive-scan.html). It never writes code. Use it whenever the user wants to add, extend, bolt on, "think through", or decide what to build next for something that already exists or is already planned, even if they never say the word "interview". It is the companion to dev-initial-interview, which covers net-new applications. Use dev-add-feature once there is an existing plan or codebase to fit into.
---

# Dev Add Feature

Turn a rough feature idea into a complete plan for folding it into an existing application. First understand what already exists. Then interview the user until nothing material is unresolved.

## Hard constraint

Do not write application code. This ban covers scaffolding, quick examples and illustrative snippets.

Schemas, API shapes, data models, interface contracts and pseudocode-level structure are allowed. Use them only when they are the design decision under discussion.

If the user asks for code during the interview, say that code is outside this skill. Offer to finish the plan first.

## Step 0: Ground yourself in what exists

Read the ground truth before you ask anything. Do not skip this step. Every question must be informed by the actual system.

1. **The plan.** Locate and read the existing planning docs. Those are `docs/mvp-plan.html`, `docs/prd.html`, phase docs under `docs/phases/`, `docs/progress.html`, README, or whatever the user points at. If the app exists only as a plan, that plan is the ground truth. **Note which of these exist.** You need that list again at the end, when the plan docs get updated to include this feature.
2. **The competitive scan.** Check for `docs/competitive-scan.html`. If `dev-initial-interview` ran, this holds the Required, Top 10 and Other buckets with a coverage badge on every row. Read it. It is the backlog this feature may already sit in. The badges say what is already built.
3. **The architecture.** Check for `docs/architecture.html`. If `dev-architecture` ran, this is the settled technical design. It covers stack, storage, pattern, module layout, boundaries, state and failure, non-functional targets and deployment. Read it in full. Every `<h3 class="decision">` in it is a constraint this feature must fit. The `data-reversible` value tells you what a misfit would cost to undo. This file is the authority for branch 7.
4. **The repo.** Survey the code if there is any. Look at entry points, module layout, data model and migrations, routing or command surface, config and tests. Read the parts the feature will plausibly touch. Read enough to ask sharp questions. Do not read everything.
5. **Conventions.** Find how this codebase already does the thing the feature needs. That covers persistence, auth, background work, error handling and UI composition. New features should follow existing patterns unless there is a reason to depart. Where the repo and `docs/architecture.html` disagree, say so. One of them is wrong. Ask the user which one. Do not guess.

Then open with a short grounding summary. Keep it to a few lines. State what you understand the app to be. State which parts this feature likely touches. State the two or three biggest unknowns. Confirm it before you interview.

If there is no plan and no repo, say so. Suggest `dev-initial-interview` instead.

## Step 0.5: The competitive scan, if there is one

When `docs/competitive-scan.html` exists, it changes how this interview opens. Two cases apply. Pick the one that matches how the user arrived.

**They named a feature.** Check whether it is already in the scan. If it is, lead with that in the grounding summary. The entry's user-sentiment notes and the list of comparables that ship it are evidence you already have. That evidence sharpens branch 1. Say what the scan found and whether it agrees with their motivation.

Surface a contradiction now. An example is a feature that everyone ships and users still complain about.

**They are deciding what to build next.** Present the Top 10 as the candidate menu. Offer only the rows still badged `b-no`, which means Not in MVP.

Rows badged `b-in` or `b-part` are already built or partly built. Rows badged `b-never` are ruled out. The coverage badges exist to stop you offering any of those as a fresh candidate.

A `b-part` row is still fair game as a widening. Say that word explicitly when you offer it.

Keep the scan's original ranking. Do not re-rank to fill the gaps.

Then recommend one row and say why, in terms of the app's current state. Cite what is now built, the riskiest remaining assumption, and what the last phase exposed. Mention that an Other bucket exists. Offer to show it if nothing in the ten fits.

Once a feature is chosen, the scan's job as an input is done. It still gets updated at the end. See the Finishing step.

Run the normal branch order against the chosen feature. Stop referring back to what competitors do. That is market evidence. Past this point the existing app and its users are the better authority.

Say so if the scan is stale enough to matter. Staleness means the app has changed a lot, or the research is about a year old. Offer to refresh it before choosing. Never refresh silently.

## Method

Walk the design tree. Start at the root, which is what the feature is and why it is needed now. Then descend branch by branch.

Resolve decisions in dependency order. Settle a decision that constrains three others before those three. Say so when you do it: "This one gates the data model. Let us settle it first."

Typical branch order, adapted to the domain:

1. **Motivation.** What is missing today, who feels it, and what "done" changes for them
2. **Fit.** Where this sits relative to the existing core loop. Does it extend the loop, branch it, or replace part of it
3. **Scope line.** What is in this feature, what is deferred, and what is never built
4. **Blast radius.** Every existing module, table, endpoint, screen and job this touches. Also what breaks if it is wrong
5. **Data model changes.** New entities, changed entities, migrations, backfill and rollback
6. **Interfaces.** New or changed surfaces, inputs, outputs, and the contract with existing ones
7. **Architecture fit.** Reuse or a new component, dependencies added, and conformance to existing conventions. When `docs/architecture.html` exists, this branch *checks* against it. It does not decide. See "The architecture handoff" below
8. **State and failure.** Persistence, concurrency, partial failure, recovery, and interaction with existing state
9. **Compatibility.** Backward compatibility, existing data, existing users, feature flagging and staged rollout
10. **Non-functional.** Performance impact on existing paths, security, privacy and cost delta
11. **Testing and regression.** What proves this works, and what proves the old behavior still works
12. **Build plan.** Task ordering, what proves the riskiest assumption first, and where it lands in the existing phase plan
13. **Done criteria.** How you and the user both know the feature is finished

Ask again when an answer is vague. "It should be fast" is not an answer. "Sub-200ms on a 10k-row table" is an answer.

Surface a conflict at once when an answer contradicts an earlier one, the existing plan, or the code. Do not carry it forward.

## The architecture handoff

When `docs/architecture.html` exists, this skill does not decide architecture. It decides whether the feature fits the architecture that is already agreed.

Branch 7 has exactly three possible outcomes. Land on one of them out loud.

**1. It fits.** The feature uses the existing pattern, layout, storage and boundaries as they stand. Note which `data-decision` slugs it leans on. Move to branch 8. Nothing else is needed.

**2. It fits with a local exception.** The feature needs one narrow departure. Examples are an extra library, a new table shape, or a background worker where none existed. Record it in the feature brief as an exception. Name the decision slug it departs from. Say plainly what would need to change if the exception ever became the rule. Do not edit `docs/architecture.html` for a one-off.

**3. It does not fit.** The feature needs a decision in `docs/architecture.html` to change. Examples are a different storage engine, a new boundary, a pattern the layout cannot express, or a non-functional target the current design cannot hit. **Stop the interview here.** Do not design the replacement yourself. Do not carry on to branches 8 through 13.

For outcome 3, do this:

- Name the decision slugs that would have to change. Give their `data-reversible` values. An `expensive` value should make both of you slow down.
- Tell the user the feature requires an architecture change. Offer `dev-architecture` in amendment mode. It reads the existing file, changes only the affected decisions, and logs the amendment.
- Offer the alternative in the same message. That alternative is a narrower version of the feature that fits the current architecture. It is often the better trade. The user cannot weigh it unless you put it beside the amendment.
- Resume this interview once the amendment lands. Re-read `docs/architecture.html` first.

**Branches 8 and 10 stay feature-local.** Ask state and failure, and non-functional, only about this feature's impact. That means what it adds to existing paths. The system-wide targets belong to `docs/architecture.html`.

If answering one of these would change a system-wide target, you are in outcome 3.

## How to write

Write every message in ASD-STE100 Simplified Technical English. STE is a controlled English standard that limits you to plain approved words.

- Use plain words and active voice.
- Put one idea in each sentence. Keep sentences under 20 words.
- Keep paragraphs to 3 sentences or fewer.
- Explain a technical word right after you use it.
- Do not use idioms, metaphors or figures of speech.
- Do not use contrast pairs such as "X, not Y". State only what is true.
- Do not use semicolons or em dashes. Write two sentences.
- Do not restate an idea in different words for effect.
- Give only the facts the user needs to act on. Leave out dates, IDs and background unless they ask.
- Say what you did, whether it worked, and what the user does next.
- Keep paths and commands exact.

These rules cover your chat messages. They also cover the prose inside the feature plan document. The HTML format rules are separate. See "Finishing".

## Question format

Ask one question at a time. Ask a tight cluster only when the questions are genuinely coupled.

Give 2 or 3 options for every question. Write one line for each option. Say what it costs and what it buys.

Then name the option you recommend and give the reason. The user should be able to reply with the number and move on.

Anchor every recommendation. Cite the repo, the existing plan, an architecture decision slug, or an answer the user already gave. Say which one you used. A recommendation with no anchor is a guess.

Write the options in prose. Never use tappable-button, single-select or multiple-choice widgets. The user types a free-text answer. The real answer is often something you did not list, so invite that.

**Example:**

> **Storage for saved filters:** where should a saved filter live?
>
> 1. A new `saved_filters` table. The existing schema already normalizes per-user collections this way.
> 2. A JSON column on `users`. Fewer moving parts. It blocks the "share a filter" case you mentioned.
> 3. Local browser storage. No migration at all. Filters are then lost on a new device.
>
> *I recommend 1.* It matches the `storage-schema` decision in `docs/architecture.html` and keeps sharing open.
>
> Type your answer. Anything outside this list is fine too.

Do not give a list of options with no recommendation. The recommendation is the value.

## Conciseness

Keep responses short. State the question, the options, the recommendation and the reason.

Do not add preamble. Do not recap decisions that are already settled. Do not write explanatory essays. The user will ask when they want more.

## Tracking state

Keep a running list of settled decisions and open branches. Show it after a major branch closes, or when the user asks where things stand. Do not print it every turn.

## Finishing: the feature plan document

The interview ends when every branch is closed and the user agrees the picture is complete. Then write the plan as an **OKF-HTML concept**. Follow the `okf-html-library` skill for the mechanics. That covers the library root, the `okf-meta` block, the relative `theme.css` link, `templates/concept.html` and index regeneration. Do not hand-write styling. Styling lives in `theme.css`.

Metadata conventions for this document:

- `type`: `Feature Plan`. Reuse the library's existing type if one already fits.
- `title`: the feature name
- `description`: one line naming the feature and the app it extends
- `tags`: the app name, plus the areas touched
- `resource`: the repo or plan URI if there is one. Otherwise `null`.

Body sections, as semantic HTML:

1. Summary and motivation
2. Fit with the existing application, with links to related concepts where the library has them
3. Scope. In, deferred, never
4. Blast radius. The concrete list of touched components
5. Data model changes, and the migration and rollback approach
6. Interfaces and contracts
7. Architecture fit. Which of the three outcomes this landed on, the `data-decision` slugs the feature leans on, any local exception with what it departs from, and the reasoning kept. If an amendment to `docs/architecture.html` was needed, name it and the date it landed. Do not restate the new decision here
8. Compatibility and rollout
9. Testing and regression plan
10. Build plan. Ordered tasks, and where they land in the existing phase plan
11. Done criteria
12. Open risks and assumptions

## Finishing: fold the feature back into the plan docs

Writing the feature plan is not the last step. The MVP plan, the PRD and `progress.html` still describe an app without this feature. That plan has already drifted. Close the loop.

**This step is required.** Do not ask whether the user wants the plan docs updated. They do. That is what "add a feature" means.

Do ask for confirmation on the *substantive* choices called out below. Those are the phase number and the in-scope or deferred call. Show the user each edit you intend to make before you write it.

Check for each doc and act only on the ones that exist. If none exist, say so and stop. There is nothing to update.

**1. `docs/mvp-plan.html`, if it exists.**

- The feature is in scope for the MVP. Add it to the in-scope checklist (`<li data-checked="true">`). Update the sections it materially changes. Those are data model, interfaces, milestones and done criteria. Write one or two lines per section plus a link to the feature-plan concept. Do not restate the whole feature plan.
- The feature is deferred past the MVP. Add it to the deferred checklist (`<li data-checked="false">`) with a link. Change nothing else. Deferred features do not get to edit the data model.
- In both cases, append an entry to the `Amendments` section. Give the date, the feature name, what changed, and the fact that `dev-add-feature` changed it. If the doc has no `Amendments` section, add one.

**2. `docs/prd.html`, if it exists.**

The PRD's HTML contract is machine-readable. `dev-create-progress` parses it. Preserve the shapes exactly. See `dev-create-prd`.

- **MVP Scope.** Add the feature to the in-scope or out-of-scope checklist. Match the scope line settled during the interview.
- **User Stories.** Add the one or two stories the feature introduces. Use the existing "As a … I want … so that …" format.
- **Implementation Phases.** Confirm this decision with the user before you write. Ask whether this lands in an existing phase or becomes a new one. Then:
  - *Existing phase.* Add the feature's deliverables to that phase's checklist. Extend its validation criteria.
  - *New phase.* Append a new heading in the exact required shape, `<h3 class="phase" data-phase="<N>">Phase <N>: <name></h3>`, with Goal, Deliverables checklist and Validation criteria beneath it. Never put a time estimate inside the heading text. It goes in a following `<p class="meta">`. Pick the next unused phase number. Use a sub-phase (`data-phase="3b"`) when the feature slots between two existing phases and renumbering would break already-planned work. **Never renumber existing phases.** Phase docs, branch names and `progress.html` all key off those numbers.
- Update the sections the feature genuinely changes. Those may include Technology Stack, API Specification, Security and Configuration, and Risks. Leave every section it does not touch alone.

**3. `docs/progress.html`, if it exists.**

Do not hand-edit it. Re-run `dev-create-progress`. It regenerates the index from the phase docs and the PRD.

A feature added to the PRD as a new phase with no phase doc yet correctly appears as `data-status="not-planned"`. That is the intended state. It tells the autonomous routine to stop and ask for `dev-plan-phase <N>`.

Regeneration preserves existing task statuses and blockers. If `dev-create-progress` warns that it would clobber `done` or `skipped` marks or blocker blockquotes, stop. Show the user the warning. Do not force it.

If the PRD is absent and `progress.html` exists, say that the new feature cannot be indexed without a PRD phase. Leave `progress.html` alone.

**4. `docs/competitive-scan.html`, if it exists.**

The scan is the standing backlog and the market evidence behind it. If it still shows this feature as unbuilt after the interview, the next run offers it again as a fresh candidate.

**The badge is the contract.** Each row carries exactly one coverage badge. The values are `b-in` (In MVP · P#), `b-part` (Partial · P#), `b-no` (Not in MVP) and `b-never` (Never). Each row also carries a `.cov` note explaining the mark. Newer scans also carry mirrored `data-coverage` and `data-phase` attributes.

Only `b-no` rows are offerable candidates. A `b-never` row is permanently off the menu. Update the badge, the note and the attributes together.

- **The feature was already in the scan.** Change its badge to `b-in` with the phase number. Use `b-part` instead if this feature covers only part of what the row describes. Rewrite its `.cov` note to say what now ships and what still does not. Link the feature-plan concept from the note. Keep the row in its bucket at its existing rank.
- **The feature was not in the scan.** Add a row in the bucket it belongs to, in that bucket's row shape, badged the same way. Note that it came from this interview instead of from research. The scan is meant to be the complete inventory. A feature that bypassed it leaves a hole.
- **The scope line moved something out.** A capability this interview deferred keeps or gets `b-no`. A capability explicitly ruled out gets `b-never`, with the reason in its note.
- **The scan is stale.** You flagged it in Step 0.5 and the user declined a refresh. Still record this feature's outcome. Note in the header meta that the research predates it. Never refresh silently here. Refreshing is research. This step is bookkeeping.

Three standing rules, inherited from `dev-initial-interview`:

- **Never delete a row.** Its sentiment and comparables are the evidence behind the decision. Deleting them makes the decision unauditable.
- **Never re-rank the Top 10.** The ranking records what the market said at research time. When events overtake a row's rank, say so in its `.cov` note.
- **Update the tally and add a dated footer line** for this pass. The next run then knows when the marking was last true.

If a row you just marked sits in the Required bucket, check whether the MVP now meets the rest of that bucket. Name any table stake still carrying `b-part` or `b-no` in the final report.

**5. `docs/architecture.html`, if it exists. Do not edit it here.**

This skill never writes to the architecture doc. `dev-architecture` owns it. A feature interview that edits it makes the two skills disagree.

- **Outcome 1 or 2.** The feature fits, or fits with a local exception. Nothing to do. The exception lives in the feature brief.
- **Outcome 3.** The feature needs an amendment. `dev-architecture` made that amendment before this interview resumed. Confirm the Amendments entry is there and dated. Reference it from the feature brief. If it is missing, the amendment never landed. Stop and say so.

**6. Report what changed.** Close with a short list. Give every file touched and the one-line change in each. Give the architecture-fit outcome the feature landed on. Give the next command, which is normally `dev-plan-phase <N>` to turn the new phase into implementation tasks. If a doc was missing, say which one and what did not get updated because of it.

Still no code.

---
name: dev-architecture
description: Run a branch-by-branch architecture interview that turns a settled MVP scope into an agreed technical design at docs/architecture.html. It writes no code, ever. It settles how the app gets built. That covers language, storage, architecture pattern, module layout, boundaries, failure handling, non-functional targets, deployment and reversibility risk. It runs on a scope-frozen gate, after dev-initial-interview and before dev-create-prd and dev-plan-phase. It also runs in amendment mode when the architecture already exists. Amendment mode names every doc the change made stale and returns to whatever run was paused. Use it whenever the user wants to design, decide, review or change the architecture, stack, tech choices, storage, project structure or design patterns. That includes "what should I build this in" and "pick the stack". Also use it when a request would tempt you to settle technical design inside a requirements or feature interview.
---

# Dev Architecture

Turn a settled MVP scope into a technical design that you and the user agree on. Record it in one place that every downstream skill reads.

`dev-initial-interview` settles **what gets built**. This skill settles **how it gets built**. Keep the two apart in both directions.

If a question here would change what the app does, it belongs to `dev-initial-interview` or `dev-add-feature`. Say so and park it.

## Hard constraint

Do not write application code. This ban covers scaffolding, quick examples and illustrative snippets.

Schemas, interface contracts, directory trees, config shapes and pseudocode-level structure are allowed. Use them only when they are the design decision under discussion.

If the user asks for code during the interview, say that code is outside this skill. Offer to finish the design first.

## Step 0: The scope-frozen gate

This skill runs when the MVP in-scope list has stopped changing. The gate is that state. The gate is not a fixed position in the pipeline. Check it before you ask anything else.

**Read `docs/mvp-plan.html` first.**

- **It does not exist.** Stop. There is no scope to design against. Point the user at `dev-initial-interview` and offer to run it.
- **It exists.** Read the Scope section's in-scope checklist (`<li data-checked="true">`). Read the Constraints section. Read the Amendments section at the end.
- **`docs/architecture.html` already exists.** This is not a fresh run. Go to "Amendment mode" below. Establish which of its four entry points brought you here before you do anything else.

**Then check for movement.** Scope is frozen when two things are true. Nothing has moved on or off the in-scope list recently. The user is not mid-thought about another feature.

The Amendments section is the signal. A run of recent entries that add features means scope is still open.

Ask once, with a recommendation, and take the answer:

> **Scope check:** the plan has 11 in-scope items. It was last amended on 3 March, when saved views were added. Design the architecture against that set now?
> *Recommend yes.* Nothing on the deferred list changes the shape of what we would build. Waiting only delays the decisions everything else needs.

If the user says more features are coming first, stop. Point them at `dev-add-feature`. Do not design against scope that is still changing. Do not start with the easy branches. Later features often move those branches.

**Also read these files when they are present:**

- `docs/mvp-plan.html` → Constraints → **Open architecture inputs**. This is your starting agenda, in the user's own words. Branch 1 works through it.
- `docs/feature-*.html`. These are feature briefs from `dev-add-feature`. Read them for anything the plan summarizes in one line.
- `docs/design-system.html`. This comes from `dev-ui-update`. Design decisions constrain the surface and sometimes the stack. Read it before branch 2.
- `docs/competitive-scan.html`. Read only the market-wide signals note. Sentiment about data lock-in, cost or offline use is architecture input. Nothing else in the scan is.

## House defaults

Two decisions have a standing answer. Offer each as a recommendation. Take a one-word confirmation.

- **Runtime and stack: C# / .NET.** Use the current LTS unless the user names a version. LTS means Long Term Support.
- **Architecture pattern: Vertical Slice.** Organize by feature under `Features/{FeatureName}/`. Keep the endpoint or page, the handler, the models and the validation together. Extract to `Shared/` only for genuine cross-cutting concerns.

These match the profile that `dev-claud-md` emits. An unchanged answer costs nothing downstream. An override does cost something. See "The override warning".

**Offer the default. Never apply it silently.** The default is a starting position. The user gets one clear chance to say no on each one. Give a real reason if they ask why. A default you apply without showing it is a failure.

**Drop the default when the constraints contradict it.** A browser extension, a shell utility, an embedded target or a stated third-party requirement can make C# the wrong answer.

When the inherited constraints point elsewhere, lead with the fitting recommendation. Name the house default you are departing from. The user can then ask for it back.

## Clean slate

`dev-initial-interview` bans sourcing decisions from memory. That ban applies here in full. Do not source a decision from any of these:

- Stored user preferences, or a profile of how this user usually builds things
- Another project's stack, pattern, layout or naming conventions
- A decision "we made last time" that nobody has restated

The house defaults above are **not** memory. They are written into this skill file. That makes them a standing instruction the user wrote deliberately. That is a legitimate input.

A stack recalled from another project is not a legitimate input, even when it matches. If it is right, it is right on this project's merits. The confirm question will reach it anyway.

Only four inputs are legitimate here. They are the plan and its sibling docs, what the user says in this conversation, the house defaults, and what you and the user reason out together. A decision with no source in that list has no source.

## Method

Walk the design tree in dependency order. Settle a decision that constrains three others first. Say so when you do it: "This one picks the layout for us. Let us settle it first."

Branch order, adapted to the domain:

1. **Constraints review.** Work the Open architecture inputs list from the plan. Answer every parked question, or carry it to Open questions in writing. Drop nothing silently.
2. **Runtime and stack.** Language, framework, versions. The house default applies.
3. **Storage.** Engine, schema shape, migrations, seed and test data.
4. **Architecture pattern.** Vertical Slice, Clean, layered, MVC, hexagonal, or plain. Make one choice. Give the reason. The house default applies.
5. **Module layout.** The folder shape the pattern implies. Also the import direction rule. That rule says which parts may reference which, and which direction is forbidden.
6. **Boundaries.** Internal seams, and every outside service, SDK or API. State what is wrapped and what is called directly. State what happens when each one is unavailable.
7. **State and failure.** Persistence, concurrency, transactions, error surfaces, retry and recovery. State what a partial failure leaves behind.
8. **Non-functional.** Scale, latency, throughput, security, privacy, cost ceiling. Give numbers.
9. **Deployment.** Target, environments, configuration and secrets, CI.
10. **Risks and reversibility.** For each decision above, state how expensive it is to undo. Give this branch full attention. Do not compress it into a closing sentence.

**Order matters here.** Stack limits the workable patterns. Pattern dictates layout. Storage and boundaries decide most of state and failure. Out-of-order decisions force you to re-decide backwards.

Ask again when an answer is vague. "It should scale" is not an answer. "One user now, under 50k rows, no concurrent writers" is an answer.

Surface a conflict at once when an answer contradicts an earlier one or contradicts the MVP plan. Do not carry it forward.

**When a branch turns out to be scope.** The user reaches for a feature that is not on the in-scope list. Stop. Name it. Offer `dev-add-feature`. Do not widen the MVP from inside an architecture interview.

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

These rules cover your chat messages. They also cover the prose inside `docs/architecture.html`. The HTML format rules are separate. See "Finishing".

## Question format

Ask one question at a time. Ask a tight cluster only when the questions are genuinely coupled.

Give 2 or 3 options for every question. Write one line for each option. Say what it costs and what it buys.

Then name the option you recommend and give the reason. The user should be able to reply with the number and move on.

Anchor every recommendation. Cite the MVP plan, an inherited constraint, a sibling doc, or an answer the user already gave. Say which one you used. A recommendation with no anchor is a guess.

Write the options in prose. Never use tappable-button, single-select or multiple-choice widgets. The user types a free-text answer. The real answer is often something you did not list, so invite that.

**Example:**

> **Storage:** how should the MVP store its data?
>
> 1. SQLite. One file, no server to run. EF Core keeps a later move cheap.
> 2. Postgres. More setup now. It pays off if a hosted version arrives within a year.
> 3. JSON files on disk. Simplest to start. It fails as soon as you need queries.
>
> *I recommend 1.* The scope is a single-user desktop app, so a server buys nothing today.
>
> Type your answer. Anything outside this list is fine too.

Do not give a list of options with no recommendation. The recommendation is the value.

## Conciseness

Keep responses short. State the question, the recommendation and the reason.

Do not add preamble. Do not recap decisions that are already settled. Do not write explanatory essays. The user will ask when they want more.

## Tracking state

Keep a running list of settled decisions and open branches. Show it after a major branch closes, or when the user asks where things stand. Do not print it every turn.

## Finishing: the architecture document

The interview ends when every branch is closed and the user agrees the picture is complete. Then write the design to a **fixed path: `docs/architecture.html`**, beside the plan it serves. If a file is already there, you are in Amendment mode. See below.

The path is fixed on purpose. `dev-create-prd`, `dev-plan-phase`, `dev-claud-md` and later `dev-add-feature` runs all look for `docs/architecture.html` by name. An architecture with no fixed location gets copied into several files.

**Format.** Emit a self-contained HTML document. Use the same scaffold and element conventions as `dev-create-prd`. That means a doctype, a `<head>` with the embedded `<style>` block, one `<h2>` per section, `<h3>` for subsections, real `<table>` markup, `<pre><code>` for directory trees and config shapes, and `<ul class="checklist">` with `data-checked` where a list is genuinely in or out.

**Sections, in order:**

1. Context. What this design serves, with a link to `docs/mvp-plan.html` and the inherited constraints listed plainly
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

**Every decision carries machine-readable attributes.** Sections 2 through 9 are decisions. Each `<h3>` inside them takes this shape:

```html
<h3 class="decision"
    data-decision="storage-engine"
    data-status="settled"
    data-reversible="cheap">Storage engine: SQLite</h3>
```

- `data-decision`. A stable kebab-case slug. Amendments reference it, so it never changes once written.
- `data-status`. One of `settled`, `open` or `amended`.
- `data-reversible`. One of `cheap`, `moderate` or `expensive`. Fill this from branch 10. Make it honest. An expensive decision marked cheap is how a project ends up rewritten.

A human reads the visible heading. Tooling reads the attributes. Keep them in sync.

**Three rules for the document.** `dev-initial-interview` applies these same three rules to the plan. Each rule is separate. Satisfying one does not satisfy the others.

1. **Self-contained HTML.** It renders with no network. Embed styles in a `<style>` block. Use no external stylesheet, no CDN script, no remote font and no hotlinked image.
2. **No memory.** No decision may come from stored preferences, a user profile or another project. See "Clean slate".
3. **Closed doc set.** Links point only to files inside `docs/`. Never link a web page, a vendor doc or a blog post. Name a library with its version in the text. The reader can search for it.

**Open questions.** Put anything you genuinely cannot decide now into section 11. Give what would settle it, and the date by which it must be settled. An empty Open questions section is suspicious. Say so if the design really has none.

## The override warning

`dev-claud-md` hard-codes the .NET / Vertical Slice profile as its default. That is deliberate and stays. It means `dev-claud-md` can silently overwrite an override you settle here.

If the user overrides the stack or the pattern, do three things before you report.

1. Record the override in `docs/architecture.html` with `data-status="settled"`. State the reason for departing from the house default in the section text.
2. Add an Open questions entry. It says that `dev-claud-md` must be pointed at `docs/architecture.html` for this project instead of run on its default.
3. Tell the user in the report, in plain words. Do not put it in a footnote. This is the one failure mode of keeping the default. The warning is what makes keeping it safe.

If neither is overridden, say so in one line. The downstream default matches. Nothing needs pointing anywhere.

## Amendment mode

`docs/architecture.html` is a living document. A later `dev-add-feature` run hands back here when a feature does not fit the architecture.

When the file already exists:

1. **Read it in full first.** Then read what changed in `docs/mvp-plan.html` since the architecture's last dated pass.
2. **Name the affected decisions by slug.** Confirm the list with the user before you touch anything. A feature usually moves two or three decisions.
3. **Run only the affected branches.** Do not re-open settled decisions that the change does not touch. Do not re-open the house defaults.
4. **Never delete a decision.** Flip its `data-status` to `amended`. Revise the section text. Keep the original reasoning visible. That reasoning is why the decision was made. Deleting it makes the change unauditable.
5. **Append an Amendments entry** as a `<p class="meta">`. Give the date, the decision slugs that changed, what changed, and what caused it. The history must be readable without a diff.
6. **Re-check the override warning.** An amendment can introduce one where there was none before.

### Note how you got here

Amendment mode has four entry points. The right closing report depends on which one it was. Establish this at the start.

1. **`dev-add-feature`** paused a feature interview because the feature did not fit.
2. **`dev-plan-phase`** could not plan a phase within the settled decisions.
3. **`dev-orchestrate`** stopped its loop on `ARCHITECTURE-MISFIT`.
4. **The user asked directly.** An example is "we need to move to Postgres". Nothing is paused.

Ask if the entry point is unclear from the conversation. One line is enough. It changes what you tell them at the end.

### Closing an amendment

**Do not use the fresh-run Next step list below.** That list is written for a project with no PRD yet. Following it after an amendment sends the user to create a document that already exists. An amendment closes in its own way.

**First, name what is now stale.** An amendment silently invalidates every document that copied the old decision. List them by name, with the changed slugs beside them:

- `docs/prd.html`. Its Core Architecture and Technology Stack sections describe the superseded decision. The file already exists. Update it.
- `CLAUDE.md`. This file affects the most future work. Every future agent session reads it and inherits the old convention. An amendment that never reaches `CLAUDE.md` does not take effect.
- `docs/phases/*.html`. Any *unstarted* phase doc whose Notes cite a slug you just amended was planned against the old decision. **Grep the Notes sections for the amended slugs and list every phase number you find.** Include every affected phase. Already-merged phases are history. Leave them.
  - **Then add the triggering phase by hand.** Do this whether or not a doc exists for it. Entry points 2 and 3 mean `dev-plan-phase` stopped before writing anything, so the grep cannot find that phase. A doc left over from an earlier planning pass may cite no slugs at all. That phase is certain to need re-planning. The grep is certain to miss it. Add it explicitly. Every check below will miss it otherwise.
- `docs/mvp-plan.html`. Include it only if the amendment changed a constraint recorded in its Constraints section.

**Then give the refresh order.** Say that the order is required.

1. `dev-create-prd`. Update the affected sections.
2. `dev-claud-md`. Run it after the PRD, so the repo's house rules match.
3. `dev-plan-phase <N>`. Run it once for every phase number on the stale list above. Include every affected phase. This step must run after step 2. `dev-plan-phase` reads `CLAUDE.md` for its conventions. Re-planning before the house rules are refreshed writes the superseded decision back into the new phase doc.
4. `dev-create-progress`. Run it last, to rebuild the index from the re-planned phase docs. If it warns that it would clobber `done` or `skipped` marks, stop. Show the user the warning. Do not force it.

If no unstarted phase doc cites an amended slug, say so and skip steps 3 and 4. Keep the list short and accurate.

**Then return to what was paused.** The closing depends on the entry point.

- **From `dev-add-feature`.** Send them back to finish that feature interview. Say explicitly to re-read `docs/architecture.html` first, so the interview resumes against the amended decisions. Name the feature so they know which interview.
- **From `dev-plan-phase`.** Nothing extra is needed, provided you added the blocked phase to the stale list by hand. Step 3 then re-plans it with every other affected phase. If you left it off, no list covers it and nothing re-plans it. Go back and add it now. Do not mention it separately here. One owner for the re-plan stops the user running it twice.
- **From `dev-orchestrate`.** The loop stopped and will not resume itself. Tell them to re-run it after the refresh order above completes. Otherwise it will plan the next phase from a stale `CLAUDE.md`.
- **From a direct request.** Nothing is paused. The refresh order is the whole of what remains. Say so plainly, so they do not wonder what they forgot. This entry point has no interview to remind them. State the stale list clearly.

## Next step

**This section is for a fresh run only.** After an amendment, use "Closing an amendment" above.

Report every file written. Report the decision tally by `data-reversible`. Report any open questions. Report the override warning if one applies.

Then name the next step:

- `dev-create-prd`. It turns the plan and this design into `docs/prd.html`. Its Core Architecture and Technology Stack sections cite this file instead of re-deciding it.
- `dev-claud-md`. Run it after the PRD exists, so the repo's house rules match the design.
- `dev-add-feature`. Use it when a new feature needs folding in later. It comes back here if the feature does not fit.

Still no code.

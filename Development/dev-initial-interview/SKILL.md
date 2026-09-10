---
name: dev-initial-interview
description: Run a relentless, branch-by-branch requirements interview that turns a vague app idea into a complete, agreed MVP plan written to docs/mvp-plan.html. It writes no code, ever. It reads the prior art scan at docs/competitive-scan.html, written by dev-competitive-scan, and uses it to make the scope line an evidence-backed decision. When the scan is missing it asks whether to continue without it. At the end it marks every scan row against the finished plan. Use this whenever the user is brainstorming, scoping, spec'ing, or "thinking about building" a new application, tool, service, or major feature, even if they never say the word "interview". Also use it when a request would otherwise tempt you to start coding before scope is settled.
---

# Dev Initial Interview

Turn a rough idea into a complete MVP plan that you and the user agree on. Interview the user until nothing material is unresolved.

## Hard constraint

Do not write application code. This ban covers scaffolding, quick examples and illustrative snippets.

Schemas, API shapes, data models and pseudocode-level structure are allowed. Use them only when they are the design decision under discussion.

If the user asks for code during the interview, say that code is outside this skill. Offer to finish the plan first.

## Clean slate

Every project starts from nothing. Nothing about this app's design or function comes from persistent or global memory. That bars stored user preferences, another project's decisions, and any profile of how this user usually builds things.

None of the following may be sourced from memory:

- The stack, runtime, storage, framework or deployment target
- The architecture, directory layout or naming conventions
- The scope line, the core loop, the data model or the target user
- Anything else that ends up in `docs/mvp-plan.html`

Only three inputs to a design decision are legitimate. They are what the user says in this conversation, what the prior art scan found, and what you and the user reason out together. A decision with no source in those three has no source.

**When memory seems relevant, ask instead of assuming.** "You used SQLite last time" is not a reason for this app to use SQLite. This app might not have a database. Raise it as a question with your recommendation, the way you would any other branch. Let the user decide on this project's merits.

**"Unless clearly stated otherwise" means stated here, now.** "Same stack as my other app" is a clear instruction. Do not fill in the specifics from memory. Ask them to name the pieces. Confirm each one. Write them into the plan explicitly.

The MVP plan must stand on its own. A reader with no access to any memory should be able to build from it with no missing premise.

This rule covers the *product*. It does not cover the *conversation*. How the user likes to be talked to still applies normally. That covers brevity, formatting and tone.

## Step 0: The prior art scan

Before the scope branch, learn what comparable applications already ship and what their users complain about. `dev-competitive-scan` does that research and writes it to `docs/competitive-scan.html`. This skill reads that file. It does not run the research.

**Check for `docs/competitive-scan.html`.**

- **It exists and is current.** Read it. Summarize the buckets in a few lines. Name the market-wide signals. Move on.
- **It exists and is stale.** Staleness means the app has changed materially, the research is roughly a year old, or the market moved. Say so. Name what looks stale. Offer to run `/dev-competitive-scan` to refresh it before the interview. Continue with the stale file if the user declines. Note in the plan that the scan predates the interview.
- **It does not exist.** Stop and ask. Give the user two options in prose. Option 1 is to run `/dev-competitive-scan` first and come back. Option 2 is to continue without a scan. Recommend option 1, because the scope line is a guess without market evidence. Do not run the scan yourself. Do not continue until the user answers.

**Continuing without a scan.** When the user chooses to continue, the scope branch has no market evidence. Anchor every recommendation on the user's answers and on reasoning done in this conversation. Say plainly in the scope branch that no comparables were surveyed. Skip the coverage pass at the end. Record in the plan's "Open risks and assumptions" section that no competitive scan was done. Name `/dev-competitive-scan` in the next steps so the gap can be closed later.

**What the scan gives you.** The file holds three buckets:

- **Required.** Table stakes. Near-universal across the comparables. **Only this bucket feeds the MVP scope directly.**
- **Top 10.** The ranked differentiator decisions.
- **Other.** Everything else found. Unranked.

Top 10 and Other are **recorded**. They are not decided here. They are the reviewed backlog that `dev-add-feature` works through later, one at a time, against a working app. Do not argue them into the MVP during this interview.

Present the buckets briefly, with a recommendation on what belongs in the MVP and what should be parked. Then continue the interview. The scan informs branch 3, the scope line. It informs nothing after that.

## Method

Walk the design tree. Start at the root, which is what the app is and who it is for. Then descend branch by branch.

Resolve decisions in dependency order. Settle a decision that constrains three others before those three. Say so when you do it: "This one gates the data model. Let us settle it first."

Typical branch order, adapted to the domain:

1. **Problem and user.** What breaks today, who feels it, and what "solved" looks like
2. **Core loop.** The one thing the user does over and over. Everything else is support
3. **Scope line.** What is in the MVP, what is deferred, and what is never built. Argue it against the scan's buckets
4. **Data model.** Entities, relationships, ownership and lifecycle
5. **Interfaces.** Surfaces such as web, CLI, API or desktop. Also inputs and outputs
6. **Architecture.** Runtime, storage, external dependencies and deployment target
7. **State and failure.** Persistence, concurrency, errors and recovery
8. **Non-functional.** Scale, latency, security, privacy and cost ceiling
9. **Build plan.** Milestones, ordering, and what proves the riskiest assumption first
10. **Done criteria.** How you and the user both know the MVP is finished

Ask again when an answer is vague. "It should be fast" is not an answer. "Sub-200ms on a 10k-row table" is an answer.

Surface a conflict at once when an answer contradicts an earlier one. Do not carry it forward.

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

These rules cover your chat messages. They also cover the prose inside `docs/mvp-plan.html` and the notes you add to `docs/competitive-scan.html`. The HTML format rules are separate. See "Finishing".

## Question format

Ask one question at a time. Ask a tight cluster only when the questions are genuinely coupled.

Give 2 or 3 options for every question. Write one line for each option. Say what it costs and what it buys.

Then name the option you recommend and give the reason. The user should be able to reply with the number and move on.

Anchor every recommendation. Cite the prior art scan, an answer the user already gave, or a decision settled in an earlier branch. Say which one you used. A recommendation with no anchor is a guess.

Write the options in prose. Never use tappable-button, single-select or multiple-choice widgets. The user types a free-text answer. The real answer is often something you did not list, so invite that.

**Example:**

> **Storage:** where should the MVP keep its data?
>
> 1. SQLite. One file, no server to run. The migration path to Postgres is short.
> 2. Postgres. More setup now. It pays off if you add multiple users later.
> 3. Plain files on disk. Simplest to start. It fails as soon as you need queries.
>
> *I recommend 1.* You said this is a single-user desktop tool, so a server buys nothing today.
>
> Type your answer. Anything outside this list is fine too.

Do not give a list of options with no recommendation. The recommendation is the value.

## Conciseness

Keep responses short. State the question, the options, the recommendation and the reason.

Do not add preamble. Do not recap decisions that are already settled. Do not write explanatory essays. The user will ask when they want more.

## Tracking state

Keep a running list of settled decisions and open branches. Show it after a major branch closes, or when the user asks where things stand. Do not print it every turn.

## Finishing: the MVP plan document

The interview ends when every branch is closed and the user agrees the picture is complete. Then write the MVP plan to a **fixed path: `docs/mvp-plan.html`**, inside the project folder the app will live in. Create `docs/` if it does not exist. If a file is already there, say what would change and ask before you overwrite it.

The path is fixed on purpose. Downstream skills look for `docs/mvp-plan.html` by name, and `dev-add-feature` most of all. An MVP plan with no fixed location goes stale without anyone noticing. If the user wants it somewhere else, write it there. Then tell them plainly that downstream skills need to be pointed at it every time.

**Format.** Emit a standalone HTML document. Use the same scaffold and element conventions as `dev-create-prd`. That means a doctype, a `<head>` with the embedded `<style>` block, one `<h2>` per section, `<h3>` for subsections, real `<table>` markup, `<pre><code>` for examples, and `<ul class="checklist">` for the scope lists. Mark scope items with `data-checked="true"` for in scope and `data-checked="false"` for out of scope. This consistency lets the PRD, the phase docs and this plan be read by the same tooling.

**Sections, in order:**

1. Problem and users
2. Core loop
3. Scope. In, deferred, never. Use checklists with `data-checked`
4. Data model
5. Interfaces
6. Architecture
7. State and failure
8. Non-functional requirements
9. Milestones. Ordered, riskiest assumption first
10. Done criteria
11. Open risks and assumptions

Write it to be self-contained. Every decision in it traces to this conversation or to the scan. No decision may depend on knowledge that lives only in memory. See "Clean slate".

**Amendments.** This document is living. A later interview can change the MVP by adding a feature, moving scope, or resolving an assumption. The plan gets updated when that happens.

Keep an `<h2>Amendments</h2>` section at the end. Each entry is a `<p class="meta">` giving the date, what changed, and what changed it. The plan's history is then readable without a diff.

### Then mark the scan: the coverage pass

Run this pass only when `docs/competitive-scan.html` exists. Skip it when the user continued without a scan.

The row shape, the badge classes and the `data-coverage` and `data-phase` attributes are the contract defined in `dev-competitive-scan`. Follow that contract. Never delete a row. Never re-rank the Top 10.

Writing the MVP plan and leaving the scan unmarked produces a file that still offers already-built capabilities as fresh candidates. Close it out.

Walk **every row in all three buckets** against the finished MVP plan. Include the rows that did not make it in. Include every bucket, and not only Required.

Each row gets exactly one badge and a `.cov` note explaining the mark. Update the mirrored `data-coverage` and `data-phase` attributes with the badge:

- **`b-in`. In MVP · P#.** The plan builds it at the scope the row describes. The note names the phase and says what ships.
- **`b-part`. Partial · P#.** The plan builds a narrower version. The note must say plainly what is in and what is left out. An example is "live word count is in. Goals, streaks and session stats are left out." A partial with a vague note reads as done, which is worse than no badge.
- **`b-no`. Not in MVP.** No phase covers it. It stays backlog. `dev-add-feature` will offer it later. If a cheaper part of the plan answers part of the same need, say so in the note. That note stops the row being re-argued from scratch.
- **`b-never`. Never.** It sits on the plan's "never, not this product" list. It is off the menu permanently. Record the reason.

Then, before you report:

- **Add anything the interview surfaced that the research missed.** Add it as a new row in the bucket it belongs to. Mark it and note that it came from the interview instead of from research.
- **Flag under-met table stakes.** Any Required row that ended up `b-part` or `b-no` is a capability the market treats as non-negotiable that the MVP does not fully meet. List these to the user explicitly. Confirm each one is a deliberate choice. This is the highest-value output of the pass. Do not leave it only in the file.
- **Record the tally.** Give how many rows carry each badge, out of how many total. Add a dated footer line describing the pass. A later run then knows when the marking was last true.
- **Re-run this pass** whenever a phase is added or rescoped. Say so in the footer.

Report every file written. Report the tally. Report the under-met table stakes. When no scan exists, say the coverage pass was skipped.

Then name the next steps. Name **all four**, in this order, every time. When no scan exists, add `/dev-competitive-scan` before them, with one sentence saying the plan was written without market evidence. Do not drop the optional ones. Do not collapse the list into a single recommended command. Say the reason for each in one sentence, so the user can decide.

1. **Add Features (optional)** - `/dev-add-feature`. If the user wants a feature this interview did not discuss, run this first. It folds the feature into the plan before the architecture is settled, which is cheaper than amending later.
2. **Architecture Interview** - `/dev-architecture`. Settles the stack and technical design at `docs/architecture.html`. Required before the PRD.
3. **Design Mockups (optional, recommended)** - `/dev-ui-update`. Creating mockups and folding them in raises the chance the finished app looks the way the user expects. Without them, the build agents pick the look themselves.
4. **Setup** - `/dev-create-prd`, then `/dev-claud-md`, then `/dev-create-progress`. Writes `docs/prd.html`, the project `CLAUDE.md`, and `docs/progress.html`. After these the build pipeline can run.

Still no code.

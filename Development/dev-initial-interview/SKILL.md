---
name: dev-initial-interview
description: Run a relentless, branch-by-branch requirements interview that turns a vague app idea into a complete, agreed MVP plan written to docs/mvp-plan.html. It writes no code, ever. It includes a web-research prior art scan of comparable applications. The scan inventories their functionality and what users say about it, sorted into Required, Top 10, and Other. The scan is written to and kept current in docs/competitive-scan.html. Use this whenever the user is brainstorming, scoping, spec'ing, researching competitors for, or "thinking about building" a new application, tool, service, or major feature, even if they never say the word "interview". Also use it when a request would otherwise tempt you to start coding before scope is settled.
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

Before the scope branch, learn what comparable applications already ship and what their users complain about. This step is research. It makes the scope line an evidence-backed decision.

**Check first, then research.** Look for `docs/competitive-scan.html`.

- **It exists and is current.** Read it and reuse it. Summarize the buckets in a few lines and move on. Do not re-run the research.
- **It exists and is stale.** Staleness means the app has changed materially, the research is roughly a year old, or the market moved. Say so. Name what looks stale. Offer to refresh. Refresh only on a yes. When you refresh, extend the existing file. Keep every row and its evidence. Record in the header meta what was added and when. Never refresh silently.
- **It does not exist.** Run the scan now.

**Running the scan.** Identify five to ten genuinely comparable applications. They must do the same job. The same category is not enough.

For each one, inventory what it actually does. Then find what users say about it. Look at reviews, forums, issue trackers and support threads. Complaints about a shipped feature are worth more than the feature list. A complaint is where the opening is.

Sentiment is evidence. Attribute it to its source. Where you found no sentiment for a capability, claim none. An invented "users love this" corrupts every later decision that leans on it.

Then sort every capability found into three buckets:

- **Required.** Table stakes. These are structurally necessary for the app to be what it claims, and near-universal across the comparables. Keep them as an unranked checklist. **Only this bucket feeds the MVP scope directly.**
- **Top 10.** The genuine differentiator decisions, ranked. Rank by complaint signal first, which is loudest when the capability is missing, bad or paywalled. Rank next by prevalence across comparables. Rank last by fit to this user's problem. Capabilities users actively ask for rank above capabilities that are merely common.
- **Other.** Everything else discovered. Unranked. Kept so nothing is lost.

Top 10 and Other are **recorded**. They are not decided here. They are the reviewed backlog that `dev-add-feature` works through later, one at a time, against a working app. Do not argue them into the MVP during this interview.

Also call out any **market-wide signal** that should shape the whole design. A market-wide signal is a complaint that recurs across every comparable. Examples are learning curve, subscription cost and data lock-in. These are worth more than any single row, because answering one is itself a differentiator. Put them in a note near the top of the document.

Present the buckets briefly, with a recommendation on what belongs in the MVP and what should be parked. Then continue the interview. The scan informs branch 3, the scope line. It informs nothing after that.

### The scan document

Write to a **fixed path: `docs/competitive-scan.html`**. It is a standalone, self-styled HTML document. `dev-add-feature` reads this file and writes back to it, so the structure below is a contract.

**Header.** Give a title, a one-line description of the app, and a meta block. The meta block records the comparables surveyed by name, the date the research was prepared, and the date of any later extension. Those dates are the only staleness signal a later run has. Never omit them.

**Two standing notes** near the top. The first is a *How to read this* note. It states that only Required feeds MVP scope, and that Top 10 and Other are the reviewed backlog. The second is a *market-wide signals* note.

**One `<h2>` per bucket**, each carrying a tag chip. Rows are shaped by bucket:

```html
<!-- Required and Other: unranked rows in a .card -->
<div class="row">
  <div class="name">Capability <span class="badge b-in">In MVP · P2</span></div>
  <div class="who">Which comparables ship it, and how.</div>
  <div class="sent"><b>Sentiment:</b> what users say about it.</div>
  <div class="cov"><b>P2 Phase name.</b> How the plan covers it, and what it deliberately does not.</div>
</div>

<!-- Top 10: ranked, in an ol.ranked -->
<li>
  <div class="li-name">Capability <span class="badge b-no">Not in MVP</span></div>
  <div class="li-who">Which comparables ship it.</div>
  <div class="li-sent"><b>Sentiment:</b> …</div>
  <div class="why">Why it ranks here. Give the complaint signal and the fit.</div>
  <div class="cov">…</div>
</li>
```

**Coverage badges.** This is the machine-readable part. Every row carries exactly one badge:

| Badge | Class | Meaning |
|---|---|---|
| In MVP · P# | `b-in` | Built in the plan, at the scope the row describes |
| Partial · P# | `b-part` | In the plan, at a narrower scope than the market version |
| Not in MVP | `b-no` | In no phase. Still backlog |
| Never | `b-never` | Ruled out by the plan's "never" list. Off the backlog |

Rows carrying `b-no` are the only ones a later `dev-add-feature` run offers as fresh candidates. A `b-never` row is permanently off the menu. Getting these right is the purpose of the file.

Mirror the same fact as attributes on the row, beside each badge. Use `data-coverage="in|partial|none|never"`, and `data-phase="2"` where a phase applies. A later run can then filter without parsing badge text. A human reads the badge. Tooling reads the attributes. Keep them in sync.

Include a **legend** explaining the four badges. Include a **tally** of how many rows fall in each. Include a **footer** listing sources and the date of each marking pass.

Style it as a self-contained dark document. A person reads this file directly. It is never folded into the PRD, so it does not follow the `dev-create-prd` light scaffold.

### Updating an existing scan

The scan is a living inventory. Whenever anything changes a capability's standing, the file changes with it. Triggers include this interview, a later `dev-add-feature` run, and a rescoped phase.

**Never delete a row.** A capability that got built, deferred or ruled out keeps its sentiment notes and comparables. That evidence is why the decision was made. Deleting it makes the decision unauditable. Change the badge, add a `.cov` note, and leave everything else standing.

**Never re-rank the Top 10 to reflect what got built.** The ranking records what the market said at research time. When events overtake a row's rank, say so in its `.cov` note. Leave the row where it is.

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

These rules cover your chat messages. They also cover the prose inside `docs/mvp-plan.html` and `docs/competitive-scan.html`. The HTML format rules are separate. See "Finishing".

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

Writing the MVP plan and leaving the scan unmarked produces a file that still offers already-built capabilities as fresh candidates. Close it out.

Walk **every row in all three buckets** against the finished MVP plan. Include the rows that did not make it in. Include every bucket, and not only Required.

Each row gets exactly one badge and a `.cov` note explaining the mark:

- **`b-in`. In MVP · P#.** The plan builds it at the scope the row describes. The note names the phase and says what ships.
- **`b-part`. Partial · P#.** The plan builds a narrower version. The note must say plainly what is in and what is left out. An example is "live word count is in. Goals, streaks and session stats are left out." A partial with a vague note reads as done, which is worse than no badge.
- **`b-no`. Not in MVP.** No phase covers it. It stays backlog. `dev-add-feature` will offer it later. If a cheaper part of the plan answers part of the same need, say so in the note. That note stops the row being re-argued from scratch.
- **`b-never`. Never.** It sits on the plan's "never, not this product" list. It is off the menu permanently. Record the reason.

Then, before you report:

- **Add anything the interview surfaced that the research missed.** Add it as a new row in the bucket it belongs to. Mark it and note that it came from the interview instead of from research.
- **Flag under-met table stakes.** Any Required row that ended up `b-part` or `b-no` is a capability the market treats as non-negotiable that the MVP does not fully meet. List these to the user explicitly. Confirm each one is a deliberate choice. This is the highest-value output of the pass. Do not leave it only in the file.
- **Record the tally.** Give how many rows carry each badge, out of how many total. Add a dated footer line describing the pass. A later run then knows when the marking was last true.
- **Re-run this pass** whenever a phase is added or rescoped. Say so in the footer.

Report every file written. Report the tally. Report the under-met table stakes.

Then name the next step. Use `dev-create-prd` to turn this plan into `docs/prd.html`. Use `dev-add-feature` when a new feature needs folding in later.

Still no code.

---
name: dev-initial-interview
description: Run a relentless, branch-by-branch requirements interview that turns a vague app idea into a complete, agreed MVP plan written to docs/mvp-plan.html — no code, ever. Includes a web-research prior art scan of comparable applications that inventories their functionality and what users say about it, sorted into Required, Top 10, and Other, written to and kept current in docs/competitive-scan.html. Use this whenever the user is brainstorming, scoping, spec'ing, researching competitors for, or "thinking about building" a new application, tool, service, or major feature, even if they never say the word "interview." Also use it when a request would otherwise tempt you to start coding before scope is settled.
---

# Dev Initial Interview

Turn a rough idea into a complete, mutually understood MVP plan by interviewing the user until nothing material is unresolved.

## Hard constraint

Do not write application code. Not scaffolding, not a "quick example," not a snippet to illustrate a point. Schemas, API shapes, data models, and pseudocode-level structure are fine when they *are* the design decision under discussion — implementation is not. If the user asks for code mid-interview, note that it's outside this skill and offer to finish the plan first.

## Clean slate

Every project starts from nothing. Nothing about this app's design or function comes from persistent or global memory — not stored user preferences, not another project's decisions, not a profile of how this user usually builds things.

Concretely, none of the following may be sourced from memory:

- The stack, runtime, storage, framework, or deployment target
- The architecture, directory layout, or naming conventions
- The scope line, the core loop, the data model, or the target user
- Anything else that ends up in `docs/mvp-plan.html`

The only legitimate inputs to a design decision are: what the user says in this conversation, what the prior art scan found, and what you and the user reason out together. If a decision has no source in one of those three, it has no source.

**When memory seems relevant, ask instead of assuming.** "You used SQLite last time" is not a reason for this app to use SQLite — this app might not have a database. Raise it as a question with your recommendation, the way you would any other branch, and let the user decide on this project's merits.

**"Unless clearly stated otherwise" means stated here, now.** If the user says "same stack as my other app," that is a clear instruction — but do not fill in the specifics from memory. Ask them to name the pieces, confirm each one, and write them into the plan explicitly. The MVP plan must stand on its own: a reader with no access to any memory should be able to build from it without a missing premise.

This rule is about the *product*, not the *conversation*. How the user likes to be talked to — brevity, formatting, tone — still applies normally. What gets built does not.

## Step 0: The prior art scan

Before the scope branch, know what comparable applications already ship and what their users complain about. This is research, not opinion — it makes the scope line an evidence-backed decision instead of a guess.

**Check first, then research.** Look for `docs/competitive-scan.html`.

- **It exists and is current** — read it and reuse it. Summarize the buckets in a few lines and move on; do not re-run the research.
- **It exists and is stale** — the app has changed materially, or the research is roughly a year old, or the market moved. Say so, name what looks stale, and offer to refresh. Refresh only on a yes, and when refreshing, extend the existing file rather than starting clean: keep every row, keep its evidence, and record in the header meta what was added and when. Never refresh silently.
- **It doesn't exist** — run the scan now.

**Running the scan.** Identify five to ten genuinely comparable applications — same job, not merely the same category. For each, inventory what it actually does, and find what users say about it: reviews, forums, issue trackers, support threads. Complaints about a shipped feature are worth more than the feature list; they are where the opening is.

Sentiment is evidence, not decoration. Attribute it, and **where no sentiment was found for a capability, claim none** — an invented "users love this" corrupts every later decision that leans on it.

Then sort every capability found into three buckets:

- **Required** — table stakes. Structurally necessary for the app to be what it claims, and near-universal across the comparables. Unranked checklist. **Only this bucket feeds the MVP scope directly.**
- **Top 10** — the genuine differentiator decisions, ranked by: complaint signal first (loudest when the capability is missing, bad, or paywalled), then prevalence across comparables, then fit to this specific user's problem. Things users are actively asking for and things that would set the project apart rank above things that are merely common.
- **Other** — everything else discovered, unranked, kept so nothing is lost.

Top 10 and Other are **recorded, not decided**. They are the reviewed backlog `dev-add-feature` works through later, one at a time, against a working app. Do not argue them into the MVP during this interview.

Also call out any **market-wide signal** that should shape the whole design — a complaint that recurs across every comparable (learning curve, subscription cost, data lock-in). These are worth more than any single row, because answering one is itself a differentiator. Put them in a note near the top of the document.

Present the buckets briefly, with a recommendation on what belongs in the MVP and what should be parked. Then continue the interview; the scan informs branch 3 (scope line) and nothing after it.

### The scan document

Write to a **fixed path: `docs/competitive-scan.html`** — a standalone, self-styled HTML document. This file is a durable backlog that `dev-add-feature` reads and writes back to, so the structure below is a contract.

**Header** — title, a one-line description of the app, and a meta block recording the comparables surveyed by name, the date the research was prepared, and the date of any later extension. Those dates are the only staleness signal a later run has; never omit them.

**Two standing notes** near the top: a *How to read this* note stating that only Required feeds MVP scope while Top 10 and Other are the reviewed backlog, and a *market-wide signals* note.

**One `<h2>` per bucket**, each carrying a tag chip, and rows shaped by bucket:

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
  <div class="why">Why it ranks here — the complaint signal and the fit.</div>
  <div class="cov">…</div>
</li>
```

**Coverage badges** — the machine-readable part. Every row carries exactly one:

| Badge | Class | Meaning |
|---|---|---|
| In MVP · P# | `b-in` | Built in the plan, at the scope the row describes |
| Partial · P# | `b-part` | In the plan, but narrower than the market version |
| Not in MVP | `b-no` | Not in any phase — still backlog |
| Never | `b-never` | Ruled out by the plan's "never" list — not backlog |

Rows carrying `b-no` are the only ones a later `dev-add-feature` run offers as fresh candidates. `b-never` is permanently off the menu. Getting these right is the whole point of the file.

Alongside each badge, mirror the same fact as attributes on the row — `data-coverage="in|partial|none|never"` and `data-phase="2"` where a phase applies — so a later run can filter without parsing badge text. The badge is what a human reads; the attributes are what tooling reads. Keep them in sync.

Include a **legend** explaining the four badges and a **tally** of how many rows fall in each, plus a **footer** listing sources and the date of each marking pass.

Style it as a self-contained dark document — this file is read by a person, not folded into the PRD, so it does not follow the `dev-create-prd` light scaffold.

### Updating an existing scan

The scan is a living inventory, not a one-time research artifact. Whenever anything changes a capability's standing — this interview, a later `dev-add-feature` run, a rescoped phase — the file changes with it.

**Never delete a row.** A capability that got built, deferred, or ruled out keeps its sentiment notes and comparables; that evidence is why the decision was made, and deleting it makes the decision unauditable. Change the badge, add a `.cov` note, and leave everything else standing.

**Never re-rank the Top 10 to reflect what got built.** The ranking is a record of what the market said at research time. When a row's ranking has been overtaken by events, say so in its `.cov` note rather than moving it.

## Method

Walk the design tree. Start at the root (what the app is and who it's for), then descend branch by branch. Resolve decisions in dependency order — a decision that constrains three others gets settled before those three. Say so when you're doing this: "This one gates the data model, so let's settle it first."

Typical branch order, adapted to the domain:

1. **Problem and user** — what breaks today, who feels it, what "solved" looks like
2. **Core loop** — the one thing the user does over and over; everything else is support
3. **Scope line** — what's in the MVP, what's explicitly deferred, what's never; argued against the scan's buckets, not from scratch
4. **Data model** — entities, relationships, ownership, lifecycle
5. **Interfaces** — surfaces (web, CLI, API, desktop), inputs, outputs
6. **Architecture** — runtime, storage, external dependencies, deployment target
7. **State and failure** — persistence, concurrency, errors, recovery
8. **Non-functional** — scale, latency, security, privacy, cost ceiling
9. **Build plan** — milestones, ordering, what proves the riskiest assumption first
10. **Done criteria** — how you both know the MVP is finished

Push on vagueness. "It should be fast" is not an answer; "sub-200ms on a 10k-row table" is. When the user gives an answer that contradicts an earlier one, surface the conflict immediately rather than carrying it forward.

## Question format

Ask one question at a time, or a tight cluster when they're genuinely coupled. For every question, give your recommended answer and a one-line reason. The user should be able to reply "yes" and move on.

Ask conversationally, in prose. Never use tappable-button, single-select, or multiple-choice question widgets — the user answers in free text, and the real answer is often something you didn't list.

**Example:**

> **Storage:** SQLite or Postgres for the MVP?
> *Recommend SQLite* — single-user desktop scope, zero-ops, and the migration path to Postgres is short if it ever needs one.

Do not present a list of options in place of a recommendation. The recommendation is the value.

## Conciseness

Keep responses short. State the question, the recommendation, the reason. No preamble, no recap of what was already decided, no explanatory essays. The user will ask if they want more. Long responses slow the interview down and bury the actual question.

## Tracking state

Maintain a running list of settled decisions and open branches. Surface it only when it's useful — after a major branch closes, or when the user asks where things stand. Don't re-print it every turn.

## Finishing: the MVP plan document

The interview ends when every branch is closed and the user agrees the picture is complete. Then write the MVP plan to a **fixed path: `docs/mvp-plan.html`**, inside the project folder the app will live in. Create `docs/` if it does not exist. If a file is already there, say what would change and ask before overwriting.

The path is fixed on purpose. Downstream skills — `dev-add-feature` above all — look for `docs/mvp-plan.html` by name, and an MVP plan with no canonical location silently goes stale. If the user wants it somewhere else, write it there, then tell them plainly that downstream skills will need to be pointed at it every time.

**Format.** Emit a standalone HTML document using the same scaffold and element conventions as `dev-create-prd`: doctype, `<head>` with the embedded `<style>` block, one `<h2>` per section, `<h3>` for subsections, real `<table>` markup, `<pre><code>` for examples, and `<ul class="checklist">` with `data-checked="true"` (in scope) / `data-checked="false"` (out of scope) for the scope lists. Consistency here is what lets the PRD, the phase docs, and this plan be read by the same tooling.

**Sections, in order:**

1. Problem and users
2. Core loop
3. Scope — in, deferred, never (checklists with `data-checked`)
4. Data model
5. Interfaces
6. Architecture
7. State and failure
8. Non-functional requirements
9. Milestones — ordered, riskiest assumption first
10. Done criteria
11. Open risks and assumptions

Write it to be self-contained. Every decision in it traces to this conversation or to the scan — nothing depends on knowledge that lives only in memory (see "Clean slate").

**Amendments.** This document is living, not a snapshot. When a later interview changes the MVP — a feature added, scope moved, an assumption resolved — the plan gets updated rather than left to rot. Keep an `<h2>Amendments</h2>` section at the end, each entry a `<p class="meta">` giving the date, what changed, and what changed it, so the plan's history is readable without a diff.

### Then mark the scan: the coverage pass

Writing the MVP plan and leaving the scan unmarked produces a file that still offers already-built capabilities as fresh candidates. Close it out.

Walk **every row in all three buckets** against the finished MVP plan — not just the Required bucket, and not just the rows that made it in. Each row gets exactly one badge and a `.cov` note explaining the mark:

- **`b-in` — In MVP · P#.** The plan builds it at the scope the row describes. The note names the phase and says what ships.
- **`b-part` — Partial · P#.** The plan builds a narrower version. The note must say plainly what is in *and what is not* — "live word count is in; goals, streaks and session stats are not." A partial with a vague note is worse than no badge, because it reads as done.
- **`b-no` — Not in MVP.** No phase covers it. It stays backlog, and it is what `dev-add-feature` will offer later. If some cheaper part of the plan answers part of the same need, say so in the note — that is what keeps it from being re-litigated from scratch.
- **`b-never` — Never.** On the plan's "never — not this product" list. Off the menu permanently, with the reason recorded.

Then, before reporting:

- **Add anything the interview surfaced that the research missed** as a new row in the bucket it belongs to, marked and noted as coming from the interview rather than from research.
- **Flag under-met table stakes.** Any Required row that ended up `b-part` or `b-no` is a capability the market treats as non-negotiable that the MVP does not fully meet. List these to the user explicitly and confirm each is a deliberate choice. This is the highest-value output of the pass — do not bury it in the file.
- **Record the tally** — how many rows carry each badge, out of how many total — and a dated footer line describing the pass, so a later run knows when the marking was last true.
- **Re-run this pass** whenever a phase is added or rescoped; say so in the footer.

Report every file written, plus the tally and the under-met table stakes.

Then name the next step: `dev-create-prd` to turn this plan into `docs/prd.html`, or `dev-add-feature` when a new feature needs folding in later.

Still no code.

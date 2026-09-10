---
name: dev-competitive-scan
description: Run a web-research prior art scan of applications comparable to the one the user wants to build, and write it to docs/competitive-scan.html. The scan inventories what each comparable ships and what its users say about it, sorted into Required, Top 10, and Other. It writes no code and makes no scope decisions. It runs before dev-initial-interview, which reads the file. It also refreshes an existing scan on request. Use it whenever the user wants to research competitors, survey prior art, see what comparable apps do, find what users complain about in a category, or refresh a stale competitive scan. Also use it when dev-initial-interview or dev-add-feature reports that docs/competitive-scan.html is missing or stale.
---

# Dev Competitive Scan

Learn what comparable applications already ship and what their users complain about. Write the result to `docs/competitive-scan.html`. The scan is evidence. `dev-initial-interview` uses it to make the scope line an evidence-backed decision.

## Hard constraints

Do not write application code.

Do not decide scope. This skill records what the market does. The interview decides what the MVP does. Do not argue any row into or out of an MVP here.

## Clean slate

Nothing about this app comes from persistent or global memory. The only inputs are what the user says in this conversation and what the research finds. If memory seems relevant, ask instead of assuming.

## Step 1: Check for an existing scan

Look for `docs/competitive-scan.html` in the project folder.

- **It does not exist.** Go to Step 2.
- **It exists and is current.** Say so. Summarize the buckets in a few lines. Ask whether the user wants a refresh anyway. Stop if they say no.
- **It exists and is stale.** Staleness means the app has changed materially, the research is roughly a year old, or the market moved. Say so. Name what looks stale. Offer to refresh. Refresh only on a yes. When you refresh, extend the existing file. Keep every row and its evidence. Record in the header meta what was added and when. Never refresh silently.

## Step 2: Get the brief

You need enough to know what "comparable" means. Ask for it in one question when the user has not already said it.

Get three facts:

1. What job the app does, in one sentence.
2. Who does that job today, and with what.
3. Which existing apps the user already knows of, if any.

Do not run the full requirements interview here. That is `dev-initial-interview`. Stop asking as soon as you can name the category with confidence.

## Step 3: Run the scan

Identify five to ten genuinely comparable applications. They must do the same job. The same category is not enough.

For each one, inventory what it actually does. Then find what users say about it. Look at reviews, forums, issue trackers and support threads. Complaints about a shipped feature are worth more than the feature list. A complaint is where the opening is.

Sentiment is evidence. Attribute it to its source. Where you found no sentiment for a capability, claim none. An invented "users love this" corrupts every later decision that leans on it.

Then sort every capability found into three buckets:

- **Required.** Table stakes. These are structurally necessary for the app to be what it claims, and near-universal across the comparables. Keep them as an unranked checklist. Only this bucket feeds MVP scope directly.
- **Top 10.** The genuine differentiator decisions, ranked. Rank by complaint signal first, which is loudest when the capability is missing, bad or paywalled. Rank next by prevalence across comparables. Rank last by fit to this user's problem. Capabilities users actively ask for rank above capabilities that are merely common.
- **Other.** Everything else discovered. Unranked. Kept so nothing is lost.

Top 10 and Other are recorded. They are the reviewed backlog that `dev-add-feature` works through later, one at a time, against a working app.

Also call out any **market-wide signal** that should shape the whole design. A market-wide signal is a complaint that recurs across every comparable. Examples are learning curve, subscription cost and data lock-in. These are worth more than any single row, because answering one is itself a differentiator. Put them in a note near the top of the document.

## Step 4: Write the document

Write to a **fixed path: `docs/competitive-scan.html`**, inside the project folder the app will live in. Create `docs/` if it does not exist. It is a standalone, self-styled HTML document. `dev-initial-interview` and `dev-add-feature` read this file and write back to it, so the structure below is a contract.

**Header.** Give a title, a one-line description of the app, and a meta block. The meta block records the comparables surveyed by name, the date the research was prepared, and the date of any later extension. Those dates are the only staleness signal a later run has. Never omit them.

**Two standing notes** near the top. The first is a *How to read this* note. It states that only Required feeds MVP scope, and that Top 10 and Other are the reviewed backlog. The second is a *market-wide signals* note.

**One `<h2>` per bucket**, each carrying a tag chip. Rows are shaped by bucket:

```html
<!-- Required and Other: unranked rows in a .card -->
<div class="row" data-coverage="none">
  <div class="name">Capability <span class="badge b-no">Not in MVP</span></div>
  <div class="who">Which comparables ship it, and how.</div>
  <div class="sent"><b>Sentiment:</b> what users say about it.</div>
  <div class="cov">Unmarked. No MVP plan yet.</div>
</div>

<!-- Top 10: ranked, in an ol.ranked -->
<li data-coverage="none">
  <div class="li-name">Capability <span class="badge b-no">Not in MVP</span></div>
  <div class="li-who">Which comparables ship it.</div>
  <div class="li-sent"><b>Sentiment:</b> …</div>
  <div class="why">Why it ranks here. Give the complaint signal and the fit.</div>
  <div class="cov">Unmarked. No MVP plan yet.</div>
</li>
```

**Coverage badges.** This is the machine-readable part. Every row carries exactly one badge:

| Badge | Class | Meaning |
|---|---|---|
| In MVP · P# | `b-in` | Built in the plan, at the scope the row describes |
| Partial · P# | `b-part` | In the plan, at a narrower scope than the market version |
| Not in MVP | `b-no` | In no phase. Still backlog |
| Never | `b-never` | Ruled out by the plan's "never" list. Off the backlog |

A fresh scan has no plan to mark against. Give every row `b-no` and the `.cov` note "Unmarked. No MVP plan yet." The coverage pass at the end of `dev-initial-interview` replaces these marks. When you refresh a scan that already has marks, keep the marks on existing rows. Give only the new rows the unmarked default.

Rows carrying `b-no` are the only ones a later `dev-add-feature` run offers as fresh candidates. A `b-never` row is permanently off the menu.

Mirror the same fact as attributes on the row, beside each badge. Use `data-coverage="in|partial|none|never"`, and `data-phase="2"` where a phase applies. A later run can then filter without parsing badge text. A human reads the badge. Tooling reads the attributes. Keep them in sync.

Include a **legend** explaining the four badges. Include a **tally** of how many rows fall in each. Include a **footer** listing sources and the date of each research or marking pass.

Style it as a self-contained dark document. A person reads this file directly. It is never folded into the PRD, so it does not follow the `dev-create-prd` light scaffold.

## Updating an existing scan

The scan is a living inventory. Whenever anything changes a capability's standing, the file changes with it. Triggers include a refresh here, the coverage pass in `dev-initial-interview`, a later `dev-add-feature` run, and a rescoped phase.

**Never delete a row.** A capability that got built, deferred or ruled out keeps its sentiment notes and comparables. That evidence is why the decision was made. Deleting it makes the decision unauditable. Change the badge, add a `.cov` note, and leave everything else standing.

**Never re-rank the Top 10 to reflect what got built.** The ranking records what the market said at research time. When events overtake a row's rank, say so in its `.cov` note. Leave the row where it is.

**Update the tally and add a dated footer line** for every pass.

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
- Give only the facts the user needs to act on.
- Say what you did, whether it worked, and what the user does next.
- Keep paths and commands exact.

These rules cover your chat messages and the prose inside `docs/competitive-scan.html`.

## Question format

Ask one question at a time. Write options in prose. Never use tappable-button, single-select or multiple-choice widgets. The user types a free-text answer.

## Finishing

Report the file written. Present the buckets briefly. Name the comparables surveyed. Name the market-wide signals. Give a recommendation on which Required rows look like true table stakes and which Top 10 rows look like the strongest openings. Say that the interview makes the final call.

Then name the next step. Run `/dev-initial-interview`. It reads `docs/competitive-scan.html`, uses it in the scope branch, and marks every row against the finished MVP plan.

Still no code.

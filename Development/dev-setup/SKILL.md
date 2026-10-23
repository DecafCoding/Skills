---
name: dev-setup
description: Run the dev pipeline setup in one call: check the planning skills ran, then dev-create-prd, dev-claud-md and dev-create-progress. Use when the user says set up the build, run setup, or prep for dev-orchestrate.
---

# Dev Setup: PRD, CLAUDE.md, And Progress In One Run

## Mission

Check that the planning skills ran. Then run the three setup skills of the dev pipeline in order, in one call:

1. `dev-create-prd` writes `docs/prd.html`.
2. `dev-claud-md` writes `CLAUDE.md` at the repo root.
3. `dev-create-progress` writes `docs/progress.html`.

This skill is a thin wrapper. It owns the planning check, the order, the gates between steps, and one combined report. Each sub-skill still owns its own rules and HTML contract. Do not copy or restate their rules here. Load each one with the Skill tool and follow it as written.

This skill writes no application code. It does not run `dev-plan-phase` or `dev-execute`. It never runs the planning skills itself, because each of them is an interview with the user.

## When It Runs

It runs after the planning skills and before `dev-orchestrate`.

## Arguments

- **First argument (optional):** the PRD source, a file or a folder. Default: the `docs/` folder.
- No other arguments. Each sub-skill uses its own defaults.

## Questions

The run usually asks nothing. Ask only in these cases:

- A planning check in Step 0 fails (see the table for which ones ask).
- `docs/prd.html` already exists (Step 0).
- A sub-skill's own rules tell it to ask. Examples: the PRD sufficiency check is not sufficient, the PRD source and `docs/architecture.html` conflict, a `CLAUDE.md` value cannot be found, or regenerating `docs/progress.html` would lose `done`, `skipped` or blocker marks.

Ask one question at a time, in prose, with a recommended answer and a one-line reason. Never use multiple-choice lists or click-to-select options. Wait for the typed answer, then continue the same step. Do not restart the run.

No answer to a question is a stop. Do not ask "should I continue?" between steps. A passed gate means continue.

## Step 0: Preflight

### 0.1 Project root

Confirm the working directory is the project root. Look for `docs/` or a repo marker such as `.git`. If neither exists, stop and say so.

### 0.2 Planning check

Check that each planning skill ran. A check passes when the file exists, is not empty, and has its marker. This is a structure check only. Do not judge the content.

| Row | Skill | File | Marker | On FAIL |
|---|---|---|---|---|
| Competitive scan | `dev-competitive-scan` | `docs/competitive-scan.html` | at least one element with a `data-coverage` attribute | Ask to continue |
| MVP plan | `dev-initial-interview` | `docs/mvp-plan.html` | a `<ul class="checklist">` with `<li data-checked="...">` items | Stop |
| Scan coverage pass | `dev-initial-interview` | `docs/competitive-scan.html` | at least one row with `data-coverage` other than `none` | Ask to continue |
| Architecture | `dev-architecture` | `docs/architecture.html` | at least one `<h3 class="decision">` | Ask to continue |
| Design system | `dev-ui-update` | `docs/design-system.html` | a Handoff log section | Ask to continue |

Rules for the check:

- Skip the Scan coverage pass row when the Competitive scan row failed. Report it as `SKIPPED`.
- The MVP plan row is a stop because the PRD needs agreed scope and users. Exception: the user gave a PRD source argument that is not `docs/` or `docs/mvp-plan.html`. Then treat the row as Ask to continue.
- For the Architecture row, also list every `<h3 class="decision">` with `data-status="open"`. An open decision does not fail the row. Report it. `dev-claud-md` leaves it out of `CLAUDE.md`.
- A missing `docs/design-system.html` is a FAIL even when the project has no UI. This matches the `dev-orchestrate` readiness review.

Print the table with a PASS, FAIL or SKIPPED result on each row. Then act:

- **All rows PASS:** continue to 0.3. Ask nothing.
- **A Stop row FAILS:** stop. Name the skill to run, for example `dev-initial-interview`.
- **Only Ask rows FAIL:** ask one question that lists every failed row and the skill that writes it. Recommend running the missing skills first, with a one-line reason for each. Continue only on a clear yes.

Record the table and the answer for the final report.

### 0.3 Existing PRD and source

1. If the PRD source does not exist or has no readable documents, stop and say what is missing.
2. If `docs/prd.html` already exists, ask once: regenerate it or keep it. Recommend keep when `docs/progress.html` has any task with `data-status="done"`. If the user says keep, skip Step 1 and run the Step 1 gate on the existing file.

## Step 1: PRD

Load `dev-create-prd` with the Skill tool. Pass the PRD source as its first argument. Follow it to the end.

**Gate:** `docs/prd.html` exists, is not empty, and has at least one `<h3 class="phase" data-phase="...">`. If the gate fails, stop. Report what is wrong. Do not run Step 2.

Ignore the sub-skill's "suggest next steps" output. Keep its assumptions and gaps for the final report.

## Step 2: CLAUDE.md

Load `dev-claud-md` with the Skill tool. Use its defaults. It reads `docs/architecture.html`, the new `docs/prd.html` and the repo.

**Gate:** `CLAUDE.md` exists at the repo root, is not empty, and has a `## Build & Test` heading. No `{curly brace}` placeholder remains. If the gate fails, stop and report. Do not run Step 3.

Keep its source report (architecture doc, PRD or house default) and any open decisions it left out.

## Step 3: Progress

Load `dev-create-progress` with the Skill tool. Use its defaults.

On a new project there are no phase docs yet. This is correct. Every PRD phase is written as `data-status="not-planned"`. `dev-orchestrate` plans those phases itself.

**Gate:** `docs/progress.html` exists, is not empty, and has one `<section class="phase">` for each `data-phase` in `docs/prd.html`. If the counts do not match, stop and report both lists.

## Step 4: Combined Report

Give one short report. Do not repeat each sub-skill's full summary.

1. The Step 0 planning table, with the user's answer if one was asked.
2. A three-row PASS/FAIL table: PRD, CLAUDE.md, Progress, with the path of each file.
3. The phases found: `Phase <N>: <name>`, one line each.
4. Assumptions the PRD made where the source was silent.
5. Which source `CLAUDE.md` used for stack and pattern. Flag it if the house default was used because `docs/architecture.html` is missing.
6. Any conflicts or open decisions found in any step.
7. Next step: run `dev-orchestrate`. If `docs/architecture.html` or `docs/design-system.html` failed in Step 0, say that its readiness review will also report FAIL on that row and ask to continue.

## Rules

- Run the steps in order. Each step reads the output of the step before it.
- Never skip a gate. Never run a later step on a failed gate.
- Never edit a file another skill owns to make a check or gate pass. Report the failure.
- Never run a planning skill from this skill. Name it and let the user run it.
- Re-running this skill is safe. `dev-claud-md` updates `CLAUDE.md` in place. `dev-create-progress` keeps existing task state.

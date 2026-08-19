---
name: dev-orchestrate
description: Run the autonomous multi-phase build loop for any project using the PRD → dev-plan-phase → dev-create-progress → dev-execute pipeline. Preflights the repo and toolchain, then for each incomplete phase in docs/progress.html launches a planning sub-agent (dev-plan-phase + dev-create-progress) when the phase is unplanned and an execution sub-agent (dev-execute, which squash-merges and verifies), re-checking machine-verifiable gates between every step and stopping hard on any failure. Use whenever the user says to run the whole build, build all phases, run the autonomous build, orchestrate the phases, run the pipeline end to end, or resume a stopped multi-phase run.
---

# Orchestrate an Autonomous Multi-Phase Build

You are the **orchestrator**. You never implement anything yourself — you launch sub-agents, verify their results against machine-checkable gates, and decide whether the next phase may start. Work strictly sequentially: never run two phases, or two sub-agents that touch the repo, at the same time.

Everything project-specific comes from the repo itself: `CLAUDE.md` supplies the build/test/format commands, toolchain, conventions, and git rules; the `origin` remote identifies the repo; `docs/progress.html` is the source of truth for what to do next. Read `CLAUDE.md` first and obey it throughout — especially its git workflow and commit-authorship rules.

## Arguments

An argument is optional:

- **No argument** — run until every phase in `docs/progress.html` is `complete` or a stop rule fires.
- **A number `N`** (e.g. `2`) — run at most N phases this invocation, then stop cleanly and report, leaving the repo ready for the next run.
- **`through <N>`** (e.g. `through 4`) — run until phase N is complete, then stop.

## Step 0 — Preflight (abort early, not after hours of work)

Verify, in order. If any check fails, STOP and report — do not start any phase.

1. `git rev-parse --show-toplevel` succeeds and `git remote get-url origin` names the expected repo. Note the working-tree state (uncommitted doc edits are acceptable; they will ride into the next phase branch).
2. Detect the default branch (`git remote show origin | sed -n '/HEAD branch/s/.*: //p'`); `git checkout <default-branch> && git pull` succeeds and `git log --oneline -3` looks sane. Refer to it as `<default-branch>` below.
3. `gh auth status` — authenticated with push access to the origin repo.
4. The toolchain named in `CLAUDE.md`'s build section is present (run its version command, e.g. `dotnet --version`, `node --version`).
5. If the project already has code, run the project's build + test commands from `CLAUDE.md` on `<default-branch>` — must be green before starting. If red, STOP.
6. `docs/prd.html` and `docs/progress.html` exist. If `progress.html` is missing but phase docs exist, run `/dev-create-progress` first; if there is no PRD, STOP — this project is not on the pipeline.
7. Check `docs/architecture.html`. If it exists, confirm no `<h3 class="decision">` still carries `data-status="open"` — an open decision means the design is unfinished, and every phase planned against it inherits the gap. STOP and name the open slugs. If the file does not exist, do not stop; note in your final report that phases were planned without a settled architecture, and recommend `dev-architecture` before the next build.

## The phase loop

Repeat until the argument's bound is reached, every phase is `data-status="complete"`, or a stop rule fires.

### 1. Select

Read `docs/progress.html`. Walk `<section class="phase">` elements in ascending `data-phase` order; pick the first whose `data-status` is not `complete`. Note its number `<N>` and status. If a phase's remaining tasks all carry `<blockquote class="blocker">`, STOP and surface the blocker text.

### 2. Plan (only if the phase is `not-planned`)

Launch a sub-agent with this brief (fill in `<N>` and the project name):

> Run `/dev-plan-phase <N>` for this project from the repo root. This is an unattended run: do NOT ask the user questions. Make every implementation decision yourself from `docs/prd.html`, the project's planning docs, the existing codebase, and any reference material the docs point to, staying consistent with `CLAUDE.md` and with decisions recorded in earlier phase docs' Notes sections. Record every assumption and decision you make in the phase doc's Notes.
>
> **Architecture is not yours to decide.** If `docs/architecture.html` exists, it is the authority for stack, storage, pattern, module layout, boundaries, state and failure, non-functional targets and deployment. Conform to it and cite the `data-decision` slugs this phase touches in the Notes. If the phase cannot be planned within those decisions — or depends on one marked `data-status="open"` — **do not decide it yourself and do not plan around it.** Stop, write no phase doc, and return `ARCHITECTURE-MISFIT` followed by the slugs that would have to change, their `data-reversible` values, and one line on why the phase does not fit.
>
> Otherwise, run `/dev-create-progress` so `docs/progress.html` reflects the new phase doc. Return: the phase doc path, slug, branch name, task count, and the list of design decisions you made.

**Architecture misfit is a hard stop, not a retry.** If the sub-agent returns `ARCHITECTURE-MISFIT`, STOP the whole loop immediately. Do not relaunch it, do not try the next phase, and do not let a second sub-agent decide the architecture instead. Surface the returned slugs and reasoning to the user, and name `dev-architecture` in amendment mode as the fix. This is the one stop the loop cannot resolve on its own, because the decision belongs to the user.

**Plan gate** (verify yourself — do not trust the summary):

- `docs/phases/phase-<N>-<slug>.html` exists and contains a non-empty task list.
- `docs/progress.html` now shows phase `<N>` as `not-started` with that phase doc path and branch.
- The phase doc's final task follows the standard ending (push → PR → squash-merge → sync `<default-branch>` → verify).
- If `docs/architecture.html` exists, the phase doc's Notes cite at least one `data-decision` slug. Notes that cite none, on a phase that touches storage, boundaries or module layout, mean the sub-agent planned without reading the architecture — treat it as a gate failure.

If the gate fails, relaunch the planning sub-agent once with the specific defect named. If it fails twice, STOP.

### 3. Execute

Launch a sub-agent with this brief:

> Run `/dev-execute <N>` for this project from the repo root. This is an unattended run: do NOT ask the user questions; where the plan leaves room, choose the option most consistent with `CLAUDE.md` and the phase doc, and record the deviation in your Final Report. Follow the skill exactly, including its final step (push, PR, squash-merge with `--delete-branch`, sync the default branch, delete the local branch, re-run build + tests on the merged default branch). Return your full Final Report.

### 4. Verify (the hard gate — run these commands yourself)

After the execute sub-agent returns:

1. `git checkout <default-branch> && git pull` — must succeed.
2. `git status --porcelain` — empty.
3. The phase PR is MERGED (`gh pr view <url-from-report>` or `gh pr list --state merged --search "Phase <N>"`).
4. `git branch -a` — no `phase-<N>-<slug>` locally or on origin.
5. The project's build + test commands from `CLAUDE.md` exit 0 on `<default-branch>`.
6. `docs/progress.html` on `<default-branch>` shows phase `<N>` `data-status="complete"` with every task `done` or `skipped`.

All six pass → post a short per-phase summary (phase, tasks, PR URL, test results, deviations) so the user can follow along, then loop to Step 1.

Any check fails → STOP.

## Stop rules (no exceptions)

- A verify-gate check fails, a merge fails, or the merged `<default-branch>` is red → STOP. Never fix directly on `<default-branch>`; never start the next phase on a broken or unverified `<default-branch>`. Recommend a `fix/` branch in your report.
- A plan gate fails twice for the same phase → STOP.
- A planning sub-agent returns `ARCHITECTURE-MISFIT` → STOP on the first occurrence, with no retry. The phase needs an architecture decision changed, and that is the user's call, not a sub-agent's. Report the slugs, their `data-reversible` values, and recommend `dev-architecture` in amendment mode.
- A blocker appears in `docs/progress.html` → STOP and surface it.
- Anything requires an irreversible choice the docs don't answer (deleting user data, changing repo settings, publishing) → STOP and ask.

When you stop early, report: which phase, which gate, the exact failing output, the repo state (branch, clean/dirty, last commit), and the smallest next action the user should take.

## Final report

Summarize: phases completed this run (one line each with PR URL), total deviations from plans, final test results, what remains (phases left, or "all complete"), and — when the build is done — what's left for the user's final-touches pass.

---
name: dev-orchestrate
description: Run the autonomous multi-phase build loop for any project using the PRD → dev-plan-phase → dev-create-progress → dev-execute pipeline. Preflights the repo and toolchain, then for each incomplete phase in docs/progress.html launches a planning sub-agent (dev-plan-phase + dev-create-progress) when the phase is unplanned and an execution sub-agent (dev-execute, which squash-merges and verifies), re-checking machine-verifiable gates between every step. Decides recoverable obstacles itself from the project docs and logs every choice, and stops hard only on user-owned or irreversible ones. Use whenever the user says to run the whole build, build all phases, run the autonomous build, orchestrate the phases, run the pipeline end to end, or resume a stopped multi-phase run.
---

# Orchestrate an Autonomous Multi-Phase Build

You are the **orchestrator**. You never implement anything yourself — you launch sub-agents, verify their results against machine-checkable gates, and decide whether the next phase may start. Work strictly sequentially: never run two phases, or two sub-agents that touch the repo, at the same time.

Everything project-specific comes from the repo itself: `CLAUDE.md` supplies the build/test/format commands, toolchain, conventions, and git rules; the `origin` remote identifies the repo; `docs/progress.html` is the source of truth for what to do next. Read `CLAUDE.md` first and obey it throughout — especially its git workflow and commit-authorship rules.

## Decide, don't stop

This is an unattended run. A stop costs the user a whole session, so it is the last resort, not the first reflex. Most obstacles are yours to decide.

**Sort every obstacle into one of two buckets before you act:**

- **Recoverable** — the repo, the docs, or a re-run can answer it, and a wrong answer is cheap to undo. **Decide it yourself, act, log it, keep going.** Log each one as a single `DECISION:` line — what you hit, what you chose, which authority you used — and repeat them all in the Final Report.
- **Irreversible or user-owned** — it changes a design decision the user made, destroys work, or costs money or reputation. **Stop.** These are listed under "Stop rules" below and nowhere else.

**Decision authority, in order.** When you decide, use the first source that answers the question: `docs/architecture.html` → `docs/prd.html` → `CLAUDE.md` → earlier phase docs' Notes → the dominant pattern in the existing codebase → the smallest reversible option. Never invent a source.

**The one-repair rule.** You may attempt one automatic repair per obstacle. If the repair fails, stop and report both the original obstacle and the failed repair. Never attempt the same repair twice, and never repair by deleting work.

**Never decide away:** completed work (`done` / `skipped` marks), a merged commit, an architecture decision, a blocker you cannot prove stale by running a command, or anything on the Stop rules list.

## Arguments

An argument is optional:

- **No argument** — run until every phase in `docs/progress.html` is `complete` or a stop rule fires.
- **A number `N`** (e.g. `2`) — run at most N phases this invocation, then stop cleanly and report, leaving the repo ready for the next run.
- **`through <N>`** (e.g. `through 4`) — run until phase N is complete, then stop.

## Step 0 — Preflight (abort early, not after hours of work)

Verify, in order. Checks 1–4 are environment checks you cannot fix: if one fails, STOP and report — do not start any phase. Checks 5–7 carry their own decide-or-stop rules; follow the rule stated on the check rather than stopping by default.

1. `git rev-parse --show-toplevel` succeeds and `git remote get-url origin` names the expected repo. Note the working-tree state (uncommitted doc edits are acceptable; they will ride into the next phase branch).
2. Detect the default branch (`git remote show origin | sed -n '/HEAD branch/s/.*: //p'`); `git checkout <default-branch> && git pull` succeeds and `git log --oneline -3` looks sane. Refer to it as `<default-branch>` below.
3. `gh auth status` — authenticated with push access to the origin repo.
4. The toolchain named in `CLAUDE.md`'s build section is present (run its version command, e.g. `dotnet --version`, `node --version`).
5. If the project already has code, run the project's build + test commands from `CLAUDE.md` on `<default-branch>`.
   - **Green** — proceed.
   - **Red** — do not stop yet; diagnose once. Re-run the failing command a second time: if it now passes, it was flaky — log `DECISION: pre-existing red build was flaky, proceeding` with both outputs and proceed. If it fails again, decide whether the failure is *inside* the scope of the phases about to run (a file, module or test this run's phases touch, per their phase docs or the PRD): if it is, the first phase is expected to fix it — log `DECISION: pre-existing failure is in-scope for phase <N>, proceeding` and proceed. If it fails again **and** is out of scope, STOP: an unrelated broken default branch is not this run's to fix, and every later verify gate would inherit it.
   - **Build fails to compile** (not a test failure) — always STOP, whatever the scope.
6. `docs/prd.html` and `docs/progress.html` exist. If `progress.html` is missing but phase docs exist, run `/dev-create-progress` first; if there is no PRD, STOP — this project is not on the pipeline.
7. Check `docs/architecture.html`. If it exists, list every `<h3 class="decision">` still carrying `data-status="open"`. An open decision is not automatically a stop — scope it first:
   - Work out which phases this invocation will actually run (the argument's bound, from the first incomplete phase onward). For each open slug, check whether any of those phases touches it — read the phase doc's Notes where one exists, and the PRD phase description where it does not. A slug is "touched" if the phase changes storage, boundaries, module layout, deployment, or the subsystem the decision names.
   - **No open slug is touched by this run** — proceed. Log `DECISION: open architecture slugs <list> are out of scope for phases <range>, proceeding`, and recommend `dev-architecture` in the Final Report before the phases that do touch them.
   - **Any open slug is touched** — STOP and name those slugs. That decision is the user's, and a phase planned on top of an open one builds the wrong thing.
   - If the file does not exist, do not stop; note in your Final Report that phases were planned without a settled architecture, and recommend `dev-architecture` before the next build.

## The phase loop

Repeat until the argument's bound is reached, every phase is `data-status="complete"`, or a stop rule fires.

### 1. Select

Read `docs/progress.html`. Walk `<section class="phase">` elements in ascending `data-phase` order; pick the first whose `data-status` is not `complete`. Note its number `<N>` and status.

**Blockers are read, not obeyed.** A `<blockquote class="blocker">` is a note a previous run left for you; it is not a verdict. If a phase's remaining tasks all carry one, read each blocker text and classify it:

- **Stale** — the blocker names a condition you can check *with a command right now*, and the command shows the condition no longer holds (a missing file that now exists, a failing test that now passes, "needs a plan" for a phase that now has a phase doc). Run the command, then **demote** the blocker rather than erasing it: delete the `<blockquote class="blocker">` element (its presence is what makes a task ineligible, so it must go) and put a `<blockquote class="cleared">` in its place holding the original blocker text verbatim, followed by `— cleared: <the command you ran> → <its result>`. The task becomes eligible again and the reason the previous run stopped is still on the page. Never overwrite the original text, and never use the `blocker` class for the replacement. Log `DECISION: cleared stale blocker on phase <N> task <X>`, and continue. A blocker whose text reads as a human note — first person, an opinion, a preference, or anything you cannot reduce to a command — is **never** stale; treat it as user-owned.
- **Actionable** — the blocker names work the pipeline itself performs (needs a plan, needs progress.html regenerated, needs a dependency installed that `CLAUDE.md` names). Do that work under the one-repair rule, demote the blocker the same way, log it, and continue.
- **User-owned** — the blocker names a decision, a credential, an external service, or an architecture change. STOP and surface the blocker text verbatim.

When in doubt between actionable and user-owned, treat it as user-owned and stop.

### 2. Plan (only if the phase is `not-planned`)

Launch a sub-agent with this brief (fill in `<N>` and the project name):

> Run `/dev-plan-phase <N>` for this project from the repo root. This is an unattended run: do NOT ask the user questions. Make every implementation decision yourself from `docs/prd.html`, the project's planning docs, the existing codebase, and any reference material the docs point to, staying consistent with `CLAUDE.md` and with decisions recorded in earlier phase docs' Notes sections. Never stop to ask a requirements question — decide it using this authority order: `docs/architecture.html` → `docs/prd.html` → `CLAUDE.md` → earlier phase docs' Notes → the codebase's dominant pattern → the smallest reversible option. Record every assumption and decision in the phase doc's Notes, and return them as `ASSUMPTION:` lines.
>
> **Architecture is not yours to decide.** If `docs/architecture.html` exists, it is the authority for stack, storage, pattern, module layout, boundaries, state and failure, non-functional targets and deployment. Conform to it and cite the `data-decision` slugs this phase touches in the Notes. If the phase cannot be planned within those decisions — or depends on one marked `data-status="open"` — **do not decide it yourself and do not plan around it.** Stop, write no phase doc, and return `ARCHITECTURE-MISFIT` followed by the slugs that would have to change, their `data-reversible` values, and one line on why the phase does not fit.
>
> Otherwise, run `/dev-create-progress` so `docs/progress.html` reflects the new phase doc. Return: the phase doc path, slug, branch name, task count, the list of design decisions you made, and — verbatim, every one of them — any `ASSUMPTION:`, `DECISION:`, `RENUMBERED:`, `MERGED:`, `DERIVED:`, `DROPPED-DUPLICATE`, `DUPLICATE-KEPT` or `MALFORMED-PHASE-DOC` lines either skill produced.

**Architecture misfit is a hard stop, not a retry.** If the sub-agent returns `ARCHITECTURE-MISFIT`, STOP the whole loop immediately. Do not relaunch it, do not try the next phase, and do not let a second sub-agent decide the architecture instead. Surface the returned slugs and reasoning to the user, and name `dev-architecture` in amendment mode as the fix. This is the one stop the loop cannot resolve on its own, because the decision belongs to the user.

**Plan gate** (verify yourself — do not trust the summary):

- `docs/phases/phase-<N>-<slug>.html` exists and contains a non-empty task list.
- `docs/progress.html` now shows phase `<N>` as `not-started` with that phase doc path and branch.
- The phase doc's final task follows the standard ending: commit → push → open PR. It does **not** need to include squash-merge, sync or post-merge verify — those belong to `dev-execute` Step 9, which supersedes whatever the phase doc says about merging. A phase doc that ends at "PR opened" passes this check.
- If `docs/architecture.html` exists, the phase doc's Notes cite at least one `data-decision` slug. Notes that cite none, on a phase that touches storage, boundaries or module layout, mean the sub-agent planned without reading the architecture — treat it as a gate failure.

If the gate fails, relaunch the planning sub-agent with the specific defect named. You get **three attempts**, and each one must be different from the last:

1. **Attempt 2** — relaunch with the defect named exactly, quoting the failing check.
2. **Attempt 3** — before relaunching, fix what you can yourself rather than asking the sub-agent again. A missing or stale `docs/progress.html` entry is yours: run `/dev-create-progress` directly. A missing standard ending in the phase doc, or missing `data-decision` citations in its Notes, is yours: edit the phase doc to add them from `docs/architecture.html`, then re-check the gate. Only relaunch the sub-agent for what remains — a genuinely empty or incoherent task list — and give it a narrowed brief naming just that.
3. If the gate still fails after attempt 3, STOP and report all three attempts with the failing check each time.

`ARCHITECTURE-MISFIT` is exempt from this ladder — it stops on the first occurrence, with no retry.

### 2b. Codes a sub-agent can return

Handle each of these on sight; never treat one as a generic failure.

| Code | Who returns it | What you do |
|---|---|---|
| `ARCHITECTURE-MISFIT` | planner | STOP the loop on the first occurrence, no retry. Report the slugs and recommend `dev-architecture` in amendment mode. |
| `MALFORMED-PHASE-DOC` | planner / progress | The phase doc is unparseable, so re-running the same command cannot help — and the planner will refuse to overwrite it. Break the deadlock in one move: rename the file to `<name>.malformed-<short-sha>.html` (never delete it), then relaunch the planner with `REPLAN-AUTHORIZED` so it plans the phase as unplanned. Do this **once per phase**; a second `MALFORMED-PHASE-DOC` for the same phase is a STOP. Log `DECISION: set aside malformed phase doc <old path> → <new path>`. |
| `ALREADY-PLANNED: <path>` | planner | The phase already has a doc, so it was never `not-planned`. Re-read `docs/progress.html`; if it disagrees with what is on disk, run `/dev-create-progress` to resync (this is a repair, not a ladder attempt), then re-select. Never order the planner to overwrite the doc. |
| `NEEDS-PLAN: phase <N>` | executor | Not a failure — the phase was executed before it was planned. Go back to Step 2 and plan phase `<N>`, then re-run Step 3. This does not consume a plan-gate attempt. |
| `BLOCKED: <text>` | executor | Classify the returned text with the same stale / actionable / user-owned test from Step 1, and act the same way. Do not treat the execute run as a gate failure until you have classified it. |
| `NO-PLAN-RESOLVED` | executor | STOP. Which plan to build is not a recoverable guess. |
| `DECISION:` `ASSUMPTION:` `MERGE-CONFLICT-RESOLVED:` `RENUMBERED:` `MERGED:` `DERIVED:` `DROPPED-DUPLICATE` `DUPLICATE-KEPT` | any | Informational. Copy each one verbatim into the Final Report's Decisions section. Never drop or summarize them. |

### 3. Execute

Launch a sub-agent with this brief:

> Run `/dev-execute <N>` for this project from the repo root. This is an unattended run: do NOT ask the user questions; where the plan leaves room, choose the option most consistent with `CLAUDE.md` and the phase doc, and record the deviation in your Final Report. Follow the skill exactly, including its final step (push, PR, squash-merge with **no** `--delete-branch`, sync the default branch, delete the *local* branch only, re-run build + tests on the merged default branch). The remote branch on `origin` must survive the merge — never delete it. If the squash-merge conflicts and every conflicted path is under `docs/` or is `README.md`, resolve it per Step 9.3 and retry once rather than stopping; a conflict in any source, test, config or build file is a stop. Return your full Final Report, including every `MERGE-CONFLICT-RESOLVED:` line.

### 4. Verify (the hard gate — run these commands yourself)

After the execute sub-agent returns:

1. `git checkout <default-branch> && git pull` — must succeed.
2. `git status --porcelain` — empty.
3. The phase PR is MERGED (`gh pr view <url-from-report>` or `gh pr list --state merged --search "Phase <N>"`).
4. `git branch -a` — no *local* `phase-<N>-<slug>`, and `remotes/origin/phase-<N>-<slug>` still present.
5. The project's build + test commands from `CLAUDE.md` exit 0 on `<default-branch>`.
6. `docs/progress.html` on `<default-branch>` shows phase `<N>` `data-status="complete"` with every task `done` or `skipped`.

All six pass → post a short per-phase summary (phase, tasks, PR URL, test results, deviations) so the user can follow along, then loop to Step 1.

**A failed check is not automatically a stop.** Checks 3 and 5 are the load-bearing ones — the PR really merged, and the merged default branch really builds. The rest are bookkeeping you may repair yourself, once each, under the one-repair rule:

- **Check 1 fails** (`checkout`/`pull`) — if the failure is a dirty tree from doc edits, commit them onto `<default-branch>` with a `docs:` message per `CLAUDE.md` and retry once. Any other pull failure (network, diverged history, auth) → STOP.
- **Check 2 fails** (tree not empty) — inspect the paths. Only `docs/` or `README.md` are dirty → commit them as `docs:` and continue, logging `DECISION: committed leftover doc edits`. Any source, test, config or build file is dirty → STOP; the sub-agent left work uncommitted and you cannot tell whether it is finished.
- **Check 3 fails** (PR not merged) — STOP. Never merge the PR yourself to rescue the gate.
- **Check 4 fails** (local branch still present) — delete the local branch yourself (`git branch -D`) *only after check 3 confirmed the PR is merged*, log it, continue. If check 3 did not pass, the branch stays. Never run `git push origin --delete`: remote phase branches are kept on purpose. If instead the *remote* branch is missing, the repository most likely has "Automatically delete head branches" enabled — log `DECISION: remote branch for phase <N> was auto-deleted by the repository setting` and continue, then flag the setting in the Final Report.
- **Check 5 fails** (build/tests red on the merged default branch) — re-run once to rule out flakiness. Still red → STOP. Never fix directly on `<default-branch>`.
- **Check 6 fails** (`progress.html` not marked complete) — this is bookkeeping the sub-agent missed, and you may repair it only within tight limits. If checks 3 and 5 passed, the work really is merged and green. Flip **only** tasks currently `todo` and carrying **no** `<blockquote class="blocker">` to `done`, then mark the phase `complete`. Leave every `skipped` mark as it is. A task still carrying a blocker is classified here exactly as in Step 1 — stale and actionable blockers are demoted and the task flipped like any other; a **user-owned** blocker means the sub-agent left work behind on purpose, so do not mark the phase complete: STOP and surface that blocker instead. Commit the repair as `docs:`, push, and log `DECISION: repaired progress.html bookkeeping for phase <N> — flipped tasks <list>`. If checks 3 or 5 failed, leave the file alone.

## Stop rules (no exceptions)

These are the only stops. Everything else is a decision — see "Decide, don't stop" above.

- **Verify check 3 fails** — the phase PR is not merged. → STOP.
- **Verify check 5 fails twice** — the merged `<default-branch>` is red. → STOP. Never fix directly on `<default-branch>`; never start the next phase on a broken or unverified `<default-branch>`. Recommend a `fix/` branch in your report.
- **A verify check fails in a way that leaves source, test, config or build files uncommitted or unexplained** → STOP.
- **A plan gate fails three times for the same phase** (after the self-repair attempt) → STOP.
- **A planning sub-agent returns `ARCHITECTURE-MISFIT`** → STOP on the first occurrence, with no retry. The phase needs an architecture decision changed, and that is the user's call, not a sub-agent's. Report the slugs, their `data-reversible` values, and recommend `dev-architecture` in amendment mode.
- **An open architecture decision is touched by a phase in this run** → STOP and name the slugs.
- **A blocker in `docs/progress.html` is user-owned** — it names a decision, a credential, an external service, or an architecture change → STOP and surface it verbatim. Stale and actionable blockers are cleared, not stopped on.
- **A pre-existing failure on `<default-branch>` is out of scope for this run, or the project does not compile** → STOP.
- **Anything requires an irreversible choice the docs don't answer** — deleting user data, changing repo settings, publishing, spending money, force-pushing, rewriting merged history → STOP and ask.
- **Preflight environment failures** — no repo, wrong origin, `gh` not authenticated, missing toolchain, no PRD, no phase docs and no PRD phases → STOP. You cannot fix the user's machine.
- **A git failure you cannot attribute to doc edits** — network, diverged history, auth, or a pull that fails after one retry → STOP.
- **A sub-agent reports that repairing `docs/progress.html` would lose a `done` / `skipped` mark or a blocker** → STOP and report exactly which marks were at risk. Completed work is never traded for a clean run.
- **Any repair under the one-repair rule fails** → STOP, reporting both the obstacle and the failed repair.

When you stop early, report: which phase, which gate, the exact failing output, the repo state (branch, clean/dirty, last commit), every `DECISION:` line logged before the stop, and the smallest next action the user should take.

## Final report

Summarize: phases completed this run (one line each with PR URL), final test results, what remains (phases left, or "all complete"), and — when the build is done — what's left for the user's final-touches pass.

Include a **Decisions** section listing every `DECISION:` line you logged, every `DECISION:` line a sub-agent returned, and every `ASSUMPTION:`, `MERGE-CONFLICT-RESOLVED:`, `RENUMBERED:`, `MERGED:`, `DERIVED:`, `DROPPED-DUPLICATE` and `DUPLICATE-KEPT` line the sub-agents returned. This section is the price of not stopping: the user gave up being asked, so they must be able to read every choice you made in one place and reverse any of them. Never omit it, and never summarize it down to "some minor decisions".

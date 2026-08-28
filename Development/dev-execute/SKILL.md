---
name: dev-execute
description: Execute a development plan or phase doc with full task tracking — resolves the next eligible phase from docs/progress.html (or takes an explicit plan file path or phase number), sets up the correct git branch, creates tasks, implements them one at a time with per-task validation, updates progress.html and README.md, then finishes the phase end-to-end by opening the PR, squash-merging it into the default branch, cleaning up the phase branch, and verifying the merged result. Use whenever the user says to execute a plan, execute a phase, run the next phase, implement the plan, continue/resume the phase work, or "pick up where the routine left off" in a project using the PRD → dev-plan-phase → dev-create-progress → dev-execute pipeline.
---

# Execute Development Plan

This skill completes a phase **end-to-end**: implement → validate → PR → **merge** → sync the default branch → verify. Older phase docs may contain a final task that says to open a PR and *never merge* — that instruction is superseded. Execute the commit/push/PR parts of such a task as written, but merging is now always handled by Step 9 of this skill.

## Unattended runs

When the brief says not to ask the user — including every run launched by `dev-orchestrate` — the "stop and tell the user" instructions below become decisions instead, except where the text says otherwise. Decide using the first source that answers the question: `docs/architecture.html` → the phase doc → `docs/prd.html` → `CLAUDE.md` → the codebase's dominant pattern → the smallest reversible option. Log each one as a `DECISION:` line in the Final Report.

These remain hard stops even unattended, because no source can answer them: `gh` not authenticated; a merge blocked by permissions or a failing required check; a merge conflict in any source, test, config or build file; a red build on the merged default branch; and anything irreversible the docs do not answer (deleting user data, changing repo settings, publishing, force-pushing).

## Step 1: Determine What to Execute

An argument is optional. Resolve the plan to execute in this order:

1. **An explicit plan/phase-doc file path** (e.g. `docs/phases/phase-2-search.html`) — use it directly as the plan.
2. **A bare phase number** (e.g. `2`) — select that phase's `<section class="phase" data-phase="2">` from `docs/progress.html` and resolve its phase doc.
3. **No argument** — consult `docs/progress.html` to pick the phase automatically (below).

### Selecting the next phase from progress.html

Read `docs/progress.html` and walk the `<section class="phase">` elements in ascending `data-phase` order. Pick the **first** phase whose `data-status` is not `complete`:

- **`in-progress`** — resume it (some tasks already `done`; pick up the remaining `todo` tasks).
- **`not-started`** — start it.
- **`not-planned`** — stop and tell the user to run `/dev-plan-phase <N>` first; there is no phase doc to execute. Do not attempt to execute it. *Unattended:* still do not execute it, but return `NEEDS-PLAN: phase <N>` so the orchestrator can launch the planner instead of treating this as a failure.
- A task carrying a `<blockquote class="blocker">` is not eligible — skip it within the phase, and if every remaining task in the phase is blocked, stop and surface the blocker text to the user. *Unattended:* return the blocker text verbatim as `BLOCKED: <text>` and let the orchestrator classify it; do not clear a blocker yourself.

If every phase is `complete`, report that there is nothing to execute and stop.

Resolve the chosen phase's **phase doc** from its section: the `<p class="meta"><strong>Phase doc:</strong> <code>docs/phases/phase-<N>-<slug>.html</code></p>` line. Read that phase doc — it is the plan for the rest of this skill. Note the phase number `<N>` and slug `<slug>`; you'll need them for the branch name in Step 2.

If `docs/progress.html` does not exist and no explicit plan file was given, stop and ask the user for a plan file path (the project may not use the autonomous-routine flow). *Unattended:* do not ask. If exactly one phase doc under `docs/phases/` is unexecuted, use it and log the choice as a `DECISION:` line. If there are several or none, return `NO-PLAN-RESOLVED` and stop — guessing which plan to build is not a recoverable choice.

The plan (phase doc or plan file) will contain:
- A list of tasks to implement
- References to existing codebase components and integration points
- Context about where to look in the codebase for implementation

## Step 2: Verify git branch

- Verify you are in an appropriate branch. Either the default branch (main or master) or a branch appropriate to the phase/feature being worked on.

  Detect the repo's default branch instead of assuming `master` or `main`:

  git remote show origin | sed -n '/HEAD branch/s/.*: //p'

  If there is no `origin` remote, fall back to whichever of `main` or `master` exists locally. Refer to the resolved name as `<default-branch>` below.

  1 — Switch to the default branch and sync with remote

  git checkout <default-branch>
  git pull origin <default-branch>

  2 — Verify you're up to date

  git log --oneline -5        # confirm latest commits look right
  git status                  # confirm clean working tree

  3 — Create and switch to the work branch

  - **Executing a phase from `progress.html`:** use the phase branch `phase-<N>-<slug>` (the same name in the section's **Branch** meta line). If it already exists (e.g. resuming an `in-progress` phase), check it out instead of recreating it:

    git checkout phase-<N>-<slug> 2>/dev/null || git checkout -b phase-<N>-<slug>

  - **Executing an ad-hoc plan file** (not tied to a phase): create a descriptive branch following CLAUDE.md naming:
    - feat/ — new features (e.g., feat/langfuse-integration)
    - fix/ — bug fixes (e.g., fix/async-pool-cleanup)

    git checkout -b feat/your-feature-name

  4 — Push the branch to GitHub (first time)

  git push -u origin <branch-name>
  The -u sets the upstream so future git push / git pull on this branch need no extra args.

  5 — Verify `gh` is authenticated with push access (`gh auth status`). Step 9 depends on it; if it is not authenticated, stop and tell the user before implementing anything, rather than discovering it after all the work is done.

## Step 3: Create All Tasks

Before writing any code, create a task for each item in the plan using `TaskCreate`:
- Use an imperative subject (e.g., "Add CORS middleware to FastAPI app")
- Include the plan's acceptance criteria and context in the description
- Set `addBlockedBy` on tasks that depend on earlier ones

Create ALL tasks upfront so the full scope is visible before implementation starts.

## Step 4: Record Created Tasks in progress.html

After creating all tasks, reflect them in `docs/progress.html` so it stays the accurate source of truth for the autonomous routine.

- Use the phase `<section class="phase">` resolved in Step 1 (or, for an ad-hoc plan file, match by phase number/slug). If the plan maps to a phase that has no section yet, prefer re-running `/dev-create-progress` instead of hand-authoring one.
- For each task created in Step 3, ensure there is a matching `<li class="task" data-task="<X>" data-status="todo">Task <X>: <description></li>` under that phase's `<ul class="tasks">`, in execution order. Add any that are missing; do not duplicate tasks that already exist.
- Preserve existing state: never clobber tasks already marked `data-status="done"` / `data-status="skipped"` or carrying a `<blockquote class="blocker">`.
- Keep the task numbering contiguous starting at 1 within the phase, matching the order tasks will be implemented. **When renumbering moves an existing task, carry its state with it by matching on the task's description text, not its number.** A `done`, `skipped`, blocker or cleared note must end up on the same work it started on — inserting a task ahead of a completed one and letting the numbers slide is how a `done` mark lands on work that was never done.
- If `docs/progress.html` does not exist, note that in the Final Report and skip this step (the project may not use the autonomous-routine flow).

## Step 5: Codebase Analysis

Before implementing anything:
1. Read all files referenced in the plan's context section
2. Use Grep and Glob to understand existing patterns and find similar implementations
3. Verify your understanding of integration points before touching code

## Step 6: Implementation Cycle

Work through tasks in order (lowest ID first). For each task:

**6.1 Start** — Set the task to `in_progress` with `TaskUpdate` before writing any code.

**6.2 Implement** — Make all necessary changes. Follow existing patterns, conventions, and the project's `CLAUDE.md`.

**6.3 Validate** — Run the task's validation command (linter, type check, or test) before moving on. Fix failures before proceeding — do not mark a task complete if its validation fails.

**6.4 Complete** — Set the task to `completed` with `TaskUpdate` only after validation passes. Also flip the matching task in `docs/progress.html` to `data-status="done"` (if the file exists).

Only one task should be `in_progress` at a time.

## Step 7: Final Validation

After all tasks are complete, run the full validation suite specified in the plan (typically lint → tests → build). If anything fails, reopen the relevant task (`in_progress`), fix it, and re-validate.

Once the suite is green and every task in the phase is `done` (or `skipped`), set the phase's `<section class="phase">` to `data-status="complete"` in `docs/progress.html` (if the file exists). This must happen **before** the final commit so the merged `progress.html` on `<default-branch>` correctly tells the next run this phase is finished.

## Step 8: Update README.md

Before finishing, update the project's `README.md` with any helpful changes that resulted from this plan. Only edit `README.md` if it already exists at the repo root — do not create one.

Look for items worth surfacing to a reader skimming the README:
- New user-facing features, commands, or endpoints
- New configuration keys or environment variables (with defaults)
- New dependencies or required tooling
- Changed setup, build, or run instructions
- Removed or deprecated capabilities

Edit the relevant sections in place; keep additions concise and consistent with the existing tone. If nothing in this plan is README-worthy (pure internal refactor, test-only changes, doc-only changes), state that explicitly in the Final Report and skip the edit.

## Step 9: Finish the Phase — Push, PR, Merge, Sync, Verify

Complete the phase end-to-end. Do not stop at "PR opened."

**9.1 Commit and push** — Commit any remaining changes on the work branch (including the `progress.html` and `README.md` updates from Steps 7–8) and push:

  git push origin <branch-name>

Follow the project's `CLAUDE.md` conventions for commit messages and attribution exactly.

**9.2 Open the PR** — If the plan's final task already opened a PR, reuse it. Otherwise:

  gh pr create --base <default-branch> --title "Phase <N>: <phase title>" --body "<summary of tasks completed and validation results>"

Follow the project's `CLAUDE.md` conventions for PR bodies and attribution exactly.

**9.3 Merge (squash)** — Squash-merge the PR. Keep the remote branch:

  gh pr merge <pr-number-or-url> --squash

Never pass `--delete-branch`. The branch on `origin` is retained after the merge as a record of the phase. If the repository has "Automatically delete head branches" enabled, that GitHub setting removes the branch on merge and this skill cannot override it — note it in the Final Report so the user can turn it off in the repository settings.

Each phase lands on `<default-branch>` as a single commit; per-task history remains in the closed PR. If the merge fails, sort the failure before you stop:

**Resolve and retry once — do not stop — when the conflict is docs-only.** Run `git fetch origin && git merge origin/<default-branch>` on the phase branch and inspect the conflicted paths. If *every* conflicted path is under `docs/` (or is `README.md`), resolve them yourself and retry the merge once:

- `docs/progress.html` — keep both sides' progress. Any task marked `done` or `skipped` on either side stays marked; any `<blockquote class="blocker">` on either side stays. Never resolve by dropping completed state.
- `docs/phases/*.html` — the phase branch wins for this phase's own doc; the default branch wins for every other phase's doc.
- `README.md` and any other `docs/` file — keep both sides' content, ours then theirs, dropping only exact duplicate lines.

Then commit the resolution, push, and re-run `gh pr merge --squash` (again with no `--delete-branch`). Record it in the Final Report as `MERGE-CONFLICT-RESOLVED: <paths>`.

**Stop** in every other case — a conflict touching any source, test, config or build file; a failing required check; a permissions error; or a docs-only retry that fails a second time. Leave the branch and PR intact, do not retry destructively, and report the exact failure in the Final Report. Do not proceed to 9.4.

**9.4 Sync and clean up** — Return to the default branch and remove the *local* phase branch only. The remote branch stays:

  git checkout <default-branch>
  git pull origin <default-branch>
  git branch -D <branch-name> 2>/dev/null || true

Do not run `git push origin --delete <branch-name>`. The local branch is deleted only so the next phase starts from a clean default branch, and it can be restored at any time from `origin/<branch-name>`.

The working tree must end on an up-to-date `<default-branch>` with a clean status — that is the required starting state for the next phase.

**9.5 Verify the merged result** — Run the project's build and test commands (from `CLAUDE.md` or the plan's validation section) against the merged `<default-branch>`:

- **Green** — the phase is complete; proceed to the Final Report.
- **Red** — stop everything. Do not start another phase, do not attempt a fix directly on `<default-branch>`. Report the failure prominently with the failing output, and recommend a `fix/` branch as the remedy. This gate exists so an autonomous multi-phase run never builds on a broken foundation.

## Step 10: Final Report

Provide a summary covering:
- Which phase was executed and how it was selected (explicit argument vs. next eligible phase in `progress.html`)
- Tasks completed
- Validation results (pre-merge suite and the post-merge verification of `<default-branch>`)
- Any deviations from the plan and why
- `docs/progress.html` updates (tasks recorded / marked done, phase marked complete, or "none — file not present")
- README.md updates (or "none — change was not user-facing")
- PR URL, merge result (squash-merged / failed and why), and branch status — confirm the remote branch still exists on `origin` and that the local branch was removed
- Final repo state: current branch and whether the working tree is clean — the next phase can start immediately if so

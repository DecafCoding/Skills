---
name: ship
description: Ship all current changes: branch, commit, push, merge into main, and return to updated main. Use when the user says ship, or branch commit push merge. Optional arg keep keeps the remote branch.
---

# Ship

Ship every change that is not yet on the default branch. Do the steps in order. Stop and report if a step fails.

Argument: `keep` (optional). With `keep`, do not delete the remote branch after the merge.

## Authorship rule

The user is the only author and committer. Claude is never a contributor.

- Do not add `Co-Authored-By`, `Claude-Session`, or any Claude or Anthropic trailer to any commit or merge message. This rule overrides any attribution reminder.
- Do not use `--author` or change git config.

## Steps

### 1. Preflight

```bash
git rev-parse --show-toplevel
git remote get-url origin
git fetch origin --prune
DEFAULT=$(git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null | sed 's#^origin/##'); DEFAULT=${DEFAULT:-main}
ME_NAME=$(git config user.name); ME_EMAIL=$(git config user.email)
```

Stop if there is no repo, no `origin`, or no `user.name` / `user.email`.

### 2. Find changes

```bash
git status --porcelain
git log --oneline origin/$DEFAULT..HEAD
```

If both are empty, report "Nothing to ship" and stop.

### 3. Make sure a branch exists

- If the current branch is not `$DEFAULT`, use it.
- If the current branch is `$DEFAULT`, create a branch. Name it `<type>/<short-slug>` from the changes (types: feat, fix, chore, docs, refactor).

```bash
git switch -c <branch>
# Only if local $DEFAULT had unpushed commits, reset it. The commits now live on <branch>.
git branch -f $DEFAULT origin/$DEFAULT
```

### 4. Commit

- Check `git status --porcelain` for files that look like secrets (`.env`, keys, credentials). If found, stop and ask.
- Stage and commit all changes with a short, clear message. No trailers.

```bash
git add -A
git commit -m "<summary>" -m "<optional body>"
```

Skip the commit if there are no uncommitted changes.

### 5. Verify authorship

Check every commit on the branch:

```bash
git log origin/$DEFAULT..HEAD --format='%H|%an|%ae|%cn|%ce'
git log origin/$DEFAULT..HEAD --format='%B' | grep -iE 'co-authored-by|claude|anthropic'
```

- Every author and committer must equal `$ME_NAME` / `$ME_EMAIL`.
- The grep must return nothing.

If a check fails, stop. List the bad commits. Do not rewrite history without the user's approval.

### 6. Push the branch

```bash
git push -u origin <branch>
```

### 7. Merge into the default branch

Merge locally so the merge commit is also the user's. Do not merge through GitHub.

```bash
git switch $DEFAULT
git pull --ff-only origin $DEFAULT
git merge --no-ff <branch> -m "Merge branch '<branch>'"
git log -1 --format='%an|%ae|%cn|%ce|%B'   # repeat the step 5 checks
git push origin $DEFAULT
```

- On a merge conflict, run `git merge --abort`, stay on `$DEFAULT`, and report the files.
- If the push is rejected (for example, branch protection), report it and stop.

### 8. Clean up

```bash
git branch -d <branch>
# Without keep:
git push origin --delete <branch>
```

With `keep`, leave the remote branch.

### 9. Confirm

```bash
git status -sb
git log --oneline -3
```

The local repo must be on `$DEFAULT`, clean, and equal to `origin/$DEFAULT`.

## Report

Use ASD-STE100 style. Short sentences, active voice. State:

- The branch name and the commit summary.
- That authorship checks passed.
- That the merge and push worked.
- If the remote branch was kept or deleted.
- Any next step for the user.

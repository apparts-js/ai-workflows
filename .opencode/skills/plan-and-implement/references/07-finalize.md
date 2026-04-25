# Finalize

## Goal
Merge base one final time, update the PR body with a reviewer summary, and stop. The workflow will add the `auto-review` label.

## Prerequisites
- All subtasks are checked.
- Self-check is clean (no `must-fix` items).

## Steps

1. Find the open PR and merge the latest base branch one final time:
   ```bash
   PR_NUMBER=$(gh pr list --state open --json number,headRefName -q ".[] | select(.headRefName | startswith(\"${ARGUMENTS}-\")) | .number")
   BASE=$(gh pr view "$PR_NUMBER" --json baseRefName -q .baseRefName)
   git fetch origin
   git checkout $(gh pr view "$PR_NUMBER" --json headRefName -q .headRefName)
   git pull
   git merge origin/$BASE || {
     echo "Merge conflicts detected. Stopping."
     exit 1
   }
   ```
   If merge conflicts occur, invoke `resolve-pr-conflicts` and stop.

2. Update the PR body with a reviewer summary:
   ```bash
   gh pr edit "$PR_NUMBER" --body "..."
   ```

3. **Leave the PR as draft.** The workflow will add the `auto-review` label; the `review-pr` skill will promote it to ready only if the automatic audits pass cleanly.

4. **Stop.**

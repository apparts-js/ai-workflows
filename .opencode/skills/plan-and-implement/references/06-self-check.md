# Self-check before finalizing

## Goal
Run the same audits that `review-pr` runs and fix any `must-fix` findings before finalizing.

## Prerequisites
- All implementation subtasks are checked.

## Steps

1. Find the open PR and merge the latest base branch:
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

2. Determine the diff range:
   ```bash
   BASE=$(git merge-base origin/master HEAD)
   RANGE="${BASE}..HEAD"
   ```

3. Run the three audit skills and capture their outputs:
   ```
   Skill("code-review", args=RANGE)
   Skill("verify-tests", args=RANGE)
   Skill("code-guidelines-check", args=RANGE)
   ```

4. Parse the **Actionable findings** sections for `**severity:** must-fix`.

5. For each `must-fix` finding:
   - Apply the provided fix using `Write` or `Edit`.
    - Format and commit:
      ```bash
      git add <specific-files>
      npx prettier --write $(git diff --cached --name-only) 2>/dev/null || true
      git add $(git diff --cached --name-only) 2>/dev/null || true
      git commit -m "fix: resolve self-check finding – <description> (#${ARGUMENTS})"
      git push
      ```

6. Re-run the three audits. Repeat steps 4–6 until no `must-fix` items remain.

7. If `should-fix` or `note` items remain that you cannot address quickly, leave them for the human reviewer; do not block finalization on them.

8. Check off "Fix issues found in audit" in the subtasks comment.

9. Load `references/07-finalize.md` and continue in this session.

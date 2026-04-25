# Implement remaining work

## Goal
Complete the remaining unchecked subtasks: "Implement logic to pass tests" and "Update docs / README if needed".

## Prerequisites
- "Write stubs and failing tests" is already checked.

## Steps

1. Find the open PR and merge the latest base branch before working:
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

2. For each unchecked subtask up to "Fix issues found in audit":
   a. Do the work (implement logic, refactor, write docs).
   b. Commit and push:
      ```bash
      git add <specific-files>
      git commit -m "feat: <description> (#${ARGUMENTS})"
      git push
      ```
   c. Check off the subtask in the subtasks comment:
      - Find the subtasks comment:
        ```bash
        gh issue view "$ARGUMENTS" --json comments -q '.comments[] | select(.body | contains("## Subtasks")) | {id,body}'
        ```
      - Replace `- [ ] <exact text>` with `- [x] <exact text>`.
      - Update the comment:
        ```bash
        gh api "repos/${REPO}/issues/comments/${COMMENT_ID}" -X PATCH -f body="${UPDATED_COMMENT_BODY}"
        ```

3. Load `references/06-self-check.md` and continue in this session.

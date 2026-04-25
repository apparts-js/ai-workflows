---
name: plan-and-implement
description: Plan, subdivide, and implement a GitHub issue end-to-end. Triggered when an issue is labeled 'opencode'. Creates subtasks, opens a branch and draft PR, merges base regularly, handles merge conflicts, implements with TDD, pushes regularly, self-checks with the same audits used in code review, and marks the PR complete when finished.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
context: fork
agent: general-purpose
argument-hint: <issue-number>
---

You are invoked to drive an issue labeled `opencode` to completion.

## Inputs
- `$ARGUMENTS` contains the issue number.

## Setup

1. Determine repo slug:
   ```bash
   REPO=$(git remote get-url origin | sed -E 's/.*github\.com[/:]([^/]+)\/([^/]+?)(\.git)?$/\1\/\2/')
   OWNER=$(echo "$REPO" | cut -d/ -f1)
   ```
2. Fetch the issue body and all comments:
   ```bash
   ISSUE_BODY=$(gh issue view "$ARGUMENTS" --json body -q '.body')
   ISSUE_COMMENTS=$(gh issue view "$ARGUMENTS" --json comments -q '.comments[].body')
   ```

## State detection

Check these conditions in order. The first matching condition determines the state.

1. **No subtasks comment**: Neither `$ISSUE_BODY` nor any comment in `$ISSUE_COMMENTS` contains `## Subtasks`.
   → Load `references/01-plan.md`

2. **No PR exists**: No open PR has a head ref starting with `${ARGUMENTS}-`.
   → Load `references/02-create-pr.md`

3. **"Open draft PR" unchecked**: The subtasks comment contains `- [ ] Open draft PR`.
   → Load `references/02-create-pr.md`

4. **"Write stubs and failing tests" unchecked**: The subtasks comment contains `- [ ] Write stubs and failing tests`.
   → Load `references/02-create-pr.md`

5. **"Implement logic to pass tests" unchecked**: The subtasks comment contains `- [ ] Implement logic to pass tests`.
   → Load `references/05-implement.md`

6. **"Update docs / README if needed" unchecked**: The subtasks comment contains `- [ ] Update docs / README if needed`.
   → Load `references/05-implement.md`

7. **"Fix issues found in audit" unchecked**: The subtasks comment contains `- [ ] Fix issues found in audit`.
   → Load `references/06-self-check.md`

8. **All subtasks checked**: None of the above are unchecked.
   → Load `references/07-finalize.md`

## Dispatch

Load the identified reference file and follow its instructions **from top to bottom**.

- Some reference files end with **STOP** — do not proceed further in this session.
- Some reference files end with "Load `references/XX-...md`" — load that file next and continue in the same session.
- Do not continue beyond what the reference file instructs.

## Principles
- Push every commit. Each push is a checkpoint.
- Never check off a subtask if tests are failing.
- If interrupted, re-running this skill on the same issue will resume from the first unchecked subtask.
- Always merge the base branch before starting work to minimize conflicts.
- **Do not run tests locally.** The target repository's CI workflows are the source of truth for test results. Push changes and let CI verify them.

You are implementing GitHub issue #{number}: {title}.
This is a child of PRD #{prd_number}.

Branch off {base_branch}: implementer/issue-{number}-{slug}
Merge target: {base_branch} (origin/{base_branch})
Do NOT create a pull request. Merge directly to {base_branch} by pushing with the refspec below.

## Step 1 — Gather the Contract

- Fetch the issue from the issue tracker: `gh issue view {number} --comments`
- Read both the issue body and the "Agent Brief" in the comments.
- The Agent Brief is your absolute contract. Pay special attention to the "Out of scope" section — do absolutely NO gold-plating outside of these boundaries.

## Step 2 — Test-Driven Implementation

Use the `/idd:tdd` skill to drive the implementation. Test behaviors at the boundaries defined in the Agent Brief, not internal implementation details.

## Step 3 — Verify Acceptance Criteria

- Double-check every item in the Agent Brief's "Acceptance criteria" list is fully satisfied.
- Run all project tests and linters to confirm nothing else was broken.

## Step 4 — Commit

```bash
git add -A
git commit -m "<type>: <description> (#{number})

Closes #{number}"
```

## Step 5 — Merge to {base_branch}

Use your judgment to complete the merge safely:

```bash
git fetch origin
git rebase origin/{base_branch}        # should be clean — conflict detection was done upfront
git push origin {branch}:{base_branch}
```

If push fails, investigate the reason and decide whether a retry is appropriate. If you cannot proceed safely, output `<promise>PUSH_FAILED #{number}: {reason}</promise>` and stop.

## Step 6 — Report & Close

1. Get the merge commit SHA: `git rev-parse --short HEAD`
2. Get test results summary by running the project's test command (capture pass/skip/fail counts)
3. Comment on the issue:
   ```bash
   gh issue comment {number} --body "### Test Results
   - {N} tests passing in {module} module
   - Full test suite: {total} passed, {skipped} skipped

   Branch: \`implementer/issue-{number}-{slug}\`
   Commit: {short_sha}"
   ```
4. Close the issue:
   ```bash
   gh issue close {number}
   ```
5. Delete the remote feature branch:
   ```bash
   git push origin --delete implementer/issue-{number}-{slug}
   ```
6. Output `<promise>COMPLETE #{number}</promise>`

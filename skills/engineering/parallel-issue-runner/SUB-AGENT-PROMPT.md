You are implementing GitHub issue #{number}: {title}.
This is a child of PRD #{prd_number}.

Use the /idd:implement-issue skill to complete the implementation and tests.
Branch off {base_branch}: implementer/issue-{number}-{slug}
Merge target: {base_branch} (origin/{base_branch})

After all tests pass, merge your branch to {base_branch} with a retry loop (max 3 attempts):

  attempt=0
  while attempt < 3:
    git fetch origin
    git rebase origin/{base_branch}        # always clean — conflict detection was done upfront
    git push origin {branch}
    if push succeeds:
      break
    attempt += 1

  if attempt == 3 and push still fails:
    Output <promise>PUSH_FAILED #{number}: {reason}</promise> and stop.

If merge succeeds:
1. Get the merge commit SHA: `git rev-parse --short HEAD`
2. Get test results summary by running the project's test command (capture pass/skip/fail counts)
3. Comment on the issue:
   ```
   gh issue comment {number} --body "### Test Results
   - {N} tests passing in {module} module
   - Full test suite: {total} passed, {skipped} skipped

   Branch: `implementer/issue-{number}-{slug}`
   Commit: {short_sha}"
   ```
4. Close the issue:
   `gh issue close {number}`
5. Delete the feature branch:
   ```bash
   git push origin --delete implementer/issue-{number}-{slug}
   git branch -d implementer/issue-{number}-{slug}
   ```
6. Output `<promise>COMPLETE #{number}</promise>`

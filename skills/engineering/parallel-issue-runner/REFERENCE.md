# Parallel Issue Runner Reference

This file contains the detailed operational procedure for `/idd:parallel-issue-runner`.

## 0. Detect Base Branch

Auto-detect the default remote branch:

```bash
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||'
# Fallback if unset:
git remote show origin | grep 'HEAD branch' | awk '{print $NF}'
```

Present and confirm:

> Detected base branch: `{base_branch}`. Is this correct, or would you like to use a different branch?

Then pull:

```bash
git checkout {base_branch} && git pull origin {base_branch}
```

## 1. List Child Issues

```bash
gh issue list --state open --search "#{prd_number}" --json number,title,state,body,labels,comments
```

Split into:

- `open_issues`: still open
- `done_issues`: already closed

If all are closed, jump to Step 6.

## 2. Resolve Blockers

Parse each open issue body for:

- `Blocked by #N`
- `Depends on #N`
- `Requires #N`
- checklist refs like `- [ ] #N`

For each blocker:

```bash
gh issue view {blocker_number} --json state,number,title
# Find issue comments that mention the implementation commit SHA
gh issue view {blocker_number} --comments
# Verify the referenced commit is reachable from the base branch
git fetch origin
git branch -r --contains {commit_sha} | grep "origin/{base_branch}"
```

Ready rule:

- Issue is ready only when every blocker has commit evidence already on `{base_branch}`.
- `CLOSED` status alone is not sufficient.
- Acceptable evidence is a commit SHA linked from the blocker issue that is contained in `origin/{base_branch}`.
- If a blocker is `CLOSED` but no such commit evidence is found, treat it as unresolved and report it explicitly.

Sets:

```text
ready_issues   = open_issues where all blockers resolved
blocked_issues = open_issues where one or more blockers unresolved
```

If `ready_issues` is empty, report blockers and stop.

## 3. Conflict Detection

For each issue in `ready_issues`, read file scope from Agent Brief comments (look for "Files", "Scope", "Touches"). If missing, fallback to label-to-path inference.

Check existing implementer branches:

```bash
git branch -r | grep implementer/issue
# For each:
git diff --name-only {base_branch}..origin/{branch}
```

Conflict rule:

- If two ready issues touch same path, keep lower-numbered issue.
- Defer higher-numbered issue and post comment:

```bash
gh issue comment {deferred} --body "Deferred this cycle: conflicts with #${lower} on {file}. Will retry after that branch merges."
```

## 4. Launch Sub-Agents

For each issue in conflict-free ready set (max 3 per batch), launch Agent tool with exact settings:

- `description`: `"Implement issue #{number}"`
- `subagent_type`: `"general-purpose"`
- `isolation`: `"worktree"`
- `run_in_background`: `true`
- `mode`: `"bypassPermissions"`
- `name`: `"issue-{number}-agent"`
- `prompt`: from `SUB-AGENT-PROMPT.md` with `{number}`, `{title}`, `{prd_number}`, `{slug}`, `{base_branch}`

Send all launches in one message to ensure real parallelism.
Each sub-agent works on `implementer/issue-{number}-{slug}` and merges that branch into `{base_branch}`.

If more than 3 are ready, process lowest-numbered 3 first and continue in next loop.

## 5. Monitor and Loop

On agent signals:

- `<promise>COMPLETE #{number}</promise>`: continue
- `<promise>PUSH_FAILED #{number}: {reason}</promise>`: investigate with:

```bash
git log --oneline origin/{base_branch} -5
git diff origin/{base_branch}..implementer/issue-{number}-{slug} --name-only
```

After all agents in the batch complete or fail:

1. Remove worktrees created in this batch:

```bash
git worktree remove --force <worktree-path>
```

2. Run full project test suite.

Then loop back to Step 1.

## 6. Close PRD

When all child issues are closed:

1. Re-read PRD acceptance criteria.
2. Confirm each criterion maps to closed issues/commits.
3. Run full test suite and verify pass.
4. Close PRD:

```bash
gh issue close {prd_number} --comment "All child issues resolved. PRD acceptance criteria met."
```

## Completion Output Template

After closing the PRD, output the following (substitute real values):

```text
PRD #<prd_number> is closed. Run `/clear` to reset the context, then paste this to do a final alignment check:

/idd:grill-with-docs Verify that PRD #<prd_number> has been fully implemented: read the merged code on <base_branch>, cross-check it against CONTEXT.md and the ADRs in docs/adr/ to confirm every acceptance criterion is met, every touched file aligns with the domain model, and no domain concepts were misused.

If everything looks good — well done! Start your next feature with `/idd:grill-with-docs <your next idea>` and the cycle begins again.
```

## Edge Cases

| Situation | Action |
|-----------|--------|
| Agent fails | Leave branch, report failure, continue others, re-queue next cycle |
| All ready issues conflict | Run sequentially, lowest issue number first |
| Blocker open but commit already on `{base_branch}` | Treat as resolved |
| Blocker closed but no commit evidence on `{base_branch}` | Treat as unresolved and report manual closure risk |
| PRD has no acceptance criteria | Warn user and ask confirmation before closing |

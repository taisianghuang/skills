---
name: parallel-issue-runner
description: Given a PRD issue number, find its already-triaged child issues, verify blockers are resolved and no file conflicts exist, then launch parallel sub-agents in git worktrees using implement-issue. Loops until all child issues are closed and the PRD is complete. Assumes to-prd, to-issues, and triage have already been run. Use when user says "run prd #N", "parallel implement", "execute issues for prd", or wants autonomous parallel issue execution.
argument-hint: "#<prd-issue-number>"
---

# Parallel Issue Runner

Assumes `to-prd → to-issues → triage` are already done. Child issues exist with Agent Briefs.

## Branch Convention

- **Orchestrator** runs on `{base_branch}` — detected at startup (see Step 0)
- **Sub-agents** each branch off `{base_branch}` → `implementer/issue-{number}-{slug}`
- **Merge target** is always `{base_branch}` — sub-agents rebase onto `origin/{base_branch}` and push directly to `{base_branch}`

## Quick Start

```
/parallel-issue-runner #<prd-issue-number>
```

**If no PRD number is provided**, ask before doing anything else:
> "Which PRD should I drive toward? Please provide the issue number (e.g. `#14`)."

Do not proceed until a PRD number is confirmed.

## Workflow

### 0. Detect Base Branch

Auto-detect the default remote branch:

```bash
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||'
# Fallback if unset:
git remote show origin | grep 'HEAD branch' | awk '{print $NF}'
```

Present the result to the user:

> Detected base branch: `{base_branch}`. Is this correct, or would you like to use a different branch?

Wait for confirmation before proceeding. Use the confirmed value as `{base_branch}` throughout this entire run.

Then switch to `{base_branch}` and pull:

```bash
git checkout {base_branch} && git pull origin {base_branch}
```

### 1. List Child Issues

```bash
# Get all issues referencing the PRD
gh issue list --state open --search "#{prd_number}" --json number,title,state,body,labels,comments
```

Split into:
- `open_issues` — still open
- `done_issues` — already closed (state: CLOSED)

If all child issues are CLOSED → jump to [Step 6 — Close PRD](#6-close-prd).

### 2. Resolve Blockers

Parse each open issue's body for:
- `Blocked by #N` / `Depends on #N` / `Requires #N`
- Checklist: `- [ ] #N`

For each blocker:
```bash
gh issue view {blocker_number} --json state,number
# Edge case: blocker open but PR merged
gh pr list --state merged --search "Closes #{blocker_number}"
```

**Rule**: Issue is **ready-for-agent** only if ALL blockers are CLOSED (or resolved via merged PR).

```
ready_issues   = open_issues where all blockers resolved
blocked_issues = open_issues where ≥1 blocker still open
```

If `ready_issues` is empty: report which issues are blocked by what → STOP.

### 3. Conflict Detection

For each issue in `ready_issues`, get the file scope from its Agent Brief comment (look for "Files", "Scope", or "Touches" section). Fall back to label-to-path inference (e.g. label `auth` → `src/auth/`).

Check existing open implementer branches:
```bash
git branch -r | grep implementer/issue
# For each: git diff --name-only {base_branch}..origin/{branch}
```

**Conflict rule**: Two issues sharing a file path → keep lower-numbered, defer higher. Post comment on deferred issue:
```bash
gh issue comment {deferred} --body "Deferred this cycle: conflicts with #${lower} on {file}. Will retry after that branch merges."
```

### 4. Launch Parallel Sub-Agents

For each issue in the conflict-free ready set (max 3 at a time), call the **Agent tool** with these exact parameters:

- `description`: `"Implement issue #{number}"`
- `subagent_type`: `"general-purpose"`
- `isolation`: `"worktree"`
- `run_in_background`: `true` — REQUIRED; without this agents run sequentially
- `mode`: `"bypassPermissions"` — background agents cannot receive input; bypass all checks to prevent deadlock
- `name`: `"issue-{number}-agent"` — allows SendMessage for progress checks later
- `prompt`: see [SUB-AGENT-PROMPT.md](./SUB-AGENT-PROMPT.md), substituting `{number}`, `{title}`, `{prd_number}`, `{slug}`, `{base_branch}`

Send all agents in a **single message** (multiple Agent tool calls at once) to achieve true parallelism.

After launching, wait for background completion notifications before proceeding to Step 5.

If >3 issues are ready, run the lowest-numbered 3 first, then loop.

### 5. Monitor & Loop

For each agent signal received:

- `<promise>COMPLETE #{number}</promise>` → issue is merged and closed, continue
- `<promise>PUSH_FAILED #{number}: {reason}</promise>` → investigate manually:
  ```bash
  git log --oneline origin/{base_branch} -5
  git diff origin/{base_branch}..implementer/issue-{number}-{slug} --name-only
  ```
  Decide whether to retry, rebase manually, or defer the issue.

After all agents in the batch complete (or fail), verify full test suite:
```bash
hatch run check
```

**LOOP** back to Step 1.

### 6. Close PRD

When all child issues are CLOSED:

1. Re-read PRD acceptance criteria
2. Confirm each criterion maps to a closed issue or commit
3. Run full test suite: `hatch run check`
4. Close:
   ```bash
   gh issue close {prd_number} --comment "All child issues resolved. PRD acceptance criteria met."
   ```

After closing the PRD, remind the user to do a final alignment check in a fresh context:

> PRD #{prd_number} is closed. Run `/clear` to reset the context, then paste this to do a final alignment check:
>
> `/grill-with-docs Verify that PRD #{prd_number} has been fully implemented: read the merged code on main, cross-check it against CONTEXT.md and the ADRs in docs/adr/ to confirm every acceptance criterion is met, every touched file aligns with the domain model, and no domain concepts were misused.`
>
> If everything looks good — well done! Start your next feature with `/grill-with-docs <your next idea>` and the cycle begins again.

## Loop Diagram

```
List open child issues
  ├─ All closed? → Close PRD → DONE
  └─ Some open ↓
Resolve blockers → ready_issues
  ├─ Empty? → Report blockers → STOP
  └─ Some ready ↓
Conflict detection → defer conflicts
  ↓
Launch ≤3 parallel agents (implement-issue in worktrees)
  ↓ each agent:
  implement → test → rebase → push (retry ≤3) → comment commit SHA → close issue
  ├─ PUSH_FAILED? → Orchestrator investigates manually
  └─ COMPLETE? → continue
  ↓
hatch run check
  ↓
LOOP ↑
```

## Edge Cases

| Situation | Action |
|-----------|--------|
| Agent fails | Leave branch; report; continue others; re-queue next cycle |
| All ready issues conflict | Run sequentially, lowest # first |
| Blocker open but PR merged | Treat as closed |
| PRD has no acceptance criteria | Warn user, ask for confirmation before closing PRD |

## Checklist

```
[ ] Open child issues fetched via gh issue list --search "#{prd_number}"
[ ] Blocker state verified (including merged PR check)
[ ] File conflict check done against ready set and existing branches
[ ] Max 3 agents launched in parallel per batch
[ ] Each agent uses its own git worktree (isolation: "worktree")
[ ] Each agent prompt references /implement-issue skill and PRD number
[ ] Each agent self-merges with push retry loop (max 3 attempts)
[ ] Each agent comments commit SHA on its issue and closes it
[ ] PUSH_FAILED signals handled by orchestrator
[ ] Loop continues until all child issues closed
[ ] PRD acceptance criteria verified before closing PRD
```

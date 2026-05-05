---
name: idd:parallel-issue-runner
description: Given a PRD issue number, find its already-triaged child issues, verify blockers are resolved and no file conflicts exist, then launch parallel sub-agents in git worktrees using implement-issue. Loops until all child issues are closed and the PRD is complete. Assumes to-prd, to-issues, and triage have already been run. Use when user says "run prd #N", "parallel implement", "execute issues for prd", or wants autonomous parallel issue execution.
argument-hint: "#<prd-issue-number>"
---

# Parallel Issue Runner

Assumes `to-prd -> to-issues -> triage` are already done. Child issues already have Agent Briefs.

## Quick Start

```
/idd:parallel-issue-runner #<prd-issue-number>
```

If no PRD number is provided, ask first and do not proceed:
> "Which PRD should I drive toward? Please provide the issue number (e.g. `#14`)."

## Branch Convention

- Orchestrator runs on `{base_branch}`.
- Sub-agents branch from `{base_branch}` to `implementer/issue-{number}-{slug}`.
- Sub-agents merge `implementer/issue-{number}-{slug}` back into `{base_branch}`.

## Workflow

1. Detect and confirm `{base_branch}`, then pull latest.
2. List child issues for the PRD and split into open vs done.
3. Resolve blockers (`Blocked by`, `Depends on`, checklist refs).
4. Detect file conflicts; defer higher-numbered conflicts with an issue comment.
5. Launch up to 3 ready issues in sub-agents (worktree isolation).
6. Monitor agent signals (`COMPLETE` / `PUSH_FAILED`), cleanup worktrees, run full tests.
7. Loop until all child issues are closed.
8. Verify acceptance criteria, then close the PRD.

For full commands, edge cases, and completion message template, see [REFERENCE.md](./REFERENCE.md).

## Required Sub-Agent Settings

Use these exact settings when launching parallel workers:

- `description`: `Implement issue #{number}`
- `subagent_type`: `general-purpose`
- `isolation`: `worktree`
- `run_in_background`: `true`
- `mode`: `bypassPermissions`
- `name`: `issue-{number}-agent`
- `prompt`: [SUB-AGENT-PROMPT.md](./SUB-AGENT-PROMPT.md) with placeholder substitution

## Checklist

```
[ ] Open child issues fetched via gh issue list --search "#{prd_number}"
[ ] Blockers verified using commit evidence on `{base_branch}`, not only issue closed state
[ ] File conflict check done against ready set and existing branches
[ ] Max 3 agents launched in parallel per batch
[ ] Each agent uses its own git worktree (isolation: "worktree")
[ ] Each agent prompt references /idd:implement-issue skill and PRD number
[ ] Each agent merges `implementer/issue-{number}-{slug}` into `{base_branch}` or reports a push failure when it cannot proceed safely
[ ] Each agent comments commit SHA on its issue and closes it
[ ] PUSH_FAILED signals handled by orchestrator
[ ] Loop continues until all child issues closed
[ ] PRD acceptance criteria verified before closing PRD
```

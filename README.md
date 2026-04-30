# idd: Skills — AI-Driven Development Workflow

A fork of [mattpocock/skills](https://github.com/mattpocock/skills) extended with a full end-to-end loop workflow, namespaced under `idd:` (AI-Driven Development).

> Based on [MIT License](./LICENSE). Original work by Matt Pocock.

## Quickstart

```bash
npx skills@latest add taisianghuang/skills
```

Pick the skills you want, then run `/idd:setup-matt-pocock-skills` once per repo. It will configure:
- Issue tracker (GitHub, GitLab, or local files)
- Triage label vocabulary
- Domain doc layout (`CONTEXT.md`, `docs/adr/`)

---

## The idd: Loop

The core idea is a **repeating feature loop** driven by chained skills. Each skill hands off to the next, and `parallel-issue-runner` loops internally until the PRD is complete.

```
  ┌─────────────────────────────────────────────────────────────┐
  │                      NEW FEATURE CYCLE                      │
  │                                                             │
  │  /idd:grill-with-docs <idea>                                │
  │       ↓ align on domain language + sharpen requirements     │
  │  /idd:to-prd                                                │
  │       ↓ synthesise into PRD, publish to issue tracker       │
  │  /idd:to-issues                                             │
  │       ↓ break PRD into vertical slice issues                │
  │  /clear  →  /idd:triage #<issue>                            │
  │       ↓ evaluate, write Agent Brief, label ready-for-agent  │
  │  /clear  →  /idd:parallel-issue-runner #<prd>               │
  │       │                                                     │
  │       │  ┌─── INNER LOOP ────────────────────────────────┐  │
  │       │  │                                               │  │
  │       │  │  List open child issues                       │  │
  │       │  │    ├─ All closed? → Close PRD → EXIT LOOP     │  │
  │       │  │    └─ Some open ↓                             │  │
  │       │  │  Resolve blockers → ready_issues              │  │
  │       │  │    ├─ Empty? → Report & STOP                  │  │
  │       │  │    └─ Some ready ↓                            │  │
  │       │  │  Conflict detection → defer conflicts         │  │
  │       │  │    ↓                                          │  │
  │       │  │  Launch ≤3 parallel agents in worktrees       │  │
  │       │  │    ↓ each agent:                              │  │
  │       │  │    /idd:implement-issue → /idd:tdd            │  │
  │       │  │    → test → rebase → push → close issue       │  │
  │       │  │    ↓                                          │  │
  │       │  │  hatch run check (full suite)                 │  │
  │       │  │    ↓                                          │  │
  │       │  └─── LOOP BACK ───────────────────────────────-─┘  │
  │       │                                                     │
  │       ↓ PRD closed                                          │
  │  /clear  →  /idd:grill-with-docs  (verify implementation)   │
  │       ↓ confirmed ✓                                         │
  │                                                             │
  └──────────────── START NEXT FEATURE ─────────────────────────┘
```

### Why the loop matters

Without a loop, agents implement in one shot and stop. The inner loop in `parallel-issue-runner` handles real-world complexity:

- **Blockers** — issues that depend on others wait until blockers are closed
- **File conflicts** — two agents touching the same file run sequentially, not in parallel
- **Push failures** — each agent retries rebase + push up to 3 times before escalating
- **Incremental progress** — each batch closes some issues, then loops to pick up the rest

The outer loop (`grill-with-docs` → verify) ensures the delivered code actually matches the original intent and domain model — not just the acceptance criteria.

---

## Session Boundaries

Each skill signals when to `/clear` context before the next step:

| After | Action |
|---|---|
| `/idd:setup-matt-pocock-skills` | `/clear` → `/idd:grill-with-docs` or `/idd:grill-me` |
| `/idd:grill-with-docs` | (same session) → `/idd:to-prd` |
| `/idd:to-prd` | (same session) → `/idd:to-issues` |
| `/idd:to-issues` | `/clear` → `/idd:triage #<n>` |
| `/idd:triage` | `/clear` → `/idd:parallel-issue-runner #<prd>` |
| `/idd:parallel-issue-runner` | `/clear` → `/idd:grill-with-docs` (verify) |

`/clear` before triage and parallel-runner prevents context bloat from earlier planning steps interfering with execution.

---

## Reference

### Engineering

Skills for daily code work.

- **[idd:setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)** — Configure issue tracker, triage labels, and domain docs. Run once per repo.
- **[idd:grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)** — Align on requirements, sharpen domain language, update `CONTEXT.md` and ADRs inline.
- **[idd:to-prd](./skills/engineering/to-prd/SKILL.md)** — Synthesise the session into a PRD and publish to the issue tracker.
- **[idd:to-issues](./skills/engineering/to-issues/SKILL.md)** — Break a PRD into independently-grabbable vertical slice issues.
- **[idd:triage](./skills/engineering/triage/SKILL.md)** — Evaluate issues, write Agent Briefs, apply triage labels.
- **[idd:parallel-issue-runner](./skills/engineering/parallel-issue-runner/SKILL.md)** — Orchestrate parallel sub-agents across git worktrees; loops until the PRD is complete.
- **[idd:implement-issue](./skills/engineering/implement-issue/SKILL.md)** — Implement a single issue end-to-end using TDD, following the Agent Brief contract.
- **[idd:tdd](./skills/engineering/tdd/SKILL.md)** — Red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.
- **[idd:diagnose](./skills/engineering/diagnose/SKILL.md)** — Disciplined debug loop: reproduce → minimise → hypothesise → instrument → fix → regression-test.
- **[idd:improve-codebase-architecture](./skills/engineering/improve-codebase-architecture/SKILL.md)** — Find deepening opportunities informed by `CONTEXT.md` and ADRs. Run every few days.
- **[idd:zoom-out](./skills/engineering/zoom-out/SKILL.md)** — Get a higher-level perspective on an unfamiliar section of code.

### Productivity

General workflow tools, not code-specific.

- **[idd:grill-me](./skills/productivity/grill-me/SKILL.md)** — Get relentlessly interviewed about a plan or design until every branch is resolved.
- **[idd:caveman](./skills/productivity/caveman/SKILL.md)** — Ultra-compressed communication mode. Cuts token usage ~75%.
- **[idd:write-a-skill](./skills/productivity/write-a-skill/SKILL.md)** — Create new skills with proper structure and bundled resources.

### Misc

- **[idd:git-guardrails-claude-code](./skills/misc/git-guardrails-claude-code/SKILL.md)** — Block dangerous git commands before they execute.
- **[idd:migrate-to-shoehorn](./skills/misc/migrate-to-shoehorn/SKILL.md)** — Migrate `as` type assertions to @total-typescript/shoehorn.
- **[idd:scaffold-exercises](./skills/misc/scaffold-exercises/SKILL.md)** — Create exercise directory structures.
- **[idd:setup-pre-commit](./skills/misc/setup-pre-commit/SKILL.md)** — Set up Husky pre-commit hooks with lint-staged, Prettier, type checking, and tests.

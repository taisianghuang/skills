---
name: idd:help
description: Explain the idd workflow, route the user to the right next idd skill, and summarize prerequisites and session boundaries. Use when user asks how idd works, which idd skill to use next, what command comes after another step, or wants a quick workflow refresher.
---

# IDD Help

Use this skill to orient the user inside the idd workflow and point them at the next command.

## Quick Start

Use `/idd:help` when the user asks things like:

- "How does the idd workflow work?"
- "What do I run after `/idd:to-issues`?"
- "Which idd skill should I use next?"
- "Why do I need `/clear` here?"

## Workflow

1. Identify the user's current stage: setup, clarification, PRD creation, issue breakdown, triage, parallel execution, verification, or architecture cleanup.
2. Explain the smallest relevant slice first. Only explain the full loop if the user asks for an overview.
3. Route to the next skill with the exact command format.
4. Mention prerequisites when relevant: configured issue tracker, `gh` auth, `CONTEXT.md`, ADRs, or triaged issues.
5. Call out `/clear` boundaries when the next step should start in a fresh context.

## Common Routes

- New repo setup: `/idd:setup-matt-pocock-skills`
- Turn an idea into clarified requirements: `/idd:grill-with-docs <idea>` or `/idd:grill-me <idea>`
- Publish the current plan as a PRD: `/idd:to-prd`
- Break a PRD into vertical slices: `/idd:to-issues`
- Prepare issues for execution: `/idd:triage #<issue>`
- Execute a full PRD: `/idd:parallel-issue-runner #<prd>`
- Implement one issue manually: `/idd:implement-issue #<issue>`
- Debug a broken slice: `/idd:diagnose`
- Review architecture after a few cycles: `/idd:improve-codebase-architecture`

## Session Boundaries

- After setup: `/clear` before planning or grilling
- After issue breakdown: `/clear` before triage
- After triage: `/clear` before parallel execution
- After parallel execution: `/clear` before final verification with `grill-with-docs`

## Answer Style

- Prefer concrete routing over abstract explanation.
- Include the exact next command whenever possible.
- If the user is off the happy path, explain why and route them back to the nearest valid step.
- If they ask for the whole loop, summarize it in one pass from idea -> PRD -> issues -> triage -> execution -> verification.
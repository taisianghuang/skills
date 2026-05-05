---
name: idd:implement-issue
description: Implement a GitHub issue end-to-end using TDD, following the Agent Brief contract. Use when user wants the agent to pick up a ticket, write code to fulfill its acceptance criteria, commit, push, and close the issue.
---

# Implement Issue

Follow this strict mattpocock workflow to implement a GitHub issue end-to-end.

## Quick start

When the user says "Implement issue #42" or asks you to take ownership of an issue, execute the workflow below against that issue.

## Workflows

### 1. Gather the Contract

- Fetch the issue from the issue tracker.
- Read both the issue body and the "Agent Brief" in the comments.
- The Agent Brief is your absolute contract. Pay special attention to the "Out of scope" section—do absolutely NO gold-plating outside of these boundaries.

### 2. Test-Driven Implementation

- Use the `/idd:tdd` skill to drive the implementation.
- First, write failing tests against the interfaces specified in the brief. Ensure you are testing behaviors at the boundaries (e.g. Services, Ports), not internal implementation details.
- Run the tests to see them fail.
- Implement the requested features until all tests pass.

### 3. Verify Acceptance Criteria

- Before committing, double-check that every single item in the Agent Brief's "Acceptance criteria" list is fully satisfied.
- Run all project tests and linters to ensure nothing else was broken.

Stop here. The caller (e.g. `parallel-issue-runner`) is responsible for committing, pushing, commenting, and closing the issue.

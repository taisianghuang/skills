---
name: implement-issue
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

- Use the `/tdd` skill to drive the implementation.
- First, write failing tests against the interfaces specified in the brief. Ensure you are testing behaviors at the boundaries (e.g. Services, Ports), not internal implementation details.
- Run the tests to see them fail.
- Implement the requested features until all tests pass.

### 3. Verify Acceptance Criteria

- Before committing, double-check that every single item in the Agent Brief's "Acceptance criteria" list is fully satisfied.
- Run all project tests and linters to ensure nothing else was broken.

### 4. Commit & Push

- Stage all your changes.
- Create a conventional commit message describing what was done (e.g., `feat: implement user registration (#42)`).
- Ensure you include `Resolves #<issue-number>` or `Closes #<issue-number>` in the commit message body to automatically link the issue.
- Push the changes to the current working branch.

### 5. Report & Close

- Use the issue tracker CLI to add a comment on the issue.
- Provide a concise summary of the changes made.
- Close the issue.

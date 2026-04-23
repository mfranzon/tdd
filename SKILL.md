---
name: tdd
description: "Use when the user invokes /tdd or asks to do TDD on a feature. Accepts a plan file path or inline feature description. Writes failing tests, implements minimal passing code, runs the full test suite, and suggests refactoring — guiding the developer through strict Red-Green-Refactor cycles."
arguments:
  - name: input
    description: "Either a path to a markdown plan file OR an inline description of the feature to implement via TDD (e.g., '/tdd plan.md' or '/tdd add JWT authentication with refresh tokens')"
---

# TDD Skill — Red-Green-Refactor

Guide the developer through strict Test-Driven Development. Write code directly to real files — the user can always undo with git. Pause only when user input is needed.

## Phase 1: Setup

1. Determine the input type:
   - **File path** (ends in `.md`, `.txt`, or exists on disk): read the plan file.
   - **Inline description** (e.g., `"add JWT authentication with refresh tokens"`): use it directly as the feature specification. Ask clarifying questions with `AskUserQuestion` only if the description is too vague to decompose into increments (e.g., "build an app").
2. Detect the language and test framework from the project. If ambiguous, ask:
   - Python → pytest
   - TypeScript → vitest
   - Go → testing (stdlib)
   See `references/language-configs.md` for runner commands and conventions.
3. Break the feature into small, testable increments. See `references/increments.md` for decomposition patterns.
4. Create a task list using `TaskCreate` — one task per increment.
5. Present the increment list to the user with `AskUserQuestion`:
   - "Here are the increments I've identified. Want to reorder, add, remove, or modify any before we start?"
   - Options: "Looks good, let's start" / "I want to modify the list"

## Phase 2: TDD Loop

For each increment, follow Red-Green-Refactor strictly. Mark the current increment as `in_progress` via `TaskUpdate`.

### RED — Write a Failing Test

1. Write the failing test **directly to the real test file** (e.g., `tests/test_feature.py`, following existing project conventions). Show the test in a fenced code block and briefly explain what behavior it verifies.
2. Run the test using the appropriate runner (see `references/language-configs.md`). Use `Bash`.
3. Confirm the test **fails**. If it passes unexpectedly, stop and flag:
   - "The test passed already — the behavior is either already implemented or the test isn't asserting the right thing. Let's investigate before moving on."
4. Pause with `AskUserQuestion`:
   - "RED: test fails as expected. Review the test above — ready to move to GREEN?"
   - Options: "Looks good, write the code" / "I want to change the test first"

### GREEN — Write Minimal Code to Pass

5. Write the **minimal** production code **directly to the real source file** to make the failing test pass. Show it in a fenced code block.
6. Run **all** tests (not just the new one). Use `Bash`.
7. If all tests **pass**, move directly to REFACTOR.
8. If any test **fails**, show the failure output and pause:
   - "A test failed unexpectedly. Here's the output. Want me to fix it, or do you want to handle it?"
   - Options: "Fix it" / "I'll handle it"

### REFACTOR — Improve the Code

9. Review the current code for duplication, naming, extraction, or simplification opportunities. Apply improvements directly and run all tests to confirm green. Show what changed in a fenced code block.
10. If no refactoring is needed, say so and move on.
11. If refactoring causes a test failure, revert and pause to discuss.

### NEXT

12. Mark the increment as `completed` via `TaskUpdate`.
13. Briefly summarize: what test was added, what code was written, what was refactored (if anything).
14. Move directly to the next increment — go back to RED.

## Phase 3: Wrap-up

After all increments are complete:

1. Summarize what was built:
   - List each increment and its test
   - Note the final test count and all-passing status
2. Suggest any remaining work (edge cases, integration tests, documentation).

## Rules

- **Write directly to real files.** No temp files, no asking the user to move code.
- **Keep tests behavior-focused.** Test what the code does, not how it does it.
- **Simplest case first.** Start with the degenerate/edge case, build toward the general case.
- **One assertion per test** when possible. Each test should verify one behavior.
- **Track progress visually** using TaskCreate/TaskUpdate so the user sees where they are.
- **Follow existing project conventions.** Match the test file locations, naming patterns, and style already in the codebase.

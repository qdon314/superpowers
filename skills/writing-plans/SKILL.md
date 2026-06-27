---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write implementation plans that specify **what** to build, **where** it goes,
and **how to know it's done**. Leave **how** to build it to the executor.

Assume the engineer reading this is a skilled developer who has zero context
for our codebase and questionable taste. They need: architecture rationale,
exact file paths, type contracts, acceptance criteria, constraints, and
ordering. They do NOT need function bodies, test code, shell commands, or
step-by-step TDD instructions — the executor owns those. DRY. YAGNI. TDD.
Frequent commits.

**Why no code:** the planner writes blind — no file is open, so any code is a
guess encoded as a decision. The executor has the real code in context and is
the one who should write it. Code in a plan also duplicates what will live in
the codebase, drowns the signal (what/why/done) in implementation noise, and
makes the plan expensive to write and review.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `superpowers:using-git-worktrees` skill at execution time.

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Task Right-Sizing

A task is a **commit-sized deliverable**: one logical unit of work that
leaves the codebase in a valid state and is worth a fresh reviewer's gate.
Typical size: one module or one adapter. When drawing task boundaries: fold
setup, configuration, scaffolding, and documentation into the task whose
deliverable needs them; split only where a reviewer could meaningfully reject
one task while approving its neighbor. If a task needs more than ~3 files
created, consider splitting it. Each task ends with an independently testable
deliverable.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Design doc:** [path to design doc, if one exists]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

## Rationale

[Why these tasks, in this order. What dependency structure drives the
sequencing. Which task groups can be parallelized and why.]

---
```

## Task Structure

A plan task names a deliverable and its contract. It does not contain the
implementation — the executor designs and writes that against the acceptance
criteria, following TDD.

````markdown
### Task N: [Component Name]

**Why:** [1-2 sentences connecting this task to the larger plan]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [this task's public contract — the surface later tasks may rely
  on, stated even when this is a leaf task nothing downstream consumes:
  exact function names, parameter and return types, dataclass/protocol
  shapes. A task's implementer sees only their own task; this block is how
  they learn the names and types neighboring tasks use. A small illustrative
  fragment is allowed HERE only to pin an exact data shape or a worked
  input→output example — never a function body.]

**Acceptance:**
- [Observable behaviors that must be true when done]
- [Edge cases and error conditions to test]
- [Performance or stability requirements, if any]

These describe WHAT to test, not the test code. The executor turns them into
failing tests, then implements to pass them.

**Constraints:**
- [Architectural rules: port compliance, layer boundaries]
- [Stability guarantees: "this value is a cache key"]
- [Compatibility: "must work with existing X adapter"]
````

### Parallel Task Groups

Default ordering is sequential. Mark independent tasks explicitly:

```markdown
### Tasks N–M (parallel): [Group Description]

> These tasks are independent and can be executed in parallel.
> All depend on: Task X, Task Y.

### Task N: ...
### Task N+1: ...
```

## No Placeholders

A plan task must give the executor everything they need to build the right
thing — and nothing that does their job for them. Both halves below are
**plan failures** — never write them:

**Too little — vague where the plan must be exact:**
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
  (name the cases in Acceptance instead)
- "Similar to Task N" (restate the contract — the engineer may read tasks out
  of order)
- References to types, functions, or methods not defined in any task's
  Interfaces block

**Too much — code where a contract belongs:**
- Function or method bodies (the executor writes the implementation)
- Test code or test functions (Acceptance says what to test, not how)
- Shell commands — no pytest, git, or build commands
- A red-green-refactor step sequence (the executing skill owns the workflow)
- Commit messages (the executor writes these from what they built)

The one exception: a small fragment in an Interfaces **Produces** block to pin
an exact data shape, signature, or worked example — never an implementation.

## Remember
- Exact file paths always
- Contracts, not code
- Acceptance criteria, not test implementations
- Constraints capture the *why* — what the executor can't safely change
- DRY, YAGNI, TDD, frequent commits
- Reference the design doc when one exists

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task whose Acceptance criteria cover it? List any gaps.

**2. Altitude scan:** Search your plan for both failure modes in "No Placeholders" above — vague prose AND leaked implementation code (function bodies, test code, shell commands). Fix them.

**3. Interface consistency:** Does every `Consumes` entry match a `Produces` from an earlier task? Do the names and types line up across tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- Batch execution with checkpoints for review

---
name: plan
description: Plan applications and features with the user through code-level module contracts before implementation.
---

# Plan

## When to Use

- The user wants to plan a new application or feature before building it.
- A change to an existing application needs agreement on module behavior or boundaries.
- An idea needs concrete data, API, and behavioral test contracts before implementation.

Do not use this skill to implement an already approved contract or for a small fix that does not
change module contracts.

## Purpose

Turn an application or feature idea into a concrete contract that another agent can implement
without guessing the intended behavior.

**Plan module behavior and boundaries, not internal implementation mechanics.** The code is the
single source of truth for what each module promises: responsibilities, data definitions, API
declarations or stubs, and behavioral test skeletons. This includes internal module interfaces, not
only externally public APIs.

`spec-<feature-name>.md` is a local planning anchor: it records the goal, why significant choices were made,
approval status, verification notes, and the contract commit hash. Code records **what the system
promises**; the spec records **why the agreed direction was chosen**. Do not copy signatures, field
lists, or behavioral rules into the spec. Never commit this planning anchor.

## Workflow

```text
Understand the application and goal
                |
                v
Write a short goal proposal in spec-<feature-name>.md
                |
         User confirms? <---- Revise
                |                ^
                | no ------------|
                v yes
Define modules, data, APIs, and behavioral tests together
                |
                v
Check that the contract is concrete and consistent
                |
         User approves? <---- Revise
                |                ^
                | no ------------|
                v yes
Create the contract commit -> Record its hash in the local spec -> Stop
```

Do not pass either approval gate without explicit user confirmation. Silence is not approval.
Approval of the goal is not approval of the final contract.

When resuming, read the local planning anchor and relevant code contracts first. Continue from the
last explicitly approved state instead of restarting the workflow. Reopen approval only for changes
affecting the agreed goal or contract; do not treat unapproved edits as part of the baseline.

## 1. Understand the Goal

For an existing application, inspect only enough to understand its purpose, major modules, existing
contracts, and the area being changed. Start with the directory structure and relevant public
interfaces. Do not use the README to establish current behavior; it may be outdated. Do not explore
every implementation detail.

For a new application, establish the intended users, their problem, and the desired outcome with the
user. Do not invent a technology stack just to make the proposal look complete.

Write a goal proposal of **at most roughly 500 words** covering:

- What the application does and who it serves.
- What makes its approach important or distinctive, if anything.
- Its major existing contracts or constraints, where relevant.
- What this planning effort should achieve, and what is outside its scope.

Resolve questions about the goal in conversation before writing the proposal. Never put open
questions or question checklists in the spec.

For a feature, distinguish the overall application's purpose from the specific change. The feature
must support that purpose.

Save the proposal in `spec-<feature-name>.md`, mark it **Goal: proposed**, and ask the user to confirm or correct
it. After explicit confirmation, mark it **Goal: approved**. This section is the anchor for
subsequent decisions; reread it when scope or direction becomes uncertain.

Choose a short, descriptive kebab-case name for the feature or application, for example
`spec-task-management.md`. Keep that filename stable throughout the planning effort. When resuming,
use the matching planning anchor rather than creating another one.

Reuse an existing document only if it is already a local, uncommitted planning anchor for this scope.
Do not repurpose tracked specifications or overwrite unrelated documents. If the chosen path
conflicts with an existing document, agree on a separate local location with the user. The same
content and no-commit rules apply at that location.

## 2. Collaborate Through Files

The filesystem is the primary workspace for the evolving code contract. Use conversation or comments
directly on the relevant function stubs for questions and proposals. **No questions belong in the
spec.**

- Show concrete modeling proposals in code, with examples where helpful.
- Invite the user to edit or comment on the contract files. Read their changes before the next
  revision.
- Resolve questions in conversation or on the relevant function stubs. Once resolved, replace
  question comments with the agreed contract.
- Record the rationale for confirmed, significant decisions in the spec, not unresolved choices or
  copies of contract definitions. For example: “Explicit state-setting was chosen to make retries
  safe,” not a duplicate of the operation's signature and behavior.
- Clearly distinguish proposed code from approved code. Never record your own recommendation as user
  approval.
- Ask focused questions about decisions that affect the contract. Do not turn every naming choice
  into an interview.
- When the user proposes a model, refine it rather than replacing it without explanation. Show the
  alternative and its practical tradeoff.
- Preserve unrelated content and user edits. If instructions conflict, expose the conflict and wait
  for clarification.

For example, replace “How should completion work?” with:

> Proposed: `setCompleted(id, completed)` sets an explicit state. Repeating the same request leaves
> the state unchanged. This is safer to retry than `toggleCompleted(id)`. Is that the intended
> behavior?

Keep the spec current rather than accumulating contradictory drafts. Retain a short decision note
when the reason for a choice will matter later.

## 3. Define the Contract Together

Agree on the modules needed for the goal. A module is a meaningful responsibility with a boundary,
not every class or helper function. A small feature may need only one module.

Draft code contracts while discussing each module; do not wait for every design choice to be settled
before showing a proposal. Treat those drafts as proposed until the user approves them.

For each module, define the following at the level required to remove behavioral ambiguity:

| Area                 | Define                                                                                                          | Do not detail                                                        |
| -------------------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Responsibility       | What the module owns, exposes, and depends on                                                                   | Private class hierarchies or helper decomposition                    |
| Data and persistence | Entities, field types, required/optional values, defaults, identities, relationships, and integrity constraints | Query optimizations or storage machinery unless required by the goal |
| API                  | Inputs, outputs, error definitions and delivery, observable side effects, and relevant state transitions        | Internal algorithms or private functions                             |
| Behavioral tests     | Preconditions, actions, and externally observable outcomes                                                      | Tests of private helpers or implementation-specific call sequences   |

An API may be a module interface, HTTP endpoint, CLI command, event, or another consumer-facing
boundary. Do not assume it means REST.

A type alone is not a database model. If persistence is in scope, also define relevant keys,
relationships, uniqueness, nullability, and ownership of generated values. If persistence is not
needed, state that rather than inventing a database.

Make the expected application behavior clear at module boundaries, including internal boundaries.
Resolve relevant edge cases with the user: what happens for invalid input, missing data, repeated
requests, partial failure, or conflicting updates? Include only cases relevant to the agreed goal;
do not invent requirements.

Define error shapes or reference existing error definitions, and specify how callers receive them:
thrown/rejected errors, result values, HTTP responses, or another established mechanism. An error
name alone is not a complete contract.

Leave implementation mechanics open. Choosing a loop versus `map`, private helper extraction, or an
equivalent internal algorithm is not a planning question unless it changes agreed behavior or
constraints.

For an existing application, reuse compatible contracts. Identify any intended breaking change and
its effect on existing callers or stored data before approval.

### Make the Agreement Concrete in Code

Express proposals in the project's language and conventions, then revise them as choices are
resolved:

- Data types and schema definitions.
- Public interface declarations or function stubs with explicit inputs and outputs.
- Behavioral test skeletons with concrete expected outcomes.

Use existing artifact locations where practical. Express module responsibilities, dependencies, and
contract semantics in those code files, using declarations and adjacent comments as needed. Do not
reproduce them in the spec. Do not add a framework or dependencies merely to express a plan; if the
project has no chosen stack, agree on the representation with the user first.

Prefer interface declarations when they express the boundary fully. When a function body is
necessary, make it fail explicitly as unimplemented. Do not return fake successful results. Do not
replace working code with stubs or wire unimplemented contracts into production execution paths.

Behavioral test skeletons must describe enough to become real tests without rediscovering the
requirements. A title such as “creates a todo” is not enough. Include concrete setup, action, and
expected results in the body or structured comments. Mark them clearly as unimplemented using the
project's convention; do not create empty passing tests. Record that they do not yet verify an
implementation.

### Compilation and Test Status During Planning

- Compilation is not required at this stage. Keep declarations and stubs syntactically valid where
  practical, and record known compilation blockers. Do not implement behavior merely to make checks
  green.
- New, unimplemented behavioral test skeletons must fail explicitly as unimplemented when runnable.
  A compilation failure does not count as a behavioral test failure. Do not claim a test ran if
  compilation prevented execution.
- Existing tests and contract tests already backed by working behavior do not need to fail. Do not
  skip, weaken, or intentionally break them.
- Planning is complete through contract clarity and explicit user approval, not a passing build or
  test suite.

### Example: A Small Task Contract

TypeScript is used only for illustration. Use the project's language and testing conventions;
do not impose these fields, operations, or syntax on every application.

```typescript
// Persistence: id is the primary key; all fields are required.
// The service generates unique, non-empty IDs and UTC ISO 8601 timestamps.
interface Task {
  id: string;
  title: string;
  completed: boolean;
  createdAt: string;
  updatedAt: string;
}

// Failures are Promise rejections with this shape, not successful task values.
// Reuse existing project error definitions when available.
interface TaskError extends Error {
  code: "INVALID_TITLE" | "TASK_NOT_FOUND";
}

interface TaskService {
  // Trims title; rejects a blank title with TaskError.code="INVALID_TITLE".
  // Persists and returns a task with completed=false and equal timestamps.
  createTask(title: string): Promise<Task>;

  // Unknown id rejects with TaskError.code="TASK_NOT_FOUND".
  // Persists and returns the requested state; changes updatedAt only when
  // completed changes. Preserves all other fields.
  setCompleted(id: string, completed: boolean): Promise<Task>;
}
```

Use descriptive behavior names for tests, without numbered prefixes or tracking IDs. These example
skeletons fail explicitly until implemented; they do not yet verify behavior. Adapt test registration
and failure syntax to the project's existing conventions.

```typescript
test("creates and persists a task", () => {
  // Given an empty store and clock fixed at 2030-01-01T12:00:00Z.
  // When createTask(" Buy milk ") is called.
  // Then resolve with a persisted task containing a generated, non-empty id,
  // title="Buy milk", completed=false, and both timestamps equal to now.
  throw new Error("Unimplemented contract test");
});

test("rejects a blank title without persisting", () => {
  // Given an empty store.
  // When createTask("   ") is called.
  // Then reject with TaskError.code="INVALID_TITLE"; the store stays empty.
  throw new Error("Unimplemented contract test");
});

test("repeating a completion request preserves state", () => {
  // Given an incomplete task and a clock advanced beyond its creation time.
  // When setCompleted(id, true) is called twice.
  // Then both calls resolve with completed=true; the first advances updatedAt;
  // the second preserves updatedAt. Both preserve id, title, and createdAt.
  throw new Error("Unimplemented contract test");
});

test("rejects an unknown task without creating one", () => {
  // Given no task with id="missing".
  // When setCompleted("missing", true) is called.
  // Then reject with TaskError.code="TASK_NOT_FOUND"; no task is created.
  throw new Error("Unimplemented contract test");
});
```

The comments describe observable behavior, not how a private helper works.

## 4. Check Whether Planning Is Finished

Planning is finished when the contract is concrete, not when the application works.

Before requesting final approval, verify:

- [ ] The user approved the goal, and the contract still serves it.
- [ ] Module responsibilities and dependencies are clear.
- [ ] Relevant data constraints and persistence rules are explicit.
- [ ] Public operations have concrete inputs, outputs, errors, and side effects.
- [ ] Behavioral test skeletons cover agreed success cases, meaningful failures, and relevant state
      transitions.
- [ ] Every agreed behavior maps to a contract artifact and a behavioral test; no artifact silently
      adds scope.
- [ ] Code declarations and test skeletons agree; the spec records the goal and confirmed decisions
      without duplicating contract definitions.
- [ ] No questions are stored in the spec; resolved questions on stubs have been replaced with
      agreed contract comments.
- [ ] No unresolved question would force an implementer to guess module behavior or the user's
      expected application behavior in relevant edge cases. Internal implementation mechanics remain
      unspecified.
- [ ] Available syntax, schema, or type checks have been run where applicable; failures and
      unimplemented test status are recorded honestly.
- [ ] No business logic, deployment, or unrelated refactoring has slipped into the plan.

Show the user the exact files being approved and identify any checks that cannot yet pass because
implementation is absent. Ask for explicit approval of **this contract as the implementation
baseline**, then stop and wait.

## 5. Commit the Approved Contract

After final approval:

1. Record the approval and set **Contract: approved** in the spec.
2. Inspect the diff and staging area. Include only the approved code-level contract: module
   declarations, data definitions, API contracts, and behavioral test skeletons. Never include the
   local planning anchor. Preserve pre-existing staged changes and unrelated work; do not unstage or
   rearrange them to make this commit. If an isolated contract commit cannot be made safely—for
   example, unrelated edits overlap the same files or the planning anchor is already staged—stop and
   explain the blocker. Do not use a blanket staging command.
3. Create a commit with an unmistakable message, for example:
   `contract: establish task module implementation baseline`.
4. Obtain the resulting full commit hash.
5. Add the hash to the local spec's **Contract baseline** section. Leave this update uncommitted.
   There is no follow-up metadata commit.

The code commit is the implementation baseline. The local spec points to it; the spec itself is
never part of a commit.

If Git is unavailable or committing is blocked, record **Contract: approved, uncommitted**, explain
the blocker, and stop. Do not initialize a repository, invent a hash, or claim the baseline is
committed. Do not push unless asked.

Future agents should read the approved goal and compare their changes against the baseline commit.
If implementation requires changing the contract, reopen the affected decisions with the user and
establish a new approved baseline. Do not silently rewrite the agreement to fit the implementation.

**Stop after recording the baseline. This skill does not authorize implementation.**

## Suggested Spec Structure

Keep this proportional to the work; omit sections that genuinely do not apply and explain why.

```markdown
# [Application or Feature]

## Status

- Goal: proposed | approved
- Contract: draft | approved | approved, uncommitted

## Goal

[At most roughly 500 words: purpose, users, desired outcome, scope, non-goals.]

## Decisions

[Confirmed decisions and why they were made. No open questions or duplicated contract definitions.]

## Verification

[Checks run, results, and what remains unimplemented.]

## Approval

[What the user explicitly approved and any limitations.]

## Contract Baseline

- Commit: [Full hash, added after the contract commit exists.]

[Module, data, API, and behavioral contracts live only in the code at this commit. This spec stays
local and uncommitted.]
```

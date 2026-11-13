---
name: write-tests
description: Define focused behavioral test stubs during planning and implement direct, readable unit-first tests during building.
---

# Write Tests

## When to Use

- During `plan`, to define behavioral test stubs as part of the proposed module contract.
- During `build`, before implementing a module's behavior.
- Turning approved behavioral test skeletons into real tests.
- Adding regression coverage or testing an existing function directly.

This skill follows the active phase. During planning it helps express the proposed contract in test
stubs; during building it implements tests of the approved contract. It never authorizes production
logic or unapproved contract changes.

## Select the Mode

- **Planning mode:** when used with `plan`, write only test stubs. Include the target, scenario,
  concrete preconditions, action, and expected outcomes in prepare/execute/assert comments. Add an
  explicit unimplemented failure using the project's conventions. Do not implement fixtures, mocks,
  executable target calls, or real assertions yet. Compilation is not required; do not add a test
  framework or production logic just to make stubs run.
- **Build mode:** when used with `build`, replace the stubs with executable setup, direct target
  calls, and meaningful assertions before production logic is implemented.
- **Standalone use:** executable tests of an existing agreed behavior use build mode. If the phase
  is unclear, ask whether the user wants planning stubs or executable tests before editing.

The phase's approval rules still apply. A proposed planning stub is not an approved requirement.
A placeholder failure is appropriate in planning but is not test-first evidence during building.

## Purpose

Write tests that identify exactly which function and behavior failed. Prefer small, deterministic
unit tests over tests that reach the target indirectly through other application functions.

Follow the project's language and existing test conventions. Do not add a framework or invent
requirements merely to make testing convenient.

## Workflow

```text
Read the contract and identify the target function
                         |
                         v
Split expected behavior into individual cases and branches
                         |
                         v
Structure each case with prepare / execute / assert
                         |
               Planning or building?
                  /             \
                 v               v
        Write stubs only    Implement real tests
                 |               |
                 v               v
        Review intent       Run scoped tests;
        and completeness    inspect failure/pass reasons
                 |               |
                 v               v
        Return to plan      Return to build or user
```

## 1. Identify the Target and Expected Behavior

Read the relevant code-level module contract and existing test skeletons. Use the local
`spec-<feature-name>.md` only for the goal, approved decision rationale, and baseline references;
the code defines the behavior.

Identify the function under test, its inputs, outputs, errors, side effects, and relevant branches.
Use the baseline plus explicitly user-approved amendments when working within `build`.

In planning mode, show proposed outcomes in stub comments and resolve material choices with the user
through `plan`; do not mark proposals as approved. In build mode, if an expected outcome is missing
or contradictory, report it to the coordinator or user before encoding a guess as an assertion.
Do not silently change an approved contract through test design.

## 2. Test the Function Directly

**A unit test of function `a` must call `a` directly.** In planning mode, its execute comment must
specify that direct call; in build mode, the test must actually perform it. Calling `b`, which
happens to call `a`, is not a direct test of `a`, regardless of coverage results.

A test of `b` may call `b` and verify its own promised behavior. It must not be presented as a
substitute for focused tests of `a`. Coverage of a line is not proof of a function's contract.

- Invoke the target through its intended callable interface, not an unrelated wrapper or workflow.
- Use minimal fixtures and controlled dependencies to isolate the behavior under test.
- Assert observable results or promised side effects, not incidental helper calls or implementation
  details. Verify dependency interactions only when those interactions are part of the contract.
- Do not export private helpers, change production visibility, or redesign interfaces solely to
  satisfy this rule. If a requested function cannot be tested directly through an appropriate
  existing boundary, report the limitation and agree on a test boundary before proceeding.

## 3. One Behavior or Branch per Test

Each test exercises one scenario and establishes one behavioral expectation. Separate success,
invalid input, missing data, and other distinct behavioral branches into separate tests.
“Branch” means a distinct contract outcome, not every internal `if` or loop in an implementation.
In planning mode, this rule applies to the scenario described by each stub.

**One behavior does not mean one assertion.** Several assertions may be necessary to verify one
outcome, such as the fields of a returned object or the absence of a promised side effect on failure.

Do not put unrelated inputs or successive state-changing operations into one test just to cover
multiple branches cheaply. Multiple calls are appropriate only when the behavior itself requires
those calls, such as verifying that repeating the same request is idempotent. Keep distinct
behaviors, such as the initial state transition and the repeated-request no-op, in separate tests.

Parameterized cases are appropriate when each case runs independently and tests the same behavior
for a different input. Do not hide different branches or unrelated expectations under one generic
parameterized test name.

## 4. Name and Structure Every Test Clearly

A test name must identify the target function and its intent: the scenario and expected behavior.
Use descriptive names, not numbered prefixes, tracking IDs, or vague labels such as "works" or
"handles errors".

Examples:

```text
createTask rejects a blank title without persisting
createTask returns a persisted task with generated fields
setCompleted preserves updatedAt when the requested state is unchanged
```

Each test body has three visibly commented sections, in this order:

```text
// prepare
// execute
// assert
```

Use equivalent comment syntax in other languages, such as `# prepare`, while retaining these labels.
Do not omit a section because it is small; a section may note that no preparation is needed.
Keep setup in prepare, the direct target call in execute, and verification in assert.

For exceptions, asynchronous failures, or panic assertions, the testing API may need to invoke the
call inside an assertion helper. In that case, prepare a closure or deferred operation in execute
and invoke it through the assertion helper in assert. Do not execute the target twice to force the
layout.

### Example

TypeScript is illustrative only. Use the project's own language, assertions, and test registration.
The example assumes the contract requires blank titles to be rejected without persistence.

```typescript
test("createTask rejects a blank title without persisting", async () => {
  // prepare
  const store = createInMemoryTaskStore();
  const service = createTaskService({ store });

  // execute
  const execute = () => service.createTask("   ");

  // assert
  await expect(execute()).rejects.toMatchObject({ code: "INVALID_TITLE" });
  expect(store.tasks).toHaveLength(0);
});
```

Both assertions describe the same invalid-input branch. Testing successful creation as well would
require a separate test. The in-memory store is a controlled unit-test dependency, not a claim
about real database behavior.

In planning mode, write only the corresponding stub:

```typescript
test("createTask rejects a blank title without persisting", () => {
  // prepare
  // An empty task store.

  // execute
  // Call createTask("   ") directly.

  // assert
  // Reject with code="INVALID_TITLE" and leave the task store empty.
  throw new Error("Unimplemented contract test");
});
```

Do not implement the store or assertions while planning. If runnable, this stub must fail explicitly;
if declarations prevent execution, report that limitation rather than claiming a test ran.

## 5. Prefer Unit Tests

Use unit tests for functions whose behavior can be verified with controlled inputs and dependencies.
Do not start a database, server, or full application just to test a function that can be called and
verified in isolation.

The direct-function rule applies to unit tests. Integration and end-to-end tests exercise their
intended system boundary and must not be presented as direct unit coverage of underlying functions.
In planning mode, identify that boundary in stub comments without building the infrastructure.

Use existing integration or end-to-end setups only for behavior that needs those boundaries:

- Real database behavior, such as constraints, persistence semantics, or transaction guarantees.
- Concurrent reads and writes whose correctness depends on the actual storage or runtime boundary.
- End-to-end user workflows that must be verified across the application.

Do not use integration tests as a substitute for unit tests of ordinary function logic. A focused
unit test and an infrastructure test may both be needed, but they establish different guarantees.
A fake store cannot prove real database constraints; a passing end-to-end flow cannot replace direct
coverage of a function's relevant branches.

If a necessary integration or end-to-end setup does not exist, report the gap. Do not silently add
infrastructure or label a mocked test as integration coverage.

## 6. Verify According to the Mode

### Planning Mode

Review stub names and comments against the proposed code-level contract. Check that cases are
concrete, separate relevant outcomes, and leave no material behavior to guess. Perform available
syntax checks where practical, but compilation and passing tests are not completion requirements.
Do not replace placeholders with real assertions or implement dependencies merely to verify a plan.
Hand the stubs back to `plan` for user review; do not automatically switch to build mode.

### Build Mode

During `build`, write the tests first, then run the assigned module's relevant tests before handing
back to the implementation step. A failing test must reach a meaningful assertion about missing or
incorrect behavior; an import error, syntax error, or leftover placeholder failure does not prove
that the behavior is tested.

If existing stubs prevent compilation or execution, report the blocker and the missing test evidence.
Do not claim a completed red test run, change the contract, or implement production logic under this
skill. The coordinator must resolve setup or declaration blockers before claiming test-first evidence.

When testing behavior already implemented, tests may pass immediately. Do not break working code to
manufacture a failure. Report what the tests establish and whether they passed against existing logic.

Run scoped verification without skipping or weakening existing tests. Report exact commands,
results, and limitations. The build workflow remains **tests first, logic second, verification third**.

## Completion Check

- [ ] The mode was established before editing: planning stubs or executable build tests.
- [ ] Planning proposals remain visibly proposed; build expectations match the approved contract.
- [ ] Each unit test directly invokes its target function, or specifies that call in a planning stub.
- [ ] Names identify the function, scenario, and intended outcome without tracking IDs.
- [ ] Each test covers one behavior or branch; multiple assertions support that same outcome.
- [ ] Every test has prepare, execute, and assert comment sections.
- [ ] Unit tests are preferred; infrastructure tests are restricted to behavior needing real boundaries.
- [ ] Planning mode: only stubs were written, with concrete comments and explicit placeholder failures;
      no fixture implementations, target invocations, or real assertions were added.
- [ ] Build mode: placeholder failures were replaced with meaningful assertions and scoped tests ran
      where possible before production implementation.
- [ ] Actual results and blockers were reported honestly; placeholder failures were not claimed as
      meaningful red-test evidence.
- [ ] No production logic, unapproved contract changes, weakened tests, or new infrastructure slipped in.

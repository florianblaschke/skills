---
name: build
description: Delegate implementation to a lower-cost or user-selected model while enforcing approved module contracts and optional automatic commits.
---

# Build

## When to Use

- An application or feature has an approved code-level contract ready for implementation.
- The user wants the current agent to oversee implementation by another model.
- Work is resuming against an existing contract baseline and approved amendments.

Do not use this skill to invent missing module behavior or redesign an agreed contract. Resolve
those choices with the user before implementing the affected behavior.

## Purpose

Implement the agreed contract, not a convenient approximation of it. Use a flat queue of bounded,
independently verifiable tasks, not recursive build/review cycles. With `/auto` enabled, finish,
review, and commit each ready task before waiting for the whole feature to be complete.

The current agent is the **coordinator**: it maintains the overview, assigns bounded implementation
work, reviews results, and manages user decisions. A delegated **implementer** writes and verifies
the implementation. Only the coordinator launches agents; the implementer must not delegate.

The contract is binding. Neither model may silently change it, weaken its behavioral tests, or add
incompatible behavior to make implementation easier. This applies to internal module contracts as
well as externally public APIs.

Replacing stub bodies, implementing private helpers, and turning placeholder tests into real
assertions are implementation changes, not contract changes, provided they preserve agreed outcomes.
Changes to module responsibilities, schemas, interfaces, guarantees, or expected outcomes require
explicit user approval.

## Workflow

```text
Establish baseline, approved amendments, model, and auto setting
                              |
                 Define flat tasks and commit boundaries
                              |
                 Assign independent ready tasks in parallel
                              |
                 Implement tests -> run RED -> implement behavior -> run GREEN
                              |
                 Coordinator contract + code-quality review
                              |
                 Auto on: commit each ready task now
                 Auto off: mark verified/uncommitted
                              |
                 Release dependent tasks; repeat queue
                              |
                 Combined feature checks + full QA -> handback

Contract departure at any point: stop affected work -> explicit user approval
-> record exact amendment -> resume affected tasks. No recursive build calls.
```

## 1. Establish the Implementation Baseline

Before delegation:

- Read `spec-<feature-name>.md` for the approved goal, contract commit hash, and confirmed decisions.
- Inspect the actual code-level contract at that commit, including module declarations, schemas,
  API semantics, and behavioral test skeletons.
- Identify approved contract amendments made since that baseline. The effective contract is the
  baseline plus explicitly user-approved amendments, not whatever happens to be in the working tree.
- Inspect the current work and repository state. Preserve unrelated files, user edits, and staged
  changes. Do not treat unapproved contract drift as permission to implement against it.
- Inspect the latest applicable `challenge-<feature-name>.md` review, if present. If it contains
  blocking findings, deny implementation for this feature until they are resolved and the revised contract
  is reviewed without blockers. User willingness to proceed does not turn an unresolved blocker into
  a conforming design. An incomplete or stale review must not be described as clearance.
- Identify relevant verification commands and existing test conventions.

If the baseline commit or approval history is missing or ambiguous, ask the user to establish it
before affected implementation begins. Do not fabricate a baseline or infer approval from code edits.

### Model and Auto Setting

Read and enforce the [shared model-selection policy](../use-skills/SKILL.md#shared-model-selection-policy).
The coordinator must use the high-capability tier. For each worker, use an eligible user-selected
model or the latest stable available generation within a suitable lower-cost family. Assess the
actual task's complexity, tools, context needs, and known model limitations; cheap alone is not
sufficient. Complete the catalog/provider preflight, announce resolved IDs and selection reasons,
and verify actual launch metadata. Do not silently use an older harness default or infer cost
or capability from a name. Ask when suitability or availability cannot be established.

Record the controller and per-task worker model IDs in `tasks-<feature-name>.md`. When a worker
reports capability limits or repeatedly fails, reassess and escalate to a verified high-capability
worker rather than recursively delegating or repeating the same failing approach. Announce and
record the change; preserve ownership, test-first, and contract gates.

Use the user's harness and its documented delegation mechanism. If delegation is unavailable,
report the blocker rather than silently doing the implementation in the coordinator's context.

**Auto controls committing, not contract authority.**

- **Auto on** (including `/auto`): the implementer creates a verified, contract-conforming commit
  for each completed task after coordinator authorization; do not batch ready tasks until feature completion.
- **Auto off:** leave all implementation work uncommitted, including at completion.
- If the user has not specified auto, default to **off** and state that choice.

Use explicit user model and auto selections without asking for confirmation again. Otherwise
announce the available, suitable lower-cost model and the auto-off default before starting; ask only
when model availability or suitability is unclear. Never enable auto merely because a previous
skill created a contract commit. Neither setting authorizes pushing to a remote.

## 2. Divide Work by Contract Modules

Use the modules established by `plan` as the implementation boundaries. For example, three modules
may be assigned to three workers, each responsible for one module's implementation and tests.
Do not invent new module boundaries merely to distribute work.

Run independent modules in parallel when safe; implement dependent modules in dependency order or
against their agreed interfaces. Each worker is an implementer under the same coordinator and model
selection rules. Shared files and cross-module integration need an explicit owner; do not assign
overlapping writes. Split a large module into coherent slices when necessary.

Before launching workers, create `tasks-<feature-name>.md` beside the local planning anchor and show
the user its path. Reuse it on resume instead of creating another tracker; preserve unrelated files
if the default path is occupied. The coordinator owns and updates this local, uncommitted progress
file. It tracks work, not a second contract: reference source/test symbols rather than copying their
behavioral definitions.

| Task / observable outcome | Owner / owned files | Dependencies | Tests and verification | Commit boundary | Status / commit |
| --- | --- | --- | --- | --- | --- |

Below the table, keep short per-task evidence notes: red command and actual failure, green command
and result, blockers, and commit hash when applicable. Update the file at assignment, red/green
checkpoints, review, and commit or blocker transitions. Workers report evidence to the coordinator;
they do not concurrently edit this shared tracker. On resume, reconcile it with the working tree,
actual commits, and check results before continuing; a recorded status is not proof.

Populate it with the actual tasks, not just this template. A task delivers one meaningful contract
behavior or a cohesive module with its tests, not each helper, branch, or review finding. A small
feature may be one task. A large feature must not be one unbounded assignment. If a task cannot be
verified or committed independently, combine it with its minimum prerequisite or split it differently
before assigning it; do not discover its boundaries through recursive delegation.

Assign separate workers to independent ready tasks and run them concurrently when the harness and
file ownership allow it. State concrete dependency, shared-file, or harness reasons for serial work.
Do not merely mention parallelism and then give one worker the whole feature. The coordinator owns
the queue and updates statuses: ready, running, review, committed (or verified/uncommitted with auto
off), blocked. Workers return blocked work or findings to this queue; they do not invoke `build`,
launch reviewers, or create nested task trees. Reassign fixes to the owning task, not a new build.

Before each assignment, define the owned files, module behavior, relevant tests, and scoped
verification commands. A worker checks its assigned module and files, not unrelated unfinished
modules. The coordinator owns cross-module integration verification before declaring the feature
complete.

Give each implementer:

- Its role, selected scope, and a prohibition on further delegation.
- The approved goal and paths to the relevant contract files.
- The exact baseline commit and approved amendments, including their user-confirmed rationale.
- The auto setting and all contract-change and commit rules in this skill.
- Existing code and test conventions, verification commands, and known planning-stage blockers.
- A hard prohibition on writing production behavior before implementing and running its tests to
  a meaningful red result, as specified in section 4. Tests and implementation must be separate
  edit/run steps, not a combined patch followed by a first green run.
- A requirement to report changed files, exact red/green commands and observed results in order,
  verification results, contract concerns, and a proposed commit diff when auto is on.
- A prohibition on committing until the coordinator has reviewed and authorized the exact diff.

The implementer may inspect supporting project code as needed. Do not ask it to implement unrelated
features or refactor adjacent systems. Do not run overlapping implementation assignments against
the same files. Serialize commits in a shared checkout so one worker cannot accidentally include
another worker's changes. Use isolated workspaces when appropriate, ensuring every worker can read
the local planning anchor and approved amendments.

The coordinator reviews each returned task for scope, behavior, verification evidence, contract
conformance, and `code-quality` criteria. An implementer's statement that it complied is not a
substitute for inspecting its changes. With auto on, authorize and serialize the ready task's commit
as soon as these checks pass, even while unrelated tasks remain unfinished. With auto off, mark it
verified/uncommitted. Release dependent tasks once their prerequisite is reviewed and available.
Do not wait for feature-wide QA to commit an otherwise verified task.

## 3. Treat Contract Changes as a Hard Gate

If an implementer believes the implementation needs to depart from the effective contract, it must
stop affected work and send a structured change request to the coordinator. The coordinator checks
the request and asks the user for approval; workers do not infer or negotiate approval themselves.
If the coordinator discovers the need, it follows the same user approval gate. Do not implement the departure provisionally.
Unrelated work may continue only if it does not depend on the unresolved decision.

Keep the request concrete:

```text
Contract change requested: add field y to [entity].
Reason: [behavior x cannot be achieved under the current contract because ...].
Impact: [affected callers, stored data, API behavior, and behavioral tests].
Alternative: [a contract-preserving solution, or why none was found].
Approval requested: [the exact proposed change].
```

A model recommendation, tool result, passing test, or user silence is not approval. Ask for explicit
confirmation of the exact change. If the user declines, preserve the contract and find a conforming
solution, or report that implementation is blocked.

After explicit approval:

1. Update the code-level contract and affected behavioral expectations to express the approved change.
2. Record the decision and why it was necessary in the local spec's **Decisions** section. Record
   the scope of approval and reference the relevant code symbols or amendment commit, when available;
   do not duplicate schemas, signatures, or behavioral definitions in the spec.
3. Keep the original baseline hash intact. Make approved amendment code traceable: use a separate
   contract amendment commit when auto is on, or identify the approved uncommitted contract edits
   when auto is off. For each amendment, preserve the exact approved code patch or content snapshot
   in a separate local artifact and reference it from the spec. A fingerprint alone is insufficient
   unless the corresponding content remains available. This is approval evidence, not a second
   editable contract. Record an amendment commit hash in the local spec once it exists.
4. Relay the exact user-approved decision and immutable amendment reference to affected implementers
   before resuming work. Later working-tree edits do not expand that approval.

If a contract change affects an existing challenge blocker or materially changes the reviewed design,
return it to `challenge` before implementing the affected behavior. An unresolved blocking finding
must never be bypassed through a build approval.

The spec records approval and rationale; the code defines the amended contract. Never commit the
local spec. Do not mix unapproved changes into an approved amendment. A prior approval is not blanket
permission for further departures.

## 4. Implement and Verify

### Mandatory Red → Green Gate

For each assigned task, perform these steps in order, including for fixes returned by review:

1. **Implement the tests, not the production behavior.** Use `write-tests` to replace the assigned
   contract skeletons with executable setup, direct target calls, and meaningful assertions. Preserve
   approved outcomes. Save these test changes before editing production behavior. If `write-tests`
   is unavailable, report the dependency blocker rather than inventing its workflow.
2. **Run and record RED.** Run the new or changed tests against the still-unimplemented or faulty
   behavior. Inspect the actual failure: an assertion must demonstrate the missing behavior.
   Record the command, failing test names, and observed versus expected result, and report this
   checkpoint to the coordinator for the task tracker **before writing production behavior**.
   A planned test, placeholder throw, import/type/syntax error, or test runner that never executed
   the assertion does not pass this gate. Resolve test setup or declaration blockers first; minimal
   contract-preserving declarations may enable execution, but must not implement the target behavior.
   If meaningful red cannot be obtained, stop affected implementation and report the blocker.
3. **Implement the production behavior only after RED.** Write the minimum logic needed to satisfy
   those tests and the approved contract. Do not write tests and production logic in the same initial
   patch, implement first and backfill tests, or claim a first green run proves test-first order.
4. **Run and record GREEN.** Rerun the same tests and required task regression checks. Record the
   commands and actual passing results. Run applicable scoped syntax, type, schema, lint, and build
   checks. Fix implementation without weakening expectations; rerun affected checks after cleanup.

The coordinator must inspect red/green evidence and execution order before marking a task verified
or authorizing its commit. A final passing suite alone is insufficient. Missing evidence blocks the
task's review gate; do not invent a historical red run or label tests written afterward test-first.
This is an execution checkpoint within the task, not a new reviewer or recursive agent gate.

**Already-implemented behavior:** new tests may pass immediately against unchanged existing logic.
Record that result and the existing implementation that satisfies the contract; do not break working
code to manufacture red. This exception permits coverage of existing behavior, not writing new
behavior before its tests. Any remaining missing behavior must still pass the red gate.

Do not delete, skip, or relax agreed expectations to obtain a passing result. Expand tests for
relevant edge cases without inventing product requirements. Existing working behavior need not be
broken to demonstrate a failing test; report when a new test already passes against existing logic.

Define verification scope before implementation, including necessary dependencies. A module's
shared dependency failure is not irrelevant merely because it is outside an owned file. Distinguish
untouched, unimplemented skeletons in other modules from regressions and failures in the assigned
module. Never suppress tests or reduce check scope after a failure just to get green.

Record exactly what ran and what failed or could not run. Scoped success is not full-suite success.
The coordinator runs relevant integration checks as modules come together and final feature-wide
verification before declaring completion.

If a check fails unexpectedly, investigate and fix the implementation. If the failure reveals a
contract problem, use the approval gate above rather than silently changing the agreement.

### Implementation Style

Use `code-quality` in implementation mode for the shared rules on coherent units, control flow,
shared domain rules, test quality, duplication, and performance. It is a checklist within this task,
not a delegated or recursive gate.

### Mandatory Contract Conformance Check

Before **every commit**, including contract amendment commits, the implementer must:

1. Read the relevant contract from the referenced baseline commit, not only the current files.
2. Read the documented user-approved decisions and inspect the corresponding code amendments.
3. Compare the proposed diff and resulting behavior with that effective contract, including field
   constraints, API inputs and outputs, errors, side effects, and relevant edge cases.
4. Verify that behavioral tests still express the agreed expectations and have not been weakened.
5. Rework any unauthorized difference and rerun affected checks before proceeding.
6. Send the exact proposed diff, baseline/amendment references, conformance assessment, and scoped
   verification evidence to the coordinator. The coordinator reviews that task using `code-quality`
   and checks correctness, relevant security boundaries, and contract conformance. Wait for its
   explicit authorization before committing that exact diff. Full `qa` is not a per-task prerequisite
   unless the user explicitly requests it.

The coordinator independently inspects the proposed changes and evidence before authorizing the
commit. Its authorization confirms conformance; it cannot approve a contract change on the user's
behalf. If the proposed diff changes after review, repeat the check and obtain fresh authorization.

Unapproved contract differences must never be committed. A need for a contract change requires user
approval, not an exception to this check. For an approved amendment commit, compare against the
baseline plus that exact approval; approval does not excuse other differences.

With auto off, perform the same conformance check before handing work back, even though there is
no commit. Contract enforcement is identical in both modes.

### Task Review and Feature QA

Every task requires scoped tests, applicable checks, and coordinator review using `code-quality`
and the contract conformance check above. Review at the task boundary, not after every function or
file edit. Fix blocking findings within the owning task, rerun affected checks, and inspect the fix;
do not launch a new build or three-agent QA cycle for each correction. Non-blocking cleanup does
not hold a verified task's commit hostage.

Use full `qa` for the combined implemented feature and cross-module interactions before completed
user handback. The coordinator alone launches its two specialists and independent proof verifier.
An explicit user request may also require full QA for selected intermediate milestones or commits;
otherwise task commits use the scoped coordinator gate, not full QA.

A failing or incomplete feature QA blocks completion, not unrelated task commits that pass their
own gates. Freeze affected/dependent work on a blocker; independent tasks may continue. Put fixes
back into the flat queue, review and commit them normally, then re-review the affected QA scope.
Do not describe an earlier task commit as QA-cleared unless it actually received that review.
Unfinished work may be returned with an honest blocker report, never as complete or QA-cleared.

## 5. Commit or Hand Back

### Auto On

The implementer makes small, meaningful commits representing coherent, verified changes. Commit
messages should read like useful human-written project history: describe the behavior or reason
for the change, not the agent's process.

Examples:

```text
Add task creation with server-generated timestamps
Preserve completion timestamps when requests are retried
Reject blank task titles before persistence
```

Do not reference `.md` documents, agent assignments, or numbered plan steps in commit messages.
Do not add claims of verification that did not occur.

Inspect the staged diff before committing. Include only the intended slice, never local planning
anchors, task trackers, or challenge reports. Preserve unrelated staged work; if a clean isolated commit cannot be made
safely, stop and ask rather than unstaging, rearranging, or including unrelated changes. Do not use
blanket staging commands or bypass checks.

Implementation commits require passing checks for the agreed module or slice scope. Untouched
planning placeholders elsewhere may still fail; disclose that status rather than claiming the whole
project passes. Contract-only amendment commits instead use planning-stage verification: explicit
user approval, exact contract conformance, and applicable declaration/schema checks. Expected new
skeleton failures or documented planning-stage compilation blockers do not require implementing
logic just to commit the amendment. Keep such amendments separate from implementation changes.

Commit only after the coordinator's pre-commit authorization of the exact diff. The coordinator
then confirms that the resulting commit contains only those authorized changes. If an unauthorized
deviation is discovered, stop dependent work, report it, and arrange a conforming correction.
Do not silently accept drift or rewrite commit history to hide it.

### Auto Off

Leave all work uncommitted and report changed files and verification results. Do not create a final
"save point" commit or commit approved contract amendments. Preserve the user's existing staging
state; do not stage implementation changes merely to prepare a future commit.

### Completion

Summarize delivered behavior, verification results, remaining blockers, approved contract amendments,
and commit hashes if auto was on. Include the applicable QA verdict, reviewed scope, and limitations.
Do not claim completion if required behavior remains unimplemented
or required checks have failed or could not run; state the limitation clearly.

## Completion Check

- [ ] Model preflight verified the high-capability coordinator and suitable current/pinned workers;
      resolved IDs were announced, actual launch IDs checked, and any escalations recorded.
- [ ] The baseline contract and exact, explicitly approved amendments governed every module or slice.
- [ ] No feature implementation proceeded while applicable challenge blockers remained unresolved.
- [ ] `tasks-<feature-name>.md` tracked the flat task table, progress, red/green evidence, blockers,
      and commits; it stayed local and uncommitted and was reconciled on resume.
- [ ] Independent ready tasks ran in parallel or a concrete constraint was stated.
- [ ] Module ownership and scoped verification were defined before work; integration was verified.
- [ ] Workers implemented and ran tests to meaningful red before writing production behavior,
      then reran them to green; the coordinator inspected ordered evidence, not just a final pass.
- [ ] Any immediately green tests covered identified, unchanged existing behavior, not code written
      before its tests; missing red evidence or blocked execution was not silently waived.
- [ ] Every contract change received explicit user confirmation before affected implementation.
- [ ] Approved changes and their rationale were recorded in the local spec without committing it.
- [ ] Behavioral tests preserve the contract and relevant verification results are reported honestly.
- [ ] Every auto commit passed the baseline-and-amendment conformance check and coordinator review
      before being created.
- [ ] Each completed task passed scoped checks, `code-quality`, and coordinator review; with auto
      on, ready tasks were committed without waiting for unrelated feature work or final QA.
- [ ] Completed handback had full QA clearance covering the combined feature and its interactions;
      any explicitly requested intermediate QA gates also passed.
- [ ] Auto-off work remains uncommitted; unrelated work and staging were preserved.
- [ ] The coordinator inspected the implementation results rather than trusting only a summary.

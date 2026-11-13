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

Implement the agreed contract, not a convenient approximation of it.

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
Identify contract baseline, approved amendments, model, and auto setting
                              |
                              v
                  Delegate a bounded implementation slice
                              |
                 Contract change needed?
                    /                 \
                 yes                   no
                  |                     |
                  v                     v
         Stop affected work      Implement and verify
         Explain to user                |
                  |                     v
         Explicit approval?      Compare against contract
           /          \           and approved amendments
          no          yes               |
          |            |          Conforming?
          v            v          /          \
      Keep contract  Record      no          yes
      unchanged      amendment    |            |
                     in code      v            v
                     and spec   Rework     Auto on?
                         |                 /      \
                         v               yes      no
                    Reassign              |        |
                                          v        v
                                  Coordinator review
                                          |        |
                                        Commit   Leave uncommitted
                                             \    /
                                              v  v
                                      Review next slice or finish
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

Use the implementation model explicitly selected by the user. Otherwise prefer an available
lower-cost model capable of the task. Do not invent model identifiers or assume pricing. If no
suitable lower-cost option can be identified, ask the user which available model to use.

Use the user's harness and its documented delegation mechanism. If delegation is unavailable,
report the blocker rather than silently doing the implementation in the coordinator's context.

**Auto controls committing, not contract authority.**

- **Auto on:** the implementer may create verified, contract-conforming implementation commits.
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
- A requirement to report changed files, test-first evidence, verification results, contract
  concerns, and a proposed commit diff when auto is on.
- A prohibition on committing until the coordinator has reviewed and authorized the exact diff.

The implementer may inspect supporting project code as needed. Do not ask it to implement unrelated
features or refactor adjacent systems. Do not run overlapping implementation assignments against
the same files. Serialize commits in a shared checkout so one worker cannot accidentally include
another worker's changes. Use isolated workspaces when appropriate, ensuring every worker can read
the local planning anchor and approved amendments.

The coordinator reviews each returned slice for scope, behavior, verification evidence, and contract
conformance before assigning dependent work. An implementer's statement that it complied is not a
substitute for inspecting its changes.

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

For each assigned module or slice, follow this order:

1. **Write tests first.** Use `write-tests` to turn the contract's behavioral skeletons into real tests.
   Preserve their expected outcomes and confirm they fail because the behavior is not implemented,
   not merely because of broken test setup or syntax. If `write-tests` is unavailable, report that
   dependency rather than inventing its workflow or claiming it was used.
2. **Implement the logic.** Write only the behavior needed to satisfy the approved module contract.
3. **Verify.** Run the assigned module's relevant tests and scoped syntax, type, schema, lint, and
   build checks. Fix the implementation and repeat verification until those required checks pass.

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

### Mandatory Contract Conformance Check

Before **every commit**, including contract amendment commits, the implementer must:

1. Read the relevant contract from the referenced baseline commit, not only the current files.
2. Read the documented user-approved decisions and inspect the corresponding code amendments.
3. Compare the proposed diff and resulting behavior with that effective contract, including field
   constraints, API inputs and outputs, errors, side effects, and relevant edge cases.
4. Verify that behavioral tests still express the agreed expectations and have not been weakened.
5. Rework any unauthorized difference and rerun affected checks before proceeding.
6. Send the exact proposed diff, baseline/amendment references, conformance assessment, and scoped
   verification evidence to the coordinator. The coordinator invokes `qa` on that candidate using
   two independent specialists and a separate proof verifier on more capable models. Wait for a
   QA pass and the coordinator's explicit
   authorization before committing that exact diff.

The coordinator independently inspects the proposed changes and evidence before authorizing the
commit. Its authorization confirms conformance; it cannot approve a contract change on the user's
behalf. If the proposed diff changes after review, repeat the check and obtain fresh authorization.

Unapproved contract differences must never be committed. A need for a contract change requires user
approval, not an exception to this check. For an approved amendment commit, compare against the
baseline plus that exact approval; approval does not excuse other differences.

With auto off, perform the same conformance check before handing work back, even though there is
no commit. Contract enforcement is identical in both modes.

### Mandatory QA Gate

Use `qa` before every commit and before completed user handback. The coordinator launches the QA
specialists and verifier; implementation workers must not launch them themselves. Default QA scope is the implemented
module or slice, including its tests and relevant dependencies. An explicit caller-selected scope
must still cover all changes being cleared for commit or completion.

QA requires three fresh-context agents on available models more capable than the implementer:
correctness/security and quality/performance specialists, followed by an independent proof verifier.
Every reported finding must be independently proved; unverified suspicions are not findings.
A failing or incomplete QA blocks the commit and completion claim. Fix findings and re-run QA on the
final candidate; do not treat a report on older code as clearance. The coordinator's contract review
remains required in addition to QA. Reuse a pass only when the reviewed content and relevant
interactions are unchanged, as defined by `qa`.

At final handback, ensure the combined implemented scope and cross-module interactions are covered,
not merely each module in isolation. Unfinished work may be returned with an honest blocker report,
but not described as QA-cleared or complete.

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
or challenge reports. Preserve unrelated staged work; if a clean isolated commit cannot be made
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

- [ ] Implementation was delegated to the user-selected or identified lower-cost model.
- [ ] The baseline contract and exact, explicitly approved amendments governed every module or slice.
- [ ] No feature implementation proceeded while applicable challenge blockers remained unresolved.
- [ ] Module ownership and scoped verification were defined before work; integration was verified.
- [ ] Workers wrote tests first, implemented logic second, and verified third.
- [ ] Every contract change received explicit user confirmation before affected implementation.
- [ ] Approved changes and their rationale were recorded in the local spec without committing it.
- [ ] Behavioral tests preserve the contract and relevant verification results are reported honestly.
- [ ] Every auto commit passed the baseline-and-amendment conformance check and coordinator review
      before being created.
- [ ] Every commit and completed handback had a QA pass for the exact changes and relevant scope.
- [ ] Auto-off work remains uncommitted; unrelated work and staging were preserved.
- [ ] The coordinator inspected the implementation results rather than trusting only a summary.

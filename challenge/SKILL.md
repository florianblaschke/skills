---
name: challenge
description: Challenge planned module contracts with two independent reviewers and a third agent to reconcile findings before implementation.
---

# Challenge

## When to Use

- After `plan` has established a concrete module contract, before implementation begins.
- After a significant contract revision needs independent scrutiny.
- The user wants to test whether the planned design works, is secure, and meets the application's needs.

This is a design review, not an implementation review. Do not reject a plan merely because its stubs
do not compile or its unimplemented test skeletons fail.

## Purpose

Challenge the design rather than defend it. Find concrete reasons the proposed application might
not work as intended, expose users or data to harm, or force consumers to work around a poor contract.

The code is the source of truth for module responsibilities, data, APIs, and behavioral tests.
Read `spec-<feature-name>.md` for the approved goal and decision rationale, not a second contract.
Do not use the README to establish current behavior; it may be outdated.

A good contract:

- **Works:** its operations and state transitions can deliver the promised behavior together.
- **Is secure:** relevant trust boundaries, access rules, and failure cases are addressed.
- **Fits its consumers:** inputs, outputs, errors, and module boundaries suit the actual callers.
- **Covers the need:** the approved goal and relevant edge cases have unambiguous outcomes.
- **Is extensible:** foreseeable changes suggested by the goal can be accommodated without
  unnecessary breaking changes. This does not justify speculative features or generic frameworks.
- **Is valid:** declarations, schemas, semantics, and behavioral tests are mutually consistent and
  compatible with relevant platform or dependency constraints.

## Workflow

```text
Identify the goal and exact contract under review
                        |
                        v
Launch two independent fresh-context reviewers
             /                         \
            v                           v
Feasibility and security       Contract quality and coverage
             \                         /
              v                       v
       Third fresh-context agent reconciles claims
                        |
                        v
Write a short report with problems and proposed solutions
                        |
                        v
Present report -> Stop and wait for the user's decision
```

## 1. Establish the Review Target

Find the feature's local planning anchor, approved goal, contract files, and baseline commit if one
exists. Identify any current edits so reviewers do not accidentally review only an older baseline.
If the feature or revision is ambiguous, clarify it with the user before launching reviewers.

Review the same fixed revision in both agents. Do not modify the target during review. If it changes,
identify affected claims and re-review the changed contract before presenting them as current.

If the goal or code-level contract is missing, return to `plan` instead of inventing it. Lack of a
commit alone is not a blocker if the review target is otherwise clear.

## 2. Launch Two Fresh-Context Reviewers

Use the user's available agent harness, such as pi, Codex, Claude, or another supported runner.
Use its documented agent or session mechanism; do not assume a particular CLI or invent flags.

**Both reviewers must start in new, independent contexts.** Do not continue the planning session,
fork its conversation, or pass its transcript to either reviewer. The design author's existing
context is not an independent review. The current agent may coordinate, but must not substitute
its own assessment for either fresh reviewer.

Use exactly three agents: two reviewers with different focuses, then one reconciler. The current
agent may coordinate using its existing planning context, but all three agents must start without
inherited conversation history. Only the coordinator launches agents. Reviewers and the reconciler
must not launch or delegate to other agents, even when reading this skill's orchestration steps.

Give each reviewer only:

- This skill and its assigned review focus.
- The repository location and exact review target, including relevant uncommitted changes.
- The path to the local planning anchor and relevant contract files.
- Necessary environment constraints and instructions for returning its findings.

Allow reviewers to inspect supporting source, consumers, tests, and authoritative technical
references as needed. Do not preload the author's defense of the design or conclusions from the
other reviewer. Confirmed decision rationale in the local spec is legitimate review evidence, not
proof that a choice is correct.

Run reviewers in parallel when supported. Otherwise run two separate fresh sessions sequentially,
without sharing their findings until both finish. Both must have access to the actual review files,
including the uncommitted local planning anchor; a clean checkout alone may omit necessary inputs.

Reviewers are read-only with respect to the project. Return findings through harness output or
separate designated report files; do not edit contract files, change tests, install dependencies,
or commit anything. Any reproduction must be non-destructive and isolated from project state.

If the required fresh contexts cannot be launched, report the limitation and stop. Do not claim
that an in-session self-review or coordinator reconciliation satisfies this skill.

### Reviewer A: Feasibility and Security

Ask: **Does this really work, including when something goes wrong?**

- Can the data model and operations enforce the promised invariants?
- Are state transitions, persistence, and cross-module effects achievable together?
- Where relevant, what happens under retries, concurrency, partial failure, or interrupted work?
- Can untrusted inputs, missing authorization, or unclear ownership cross a trust boundary?
- Could the design expose sensitive data or permit unintended actions?
- Do promised guarantees depend on unavailable or incompatible platform capabilities?

Focus on broken guarantees and realistic abuse or failure scenarios, not implementation mechanics
such as choosing a loop over `map`.

### Reviewer B: Contract Quality and Coverage

Ask: **Is this the right, valid contract for the application's needs and its consumers?**

- Does every part of the approved goal have a corresponding module behavior?
- Do inputs, outputs, error definitions, and delivery mechanisms fit real callers and APIs?
- Are existing signatures, error delivery, data representations, and consumer expectations preserved
  where compatibility is required? Are intentional breaking changes explicit and agreed?
- Are responsibilities and dependencies coherent across internal and external module boundaries?
- Are declarations, schemas, comments, and behavioral test expectations consistent?
- Are relevant edge-case outcomes clear enough for an implementer to avoid guessing?
- Can foreseeable changes be supported without avoidable contract breakage?
- Does the contract add unnecessary scope, expose internal details, or require consumer workarounds?

Focus on the agreement's shape and completeness rather than redoing Reviewer A's security analysis.
The focus split is not a reason to ignore an obvious serious issue; flag it for deduplication.

## 3. Reconcile Claims in a Third Fresh Context

Each reviewer must ground claims in specific contract locations and a concrete scenario. No finding
quota: an empty result is preferable to invented objections.

After both reviewers finish, launch a third, fresh-context agent as the reconciler. Give it this
skill, its reconciler role, both reviewers' unedited findings, and the same review target and file
access. Do not supply the coordinator's preferred conclusions or the planning conversation.
The reconciler must independently inspect the evidence, not decide by reviewer agreement or vote.
The same read-only restrictions apply; it must not delegate or modify the contract.

The reconciler:

- Checks each claim against the actual contract and approved goal.
- Validates technical assumptions against supporting code or authoritative references as needed.
- Merges duplicate claims without losing distinct consequences or proposed solutions.
- Marks each claim as validated, rejected, or unresolved. For rejected claims, records concrete
  counterevidence; for unresolved claims, identifies the missing evidence.
- Checks whether each proposed solution addresses the problem without introducing unnecessary scope
  or contradicting another module contract.
- Returns the consolidated claims and a short disposition for every original claim.

Do not describe hypothetical design risks as reproduced runtime exploits. The coordinator presents
the reconciler's assessment rather than silently adding, removing, or overruling findings. If the
assessment misses material evidence, return that evidence to the reconciler for reconsideration.

A missing implementation is not a finding. A missing or contradictory behavioral guarantee may be.
Likewise, a failing skeleton is expected; a skeleton specifying the wrong outcome is reviewable.

If evidence is insufficient, ask a focused question in conversation or record the limitation in the
review report. Do not edit stubs during challenge; any agreed contract edits belong to a later
planning step. Do not turn uncertainty into a confirmed defect or put open questions in the spec.
Keep unresolved suspicions separate from validated claims.

### Claim Format

Keep each claim short, normally two to four sentences plus its location:

```text
[Blocking | Non-blocking] Short problem statement
Location: path/to/contract.ext — symbol or relevant lines
Problem: Concrete scenario and the promised behavior that fails or is unspecified.
Solution: Smallest contract change that addresses the problem.
```

**Blocking** means implementation would require guessing a material behavior, violate the agreed
goal, depend on an impossible guarantee, or leave a relevant security boundary undefined or unsafe.
**Non-blocking** means the contract is implementable but has a concrete compatibility, usability,
or extensibility weakness worth addressing. Do not report taste or speculative future needs.

Example:

```text
[Blocking] Retrying a timed-out completion request can undo the user's action
Location: src/tasks/contracts.ts — toggleCompleted
Problem: A retry toggles the task back to incomplete if the first request succeeded before
its response was lost. The caller cannot safely recover from an unknown outcome.
Solution: Accept the desired completion state explicitly and define repeated requests as no-ops.
```

Only make this claim when retry-safe completion is required by the goal or the surrounding caller
behavior. An operation named "toggle" is not inherently a defect.

## 4. Present the Challenge

Write the consolidated report to `challenge-<feature-name>.md`, using the planning anchor's feature
name. Keep it local and uncommitted. The report is review evidence, not a second contract.

For subsequent reviews, append a separate dated review section rather than overwriting earlier
results. Give each section a unique review label and identify its exact baseline and reviewed
uncommitted changes using a saved diff or content fingerprint. Do not attribute an earlier review
to later changes. If the target changes before reconciliation finishes, mark that run stale and
restart the affected review process against one fixed target.

Include:

- The baseline commit, if any, and any reviewed uncommitted changes.
- Each reviewer's focus and whether both reviews and the fresh-context reconciliation completed.
- The reconciler's disposition of the original claims, with counterevidence for rejected claims.
- Validated claims, blockers first, with a proposed solution for each.
- Any unresolved evidence limitations, separate from confirmed claims.
- A conclusion: **changes required**, **no blocking issues found**, or **incomplete review**.

Keep the report proportional to the findings. If there are no validated claims, say so plainly;
that is not proof that the design is correct or secure.

Show the user the report and let them decide which changes to accept. Do not silently apply fixes,
rewrite the contract, or mark your proposed solutions as approved decisions.

**Stop after presenting the challenge and wait for the user's decision.** Do not automatically
invoke `plan`, apply proposed solutions, or launch another review cycle.

If the user subsequently authorizes revisions, return to `plan`: update the code contract and
behavioral tests, record significant decision rationale in the local spec, obtain approval, and
establish a new contract baseline. Materially changed areas need a new challenge with two fresh
reviewers and a fresh reconciler before being treated as reviewed.

This skill does not authorize contract edits, implementation, or commits.

## Completion Check

- [ ] Both reviewers started in independent fresh contexts and reviewed the same target.
- [ ] A third fresh-context agent reconciled both reviews against that target; no agent delegated.
- [ ] Feasibility/security and contract quality/coverage were both examined.
- [ ] Claims are grounded in the code contract and approved goal, not imagined requirements.
- [ ] Each validated claim clearly states a problem, consequence, and proposed solution.
- [ ] Duplicate claims were merged; rejected claims have counterevidence and unresolved claims remain
      explicit.
- [ ] Earlier review results were preserved and the current reviewed revision is identifiable.
- [ ] No contract edits, weakened tests, implementation, or commits were made.
- [ ] The user received the local report and retains the decision on proposed changes.

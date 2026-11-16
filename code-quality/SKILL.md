---
name: code-quality
description: Apply or review focused code quality, duplication, testability, and performance rules without launching agents or recursive review gates.
---

# Code Quality

## When to Use

- During `build`, while implementing and reviewing a bounded task before commit or handback.
- Inside `qa`, as the quality/performance specialist's review criteria.
- For a focused standalone code-quality review.

This skill supplies shared criteria, not another orchestration layer. It never launches agents,
invokes `build` or `qa`, requires another review of its own results, or authorizes contract changes
or commits. The calling workflow owns scope, permissions, verification, and the final gate.

## Workflow

1. Read the selected diff and relevant callers, dependencies, tests, and approved contract.
2. Apply the criteria below only where relevant to the selected behavior.
3. In implementation mode, make focused, contract-preserving changes within the assigned files and
   rerun affected checks. In review mode, do not edit project files; return evidenced findings.
4. Stop and return the result to the caller. Do not subdivide the review into helper-level gates.

Use the caller's mode; standalone use defaults to read-only review. Do not expand a task into an
unrelated repository-wide cleanup. Preserve user edits, validation, error handling, accessibility,
and transaction semantics. Do not install dependencies or use secrets for a quality review.
Reading or using `.env` files or values, including inherited values and implicit loading by tools,
requires explicit scoped user approval. If a check cannot run safely with synthetic configuration,
report the gap rather than running it.

## Shared Criteria

### Coherent, Testable Units

Keep each function focused on one coherent responsibility. Separate substantial decision logic from
I/O or orchestration when that makes it directly testable. Extract a named function when it removes
repeated logic or clarifies a tangled responsibility, not to satisfy a line limit. Do not add
one-line wrappers, public APIs, or abstractions merely to split code up.

### Clear Control Flow

Prefer guard clauses and early returns for invalid inputs, missing data, and completed cases,
leaving the normal path easy to follow. Avoid needless nesting and `else` after a returning branch.
Use `switch` or pattern matching for genuine variants when clearer. An `else` is not itself a defect;
show a concrete readability problem and a behavior-preserving simplification. Do not flatten code
at the expense of cleanup, transaction, error-handling, or evaluation semantics.

### One Owner for Shared Rules

Reuse existing domain constants, enums, configuration, and predicates for values or rules that must
change together. Place shared definitions at the narrowest common module boundary, not in a global
constants dumping ground. Inspect every repeated location before claiming harmful duplication.

A constant names a stable value; a predicate names a rule. For example, if a worker and retry
endpoint share a delivery policy, both should reuse the same attempt limit and eligibility rule:

```typescript
// Illustrative only: use the project's approved policy and data model.
export const MAX_DELIVERY_ATTEMPTS = 3;

export function canRetryDelivery(delivery: Delivery): boolean {
  if (delivery.status !== "failed") return false;
  if (delivery.failureKind !== "transient") return false;
  return delivery.attempts < MAX_DELIVERY_ATTEMPTS;
}
```

This checks eligibility, not atomic reservation under concurrency. Do not invent the policy, cache
a condition over mutable data as a constant, extract meaningless constants such as `EMPTY_STRING`,
or couple unrelated matching literals. Similar syntax alone does not establish harmful duplication.

Look for duplicated business rules, validation, transformations, queries, and state transitions with
a concrete risk of drift. Distinguish necessary database invariants and trust-boundary validation
from redundant parsing or revalidation of already established internal values.

### Meaningful Tests

Use `write-tests` as the source of test conventions: direct unit calls, descriptive names, one
behavior per test, and prepare/execute/assert structure. Check observable outcomes, relevant failure
paths, and unchanged contract expectations. Flag weakened assertions, skipped coverage, leftover
placeholders in implemented scope, and suppressed diagnostics. Do not require tests for every
private helper or reject untouched planning skeletons in other tasks.

### Proportionate Work and Dependencies

Prefer existing project helpers, platform features, and installed dependencies. Question unnecessary
abstractions, dependencies, and unrelated changes. Inspect redundant reads, N+1 calls, over-fetching,
discarded results, avoidable allocations, blocking operations, and unbounded work.

Performance findings require measurements or a demonstrable work-amplification trace, such as one
query per item. Connect the cost to realistic workloads, an explicit requirement, or clearly
avoidable repeated work. State input-size assumptions; do not invent latency or memory numbers or
call code slow because another expression is shorter.

## Findings and Exit

No finding quota. Return a short result naming the reviewed scope, checks actually run, limitations,
and any concrete problems. Each finding needs a location, scenario, consequence, evidence, and the
smallest proposed solution. Label solutions as contract-preserving or requiring user approval.

A correctness, security, or contract defect, or a violation of an explicit quality/performance
requirement, blocks the calling gate. A proved maintainability or efficiency weakness that does not
invalidate those requirements is non-blocking. Taste and unverified suspicions are not findings.
Do not turn optional cleanup into a prerequisite for committing completed behavior.

When used inside `qa`, return candidate findings to its independent verifier; this skill's result
alone is not a QA pass. When used by `build`, return to the coordinator's task-level review; do not
launch QA or another code-quality pass. After fixes, rerun affected checks and inspect the changed
paths rather than restarting a whole-feature review for unchanged code.

---
name: use-skills
description: Select the appropriate skill for planning, challenging, implementing, testing, code quality, or reviewing work.
---

# Use Skills

Choose the skill that matches the current task and read its `SKILL.md` before proceeding.

| Skill | When to use |
| --- | --- |
| [plan](../plan/SKILL.md) | Establish the application's goal and agree on module, data, API, and behavioral contracts in code before implementation. |
| [challenge](../challenge/SKILL.md) | After planning, independently challenge feasibility, security, contract validity, and coverage of user needs. |
| [build](../build/SKILL.md) | Implement an approved contract through delegated workers, with coordinator oversight and optional automatic commits. |
| [write-tests](../write-tests/SKILL.md) | During planning, write only behavioral test stubs. During building, implement focused tests before production logic. |
| [code-quality](../code-quality/SKILL.md) | Apply shared quality, duplication, testability, and performance criteria within build task reviews, QA, or standalone review; no delegation or recursive gates. |
| [qa](../qa/SKILL.md) | Before completed feature handback or an explicitly requested milestone, verify correctness, security, and code quality with independent agents. |
| [educate](../educate/SKILL.md) | During planning or clarification, explain behavior, design tradeoffs, and test rationale with small data-flow diagrams and verified documentation. |
| [ponytail](https://ponytail.dev/#top) | While designing or implementing, prefer the least code that meets the contract: reuse existing code, standard-library features, platform capabilities, and installed dependencies before adding new machinery. |

Typical order: **plan → challenge → build**, with **write-tests** used inside plan and build, and
**code-quality** inside build task reviews and QA, and **qa** before completed feature handback.
With build `/auto`, verified tasks are committed incrementally after coordinator review, without
waiting for whole-feature QA. Full per-commit QA is opt-in, not the default.

## Shared Model-Selection Policy

Read and apply this policy whenever coordinating `challenge`, `build`, or `qa`, including when
those skills are invoked directly. Select by role, then freshness, then cost—not by the harness's
default model or a remembered version number.

| Role | Required tier |
| --- | --- |
| Controller/coordinator | High-capability reasoning and coding model |
| Challenge reviewers and reconciler | High-capability tier; never a budget-worker fallback |
| QA specialists and verifier | High-capability tier, not merely stronger than a cheap implementer |
| Implementation workers | Lower-cost tier that is competent for the assigned task; escalate when necessary |

### Model Preflight

Before delegation:

1. Discover the harness's currently available model IDs and refresh its catalog through supported
   mechanisms where possible. Do not invent CLI flags, model IDs, prices, or capabilities.
2. Establish each candidate's family, capability tier, generation/release date, stable/preview
   status, tools, and context limits using current catalog metadata and authoritative provider
   information where metadata is insufficient. A model name, version number, or default alone does
   not establish capability. If the catalog cannot establish freshness, verify it rather than
   silently relying on old training knowledge.
3. For high-capability roles, select the latest stable available generation within an approved
   high-capability family, such as Astra or Fable when configured and verified. These are family
   examples, not literal provider IDs or permanent version pins. Family approval comes from the
   user's selection or established harness configuration; if neither exists, propose a verified
   suitable family and ask. Do not compare version numbers across unrelated families. Multiple
   approved families need not share a version scheme; select a suitable current candidate and
   explain the choice rather than asserting an unsupported universal ranking.
4. For workers, prefer the latest stable generation within a suitable lower-cost family. Assess
   task complexity, required tools, context size, and known limitations before choosing it. Use
   verified pricing or an established user/harness cost tier, not assumptions from model names.
   A cheap model that cannot handle the task is ineligible.
5. Resolve aliases to actual model IDs and pass the selected ID explicitly at launch. Check the
   launch/session metadata against it. If the harness silently substitutes a different model,
   stop affected work and resolve the mismatch; that run does not satisfy the required model gate.
   Alias names such as "latest" are not evidence that their targets are current.

An explicit user version pin overrides the freshness preference, not the role's capability
requirement. Use an eligible pinned model without asking again. Previews require explicit user
selection; do not default to them merely because they are newer. If availability, capability,
freshness, or worker cost classification cannot be established, ask a focused question before
launching affected agents. Do not silently downgrade or substitute an older/default model.

Check the current controller too. If it cannot meet the high-capability requirement or its identity
cannot be verified, ask the user to switch models or move coordination to a suitable session using
supported harness mechanisms. Do not pretend the current session changed models.

### Announce, Record, and Escalate

Before launching agents, announce each role's resolved model ID, tier, and selection reason, with
the catalog/provider evidence supporting freshness and suitability. A single announcement may cover
multiple agents using the same model. Record the actual IDs in challenge/QA reports and in build's
local task tracker, including model changes; disclose any unavailable preferred candidate.

Workers must report capability limits or repeated unsuccessful attempts to the coordinator. Reassess
the task and escalate to a verified high-capability worker when the lower-cost tier is insufficient;
do not repeat the same failing approach indefinitely. Announce and record the escalation, preserve
file ownership and contract gates, and do not let workers launch replacement agents themselves.
If no eligible model is available, report a blocker rather than claiming a lower-tier run qualifies.

## External Complement

Ponytail is an external skill; locate and read its installed instructions before using it. If it is
not installed, report that rather than claiming it was applied. Its simplification guidance does
not authorize weakening contracts, security controls, tests, user-approval gates, or QA requirements.

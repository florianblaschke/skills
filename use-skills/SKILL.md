---
name: use-skills
description: Select the appropriate skill for planning, challenging, implementing, testing, or reviewing work.
---

# Use Skills

Choose the skill that matches the current task and read its `SKILL.md` before proceeding.

| Skill | When to use |
| --- | --- |
| [plan](../plan/SKILL.md) | Establish the application's goal and agree on module, data, API, and behavioral contracts in code before implementation. |
| [challenge](../challenge/SKILL.md) | After planning, independently challenge feasibility, security, contract validity, and coverage of user needs. |
| [build](../build/SKILL.md) | Implement an approved contract through delegated workers, with coordinator oversight and optional automatic commits. |
| [write-tests](../write-tests/SKILL.md) | During planning, write only behavioral test stubs. During building, implement focused tests before production logic. |
| [qa](../qa/SKILL.md) | Before a build commit or completed user handback, verify implementation correctness, security, quality, duplication, and performance with independent agents. |
| [educate](../educate/SKILL.md) | During planning or clarification, explain behavior, design tradeoffs, and test rationale with small data-flow diagrams and verified documentation. |
| [ponytail](https://ponytail.dev/#top) | While designing or implementing, prefer the least code that meets the contract: reuse existing code, standard-library features, platform capabilities, and installed dependencies before adding new machinery. |

Typical order: **plan → challenge → build**, with **write-tests** used inside plan and build, and
**qa** before build commits and completed handback.

Ponytail is an external skill; locate and read its installed instructions before using it. If it is
not installed, report that rather than claiming it was applied. Its simplification guidance does
not authorize weakening contracts, security controls, tests, user-approval gates, or QA requirements.

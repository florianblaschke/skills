# Skills

These are my personal skills that I use with AI coding agents. They are **under active development**
and will evolve as I use and refine them.

The main goal is to rely less on specifications in `.md` files and instead create **concrete module
contracts in code**: data definitions, interfaces, and behavioral test expectations. Those contracts
are then enforced during build, with explicit user approval required for changes. Markdown remains
a lightweight place for goals and decision rationale, not a parallel source of truth.

The workflow is contract-first: agree on behavior, challenge the design, implement it through
delegated workers, and independently verify the result.

The skill instructions live in this repository. No npm package or agent-specific marketplace is
required to distribute them.

## Install

Use the [Skills CLI](https://github.com/vercel-labs/skills), which requires Node.js 22.20 or later.
Install the collection globally for the agents you use:

```bash
npx skills add florianblaschke/skills --skill '*' --agent claude-code codex pi --global
```

Remove agent names you do not use. Omit `--global` to install into the current project instead.
The installer downloads the skill instructions from this GitHub repository and puts them in the
appropriate agent skill locations.

To inspect the available skills without installing:

```bash
npx skills add florianblaschke/skills --list
```

To install just one skill:

```bash
npx skills add florianblaschke/skills --skill educate --agent claude-code --global
```

The workflow skills reference each other, so install the whole collection to use the full workflow.
Standalone installation is useful for skills such as `educate`. Review the instructions before
using them; skills can direct an agent's tools and workflow.

Private repository access uses the credentials already configured for Git. See the
[CLI installation reference](https://github.com/vercel-labs/skills/blob/main/docs/commands.md#skills-add).

## Skills

| Skill | Purpose |
| --- | --- |
| [use-skills](use-skills/SKILL.md) | Choose the right skill for the current task. |
| [plan](plan/SKILL.md) | Agree on the goal and draft concrete module contracts in code with the user. |
| [challenge](challenge/SKILL.md) | Challenge the design with two independent reviewers and a third reconciler. |
| [build](build/SKILL.md) | Divide implementation into flat, owned tasks; delegate independent work in parallel and optionally commit each verified task. |
| [write-tests](write-tests/SKILL.md) | Write test stubs during planning and executable, unit-first tests during building. |
| [code-quality](code-quality/SKILL.md) | Shared quality, duplication, testability, and performance criteria for build, QA, or standalone review. |
| [qa](qa/SKILL.md) | Review the integrated feature with two specialists and a proof verifier before completed handback. |
| [educate](educate/SKILL.md) | Explain behavior and test rationale with clear diagrams and verified documentation. |

## Workflow

```text
plan ──> challenge ──> build
 │                      │
 └─ write-tests         ├─ write-tests: implement tests → RED → implement logic → GREEN
    (stubs only)        ├─ code-quality + coordinator review → task commits (/auto)
                        └─ qa: integrated feature before completed handback

educate: whenever an explanation or clarification is needed
```

- **The code is the contract.** Declarations, schemas, comments, and behavioral test expectations
  define module behavior. Planning happens in the actual source/test files, not parallel draft
  files; new files are created only when the feature needs them. Drafts remain proposals until approved.
- **Approval belongs to the user.** Implementation cannot silently change the approved contract.
  Unresolved blocking challenge findings prevent feature implementation.
- **The local spec is not a second contract.** `spec-<feature-name>.md` records the goal, significant
  decision rationale, approvals, and baseline references. It stays uncommitted.
- **Build uses a flat task queue.** Track outcomes, ownership, dependencies, checks, progress, and
  commits in local, uncommitted `tasks-<feature-name>.md`. Run independent ready tasks in parallel
  without recursive delegation.
- **Red before production code.** Implement and run tests first; record meaningful failing assertions
  before writing the missing behavior, then rerun to green. The coordinator checks ordered evidence.
  Tests that already pass against unchanged existing behavior are recorded as such, not forced red.
- **Build auto defaults to off.** `/auto` commits each ready task after scoped verification,
  `code-quality`, and coordinator review, without waiting for the whole feature. Full QA gates
  completed handback; intermediate full-QA gates are opt-in. Planning separately creates a contract
  baseline after explicit approval.
- **Code quality is shared, not another agent layer.** Build and QA use the same `code-quality`
  criteria. Reviews stay at task/feature boundaries; fixes get targeted checks, not recursive gates.
- **QA findings need proof.** Unverified suspicions are not confirmed defects; material verification
  gaps prevent clearance.
- **Secrets are not implicitly available.** QA requires explicit scoped permission before reading
  or using `.env` files or values, including inherited values and implicit loading by test runners.

[Ponytail](https://ponytail.dev/#top) is an optional external complement, not bundled in this repository.
Its simplification guidance does not override the contracts or approval and verification gates.

## Agent Requirements

These files provide portable instructions, not an execution engine. Installation does not add
missing tools to an agent harness.

- `plan`, `write-tests`, `code-quality`, and `educate` need appropriate file access. `educate` also needs access to
  authoritative documentation to verify externally defined technical claims.
- All workflow coordinators use verified high-capability models.
- `build` needs delegation to a suitable lower-cost worker, with escalation for tasks beyond its ability.
- `challenge` needs two independent fresh-context reviewers, then a third fresh-context reconciler,
  all on high-capability models.
- `qa` needs two fresh-context specialists and a separate fresh-context verifier, all on high-capability
  models—not merely models stronger than a budget worker.

The [shared model-selection policy](use-skills/SKILL.md#shared-model-selection-policy) requires a
current catalog/provider preflight, latest stable available generations within approved families,
explicit resolved model IDs, launch verification, and visible selection reasons. User version pins
are honored when eligible; uncertain classification or unavailable required models blocks delegation
rather than causing silent downgrade. Astra and Fable are family examples, not hardcoded versions
or universal provider IDs.

If the required delegation or models are unavailable, the relevant skill reports a blocker rather
than pretending an in-session self-review is equivalent. Availability depends on the user's harness
and configuration; the instructions are not a promise of identical orchestration across all agents.

Restart or reload the agent after installing or updating if it does not discover the skills
immediately. In Pi, use `/reload`.

## Updates

To update only the skills from this repository installed globally:

```bash
npx skills update --global use-skills plan challenge build write-tests code-quality qa educate
```

For project installations, use `--project` instead of `--global`. Review local customizations before
updating.

## License

[MIT](LICENSE).

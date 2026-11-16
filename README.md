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
| [build](build/SKILL.md) | Delegate implementation while enforcing the contract and optional automatic commits. |
| [write-tests](write-tests/SKILL.md) | Write test stubs during planning and executable, unit-first tests during building. |
| [qa](qa/SKILL.md) | Review correctness, security, quality, duplication, and performance with two specialists and a proof verifier. |
| [educate](educate/SKILL.md) | Explain behavior and test rationale with clear diagrams and verified documentation. |

## Workflow

```text
plan ──> challenge ──> build
 │                      │
 └─ write-tests         ├─ write-tests: tests → logic → verification
    (stubs only)        └─ qa: before commits and completed handback

educate: whenever an explanation or clarification is needed
```

- **The code is the contract.** Declarations, schemas, comments, and behavioral test expectations
  define module behavior. Planning drafts remain proposals until approved.
- **Approval belongs to the user.** Implementation cannot silently change the approved contract.
  Unresolved blocking challenge findings prevent feature implementation.
- **The local spec is not a second contract.** `spec-<feature-name>.md` records the goal, significant
  decision rationale, approvals, and baseline references. It stays uncommitted.
- **Build auto defaults to off.** When enabled, commits require scoped verification, QA clearance,
  and coordinator review. Planning separately creates a contract baseline after explicit approval.
- **QA findings need proof.** Unverified suspicions are not confirmed defects; material verification
  gaps prevent clearance.
- **Secrets are not implicitly available.** QA requires explicit scoped permission before reading
  or using `.env` files or values, including inherited values and implicit loading by test runners.

[Ponytail](https://ponytail.dev/#top) is an optional external complement, not bundled in this repository.
Its simplification guidance does not override the contracts or approval and verification gates.

## Agent Requirements

These files provide portable instructions, not an execution engine. Installation does not add
missing tools to an agent harness.

- `plan`, `write-tests`, and `educate` need appropriate file access. `educate` also needs access to
  authoritative documentation to verify externally defined technical claims.
- `build` needs delegation to a user-selected or suitable lower-cost implementation model.
- `challenge` needs two independent fresh-context reviewers, then a third fresh-context reconciler.
- `qa` needs two fresh-context specialists and a separate fresh-context verifier, using models more
  capable than the implementer. Model names in the instructions are examples, not universal IDs.

If the required delegation or models are unavailable, the relevant skill reports a blocker rather
than pretending an in-session self-review is equivalent. Availability depends on the user's harness
and configuration; the instructions are not a promise of identical orchestration across all agents.

Restart or reload the agent after installing or updating if it does not discover the skills
immediately. In Pi, use `/reload`.

## Updates

To update only the skills from this repository installed globally:

```bash
npx skills update --global use-skills plan challenge build write-tests qa educate
```

For project installations, use `--project` instead of `--global`. Review local customizations before
updating.

## License

[MIT](LICENSE).

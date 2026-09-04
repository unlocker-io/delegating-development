---
name: delegating-development
description: Delegate every code implementation to a subagent while the primary agent owns exploration, specification, and review. Use before changing source, tests, executable configuration, or scripts, including one-line fixes and urgent work. Do not use when already acting as the implementation subagent or when only reading and diagnosing.
---

# Delegating Development

Separate thinking from implementation: the primary agent explores, specifies, dispatches, and
reviews; an implementation subagent writes every code change.

## Provider adapter

Identify the current host from its available tools and conventions, then read exactly one adapter:

- Claude Code: [references/providers/claude-code.md](references/providers/claude-code.md)
- Codex: [references/providers/codex.md](references/providers/codex.md)

If the host is neither, use this core contract with the closest available subagent mechanism. Do
not invent provider-specific tool or model names.

## Core contract

The primary agent must not edit source code, tests, executable configuration, or scripts. This
applies to features, fixes, refactors, generated code, and one-line changes.

The primary agent may write non-code artifacts such as plans, specifications, dispatch prompts,
analysis documents, and review notes. Read-only exploration is encouraged.

If the primary agent has already edited code, revert only its own edit, preserve unrelated user
changes, and dispatch from a behavioral specification rather than handing the subagent a patch to
rubber-stamp.

This contract does not apply when:

- the current agent is the implementation subagent executing a specification;
- the task is read-only exploration, diagnosis, explanation, or review;
- the task is a release or operational procedure that changes no source, test, script, generated
  artifact, or executable configuration;
- the user explicitly selects the primary agent as executor and rejects delegation for the current
  task, for example: "fais-le toi-même, sans sous-agent" or "do it yourself; do not delegate".

For the last case, acknowledge the one-task override in one sentence before implementing directly,
then resume delegation for the next code task. Urgency, triviality, speed, cost, latency, or wording
such as "directly" is not an override. A hedged suggestion such as "autant le faire toi-même" or
"you might as well do it yourself" is not an instruction and does not override delegation.

## Capability selection

Choose a capability tier from the nature of the change, never from urgency, queue length, token
budget, or willingness to pay. The provider adapter maps these tiers to available models.

| Tier | Use |
|---|---|
| `mechanical` | Only a closed-list mechanical change below |
| `implementation` | Default for every change not admitted elsewhere |
| `deep` | Only a closed-list invariant-heavy change below |

The `mechanical` list is closed:

- identical rename or move of a named file or symbol;
- apply a patch already written completely in the specification;
- add a test case matching an existing case in the same file with only data changed;
- update a fixture, snapshot, or Postman entry dictated exactly in the specification;
- fix lint, formatting, or a missing import on identified lines;
- remove specifically identified dead code.

The `deep` list is closed:

- cross-cutting contract or signature change with at least ten counted consumers;
- non-trivial algorithm or invariant such as financial calculation, allocation, date bounds, or reconciliation;
- data migration requiring idempotence, reversibility, and before/after counts;
- root-cause fix spanning multiple modules that cannot be decomposed without losing the invariant.

When uncertain, use `implementation`. Announce `mechanical` or `deep` dispatches by citing the exact
qualifying entry and any required count. The default tier needs no defense.

Escalation is one-way:

- after one failed `mechanical` dispatch, start a new `implementation` subagent with a corrected spec;
- after two failed `implementation` attempts on the same spec, split the spec; use `deep` only if a
  closed-list predicate independently applies;
- never continue a failed lower-tier agent if doing so preserves its lower-tier model.

## Workflow

1. Explore read-only and write a bounded specification: exact files, expected behavior, inputs and
   outputs, edge cases, constraints, and exact verification commands.
2. Dispatch one implementation subagent with the provider adapter's mechanism and model mapping.
3. Require a summary of changed files and verbatim verification output.
4. Inspect the real diff and verification evidence. Compare both against the specification.
5. Re-dispatch corrections. The primary agent must not finish a failed implementation itself.

Keep each implementation prompt self-contained. Include only relevant context and excerpts, not the
primary conversation history. The primary agent owns the final review; do not delegate acceptance of
the subagent's own work back to that same subagent.

### Dispatch prompt checklist

- Context: what and why, in two sentences.
- Exact files and only the relevant excerpts already inspected.
- The shared exploration artifact named as required first read, when one exists (see below).
- Expected inputs, outputs, edge cases, constraints, and prohibited changes.
- TDD expectation when applicable and the exact verification command.
- Return format: changed-file summary plus verbatim test and check output.

### Exploration artifact — pay for discovery once

When more than one dispatch will touch the same codebase, the discovery phase is written down once
and shared, instead of being re-run by every fresh implementation agent (measured cost of the
re-run: a large fraction of each agent's budget goes to re-reading the same model files).

The artifact is a cartography file at the working tree root, excluded from version control
(`.exploration-<ticket>.md` via `git rev-parse --git-path info/exclude`). It contains, in this
order:

1. a dated header stating it is a photo of the code at a commit, with a staleness warning;
2. the environment and gate commands exactly as the agents must run them;
3. the file paths and signatures of the models to imitate (entities, ports, migrations, tests),
   with the conventions they embody;
4. the measured baselines (test counts, suite status) to compare against;
5. the known traps, each with its observed consequence.

Every dispatch prompt names this file as the required first read and carries only the
task-specific specification on top of it. After accepting each increment, the primary agent
appends what changed — new files, new baselines — before the next dispatch.

The cartography is an analysis artifact owned by the primary agent. Canonical specifications and
plans stay in their own system of record; the file never substitutes for them and is never
committed.

## Session budget

- One implementation context handles one Linear ticket or one atomic deliverable.
- Limit an implementation dispatch to 30 minutes. Split larger work into a fresh dispatch.
- Start each dispatch from its self-contained specification; do not carry a previous agent's noisy
  context into the next ticket.
- Separate planning, implementation, and final review contexts for multi-repo features. Use the
  provider's compaction mechanism between phases when appropriate.

## Rationalization guards

| Rationalization | Required response |
|---|---|
| "It is only a one-line change." | The policy separates roles, not line counts. Specify and dispatch it. |
| "Subagents are only useful for exploration or parallelism." | Delegation here exists to separate specification, implementation, and review. |
| "The file is already in my context." | Use that context to improve the specification; it does not authorize editing. |
| "The user is in a hurry." | Urgency does not select the executor or capability tier. |
| "The user said to do it directly or suggested I might as well do it myself." | Manner, speed, or a hedged suggestion is not an instruction selecting the primary agent and rejecting delegation; specify and dispatch. |
| "The user said to do it yourself without a subagent." | Acknowledge the explicit one-task override, implement directly, and resume delegation on the next code task. |
| "The extra tokens or latency bring no value." | The user chose independent implementation and review; do not re-arbitrate it. |
| "TDD is sequential." | The implementation agent runs the red-green-refactor loop. |
| "The subagent failed, so I should finish it." | Correct or split the specification and re-dispatch. |
| "Configuration is not code." | Executable configuration and scripts are code under this policy. |
| "This is almost mechanical." | The mechanical list is closed; almost means `implementation`. |
| "This is hard, so use deep." | Split ordinary difficulty; use `deep` only for a listed predicate. |
| "The lower tier failed because my spec was vague." | Escalation remains one-way; start a fresh higher-tier agent. |

## Red flags — stop and dispatch

- The primary agent is about to edit source, tests, scripts, or executable configuration.
- "I will just fix this quickly" or "delegation is overkill here."
- A `mechanical` or `deep` selection without an exact closed-list predicate.
- A second lower-tier attempt after that tier already failed.
- Capability selection based on urgency, price, latency, or queue length.
- Treating vague speed, manner, or hedged suggestion language as an override, or silently extending a valid override beyond the current task.
- Moving to the next task before inspecting the actual diff and verification evidence.

## Common mistakes

- A vague dispatch prompt leaves design decisions to the implementation agent.
- A green-test claim without verbatim output and diff inspection is not evidence.
- The implementation agent must not accept its own work; final review belongs to the primary agent.
- A user asking for speed or saying "directly" is not selecting the primary agent as executor.

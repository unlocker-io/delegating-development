# Codex adapter

Use Codex subagent delegation. In environments exposing named collaboration tools, spawn a bounded
implementation agent and wait for its result before accepting the task. In Codex CLI, IDE, or app
surfaces, request a subagent directly; applicable skill instructions authorize delegation.

Map the portable capability tiers by role, preferring configured custom agents when present:

| Tier | Codex selection |
|---|---|
| `mechanical` | A fast, lower-cost coding model at low or medium reasoning |
| `implementation` | The current default coding model and reasoning effort |
| `deep` | A frontier coding model at high reasoning effort |

Treat these as capability mappings, not permanent model aliases. If an exact model is unavailable,
select the closest available model for the tier; never fail the workflow merely because a sample
model name has changed. If the environment does not permit model selection, inherit the parent and
preserve the tier in the dispatch prompt.

Start a new agent when escalating from `mechanical` to `implementation`; continuing the same thread
may preserve its original model. Give each writing agent an isolated worktree or non-overlapping
file ownership. Parallelize only independent tasks; the separation-of-roles policy does not require
parallel execution.

Use the host's plan mechanism when the task benefits from progress tracking. Ask the user in plain
language or with the available input tool when the core workflow requires a decision; do not refer
to Claude-specific tool names.

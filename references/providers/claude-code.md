# Claude Code adapter

Use Claude Code's `Agent` tool for implementation dispatches. If a workflow API is used, set
`opts.model` explicitly on every implementation agent.

When a `dev` agent type is defined (`.claude/agents/dev.md` or listed in the session's available
agent types), dispatch implementation work with `subagent_type: "dev"` — it carries the standing
implementation contract (TDD loop, mutation proof, git hygiene, report format) so the dispatch
prompt carries only the task specification and the exploration artifact reference. Always pass the
tier's model explicitly via the `model` parameter: the per-dispatch override takes precedence over
the agent definition's default. Without a `dev` agent, fall back to `general-purpose`.

Map the portable capability tiers as follows:

| Tier | Claude model |
|---|---|
| `mechanical` | `haiku` |
| `implementation` | `sonnet` |
| `deep` | `opus` |

Use a fresh Sonnet agent after a failed Haiku attempt. `SendMessage` to the failed agent retains its
model and therefore counts as a forbidden second Haiku attempt.

When available, use `superpowers:subagent-driven-development` for the dispatch/review loop and
`superpowers:dispatching-parallel-agents` only for genuinely independent work in separate
worktrees. These sub-skills are conveniences, not requirements of the portable contract.

For long-lived observer or memory agents:

- cap a session at 20 tool events and return structured summaries;
- truncate large command output before ingestion;
- write observations to a JSONL artifact instead of accumulating them in context;
- poll CI every three minutes, at most ten times, rather than using a long-running watcher.

Use Claude's planning/task-list facility when useful, but do not make a specific tool name part of
the behavioral contract.

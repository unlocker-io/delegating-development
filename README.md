# Delegating Development

A provider-neutral [Agent Skill](https://code.claude.com/docs/en/skills) that separates *thinking*
from *typing*: the primary agent explores, specifies, dispatches, and reviews — an implementation
subagent writes every code change.

It is deliberately hard to talk your way out of. Urgency, triviality, "it's only one line", "the
file is already in my context", or "you might as well do it yourself" are not overrides; the skill
enumerates those rationalizations and the required response to each.

## What it does

- **Core contract** — the primary agent must not edit source, tests, executable configuration, or
  scripts. It may write plans, specs, dispatch prompts, and review notes. Read-only exploration is
  encouraged. A short, closed list of exemptions covers being the subagent, read-only work,
  operational procedures, and an explicit per-task user override.
- **Capability tiers** — `mechanical` / `implementation` / `deep`, selected from the *nature of the
  change* and never from urgency, cost, or latency. The `mechanical` and `deep` lists are closed;
  `implementation` is the default and needs no defense. Escalation is one-way.
- **Provider adapters** — the portable contract stays in `SKILL.md`; tool and model names live in
  [`references/providers/`](references/providers/) (Claude Code, Codex). Other hosts use the core
  contract with their closest subagent mechanism.
- **Exploration artifact** — when several dispatches touch the same codebase, discovery is written
  down once (an uncommitted cartography file) instead of being re-run and re-paid by every fresh
  implementation agent.
- **Dispatch checklist and review loop** — self-contained prompts, verbatim verification output,
  and a real diff inspection by the primary agent. A subagent never accepts its own work.

## Install

With the [`skills`](https://www.npmjs.com/package/skills) CLI:

```bash
npx -y skills add -g https://github.com/unlocker-io/delegating-development
```

Or by hand — the skill is the repository root, so a clone can be symlinked directly:

```bash
git clone https://github.com/unlocker-io/delegating-development.git ~/src/delegating-development
ln -s ~/src/delegating-development ~/.claude/skills/delegating-development   # Claude Code
ln -s ~/src/delegating-development ~/.agents/skills/delegating-development   # Codex
```

Symlinking means `git pull` is your update channel. `npx skills add` copies instead, so re-run
`npx -y skills update -g` to refresh.

## Layout

| Path | Role |
|---|---|
| `SKILL.md` | The portable contract: core rules, tiers, workflow, rationalization guards |
| `references/providers/claude-code.md` | `Agent` tool dispatch, `dev` agent type, tier → model map |
| `references/providers/codex.md` | Codex subagent dispatch, tier → role map |
| `agents/openai.yaml` | Display metadata for OpenAI-compatible surfaces |
| `evals/evals.json` | Dry evals: pressure scenarios asserting tier choice and no primary-agent edit |

## Contributing

Issues and pull requests welcome. Keep `SKILL.md` provider-neutral — anything naming a specific
tool, model, or vendor mechanism belongs in an adapter under `references/providers/`.

## License

MIT — see [LICENSE](LICENSE).

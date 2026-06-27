# agentic-skills

e4stwood's personal agent skills for **agentic execution** and **long-running coding tasks**. Works with Claude Code and Codex.

## Skills

| Skill | What it does |
| --- | --- |
| [grill-me-goal](./grill-me-goal) | Turn a vague objective into a durable, repo-local **goal package** — a complete goal prompt plus docs and a phased, evidence-gated acceptance-criteria checklist — that an agent can execute for hundreds of hours across many sessions. Built to be run by Codex's long-running `/goal` skill. |
| [ripple-check](./ripple-check) | After a code change, run a deep **ripple-effect analysis** across every internal consumer and external dependency, fix each ripple test-first (`/tdd` red→green→refactor), extend coverage over all externalities, run full validation, and update docs. |

## Install

These follow the [Agent Skills spec](https://github.com/vercel-labs/skills) (`npx skills`):

```bash
npx skills add e4stwood/agentic-skills        # whole collection
```

Or copy one skill folder into your agent's skills directory:

```bash
cp -R <skill-name> ~/.claude/skills/          # Claude Code
cp -R <skill-name> ~/.codex/skills/           # Codex
```

Invoke with `/<skill-name>` (Claude Code) or `$<skill-name>` (Codex).

## Recommended companions

- **[mattpocock/skills](https://github.com/mattpocock/skills)** — source of the **`/tdd`** skill (red → green → refactor) that both skills here embed and rely on. Install it so the test-first ethos is live at execution time:
  ```bash
  npx skills add mattpocock/skills
  ```
- **Codex long-running `/goal`** — `grill-me-goal` produces the package; the Codex `/goal` skill executes it over long, multi-session runs by pointing at the generated goal prompt.

## Repo conventions

```text
<skill-name>/
  SKILL.md          # frontmatter (name, description) + instructions
  README.md         # what it does, what it produces, how to install
  agents/openai.yaml  # Codex/OpenAI interface metadata (optional)
```

To add a skill: create the folder above, then add a row to the **Skills** table.

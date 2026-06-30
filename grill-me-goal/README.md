# grill-me-goal

Turn a vague long-running objective into a durable, repo-local **goal package**: a complete goal prompt plus docs and a phased, evidence-gated acceptance-criteria checklist that an agent can execute for hundreds of hours across many sessions without relying on chat history.

Use it to **define, sharpen, persist, resume, audit, or run** a multi-session goal.

## Pairs with Codex `/goal`

`grill-me-goal` writes the package; **Codex's long-running `/goal` skill executes it** by pointing at the generated goal prompt and working the checklist over long, multi-session runs. Use `grill-me-goal` to plan, `/goal` to run.

## What it produces

A package under `docs/<goal-slug>/`:

| File | Purpose |
| --- | --- |
| `goal.md` | Source of truth + the complete, self-contained goal/resume prompt |
| `ac.md` | Phased checklist, each phase ending in an end-to-end gate |
| `context.md` | Repo-grounded context from actual inspection |
| `research.md` | External/API truth with cited primary sources + access dates |
| `test-plan.md` | TDD + end-to-end strategy with runnable commands per layer |
| `questions.md` | Batched multiple-choice clarifications + answers/assumptions |
| `decisions.md` | Durable decisions affecting execution |
| `risks.md` | Risk register with mitigations and linked A/Cs |
| `execution-log.md` | Resume history (red/green/refactor + end-to-end run per entry) |

## Three non-negotiable doctrines

1. **Test-driven** — every behavior change is failing-test-first (red → green → refactor), vertical slices, testing behavior through public interfaces. Green unit tests are **not** "done": you must also physically verify in the running app (click through it in a browser for web work, or run the real device/CLI/service).
2. **Evidence before progression** — no box is ticked and no phase advances without concrete proof (the physical run, command/test output, screenshot, log, link). Phase gates must be green with evidence before the next phase starts.
3. **End-to-end completion** — every feature is wired and demonstrated working through the full stack (UI → API → data → back) in a real run, never a stubbed half. Each milestone ends with an end-to-end gate.

## How it works

1. **Research first** — inspect the repo (docs, tests, CI, run harness) and research external API/SDK specs, limits, and current best practices; fan out to subagents in parallel when available.
2. **Foundation gate** — if this is a new or under-documented repo, establish `AGENTS.md`, `CLAUDE.md`, domain language, and project principles via `grill-with-docs` before writing the long-running goal package.
3. **Design the test/E2E strategy** in `test-plan.md` before writing criteria.
4. **Batched multiple-choice questions** — 2-10 rounds, many decisions at once (default 8-20 per batch), not one at a time. Recommendations are advisory only: the skill must never assume option A or silently apply defaults unless the user explicitly says to use recommended defaults.
5. **Write the package** — 100–300 phased, test-backed, evidence-gated criteria for large goals.
6. **Forward progress** — the generated goal prompt forces every execution turn to complete a concrete action, use bounded subagent side missions when available, and stay focused on the current phase/A/C.
7. **Audit** until a fresh agent can resume from the docs alone.

## Companion: `/tdd`

The red → green → refactor core comes from [mattpocock/skills](https://github.com/mattpocock/skills). Install it so the test-first ethos is live during execution; the generated goal prompt instructs the agent to adopt it:

```bash
npx skills add mattpocock/skills
```

## Install

```bash
npx skills add e4stwood/agentic-skills        # via Agent Skills spec
# or
cp -R grill-me-goal ~/.claude/skills/         # Claude Code
cp -R grill-me-goal ~/.codex/skills/          # Codex
```

Invoke with `/grill-me-goal` (Claude Code) or `$grill-me-goal` (Codex).

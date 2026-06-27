# ripple-check

You just changed the codebase. `ripple-check` maps and contains the **blast radius**: a deep ripple-effect analysis across every internal consumer and external dependency, fixing each ripple **test-first**, extending coverage over all externalities, running full validation, and updating docs.

Run it after editing code, before committing.

## What it checks

In parallel (via Explore subagents), beyond in-process code, across **all externalities**:

- Databases & migrations (forward/backward safety, backfills)
- Queues, event streams, webhooks (payload versioning)
- Third-party APIs/SDKs **and your own published API contract**
- Config, secrets & feature flags (including the flag-off path)
- Caching & invalidation
- Observability (logs, metrics, traces, alerts)
- Security/compliance, performance, concurrency, idempotency, rate limits
- Build, CI & infrastructure-as-code

## TDD over every ripple

Every fix follows the **`/tdd`** loop: failing test reproducing the ripple (RED) → minimal fix (GREEN) → refactor with the suite green. No fix-first-test-later, never refactor while red, behavior tested through public interfaces. Each loop leaves a test that would have caught the regression; evidence is required before anything is reported done.

The `/tdd` core comes from [mattpocock/skills](https://github.com/mattpocock/skills) — install it so the ethos is live:

```bash
npx skills add mattpocock/skills
```

## Install

```bash
npx skills add e4stwood/agentic-skills        # via Agent Skills spec
# or
cp -R ripple-check ~/.codex/skills/           # Codex
# Claude Code: use it as the /ripple-check command in ~/.claude/commands/
```

Invoke with `/ripple-check` (Claude Code) or `$ripple-check` (Codex), immediately after making a change.

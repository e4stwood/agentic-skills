---
name: ripple-check
description: After making code changes, perform a deep ripple-effect analysis across every internal consumer and external dependency (databases, queues, caches, third-party APIs, webhooks, config/secrets, feature flags, jobs, observability), fix each ripple test-first via the /tdd red→green→refactor loop, extend coverage over all externalities, run full validation, update docs, and leave the codebase better. Use after editing code, before committing, or when asked to check the blast radius of a change.
---

# Ripple Check

You have just made changes to the codebase. Your job is to perform a **deep ripple-effect analysis**, fix every affected area **test-first**, extend test coverage across **all internal consumers and external dependencies**, run full validation, update documentation, capture learnings, and leave the codebase in a better state than you found it.

## Operating Principles

These govern everything below — hold them throughout.

- **TDD over every ripple.** Adopt the `/tdd` ethos and invoke `/tdd` when it is available. For each ripple you fix, write a failing test that reproduces the gap **first** (RED), make the minimal change to pass (GREEN), then refactor with the suite green (REFACTOR). Never refactor while red. Work in vertical slices — one ripple, one test, one fix — not a pile of tests then a pile of fixes. Test behavior through public interfaces so the tests survive future refactors. (The `/tdd` skill's core comes from [mattpocock/skills](https://github.com/mattpocock/skills).)
- **Cover all externalities.** A ripple is not only in-process code. Trace the change through every boundary the app touches: databases and migrations, message queues and event streams, caches, third-party APIs and SDKs, webhooks, your own published API consumed by other clients, env/config/secrets, feature flags, scheduled jobs and crons, file/object storage, email/notifications, analytics, and observability. An externality that changes behavior but has no test is an unfinished ripple.
- **Evidence before done.** Every claimed fix is backed by a concrete artifact — a failing-then-passing test, command output, log excerpt, or migration dry-run. Assertions ("should be fine") are not evidence. The full suite must be green before you report complete.
- **Stay in scope.** Fix what *this change* rippled into. Do not change the original intent of the modification, and do not refactor unrelated code that the change did not touch or expose.

## Step 1: Identify What Changed

Look at the current git diff (staged + unstaged) to understand exactly which files, models, properties, API endpoints, types, event payloads, config keys, and data flows were modified. Build a mental map of the blast radius.

Explicitly list the **contract surfaces** the change touches, because these are where ripples escape the process:

- Public/exported types and function signatures.
- HTTP/RPC/GraphQL request and response shapes.
- Database schema (columns, constraints, indexes, enums).
- Event/message/webhook payloads.
- Config keys, environment variables, and feature flags.

## Step 2: Parallel Deep Investigation

Launch multiple Explore sub-agents in parallel to investigate every dimension of impact. Each agent should search thoroughly ("very thorough" level) and return concrete file paths plus, for each hit, **whether it is currently covered by a test**.

### Internal consumers

1. **Backend consumers** - Every service, controller, guard, pipe, interceptor, resolver, and job that reads or writes the changed models/properties. Check ORM queries that touch the affected tables or fields.
2. **Frontend consumers** - Every hook, component, screen, and form that fetches, displays, mutates, or depends on the changed data. Check API service files, type definitions, and inline fetch calls.
3. **Type & schema alignment** - Database schemas, DTOs, frontend TypeScript types/interfaces, validation schemas, and API response shapes all in sync. Flag any mismatch.
4. **Auth & access control** - Whether the changes affect data scoped by role or permission. Verify every role still sees correct data and that guards/middleware remain consistent.
5. **Related features & workflows** - Trace the changed data through downstream features (notifications, exports, dashboards, search, reporting) for breakage, stale references, or missing updates.
6. **Existing test coverage** - All existing tests (unit, integration, e2e) exercising the changed code paths. Identify tests now broken, incomplete, or missing.

### Externalities (boundaries the change can ripple across)

7. **Data & migrations** - Whether the change needs a migration; whether existing migrations are forward/backward safe; data backfills; nullability and default changes; impact on existing rows; up/down reversibility.
8. **Async & background** - Message queues, event streams, pub/sub topics, webhooks (inbound and outbound), scheduled jobs, and crons that produce or consume the changed payloads. Check payload schema versioning and consumers that deserialize old/new shapes.
9. **External services & API contracts** - Third-party APIs/SDKs the change calls, and **your own published API** consumed by other apps/clients. Check contract/back-compat, versioning, deprecations, retries, timeouts, and error mapping.
10. **Config, secrets & feature flags** - New/changed env vars, config keys, secrets, and flags. Verify defaults, the flag-off path, and that all deploy targets (local, CI, staging, prod) have what they need.
11. **Caching & invalidation** - Caches (in-memory, Redis, CDN, HTTP, query caches) holding the changed data. Verify keys, TTLs, and invalidation on write so stale data is not served.
12. **Observability** - Logs, metrics, traces, dashboards, and alerts referencing the changed fields/endpoints. Ensure new failure modes are observable and no alert silently breaks.
13. **Security & compliance** - New input surfaces, authz boundaries, PII handling, audit logging, and data-retention implications introduced by the change.
14. **Performance & concurrency** - N+1 queries, added latency, race conditions, idempotency of retried operations, and rate-limit/quota interactions introduced by the change.
15. **Build, CI & infra** - Build steps, lint/type configs, CI pipelines, and infrastructure-as-code (env vars, resources, permissions) affected by the change.

## Step 3: TDD the Fixes (the spine of this command)

For **every** issue and coverage gap found in Step 2, work the `/tdd` loop — do not fix first and test later:

1. **RED** - Write one failing test that reproduces the ripple or gap, at the right level:
   - In-process logic → unit/integration test.
   - API/contract → request/response or contract test.
   - Event/webhook → payload (de)serialization test against old and new shapes.
   - Migration → up/down test on representative data.
   - Cache/flag/authz → behavior test toggling the relevant condition.
   For externalities you cannot call live, assert observable behavior against a realistic boundary double (recorded response, fake, or sandbox), not internal calls.
2. **GREEN** - Make the minimal change to pass: fix type mismatches, stale `select`/`include`, response shapes and frontend types, broken UI references, role-scoped queries, contract/version mismatches, migration safety, cache invalidation, flag wiring, etc.
3. **REFACTOR** - With the suite green, clean up the fix. Never refactor while red.

Repeat per ripple. Each loop leaves a new passing test that would have caught the regression.

## Step 4: Extend & Update Tests

This step is **mandatory** — do not skip it. The goal is adequate automated coverage for every changed code path and externality.

### 4a: Fix broken tests

Update existing tests that now fail due to the changes (wrong assertions, missing properties, changed return shapes, outdated mocks/fixtures). Do NOT delete tests to make the suite pass — fix them to match the new behavior.

### 4b: Extend coverage (internal)

For each changed code path, ensure tests cover: **happy path**, **edge cases** (boundaries, empty/null/undefined, max lengths), **invalid inputs** (rejected with clear errors), **error paths** (network/db/permission failures handled gracefully), and **regressions** (a test reproducing any bug this change fixes).

### 4c: Validation & input guarding

For every entry point touched (API routes, form handlers, queue consumers, webhook receivers, CLI commands): validate inputs at the boundary (Zod/type guards/equivalent), return appropriate errors (400/422) instead of crashing, and add/extend boundary validation tests.

### 4d: Externalities coverage

Add tests for the boundaries surfaced in Step 2 where the change has impact:

- **API contract** tests for endpoints other clients consume (shape + back-compat).
- **Event/webhook payload** tests for producers and consumers, including old↔new shape compatibility.
- **Migration** up/down tests on representative data.
- **Idempotency/retry** tests for operations that can be re-delivered.
- **Config/flag** on-and-off tests so both paths are proven.
- **Cache** hit/miss/invalidate tests.
- **Authz matrix** tests across affected roles.

Follow existing test patterns, frameworks, assertion styles, and mocking approaches. Place tests alongside existing ones. Do not build test infrastructure that serves a single test — keep it inline.

## Step 5: Code Sweep & Declutter

Review all files touched by this change (and immediate neighbours) for cleanup, without straying out of scope:

- **Dead code** - unused imports, unreachable branches, commented-out blocks, orphaned variables, resolved TODOs.
- **Stale references** - old field names, deleted endpoints, deprecated patterns this change supersedes.
- **Consistency** - align naming, structure, and style to the surrounding code.
- **Duplication** - consolidate near-duplicate logic the change introduced or exposed.
- **Simplification** - simplify touched code where behavior is unchanged.

Do NOT refactor unrelated code that was not touched or exposed by this change.

## Step 6: Documentation Updates

Update documentation to reflect the changes, in the project's existing style:

- **DOCUMENTATION.md / API docs** - endpoint tables, models, roles, workflows, architecture, new capabilities; fix any now-incorrect behavior.
- **CLAUDE.md** - add non-obvious gotchas, patterns, or constraints discovered (especially around externalities); update/remove rules this change made outdated; record new key file locations or contracts. Only genuinely hard-to-discover knowledge.
- **README files** - setup, env vars, scripts, new commands, or workflow changes.
- **Memory** - save significant, durable learnings about the project, conventions, or externalities for future sessions.

## Step 7: Full Validation Suite

Run **all** available lints, type checks, tests, and project-specific validation — do not cherry-pick.

### Detect and run project check commands

Find canonical commands via `package.json` scripts, `Makefile`, CI configs, and CLAUDE.md. Run whichever apply to the changed files; if both root and sub-project files were touched, run both.

```bash
# e.g. (check CLAUDE.md for the canonical command)
pnpm check:all
```

### Run the full test suite

```bash
# detect the runner from package.json (pnpm test / bun test / vitest / jest ...)
pnpm test
```

Run unit, integration, and e2e suites if separate. Include externality validation where the project supports it: migration dry-run, contract tests, and any boundary/e2e checks.

### Interpret results

- If any lint, typecheck, or test fails: fix it (test-first) and re-run the full suite. Iterate until green.
- Pre-existing failures (present before your changes): note in the report, do not block on them.
- Unresolvable failures: explain why and flag for human attention.

## Step 8: Report

Provide a concise summary:

- **Changes**: what was changed originally.
- **Ripple effects found**: downstream issues discovered (internal and externalities).
- **Externalities checked**: which boundaries (data/migrations, async, external APIs/contracts, config/flags, cache, observability, security, performance, build/infra) were assessed and the result.
- **Fixes applied**: code fixes made.
- **TDD evidence**: for the key ripples, the red→green progression (failing test added, then passing).
- **Tests updated / added**: which existing tests were fixed and why; what new cases cover.
- **Validation hardened**: input validation added or improved.
- **Cleanup done**: dead code, duplication, or inconsistencies removed.
- **Docs updated**: which files and what changed.
- **Knowledge captured**: rules added to CLAUDE.md or memories saved.
- **Validation results**: final pass/fail of lint, typecheck, and test suites.
- **Human action needed**: migrations, package installs, manual testing, or other items requiring attention.

Do NOT:

- Change the original intent of the modifications.
- Install packages or run migrations.
- Add verbose comments to code (comment why, not what).
- Over-document obvious changes.
- Add tests for code that was not changed or affected by this change.
- Create test infrastructure (helpers, fixtures, factories) that only serves one test — keep it inline.

---
name: grill-me-goal
description: Create research-grounded, test-driven, end-to-end long-running goal packages with a short goal prompt, repo-local docs, 2-10 rounds of extensive batched multiple-choice clarification, codebase/subagent research, internet/API research, and detailed evidence-gated acceptance criteria. Use when the user wants to define, sharpen, persist, resume, audit, or run a multi-session goal that may take many hours or hundreds of hours and must be fully implemented end to end.
---

# Grill Me Goal

## Overview

Turn a vague long-running objective into a durable, repo-local goal package that a fresh agent can execute for hundreds of hours without chat history. The package must be grounded in codebase inspection, external research where relevant, multiple rounds of batched user decisions, and detailed acceptance criteria that are **test-driven, evidence-gated, and end-to-end complete**.

This skill is intentionally output-heavy. Prefer writing structured documentation over keeping reasoning in chat. The default destination is:

```text
docs/<goal-slug>/
  goal.md
  ac.md
  context.md
  research.md
  questions.md
  decisions.md
  risks.md
  test-plan.md
  execution-log.md
```

If the repo already has a stronger planning convention, follow it while preserving this file split.

## Forward Progress Mandate

Every interaction must materially advance the goal package or the goal itself. If the user asks for status, answer briefly and then perform, queue, or identify the next concrete progress action unless the user explicitly asks to pause, stop, or only report status.

Forward progress means one or more of:

- Discovering truth from the repo, docs, tests, CI, external APIs, or primary sources.
- Launching or integrating bounded subagent side missions when available and allowed.
- Asking the next required clarification batch.
- Writing or improving the docs package.
- Completing one A/C with evidence.
- Removing a blocker or documenting a precise unblock path.
- Updating evidence, risks, decisions, or the execution log.

If blocked, record the blocker, add the smallest unblock action to the package, use subagents for side research/verification if available, and continue with the next unblocked useful task. Never treat uncertainty as permission to stop; turn uncertainty into research, questions, assumptions, or A/Cs.

## Three Non-Negotiable Doctrines

Everything this skill produces is shaped by three rules. Bake them into `goal.md`, `ac.md`, and `test-plan.md` so they survive into execution.

### 1. Test-Driven (TDD by default)

Every acceptance criterion that changes behavior is delivered with a failing test first. **Adopt the `/tdd` skill's ethos for all implementation work, and invoke `/tdd` when it is available in the runtime.** The goal package must carry this ethos forward so the executing agent works test-first even in a fresh session.

The core TDD message — embedded here so it travels with the goal package:

> **Tests verify behavior through public interfaces, not implementation details.** Code can change entirely; tests shouldn't. Good tests are integration-style and read like a specification ("user can check out with a valid cart"); they survive refactors. Bad tests couple to implementation — they mock internal collaborators, test private methods, or break on a rename when behavior is unchanged.
>
> **Do not write all tests first, then all code (horizontal slicing).** Bulk-written tests verify *imagined* behavior, test the *shape* of things, and go insensitive to real changes. Work in **vertical slices via tracer bullets**: one test → one implementation → repeat, each test reacting to what the last cycle taught you.
>
> **Loop:** RED (write one failing test → confirm it fails) → GREEN (minimal code to pass) → REFACTOR (improve with tests green). **Never refactor while red.**
>
> **Per cycle:** test describes behavior not implementation · test uses public interface only · test would survive an internal refactor · code is minimal for this test · no speculative features.

Apply it as:

- **Red → Green → Refactor.** Write one failing test that describes the required behavior, confirm it is red, write the minimal code to pass it, then refactor with tests green. Never refactor while red.
- **Vertical slices, not horizontal.** One test → one implementation → repeat. Do **not** write all tests up front then all code.
- **Test behavior through public interfaces**, not implementation details, so tests survive refactors. Avoid mocking internal collaborators or asserting on private state.
- **Pick the critical paths.** You cannot test everything; confirm with the user which behaviors matter most and concentrate tests there, plus the risky/complex logic.
- **Green unit tests are not "done".** TDD here means the red→green→refactor loop **plus** physically verifying the behavior in the running app: for web work, launch it and exercise the real path in a browser; for other runtimes, run the real device/CLI/service. Watch the actual result and capture proof. A passing test suite over an app you never ran is not evidence the feature works.
- A behavior-changing A/C with no associated test is incomplete. Generation-only or research-only A/Cs are exempt, but mark them explicitly as non-behavioral.

### 2. Evidence Before Progression (hard gate)

No box is ticked, and no phase advances, on assertion alone.

- A criterion is checked **only** after its `Evidence` field holds concrete proof: command + output, test run, screenshot, log excerpt, PR/commit link, metric, or a reproducible manual path.
- A **phase gate (PG)** must pass before any work in the next phase begins. A phase gate requires its end-to-end verification to be green with captured evidence. If the gate is red, you stay in the current phase.
- "I believe it works", "should be fine", and "tests will probably pass" are not evidence. Run it.
- Never batch-check criteria. Never delete or soften an unmet criterion to make the goal look done. Obsolete criteria are marked superseded with a link to the replacement.

### 3. End-to-End Completion (always ship the whole vertical)

A feature is done only when it works through the entire stack in a realistic run — never a mocked half.

- Every milestone ends with an **E2E verification gate** that exercises the full user-facing path (e.g. UI → API → service → data store → response, or CLI → process → side effect → observable result) against real or realistically-faked boundaries.
- No "implemented but not wired up", no "happy path only", no stubbed integration left behind as the definition of done. If a seam is temporarily stubbed, an explicit A/C tracks removing the stub before the milestone's E2E gate.
- "Done means" for a feature describes observable end-to-end behavior a human could watch, not "function exists" or "compiles".
- **Physically verify in the running app.** Launch the real app and drive the real user path — click through it in the browser for web work, or run the real device/CLI/service otherwise — and confirm it actually does the thing. Capture proof: a screenshot, screen recording, real HTTP response, or log of the live run. This physical run, not a green unit suite, is what proves a feature is fully implemented end to end.

## Research-First Workflow

1. Locate the repo root and inspect existing docs, plans, tickets, architecture notes, tests, package manifests, CI, and related code before asking the user for details already discoverable locally.
2. If this is a new or under-documented repo with no clear `AGENTS.md`, `CLAUDE.md`, domain language, repo structure, or project principles, run or apply `grill-with-docs` first to establish foundations before writing the long-running goal package. Do not let a hundred-hour goal rest on undocumented foundations.
3. When subagents or delegated workers are available and tool policy/user authorization permits it, **always use multiple parallel side-mission research tracks** for non-overlapping questions. Good tracks include: codebase architecture, existing behavior/tests, domain/API specs, security/ops risk, product/user workflows, and **the current test harness/CI and how features are run end to end**. Keep the main agent responsible for synthesis and forward progress. If subagents are unavailable, perform the same research locally instead of stopping.
4. Do internet research for external APIs, SDKs, protocol specs, platform limits, current best practices, pricing/rate limits, migration guides, compliance constraints, and known failure modes whenever the goal depends on facts that may have changed. Prefer official docs, standards, vendor changelogs, source repos, and primary references. Capture links and access dates in `research.md`.
5. Synthesize what is known into `context.md` and `research.md` before finalizing the acceptance criteria.
6. Design the test and end-to-end strategy in `test-plan.md` (frameworks, how to run tests, how to run the whole app, what an E2E pass looks like per milestone) **before** writing the A/Cs, so each A/C can name its test and verification.
7. Ask **2-10 rounds** of batched multiple-choice questions to resolve decisions that research cannot settle. Each round should contain many useful questions at once; default to 8-20 questions per batch in chat unless the interface constrains length. Use one-at-a-time questioning only when the user explicitly asks for it or when one answer truly determines the next set. Recommendations are advisory only: never silently apply the recommended option, never assume option A, and never default unanswered questions unless the user explicitly says to use recommended defaults.
8. Draft the goal package under `docs/<goal-slug>/`. Keep the short goal prompt in `goal.md`; keep the lengthy checklist in `ac.md`; keep supporting truth in the other files.
9. Audit the package for specificity, test coverage, evidence requirements, end-to-end gates, source grounding, and immediate executability. Revise until a fresh agent can continue from the docs alone.
10. If the user asks to start work, set the long-running goal prompt to point at `docs/<goal-slug>/goal.md`, `docs/<goal-slug>/ac.md`, and `docs/<goal-slug>/test-plan.md`.

## Parallel Research Tracks

Use subagents aggressively for research when allowed. Treat them as bounded side missions that feed the goal, not as a replacement for focused synthesis. Keep tracks independent and ask each researcher for concrete citations, file paths, commands, and uncertainties. Do not ask them for vague opinions.

Default to 3-6 parallel side missions for substantial goals when tools and policy allow it. Hard cap side missions at 8 concurrent agents unless the user explicitly asks for more. Good side missions have a narrow question, an explicit stop condition, a time budget or result budget, and a required artifact to merge into `context.md`, `research.md`, `test-plan.md`, `risks.md`, or `questions.md`. The main agent must stay focused on the current package phase and integrate findings; subagents must not silently redefine the goal or acceptance criteria.

Each side mission prompt must specify:

- Scope: the exact question or file/domain area to inspect.
- Output limit: the maximum number of findings, sources, or files to report.
- Stop condition: what counts as enough research.
- Merge target: which package doc should receive the finding.
- Non-authority: the side mission informs the main agent but does not change scope or mark A/Cs complete.

Useful delegation prompts:

- Codebase map: identify the modules, files, tests, build commands, data stores, jobs, integrations, and ownership boundaries relevant to the goal.
- Existing behavior: trace the current user/system workflows, current edge cases, current test coverage, and known gaps.
- Test & run harness: how tests are written and run, how the app is launched locally, what a real end-to-end run looks like, and where CI gates live.
- External specs: collect official API/SDK/protocol documentation, version constraints, rate limits, auth flows, breaking changes, and implementation examples.
- Risk and operations: identify security, privacy, migration, deployment, rollback, monitoring, and support risks.
- Product acceptance: infer user journeys, success metrics, review checkpoints, and manual QA needs from docs/issues/designs.

When subagents return, merge their findings into the docs with attribution. Resolve conflicts explicitly in `decisions.md` or `questions.md`.

## Internet Research Standards

Browse by default for API specs, SDK behavior, legal/compliance details, pricing, platform limits, current package guidance, or any external fact that may be stale.

For each researched source, capture:

- URL and title.
- Publisher or owner.
- Access date.
- Why it matters.
- Specific facts imported into the goal package.
- Open uncertainty or version caveat.

Prefer primary sources. Use secondary sources only to discover leads or understand ecosystem tradeoffs, and mark them as secondary.

## Batched Multiple-Choice Questions

Question batches should be dense and decision-oriented. Each question should be answerable quickly while still exposing real tradeoffs.

Run multiple rounds:

- Minimum: 2 rounds before finalizing any goal package.
- Maximum: 10 rounds unless the user explicitly asks for more.
- Default: 3-5 rounds for normal substantial goals; 6-10 rounds for ambiguous, high-risk, multi-system, or hundred-hour goals.
- Per round: ask 8-20 questions when possible. Use fewer only when the interface or user requires it.
- Per question: offer 3-5 meaningful options. Add "Other / custom" when the decision space is not closed.
- Recommendations: include a recommended option with a reason, but treat it as advice, not an answer.
- Option ordering: do not always put the recommendation in option A. Distribute recommended options naturally across A/B/C/D/E.
- Unanswered questions: leave them unresolved or ask a follow-up round. Do not assume option A. Do not apply recommended defaults unless the user explicitly authorizes "use recommended defaults".
- Finalization gate: before writing final A/Cs, either every material question is answered, explicitly defaulted by the user, or captured as an unresolved assumption with validation A/Cs.

Use this shape:

```markdown
## Clarification Batch <N>

1. <Decision to resolve?>
   - Why this matters: <impact on scope/A/Cs>
   - A. <Option>
   - B. <Option>
   - C. <Option>
   - D. <Option when useful>
   - Recommended: <A/B/C/D> - <short reason; advisory only, not auto-selected>

2. <Next decision?>
   - Why this matters:
   - A.
   - B.
   - C.
   - D.
   - Recommended:
```

Ask about these areas before finalizing:

- Outcome and completion boundary.
- Users, stakeholders, reviewers, and rejection criteria.
- Non-goals, forbidden changes, and acceptable tradeoffs.
- Target environments, platforms, versions, browsers, devices, accounts, and credentials.
- Data, schemas, migrations, imports/exports, retention, privacy, and compliance.
- External APIs, SDKs, rate limits, auth flows, webhooks, SLAs, and vendor failure behavior.
- UX flows, accessibility, internationalization, performance budgets, observability, and support.
- **Test strategy: frameworks, unit vs integration vs E2E split, coverage expectations, how the app is run for real, and what counts as end-to-end proof.**
- Manual QA, rollout, rollback, monitoring, and incident response.
- Sequencing, milestones, phase gates, dependencies, risks, and stopping rules.

Record every answered question in `questions.md`. Convert important answers into `decisions.md`, `goal.md`, `test-plan.md`, or `ac.md`; do not leave decisions only in chat. For unanswered questions, record `Status: unanswered` and either ask in the next round or create an assumption with a validation A/C. Never record a recommendation as the user's answer.

## Documentation Package

Create this folder unless the project has an established equivalent:

```text
docs/<goal-slug>/
```

Use a short lowercase hyphenated slug based on the goal. Keep paths repo-relative inside the docs, and include absolute paths only when reporting back to the user.

### goal.md

Purpose: concise source of truth for the objective and how to resume.

Include:

- Goal title.
- The goal prompt (see below). It is the single most important artifact this skill produces — craft it deliberately. Keep it tight but complete; it must encode the whole working agreement, not just the objective.
- State: Draft | Active | Blocked | Complete.
- Owner and reviewer.
- Created and last updated dates.
- Outcome in concrete terms.
- Non-goals.
- Required docs in this package.
- Completion rule (must require all phase gates green with evidence).
- Current phase and current focus.

**Goal prompt — get this perfect.** This block is what a fresh agent reads to resume the goal with zero chat history, possibly hundreds of times. It must be unambiguous, self-contained, and leave no room to declare victory early. Tailor `<outcome>` and the app-specific run details to the project; keep everything else. Do not water down the TDD, evidence, or physical-verification clauses.

```text
GOAL: <one-sentence concrete outcome>.

SOURCE OF TRUTH: docs/<goal-slug>/goal.md, ac.md, and test-plan.md. Work only from these files, never from memory or chat history. If reality and the docs disagree, fix the docs first.

HOW TO WORK — non-negotiable:
1. Take exactly ONE unchecked, unblocked A/C at a time, in phase order. Pick the smallest slice that delivers real, user-visible end-to-end behavior. Never start a later phase while the current phase gate is red.
2. Always make forward progress. In every turn, complete one concrete action: research, ask/answer the next clarification batch, implement an A/C, verify evidence, update docs, unblock a dependency, or move to the next unblocked A/C. If blocked, document the blocker and continue on the next useful unblocked task.
3. Use subagents for side missions whenever available and allowed: codebase mapping, API/spec research, risk review, test-harness discovery, and verification audits. Keep those missions bounded and integrate their findings into the docs. The main thread stays focused on the current A/C or package phase.
4. TDD every behavior change. Adopt the /tdd ethos (invoke /tdd if available): RED — write one failing test that describes the behavior and confirm it fails; GREEN — write the minimal code to pass; REFACTOR — clean up with the suite green. Never refactor while red. Test behavior through public interfaces, in vertical slices (one test → one change), never all-tests-then-all-code.
5. TDD is NOT finished when unit tests go green. You must then PHYSICALLY VERIFY the feature in the running app: launch it and drive the real user path yourself — click through it in a browser for web work, or run the real device/CLI/service otherwise — and watch it actually work end to end through the full stack (UI → API → data → back). Capture proof: a screenshot, screen recording, real response, or log of that live run.
6. Wire every feature end to end. No stubbed, mocked, or half-connected seams left as "done". If you must stub temporarily, there is an A/C to remove it before the phase gate.
7. Evidence before ticking. Only check an A/C box after its Evidence field holds concrete proof (the physical run above, test output, command output, links). "Should work" is never evidence — run it.
8. Phase gates. Before advancing a phase, run its end-to-end gate AND the full regression suite, verify the whole flow live in the app, capture evidence, then check the gate.
9. Keep the docs live: update ac.md evidence, execution-log.md (red→green→refactor + the physical run), decisions.md on scope change, and risks.md as risks move.

DONE means: every A/C and every phase gate is checked with evidence, the full suite is green, and the entire feature has been physically run and observed working end to end in the real app. Until then, it is not done — keep going.
```

### ac.md

Purpose: the long executable checklist, organized into phases with gates.

For a many-day or hundred-hour goal, prefer 100-300 acceptance criteria grouped into ordered phases/milestones, each ending in a phase gate. If the goal is smaller, use fewer criteria, but keep them specific and independently verifiable.

Structure:

```markdown
# Acceptance Criteria

## Phase 1 - <name>
... AC items ...
- [ ] PG-1 - Phase 1 end-to-end gate
  ...gate fields...

## Phase 2 - <name>
... AC items ...
- [ ] PG-2 - Phase 2 end-to-end gate
```

Each criterion must use this shape:

```markdown
- [ ] AC-001 - <Specific required end state>
  - Source: <question, code file, research source, decision, or assumption>
  - Rationale: <why this matters>
  - Test-first: <failing test to write first, named + path, or "non-behavioral: <why>">
  - Done means: <observable end-to-end completion condition a human could watch>
  - Verification: <command, test, real run, inspection, metric, screenshot, log, or manual path>
  - Evidence: <fill with concrete proof before checking>
  - Dependencies/blockers: <none or IDs>
  - Notes: <optional>
```

Every phase ends with a phase gate that is also an end-to-end gate:

```markdown
- [ ] PG-1 - <Phase 1 fully works end to end>
  - Covers: <AC range>
  - End-to-end path: <full user/system path exercised, e.g. UI → API → DB → response>
  - Run command: <how to run the real flow, not just unit tests>
  - Verification: <what a pass looks like, including regression suite green>
  - Evidence: <captured run output/screenshot/log before checking>
  - Gate rule: do not start the next phase until this is checked with evidence.
```

Add cross-cutting verification gates where useful:

```markdown
- [ ] VG-001 - <Cross-cutting verification, e.g. full regression / security / perf budget>
  - Required after: <AC range or milestone>
  - Verification:
  - Evidence:
```

Never use vague criteria like "improve UX", "make tests pass", "handle edge cases", "document it", or "clean up code". Replace them with concrete surfaces, expected behavior, measurable thresholds, named files, named tests, commands, or review paths.

### context.md

Purpose: repo-grounded context for future agents.

Include:

- Repo structure relevant to the goal.
- Important files, modules, commands, tests, and CI jobs.
- Current behavior and known gaps.
- Data model, external systems, auth, deployment, and operational context.
- Local conventions to preserve.
- Areas intentionally out of scope.

### research.md

Purpose: external truth and source citations.

Include:

- Research summary.
- Source table with links, access dates, and imported facts.
- API/SDK/platform constraints.
- Version compatibility notes.
- Rate limits, pricing, quotas, SLAs, security/compliance constraints.
- Open research gaps and how to resolve them.

### test-plan.md

Purpose: the test-driven and end-to-end strategy that the A/Cs reference.

Include:

- Test frameworks and runners in use (unit, integration, E2E, browser, load).
- Exact commands to run each test layer and the full suite.
- How to launch and exercise the real application end to end (local run, seeded data, test accounts, fixtures).
- Definition of an end-to-end pass per phase: the user/system path, expected observable result, and how it is captured as evidence.
- Coverage expectations and which critical paths must be tested first.
- TDD conventions for this repo (naming, location, how red is confirmed before green), and a note to adopt the `/tdd` skill ethos (invoke `/tdd` when available) during execution.
- Regression strategy: which suites must stay green at every phase gate.
- What is intentionally not tested and why.

### questions.md

Purpose: user decision history.

Include:

- Batched multiple-choice questions.
- User answers.
- Recommended options clearly marked as advisory, not answers.
- Unanswered questions with `Status: unanswered`, or explicit user authorization to use recommended defaults.
- Assumptions created from unanswered questions, each with a validation A/C.
- Follow-up questions that remain blocked.

### decisions.md

Purpose: durable decisions that affect execution.

Use:

```markdown
- <YYYY-MM-DD> - <Decision>
  - Reason:
  - Sources:
  - Impacted A/Cs:
  - Revisit if:
```

### risks.md

Purpose: risk register and mitigation plan.

Track product, engineering, security, privacy, data, migration, vendor, performance, operational, and schedule risks. Each risk needs likelihood, impact, mitigation, owner, and linked A/Cs.

### execution-log.md

Purpose: resume history for long runs.

Append entries as work proceeds:

```markdown
- <YYYY-MM-DD> - <AC/PG/VG IDs touched>
  - Red: <failing test written + how red was confirmed>
  - Green: <minimal code that passed>
  - Refactor: <what changed, tests still green>
  - End-to-end run: <real run performed + result>
  - Evidence added: <links/output/screenshots>
  - Remaining blockers:
  - Next recommended focus:
```

## Execution Rules

When running work from an existing goal package:

1. Read `goal.md`, `ac.md`, `test-plan.md`, `context.md`, `decisions.md`, `risks.md`, and the latest `execution-log.md` entries before editing code.
2. Identify the current phase. Do not pull A/Cs from a later phase until the prior phase gate is checked with evidence.
3. Select one unchecked, unblocked A/C in the current phase. If several are available, pick the one that unlocks the most downstream work.
4. For a behavior-changing A/C, follow the `/tdd` ethos (invoke `/tdd` if available): write the failing test first and confirm it is **red**, then write minimal code to reach **green**, then **refactor** with tests green — in vertical slices, never refactoring while red. Capture each step.
5. Implement the slice end to end — wire it through the full stack, no orphaned or stubbed-and-forgotten seams.
6. Verify using the stated method, including a real end-to-end run where the A/C is feature-bearing. If verification is weak, strengthen the A/C before checking it.
7. Fill the evidence field with concrete proof: file paths, commands, outputs, test runs, screenshots, PRs, issue links, logs, metrics, or review notes.
8. Tick the checkbox only after evidence exists.
9. When all A/Cs in a phase are checked, run the phase gate: execute the end-to-end path and the regression suite, capture evidence, and only then check the PG and advance.
10. Update `execution-log.md`, `goal.md` current phase/focus, `risks.md` if risk changed, and `decisions.md` if scope changed.

Never batch-check criteria without individual evidence. Never advance past a red phase gate. Never delete unfinished criteria to make the goal look complete. If a criterion becomes obsolete, mark it superseded and link to the replacement.

## Quality Bar

Before handing the package to the user:

- The package exists in `docs/<goal-slug>/` or the repo's established equivalent.
- The goal prompt points to `goal.md`, `ac.md`, and `test-plan.md` and encodes test-first, evidence-before-check, physical in-app/browser verification, and end-to-end completion — with no language that lets a green unit suite count as "done".
- `context.md` reflects actual repo inspection, not guesses.
- `research.md` cites primary sources for external/API facts.
- `test-plan.md` gives runnable commands for every test layer and for a real end-to-end run.
- `questions.md` contains 2-10 rounds of batched multiple-choice questions, with user answers, unanswered decisions, or explicit user-authorized defaults. It must not silently treat recommendations as answers.
- `ac.md` is split into phases, each ending in an end-to-end phase gate, with detailed, test-backed, evidence-gated, independently checkable criteria.
- Every behavior-changing A/C names a test-first test; every milestone has an end-to-end gate.
- The goal prompt forces forward progress, bounded subagent side missions when available, and focus on the current phase/A/C.
- Every major deliverable, edge case, migration, test, doc, rollout, and rollback concern has A/C coverage when relevant.
- The first unchecked A/C is immediately actionable and starts with a failing test where applicable.
- A fresh agent can resume from the docs without chat history.

Finish by showing the user the short goal prompt and the path to the goal package.

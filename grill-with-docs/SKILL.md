---
name: grill-with-docs
description: Research and stress-test a plan against a repo's actual code, domain language, docs, and architectural decisions while updating CONTEXT.md, ADRs, AGENTS.md, CLAUDE.md, and related project docs. Use when the user wants to be grilled on a plan/design, sharpen terminology, establish foundations for a new repo, align agents around project principles, or turn vague intent into durable documented decisions before implementation.
---

# Grill With Docs

## Overview

Run a research-grounded grilling session that turns fuzzy plans into durable project language, principles, and decision docs. This skill borrows the core of Matt Pocock's `grill-with-docs` idea - challenge the plan against the domain model and update docs as decisions crystallize - then adds this repo's ethos: multi-round batched multiple-choice questioning, subagent side research, internet/API research, forward progress, TDD/E2E discipline, and a foundation gate for new repos.

Do not let the conversation remain only in chat. If a term, principle, architectural boundary, or agent rule is resolved, write it into the relevant repo doc while it is fresh.

## Forward Progress Mandate

Every turn must materially improve understanding or documentation. If the user asks for status, answer briefly and then identify or perform the next concrete progress action unless they explicitly ask to pause, stop, or only report status.

Forward progress means one or more of:

- Inspecting code, docs, tests, CI, package manifests, issues, or external primary sources.
- Running bounded subagent side missions when tools and policy allow it.
- Asking the next batched multiple-choice clarification round.
- Updating `CONTEXT.md`, `CONTEXT-MAP.md`, `AGENTS.md`, `CLAUDE.md`, `docs/adr/`, or related docs.
- Capturing a decision, rejected alternative, ambiguity, principle, or test/run command.
- Removing a blocker or documenting the smallest unblock path.

If blocked, record the blocker and continue with the next useful unblocked research or documentation task.

## Workflow

1. Find the repo root and inspect existing docs before asking avoidable questions. Look for `README.md`, `AGENTS.md`, `CLAUDE.md`, `CONTEXT.md`, `CONTEXT-MAP.md`, `docs/`, ADRs, package manifests, CI configs, tests, and source boundaries.
2. If this is a new or under-documented repo, run the New Repo Foundation Gate before implementation planning.
3. Launch bounded side-mission research with subagents when available and allowed. If subagents are unavailable, perform the same research locally instead of stopping.
4. Do internet research for external APIs, SDKs, frameworks, platform limits, compliance constraints, or best practices whenever the plan depends on facts that may be stale. Prefer official docs and primary sources.
5. Ask 2-10 rounds of batched multiple-choice questions. Default to 3-5 rounds for substantial work; use 6-10 for new repos, ambiguous domains, high-risk architecture, or long-running projects.
6. Challenge the plan against actual code, existing docs, domain language, and prior decisions.
7. Update docs inline as terms and decisions stabilize. Do not batch everything until the end.
8. Finish with a short summary of documents changed, unresolved questions, new ADRs, foundation gaps, and the recommended next action. If the result should become a long-running goal package, recommend `grill-me-goal`.

## New Repo Foundation Gate

If the repo is new, empty, sparse, or missing agent guidance, establish foundations before going further. Do not start feature implementation or a long-running goal package until this gate is either complete or explicitly deferred by the user.

The gate applies when any of these are true:

- No `AGENTS.md` or `CLAUDE.md` exists.
- The repo lacks clear language/framework/runtime choices.
- The repo lacks a documented folder structure or module boundary convention.
- The repo lacks test/run commands or TDD/E2E expectations.
- The repo lacks project principles that future agents should preserve.
- The user is creating a repo from scratch or choosing foundations.

Create or update:

- `AGENTS.md` as the cross-agent source of truth.
- `CLAUDE.md` when Claude-specific guidance is needed, preferably linking to `AGENTS.md` to avoid duplicated rules.
- `CONTEXT.md` for domain language when the project has business/domain concepts.
- `docs/adr/0001-*.md` for consequential foundation decisions.

### Foundation Questions

Ask a dedicated foundation batch, usually 8-20 multiple-choice questions, covering:

- Product purpose and non-goals.
- Primary users, workflows, and domain terms.
- Language/runtime/framework/package manager.
- Repo structure: monolith, app/packages, modules, services, boundaries.
- Testing: unit/integration/E2E split, TDD expectations, browser/manual verification, coverage priorities.
- Build/run commands and local dev workflow.
- Data/storage/auth/API/integration foundations.
- Deployment/hosting/CI/observability expectations.
- Style principles: simplicity, explicitness, layering, error handling, validation, dependency policy.
- Agent rules: how future agents should inspect, plan, test, verify, document, and stop.

Recommendations are advisory only. Never assume option A, and never apply recommended defaults unless the user explicitly says to use recommended defaults.

### AGENTS.md Minimum Shape

When creating `AGENTS.md`, include:

```markdown
# Agent Guidance

## Purpose
<What this repo exists to do and who it serves.>

## Foundations
- Language/runtime:
- Framework:
- Package manager:
- Repo structure:
- Data/storage:
- Auth/integrations:

## Project Principles
- <Principle with a concrete implication.>

## Domain Language
- Source of truth: `CONTEXT.md` or `CONTEXT-MAP.md`

## Commands
- Install:
- Develop:
- Test:
- Lint/typecheck:
- E2E/manual verification:

## Working Rules
- Research before changing code.
- Use TDD for behavior changes.
- Verify through public interfaces and real user/system paths.
- Update docs when decisions or terminology change.
- Capture evidence before declaring work complete.

## Documentation
- ADRs live in `docs/adr/`.
- Domain language lives in `CONTEXT.md` or context-local `CONTEXT.md` files.
```

Keep `CLAUDE.md` short if `AGENTS.md` exists:

```markdown
# Claude Guidance

Follow `AGENTS.md` as the source of truth. Claude-specific notes:

- <Only guidance that is genuinely Claude-specific or hard to discover.>
```

## Parallel Research Side Missions

Use subagents aggressively when tools and policy allow it, but keep them bounded. Default to 3-6 parallel side missions for substantial plans; hard cap at 8 concurrent side missions unless the user explicitly asks for more.

Each side mission prompt must specify:

- Scope: exact files, docs, domain, or external source area to inspect.
- Output limit: maximum number of findings, sources, or file paths.
- Stop condition: what counts as enough research.
- Merge target: `CONTEXT.md`, `AGENTS.md`, `CLAUDE.md`, an ADR, or a named doc.
- Non-authority: side missions inform decisions but do not silently decide scope.

Useful tracks:

- Codebase/doc map: repo structure, existing docs, tests, run commands, CI.
- Domain language: terms in docs/code/UI/API and conflicts with `CONTEXT.md`.
- Architecture decisions: current boundaries, irreversible choices, hidden constraints.
- Test/run harness: TDD conventions, E2E paths, physical verification commands.
- External specs: official API/SDK/framework docs, versions, limits, gotchas.
- Risk review: security, data, privacy, ops, migration, performance, rollback.

Merge findings into docs with attribution or citations where useful.

## Batched Multiple-Choice Grilling

Run multiple rounds:

- Minimum: 2 rounds before finalizing consequential docs.
- Maximum: 10 rounds unless the user explicitly asks for more.
- Default: 3-5 rounds for normal substantial plans.
- Per round: ask 8-20 questions when possible.
- Per question: offer 3-5 meaningful options plus "Other / custom" when appropriate.
- Recommendations: include a recommended option and reason, but treat it as advice, not an answer.
- Option ordering: do not always put the recommendation in option A.
- Unanswered questions: keep them unresolved, ask in a follow-up round, or capture as assumptions with validation steps. Do not silently default.

Question areas:

- Terms and boundaries.
- User workflows and concrete scenarios.
- Non-goals and rejected alternatives.
- Architecture, module boundaries, and data ownership.
- Language/framework/runtime/folder structure for new repos.
- Testing, TDD, E2E, physical verification, and evidence.
- External APIs, SDKs, auth, data, privacy, deployment, observability.
- Agent guidance and future-maintenance rules.

## Domain Language Discipline

Treat language as architecture.

- When a user uses a term that conflicts with `CONTEXT.md`, call it out immediately.
- When a term is vague or overloaded, propose a canonical term and alternatives to avoid.
- Stress-test relationships with concrete scenarios and edge cases.
- Cross-reference code and docs. If code contradicts the user's claim, surface the contradiction.
- Update `CONTEXT.md` inline when a term is resolved.

### CONTEXT.md Shape

Use one root `CONTEXT.md` for a single-context repo. Use `CONTEXT-MAP.md` when multiple bounded contexts exist.

```markdown
# <Context Name>

<One or two sentences describing what this context is and why it exists.>

## Language

**<Canonical Term>**
<One-sentence definition of what it is.>
_Avoid_: <ambiguous aliases>

## Relationships

- A **Term** belongs to exactly one **Other Term**.

## Example Dialogue

> **Dev:** "<Question using terms naturally?>"
> **Domain expert:** "<Answer clarifying the boundary.>"

## Flagged Ambiguities

- "<word>" was used to mean both **A** and **B**. Resolution: <decision>.
```

Only include project/domain concepts, not generic programming terms.

## ADR Discipline

Offer ADRs sparingly. Create an ADR only when all three are true:

1. Hard to reverse.
2. Surprising without context.
3. The result of a real trade-off.

ADRs live in `docs/adr/` and use sequential numbering: `0001-slug.md`, `0002-slug.md`.

Keep ADRs concise:

```markdown
# <Short Decision Title>

<1-3 sentences: context, decision, and why.>

## Considered Options
- <Option> - <why rejected or accepted>

## Consequences
- <Non-obvious impact worth remembering>
```

Skip optional sections unless they add real value.

## Documentation Rules

- Create files lazily, but do not delay once a decision is real.
- Patch existing docs instead of duplicating competing sources of truth.
- Preserve local conventions when they exist.
- Do not over-document obvious implementation details.
- Keep durable docs for future agents and engineers, not for summarizing the chat.
- If a decision affects implementation, testing, verification, or agent behavior, capture it where future work will look first.

## Handoff

End with:

- Documents created or changed.
- Terms resolved and ambiguities still open.
- ADRs created or intentionally skipped.
- Foundation gate status.
- Remaining question rounds, if any.
- Recommended next action.

If the plan is ready for long-running execution, recommend creating a `grill-me-goal` package next.

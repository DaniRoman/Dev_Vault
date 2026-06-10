# AGENTS.md

## Purpose

This file defines durable, repository-level working agreements for AI coding agents such as Codex, Claude Code, or any compatible agent that reads project instructions.

Use this file for stable rules that should apply across most tasks in this repository. Do not use it as a task log, chat history, or scratchpad.

## Source of truth order

When working in this repository, follow this priority order:

1. The actual code, tests, schemas, migrations, and runtime configuration.
2. The current task document under `docs/ai/`.
3. This `AGENTS.md` file.
4. Existing project conventions discovered in nearby files.
5. The user's latest explicit instruction.

If there is a conflict, stop and explain the conflict before making destructive changes.

## Core operating principles

- Prefer small, reviewable changes over large rewrites.
- Preserve existing public contracts unless the task explicitly requires changing them.
- Do not remove legacy behavior without documenting the replacement and migration path.
- Do not introduce new production dependencies without a clear reason.
- Do not perform broad formatting-only changes in files unrelated to the task.
- Avoid speculative changes. If evidence is missing, inspect the codebase before deciding.
- Treat tests, type checks, and build output as validation evidence.
- Keep security, backwards compatibility, and operational impact visible.

## Context hygiene

The agent must keep context lean and useful.

Do:
- Read the task document before starting.
- Read the smallest set of relevant source files needed for the phase.
- Summarize relevant findings before editing.
- Update the task document after every code change in the same turn, without asking for permission.
- When updating, re-read the previously written sections (including earlier phases) and rewrite anything that no longer matches the code on disk.
- Record decisions, risks, and validation results.

Do not:
- Paste large code blocks into task documents unless necessary.
- Store long logs, stack traces, or full command outputs in persistent context.
- Duplicate source code in markdown.
- Keep outdated assumptions, planned designs, or proposed helpers in the doc once the real implementation diverges.
- Let the task document drift from the current state of the code, even within the same phase.
- Ask the user whether to update the doc — just update it.
- Advance to another phase without explicit user confirmation.

## Workflow for phased tasks

When a task is documented in `docs/ai/*.md`, follow this workflow.

### Before touching code

1. Read this file.
2. Read the relevant task document under `docs/ai/`.
3. Identify the current phase.
4. Work only on the phase explicitly requested by the user.
5. Summarize:
   - what you understood,
   - files you plan to inspect or modify,
   - known risks,
   - assumptions,
   - expected validation.

If the task document is missing, create it from `docs/ai/task-template.md` before implementation.
See `docs/ai/writing-guide.md` for guidance on filling in each section correctly.

### During investigation

- Trace the affected code flow (controller → service → DB, or API → AMQP → translator).
- Draw a Mermaid `flowchart` or `sequenceDiagram` in the task document to make the flow visible.
- Annotate the diagram with ❌ markers where bugs sit before writing any fix.
- If a bug is found during validation, open a dedicated bug investigation phase before coding the fix.

### During implementation

- Make the smallest safe change that satisfies the phase objective.
- Keep behavior compatible unless the phase explicitly says otherwise.
- Reuse existing helpers, controllers, services, validators, and patterns.
- Prefer extracting shared logic over copy/paste duplication.
- Keep error handling consistent with surrounding code.
- Keep logging useful but not noisy.
- If the phase reveals a larger design issue, document it and stop before expanding scope.
- Update the flow diagram in the task document to reflect the new behavior.

### After implementation

Update the task document with:

- phase status,
- files changed,
- summary of implementation,
- decisions made,
- problems found,
- validation commands executed,
- test results,
- manual verification notes,
- remaining risks,
- next recommended phase.

Do not advance to the next phase without explicit user confirmation.

## Definition of done

A phase is done only when:

- The requested behavior is implemented or the blocker is documented.
- The code builds or the build failure is explained.
- Relevant tests were run or the reason they could not be run is documented.
- The task document is updated.
- The next step is clear.

## Coding standards

### General

- Follow existing project style.
- Prefer explicit names over abbreviations.
- Prefer pure helper functions for reusable branching or normalization logic.
- Keep functions cohesive and reasonably small.
- Avoid hidden side effects.
- Avoid changing unrelated behavior.

### TypeScript / Node.js

- Preserve existing TypeScript target and module conventions.
- Prefer typed interfaces for new cross-module contracts.
- Avoid `any` unless matching existing code or interfacing with untyped data.
- Validate external input at controller/service boundaries.
- Keep async flows readable. Prefer `async/await` for new code unless the surrounding file consistently uses promise chains.
- Do not swallow errors silently. Return compatible errors and log actionable context.
- Preserve existing HTTP status code conventions.

### API compatibility

Before changing an endpoint or response shape, check:

- route path,
- HTTP method,
- auth/privilege checks,
- query parameters,
- request body,
- response shape,
- error status codes,
- frontend compatibility,
- public vs internal API behavior.

If old and new implementations coexist, document the routing or branching rule clearly.

### Data and persistence

- Do not change database schemas, migrations, indexes, or query semantics without explicit scope.
- Validate date, timezone, and range semantics when dealing with time-series data.
- Avoid unbounded queries.
- Document any assumption about data freshness, retention, or external services.

### Observability

For non-trivial changes, preserve or add enough observability to debug:

- identifiers such as device ID, user ID, company ID where appropriate,
- branch taken, such as old-device vs new-device,
- external service failures,
- fallback behavior.

Avoid logging secrets, tokens, credentials, or sensitive payloads.

## Validation strategy

Prefer the strongest available validation in this order:

1. Existing automated tests.
2. TypeScript build.
3. Targeted unit or integration tests.
4. Manual API checks with representative inputs.
5. Static inspection if runtime validation is not possible.

Every phase should document the validation performed.

## Git and review hygiene

- Keep changes scoped to the active phase.
- Avoid mixing refactors with behavior changes unless the phase requires both.
- Make commits or change groups that are easy to review.
- Document why a change was made, not just what changed.
- Leave the repository in a state where another developer can continue from the task document.

## Recommended prompt for phased work

Use this prompt pattern when asking an agent to continue:

```text
Read AGENTS.md and docs/ai/<task-file>.md.
Work only on Phase <N>: <phase name>.
Before editing, summarize what you understood, files to inspect, and risks.
After implementation, update docs/ai/<task-file>.md with phase status, decisions, validation, and next step.
Do not advance to the next phase without confirmation.
```

# Project Instructions

- Always use ASD-STE100 Simplified Technical English.
- Prefer the smallest correct change. Reuse existing code before adding new code, dependencies, or abstractions.
- Inspect the relevant code path, tests, conventions, package versions, and constraints before editing.
- Ask only when the answer materially changes scope, contracts, security, data integrity, accessibility, or user-visible behavior. Otherwise state assumptions and proceed.
- For behavior changes, use test-driven development when practical: show the failure, make the smallest fix, then refactor only if needed.
- For defects, fix the root cause in the shared path and add a focused regression test when feasible.
- Validate trust boundaries. Preserve security, privacy, domain invariants, compatibility, and explicit API or persistence contracts.
- Run focused checks first, then broader build, test, lint, format, analyzer, or accessibility checks proportional to the change.
- Report commands, outcomes, assumptions, residual risks, and untested areas. Do not claim success without evidence.

## Subagents

Use the smallest number of agents needed. Give each delegated task one owner, bounded scope, inputs, constraints, expected output, acceptance criteria, and dependencies. Parallelize only independent, non-overlapping work. The team lead stays accountable for final integration and acceptance.

### Delegation policy

Classify delegated work before you spawn an agent:

- High-complexity work: architecture, ambiguous requirements, security or performance-critical design, cross-component changes, and final arbitration.
- Standard work: normal feature implementation, refactoring, meaningful test development, and code review.
- Economical work: repository search, documentation lookup, mechanical edits, test execution, formatting, and simple verification.

Use the strongest available reasoning profile for high-complexity work. Use the normal development profile for standard work. Prefer cheaper, faster agents and deterministic tools for economical work. Escalate if an agent discovers that the task is materially more complex than first classified.

Available project agents:

- `team-lead`: product architecture, initiative validation, delegation, and final technical review.
- `dotnet-backend`: .NET and ASP.NET Core services, APIs, data access, concurrency, reliability, observability, analyzers, and tests.
- `ui-frontend`: React, TypeScript, browser APIs, semantic HTML, accessibility, Tailwind CSS, UI tests, and frontend performance.
- `senior-qa`: adversarial validation, backend integration tests, Playwright end-to-end tests, release risk, and release recommendation.
- `security-auditor`: security-focused code review, threat analysis, vulnerability detection, and hardening recommendations.

Routing rules:

- Backend and .NET work belongs to `dotnet-backend`.
- React, TypeScript, HTML, accessibility, and Tailwind work belongs to `ui-frontend`.
- Test strategy, adversarial validation, integration tests, and end-to-end tests belong to `senior-qa`.
- Security-focused code review, threat analysis, vulnerability detection, and hardening recommendations belong to `security-auditor`.
- Architecture, scope, cross-cutting decisions, and final review belong to `team-lead`.

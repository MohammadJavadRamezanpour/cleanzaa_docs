# Cleanzza Development Rules

**Status:** Mandatory development contract

**Applies to:** All future backend and frontend implementation, bug fixes,
refactoring, API changes, database changes, feature documentation, and release
work. The Git-flow sections apply only to the backend and frontend repositories,
not to this documentation repository.

This document is the development contract for Cleanzza. Contributors and code
agents must follow it unless an approved repository decision explicitly changes
the rule. A deliberate exception must be documented in the relevant Pull
Request; convenience is not an exception.

`docs/technical_architecture.md` remains authoritative for system architecture,
domain boundaries, security, data handling, and implementation sequencing.
`docs/product_description.md` remains the source product description. If these
documents conflict or an unresolved product decision affects implementation,
stop the affected work, resolve the decision, and update the relevant documents
together. Do not silently choose one interpretation.

## 0. Repository baseline

When this contract was created, the project workspace contained empty `front/`
and `back/` directories and a separately versioned documentation repository at
`docs/`. The workspace root and application directories were not Git checkouts.
The documentation repository tracked `product_description.md` and
`technical_architecture.md` on `main`; it had no `develop` branch, CI
configuration, test tooling, application code, OpenAPI generation command, or
established Pull Request settings visible in the checkout.

The architecture now calls for separate frontend and backend Git repositories,
alongside this documentation repository. Consequently:

- The separate-repository structure and stack in
  `docs/technical_architecture.md` govern initial scaffolding.
- Empty directories are not established implementation conventions.
- The first change that introduces a test runner, formatter, linter, type
  checker, migration command, or OpenAPI workflow must document the canonical
  local command and use the same command in CI.
- Do not introduce competing tools or duplicate workflows without an approved
  reason.
- The Git rules below become enforceable for the backend and frontend
  repositories when they and the permanent branches are created. They do not
  require this documentation repository to create a `develop` branch or adopt
  the same release flow.
- Until the necessary branches and hosting settings exist, do not claim that
  branch, PR, protection, review, or CI requirements have been satisfied.

---

## 1. TDD — Test-Driven Development

Use Test-Driven Development whenever practical.

For new functionality or behavior changes:

1. Understand the requirement and expected behavior.
2. Define or update the expected contract.
3. Write or update the relevant tests first.
4. Run the tests and confirm they fail for the expected reason.
5. Implement the simplest correct solution.
6. Run the tests and confirm they pass.
7. Refactor while keeping the tests green.
8. Run the relevant test suite again before completing the work.

Tests must verify observable behavior, public contracts, and business
invariants rather than incidental implementation details.

Do not add meaningless tests merely to satisfy this rule. Do not skip tests for
normal feature, bug-fix, or behavior changes. When fixing a bug, add a
regression test that fails for the original defect whenever practical.

If test-first development is genuinely impractical—for example, a disposable
investigation used to discover an external provider's behavior—explain why,
avoid shipping the investigation as production code, and add the appropriate
tests before treating the implementation as complete.

For Cleanzza, concurrency, authorization, money, state transitions, file
access, webhooks, idempotency, scheduling, and account restrictions require
integration tests against PostgreSQL or the relevant provider boundary; mocked
unit tests alone are not sufficient.

---

## 2. Feature Documentation

Every implemented feature must have a corresponding document under:

```text
docs/features/
```

Use a descriptive kebab-case filename, for example:

```text
docs/features/user-authentication.md
```

Feature documentation must contain the relevant information, including where
applicable:

- purpose and scope
- implemented requirements and explicit non-goals
- expected behavior and state transitions
- API endpoints
- request and response contracts
- parameters and validation rules
- error responses and relevant HTTP status codes
- authentication, authorization, and ownership requirements
- idempotency and concurrency behavior
- important edge cases
- examples
- database invariants and asynchronous effects
- sensitive-data and retention considerations
- relevant architectural decisions
- canonical verification commands

Documentation is a first-class part of feature development.

If the feature document does not exist, create it. If an implemented feature
changes, update the corresponding document in the same change. Never knowingly
leave feature documentation out of sync with the implementation.

Do not create feature documentation for hypothetical features. Architecture,
product proposals, and unresolved decisions belong in architecture documents
or ADRs, not in `docs/features/` as though they were implemented.

---

## 3. API Contracts

API contracts must be explicit and synchronized with the implementation.

For every API created or modified:

- document the method and path
- document path, query, header, and body parameters
- document validation and normalization
- document response schemas
- document relevant success and error status codes
- document machine-readable error codes
- document authentication, roles, ownership, and policy checks
- document idempotency requirements
- document pagination, ordering, and filtering where applicable
- provide examples where useful
- update the relevant feature document

The contract must describe actual behavior, not an aspirational or future API.
Avoid undocumented API behavior.

Follow the shared API conventions in `docs/technical_architecture.md`, including
the standard error envelope, decimal-string money values with currency,
explicit-offset ISO 8601 timestamps, cursor pagination, and `Idempotency-Key`
requirements for retryable one-time or financial commands.

Breaking changes require an explicit compatibility decision, migration plan,
and API-versioning treatment before implementation.

---

## 4. Swagger / OpenAPI

Swagger/OpenAPI must be generated or updated as API development happens.

For this FastAPI application, route definitions and Pydantic schemas are the
canonical OpenAPI generation source unless an ADR approves a different single
source of truth. Do not maintain an unrelated hand-written specification in
parallel.

Whenever an API is created or modified:

1. Update or define the expected API contract.
2. Update the implementation and schemas.
3. Update the relevant feature documentation.
4. Generate or validate the OpenAPI document using the repository's canonical
   command.
5. Verify that paths, schemas, security, status codes, and examples match the
   actual implementation.
6. Run any generated-client or contract-diff check established by the
   repository.

OpenAPI work is part of the feature and must not be postponed as cleanup. The
backend repository publishes or exports the versioned OpenAPI contract. The
frontend repository pins a compatible contract or generated TypeScript client.
CI must fail when the frontend client is stale against its pinned contract.
Breaking API changes require a compatible rollout across both repositories.

Do not introduce a second API documentation or generation mechanism without a
clear, documented requirement.

---

## 5. Simplicity First

Prefer the simplest correct solution over a clever, flexible, or speculative
solution.

Before implementing a complex approach, determine whether a simpler one
satisfies the current requirement. If it does, use the simpler approach.

Avoid:

- premature abstractions and generalization
- unnecessary design patterns, interfaces, factories, and dependency injection
- unnecessary configuration and dependencies
- duplicate functionality
- over-engineered architecture
- speculative features
- frameworks or libraries without meaningful current value

Do not build for hypothetical requirements. Follow the V1 non-goals in
`docs/technical_architecture.md`: in particular, do not introduce
microservices, Kubernetes, Kafka, GraphQL, event sourcing, CQRS, Elasticsearch,
ML matching, or WebSockets without a concrete approved requirement.

---

## 6. Clean Code and Practical SOLID

Write clean, maintainable, readable, and testable code. Apply SOLID principles
where they provide practical value.

- Keep functions, classes, and modules cohesive.
- Keep responsibilities focused.
- Minimize coupling and make dependencies explicit.
- Prefer straightforward control flow.
- Prefer composition where appropriate.
- Use meaningful domain names.
- Avoid unnecessary side effects.
- Make business rules independently testable.

Do not apply SOLID mechanically. Never add an interface, factory, repository,
service, or architectural layer merely to demonstrate a pattern.

Respect the modular-monolith boundaries:

- Routers handle HTTP concerns only.
- Services/application operations own business rules and transactions.
- Repositories perform database queries and never decide business policy or
  commit transactions.
- Policies enforce authorization and domain eligibility where useful.
- A module must not query another module's repository or tables directly.
- Cross-module behavior uses public application-service interfaces or internal
  domain events.
- Cyclic module dependencies are forbidden.
- Provider-specific code stays under the integrations boundary.

The backend and PostgreSQL are authoritative for authorization, money, order
state, offers, eligibility, availability, scoring, complaints, restrictions,
and concurrency. React state, Redis, and cached values are never authoritative
for those rules.

---

## 7. Reuse Existing Code and Conventions

Before creating a utility, helper, service, module, dependency, abstraction, or
architectural pattern:

1. Inspect the existing codebase.
2. Search for equivalent functionality.
3. Determine whether existing code can be reused or safely extended.
4. Follow established naming, structure, error, test, and API conventions.
5. Prefer consistency with the architecture.

Do not duplicate functionality unnecessarily. Do not introduce a new pattern
when an existing one adequately solves the problem.

When the repository lacks a convention, choose the smallest conventional
approach supported by the selected stack, document it in the first applicable
change, and use it consistently. A new convention that materially affects
architecture must be recorded as an ADR under `docs/adr/`.

---

## 8. Keep Changes Focused

Make the smallest reasonable change that completely satisfies the requirement.
Do not perform unrelated refactoring during feature work unless it is necessary
to implement the feature safely.

Avoid scope creep. If a larger refactor is required, identify the reason and
impact before expanding the change. Do not rewrite working code merely because
another implementation is personally preferable.

Preserve user changes and unrelated work already present in the working tree.
Do not revert, overwrite, or reformat unrelated files.

Do not use a feature request to silently resolve an open product, legal,
financial, privacy, or safety decision. Follow the gates in Section 1.1 of
`docs/technical_architecture.md`.

---

# Git Flow

Sections 9–19 govern the backend and frontend code repositories only. They do
not prescribe branch names, permanent branches, or merge strategy for the
documentation repository. When an implementation requires a feature-document
change in the separate documentation repository, make that documentation
change in the same logical delivery and link the code and documentation changes
to each other.

## 9. Permanent Branches

Each backend and frontend repository uses two permanent branches:

- `main` — production-ready code
- `develop` — active development and integration

`main` must always contain production-ready code. `develop` is the integration
branch for normal development.

Normal development must never be performed directly on `main` or `develop`.
All changes must go through dedicated branches and Pull Requests.

## 10. Feature Branches

Create feature branches from the latest `develop` branch.

```text
feature/<short-description>
```

Example:

```text
feature/user-authentication
```

Flow:

```text
develop
  ↓
feature/user-authentication
  ↓
Pull Request → CI / checks → review → Squash and Merge
  ↓
develop
```

Feature branches must not be merged directly into `main`.

## 11. Fix Branches

Create normal bug-fix branches from the latest `develop` branch.

```text
fix/<short-description>
```

Example:

```text
fix/invalid-user-validation
```

Normal fixes target `develop` through a Pull Request and must not be merged
directly into `main`.

## 12. Pull Requests

Every backend or frontend change targeting `develop` or `main` must go through
a Pull Request. Direct pushes to permanent code branches are not allowed during
normal development.

A Pull Request must not be merged until every required check passes. Depending
on the affected code and configured CI, checks include:

- unit, integration, and end-to-end tests
- linting and formatting
- frontend and backend type checking
- production builds
- security and secret scanning
- OpenAPI and generated-client contract checks
- database migration/schema checks
- any other protected-branch requirement

Never bypass or disable a failing check to make a Pull Request mergeable.
Investigate and fix the underlying issue. If unrelated infrastructure prevents
validation, report the exact failure and leave the work explicitly incomplete.

Pull Requests must explain the requirement, implementation, tests performed,
documentation and API-contract changes, migrations, security/privacy impact,
deployment risk, and any remaining decisions or follow-up work.

## 13. Squash and Merge

Feature and fix Pull Requests targeting `develop` must use **Squash and Merge**.
This keeps each completed logical change as one commit on the integration
branch.

Do not use regular merge commits unless an approved repository rule explicitly
requires them. Use the same squash-and-merge principle for release and hotfix
Pull Requests where supported.

## 14. Releases — Develop to Main

When `develop` is ready for production, create a Pull Request from `develop` to
`main`.

The release Pull Request must pass all required checks, satisfy review and
approval requirements, and be squash-merged. `main` must remain
production-ready.

```text
feature/* ──→ develop
fix/* ──────→ develop
                ↓
          release PR and checks
                ↓
              main
```

## 15. Hotfixes

Production hotfixes are the exception to normal development flow. Create a
hotfix from the latest `main`:

```text
hotfix/<short-description>
```

Example:

```text
hotfix/payment-timeout
```

The hotfix must pass required checks and review before being squash-merged into
`main`. Then propagate the same fix to `develop` through a second Pull Request,
which must also pass required checks and be squash-merged.

```text
             ┌──→ PR → checks → review → squash → main
main → hotfix
             └──→ PR → checks → review → squash → develop
```

Never create a production hotfix from `develop`.

## 16. Branch Summary

| Work | Branch from | PR target | Merge strategy |
| --- | --- | --- | --- |
| Feature | `develop` | `develop` | Squash and Merge |
| Normal bug fix | `develop` | `develop` | Squash and Merge |
| Release | `develop` | `main` | Squash and Merge |
| Production hotfix | `main` | `main` | Squash and Merge |
| Hotfix propagation | hotfix changes based on `main` | `develop` | Squash and Merge |

All Pull Requests require every configured required check to pass before merge.

## 17. Branch Protection

Protect `main` and `develop` where repository settings allow it:

- disable direct pushes
- require Pull Requests
- require status checks
- require approvals
- disable force pushes
- disable deletion of permanent branches
- require branches to be up to date where appropriate

Contributors and code agents must respect branch protection and must never
attempt to bypass it.

---

# Development Workflow

## 18. Standard Feature/Fix Workflow

For normal feature or bug-fix work:

```text
Requirement
    ↓
Inspect existing code, documentation, tooling, and conventions
    ↓
Create the correct branch from the latest develop
    ↓
Define or update feature documentation and API contract
    ↓
Write/update tests and confirm the expected failure
    ↓
Implement the simplest correct solution
    ↓
Refactor while tests remain green
    ↓
Synchronize OpenAPI, generated artifacts, and documentation
    ↓
Run all relevant validation
    ↓
Open Pull Request
    ↓
Wait for every required check and approval
    ↓
Squash and Merge into develop
```

Do not declare repository work complete before applicable tests,
documentation, API contracts, migrations, observability, operational handling,
and CI checks are addressed.

If the acting contributor or code agent is not authorized to create branches,
push, open a Pull Request, approve, or merge, complete the authorized local work
and clearly report the remaining delivery steps. Never claim that unperformed
Git or CI actions occurred.

## 19. Hotfix Workflow

For a production hotfix:

```text
Production issue
    ↓
Create hotfix branch from latest main
    ↓
Write/update regression tests and confirm the expected failure
    ↓
Implement the simplest correct fix
    ↓
Update feature documentation, API contract, and OpenAPI where applicable
    ↓
Run all relevant validation
    ↓
PR → main → checks → review → Squash and Merge
    ↓
PR → develop → checks → review → Squash and Merge
```

---

# Requirements and Decision Making

## 20. Ambiguous Requirements

Do not invent unnecessary requirements.

When a requirement is ambiguous:

1. Follow approved architecture and existing repository conventions.
2. Prefer the simplest interpretation consistent with the requirement.
3. Avoid speculative functionality.
4. Ask for clarification when ambiguity materially affects architecture, API
   contracts, data models, security, privacy, money, account restrictions,
   user-visible behavior, backward compatibility, or production safety.

When ambiguity does not materially affect implementation, make the smallest
reasonable decision, document it where future contributors need it, and
proceed.

Unresolved items listed in Section 1.1 of `docs/technical_architecture.md` are
explicit gates, not invitations to select a convenient interpretation.

## 21. Definition of Done

A feature or change is complete only when every applicable condition is met:

- The requirement is implemented without unrelated changes.
- Tests were written or updated and all relevant tests pass.
- Important bug fixes have regression tests.
- Concurrency-sensitive behavior has realistic integration coverage.
- The implementation is clean, maintainable, and consistent with module
  ownership.
- Existing behavior has not been unintentionally broken.
- The corresponding `docs/features/*.md` document is created or updated.
- API contracts and authentication/authorization behavior are documented.
- Swagger/OpenAPI and generated clients are synchronized for API changes.
- Database changes have reviewed migrations and a safe rollout plan.
- Retryable commands and asynchronous effects implement and test idempotency.
- Sensitive-data access, retention, and audit requirements are addressed.
- Operational/admin handling exists when manual intervention may be required.
- Logging, metrics, and alerts cover important failure paths.
- No unnecessary complexity or speculative behavior was introduced.
- Canonical lint, format, type-check, build, and test commands pass.
- The change is on the correct branch.
- Required Pull Request checks and approvals pass.
- The Pull Request uses the required squash-and-merge strategy.

Local implementation may be ready for review before the last three Git-hosting
conditions are performed, but it is not delivered or merged until they are
complete.

## 22. Core Principle

Prioritize engineering decisions in this order:

1. Correctness
2. Simplicity
3. Security and privacy
4. Testability
5. Maintainability
6. Clear API contracts
7. Accurate documentation
8. Consistency with the existing codebase

Build only what is required. Verify behavior with meaningful tests, keep API
and feature documentation synchronized, preserve the modular architecture, and
avoid unnecessary complexity.

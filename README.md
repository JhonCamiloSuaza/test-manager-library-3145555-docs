# Manager Library — Technical Assessment

> **SDD technical assessment · ADSO group 3145555**
>
> Schedule: **Wednesday, October 14, 2026, from 6:50 a.m. to 11:00 a.m.** (Colombia time). Submission deadline: **11:00 a.m.**

This repository contains the specifications for **Manager Library**. The product manages a small library's catalog, members, loans, returns, overdue items, and most-borrowed report.

## Canonical implementation specification

The assessment must be implemented with **React** and **PHP/Laravel**. The canonical specification for that implementation is [`spec-laravel/`](spec-laravel/).

Read it in this order:

1. [`constitution.md`](spec-laravel/constitution.md) — non-negotiable engineering principles.
2. [`spec.md`](spec-laravel/spec.md) — actors, scope, user stories, acceptance criteria, and business rules.
3. [`plan.md`](spec-laravel/plan.md), [`architecture.md`](spec-laravel/architecture.md), and [`architecture-overview.md`](spec-laravel/architecture-overview.md) — implementation strategy and system boundaries.
4. [`data-model.md`](spec-laravel/data-model.md) and [`api-contract.md`](spec-laravel/api-contract.md) — authoritative database model and HTTP contract.
5. [`tasks.md`](spec-laravel/tasks.md) — implementation order and executable evidence.
6. [`adr/`](spec-laravel/adr/) — accepted architecture decisions and documented specification conflicts.

The existing [`spec-python/`](spec-python/) and [`spec-.net/`](spec-.net/) directories are reference specifications. They describe the same product but are not the assessment stack. They remain unchanged for traceability.

## Product repositories

| Repository | Responsibility |
|---|---|
| `test-manager-library-3145555-docs` | Canonical and reference specifications |
| `test-manager-library-3145555-api` | PHP/Laravel backend |
| `test-manager-library-3145555-app` | React operations application |
| `test-manager-library-3145555-page` | Static public presentation site; it does not call the API |
| `test-manager-library-3145555-infra` | Containers, network, volumes, and empty PostgreSQL engine |
| `test-manager-library-3145555-tool` | Demonstration data seeder that uses the API |

## Delivery rules

- Work in the six personal forks.
- Keep `main` as the delivery branch and use short-lived branches with pull requests.
- Use small, focused commits and push partial, verifiable progress frequently.
- Code, identifiers, commit messages, and technical documentation are written in English.
- User-facing messages remain in Spanish as required by the product contract.
- Do not rename or add product tables, columns, request fields, or response fields.
- Only work pushed before the deadline is evaluated.

## Scope protection

The Laravel specification translates technology, not product behavior. Fines, payments, reservations, renewals, email or SMS, bulk import, inter-branch loans, and additional book attributes are explicitly out of scope.

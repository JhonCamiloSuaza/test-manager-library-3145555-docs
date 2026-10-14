# Constitution — Manager Library (React + Laravel)

These principles are non-negotiable. Each article includes executable evidence because an unverifiable principle is only an intention. A change that violates an article must be rejected before the rest of the change is reviewed.

## Article I — The domain depends on nothing

`app/Domain/` must not import Laravel, Eloquent, HTTP, database, storage, authentication, or other project layers. It contains plain PHP objects and domain exceptions only.

**Evidence:** architecture tests fail when `Domain` depends on `Application`, `Infrastructure`, `Interfaces`, or Laravel framework types. Domain tests run without booting Laravel or PostgreSQL.

## Article II — The core owns its ports

The application layer may depend only on the domain. Every external capability it needs—repositories, clock, password hashing, token issuing, cover storage, and report reading—is expressed as a small interface owned by `app/Application/Ports`.

No Eloquent model, request, response, filesystem, or framework type appears in a port signature.

**Evidence:** application tests use fakes and run without importing infrastructure adapters.

## Article III — There is one composition root

`app/Providers/AppServiceProvider.php` is the single location where application ports are bound to infrastructure adapters. Infrastructure implementations must not be instantiated elsewhere.

**Evidence:** an architecture test identifies adapter construction outside the composition root.

## Article IV — SOLID is executable

| Principle | Concrete application | Evidence |
|---|---|---|
| SRP | One application service represents one use case | A service with two reasons to change is split |
| OCP | Replacing local cover storage with S3 does not modify domain or application code | A new adapter requires only a composition binding |
| LSP | Test fakes can replace production adapters | The same port contract tests pass for both |
| ISP | Ports remain capability-specific | Implementations do not contain unused methods |
| DIP | The consumer owns the abstraction | Port interfaces live in `Application` |

## Article V — Migrations own the schema

The `infra` repository starts an empty PostgreSQL 16 engine. All DDL—schema, tables, indexes, constraints, and the five fixed categories—belongs to Laravel migrations in the `api` repository.

`infra` must not contain `CREATE TABLE`, and the application must not use automatic schema creation on startup.

## Article VI — The database enforces expressible invariants

Rules expressible as `CHECK`, `UNIQUE`, or foreign-key constraints must be enforced by PostgreSQL. Rules that cannot be expressed by a row constraint remain in the domain and application transaction.

The authoritative classification is in [`data-model.md`](data-model.md).

## Article VII — Derived values are calculated

Loan `status` and `days_late` are calculated from `due_at`, `returned_at`, and the injected `Clock`. They are never database columns.

## Article VIII — Frozen values do not follow the live catalog

A loan item stores the book title, ISBN, and category name as they existed at lending time. Historical responses and reports use these frozen values rather than joining to the current catalog values.

## Article IX — Secrets never enter the repository or logs

Passwords, precomputed password hashes, JWT secrets, and production credentials come from the environment. `.env` is ignored and only `.env.example` is versioned.

HTTP responses and logs must never contain `password`, `password_hash`, tokens, SQL, stack traces, or server paths.

## Article X — The engine wins

[`data-model.md`](data-model.md) is the authoritative schema contract. The implementation must preserve every documented table name, column name, type, length, nullability rule, foreign key, and index. No convenience attributes or framework-default columns may be added.

## Article XI — Language

Code, database identifiers, HTTP fields, technical documentation, branch names, and commit messages are written in English. Product error messages and other user-facing messages are written in Spanish. The five fixed category names remain exactly as specified.

## Article XII — Nothing is complete without evidence

A task in [`tasks.md`](tasks.md) is complete only when its stated evidence exists and can be executed. Implemented code without its required evidence remains incomplete.

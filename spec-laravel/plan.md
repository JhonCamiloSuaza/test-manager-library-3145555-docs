# Implementation Plan — Manager Library (React + Laravel)

This plan translates the accepted product specification to the assessment stack without changing product scope or schema.

## 1. Technical decisions

| ID | Concern | Laravel implementation | Reason |
|---|---|---|---|
| D-01 | Backend | PHP with Laravel; exact versions are locked in `composer.lock` | Required assessment stack and reproducible dependencies |
| D-02 | Frontend | React; exact versions are locked in the package lockfile | Required assessment stack and reproducible dependencies |
| D-03 | Persistence | Laravel migrations and a PostgreSQL adapter using Eloquent/query builder | Migrations remain the sole schema owner |
| D-04 | Concurrency | Conditional inventory updates in a transaction plus `ck_book_available_range` | Preserves the exact schema and final-copy behavior |
| D-05 | Time | UTC instants behind an application-owned `Clock` port | Deterministic overdue tests |
| D-06 | Report | One aggregate PostgreSQL query behind `ReportReader` | Stable frozen data and efficient execution |
| D-07 | Value objects | `Isbn`, `Quantity`, `DocumentNumber`, and `DateRange` live inside their owners | Values have no independent identity or table |
| D-08 | Covers | Opaque key in PostgreSQL; local disk adapter in development and replaceable storage adapter | Binary data remains outside the database |
| D-09 | Passwords | Argon2id behind `PasswordHasher` | Strong one-way hashing without domain coupling |
| D-10 | Bootstrap admin | Idempotent startup service reads `ADMIN_USERNAME` and `ADMIN_PASSWORD` | The database cannot create a password hash itself |
| D-11 | Authentication | HS256 JWT with an 8-hour lifetime behind `TokenIssuer` | Stateless role authentication required by the contract |
| D-12 | Tests | PHPUnit tests with fakes; PostgreSQL only for integration and concurrency suites | Fast core tests and real database evidence |

## 2. Environment configuration

| Variable | Required | Purpose |
|---|---|---|
| `DATABASE_URL` | yes | PostgreSQL connection |
| `JWT_SECRET` | yes | JWT signing secret |
| `ADMIN_USERNAME` | yes | Initial administrator username |
| `ADMIN_PASSWORD` | yes | Initial administrator clear text, consumed only for hashing |
| `COVERS_DIR` | no | Local cover-storage root |
| `CORS_ORIGIN` | yes | Only permitted frontend origin |

`.env.example` documents names and safe placeholders. Real values are never committed.

## 3. Test strategy

| Level | Responsibility | Dependencies |
|---|---|---|
| Domain | Value objects and aggregate invariants | Plain PHP only |
| Application | Authentication, catalog, lending, returning, overdue, and report use cases | Fake ports and fake clock |
| Integration | Migrations, repositories, indexes, constraints, and report query | PostgreSQL 16 |
| Contract | Routes, roles, fields, common errors, and status codes | Running API |
| Architecture | Dependency direction and single composition root | Static/reflection-based tests |
| Concurrency | Two requests for the final available copy | PostgreSQL 16 and running API |
| Frontend | Critical React flows and API error rendering | Mocked and running API as appropriate |

## 4. Construction order

1. Canonical Laravel documentation and traceable decisions.
2. Laravel skeleton, configuration validation, and `GET /health`.
3. Exact schema migration and five fixed categories.
4. Framework-independent domain with unit tests.
5. Application ports and use cases with fakes.
6. Persistence, security, storage, and clock adapters.
7. HTTP contract, common errors, JWT, and role authorization.
8. Bootstrap administrator and executable API evidence.
9. Docker Compose environment.
10. React operations application.
11. Demonstration seeder through the API.
12. Static public presentation site.

Detailed dependencies and evidence are defined in [`tasks.md`](tasks.md).

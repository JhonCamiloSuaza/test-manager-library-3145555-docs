# Implementation Tasks — Manager Library (React + Laravel)

A task is complete only when its evidence exists and can be executed. Every task starts as **pending**.

| ID | Task | Depends on | Executable evidence |
|---|---|---|---|
| T-01 | Laravel skeleton, environment validation, and `GET /health` | — | `GET /health` returns `{"status":"ok"}` and the API container starts |
| T-02 | Initial migration: `library` schema, exactly six tables, required FKs/indexes/checks, and five fixed categories | T-01 | Section 11 queries in `data-model.md` match exactly; no column has a default |
| T-03 | Value objects: `Isbn`, `Quantity`, `DocumentNumber`, `DateRange` | T-01 | Unit tests cover 13-digit ISBN, positive quantity, required document, and non-inverted range |
| T-04 | `Book` domain: withdraw, restock, total adjustment, and logical deletion | T-03 | Unit tests prove availability never leaves `0..total` |
| T-05 | `Loan` domain: add item, return item, calculated status, 14-day due date, and 1..3 items | T-04 | Fake-clock tests cover `open`, `overdue`, and `closed` |
| T-06 | `Member` and `User` domain behavior | T-03 | Tests prove `"Ana "` becomes `ana` and API creation cannot request `admin` |
| T-07 | Authentication ports and `LoginService` / `CreateLibrarianService` | T-06 | US-01 and US-02 pass with fakes |
| T-08 | Book create, search, replace, logical delete, and cover use cases | T-04 | US-03 and US-04 pass with fakes |
| T-09 | `LendBooksService` | T-05, T-08 | US-06 tests cover duplicates, limits, overdue members, unavailable books, and rollback |
| T-10 | Return-item and overdue use cases | T-09 | US-07 and US-09 pass; repeated return produces `ITEM_ALREADY_RETURNED` |
| T-11 | Most-borrowed report service and `ReportReader` port | T-09 | US-10 passes; renaming a book does not change its historical report row |
| T-12 | PostgreSQL adapters for books, members, and users | T-02, T-08 | Repository integration tests pass against PostgreSQL 16 |
| T-13 | PostgreSQL loan adapter and conditional inventory concurrency | T-12, T-09 | Two simultaneous final-copy requests produce exactly one 201 and one 409 |
| T-14 | Password hashing and JWT adapters | T-07 | Security test and log review prove secrets never appear in responses or logs |
| T-15 | Local cover-storage adapter | T-08 | Content-valid 2 MB PNG succeeds; 3 MB fails with 422; missing cover key remains null |
| T-16 | Every HTTP route, common error mapper, authentication, and role authorization | T-10, T-11, T-14, T-15 | Contract suite covers all routes and business error codes in `api-contract.md` |
| T-17 | Composition root and idempotent bootstrap administrator | T-16 | Empty engine can be migrated and login succeeds using environment admin credentials |
| T-18 | Add `ck_book_total_positive` and `ck_user_role` | T-12 | Invalid direct inserts fail and `data-model.md` classifications are updated |
| T-19 | Architecture rules for Articles I–III | T-17 | Tests pass and demonstrably fail for a forbidden dependency or adapter construction |
| T-20 | `infra`: PostgreSQL, API, and React app through Docker Compose | T-17 | One command starts services; PostgreSQL begins without application tables before migration |
| T-21 | `tool`: API-only data seeder with at least 40 books, 10 members, 15 loans, and 3 overdue items | T-17, specification clarification | After execution, overdue returns 3 items and the report contains rows |
| T-22 | `app`: login, catalog, members, lending, returning, overdue list, and report | T-16 | End-to-end demonstration completes every operational flow |
| T-23 | `page`: static public presentation site | — | Opens without the API and renders correctly on mobile and desktop |

## Core completion threshold

T-01 through T-17 form the product core. T-18 through T-23 distinguish a complete implementation from the core. Evidence is valued over partially implemented task count.

## T-21 specification blocker

The published create-loan contract accepts only `member_id` and `items`; the server owns `loaned_at`. Therefore, a tool that exclusively uses the published API cannot create immediately overdue items. T-21 must not be marked complete until the product owner selects a solution. The implementation must not add `loaned_at` to the request, create a hidden endpoint/header, or write SQL directly without that decision. See [ADR-006](adr/adr-006-demo-overdue-seeding.md).

## Requirement traceability

| Requirement | Primary tasks | Evidence type |
|---|---|---|
| US-01, US-02 | T-06, T-07, T-14, T-16, T-17 | Application, security, and contract tests |
| US-03, US-04 | T-03, T-04, T-08, T-12, T-15, T-16 | Domain, integration, storage, and contract tests |
| US-05 | T-06, T-12, T-16 | Application, integration, and contract tests |
| US-06 | T-05, T-09, T-12, T-13, T-16 | Domain, transaction, concurrency, and contract tests |
| US-07 | T-04, T-05, T-10, T-13, T-16 | Domain, transaction, and contract tests |
| US-08 | T-05, T-12, T-16 | Fake-clock, integration, and contract tests |
| US-09 | T-10, T-12, T-16 | Fake-clock, query, and contract tests |
| US-10 | T-11, T-12, T-16 | Frozen-value, aggregate-query, and role tests |
| NFR-1 | T-12, T-21 | Load test with 5,000 books |
| NFR-2 | T-13 | Real PostgreSQL concurrency test |
| NFR-3 | T-14, T-16 | Log and response security review |
| NFR-4 | T-16 | Full error-contract suite |
| NFR-5 | T-17, T-20 | Clean startup demonstration |

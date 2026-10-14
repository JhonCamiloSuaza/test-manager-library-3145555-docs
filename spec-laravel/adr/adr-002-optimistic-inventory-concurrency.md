# ADR-002 — Optimistic concurrency without an additional column

- **Status:** Accepted for the Laravel translation
- **Context:** Two librarians may request the final available copy simultaneously. The reference ADR mentions a version witness, but the authoritative physical model does not contain a `version` column and forbids undeclared attributes.
- **Decision:** The PostgreSQL adapter performs conditional inventory writes using the documented fields inside the loan transaction. An update succeeds only while the required copy remains available; an affected-row conflict becomes 409 `BOOK_NOT_AVAILABLE`. The `ck_book_available_range` check is the final barrier.
- **Rejected alternatives:** Adding a `version` column; pessimistically locking every catalog read; maintaining an in-memory counter.
- **Consequences:** No schema attribute is added. A constraint violation indicates an adapter defect, not a normal business conflict. Every partial loan write is rolled back.
- **Evidence:** Two simultaneous API requests for a book with one available copy produce exactly one 201 and one 409, and availability ends at zero.

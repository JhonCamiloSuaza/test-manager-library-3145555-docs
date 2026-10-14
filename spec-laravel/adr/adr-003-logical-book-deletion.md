# ADR-003 — Books use logical deletion

- **Status:** Accepted
- **Context:** Physical book deletion would break loan history and reporting.
- **Decision:** `book.deleted_at` records logical deletion. Catalog queries centralize the `deleted_at IS NULL` filter. A book with an open loan item returns 409 `BOOK_HAS_OPEN_LOANS`. ISBN uniqueness uses a partial index over active books.
- **Rejected alternatives:** Physical cascade deletion; adding an `active` attribute.
- **Consequences:** A logically deleted ISBN may be registered again. The restrictive `loan_item.book_id` FK prevents manual deletion from orphaning history. Cover binary data is the only physically deleted book data.
- **Evidence:** An inactive book returns 404, and direct physical deletion of a referenced book fails.

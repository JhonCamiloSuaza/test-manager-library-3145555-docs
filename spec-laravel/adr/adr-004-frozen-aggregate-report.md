# ADR-004 — Reports aggregate frozen values

- **Status:** Accepted
- **Context:** Reading current catalog values would change closed-period reports after a title or category edit.
- **Decision:** The report is not persisted. `ReportReader` executes one aggregate query grouped by frozen `book_id`, `book_title`, and `category_name`. `loan_item.category_name` deliberately has no FK.
- **Rejected alternatives:** Joining current catalog values; selecting the latest value in the period; materializing a report table.
- **Consequences:** A book moved between categories can produce two rows in one range. The report contains no member or librarian information.
- **Evidence:** After lending and renaming a book, the same report range retains the original title.

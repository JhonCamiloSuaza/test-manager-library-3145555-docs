# Product Specification — Manager Library

Manager Library enables a small library to manage its book catalog, members, loans, per-item returns, overdue items, and most-borrowed report.

This product is governed by [`constitution.md`](constitution.md). The database model lives only in [`data-model.md`](data-model.md), and the HTTP contract lives only in [`api-contract.md`](api-contract.md).

## 1. Actors

| Actor | Capabilities | Authentication |
|---|---|---|
| **Administrator** (`admin`) | Maintains books and members, creates librarians, and views every report | Username and password produce a token |
| **Librarian** (`librarian`) | Registers loans and returns and views books, members, and overdue items | Username and password produce a token |
| **Member** | Borrows books; it is business data, not a system user | None |

The deployment provisions the `admin` role from environment variables. Runtime API operations never grant that role.

## 2. Scope

**In scope:** optional book covers, five fixed categories, members, loans containing multiple items, per-item returns, overdue items, a most-borrowed report, and role-based login.

**Permanently out of scope:** fines, payments, reservations, renewals, email, SMS, bulk import, inter-branch loans, and book attributes beyond those in this specification. Publisher, publication year, and language must not be added.

## 3. User stories and acceptance criteria

### HU-01 — Log in

- **CA-01.1** Valid credentials return a token that expires after 8 hours.
- **CA-01.2** Invalid credentials return 401 with a message that does not reveal whether the username or password failed.
- **CA-01.3** Every endpoint except login and `/health` requires a token. Missing or invalid tokens return 401; insufficient roles return 403.

### HU-02 — Create librarians (admin)

- **CA-02.1** An administrator creates a user with username, password, and the `librarian` role.
- **CA-02.2** The username is trimmed, lowercased, and unique. A duplicate returns 409.
- **CA-02.3** Requesting the `admin` role through the API returns 422. Passwords never appear in responses or logs.

### HU-03 — Manage books (admin)

- **CA-03.1** A book is created with `title`, `author`, `isbn`, `category_id`, and `total_copies`. `available_copies` starts equal to `total_copies`.
- **CA-03.2** ISBN contains exactly 13 digits and is unique among active books. A duplicate returns 409.
- **CA-03.3** `total_copies` is at least 1. Replacing the total adjusts available copies by the same difference and never produces a negative value; an invalid reduction returns 422.
- **CA-03.4** A JPG or PNG cover of at most 2 MB stores an opaque key. A missing cover uses `null`, never an empty string.
- **CA-03.5** Deleting a book performs a logical deletion. It disappears from the catalog but remains in loan history. A book with open loan items cannot be deleted and returns 409.

### HU-04 — Search books

- **CA-04.1** Search supports partial title or author text, category filtering, active books only, title ordering, and `page`/`size` pagination with a maximum size of 50.
- **CA-04.2** The response includes the total result count.
- **CA-04.3** A missing or logically deleted book returns 404.

### HU-05 — Manage members

- **CA-05.1** Administrators and librarians create members with `full_name` and `document_number`. A duplicate document returns 409.
- **CA-05.2** Members can be listed and searched by name or document with pagination.
- **CA-05.3** Members cannot be deleted.

### HU-06 — Register a loan

- **CA-06.1** A loan belongs to one member, records the authenticated user, and contains 1 to 3 items.
- **CA-06.2** A book cannot appear twice in the same loan; duplicates return 422.
- **CA-06.3** Each item decrements one available copy in the same transaction. If any book is unavailable, the API returns 409 and persists nothing.
- **CA-06.4** A member cannot have more than three unreturned items in total. A violation returns 422 and includes the current count.
- **CA-06.5** A member with any overdue item cannot borrow another book; the API returns 409.
- **CA-06.6** `due_at` equals `loaned_at + 14 days`. Each item freezes the book title, ISBN, and category name at that instant.
- **CA-06.7** A registered loan cannot be edited or deleted.

### HU-07 — Register a return

- **CA-07.1** One loan item is returned at a time. The operation sets `returned_at` and increments available copies by one.
- **CA-07.2** Returning an already returned item produces 409 and never increments inventory twice.
- **CA-07.3** Available copies never exceed total copies.
- **CA-07.4** A loan is `closed` when every item has been returned.

### HU-08 — View and list loans

- **CA-08.1** Loan detail includes its items and calculated status: `open`, `overdue`, or `closed`.
- **CA-08.2** Loans can be filtered by date range, status, and member, ordered by loan date descending, and paginated.
- **CA-08.3** Status is calculated and never stored.

### HU-09 — View overdue items

- **CA-09.1** The result includes unreturned items whose `due_at` is earlier than the current instant, including book, member, and calculated late days.
- **CA-09.2** Items are ordered by greatest delay first and are accessible to `admin` and `librarian`.

### HU-10 — View the most-borrowed report

- **CA-10.1** For a date range, the report contains one row per frozen `book_id`, `book_title`, and `category_name`, with its borrowing count, ordered from highest to lowest.
- **CA-10.2** An end date before the start date returns 422. A range without loans returns an empty list.
- **CA-10.3** The report is not divided by librarian or member.
- **CA-10.4** A closed-period report does not change after a book or category is renamed.
- **CA-10.5** Only `admin` can access the report.

## 4. Business rules

| ID | Rule |
|---|---|
| RN-1 | Available copies remain between 0 and total copies. |
| RN-2 | The loan period is 14 days; a member may have at most 3 unreturned items. |
| RN-3 | A member with overdue items cannot borrow until those items are returned. |
| RN-4 | Frozen title, ISBN, and category values do not follow the current catalog. |
| RN-5 | Loans and returns are never deleted; books use logical deletion. |
| RN-6 | Every timestamp is a UTC instant. User-facing messages are Spanish; identifiers are English. |

## 5. Non-functional requirements

| ID | Requirement | Evidence |
|---|---|---|
| RNF-1 | Book search responds in less than 300 ms with 5,000 books | Load test using demonstration data |
| RNF-2 | When two librarians concurrently borrow the final copy, exactly one succeeds and one receives 409 | PostgreSQL concurrency test |
| RNF-3 | Plain-text passwords never travel beyond the login/create-user request, are never stored, and never appear in logs | Response and log review |
| RNF-4 | Every HTTP error uses the common error body | Contract tests |
| RNF-5 | One command starts the system, and the database engine initially contains no application tables | `infra` README and startup evidence |

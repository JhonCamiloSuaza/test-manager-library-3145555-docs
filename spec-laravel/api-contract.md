# HTTP API Contract — Manager Library

This contract is independent of implementation technology. Laravel must preserve the routes, fields, status codes, roles, and behavior defined here.

## 1. Conventions

| Topic | Contract |
|---|---|
| API base | `/api/v1` for every route below except the technical `/health` endpoint |
| Health | `GET /health` returns `{ "status": "ok" }` without authentication |
| Format | JSON encoded as UTF-8; field names use `snake_case` |
| Identifiers | UUID v4 strings |
| Instants | ISO 8601 UTC with `Z`, for example `2026-10-14T14:15:00Z` |
| Pagination | `page=1&size=20`; `size` has a maximum of 50 |
| Authentication | `Authorization: Bearer <token>` except login and health |
| Messages | User-facing messages are Spanish; fields and error codes are English |
| Money | The product has no monetary values |

Paginated responses use this exact shape:

```json
{
  "items": [],
  "page": 1,
  "size": 20,
  "total": 0
}
```

### Common error body

Every error, including authentication, authorization, validation, missing routes, unsupported methods, malformed JSON, and unexpected framework errors, uses this shape:

```json
{
  "code": "BOOK_NOT_AVAILABLE",
  "message": "No quedan ejemplares disponibles de «Cien años de soledad».",
  "details": [
    { "field": "items[0].book_id", "issue": "sin ejemplares" }
  ]
}
```

`details` may be empty. It never contains stack traces, SQL, server paths, passwords, password hashes, or tokens.

### Status codes

| Status | Meaning |
|---|---|
| 200 | Successful operation |
| 201 | Resource created |
| 204 | Successful operation without a response body |
| 401 | Missing, invalid, or expired token; or invalid credentials |
| 403 | Authenticated user has an insufficient role |
| 404 | Resource does not exist or is logically deleted |
| 409 | State conflict, duplicate, unavailable book, repeated return, overdue member, or open loans |
| 422 | Invalid field, input invariant, member limit, duplicate loan item, or inverted date range |

### Business error codes

| `code` | Status | Source |
|---|---:|---|
| `INVALID_CREDENTIALS` | 401 | AC-01.2 |
| `FORBIDDEN_ROLE` | 403 | AC-01.3 |
| `NOT_FOUND` | 404 | Missing resource |
| `USERNAME_TAKEN` | 409 | AC-02.2 |
| `ISBN_TAKEN` | 409 | AC-03.2 |
| `DOCUMENT_TAKEN` | 409 | AC-05.1 |
| `BOOK_HAS_OPEN_LOANS` | 409 | AC-03.5 |
| `BOOK_NOT_AVAILABLE` | 409 | AC-06.3 |
| `MEMBER_HAS_OVERDUE` | 409 | AC-06.5 |
| `ITEM_ALREADY_RETURNED` | 409 | AC-07.2 |
| `MEMBER_LIMIT_EXCEEDED` | 422 | AC-06.4 |
| `DUPLICATE_BOOK_IN_LOAN` | 422 | AC-06.2 |
| `VALIDATION_ERROR` | 422 | Invalid request field |
| `INVALID_DATE_RANGE` | 422 | AC-10.2 |

## 2. Authentication and health

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/health` | Public | Technical health check; route is not prefixed |
| POST | `/api/v1/auth/login` | Public | Issue an 8-hour token |

Login request:

```json
{ "username": "ana", "password": "********" }
```

Successful response:

```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 28800,
  "role": "librarian"
}
```

## 3. Users

| Method | Route | Role | Description |
|---|---|---|---|
| POST | `/api/v1/users` | `admin` | Create a librarian |

Request; `role` accepts only `librarian`:

```json
{ "username": "Ana ", "password": "********", "role": "librarian" }
```

Response — 201:

```json
{ "id": "…", "username": "ana", "role": "librarian" }
```

The response never contains `password` or `password_hash`.

## 4. Categories

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/api/v1/categories` | Authenticated | Return the five categories ordered by name |

Categories have no create, update, or delete endpoints.

## 5. Books

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/api/v1/books` | Authenticated | Search with `q`, `category_id`, `page`, and `size` |
| GET | `/api/v1/books/{id}` | Authenticated | Return active book detail |
| POST | `/api/v1/books` | `admin` | Create a book |
| PUT | `/api/v1/books/{id}` | `admin` | Replace title, author, category, and total copies |
| DELETE | `/api/v1/books/{id}` | `admin` | Logically delete a book |
| GET | `/api/v1/books/{id}/cover` | Authenticated | Return cover binary or 404 |
| PUT | `/api/v1/books/{id}/cover` | `admin` | Upload `multipart/form-data` field `file` |
| DELETE | `/api/v1/books/{id}/cover` | `admin` | Remove a cover |

Book response:

```json
{
  "id": "…",
  "title": "Cien años de soledad",
  "author": "Gabriel García Márquez",
  "isbn": "9780307474728",
  "category": { "id": "…", "name": "Novela" },
  "total_copies": 4,
  "available_copies": 3,
  "cover_key": null
}
```

Create request fields are exactly `title`, `author`, `isbn`, `category_id`, and `total_copies`. The server calculates `available_copies`.

The replace operation does not change `isbn`. The cover key is opaque and is not a filesystem path or public URL. Cover content must be validated as `image/jpeg` or `image/png`, independently of its extension, with a maximum size of 2 MB.

## 6. Members

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/api/v1/members` | `admin`, `librarian` | Search with `q`, `page`, and `size` |
| GET | `/api/v1/members/{id}` | `admin`, `librarian` | Return detail with `active_loans` count |
| POST | `/api/v1/members` | `admin`, `librarian` | Create from `full_name` and `document_number` |

`active_loans` is a calculated response field representing the current unreturned count; it is not a database column.

Members have no update or delete endpoint.

## 7. Loans and returns

| Method | Route | Role | Description |
|---|---|---|---|
| POST | `/api/v1/loans` | `admin`, `librarian` | Register a loan |
| GET | `/api/v1/loans` | `admin`, `librarian` | Filter by `from`, `to`, `status`, `member_id`, `page`, and `size` |
| GET | `/api/v1/loans/overdue` | `admin`, `librarian` | List overdue unreturned items |
| GET | `/api/v1/loans/{id}` | `admin`, `librarian` | Return loan detail and calculated status |
| POST | `/api/v1/loans/{loan_id}/items/{item_id}/return` | `admin`, `librarian` | Return one item |

Static route `/loans/overdue` must take precedence over `/loans/{id}`, or `{id}` must be constrained to UUID format.

Create-loan request:

```json
{
  "member_id": "…",
  "items": [
    { "book_id": "…" },
    { "book_id": "…" }
  ]
}
```

Create-loan response — 201:

```json
{
  "id": "…",
  "member_id": "…",
  "loaned_at": "2026-10-14T14:15:00Z",
  "status": "open",
  "items": [
    {
      "id": "…",
      "book_id": "…",
      "book_title": "Cien años de soledad",
      "isbn": "9780307474728",
      "category_name": "Novela",
      "due_at": "2026-10-28T14:15:00Z",
      "returned_at": null
    }
  ]
}
```

`status` is calculated:

- `open`: at least one item is unreturned and none is overdue.
- `overdue`: at least one unreturned item has `due_at` earlier than the current instant.
- `closed`: every item is returned.

Returning an item responds 200 with that item and a non-null `returned_at`. Repeating the return responds 409 and does not increment availability.

Overdue response items contain exactly `loan_id`, `item_id`, `book_title`, `member_name`, `due_at`, and `days_late`. Member name is permitted in this operational response but never in the aggregate report.

## 8. Most-borrowed report

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/api/v1/reports/most-borrowed` | `admin` | Require inclusive date parameters `from` and `to` |

Response:

```json
{
  "from": "2026-10-01",
  "to": "2026-10-31",
  "rows": [
    {
      "book_id": "…",
      "book_title": "Cien años de soledad",
      "category_name": "Novela",
      "times_borrowed": 12
    }
  ]
}
```

The report groups by frozen `book_id`, `book_title`, and `category_name`. It orders by `times_borrowed` descending and then title. It never contains member or librarian information. A book that changed category during the range may produce two rows.

## 9. Security

| Concern | Contract |
|---|---|
| Token | Signed JWT, 8-hour lifetime, containing `sub` and `role` only |
| Password | Store only a strong hash; never return or log the hash or clear text |
| Authorization | Evaluate role on every endpoint; 403 must not reveal resource existence |
| CORS | Allow only the frontend origin configured by environment |
| Cover | Validate binary content and the 2 MB maximum before storage |

# ADR-005 — Health and cover route clarifications

- **Status:** Accepted for the Laravel translation
- **Context:** The contract declares `/api/v1` as the base, while HU-01 and T-01 repeatedly identify `/health` as the public technical route. The book section requires `GET /books/{id}/cover` in prose but omits it from the endpoint table.
- **Decision:** `GET /health` is the unprefixed technical health endpoint. Product endpoints, including login, use `/api/v1`. `GET /api/v1/books/{id}/cover` is included because the existing contract explicitly defines it as the way to render an opaque cover key.
- **Rejected alternatives:** Duplicating health at two paths; exposing storage paths; omitting cover retrieval.
- **Consequences:** There is one health route and one authenticated cover-download route. No product field or persistence attribute changes.
- **Evidence:** Contract tests verify both canonical routes and confirm that cover download requires authentication.

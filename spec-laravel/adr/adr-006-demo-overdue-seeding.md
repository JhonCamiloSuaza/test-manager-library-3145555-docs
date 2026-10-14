# ADR-006 — Demonstration overdue seeding conflict

- **Status:** Pending product-owner clarification
- **Context:** T-21 requires an API-only tool to create three immediately overdue items. The published create-loan request contains only `member_id` and `items`; `loaned_at` is assigned by the server and `due_at` is fourteen days later.
- **Decision:** No implementation decision is authorized yet. The conflict is documented instead of silently extending the product contract.
- **Prohibited implicit workarounds:** Adding `loaned_at` to the public request, introducing an undocumented endpoint or header, writing directly to PostgreSQL, or storing a derived status.
- **Candidate owner decisions:** Run the API with a controlled demonstration clock during seeding; authorize a documented administrative test mechanism; or revise the requirement that the initial demonstration contains overdue data.
- **Consequences:** The tool may seed books, members, and current loans through the API, but cannot truthfully claim T-21 overdue evidence until this ADR is resolved.
- **Evidence required after resolution:** The selected mechanism is documented in the API contract and produces exactly three overdue response items without changing the database model.

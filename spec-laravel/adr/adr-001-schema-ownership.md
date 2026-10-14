# ADR-001 — Application migrations own the schema

- **Status:** Accepted
- **Context:** DDL in both infrastructure initialization and application migrations would create two sources of truth.
- **Decision:** Laravel migrations own the `library` schema, tables, indexes, constraints, and category seed. `infra` creates only the empty PostgreSQL engine, database user, network, and volume.
- **Rejected alternatives:** SQL initialization scripts in `infra`; automatic ORM schema creation at application startup.
- **Consequences:** PostgreSQL extensions and indexes are created in the migration that first needs them. Manual production schema changes are defects.
- **Evidence:** `infra` contains no `CREATE TABLE`, and the verification queries in `data-model.md` match the migrations.

# Schema Design and Migrations

---

## The expand/contract pattern

The only safe way to change a schema that is in use by running software. Every migration that changes the meaning of existing data should follow three phases:

**Phase 1 — Expand:** Add the new structure (column, table, index). Do not remove anything. Both old and new code can coexist against this schema — old code ignores the new column; new code can start writing to it.

**Phase 2 — Migrate:** Backfill existing data. Move reads to the new structure. Deploy the application code that reads from the new column.

**Phase 3 — Contract:** Remove the old structure. This happens only after no running application version reads or writes the old column.

**The rule:** never put a `DROP COLUMN` or `DROP TABLE` in the same migration as the business logic change that made it obsolete. The old column must be safe to remove — meaning every application instance that was using it is gone from production.

---

## Expensive schema mistakes

**NOT NULL column without a default on a large table.** Adding a NOT NULL column requires every existing row to have a value. Without a default, the migration fails unless you backfill first. With a default, many databases will rewrite the entire table (table lock for the duration). On large tables, use a nullable column first, backfill, then add the NOT NULL constraint.

**Natural keys that change.** Using email address, username, or phone number as a primary key creates a migration problem when that value changes (and it will — users change their email). Use a surrogate key (auto-increment, UUID) as the primary key. Put uniqueness constraints on natural keys separately.

**Boolean columns that are really enums.** Status fields with two states today will have three states next year. `is_active` becomes `status: active | inactive | suspended | deleted`. Booleans cannot represent this without a schema migration. Use a string or integer enum from the start when the domain has more than two meaningful states.

**Storing JSON to avoid schema work.** JSON columns defer the schema decision, not eliminate it. Querying, indexing, and migrating JSON blobs is harder than structured columns. Use JSON when the structure is genuinely variable. Use columns when the structure is known.

**Missing indexes for query patterns.** An index added after the table is large requires a full table scan to build. Identify the query patterns when designing the table and add indexes for them. Compound indexes must match query ordering — `(user_id, created_at)` helps queries filtering by user_id; it does not help queries filtering by created_at alone.

---

## Safe migration checklist

Before running a migration against a production database:

- [ ] Does this migration acquire a table lock? (Adding NOT NULL, changing column type, rewriting rows)
- [ ] Does this migration remove a column or table that any running application version still reads?
- [ ] Has the migration been tested against a production-sized dataset? (A migration that takes 10ms on 1k rows takes 10 minutes on 1M rows)
- [ ] Is there a rollback plan if the migration fails halfway?
- [ ] Is this migration idempotent — safe to run twice? (Useful if it is interrupted and retried)

---

## ID design

**UUID vs. auto-increment integers:** UUIDs are globally unique (safe for distributed systems, safe to generate client-side, do not expose record count). Auto-increment integers are sortable, smaller, and more cache-friendly. For most internal tables, auto-increment is fine. For IDs that appear in URLs or are shared externally, consider whether exposing sequential IDs leaks information (e.g., competitor scraping record counts).

**Never reuse IDs.** A deleted record's ID should never be assigned to a new record. Cached references, audit logs, and external integrations hold onto IDs. Reuse causes silent data corruption.

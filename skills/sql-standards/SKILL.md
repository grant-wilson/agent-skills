---
description: sql-standards conventions for this project. Applies when working with files matching *.sql.
paths: ["*.sql"]
---

- SQL that touches user input is always parameterized — never string-concatenated,
in application code or in dynamic SQL. This is a security gate, not a style
preference.
- No `SELECT *` outside ad-hoc exploration, and `INSERT` always names its columns.
Explicit lists keep queries stable when the schema evolves and make review diffs
meaningful.

```sql
INSERT INTO orders (id, customer_id, total_cents, placed_at)
VALUES (:id, :customer_id, :total_cents, :placed_at);
```
- Solve problems with set operations — joins, window functions, CTEs — not cursors
or row-by-row loops. Use CTEs to name each step of a complex query; if a CTE
chain stops fitting in one screen, extract a view.

```sql
WITH ranked AS (
  SELECT customer_id, total_cents,
         row_number() OVER (PARTITION BY customer_id ORDER BY placed_at DESC) AS rn
  FROM orders
)
SELECT customer_id, total_cents
FROM ranked
WHERE rn = 1;
```
- Every table has a primary key. Enforce integrity in the schema — `NOT NULL` by
default, foreign keys for every relationship, `UNIQUE` and `CHECK` constraints
for business invariants — rather than trusting application code to behave.
- `snake_case` throughout; tables plural (`orders`), columns singular, primary key
`id`, foreign keys `<table-singular>_id`, indexes `ix_<table>_<columns>`,
constraints named explicitly (never auto-generated names, which differ per
environment).
- Write predicates the optimizer can use an index for: no functions or casts on
the filtered column (`WHERE placed_at >= :start`, not
`WHERE date(placed_at) = :day`), no leading-wildcard `LIKE`. Add indexes to
serve known query patterns and justify each one — every index taxes writes.
- Schema changes ship as versioned, forward-only migration scripts that are
idempotent (guarded re-runs) and reviewed like code. Never edit a migration that
has run in any shared environment — write a new one. Destructive changes
(dropping columns/tables) deploy in a separate release after code stops
referencing them.
- Wrap multi-statement writes in an explicit transaction that spans the smallest
possible scope — never user think-time or external calls. Batch large backfills
into chunks to keep locks and the undo log small.

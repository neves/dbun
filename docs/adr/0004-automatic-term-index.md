# Automatic term index in one FTS5 table

Status: proposed

Every scalar field of every document is indexed automatically as a synthetic term (`status=paused`, `authorId=users_…`) in one contentless FTS5 table whose rowid is the document id. Plain words go into the same table for full-text search. No field is declared.

Storing raw JSON in FTS5 does not work, because FTS5 indexes words and cannot tell which field a word came from. Real SQLite expression indexes are faster and smaller, but each one needs DDL, and every index would have to exist before an older app version writes, or it would silently return wrong results. Synthetic terms keep the physical schema frozen and make "just insert and filter" fast from the first document.

In a benchmark of 200k documents with 8 fields, inserts took about 2.05 s against 0.50 s for a plain table, and the file was about 47 MB against about 36 MB. An equality lookup took 0.23 ms and an AND of two fields under 1 ms. References add one `_ref=<id>` term per id value, which made a reference-heavy run about 30% slower again.

**Consequences**

- Roughly 4x slower inserts and about 30% more disk than plain storage, accepted for zero declarations.
- Equality filters, reverse reference lookups and `db.referencing(id)` are all index lookups.
- FTS5 cannot do ranges, ordering or uniqueness. Ranges and sorts narrow with the terms, then compare and sort in memory, and a sort over a large collection with no equality filter is a full scan (about 115 ms per 200k documents). A declared ordered index is a later opt-in.
- Unique constraints use the `key` column instead of the terms.
- The terms are derived data, rebuilt from `documents`. How numbers, booleans, null and missing fields are encoded as terms is still open.

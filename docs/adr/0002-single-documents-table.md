# One documents table with a collection column

Status: proposed

All collections live in one `documents` table, with the collection name in its own `collection` column and a `(collection, key)` unique index, instead of a table per collection created on first insert.

Creating a table is cheap, so "the database never changes" is not the reason. The reasons are that `db.get(id)` and `db.referencing(id)` cross collections and want one table and one term index; every tenant file is identical, so none ever needs its structure checked after a lib upgrade; and collection names can be any string, with no SQL identifier rules.

In a benchmark of 200k documents (150k posts, 50k comments), an unindexed filter over the comments took 31 ms in the shared table (limited by id range), 27 ms in its own table and 85 ms scanning everything. A table per collection would make scans about 15% faster and dropping a large collection instant.

**Consequences**

- Dropping a large collection means deleting its rows, which is slow.
- The collection name is a column, not part of the stored id, so the id can be a compact integer.
- Collection scans rely on the `(collection, …)` indexes, not on a physical separation.
- Splitting collections into tables later would be a data migration across every tenant file.

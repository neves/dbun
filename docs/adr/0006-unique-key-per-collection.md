# Optional unique key per collection

Status: proposed (the key itself was requested; the details below are not yet answered)

Each document may carry a `key`: an optional string, unique within its collection, that the client assigns and can fetch by (`getByKey`, `upsert`). It is a nullable column with a unique index on `(collection, key)`.

The term index can find a document by an external value such as an email but cannot reject a duplicate, and a client-chosen natural key should not have to be the document id. A real unique constraint covers both, and `upsert` by key is the natural fit for syncing from an external system.

**Consequences**

- NULL keys never collide, so documents without a key are unaffected.
- References use the `id`, not the key. The key is a lookup handle only, so `referencing` stays precise when a key changes.
- The key is stored exactly as given. The caller normalizes case.
- Deleting a document frees its key, because the unique index lives on `documents` only.
- Join collections for many-to-many relations can use a composite key such as `<idA>:<idB>` to make duplicate pairs impossible.

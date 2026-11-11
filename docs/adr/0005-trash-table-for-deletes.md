# Deleted documents move to a trash table

Status: proposed

Deleting a document moves its row from `documents` to `trash`, which has the same columns plus `deleted_at`, in one transaction that also removes its index terms. Restoring moves it back, bumps `version` and re-indexes it from `data`.

A `deleted_at` or `is_deleted` column on live rows would make every query filter on it, and a boolean would lose when the deletion happened. A separate table means live queries never see deleted rows, and deleted documents are removed from reference lookups and search, which is the right behavior anyway.

**Consequences**

- Posts that reference a deleted author keep a dangling id until the author is restored. `get` returns `null` for it in the meantime.
- Deleting frees the document's `key`. Restoring fails with `ConflictError` if the key has since been reused.
- A concurrent editor's `{ version }` check fails against a deleted document, and restoring bumps `version`, so stale edits cannot overwrite it.
- Retention is a purge of `trash` rows older than a cutoff. Who schedules it is open.
- Full change history is a different feature, a `changes` log, and is out of the MVP.

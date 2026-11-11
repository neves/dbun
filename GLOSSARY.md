# dbun

An embedded database for Bun applications where the client stores and queries data without ever writing or running a migration.

## Language

**Document**:
A JSON object stored by the client, with no shape imposed by the database.
_Avoid_: Row, record, entity

**Collection**:
A named group of documents that comes into existence the first time a document is inserted into it.
_Avoid_: Table, model

**Reference**:
A field in one document holding the id of another document, traversable in both directions and never enforced by the database.
_Avoid_: Foreign key, relation, link, edge

**Tenant**:
An isolated owner of data, stored as its own SQLite file in the data directory. Every collection is reached through a tenant.
_Avoid_: Account, workspace, organization, database

**Id**:
The lib-generated identifier of a document. A time-sortable integer in storage, and always a string of the form `<collection>_<base36 integer>` at API level.
_Avoid_: Primary key, uuid

**Key**:
An optional string a client assigns to a document, unique within its collection, usable to fetch the document. A lookup handle only, never a reference.
_Avoid_: External id, natural key, slug

**Version**:
An integer starting at 1 that the lib bumps on every write, used to detect that two editors changed the same document.
_Avoid_: Revision, etag

**Trash**:
Where a deleted document lives, with the time it was deleted, until it is restored or purged.
_Avoid_: Soft delete, recycle bin, archive

**Term**:
A synthetic string such as `status=paused` that the lib derives from a document and writes to the index so every field can be filtered.
_Avoid_: Token, index entry

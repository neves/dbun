# One SQLite file per tenant

Status: proposed

dbun is multi-tenant by default, so every read and write belongs to a tenant. We draw the tenant boundary at the SQLite file: each tenant gets its own file in one data directory, every file has the same physical schema, and collections are reachable only through a tenant handle (`db.tenant("acme").posts`).

The alternative is one shared file with a tenant on every document. That would force a tenant into every term and id, and isolation would depend on the lib never forgetting a filter. With a file per tenant a query cannot reach another tenant's data, deleting or exporting a tenant is a file operation, and the term index, ids and references work as if there were one tenant.

**Consequences**

- The lib keeps a bounded cache of open file handles and reopens on demand. How many open files are practical has not been measured.
- `referencing` and every other query work within one file. Cross-tenant queries mean opening each file.
- Data shared by several tenants lives in one reserved shared file, and tenant documents hold plain ids that point into it.
- Moving to a shared file later would mean rewriting ids and terms, so this is hard to reverse.


## MongoDB Best Practices (NestJS-Oriented, Production-Ready)

You are an expert in **MongoDB, Mongoose, and scalable backend systems**.
You design **efficient, secure, and maintainable data models** optimized for **high-performance NestJS applications**.

---

## MongoDB Design Principles

* Design schemas based on **access patterns**, not theoretical normalization
* Prefer **explicit schema design** over flexible “schemaless” usage
* Optimize for **read performance first**, then writes
* Model data to avoid excessive joins (`$lookup`) in hot paths

---

## Schema Design (Mongoose)

* Every collection MUST have a defined schema
* Disable implicit behavior:

  * Disable versioning unless required
  * Disable auto timestamps unless needed
* Use:

  * Explicit field types
  * Required flags
  * Default values
* Avoid deeply nested documents (>3 levels)

---

## Data Modeling Rules

* **Embed** when:

  * Data is tightly coupled
  * Data is read together
  * Cardinality is bounded
* **Reference** when:

  * Data grows unbounded
  * Data is shared across domains
  * Independent lifecycle exists

---

## Indexing (CRITICAL)

* Create indexes for:

  * All frequently queried fields
  * Foreign reference fields
  * Sort + filter combinations
* Use compound indexes deliberately
* Never rely on collection scans in production
* Periodically audit unused indexes

**Rule:**

> If a query runs often, it must be indexed.

---

## ObjectId Usage

* Always use `ObjectId` for `_id`
* Never use strings where `ObjectId` is expected
* Validate ObjectId inputs at API boundaries
* Do NOT expose raw ObjectIds directly to clients if avoidable

---

## Query Best Practices

* Always use projections (`select`)
* Limit returned fields explicitly
* Avoid `find()` without filters
* Paginate all list queries
* Avoid `$where` and regex-heavy queries on large collections

---

## Pagination

* Prefer **cursor-based pagination** for large datasets
* Use `_id` or indexed fields for cursors
* Avoid skip-based pagination on large collections

---

## Transactions

* Use transactions **only when required**
* Keep transactions short-lived
* Never perform external calls inside transactions
* Prefer eventual consistency when possible

---

## Validation & Data Integrity

* Enforce validation at:

  * DTO level (NestJS)
  * Schema level (MongoDB)
* Never trust client-provided data
* Use enums and value constraints
* Normalize data (lowercase emails, trimmed strings)

---

## Repository Pattern (NestJS)

* Access MongoDB ONLY via repositories
* Repositories:

  * Contain database queries only
  * Do NOT contain business logic
* Services must not know Mongoose internals
* Never use models directly in controllers

---

## Performance & Optimization

* Use `.lean()` for read-heavy queries
* Avoid over-fetching
* Use bulk operations for batch updates
* Monitor query execution time
* Use explain plans for optimization

---

## Concurrency & Consistency

* Design for eventual consistency
* Avoid document-level contention
* Use atomic operators (`$set`, `$inc`, `$push`)
* Avoid read-modify-write race conditions

---

## Error Handling

* Handle:

  * Duplicate key errors
  * Validation errors
  * Cast errors
* Never leak raw MongoDB errors to clients
* Map DB errors to domain-level exceptions

---

## Security

* Never store secrets in plaintext
* Hash sensitive data (passwords, tokens)
* Apply field-level access control
* Avoid exposing internal IDs unnecessarily
* Use least-privilege database users

---

## Migrations & Schema Evolution

* MongoDB schemas evolve — plan for it
* Use migrations for:

  * Data backfills
  * Structural changes
* Ensure migrations are:

  * Idempotent
  * Versioned
* Never rely on manual DB edits in production

---

## Monitoring & Observability

* Monitor:

  * Slow queries
  * Index usage
  * Connection pool size
* Enable query logging in non-prod
* Track error rates and timeouts

---

## Testing

* Use real schemas in tests
* Use isolated databases for test runs
* Clean collections between tests
* Do NOT mock MongoDB queries unless necessary

---

## Anti-Patterns to Avoid ❌

* Unindexed production queries
* Over-embedded documents
* God collections
* Skip-based pagination on large datasets
* Direct model access in controllers
* Storing derived data without justification

---

## Golden Rule 🟡

> **If MongoDB feels slow, your schema or indexes are wrong — not MongoDB.**


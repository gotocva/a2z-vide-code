## NestJS Best Practices (Production-Ready)

You are an expert in **NestJS, TypeScript, and scalable backend architecture**.
You write **secure, testable, performant, and maintainable backend code** following **NestJS and modern TypeScript best practices**.

---

## TypeScript Best Practices (Backend)

* Enable **strict type checking**
* Prefer **type inference** when the type is obvious
* Avoid `any`; use `unknown` and narrow explicitly
* Use **readonly** for immutable data
* Prefer **enums or union types** over magic strings
* Never suppress TypeScript errors (`@ts-ignore`) without justification

---

## NestJS Architecture Principles

* Follow **domain-driven module design**
* Each module must represent a **single business domain**
* Keep **controllers thin**, **services focused**
* Avoid cross-module imports; use explicit exports
* Never place business logic in controllers

---

## Modules

* One module per domain
* Modules should:

  * Declare providers they own
  * Export only what is required
* Do NOT create “shared” modules with business logic
* Use **dynamic modules** only when configuration truly varies

---

## Controllers

* Controllers handle **HTTP concerns only**
* No business logic
* Validate inputs using **DTOs**
* Always:

  * Return explicit DTOs
  * Use proper HTTP status codes
* Do NOT access database or repositories directly

---

## Services

* One service = one responsibility
* Services orchestrate business logic
* Prefer **pure functions** for complex logic
* Use `@Injectable()` with explicit scope when needed
* Avoid God services (>300–400 LOC)

---

## DTOs & Validation

* All incoming requests MUST use DTOs
* Use `class-validator` + `class-transformer`
* DTOs must:

  * Be immutable
  * Represent API contracts only
* Do NOT reuse DTOs as database models

---

## Database & Repositories

* Access the database ONLY via repositories
* Repositories:

  * Contain persistence logic only
  * Do NOT contain business rules
* Keep schemas/entities isolated from services
* Never leak ORM entities to controllers

---

## Dependency Injection

* Prefer constructor injection
* Avoid optional dependencies
* Do NOT use `ModuleRef` unless absolutely required
* Avoid circular dependencies (`forwardRef` is a last resort)

---

## Configuration

* Use `@nestjs/config`
* Centralize configuration by domain:

  * app
  * database
  * auth
  * cache
* Configuration must be:

  * Typed
  * Validated
* Do NOT access `process.env` directly outside config files

---

## Security

* Enable **global validation pipe**
* Enable **whitelisting** and **forbid non-whitelisted properties**
* Use **guards** for authentication & authorization
* Use **interceptors** for response shaping
* Never trust client input
* Mask sensitive fields (passwords, tokens) everywhere

---

## Guards, Pipes, Interceptors

* Guards → authorization & access control
* Pipes → validation & transformation
* Interceptors → logging, metrics, response formatting
* Filters → centralized error handling
* Each must serve **one concern only**

---

## Error Handling

* Use custom exceptions extending `HttpException`
* Never throw raw errors
* Centralize error formatting using exception filters
* Avoid leaking internal error details

---

## Async & Background Jobs

* Offload heavy tasks to:

  * Queues
  * Workers
  * Cron jobs
* Never block HTTP request lifecycle
* Ensure jobs are idempotent

---

## Events & Messaging

* Use event-driven architecture where applicable
* Avoid synchronous cross-module calls
* Events must:

  * Be explicit
  * Be version-safe
  * Fail gracefully

---

## Logging & Observability

* Use structured logging (JSON-friendly)
* Add correlation IDs for tracing
* Log:

  * Requests
  * Errors
  * External service failures
* Do NOT log sensitive data

---

## Testing

* Write:

  * Unit tests for services
  * E2E tests for controllers
* Mock:

  * External services
  * Databases
* Do NOT mock internal business logic
* Ensure tests are deterministic

---

## Performance

* Use caching where applicable
* Avoid N+1 database queries
* Paginate all list endpoints
* Use streaming for large payloads
* Monitor memory usage

---

## Code Style & Maintainability

* Consistent naming:

  * `*.controller.ts`
  * `*.service.ts`
  * `*.module.ts`
* Avoid deep relative imports
* Use path aliases
* Prefer clarity over cleverness

---

## Anti-Patterns to Avoid ❌

* Business logic in controllers
* Shared “utils” dumping ground
* God services
* Circular dependencies
* Direct database access in controllers
* Returning ORM entities to clients

---

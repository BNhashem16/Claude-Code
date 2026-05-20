# API Contract & N+1 Ban — Mandatory Rule

> Every API endpoint in this repository MUST return data through a `JsonResource`, follow the project's response envelope, paginate list endpoints, and execute a bounded number of queries. N+1 query patterns are blocking issues.
>
> Applies to controllers, resources, and AI-assisted changes. No exceptions.

## Why

API consumers (mobile apps, frontends, partners) depend on stable response shapes and predictable latency. Inconsistent envelopes break clients silently; N+1 queries turn a 50ms response into a 5s one as data grows. Both bugs are invisible in development on small seed data — they only show up in production.

## Hard Requirements

### 1. JsonResource Everywhere

- Every API response payload goes through a `JsonResource` or `ResourceCollection`. Never hand-build `response()->json([...])` for resource shapes.
- One Resource class per model exposed via API. Reuse it across show / index / store / update.
- `toArray(Request $request): array` MUST use shaped arrays (`array{...}` PHPDoc) consistent with the PHPStan max rule.
- Conditional fields use `$this->when(...)`, `$this->whenLoaded(...)`, `$this->whenNotNull(...)` — never inline `if`/`?:` in array literals.

### 2. Consistent Response Envelope

- Follow the **existing convention in this repo** for success and error envelopes. Check sibling controllers and `App\Http\Resources\*` before inventing a new shape.
- Generic shape (use whatever this repo already uses):
  - Success list: `{ data: [...], meta: { current_page, per_page, total, last_page }, links: { ... } }`
  - Success single: `{ data: { ... } }`
  - Error: `{ message, errors?: { field: [...] } }` (Laravel default) or whatever envelope the existing handler emits.
- Never mix envelopes in the same endpoint version. If the repo is inconsistent, do not introduce a third style — match the most-used one and flag the inconsistency in the PR.

### 3. Pagination Required for Lists

- Any endpoint that returns a collection of model instances MUST paginate: `->paginate(...)` or `->cursorPaginate(...)`.
- Cursor pagination for large / append-only data (orders, audit logs, messages). Page pagination for catalogs.
- A default `per_page` MUST be set; the client may override within a capped range (e.g. 10–100). Reject `per_page > 100` (configurable) in the FormRequest.
- `$resource::collection($paginator)` — Laravel preserves pagination metadata automatically.

### 4. HTTP Status Codes

| Action | Code |
| --- | --- |
| GET resource found | 200 |
| GET resource not found | 404 (`abort(404)`, do not 200 with `null`) |
| POST create succeeded | 201, with `Location` header where applicable |
| PUT/PATCH succeeded | 200 (with the updated resource) |
| DELETE succeeded | 204 (no body) |
| Validation failure | 422 (Laravel default via FormRequest) |
| Unauthenticated | 401 |
| Unauthorized (authenticated but forbidden) | 403 |
| Rate limited | 429 |
| Server error | 500 (don't leak the message in production) |

Do not return 200 with `{ success: false, error: ... }`. Use the correct status code and let HTTP carry the meaning.

### 5. Versioning

- API routes live under `/api/v1`, `/api/v2`, etc. Never break a published version — add `v2` instead.
- `Route::prefix('v1')->name('api.v1.')->group(...)` keeps generation tidy.
- Document breaking changes in the PR description with a migration path for clients.

### 6. Eager Loading — N+1 Ban

- Any controller / resource that accesses a relation MUST load it via `->with(...)` (on the query) or `->load(...)` (on the model) BEFORE the relation is touched.
- The `whenLoaded(...)` pattern in resources is mandatory: it makes the relation contract explicit and prevents accidental N+1 from a resource that "just works" in tests with small data.
- **Banned**: iterating models with a `foreach` / `collect()->each` / `array_map` that accesses a relation without prior eager loading.
- Use `withCount(...)`, `withExists(...)`, `withAggregate(...)` instead of `$model->relation->count()` in a loop.
- Use `whereIn(...)` over N individual queries.

### 7. Per-Page Query Budget

- A normal API response SHOULD execute a small, bounded number of queries (target: ≤ 10 for typical pages; ≤ 5 for read-only `show` endpoints).
- Verify locally with Laravel Telescope, `DB::listen(...)`, or `clockwork` before pushing.
- Tests for list endpoints MAY assert query count: `$this->assertDatabaseQueryCount(N);` (Pest/PHPUnit helper if available) so regressions are caught in CI.

### 8. Idempotency for Writes

- `PUT` / `DELETE` are idempotent by HTTP semantics — write handlers MUST be safe to call twice.
- `POST` write endpoints that create resources from a client request (orders, payments) SHOULD accept an `Idempotency-Key` header and short-circuit on replay.
- Background jobs triggered by API calls implement `ShouldBeUnique` with `uniqueId()` to avoid double-execution.

### 9. Caching Headers (Where Applicable)

- Read endpoints that serve immutable / slowly-changing data set `Cache-Control: public, max-age=...` and an `ETag` / `Last-Modified` header.
- Honor `If-None-Match` / `If-Modified-Since` and return `304` when unchanged.
- Authenticated, user-specific responses use `Cache-Control: private` and never `public`.

### 10. Rate Limiting

- Every public mutation endpoint has a `throttle:` middleware (see [`security-baseline.md`](security-baseline.md)). Standardize 429 responses through the same error envelope as the rest of the API.

## Quick Self-Checklist

- [ ] Every API payload goes through a `JsonResource` / `ResourceCollection`.
- [ ] Response envelope matches existing convention in this repo; no new shape invented.
- [ ] All list endpoints paginate; `per_page` is capped.
- [ ] HTTP status codes used correctly (201, 204, 422, 401 vs 403, 429).
- [ ] Routes are versioned (`/api/v1`).
- [ ] Every relation accessed in a controller / resource is eager-loaded with `->with()` or `->load()`.
- [ ] `whenLoaded(...)` used in resources for relations.
- [ ] No relation access inside a model loop without prior eager-load.
- [ ] Query count for the page is bounded; verified locally.
- [ ] Idempotency considered for `POST` writes; jobs are `ShouldBeUnique` where needed.
- [ ] Cache headers set for cacheable reads; `private` for user-specific responses.

> Treat this file as binding alongside [`phpstan-max.md`](phpstan-max.md), [`laravel-idiomatic.md`](laravel-idiomatic.md), [`security-baseline.md`](security-baseline.md), and [`migrations-safety.md`](migrations-safety.md). Inconsistent contracts and N+1 queries are blocking.

# Security Baseline — Mandatory Rule

> Every change in this repository MUST satisfy the security checklist below before it is committed, pushed, or proposed in a PR. Security regressions are not "small bugs" — they are blocking issues.
>
> Applies to features, fixes, refactors, generated code, and AI-assisted changes. No exceptions.

## Why

The cheapest moment to prevent SQL injection, mass-assignment, broken access control, and credential leaks is when the line is written. Catching them in code review (or worse, in production) is 10–100× more expensive.

## Hard Requirements

### 1. Mass Assignment

- Every Eloquent model MUST declare `protected $fillable = [...]` listing the safe-to-fill columns.
- `protected $guarded = []` is **banned** — it disables mass-assignment protection entirely.
- `Model::unguard()`, `Model::unguarded(...)`, and `forceFill(...)` are banned in application code. Seeders are the only acceptable place for `forceFill`/`unguarded` blocks, and they MUST be re-guarded afterwards.
- Never pass `$request->all()` or `$request->input()` into `create()` / `update()` / `fill()`. Use `$request->validated()` from a `FormRequest`.

### 2. Authorization

- Every Eloquent model that has user ownership or per-user visibility MUST have a `Policy` registered in `AuthServiceProvider`.
- Controllers MUST call `$this->authorize('verb', $model)` (or `Gate::authorize(...)`) before returning or mutating a model. Route model binding alone is not authorization.
- `abort_if`, `abort_unless`, and explicit policy checks are required on any endpoint that returns data the requester might not own.
- Inline role checks (`if ($user->role === 'admin')`) are banned outside Policies/Gates.

### 3. Validation & Input Hygiene

- All HTTP writes (POST/PUT/PATCH/DELETE) go through a dedicated `FormRequest` with `rules()`, `authorize()`, and (where needed) `prepareForValidation()`.
- Read `$request->validated()` (or `validated('field')`) — never `$request->all()` or `$request->input()` for trusted use.
- Validate file uploads: explicit `mimes:`, `max:` size, and a strict allowlist. Never trust the client-provided filename — generate a new one (`Str::uuid()`) before storing.
- Stripping HTML from rich-text fields requires a vetted sanitizer (HTMLPurifier or equivalent), never a regex.

### 4. SQL & Query Safety

- No `DB::raw(...)`, `whereRaw(...)`, `havingRaw(...)`, `orderByRaw(...)`, `selectRaw(...)`, or `DB::statement(...)` with user-controlled values. Use bindings: `whereRaw('lower(name) = ?', [$name])`.
- Prefer Eloquent / Query Builder over raw SQL strings.
- No string concatenation into SQL — ever.

### 5. File System & Storage

- File reads/writes for application data go through `Storage::disk(...)`. Never accept user-controlled absolute paths.
- Validate uploaded MIME server-side (`$file->getMimeType()`), not the client-claimed extension.
- Store user uploads outside the web root unless the route is intentionally public; serve via signed URLs (`Storage::temporaryUrl(...)`) for private content.
- Path traversal protection: reject any path containing `..`, leading `/`, or null bytes.

### 6. Authentication & Sessions

- Passwords use `Hash::make(...)` with the configured driver (bcrypt/argon2id). Never store plaintext, MD5, SHA1, SHA256, or any custom hash.
- API tokens (Sanctum/Passport) are revoked on password change and on logout (`$user->tokens()->delete()` or `$user->currentAccessToken()->delete()`).
- Session invalidation on password change: `Auth::logoutOtherDevices(...)`.
- Multi-factor / 2FA tokens never appear in logs or URLs.

### 7. Rate Limiting & Abuse

- Every public POST/PUT/PATCH/DELETE endpoint has a `throttle:` middleware appropriate to its sensitivity.
- Auth endpoints (login, register, password reset, OTP) have stricter throttles (e.g. `throttle:5,1` per IP **and** per identifier).
- Long-running endpoints have request timeouts.

### 8. Secrets

- No API keys, tokens, passwords, JWT secrets, or private keys in source code — ever.
- `env(...)` is only called inside `config/*.php` files. Application code reads `config(...)`.
- `.env.example` MUST be updated in the same PR whenever a new env var is introduced.
- Logs, error pages, and API responses MUST NOT leak secrets, full stack traces (in production), or PII identifiers.

### 9. Output Safety

- Blade auto-escaping is mandatory. `{!! !!}` is only acceptable for content from a trusted source AND with a one-line `// why:` justification on the same block.
- JSON responses go through `JsonResource` — never `echo json_encode(...)`.
- Email / SMS templates: any user-supplied content rendered into HTML must go through a sanitizer or be plain-text-only.

### 10. Transport & Headers

- HTTPS-only in production: `URL::forceScheme('https')` in `AppServiceProvider` for non-local envs.
- Cookies: `secure` + `httponly` + `sameSite=lax` minimum; `sameSite=strict` for session cookies where possible.
- CORS: never `allowed_origins: ['*']` in production. Scope to known frontends.
- CSRF: web routes use `VerifyCsrfToken`; API routes use Sanctum / Passport with proper token validation.

### 11. Logging & Error Reporting

- Use structured logs: `Log::info('order.shipped', ['order_id' => $order->id])`. Never log full request payloads, tokens, or PII unredacted.
- Production exception responses do not echo the message verbatim — use a generic error envelope.
- Sentry (or equivalent) breadcrumbs MUST scrub PII / tokens before send.

## Quick Self-Checklist

- [ ] Every model has `$fillable`; no `$guarded = []`, no `unguard()`.
- [ ] Every model with ownership has a Policy registered and called.
- [ ] All writes go through a `FormRequest`; only `validated()` reaches the model.
- [ ] No `*Raw(...)` or `DB::statement(...)` with user input.
- [ ] All file uploads validated (`mimes`, `max`) and stored with generated names.
- [ ] Public mutation endpoints have `throttle:` middleware; auth endpoints have stricter throttles.
- [ ] No secrets in code; `env(...)` only in `config/`; `.env.example` updated.
- [ ] No `{!! !!}` without a `// why:` justification.
- [ ] CORS scoped; CSRF / Sanctum / Passport correctly applied per route group.
- [ ] Logs and error responses don't leak tokens or PII.

> Treat this file as binding alongside [`phpstan-max.md`](phpstan-max.md) and [`laravel-idiomatic.md`](laravel-idiomatic.md). Security regressions are blocking, regardless of whether the code "works at runtime".

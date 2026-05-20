# Laravel-Idiomatic Code (Framework First, Native PHP Last) — Mandatory Rule

> Every new line of PHP written in this repository MUST be expressed in **idiomatic Laravel** when an idiomatic Laravel form exists. Native PHP is only acceptable when the framework genuinely has no equivalent.
>
> This rule applies to features, bug fixes, refactors, generated code, and AI-assisted changes. There are no exceptions for "small" snippets.

## Why

This is a Laravel application. Laravel's helpers, facades, Eloquent, Collections, validation, and lifecycle primitives carry guarantees (immutability, macroability, container resolution, testability, request lifecycle awareness) that hand-rolled native PHP does not. Mixing the two in one codebase leaks abstractions, breaks testing, and makes refactors brittle.

## Hard Requirements

When writing or modifying PHP in this project, Claude MUST prefer the Laravel form on the left over the native form on the right:

### Strings

| Use | Not |
| --- | --- |
| `Str::of($value)->trim()->lower()->slug()` or `str($value)->...` | `strtolower(trim($value))` + manual slug |
| `Str::startsWith`, `Str::endsWith`, `Str::contains`, `Str::after`, `Str::before`, `Str::between`, `Str::replace`, `Str::limit`, `Str::squish`, `Str::ascii`, `Str::studly`, `Str::camel`, `Str::snake`, `Str::kebab`, `Str::headline`, `Str::mask`, `Str::uuid`, `Str::ulid`, `Str::random` | `strpos`, `str_starts_with`, `str_ends_with`, `substr`, `str_replace`, `substr` slicing, `uniqid`, `bin2hex(random_bytes(...))` |
| `__('messages.key')`, `trans_choice(...)` | hand-rolled translation arrays |

### Arrays & Collections

| Use | Not |
| --- | --- |
| `collect($items)->map()->filter()->values()` | chained `array_map` / `array_filter` / `array_values` |
| `Arr::get`, `Arr::set`, `Arr::pluck`, `Arr::only`, `Arr::except`, `Arr::has`, `Arr::wrap`, `Arr::flatten`, `Arr::dot`, `Arr::undot` | hand-rolled `foreach` to do the same |
| `data_get($payload, 'user.address.city')` | nested null-coalescing chains |
| `LazyCollection::make(...)` for streaming | manual generator + counters |

Return typed collections from services: `Collection<int, Order>`. Never return raw `array` when a `Collection` would be more expressive (consistent with the PHPStan max rule).

### Dates & Times

| Use | Not |
| --- | --- |
| `Carbon\Carbon` / `Carbon\CarbonImmutable` (`now()`, `today()`, `$model->created_at->addDays(3)`) | `new DateTime()`, `date()`, `strtotime()`, `mktime()` |
| `now()->toDateString()`, `->diffForHumans()`, `->isoFormat(...)` | `date('Y-m-d')`, manual diff math |

### Filesystem & I/O

| Use | Not |
| --- | --- |
| `Storage::disk('public')->put(...)`, `Storage::url(...)`, `Storage::exists(...)` | `file_put_contents`, `fopen`, `is_file` on absolute paths |
| `File::get`, `File::put`, `File::exists`, `File::isDirectory` (for local-only filesystem ops outside Storage) | `file_get_contents`, `mkdir`, `is_dir` |

### HTTP

| Use | Not |
| --- | --- |
| `Http::withToken(...)->retry(3, 100)->post(...)` + `Http::fake(...)` in tests | `curl_*`, `file_get_contents($url)`, Guzzle directly |
| `Route::get`, `Route::resource`, `Route::apiResource`, route model binding | hand-rolled dispatchers |
| `URL::to(...)`, `route('orders.show', $order)`, `URL::signedRoute(...)` | concatenating base URL strings |

### Requests & Validation

| Use | Not |
| --- | --- |
| Dedicated `FormRequest` classes (`StoreOrderRequest`, `UpdateOrderRequest`) with `rules()`, `authorize()`, `prepareForValidation()`, `messages()` | inline `$request->validate([...])` in controllers for non-trivial flows |
| `$request->validated()` returning typed values (or `validated('field')`) | `$request->input()`, `$request->all()` for untrusted writes |
| `Validator::make(...)` only for non-HTTP validation (jobs/commands) | manual `if`-chains |

### Authorization

| Use | Not |
| --- | --- |
| Policies (`OrderPolicy@view`), `$this->authorize('view', $order)`, `Gate::authorize(...)` | inline role checks like `if ($user->role === 'admin')` |
| `auth()->user()`, `Auth::user()`, `auth()->id()` | `$_SESSION`, manual token parsing |

### Database

| Use | Not |
| --- | --- |
| Eloquent models with typed casts, scopes, accessors, mutators | raw `DB::select` strings unless the query is impossible in Eloquent |
| `DB::transaction(function () { ... })` with closures | `DB::beginTransaction` / `DB::commit` / `DB::rollBack` manually (only if you need re-entrant control) |
| Eloquent relationships (`belongsTo`, `hasMany`, `hasManyThrough`, `morphMany`) with eager loading (`->with(...)`, `->load(...)`) | manual JOIN strings in service classes |
| `Model::query()->whereBelongsTo(...)`, `withCount`, `withExists`, `withAggregate` | hand-rolled subqueries when Eloquent helpers exist |
| Query scopes (`#[Scope]` or `scopeXxx` methods) | duplicated `where(...)` chains across call sites |
| Migrations via `Schema::table(...)` + `Blueprint`; column changes include all existing attributes | raw `DB::statement('ALTER TABLE ...')` |
| Factories + Seeders | manually inserted rows |

### Responses

| Use | Not |
| --- | --- |
| `JsonResource` / `ResourceCollection` for every API payload, with `toArray(Request $request): array` | hand-built `response()->json([...])` for resource shapes that exist or could exist |
| `response()->json(...)`, `response()->noContent()`, `abort(404)`, `abort_if`, `abort_unless` | manual `http_response_code()` + `echo json_encode(...)` |
| API versioning conventions already in this repo | inventing a new structure |

### Caching, Queues, Events, Notifications, Mail

| Use | Not |
| --- | --- |
| `Cache::remember`, `Cache::tags(...)->flush()`, `Cache::lock(...)->block(...)` | hand-rolled file/DB caches |
| Queued Jobs (`implements ShouldQueue`), with `dispatch(new Job(...))`, retries, backoff, `uniqueId()` for `ShouldBeUnique` | `exec()` background tasks |
| Events + Listeners with typed payloads | inline coupling in controllers |
| Notifications via `Notification::send(...)` over Mail, Database, Slack, FCM channels | direct mail/SMS calls |
| `Mail::to($user)->send(new OrderShipped($order))` mailables | building MIME by hand |

### Logging & Errors

| Use | Not |
| --- | --- |
| `Log::info`, `Log::error('...', ['order_id' => $order->id])` with structured context arrays | `error_log`, `print_r`, `dump` in production paths |
| Custom exceptions (`OrderCannotBeShippedException`) with `render()` methods | generic `Exception` strings parsed by callers |
| `report($e)`, `report_if`, `rescue(fn () => ...)` | bare `try { ... } catch (\Throwable $e) {}` swallowing errors |

### Configuration & Environment

| Use | Not |
| --- | --- |
| `config('services.stripe.key')`, with the value declared in `config/services.php` | `env('STRIPE_KEY')` outside config files (env reads are unreliable when config is cached) |
| `app()->environment('production')`, `app()->isLocal()` | `getenv('APP_ENV')` |

### Containers, Bindings, Helpers

| Use | Not |
| --- | --- |
| Constructor injection, `app(MyService::class)`, `resolve(...)` | manual `new MyService(new Dep(...))` for services that should be container-managed |
| Service providers for bindings, singletons, macros | ad-hoc factory functions |
| Macros on `Str`, `Collection`, `Request`, `Response` when extending behavior | global helper functions in `app/Support` for trivial cases |

### Blade & Views

| Use | Not |
| --- | --- |
| Blade components, slots, `@props`, `@include`, `@stack`, `@push`, `@can`, `@auth` | raw `<?php echo ?>` in `.blade.php` |
| `Vite::asset`, `@vite([...])` | hard-coded `<script src="/build/...">` |

### Testing (mirror these expectations in tests)

| Use | Not |
| --- | --- |
| Pest/PHPUnit feature tests with `RefreshDatabase`, `actingAs`, `assertDatabaseHas`, `assertJsonStructure` | manual HTTP calls or DB asserts |
| Factories + states | hard-coded fixture arrays |
| `Http::fake`, `Mail::fake`, `Notification::fake`, `Queue::fake`, `Event::fake`, `Storage::fake('public')` | real network/disk/queue calls |
| Time travel: `$this->travelTo(now()->addDay())`, `Carbon::setTestNow(...)` | sleeping or modifying system time |

## Anti-Patterns to Block

- `new \DateTime(...)`, `date(...)`, `strtotime(...)`, `time()` for business-logic timestamps. (Use Carbon / `now()`.)
- Raw `$_GET`, `$_POST`, `$_SERVER`, `$_SESSION`, `$_COOKIE`, `$_FILES` access. (Use `Request`, `session()`, `Cookie::get(...)`, etc.)
- `file_get_contents`, `file_put_contents`, `fopen` for app data. (Use `Storage` or `File`.)
- `mail()` (native). (Use `Mail::` or a Notification channel.)
- `header(...)`, `setcookie(...)`, `http_response_code(...)`, `echo json_encode(...)`. (Use `response()` and a `Response`/`JsonResponse` return type.)
- `curl_*`, `file_get_contents($url)`, raw Guzzle from inside services. (Use `Http`.)
- Calling `env(...)` outside `config/` files.
- Manual SQL string interpolation. (Use bindings via Eloquent or the Query Builder.)
- `dd`, `dump`, `var_dump`, `print_r`, `ray` left in committed code. (Use `Log::*` with context arrays.)

## Decision Procedure (apply on every new file or edit)

1. Identify the operation (string transform, array reshape, HTTP call, DB read, file write, time math, response build, etc.).
2. Ask: is there a Laravel helper, facade, Eloquent feature, Collection method, or Carbon method that does this?
3. If yes — use it. If no, document **why** in a one-line `// why:` comment and proceed with native PHP.
4. Re-check after the edit: would a Laravel reviewer point at this line and say "we have a helper for that"? If yes, change it before committing.

## Quick Self-Checklist

- [ ] All string ops go through `Str::` / `str()` where a helper exists.
- [ ] All array reshapes go through `Arr::` / `collect()` / `data_get()`.
- [ ] All dates use Carbon; no `new DateTime`, no `date()`, no `strtotime`.
- [ ] All HTTP traffic uses `Http::`; no `curl`, no raw Guzzle.
- [ ] Request input is read via a `FormRequest` and `validated()`.
- [ ] Authorization goes through Policies / Gates, not inline role checks.
- [ ] DB access is Eloquent (or Query Builder with bindings); no raw SQL strings.
- [ ] API responses use `JsonResource` / `ResourceCollection`.
- [ ] Side effects (mail, SMS, push, slow IO) are queued Jobs / Notifications / Events.
- [ ] `config(...)` everywhere; `env(...)` only inside `config/`.
- [ ] No `dd` / `dump` / `var_dump` / `print_r` / `ray` in committed code.
- [ ] Tests use `RefreshDatabase`, factories, and `*::fake()` for external surfaces.

> Treat this file as binding alongside [`phpstan-max.md`](phpstan-max.md). Code that violates it is incomplete, regardless of whether it "works at runtime".

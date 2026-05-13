# Laravel Dead Code Audit — Hardened Prompt (v2)

I want you to act as a senior Laravel codebase auditor and refactoring engineer.

Your task: deeply inspect the entire Laravel project and identify all dead code, unused files, and unnecessary code — safely, with multi-signal verification and convention-aware reasoning.

You MUST NOT modify anything. Analysis only.

============================================================
## SCOPE
============================================================

Scan recursively. Detect:

- Unused PHP files
- Unused classes / abstract classes / interfaces / enums
- Unused methods / functions / closures
- Unused services / repositories / actions / DTOs
- Unused traits
- Unused helper functions (`app/helpers.php`, autoloaded files)
- Unused jobs / mailables / notifications
- Unused events / listeners / subscribers
- Unused form requests / API resources / resource collections
- Unused controllers / invokable controllers
- Unused routes (api / web / console / channels)
- Unused middleware (route + global + group)
- Unused commands (artisan)
- Unused policies / gates / abilities
- Unused observers
- Unused custom casts
- Unused custom validation Rule classes
- Unused view composers / creators
- Unused Blade components / `@directives` / view files
- Unused migrations (orphaned, never-referenced models)
- Unused seeders / factories
- Unused config entries + entire config files
- Unused imports (`use` statements)
- Unused composer dependencies (require + require-dev)
- Unused npm dependencies (if relevant to backend assets)
- Unused environment variables (`.env` keys never read)
- Unused language / translation keys
- Unreachable code blocks (after return / throw / exit)
- Legacy commented-out code blocks
- Dead conditional branches (constant-false conditions)

### IGNORE

- `vendor/`
- `storage/`
- `node_modules/`
- `bootstrap/cache/`
- `public/build/`, `public/hot/`
- `_ide_helper.php`, `.phpstorm.meta.php`

============================================================
## HARD-KEEP LIST — NEVER MARK AS DEAD AUTOMATICALLY
============================================================

The following categories are ALWAYS auto-discovered or convention-resolved in Laravel. NEVER move them to "Safe to Delete" — they go to "Suspicious — Manual Review" section ONLY, regardless of static-scan results.

Categories (folder-based auto-discovery):

1. **`app/Policies/*Policy.php`**
   - Resolved by `Gate::policy()` or naming convention
   - Methods called by **ability string** (NOT method-name)
   - Triggers: `$user->can('x')`, `authorize('x')`, `@can('x')`, `Gate::allows('x')`, `authorizeResource()`, `can:x` middleware, `Policy::class` in `$policies` array

2. **`app/Observers/*Observer.php`**
   - Registered via `Model::observe()` or `#[ObservedBy]` attribute
   - Methods called by event name (`creating`, `saved`, ...)

3. **`app/Listeners/**/*.php`**
   - Registered in `EventServiceProvider $listen`
   - Or via `#[AsEventListener]` attribute
   - Or auto-discovered from method type-hint

4. **`app/Http/Middleware/*.php`**
   - Registered in Kernel by **alias string**
   - Called via `'auth'`, `'throttle:60,1'` etc.

5. **`app/Http/Requests/*Request.php`**
   - Resolved by controller method type-hint
   - `rules()` / `authorize()` called by framework

6. **`app/Notifications/*.php`**
   - Sent via `Notification::send($user, new X())`
   - `via()` returns channel names as strings

7. **`app/Mail/*.php`**
   - Sent via `Mail::to()->send(new X())`

8. **`app/Jobs/*.php`**
   - Dispatched via `X::dispatch()` or `dispatch(new X())`
   - Serialized in DB by FQCN

9. **`app/Console/Commands/*.php`**
   - Resolved by `$signature`, NOT class name
   - Auto-registered from `Commands/` folder

10. **`app/Rules/*.php`**
    - Used in validation array: `'field' => [new XRule]`
    - Or as string: `'field' => 'x_rule'`

11. **`app/Casts/*.php`**
    - Listed in Model `$casts` array, often as `::class`

12. **`app/Events/*.php`**
    - Dispatched via `event(new X())` or `X::dispatch()`

13. **`app/Providers/*ServiceProvider.php`**
    - Listed in `config/app.php` providers
    - Or auto-discovered via `composer.json extra.laravel`

14. **`app/View/Components/*.php`**
    - Auto-discovered, used in Blade as `<x-component>`

15. **`app/Broadcasting/*.php`**
    - Channel auth classes, routed by string name

16. **`database/factories/*Factory.php`**
    - Resolved by `Model::factory()` convention

17. **`database/seeders/*Seeder.php`**
    - Called from `DatabaseSeeder` or `artisan db:seed`

18. **`resources/views/**/*.blade.php`**
    - Referenced by string name `view('x.y.z')`

19. **`resources/lang/**` or `lang/**`**
    - Referenced by string key `__('messages.x')`

### RULE

For any file under these paths, even with ZERO static references found, output category is FORCED to:

> **"Suspicious — Manual Review Required"**

NEVER place these under "Safe to Delete", regardless of confidence score or detection signals.

### Required additional verification per category

Example (Policies):
- Grep ability strings used in `can()` / `authorize()` / `@can` / middleware `can:`
- Check `AuthServiceProvider $policies` array
- Check auto-discovery convention: `App\Models\X` → `App\Policies\XPolicy`
- Check all Form Requests for `authorize()` using same abilities
- Check policy methods are called by name matching ability string

============================================================
## CONVENTION-BASED RESOLUTION AWARENESS
============================================================

Many Laravel components are resolved by **folder + naming convention**, with ZERO explicit `use` statement anywhere in the codebase. Static reference scans WILL return zero hits, but the code is fully live.

Examples:

- `App\Models\Post` → `App\Policies\PostPolicy` (auto)
- `App\Models\Post` → `Database\Factories\PostFactory` (auto)
- `Route::resource('posts', PostController::class)` → auto-calls `PostPolicy::viewAny / view / create / update / delete`
- `authorizeResource(Post::class)` inside controller constructor → same as above
- `'can:update,post'` middleware → `PostPolicy::update($user, $post)`
- `#[ObservedBy(PostObserver::class)]` attribute on model
- Auto-registered listeners by event type-hint

If a file lives under a convention folder AND no static ref exists, the answer is ALMOST CERTAINLY auto-discovery — NOT dead code. Mark **"Suspicious"** not "Safe to Delete".

============================================================
## STRICT LARAVEL AWARENESS RULES
============================================================

Treat these as ALWAYS-DYNAMIC. Never mark dead from static scan alone:

- Service container: `app()`, `resolve()`, `App::make()`, `make()`
- Facades + facade aliases (`config/app.php` aliases)
- Reflection (`ReflectionClass`, `ReflectionMethod`, ...)
- Event system + wildcard listeners + `Event::listen` strings
- Observers (`Model::observe`, `$observables`, `ObserverServiceProvider`)
- Policies / Gates (`Gate::define`, ability strings, `authorize()`)
- Middleware (route + group + global, alias map in Kernel)
- Service providers (`config/app.php` + composer auto-discovery)
- Macros (Builder, Collection, Request, Response, Str, ...)
- Mixin (`mixin()`)
- Model relationships (`hasMany`, `morphTo`, `morphMap`)
- Accessors / mutators (Eloquent attributes, `getXAttribute`, casts)
- Local + global scopes (`scopeX`, `booted()`, `addGlobalScope`)
- Casts (`$casts`, custom `CastsAttributes`, enum casts)
- Artisan commands (signature parsing, schedule)
- Route model binding (implicit + explicit, `resolveRouteBinding`)
- Blade templates (view names, `@include`, `@component`, `x-slot`)
- Dynamic class instantiation in config arrays
- Auto-discovery (`composer.json` → `extra.laravel`)
- Form Requests (resolved by controller type-hint)
- Custom Rule classes (string in validation arrays)
- Notification channels / Broadcasting channels
- View composers / creators (`View::composer` string names)
- Model factories + seeders
- `$dispatchesEvents` map on models
- `Relation::morphMap` (DB stores morph type as string alias)
- `$observables` array
- Mail Markdown components (`resources/views/vendor/mail`)
- Auth guards / providers (`config/auth.php`)
- Filesystem disks, queue connections, DB connections (referenced by string name)
- DB tables `jobs` / `failed_jobs` / `notifications` store FQCN as string
- Tests (PHPUnit, Pest) as legit entry points
- Compiled caches (`bootstrap/cache/*.php`)

============================================================
## ZERO-UNUSED-CODE STRATEGY (STRICT MULTI-PASS MODE)
============================================================

Do NOT rely on simple static detection. Run ALL passes:

### PASS 1 — Static AST scan
- Symbol references, `use` statements, `::class` refs

### PASS 2 — Execution graph build

Entry points:
- Routes (api / web / console / channels)
- Service providers (boot / register)
- Artisan commands (handle method)
- Event dispatch system
- Queue workers (job handle method)
- Scheduled tasks (`Console/Kernel schedule`)
- Middleware pipeline
- Tests (test methods)

Traverse:
```
route → middleware → controller → service → model → external
event → listener → job → handler
command → services → dependencies
blade → view composer → controller → view data flow
schedule → command → services
queue:work → job → handler
```

### PASS 3 — Container binding scan
- Scan all ServiceProviders for `bind` / `singleton` / `instance` / `when`
- Scan tagged bindings
- Scan contextual bindings (`when()->needs()->give()`)

### PASS 4 — Config-driven scan

Scan EVERY array in `config/` for:
- FQCN string literals
- `::class` references
- Driver / provider keys mapped to classes

### PASS 5 — DB-stored class scan
- Inspect schema of `jobs`, `failed_jobs`, `notifications` tables
- Note: payloads contain serialized FQCNs
- Morph map values

### PASS 6 — String-reference grep (per candidate)

For each suspected-dead symbol, grep for:
- FQCN as string literal
- Class basename
- `::class` refs in arrays
- kebab-case / snake_case / camelCase variants
- Route name patterns
- View name patterns (`'users.show'`)
- Config key patterns (`'services.x'`)
- Translation key patterns (`'messages.x'`)
- **Gate / Policy ability strings** (mandatory if policy)
- Artisan command signatures (`signature` property, not class)

### PASS 7 — Cross-check files
- `.env`, `.env.example`
- `phpunit.xml` / `phpunit.xml.dist` / pest config
- `composer.json` (autoload, scripts, `extra.laravel`)
- `package.json` (if backend referenced)
- `bootstrap/cache/*.php` (compiled provider / package list)
- `artisan`, `public/index.php` (entry files)

### PASS 8 — Production signal (if available)
- Last N days of production logs → which classes / routes actually hit
- Treat any production hit in last 30 days as **KEEP automatically**

============================================================
## USAGE DECISION RULES
============================================================

### Code is USED if ANY of:
- Reachable from a verified execution entry point
- Resolved via Laravel container (bind / singleton / contextual)
- Triggered via events / observers / policies / gates
- Used via reflection / facades / macros / mixins
- Referenced in Blade templates
- Referenced in config arrays
- Referenced as string FQCN anywhere
- Hit in production logs in last 30 days
- Modified in git in last 30 days (uncertainty signal → KEEP)
- File path matches HARD-KEEP LIST → forced KEEP

### Code is UNUSED only if ALL of:
- Zero references in execution graph
- No dynamic Laravel mechanism can trigger it
- No container binding resolves it
- No config-driven instantiation exists
- No string-literal reference anywhere
- Not referenced from any test
- Not in production traffic (if logs available)
- File path does NOT match HARD-KEEP LIST
- Ability / event / view / route string scan performed and empty

============================================================
## SAFETY HARD RULES
============================================================

- "Safe to Delete" REQUIRES **≥4 independent detection methods** agreeing on dead status
- "Safe to Delete" REQUIRES confidence **≥99%**
- Any public method on an interface / abstract class → **KEEP** automatically
- Any class implementing a Laravel contract → **KEEP** unless contract usage proven dead end-to-end
- Any class with `@api` / `@public` docblock → **KEEP**
- Any code modified in git in last 30 days → **KEEP** (uncertain)
- No deletion suggestion may cross module boundary without explicit DI graph proof
- Any doubt → **KEEP** (uncertain usage)
- Prefer false negatives (keep too much) over false positives (delete live code)
- HARD-KEEP LIST overrides EVERYTHING

============================================================
## PER-FINDING METADATA (REQUIRED)
============================================================

Every finding MUST include:

| Field | Description |
|---|---|
| `path` | Project-relative path |
| `symbol` | Class / method / function (FQCN) |
| `confidence` | 0–100% (numeric) |
| `detection_method` | static \| container \| config \| db \| tests \| combined |
| `detection_signals` | List of which passes flagged it |
| `verification_command` | Copy-paste grep / rg command |
| `last_git_modified` | YYYY-MM-DD |
| `loc` | Line count |
| `dependencies` | What it depends on (for removal order) |
| `dependents` | What depends on it (must be empty for dead) |
| `removal_order` | Integer, leaves first (lowest = safest) |
| `reversibility` | `git revert <SHA>` plan |
| `hard_keep_match` | true / false (file path matches HARD-KEEP) |
| `ability_string_scan` | Performed? List strings scanned |
| `event_name_scan` | Performed? List strings scanned |
| `view_name_scan` | Performed? List strings scanned |
| `route_name_scan` | Performed? List strings scanned |
| `config_key_scan` | Performed? List keys scanned |

Anything with confidence < 99% → **"Suspicious — Manual Review"** section, never "Safe to Delete". Zero exceptions.

If any of the `*_scan` fields were not performed for a finding → that finding is **DISQUALIFIED** from "Safe to Delete" automatically.

============================================================
## OUTPUT FORMAT
============================================================

Generate a SINGLE Markdown file. Save at:

```
/docs/laravel-dead-code-audit.md
```

### Required sections (in order):

```
# Laravel Dead Code Audit Report

## 1. Execution Graph Summary
High-level architecture flow. Entry points, main pipelines, module map.

## 2. Detection Passes Run
Checklist of all 8 passes with status + notes per pass.

## 3. Hard-Keep Categories Found
Count per category. Confirm none entered "Safe to Delete".

## 4. Unused Files
path + symbol + confidence + signals + dependency proof.

## 5. Unused Classes
class FQCN + reasoning + full reference trace.

## 6. Unused Methods
method + class + call-chain analysis + grep results.

## 7. Unused Config / Env / Lang Keys
key + file + last-access trace.

## 8. Unused Composer Dependencies
package + last usage scan.

## 9. Suspicious — Manual Review Required
Anything <99% confidence. Items that may be dynamically used.
ALL hard-keep matches land here regardless of confidence.
Explain WHY uncertain per item.

## 10. Safe To Delete Immediately
ONLY items with ≥99% confidence AND ≥4 agreeing detection signals
AND zero dependents AND no hard-keep match AND all string-scans performed.
Sorted by removal_order (leaves first).

## 11. Removal Order DAG
Visual / textual DAG. Delete order matters — leaves first.

## 12. Risk Analysis
What could break if any suggestion is wrong.
Per-section risk score (low / med / high).

## 13. Recommendations
Best cleanup strategy. Suggested phasing
(quarantine → soft-delete → remove).

## 14. Verification Commands Appendix
All grep / rg / artisan / composer commands the human can run
to independently verify findings.
```

============================================================
## FINAL RULES (HARDENED)
============================================================

- Confidence threshold for "Safe to Delete" = **99%** (not 95%)
- Minimum **4 agreeing detection signals** required
- **HARD-KEEP categories CANNOT enter "Safe to Delete" ever**, regardless of confidence score
- If file path matches any HARD-KEEP pattern → automatic downgrade to "Suspicious", no override possible
- Mandatory: list ability strings, event names, view names, config keys, and route names scanned PER finding
- If any required string-scan was not performed for a finding → that finding is DISQUALIFIED from "Safe to Delete"
- Ask explicit human confirmation **per file**, not per batch
- If you are not 100% sure → mark as "Suspicious", never "Safe to Delete"
- Output per-finding confidence score always
- Do NOT output anything outside the markdown file
- The markdown file is the ONLY deliverable

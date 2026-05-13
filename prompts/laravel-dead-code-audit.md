I want you to act as a senior Laravel codebase auditor and refactoring engineer.

Your task: deeply inspect the entire Laravel project and identify all dead code, unused files, and unnecessary code — safely, with multi-signal verification.

You MUST NOT modify anything. Analysis only.

============================================================
SCOPE
============================================================
Scan recursively. Detect:

- Unused PHP files
- Unused classes / abstract classes / interfaces / enums
- Unused methods / functions / closures
- Unused services / repositories / actions / DTOs
- Unused traits
- Unused helper functions (app/helpers.php, autoloaded files)
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
- Unused Blade components / @directives / view files
- Unused migrations (orphaned, never referenced models)
- Unused seeders / factories
- Unused config entries + entire config files
- Unused imports (use statements)
- Unused composer dependencies (require + require-dev)
- Unused npm dependencies (if relevant to backend assets)
- Unused environment variables (.env keys never read)
- Unused language/translation keys
- Unreachable code blocks (after return/throw/exit)
- Legacy commented-out code blocks
- Dead conditional branches (constant-false conditions)

IGNORE:
- vendor/
- storage/
- node_modules/
- bootstrap/cache/
- public/build/, public/hot/

============================================================
STRICT LARAVEL AWARENESS RULES
============================================================
Treat these as ALWAYS-DYNAMIC. Never mark dead from static scan alone:

- Service container: app(), resolve(), App::make(), make()
- Facades + facade aliases (config/app.php aliases)
- Reflection (ReflectionClass, ReflectionMethod, ...)
- Event system + wildcard listeners + Event::listen strings
- Observers (Model::observe, $observables, ObserverServiceProvider)
- Policies / Gates (Gate::define, ability strings, authorize())
- Middleware (route + group + global, alias map in Kernel)
- Service providers (config/app.php + composer auto-discovery)
- Macros (Builder, Collection, Request, Response, Str, ...)
- Mixin (mixin())
- Model relationships (hasMany, morphTo, morphMap)
- Accessors / mutators (Eloquent attributes, getXAttribute, casts)
- Local + global scopes (scopeX, booted(), addGlobalScope)
- Casts ($casts, custom CastsAttributes, enum casts)
- Artisan commands (signature parsing, schedule)
- Route model binding (implicit + explicit, resolveRouteBinding)
- Blade templates (view names, @include, @component, x-slot)
- Dynamic class instantiation in config arrays
- Auto-discovery (composer.json → extra.laravel)
- Form Requests (resolved by controller type-hint)
- Custom Rule classes (string in validation arrays)
- Notification channels / Broadcasting channels
- View composers / creators (View::composer string names)
- Model factories + seeders (database/factories, DatabaseSeeder)
- $dispatchesEvents map on models
- Relation::morphMap (DB stores morph type as string alias)
- $observables array
- Mail Markdown components (resources/views/vendor/mail)
- Auth guards / providers (config/auth.php)
- Filesystem disks, queue connections, DB connections (referenced by string name)
- DB tables jobs / failed_jobs / notifications store FQCN as string
- Tests (PHPUnit, Pest) as legit entry points
- Compiled caches (bootstrap/cache/*.php)
- IDE helper files (_ide_helper.php, .phpstorm.meta.php) → ignore

============================================================
ZERO-UNUSED-CODE STRATEGY (STRICT MULTI-PASS MODE)
============================================================

Do NOT rely on simple static detection. Run ALL passes:

PASS 1 — Static AST scan
   - Symbol references, use statements, ::class refs

PASS 2 — Execution graph build
   Entry points:
   - Routes (api/web/console/channels)
   - Service providers (boot/register)
   - Artisan commands (handle method)
   - Event dispatch system
   - Queue workers (job handle method)
   - Scheduled tasks (Console/Kernel schedule)
   - Middleware pipeline
   - Tests (test methods)
   
   Traverse:
   route → middleware → controller → service → model → external
   event → listener → job → handler
   command → services → dependencies
   blade → view composer → controller → view data flow
   schedule → command → services
   queue:work → job → handler

PASS 3 — Container binding scan
   - Scan all ServiceProviders for bind / singleton / instance / when
   - Scan tagged bindings
   - Scan contextual bindings (when()->needs()->give())

PASS 4 — Config-driven scan
   Scan EVERY array in config/ for:
   - FQCN string literals
   - ::class references
   - Driver/provider keys mapped to classes

PASS 5 — DB-stored class scan
   - Inspect schema of jobs, failed_jobs, notifications tables
   - Note: payloads contain serialized FQCNs
   - Morph map values

PASS 6 — String-reference grep (per candidate)
   For each suspected-dead symbol, grep for:
   - FQCN as string literal
   - Class basename
   - ::class refs in arrays
   - kebab-case / snake_case / camelCase variants
   - Route name patterns
   - View name patterns ('users.show')
   - Config key patterns ('services.x')
   - Translation key patterns ('messages.x')
   - Gate / Policy ability strings
   - Artisan command signatures (signature property, not class)

PASS 7 — Cross-check files
   - .env, .env.example
   - phpunit.xml / phpunit.xml.dist / pest config
   - composer.json (autoload, scripts, extra.laravel)
   - package.json (if backend referenced)
   - bootstrap/cache/*.php (compiled provider/package list)
   - artisan, public/index.php (entry files)

PASS 8 — Production signal (if available)
   - Last N days of production logs → which classes/routes actually hit
   - Treat any production hit in last 30 days as KEEP automatically

============================================================
USAGE DECISION RULES
============================================================

Code is USED if ANY of:
- Reachable from a verified execution entry point
- Resolved via Laravel container (bind / singleton / contextual)
- Triggered via events / observers / policies / gates
- Used via reflection / facades / macros / mixins
- Referenced in Blade templates
- Referenced in config arrays
- Referenced as string FQCN anywhere
- Hit in production logs in last 30 days
- Modified in git in last 30 days (uncertainty signal → KEEP)

Code is UNUSED only if ALL of:
- Zero references in execution graph
- No dynamic Laravel mechanism can trigger it
- No container binding resolves it
- No config-driven instantiation exists
- No string-literal reference anywhere
- Not referenced from any test
- Not in production traffic (if logs available)

============================================================
SAFETY HARD RULES
============================================================

- "Safe to Delete" REQUIRES ≥3 independent detection methods agreeing on dead status
- Any public method on an interface / abstract class → KEEP automatically
- Any class implementing a Laravel contract → KEEP unless contract usage proven dead end-to-end
- Any class with @api / @public docblock → KEEP
- Any code modified in git in last 30 days → KEEP (uncertain)
- No deletion suggestion may cross module boundary without explicit DI graph proof
- Any doubt → KEEP (uncertain usage)
- Prefer false negatives (keep too much) over false positives (delete live code)

============================================================
PER-FINDING METADATA (REQUIRED)
============================================================

Every finding MUST include:

- path: absolute or project-relative path
- symbol: class / method / function name (FQCN)
- confidence: 0-100% (numeric)
- detection_method: static | container | config | db | tests | combined
- detection_signals: list of which passes flagged it
- verification_command: copy-paste grep / rg command to verify
- last_git_modified: YYYY-MM-DD
- loc: line count
- dependencies: what it depends on (so we know removal order)
- dependents: what depends on it (must be empty for dead)
- removal_order: integer (leaves first, lowest = safest to remove first)
- reversibility: "git revert <SHA>" plan

Anything with confidence < 95% goes to "Suspicious — Manual Review" section, never to "Safe to Delete". Zero exceptions.

============================================================
OUTPUT FORMAT
============================================================

Generate a SINGLE Markdown file. Save at:

   /docs/laravel-dead-code-audit.md

Required sections (in order):

# Laravel Dead Code Audit Report

## 1. Execution Graph Summary
High-level architecture flow discovered. Entry points, main pipelines, module map.

## 2. Detection Passes Run
Checklist of all 8 passes with status + notes per pass.

## 3. Unused Files
path + symbol + confidence + detection signals + dependency proof

## 4. Unused Classes
class FQCN + reasoning + full reference trace

## 5. Unused Methods
method + class + call-chain analysis + grep results

## 6. Unused Config / Env / Lang Keys
key + file + last access trace

## 7. Unused Composer Dependencies
package + last usage scan

## 8. Suspicious — Manual Review Required
Anything <95% confidence. Items that may be dynamically used. Explain WHY uncertain.

## 9. Safe To Delete Immediately
ONLY items with ≥95% confidence AND ≥3 agreeing detection signals AND zero dependents.
Sorted by removal_order (leaves first).

## 10. Removal Order DAG
Visual / textual DAG. Delete order matters — leaves first.

## 11. Risk Analysis
What could break if any suggestion is wrong. Per-section risk score (low/med/high).

## 12. Recommendations
Best cleanup strategy. Suggested phasing (e.g. quarantine → soft-delete → remove).

## 13. Verification Commands Appendix
All grep / rg / artisan / composer commands the human can run to independently verify findings.

============================================================
FINAL RULES
============================================================

- If you are not 100% sure → mark as "Suspicious", never "Safe to Delete"
- Output per-finding confidence score always
- Anything <95% confidence → Suspicious section, no exceptions
- Ask for human confirmation before any deletion suggestion is acted on
- Do NOT output anything outside the markdown file
- The markdown file is the ONLY deliverable

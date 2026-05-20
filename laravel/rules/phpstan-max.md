# PHPStan Max Level (Level 10) — Mandatory Rule

> Any new code, refactor, or modification in this repository MUST pass `vendor/bin/phpstan analyse` at **level 10 (max)** with **zero new errors** before it is committed, pushed, or proposed in a PR.
>
> This rule applies to every implementation — features, bug fixes, refactors, generated code, and AI-assisted changes. There are no exceptions for "small" changes.

## Why

Level 10 / `max` is PHPStan's strictest setting. It catches mixed types, missing generics, unreachable branches, and silent runtime bugs that pass every other check. Holding the line at this level is the cheapest way to keep a Laravel codebase safe to refactor by humans and by agents.

## Hard Requirements

When writing or modifying PHP in this project, Claude MUST:

1. **Type everything.**
   - Add native parameter, return, and property types on every function, method, and class property.
   - Type generics explicitly: `Collection<int, User>`, `array<string, mixed>`, `array<int, Product>`, `LengthAwarePaginator<int, Order>`. Never leave a bare `Collection`, `array`, or `iterable`.
   - Use `readonly` properties and `final` classes where the design allows.

2. **Never introduce `mixed` unless it is truly unknown.**
   - If a value comes from JSON, request input, or a third-party API, narrow it with `assert()`, `instanceof`, `is_string()`, `is_int()`, `is_array()`, or a typed DTO **before** using it.
   - Cast request input via Form Requests and `validated()` with typed accessors; do not use `$request->input()` untyped.

3. **No `@phpstan-ignore` / `@phpstan-ignore-next-line` / `// @phpstan-ignore-line` / `ignoreErrors` to silence problems.**
   - Fix the root cause. The only acceptable suppression is a documented third-party limitation, and it MUST include a one-line `// why:` justification on the same block and be approved in code review.
   - Do not lower the level in `phpstan.neon`. Do not add `treatPhpDocTypesAsCertain: false` to work around errors.

4. **Eloquent / Larastan specifics.**
   - Always run with `parseModelCastsMethod: true` (already on in this repo).
   - Type model relationships with generics: `BelongsTo<User, self>`, `HasMany<Order, self>`, `MorphTo<Model, self>`.
   - Use `@property` / `@property-read` PHPDoc on models so Larastan resolves attribute and cast types. Run `php artisan ide-helper:models -W` when adding/altering columns.
   - Prefer typed accessors/mutators (`Attribute::make(...)`) with explicit `get:` / `set:` closures returning concrete types.
   - For polymorphic relations and dynamic scopes, use Larastan's generic templates (`@template TModel of Model`).

5. **Arrays must be shaped.**
   - Replace `array` with `array<TKey, TValue>` or an `array{...}` shape.
   - For config arrays, validation rules, and API payloads, define a `@phpstan-type` alias on the class and reuse it.

6. **Closures and callables.**
   - Type closure signatures: `Closure(Request): JsonResponse`, `callable(Order): bool`.
   - No bare `callable` or `Closure` in public APIs.

7. **Strict comparisons & narrowing.**
   - Use `===` / `!==`. Use `match` over `switch` where it fits.
   - After a null-check or `instanceof`, do not re-check the type unnecessarily; PHPStan flags dead branches at level 10.

8. **No dynamic property access.**
   - Declare all properties. Do not assign to undeclared properties on stdClass-style objects — use a DTO/value object instead.

## Mandatory Workflow

Before declaring **any** implementation "done":

```bash
composer phpstan        # or: vendor/bin/phpstan analyse --memory-limit=2G
```

- The run MUST report **0 errors**. A "0 errors" baseline is the contract.
- If the change touches models, run `php artisan ide-helper:models -W -R` first so Larastan sees the latest columns/casts.
- Add new failing types to `phpstan.neon` ONLY by raising precision (e.g. `checkUninitializedProperties: true`), never by exclusion.
- CI MUST run PHPStan on every PR. If CI is missing, add it in the same PR that introduces this rule, gated to fail on any error.

## When PHPStan Is Not Yet Installed

If this repo does not have `larastan/larastan` in `composer.json` yet:

1. Still follow every principle in this file — type everything, no `mixed`, no `@phpstan-ignore`, shaped arrays, typed relationships.
2. Treat the absence of PHPStan as a bug to fix. Open a follow-up to install larastan and add `phpstan.neon` with `level: 10`, `paths: [app/, config/, database/, routes/]`, `parseModelCastsMethod: true`, mirroring the other backend repos.
3. Do not weaken this rule because the tool is missing locally.

## Quick Self-Checklist (run mentally before every edit)

- [ ] Every parameter, return, and property has a native PHP type.
- [ ] Every `Collection`, `array`, `iterable`, `LengthAwarePaginator` has its generic types filled in.
- [ ] No `mixed` outside of a narrow-then-use block.
- [ ] No new `@phpstan-ignore*` lines and no `ignoreErrors` entries.
- [ ] Eloquent relations carry generics; new columns have `@property` PHPDoc.
- [ ] `composer phpstan` (level 10) reports zero errors.
- [ ] CI runs the same command and blocks the PR on any error.

> Treat this file as binding. Code that violates it is incomplete, regardless of whether it "works at runtime".

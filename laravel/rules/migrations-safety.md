# Migrations Safety — Mandatory Rule

> Every database migration in this repository MUST be safe to run on production data without taking the app down and without losing rows. Destructive operations require explicit approval in the PR description.
>
> Applies to schema migrations, data migrations, and AI-assisted changes. No exceptions.

## Why

In a multi-collaborator codebase, the only thing standing between a typo in a migration and silent data loss in production is this rule. The cost of a bad column drop is hours of restore work — sometimes weeks if it's noticed late.

## Hard Requirements

### 1. Additive by Default

- New columns MUST be either `nullable()` OR have an explicit `default(...)` value. A `NOT NULL` column without a default will break inserts mid-deploy.
- New tables MUST include `created_at` / `updated_at` (`$table->timestamps()`) unless there is a documented reason not to.
- Foreign keys are mandatory for any column ending in `_id` that references another table. Use `$table->foreignId('user_id')->constrained()->cascadeOnDelete()` (or `nullOnDelete()` — choose deliberately).
- Indexes are mandatory on: foreign keys (auto with `constrained()`), columns used in `WHERE` filters, and any column the app sorts on.

### 2. Destructive Operations Require Approval

The following ops are **destructive** and MUST be called out in the PR description with an explicit "Destructive migration — approved by @reviewer" line:

- `dropColumn(...)`
- `drop(...)` / `dropIfExists(...)` on any non-empty table
- `change()` that narrows a type (e.g. `VARCHAR(255)` → `VARCHAR(50)`, `BIGINT` → `INT`, `TEXT` → `VARCHAR`)
- Changing a nullable column to `NOT NULL` on an existing table without a backfill step
- Renaming a column or table while the old code is still running
- `truncate()` or `DB::statement('DELETE FROM ...')` inside a migration

For each destructive op, the PR MUST describe:

1. **Why** the destruction is necessary.
2. **Backup plan** — how the data can be recovered if something goes wrong.
3. **Deploy strategy** — a two-phase deploy is required for renames and `NOT NULL` promotions:
   - Phase 1: Add the new column / make the change backwards-compatible. Backfill data in a separate job/seeder. Ship and verify.
   - Phase 2: Drop the old column / enforce the constraint in a second PR after Phase 1 is live and stable.

### 3. Column Changes Preserve All Attributes

When using `->change()` on an existing column, the migration MUST include **every existing attribute** on that column — type, nullability, default, length, comment, charset, etc. Doctrine and Laravel will silently drop attributes you omit.

Example (correct):

```php
$table->string('email', 191)->nullable()->unique()->change();
```

Example (wrong — drops `unique()` and `nullable()`):

```php
$table->string('email')->change();
```

### 4. No Data Migrations Inside Schema Migrations

- Do not run `DB::table(...)->update(...)` or `Model::query()->update(...)` inside a schema migration.
- Backfills go in a dedicated artisan command, seeder, or queued job that can be retried, paused, and logged.
- If a column needs a backfill before becoming `NOT NULL`, that backfill is a separate step in the deploy plan.

### 5. Reversibility

- Every new migration MUST implement `down()` so it can be rolled back in development.
- Production rollback strategy is documented in the PR (we rarely roll back schema in prod, but `down()` is required for local resets and CI).
- Irreversible ops (e.g. `dropColumn` where data cannot be recovered) MUST say so in the PR description.

### 6. One Concern Per Migration

- Each migration file does **one** logical change. Multiple unrelated changes belong in separate files. This makes review, rollback, and `git bisect` work.
- Migration filenames describe intent: `create_orders_table`, `add_status_to_orders_table`, `add_index_on_orders_user_id`, `remove_legacy_phone_column_from_users_table`. No `update_tables` or `fix_db`.

### 7. Foreign Keys & Cascades

- Use `constrained()` for the standard `*_id` → `id` pattern.
- Choose `cascadeOnDelete()` vs `nullOnDelete()` vs `restrictOnDelete()` **deliberately** — the default (RESTRICT) is the safest for production data.
- Document the choice in a one-line comment on the constraint line if it isn't obvious.

### 8. Never Edit a Committed Migration

- Once a migration has been merged to `main` (or worse, deployed), it is **frozen**. Any further change is a new migration on top.
- The only acceptable edits to a merged migration are: typo fixes that have no semantic effect AND were caught before any production deploy.

### 9. Indexes

- Add indexes in their own migration when the original `create_*_table` migration is already merged.
- Composite indexes mirror the most common query: `index(['user_id', 'created_at'])` for `WHERE user_id = ? ORDER BY created_at DESC`.
- Drop unused indexes — they slow writes.

### 10. Soft Deletes

- Use `$table->softDeletes()` for models that are deleted but may need recovery / audit.
- Soft-delete columns are indexed where the app filters on them.

## Quick Self-Checklist

- [ ] New columns are `nullable()` or have a `default(...)`.
- [ ] All `*_id` columns have foreign keys via `constrained()`; cascade behavior is chosen deliberately.
- [ ] No `dropColumn`, `dropIfExists`, narrowing `change()`, or `NOT NULL` promotion without a "Destructive migration — approved" line in the PR.
- [ ] `->change()` calls list **every** existing attribute on the column.
- [ ] No `DB::table(...)->update(...)` or `Model::update(...)` inside a schema migration.
- [ ] `down()` is implemented for new migrations.
- [ ] One concern per file; filename describes intent.
- [ ] Indexes added for new filter / sort columns.
- [ ] No edits to already-merged migrations.

> Treat this file as binding alongside [`phpstan-max.md`](phpstan-max.md), [`laravel-idiomatic.md`](laravel-idiomatic.md), and [`security-baseline.md`](security-baseline.md). Bad migrations are blocking.

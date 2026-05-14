# Laravel-First — Prefer Framework APIs Over Raw PHP

When building or refactoring **any feature** in this codebase, reach for the Laravel API first. Drop to plain PHP only when the framework genuinely has no equivalent. Reviewers will push back on raw-PHP-flavoured Laravel code even if it works.

This rule complements [style.md](style.md) (`Str`/`Arr`/`Number`/`Uri` helpers) and [collections.md](collections.md) (Eloquent collection patterns) with the **feature-code idiom layer**: how to write transformations, lookups, and data normalisation.

## Use Collection Pipelines, Not `array_*` Chains

Incorrect:
```php
$ids = array_values(array_unique(array_map(
    static fn (mixed $item): int => TypeCaster::int($item['product_id'] ?? 0),
    $value,
)));
```

Correct:
```php
$ids = collect($value)
    ->map(static fn (mixed $item): int => TypeCaster::int(data_get($item, 'product_id', 0)))
    ->unique()
    ->values();
```

Equivalent for the common operations:

| Raw PHP | Laravel Collection |
|---|---|
| `array_map($fn, $a)` | `collect($a)->map($fn)` |
| `array_filter($a)` | `collect($a)->filter()` |
| `array_values($a)` | `collect($a)->values()` |
| `array_unique($a)` | `collect($a)->unique()` |
| `array_keys($a)` | `collect($a)->keys()` |
| `array_combine($k, $v)` | `collect($v)->combine($k)` |
| `array_merge($a, $b)` | `collect($a)->merge($b)` |
| `array_diff($a, $b)` | `collect($a)->diff($b)` |
| `array_intersect($a, $b)` | `collect($a)->intersect($b)` |
| `array_sum($a)` | `collect($a)->sum()` |
| `array_reduce($a, $fn, $init)` | `collect($a)->reduce($fn, $init)` |
| `in_array($x, $a, true)` | `collect($a)->contains($x)` |
| `count($a) === 0` | `collect($a)->isEmpty()` |
| `array_slice($a, $o, $l)` | `collect($a)->slice($o, $l)` |
| `array_flip($a)` | `collect($a)->flip()` |

Chain them — multi-step transforms should read as one pipeline, not as nested function calls.

## Use `data_get` for Nested / Defensive Reads

Incorrect:
```php
$id = is_array($item) ? ($item['product_id'] ?? 0) : 0;
$name = isset($payload['user']['profile']['name']) ? $payload['user']['profile']['name'] : 'guest';
```

Correct:
```php
$id   = data_get($item, 'product_id', 0);
$name = data_get($payload, 'user.profile.name', 'guest');
```

`data_get` works on arrays, objects, and mixes of both. Use `Arr::get` when the input is guaranteed to be an array and dot-notation is desirable; use `data_get` when the input type is `mixed`.

## Use Eloquent / Query Builder, Not Hand-Rolled SQL

Incorrect:
```php
$users = DB::select('select * from users where active = ? and country_id = ?', [1, $countryId]);
$ids   = array_column($users, 'id');
```

Correct:
```php
$ids = User::query()
    ->where('active', true)
    ->where('country_id', $countryId)
    ->pluck('id');
```

`whereIn` accepts any `iterable` (including `Collection`), so don't `->all()` / `->toArray()` a collection just to pass it in.

## Use `Rule::*` Builders or Invokable Rules

Incorrect:
```php
'role' => 'required|string|in:admin,editor,viewer'
```

Correct:
```php
'role' => ['required', 'string', Rule::in(['admin', 'editor', 'viewer'])]
```

For anything non-trivial — bulk existence, cross-field comparisons, conditional checks — write an invokable `ValidationRule` class. Inline string rules become unreadable fast and don't play well with PHPStan.

## Use Laravel Cache / Container / Auth APIs

Incorrect:
```php
$user = $_SESSION['user'] ?? null;
file_put_contents(storage_path('cache/foo.txt'), serialize($payload));
```

Correct:
```php
$user = Auth::user();                   // or authUser() helper in this project
Cache::put('foo', $payload, $ttl);      // or Cache::remember(...)
```

For HTTP requests, use the `Http` facade, not raw `curl_*`. For mail, use `Mail::to(...)->send(new SomeMailable())`, not `mail()`.

## Look at Sibling Files Before Improvising

Before writing a transformation, **grep the codebase for the established pattern**. If `CheckoutCartRequest` exposes `loadedCarts(): Collection<int, Cart>` and the controller calls it, the new `GuestCheckoutPreviewRequest` should expose the same shape. Consistency wins over personal preference.

## When Raw PHP Is Acceptable

- Single-call leaf operations where no Laravel equivalent exists (`hash()`, `random_bytes()`, `json_encode()` with `JSON_THROW_ON_ERROR`).
- Tight inner loops where Collection allocation is measured to matter (rare — measure first).
- Bit-level operations, native math functions (`floor`, `ceil`, `round`) — these are the right tool.

Otherwise: **Laravel first.**

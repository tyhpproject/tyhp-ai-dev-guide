## 17. Compile-time helpers

| Expr | Result | → |
|------|--------|---|
| `nameof(x)` | short symbol name | string literal — `nameof(User)` → `'User'` |
| `default(T)` | default value of a type | `0`/`0.0`/`''`/`false`/`[]`/`null` |
| `typeof(T)` | runtime `\Tyhp\Type` | type expression only: `typeof(int)`, `typeof(int\|string)`, `typeof(Optional<int>)`; source alias → factory `UserId()` / `Optional(\Tyhp\Type::int())` (pkg `tyhp/core`). Value path is `\Tyhp\Type::of($value)`. |
| `variable_exists(x)` | compile-time in-scope check | `\array_key_exists('x', \get_defined_vars())` (name extracted as string literal; checker may fold to `true`/`false`) |

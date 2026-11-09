## 17. Compile-time helpers

| Expr | Result | → |
|------|--------|---|
| `nameof(x)` | short symbol name | string literal — `nameof(User)` → `'User'`, `nameof(User::$name)` → `'name'` typed `__PropertyName<User>`; a `use`-imported class works at file top level and in functions, and an import used only by `nameof` is not "unused". `nameof` is a constant expression, so an attribute parameter `__PropertyName<User>` accepts `nameof(User::$name)` and rejects a name that is not a property of `User`. A class reference is `User::class` (`__ClassName<User>`) |
| `default(T)` | default value of a type | `0`/`0.0`/`''`/`false`/`[]`/`null` |
| `typeof(T)` | runtime `\Tyhp\Type` | type expression only: `typeof(int)`, `typeof(int\|string)`, `typeof(Optional<int>)`; source alias → factory `UserId()` / `Optional(\Tyhp\Type::int())` (pkg `tyhp/core`). A declared class compiles to its fully-qualified `::class`. Value path is `\Tyhp\Type::of($value)`. |
| `variable_exists(x)` | compile-time in-scope check | `\array_key_exists('x', \get_defined_vars())` (name extracted as string literal; checker may fold to `true`/`false`) |

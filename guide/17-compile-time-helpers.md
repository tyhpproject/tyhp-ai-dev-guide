## 17. Compile-time helpers

| Expr | Result | → |
|------|--------|---|
| `nameof(x)` | short symbol name | string literal — `nameof(User)` → `'User'`, `nameof(User::$name)` → `'name'` typed `__PropertyName<User>`; a `use`-imported class works at file top level and in functions, and an import used only by `nameof` is not "unused". `nameof` is a constant expression, so an attribute parameter `__PropertyName<User>` accepts `nameof(User::$name)` and rejects a name that is not a property of `User`. A class reference is `User::class` (`__ClassName<User>`) |
| `default(T)` | default value of a type | `0`/`0.0`/`''`/`false`/`[]`/`null`, or a class/enum `operator default(): self` |
| `typeof(T)` | runtime `\Tyhp\Type` | type expression only: `typeof(int)`, `typeof(int\|string)`, `typeof(Optional<int>)`; source alias → factory `UserId()` / `Optional(\Tyhp\Type::int())` (pkg `tyhp/core`). A declared class compiles to its fully-qualified `::class`. Value path is `\Tyhp\Type::of($value)`. |
| `variable_exists(x)` | compile-time in-scope check | `\array_key_exists('x', \get_defined_vars())` (name extracted as string literal; checker may fold to `true`/`false`) |

A class or enum may declare `operator default(): self`. No parameters. Return type `self`. At most one. The body is static (`$this` is TYHP4097). A parameter is TYHP4421, a second declaration is TYHP4422, and a return type other than `self` is TYHP4420. An interface, abstract class, trait, extension, or abstract operator is TYHP4423.

With the operator, `default(ThatType)`, `default(self)`, and `default(static)` have that type and call `ThatType::__tyhpDefault()` (`public static`, return `static`). Each call runs the body. An author who wants one shared instance returns it from the body. The call is not a constant, so it cannot be a property initializer or a parameter default (TYHP4090).

Without the operator, `default(Class)`, `default(self)`, and `default(static)` are null, including in a constant context. The operator is not inherited: `default(Child)` is null unless `Child` declares it.

`default(T)` on a generic parameter calls `\Tyhp\Type::defaultValue()`. A binding whose class declares `__tyhpDefault` produces that instance; a binding that does not produces null. Hand-written PHP may declare `public static function __tyhpDefault(): static` on that class. The name is reserved in any letter case, declared operator or not (TYHP4426). The body must not evaluate `default` of the same type (TYHP4424), including by constructing that type while omitting a constructor parameter whose default is `default` of that type (TYHP4425).

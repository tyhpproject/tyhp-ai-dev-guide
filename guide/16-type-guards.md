## 16. Type guards, narrowing, `is`

**Guard functions** — the "return type" narrows an argument (body must return `bool` on all paths;
named vars in the subject must be parameters); → `: bool`:
```tyhp
function isString(mixed $value): $value is string { return \is_string($value); }
function isUser(mixed $v): $v instanceof User { return $v instanceof User; }
function hasStringAt(int $key, array $array): $array[$key] is string {
    return \array_key_exists($key, $array) && \is_string($array[$key]);
}
```
**`is`** — Tyhp token (`T_TYHP_IS`), same narrowing as `instanceof`.
Emit: `#[\Tyhp\NativeTypeTest]` type-guard for exact `T` → `\Fqn($x)` or `\Class::method($x)`
(e.g. `$x is string` → `\is_string($x)`);
unmarked class/interface/enum/trait → `$x instanceof Fqn`;
else `\Tyhp\Type::is`. `$x is null` is allowed (`\is_null` when marked).

**What narrows in an `if`:**

| Condition | true | false |
|-----------|------|-------|
| `$x instanceof T` | `$x: T` | `$x` minus `T` |
| `$x === null` / `$x !== null` | `null` / non-null | non-null / `null` |
| `\is_string/\is_int/\is_float/\is_bool/\is_array/\is_null/\is_object/\is_callable($x)` | that type | negative |
| your guard `isFoo($x)` | guarded type | negative |
| `$a && $b` | narrows both | — |
| `$a \|\| $b` | — | negative-narrows both |

Later operands of `&&` / `||` are checked under PHP short-circuit polarity (right of `&&` after a true left; right of `||` after a false left).

Not narrowed: loose `==`/`!=`, truthiness, after reassignment.

Generic arguments on `\Closure` / `\Fiber` are compile-time. Assert a specific shape with `as`
(`$fn as \Closure<callable(int): string>`). `instanceof` / `is` test the PHP class.

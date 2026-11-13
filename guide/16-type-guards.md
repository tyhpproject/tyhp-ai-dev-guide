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
| `$x == null` / `$x != null` (loose) | — / non-null | non-null / — |
| `\is_string/\is_int/\is_float/\is_bool/\is_array/\is_null/\is_object/\is_callable($x)` | that type | negative |
| your guard `isFoo($x)` | guarded type | negative |
| `$a && $b` | narrows both | — |
| `$a \|\| $b` | — | negative-narrows both |

Later operands of `&&` / `||` are checked under PHP short-circuit polarity (right of `&&` after a true left; right of `||` after a false left).

An `if` arm that returns, throws, breaks, or continues leaves the code after the `if` with the
other arm's narrowing (`if ($x === null) { return; }` or `if ($x !== null) { … } else { return; }`
→ `$x` non-null afterwards).

`\assert($cond);` as a statement narrows the code after it as if `$cond` held
(`\assert($x !== null); $x->name;`).

`switch (true)` / `match (true)`: an arm's body sees its own condition as true; its conditions and
body see every earlier case condition as false; `default` sees every case condition as false:
```tyhp
return match (true) { $u === null => '', $u->admin => 'admin', default => $u->name };
```

Not narrowed: truthiness, after reassignment, loose `==`/`!=` against anything but `null`.

Property reads narrow like variables, including nested paths and static properties of any class
(`$this->prop`, `$obj->prop`, `$obj->a->b`, `$this->a->b`, `self::$prop`, `Foo::$prop`):
`if ($order->customer !== null) { $order->customer->name; }`. Assigning a path
ends its narrowing and the narrowing of every path under it (`$a->b = …` also clears `$a->b->c`).
A method call result is not narrowed by checking it — assign it to a variable, then check the variable.

Generic arguments on `\Closure` / `\Fiber` are compile-time. Assert a specific shape with `as`
(`$fn as \Closure<callable(int): string>`). `instanceof` / `is` test the PHP class.

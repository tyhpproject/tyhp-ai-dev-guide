## 4. Type system

**Known types, now enforced:** `int float string bool array callable iterable object mixed void
never null true false self parent static resource`. Stricter: `static` is not allowed as a
parameter or property type (return types, locals, and generic args like
`ReflectionClass<static>` are fine);
`resource` can't appear in a type you write.

**New types:**

| Type | Meaning | → |
|------|---------|---|
| `decimal` | arbitrary-precision decimal | `\Tyhp\Decimal` (pkg `tyhp/decimal`) |
| `struct` | structural value type | `array` |
| literals (`42`, `'GET'`, `3.14`) | single-value type | the scalar |

```tyhp
type HttpMethod = 'GET'|'POST'|'PUT';   // union of literal types
```
Unions/intersections/`?T` as in PHP; a union containing `mixed` → `mixed`. `mixed` is like TS `unknown`: anything
goes **into** it, but it is assignable only to `mixed` until narrowed ([§5](05-type-inference.md)). The same holds for a
generic class named without type args: raw `Box` ≈ `Box<mixed>` (each param widened to its `extends` bound), so `Box` ↛
`Box<int>` while `Box<int>` → `Box` ([§7](07-generics.md)).

**`decimal`:** same type as `\Tyhp\Decimal` (no import needed). Build values with `\Tyhp\decimal('19.99')`,
`new \Tyhp\Decimal('19.99', 2)`, or `(decimal) $x` (`$x`: `string`/`int`/`float` or a union of those →
`\Tyhp\Decimal::__from($x)`; any other source, e.g. `bool`, `?string`, arrays, or a `DecimalConvertible`
object, is TYHP4008 — use `\Tyhp\decimal($obj)`; casting an existing `decimal` is warning TYHP4205).
Use the **operators**, not method calls: `$a + $b`, `$price * 3`, `$a > $b`, `$sum += $x`, `-$a`,
`$a <=> $b`, `$a ** 2` (`+ - * / % ** == != < <= > >= <=>`, compound forms, unary `-`/`+`). Operands
may be `decimal`, `string`, `int`, `float` (left or right; `**` needs a `decimal` base and `int`
exponent) or a value typed `\Tyhp\Contracts\DecimalConvertible`; results are `decimal`/`bool`/`int`
(`+$a` is `float`). Convert any other operand type with `\Tyhp\decimal($x)`. Use methods only where no
operator exists: `->divide($x, $scale)`, `->round($p, $mode)`, `->withScale($n)`, `->sqrt($scale)`,
`->format()`. Arithmetic **truncates** to the larger operand scale (`\Tyhp\decimal('10.00') / 3` → `3.33`;
`\Tyhp\decimal('10') / 3` → `3`); widen with `withScale()` and `round()` once at the end. Pass strings,
not floats (`$a + 0.01` adds `0.01000000000000000021`). A `string`/`int`/`float` converts to `decimal`
implicitly only as a call argument or `return` value; `decimal $x = '2.5';` is TYHP4008.

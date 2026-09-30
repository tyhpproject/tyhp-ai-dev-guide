## 1. Mental model

Tyhp is a **typed superset of the PHP language**. You write `.tyhp`; the compiler emits `<?php` +
`declare(strict_types=1);`. No special VM.

- **Types are erased** in hints. Source `type` aliases still emit a `\Tyhp\Type` factory function or
  static method; hint usages expand to the underlying type.
- Some constructs compile to **Tyhp-runtime PHP**: property-hook polyfill when `output.phpVersion`
  is below 8.4; `async`/`await` → `\Tyhp\Promise::_async` / `_await`; structs → arrays (PHP sees
  `array`; no runtime shape check); remaining `$x is T` → `\Tyhp\Type::is` (`NativeTypeTest` /
  `instanceof` when those apply). `decimal`, `typeof`, type-alias factories, `using`/`:=`, operator
  overloads, and expression trees also use `\Tyhp\` packages.
- Stricter than PHP: **non-nullable by default**; **conditions must be `bool`** ([§3](03-strict-rules.md)); **every
  parameter, property, and (by default) return type must have a known type** ([§5](05-type-inference.md)).
  Locals use `$x = …` (inferred) first; `int $x = …` when inference is not enough ([§8](08-typed-locals.md)).

```tyhp
<?tyhp
namespace App;
type Id = int;                               // alias (hint → int; factory Id())
type Point = struct { int $x = 0; int $y = 0; };     // value struct → array (no runtime shape)
class Box<T = string> {                       // T-typed property → tracked PHP
    public T $value;
    public function __construct(T $value): void { $this->value = $value; }
    operator ==(self $l, self $r): bool => $l->value === $r->value;
}
function isString(mixed $v): $v is string { return \is_string($v); }   // type guard
async function load(): int { $n = await fetchAsync(); return $n; }     // → Promise::_async
$count = 0;                               // inferred local
using ($f = open()) { /* auto-disposed */ }
```

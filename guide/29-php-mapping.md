## 29. PHP-mapping reference

| Tyhp | → PHP |
|------|-------|
| `int $x = 5;` | `$x = 5;` |
| `Box<int>` | `Box` |
| `array<T>` / `callable(...)` / `iterable<T>` | `array` / `callable` / `iterable` |
| `\Closure<…>` / `\Fiber<…>` / `\Generator<…>` | `\Closure` / `\Fiber` / `\Generator` |
| `#[\Tyhp\PhpType('mixed')]` | usages stripped; PHP type is the constructor string |
| generic `T` / `T extends Foo` | `mixed` / `Foo` |
| `struct S` decl | *(nothing — erased)* |
| `new Vendor\Box<int>(…)` (foreign stamp) | `\Tyhp\Generic::bind(\Vendor\Box::class, \Tyhp\Type::int())(…)` |
| source `type X = …` decl | factory `function X(): \Tyhp\Type` (hints still expand); tyhpdef aliases inline unless `#[\Tyhp\GenericRuntime(aliasFactory: …)]` |
| struct value / `new S() with […]` | `array` / `['k'=>v]` |
| `decimal` (hint) | `\Tyhp\Decimal` |
| symbol-name / template / string types | `string` |
| `nameof(x)` | `'x'` |
| `default(int)` | `0` (`''`/`false`/`[]`/`null` by type) |
| `typeof(T)` | `\Tyhp\Type::…()`; source alias → `Alias()` / `Optional(\Tyhp\Type::int())` |
| `fn(...): $v is T` | `: bool` |
| `async function f(): T` | `function f(): \Tyhp\Promise { return \Tyhp\Promise::_async(...); }` |
| `await $p` | `\Tyhp\Promise::_await($p)` |
| `$x := new R()` | `\Tyhp\DisposableScope::create()->using(new R())` |
| `using ($r = …) {}` | `try {} finally { $r->dispose(); }` |
| `$a + $b` (overloaded) | method call on the operand type |
| `$recv->extMethod(a)` (multi-statement member) | `\ExtClass::extMethod($recv, a)` |
| `$recv->extMethod(a)` (spliced `=>` / single-`return`) | parenthesized mapping expression |
| `<?tyhp` | `<?php` + `declare(strict_types=1);` |
| `declare(php=…)` / `#[\Tyhp\Php]` | *(stripped; inactive bodies emit nothing)* |
| `declare(ext=…)` / `fallback function` / `fallback const` | *(tyhpdef only; stripped. An unmatched `declare(ext)` binds nothing)* |
| `$x \|> $f` (target &lt; 8.5) | nested call / temp (`$f($x)`), L→R |
| `(void) $e` (target &lt; 8.5) | `$e;` as a discarded statement |
| `clone($o, […])` / `clone $o with […]` (target ≥ 8.5) | `clone($o, […])` |
| `__CallableReturnType<T>` / parameter bags / Slice | callable return / `array` (bags) / `mixed` (rest / Slice unpack) |
| `__SuperType` / `__CallableThis` / `__IndexValueType` | resolved object / union / field types |
| `__SuperTypeName` / `__ClassName` | `string` |

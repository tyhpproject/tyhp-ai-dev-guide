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
| PHP `Box::new(T::int)(42)` / `Box::type(T::int)` | generated `new()` / `type()`; `new()` visibility matches the constructor and is not `final` |
| non-erased generic call, not `#[\Tyhp\InferGeneric]` | `name(types)(args)`, `binder` is the declared name, `layouts: [2]` |
| `#[\Tyhp\InferGeneric]` bare call | plain value signature; explicit `<T>` → `name__tyhpGeneric(types)(args)`, `layouts: [1]` |
| inline `fn` to `Expression` / `PropertyPath` | `\Tyhp\Expr::param(...)->…->backedBy(fn)` or `\Tyhp\PropertyPath::on(...)->…->backedBy(fn)`. A parameter typed `Expression` or `PropertyPath` is `\Tyhp\Expression\|\Closure` or `\Tyhp\PropertyPath\|\Closure`. A `\Closure` argument calls `fromClosure`. A promoted, by-ref, variadic, property-hook, or inline-closure parameter, a union with a type other than `null`, and `#[\Tyhp\PhpType]`, keep the spelled type |
| erased generic call | plain method, `layouts: [1]` |
| `lazy new User($id)` (≥ 8.4, stored property) | `ReflectionClass::newLazyGhost`; arguments evaluated first |
| `lazy new Box<int>(1)` (same compilation, ≥ 8.4) | `new_<Mangled>__tyhpLazy` (binds type args, then `__construct`) |
| `lazy new Vendor\Box<int>(…)` (foreign stamp) | eager `\Tyhp\Generic::bind(...)(...)`; defer with `->ghost(...)` |
| `lazy new` below 8.4, or no stored property | ordinary `new` / eager generic factory (TYHP4403 when nothing is stored) |
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
| PHP `await(async(fn (): int => 42))` | `\Tyhp\await(\Tyhp\async(…))`, both `#[\Tyhp\InferGeneric]`; `$promise->wait()` is the same wait |
| `$x := new R()` | `\Tyhp\DisposableScope::create()->using(new R())` |
| `using ($r = …) {}` | `try {} finally { if ($r instanceof … && !uninitialized lazy) $r->dispose(); }` |
| `$a + $b` (overloaded) | method call on the operand type |
| `$recv->extMethod(a)` (multi-statement member) | `\ExtClass::extMethod($recv, a)` |
| `$recv->extMethod(a)` (spliced `=>` / single-`return`) | parenthesized mapping expression |
| `<?tyhp` | `<?php` + `declare(strict_types=1);` |
| `<?= expr ?>` in `.tyhp` | `echo expr;` (no escaping). `.php` `<?=` unchanged |
| lossy signature (generic, struct, `callable(int): string`, class/interface/enum name) | native hint plus a docblock tag (`emit.docblocks`, default `lossy-only`) |
| `declare(php=…)` / `#[\Tyhp\Php]` | *(stripped; inactive bodies emit nothing)* |
| `declare(ext=…)` / `fallback function` / `fallback const` | *(tyhpdef only; stripped. An unmatched `declare(ext)` binds nothing)* |
| `$x \|> $f` (target &lt; 8.5) | nested call / temp (`$f($x)`), L→R |
| `(void) $e` (target &lt; 8.5) | `$e;` as a discarded statement |
| `clone($o, […])` / `clone $o with […]` (target ≥ 8.5) | `clone($o, […])` |
| `__CallableReturnType<T>` / parameter bags / Slice | callable return / `array` (bags) / `mixed` (rest / Slice unpack) |
| `__SuperType` / `__CallableThis` / `__IndexValueType` | resolved object / union / field types |
| `__SuperTypeName` / `__ClassName` | `string` |

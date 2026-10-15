## 7. Generics (compile-time only, erased)

```tyhp
class Container<T> {}
class Pair<TKey extends int|string, TValue> {}   // constraint via `extends`
class Box<T = string> {}                          // default via `= Type`
function map<T, U>(callable(T): U $fn, array<T> $in): array<U> {}
type StringMap<V> = array<string, V>;
$b = new Box<int>(5);
class Repo<T> extends Base<T> implements Query<T> {}   // generic args on extends/implements
```
- Omitted trailing type args use declared defaults (`Box` with `T = string` → `Box<string>`;
  partial `MyMap<int>` with `TValue = mixed` → `MyMap<int, mixed>`). Resolution order at call
  sites remains explicit → inference → the expected type of the call → defaults.
  A literal argument infers the widened type (`'plain'` → `string`, `1` → `int`,
  `true` → `bool`). Elements of `array<Cell<T>>` or `Cell<T> ...` merge
  (`Cell<int>` and `Cell<string>` → `T = int|string`). A union parameter
  `null|T|Box<T>` rejects a union argument that has a member matching no arm
  (`TYHP4010`). No argument binding, no expected type, and no default is
  `TYHP4377` (`Cannot infer type argument `T` for `make`. Specify it explicitly`).
- Arg count must match; each arg is checked against its `extends` constraint. A partial list
  (`Pair<int>` for `Pair<T, U>`) is `TYHP4036`.
- **Raw generic** (class named with no type args, no defaults, e.g. `Box`): when it is the **source**
  of an assignment/argument/return to an instantiation of the same class it is `Box<mixed>`, with each
  parameter widened to its declared `extends` bound (`mixed` if none). So `Box` → `Box<int>` is an error
  (`TYHP4008`/`4009`/`4010`), `Box<int>` → `Box` and `Box` → `Box<mixed>` are fine. A member read
  through a raw receiver uses that same widened bound (`mixed`, or the `extends` bound). Bounds apply
  even if the class is declared later, in another file, or in a PHP stub/tyhpdef; `T extends Node<T>`
  is valid. A redundant `instanceof` keeps the precise instantiation (`Box<int>|null` + `instanceof Box`
  stays `Box<int>`); `instanceof Box` on a `mixed` value yields the raw `Box`. Bare `\Closure` stays
  assignable to any `\Closure<…>`.
- Erasure: `Box<int>` → `Box`; unconstrained `T` acts like `mixed`, `T extends Foo` like `Foo`;
  `array<…>`/`iterable<…>`/`callable(...)` → bare `array`/`iterable`/`callable`;
  `\Closure<…>`/`\Fiber<…>`/`\Generator<…>` → `\Closure`/`\Fiber`/`\Generator`.
  Compiled Tyhp libraries stamp `#[\Tyhp\GenericRuntime(erased:, layouts: [1])]` on every emitted
  generic class/method/function (`factory`/`binder` only when helpers exist). Same-compilation
  tracked sites keep those helpers; foreign tyhpdef stamps always emit `\Tyhp\Generic::bind(...)(...)`.
  A first-class callable of a generic function (`f(...)`, `f<string>(...)`) with a convertible parameter is
  emitted as `\Tyhp\Generic::wrapCallable(<native f(...) or its binder call>, <converting factory>)`;
  `\Tyhp\Generic::bind($wrapped, Type::int())` binds the target and keeps the conversions
  (`bind` on any other anonymous closure throws `\InvalidArgumentException`).
  Overlay tyhpdefs that omit the stamp keep erase / inline emit. `#[\Tyhp\EraseGeneric]` opts
  properties out of Mechanism C tracking.

**`array<T>` / `array<K,V>`:** `array<T>` = keys `int|string`, values `T`; `array<K,V>` = keys `K`
(`extends int|string`), values `V`. `iterable<…>` = same shape.

**`callable(…): R` — callable shapes:**

| Written | Signature |
|---------|-----------|
| `callable(string): int` | `(string): int` |
| `callable(int, int): bool` | `(int,int): bool` |
| `callable(): void` | `(): void` |
| `callable(...): bool` | any params, returns `bool` — **`extends` bound only**, not a value type |
| `callable(int ...): bool` | trailing PHP variadic `int ...$values`, returns `bool` |
| `callable` | untyped |

`\Closure` is a **class**, not a second callable spelling. Spell `\Closure<callable(int): string>`
(`TCallableShape`, then optional `TThis` / `TScope`). Bare `\Closure` is gradual. A `type` aliasing
a callable must carry the return constraint:
`type Handler<TReturn extends void|never|mixed> = callable(Request): TReturn;`

**Packs splice** inside a callable-shape parameter list: `__CallableParametersRest<T>` is a pack of T's parameters, not
one Rest-wrapper argument. `__Nullable<Rest<T>>` maps each member, then splices. Postfix `T...` is a
trailing PHP variadic (last parameter before return, non-pack only). Bare `callable(...): R` is the
any-arity wildcard (`extends` only). See [§19](19-utility-types.md) for Slice / Rest.

**Optional params → arity facets:** a function/closure/method with trailing defaults is typed as an
intersection of callable-shape facets (one facet per valid arity prefix). Facets are siblings —
`callable(A, B): R` does not subtype `callable(A): R` by itself.

**`\Fiber<TResume, TCallableShape>`:** `TResume` types `$fiber->resume($value)` only. `start` /
`throw` / `Fiber::suspend()` returns are `mixed|null`. `getCurrent(): ?\Fiber<mixed, callable>`.

**`\Generator`:** declared return is `\Generator` / `\Generator<K,V>` / four-arg form (what a call
returns). Body `return` is `TReturn` / `getReturn()`. Bare `\Generator` infers all four args from
`yield` / `send` / `return`.

**`ArrayAccess<K,V>`:** `$o[$k]` is `V`. Struct/array `K` is an error. Per-key struct maps:
`\Tyhp\Contracts\ArrayAccessShape<TStruct>`. User types implement `\Iterator` or
`\IteratorAggregate`, not `\Traversable` (`TYHP4326`).

**Relative types vs generics:** bare `self` / bare `static` inherit receiver / call-site type
arguments; parameterized `self<…>` / `parent<…>` are allowed; parameterized `static<…>` is
**forbidden** (TYHP4168). Factories that stamp a method generic onto the class use
`: self<T>` or the class name — not `: static<T>`. See `content/tyhp_0150_newTypes.md` in [tyhp-docs-src](https://github.com/tyhpproject/tyhp-docs-src).

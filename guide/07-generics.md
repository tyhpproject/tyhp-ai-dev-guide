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
  Compiled Tyhp libraries stamp `#[\Tyhp\GenericRuntime]` on every emitted generic.
  An erased callable, a class, and a type alias are `layouts: [1]`. A non-erased method or
  function that is not `#[\Tyhp\InferGeneric]` is one PHP method under its declared name,
  `layouts: [2]`, `binder` set to that name: `Json::decode($T)($json)`.
  `#[\Tyhp\InferGeneric]` keeps the value signature and a `name__tyhpGeneric` companion
  stamped `layouts: [1]`. A bare call uses the plain method. Explicit type arguments use
  the companion. `\Tyhp\Generic::bind` binds layout 1 and layout 2 and takes
  `\Tyhp\TypeLike|string`. When `ext-tyhp` is loaded at `0.1.0` or newer, `bind` calls `bindGet` on entry
  and `bindPut` only for a `\Closure` result. A class bind is not cached. The bind stays in PHP.
  `Json::decode` has no separate module entry. It checks the value with `Type::is` (`typeIs` when the
  module answers; `null` runs the PHP walk). `Type::parse` calls `typeParse` and builds `Type` objects
  from the graph; `null` uses the PHP parser. A missing or older module is ignored, with no warning.
  A Tyhp `new` of a foreign class emits
  `\Tyhp\Generic::bind(\Vendor\Box::class, \Tyhp\Type::int())(…)`. A foreign layout-2 call
  emits `name(types)(args)`. PHP calls `Box::new(T::int)(42)` and `Box::type(T::int)`.
  `new()` matches the constructor's visibility and is not `final` (a private constructor is
  `private static`). Abstract classes get `type()` only. Interfaces and traits get neither.
  A first-class callable of a generic function (`f(...)`, `f<string>(...)`) with a convertible parameter is
  emitted as `\Tyhp\Generic::wrapCallable(<native f(...) or its binder call>, <converting factory>)`;
  `\Tyhp\Generic::bind($wrapped, T::int)` binds the target and keeps the conversions
  (`bind` on any other anonymous closure throws `\InvalidArgumentException`).
  Overlay tyhpdefs that omit the stamp keep erase / inline emit. `#[\Tyhp\EraseGeneric]` opts
  properties out of Mechanism C tracking.
- Forbidden: type parameters on a magic method, including `__construct` (TYHP4404). A generic
  method that overrides or implements a non-generic method (TYHP4405). A method named `new`
  or `type` on a generic class or an ancestor (TYHP4406). `#[\Tyhp\InferGeneric]` on a type
  parameter no argument can supply (TYHP4407); a by-reference `T &$out` is not a source, so
  `Json::tryDecode` stays binder-first. The attribute on a non-generic declaration is warning
  TYHP4408. `ArrayAccessShape::offsetGet` and `offsetSet` are not generic.
- A property or promoted constructor parameter whose PHP hint loses information gets
  `#[\Tyhp\PropertyType('…')]` with the full type (`public array<Post> $posts` →
  `#[\Tyhp\PropertyType('array<App\\Post>')] public array $posts`; also `?Table<Blog>`, struct types,
  and source type aliases). Read it with `\Tyhp\Type::fromProperty(Blog::class, 'posts')`, then
  `->elementType()` / `->typeArguments()`. Types that mention a type parameter (`array<T>`) are not
  stamped; a written `#[\Tyhp\PropertyType]` is replaced by the compiler's.

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
`\Tyhp\Contracts\ArrayAccessShape<TStruct>`. `offsetGet` and `offsetSet` are not generic;
the key is `__IndexKeys<TStruct>` and the value is `__IndexValueTypes<TStruct>`. User types implement `\Iterator` or
`\IteratorAggregate`, not `\Traversable` (`TYHP4326`).

**Relative types vs generics:** bare `self` / bare `static` inherit receiver / call-site type
arguments; parameterized `self<…>` / `parent<…>` are allowed; parameterized `static<…>` is
**forbidden** (TYHP4168). Factories that stamp a method generic onto the class use
`: self<T>` or the class name — not `: static<T>`. See `content/tyhp_0150_newTypes.md` in [tyhp-docs-src](https://github.com/tyhpproject/tyhp-docs-src).

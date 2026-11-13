## 31. Lazy objects (`lazy new`, `\Tyhp\Lazy`)

Write **`lazy new`** or **`\Tyhp\Generic::bind(...)->ghost(...)`**. Do not call `ReflectionClass::newLazyGhost`. A closure you pass to `newLazyGhost` skips `__initGenerics__tyhpGeneric`, so the constructor sees unbound type arguments. `\Tyhp\Lazy::ghost` is the helper when the caller supplies the initializer and the class is not generic: `T` on that method is only the class, and it does not bind type arguments. A `new` inside that initializer constructs a different instance.

`lazy` is a keyword only when the next token is `new`. `->lazy(`, `function lazy(`, and any other `lazy` stay identifiers.

```tyhp
User $user = lazy new User($id);          // args run now; __construct runs on this instance later
Box<int> $box = lazy new Box<int>(1);     // constructor observes int
$box = \Tyhp\Generic::bind(Box::class, \Tyhp\Type::int())->ghost(1);   // class or type args chosen at runtime
```

### `lazy new`

`lazy new Name(args)` and `lazy new Name<T>(args)` only, including `()`. Arguments evaluate in the caller's scope, left to right, outside the ghost closure. Positional, named, and spread arguments reach `__construct` unchanged. An argument that throws throws at the expression.

The constructor runs on that same instance the first time state is observed or modified (property read/write, property iteration, `serialize`, `json_encode`, `clone`). A method that does not touch a property does not initialize. The instance keeps its identity (`$ghost === $ghost`). A throw from the initializer propagates and PHP leaves the object lazy.

`lazy new static()` is late-bound (`static::class`). A resolved class with no constructor gets an empty initializer, not a call to `__construct`. `lazy new $expr` and `lazy new class {}` do not parse.

**Floor.** One shape, from `output.phpVersion`. No `\PHP_VERSION_ID` in the expression.

| `output.phpVersion` | Emit |
|---------------------|------|
| ≥ 8.4, stored instance state | Ghost (below) |
| < 8.4, or no stored instance state | Ordinary `new`, or the existing eager generic factory |

A class with no stored instance property cannot stay lazy: PHP never calls the initializer. Tyhp emits the eager form, so the constructor **does** run, and warns **TYHP4403**. Stored state is an instance property declared on the class, inherited, or from a used trait. Static properties and virtual hook-only properties do not count. The injected generic bag of a tracked generic class does count, so `lazy new Box<int>(1)` is lazy even when `Box` declares no property, and the ghost initializer binds the type arguments before `__construct`.

**TYHP4402** (error): internal PHP class other than `\stdClass`, a class that extends an internal class other than `\stdClass`, or a non-class (struct, interface, enum, trait, type parameter). `\stdClass` is allowed and then warns TYHP4403 (no stored property). A user class that extends `\stdClass` and has a stored property is lazy. Constructor mismatches use the same code as `new`.

**Same compilation, ≥ 8.4, tracked generic:** `\Fqn::new_<Mangled>__tyhpLazy(...)`. The closure calls `__initGenerics__tyhpGeneric` then `__construct` on that instance.

**Same compilation, ≥ 8.4, other class with stored state:** `ReflectionClass::newLazyGhost`. With arguments, an immediately-invoked wrapper evaluates them in the caller, then the initializer is `$__tyhp_obj->__construct(...$__tyhpLazyArgs)`.

**Foreign compiled generic** (`#[\Tyhp\GenericRuntime]` on a tyhpdef): `lazy new` still emits `\Tyhp\Generic::bind(...)(...)`, which is eager `new`. This build does not add a lazy factory to that class. Defer it with `->ghost(...)`.

### `\Tyhp\Lazy` (`tyhp/core`)

`final`, private constructor. Which body you get is the **package build**, not a runtime branch. The 8.4 and 8.5 builds call reflection and contain no `\PHP_VERSION_ID`. The 8.2 and 8.3 builds are eager, contain no `\PHP_VERSION_ID`, and stay eager on a newer PHP.

```
static ghost<T extends object>(callable(T): void $initializer, int $options = 0): T
static proxy<T extends object>(callable(T): T $factory, int $options = 0): T
static isLazy(object $object): bool
static initialize<T>(T $object): T
static reset<T extends object>(T $object, callable(T): void $initializer, int $options = 0): void
static preset<T extends object>(T $object, __PropertyName<T> $property, mixed $value): void
```

- `ghost` — 8.4+: `newLazyGhost`. Identity stable. Initializer throw propagates, object stays lazy. 8.2/8.3: `newInstanceWithoutConstructor()`, then the initializer. Not lazy. Does **not** bind type arguments.
- `proxy` — 8.4+: `newLazyProxy`. `$proxy !==` the factory's instance. The real instance is whatever the factory returns; a `new` inside it is an ordinary `new`. 8.2/8.3: calls the factory immediately and returns that value. Not a proxy.
- `isLazy` — 8.4+: `ReflectionClass::isUninitializedLazyObject`. Not `ReflectionProperty::isLazy`. 8.2/8.3: `false`. Disposal does not call this (the 8.2 build would not see a ghost).
- `initialize` — 8.4+: `initializeLazyObject`, same instance. 8.2/8.3: returns `$object`.
- `reset` — 8.4+: `resetAsLazyGhost`. `$options` forwarded; pass `ReflectionClass::SKIP_DESTRUCTOR` yourself. 8.2/8.3: runs the initializer now. Does not become lazy.
- `preset` — `__PropertyName<T>`. 8.4+: `setRawValueWithoutLazyInitialization` (does not initialize). 8.2/8.3: `setValue`.

`$options` on `ghost`, `proxy`, and `reset` is forwarded. `ReflectionClass::SKIP_INITIALIZATION_ON_SERIALIZE` opts out of initialization during `serialize` only. `json_encode` still initializes.

The Reflection overlay types the callbacks: `newLazyGhost` / `resetAsLazyGhost` take `callable(T): void`; `newLazyProxy` / `resetAsLazyProxy` take `callable(T): T`. Prefer `\Tyhp\Lazy` over those methods. Full signatures → [handbook §7](../handbook/07-runtime-api.md).

### `\Tyhp\Generic::bind()->ghost()`

Class-string `bind` returns a `Generic` binder. `new()` and `__invoke()` stay eager. `ghost(mixed ...$constructorArguments): object` shares their one-shot guard (`\LogicException` on a second construction).

- **8.4/8.5 build:** `newLazyGhost`. Tracked class: the initializer, bound to the class, calls `__initGenerics__tyhpGeneric` with the padded type arguments, then `__construct`, on that same instance. Erased class, or no factory: the initializer only calls `__construct`. A class with no constructor and no arguments does not call `__construct`. Arguments on a class with no constructor fail at the `ghost()` call (`newInstanceArgs`), the same failure as `new`. If PHP returns a normal instance (no stored property; it never calls the initializer), `ghost()` runs the initializer before returning so the constructor still runs.
- **8.2/8.3 build:** `ghost()` is `new()`. Not lazy.
- **Callable `bind`** returns a `\Closure` and has no `ghost`.

### Disposal

An uninitialized lazy object is not disposed. An initialized object still is. `using` / `using await` / nested `:=` emit `ReflectionClass::isUninitializedLazyObject` (behind `\PHP_VERSION_ID >= 80400` when `output.phpVersion` is below 8.4, so an 8.2-floor build still skips a ghost on PHP 8.4+). The guard does not call `\Tyhp\Lazy`. `DisposableScope` and `DisposableHelper` use the same test through `call_user_func`, because every async build type-checks against the PHP 8.2 reflection signatures, which omit the method. → [§14](14-disposal.md).

Doctrine ORM: `\Doctrine\ORM\Configuration::enableNativeLazyObjects(true)`.

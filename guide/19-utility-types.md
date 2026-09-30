## 19. Utility types (global `__…`; checker expands, then erases)

**TS-style (on object/struct types):** `__AsReadOnly<T>` `__Partial<T>` `__Required<T>`
`__Pick<T,K>` `__Omit<T,K>` `__Record<K,V>` `__Exclude<T,U>` `__Extract<T,U>`
`__NonNullable<T>` `__Nullable<T>` `__Awaited<T>`.

**Callable-keyed** (TypeScript `ReturnType`/`Parameters` for a *callable type*, not a name string).
`TCallable` is inferred from the callback argument — there is no `typeof($cb)` in type position.

| Utility | Meaning | Erases to |
|---------|---------|-----------|
| `__CallableReturnType<TCallable>` | return type of callable `TCallable` | that return (or `mixed` while unbound) |
| `__CallableParametersStruct<TCallable>` | named-arg bag (string keys) | `array` |
| `__CallableParametersTuple<TCallable>` | positional bag (`0..n-1`, `$_1` aliases) | `array` |
| `__CallableParametersRest<TCallable>` | rest-unpack of the parameter list; **pack** (splices in a callable shape) | `mixed` (so `...$args` is not an array type) |
| `__CallableParametersSlice<TCallable, TStart, TMin>` | parameter `TStart`, or variadic tail from `TStart` (`N ≥ TMin`) | `mixed` when used as rest unpack |

Defaulted parameters are **optional keys** in the bags (omit them; required keys must be present).
Bare `callable` (no signature) stays gradual. Name-string helpers `__FunctionReturnType<'strlen'>`
/ `__MethodReturnType<T, M>` still exist for literal names.

`__Nullable<T>` / `__NonNullable<T>` map a **pack** memberwise (`?P0, ?P1, …`); non-pack
`__Nullable<int>` is `int|null`.

**Closure / ancestor / index utilities** (checker expands, then erases):

| Utility | Meaning | Erases to |
|---------|---------|-----------|
| `__SuperType<T>` | `T` and parent **object** types (`object`/`null` → `object`) | those object types |
| `__SuperTypeName<T>` | class-name strings of `T` and parents (inverse of `__CompatibleTypeName`) | `string` |
| `__CurrentScope` | lexical enclosing class/enum; class in instance **and** static methods; `null` at top-level | `object\|null` |
| `__CallableThis<TCallable>` | bound `$this` (`\Closure` → `TThis`; invokable → that class; bare `callable` → `object\|null`) | `object\|null` |
| `__CallableScope<TCallable>` | visibility scope (`\Closure` → `TScope`) | `object\|string\|null` |
| `__IndexKeys<T>` | literal union of struct `T` array keys (names + `as` aliases) | `string` / `int` / `string\|int` |
| `__IndexValueType<T, K>` | field type of key `K` (distributes over union `K`) | field type |
| `__IndexValueTypes<T>` | union of all field types of `T` | `mixed` |

Overlay aliases: `__ClosureThis` = `object\|null`; `__ClosureScope<TThis>` =
`__SuperType<TThis>\|__SuperTypeName<TThis>\|null`. `'static'` is a `bind`/`bindTo` argument only,
not a stored `TScope` (`TYHP4335`). Assert Closure/Fiber generic shapes with `as`.

`\call_user_func` / `\call_user_func_array` are typed with these (assoc → Struct overload; list →
Tuple). Emit is still the PHP builtins.

```tyhp
function apply<TCallable extends callable>(
    TCallable $cb,
    __CallableParametersStruct<TCallable> $args
): __CallableReturnType<TCallable> {
    return \call_user_func_array($cb, $args);
}
```

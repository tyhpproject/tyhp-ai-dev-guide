## 5. Type inference (what you must annotate)

The checker infers where it safely can, but **types are mandatory in most positions**:

| Position | Rule |
|----------|------|
| Parameter (function/method) | **Type required** — untyped → error. |
| Property, class const | **Type required.** |
| Return type | **Required.** |
| Local `$x = expr` (no type) | **Primary form.** Inferred from `expr`. If not inferable → error `TYHP4016`. |
| Local `T $x = …` | Explicit type wins — use when inference is not enough. |
| Local `$x;` (no type, no init) | Error `TYHP4016`. |
| Closure/`fn` params | Explicit type, **or** inferred from the expected `callable(...)`/`\Closure<callable(...)>` at the call site; no context → `TYHP4138`. |
| Generator `: \Generator` | Infer `\Generator<K,V,S,R>` from `yield` / `send` / `return` in the body. Two-arg form pins K/V. |

Inference specifics:
- Scalar literals infer as **literal types** (`$x = 5` ⇒ type `5`, assignable to `int`); `int $x = 5`
  keeps `int`.
- `new Foo()` ⇒ `Foo`. A call whose callee has **no declared return type** ⇒ `mixed`. `a ? b : c` ⇒
  union of branches; `??` ⇒ `left-without-null | right`.
- Array literals infer as typed arrays — `$xs = [1,2,3]` ⇒ `array<int>` (list shorthand);
  `['a' => 1]` ⇒ `array<string, int>`; empty `[]` ⇒ `array<never, never>` (bottom keys and
  values — assignable into any `array<…>` / `array<K,V>`, including `array<string, T>`).
  Indexing a typed `array<K,V>` yields `V`.
- No body-flow return inference — declare return types.
- `mixed` is one-way: everything is assignable **to** `mixed`, but `mixed` is **not** assignable to a
  specific type without narrowing ([§16](16-type-guards.md)).
- `mixed` is the **only** top type and is **always** strict — no config relaxes it. If you know
  TypeScript: `mixed` is its `unknown`, and Tyhp has no equivalent of `any`.
- Narrow a `mixed` value before: member access / method call, `$m()`, `$m[0]`, arithmetic, `< > <= >=`,
  concatenation / interpolation / `echo` / `print`, `$m::class`, spread (`[...$m]`, `f(...$m)`), and
  `$m->p ?? 1` / `$m->p ??= 1` (TYHP4160, TYHP4120, TYHP4096). Allowed on `mixed`: `===`, `isset`, `$m ?? 1`,
  `$m['k'] ?? 1`, explicit casts. A class `const mixed X` reads as `mixed`; `!== null` leaves it `mixed`.
- Generics: a `mixed` argument binds `T = mixed`, so `T extends int|string` fails (TYHP4035); a bare `T`
  binds a nullable argument as `T = ?X`. Spread arguments are checked element by element.
- Narrowing: `||` / `&&` combine guards on **variables** (not across property or index-key guards);
  `$m instanceof C` gives `C`; a redundant guard keeps the precise type; assignment narrowing checks the
  right-hand side first (`$m = $m + 1` on `mixed` is an error).
- TYHP4101 also reports members missing on classes, interfaces, and `object` (`property_exists` /
  `method_exists` guards count); bare `object` has no members.
- Null safety and required type annotations (params, returns, untyped locals without an inferable
  initializer) are likewise unconditional — there are no `checker.strictNullChecks` /
  `checker.noImplicitAny` toggles.
- `unresolved` is **not a language type** — you cannot write it. It is the checker's internal
  error-recovery marker (assignable to and from everything so one failure doesn't cascade). Seeing it
  in a diagnostic means the checker could not resolve something; treat it as a compiler-side gap, not
  as a type to satisfy.

## 28. Availability & gotchas

**Use freely:** typed locals; generics (pass all type args); top-level `type` aliases; structs;
operator overloads; extensions (`$x->m()` + `use extension`); **`tyhp/core` scalar catalog**
(`$s->length()`, `$arr->mapped(...)` — no per-file import); `async`/`await`; `using`/`:=`;
guards + narrowing; `nameof`/`default(T)`/`typeof`/`variable_exists`; `decimal` hints; symbol-name
and template string types; property hooks (polyfill on 8.2–8.3, native on 8.4+; `&get` needs 8.4); `with` on structs **and**
classes; `declare(php=…)` / `#[\Tyhp\Php]` / `#[\Tyhp\PhpType]`; `declare(ext="name")` /
`declare(ext="!name")`; `fallback function` / `fallback const`; `internal` (omitted from
`package.tyhpdef`); PHP 8.5 `|>`, `(void)`, `clone($o, […])` (lowered when
the target is older); object shapes (`type Name = object { … }`) and `__New<T>`.

**Annotate, don't assume:** every parameter/property and return type — locals infer from
initializers (including array literals).

**Use the PHP form instead:**

| Not | Use |
|-----|-----|
| `(decimal) $x` | `\Tyhp\decimal($x)` |
| in-place `$obj with […]` on `readonly` | `clone $obj with […]` or `new … with` |
| `&get` property hook when `output.phpVersion` &lt; `8.4` | by-value `get`, or raise the target to 8.4 |
| member/class-scoped or generic `type` aliases | a top-level non-generic `type` |
| `#[\Tyhp\Php]` on `struct` / `extension` | wrap in `declare(php=…) { }` |
| empty `catch (\T) {}` (`TYHP4121`) | `catch (\T $e) { (void)$e; }` (comment-only body still warns) |

**Not in the language yet:** null-conditional assignment (`$a?->b = …`).
⚠️ `tyhp watch` is not implemented. `tyhp lint --fix` is a stub.

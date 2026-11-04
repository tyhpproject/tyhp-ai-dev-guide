## 28. Availability & gotchas

**Use freely:** typed locals; generics (pass all type args); top-level `type` aliases; structs;
operator overloads; extensions (`$x->m()` + `use extension`); **`tyhp/core` scalar catalog**
(`$s->length()`, `$arr->mapped(...)` — no per-file import); `async`/`await`; `using`/`:=`;
guards + narrowing; `nameof`/`default(T)`/`typeof`/`variable_exists`; `decimal` (operators, `(decimal)` casts); symbol-name
and template string types; property hooks (polyfill on 8.2–8.3, native on 8.4+; `&get` needs 8.4); `with` on structs **and**
classes; `declare(php=…)` / `#[\Tyhp\Php]` / `#[\Tyhp\PhpType]`; `declare(ext="name")` /
`declare(ext="!name")`; `fallback function` / `fallback const`; `internal` (omitted from
`package.tyhpdef`); PHP 8.5 `|>`, `(void)`, `clone($o, […])` (lowered when
the target is older); object shapes (`type Name = object { … }`) and `__New<T>`; HTML templates
in a `.tyhp` file (`<?= expr ?>` is a checked `echo`, no escaping — [§2](02-files-and-tags.md)).

**Annotate, don't assume:** every parameter/property and return type — locals infer from
initializers (including array literals).

**Use the PHP form instead:**

| Not | Use |
|-----|-----|
| in-place `$obj with […]` on `readonly` | `clone $obj with […]` or `new … with` |
| readonly `new … with` or `clone … with` when `output.phpVersion` &lt; `8.3` (`TYHP4400`) | raise the target to 8.3+, or pass the values to the constructor |
| `clone … with` on a `readonly class` when `output.phpVersion` &lt; `8.5` (`TYHP4401`) | raise the target to 8.5, or use `new … with` from 8.3 |
| `&get` property hook when `output.phpVersion` &lt; `8.4` | by-value `get`, or raise the target to 8.4 |
| default on a `readonly` property (or any property of a `readonly class`) when `output.phpVersion` &lt; `8.6` (`TYHP4378`) | assign it in the constructor, or a promoted parameter default (`public readonly int $x = 1` in the constructor signature) |
| enum `__debugInfo()` when `output.phpVersion` &lt; `8.6` (`TYHP4379`); `__toString`, `__get`, `__clone`, `__serialize` and the other magic methods PHP rejects on enums (`TYHP4024`) | a regular method; enums may still declare `__call`, `__callStatic`, `__invoke` |
| member/class-scoped or generic `type` aliases | a top-level non-generic `type` |
| `#[\Tyhp\Php]` on `struct` / `extension` | wrap in `declare(php=…) { }` |
| empty `catch (\T) {}` (`TYHP4121`) | `catch (\T $e) { (void)$e; }` (comment-only body still warns) |
| bare `<?` short open tag | `<?tyhp`. `<?=` in `.tyhp` is the Tyhp echo; `<?=` in `.php` stays a PHP echo |
| `?>` in a tagless `.tyhp` file | a tagged file (first 6 characters `<?tyhp` in a tagless project) |

**Not in the language yet:** null-conditional assignment (`$a?->b = …`).
⚠️ `tyhp watch` is not implemented. `tyhp lint --fix` is a stub.

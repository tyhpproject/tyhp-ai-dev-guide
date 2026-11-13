## 27. Diagnostics you'll see

Format: `file(line,col): error TYHP####: message`. Checker codes `TYHP4000`–`TYHP4999`. **error**
fails the build; **warning** fails only with `--strict`; **info**. Hide specific warnings with
`suppressWarnings` in `tyhp.json` (or `--suppress-warnings`); errors cannot be hidden.
`tyhp explain TYHP4008` (or `--explain TYHP4008`) prints the long form. Text output is rustc-style
underlines; `--quiet` is one line. Unknown names may include a **did you mean**.

| Code | Meaning |
|------|---------|
| `TYHP4008` | cannot assign type X to type Y |
| `TYHP4009` | return type not compatible with declared return |
| `TYHP4010` | argument not assignable to parameter |
| `TYHP4015` | value possibly null where non-null required |
| `TYHP4198` | property, method call, or struct/object index on a possibly null receiver — narrow it or use `?->` |
| `TYHP4014` | variable possibly undefined — declared, not assigned on every path to the read |
| `TYHP4016` | variable needs a type annotation or inferable initializer |
| `TYHP4157` | read of a typed instance property that is not initialized |
| `TYHP4158` | `unset` of a local, a typed instance property, or a static property — assign `null` on a `?T` |
| `TYHP4418` | typed static property has no initializer (`?T` and `mixed` included) |
| `TYHP4419` | typed instance property is not initialized after construction |
| `TYHP4420`–`TYHP4426` | `operator default`: return not `self` / a parameter / a second declaration / disallowed target / `default` of the same type / constructor-parameter cycle / reserved `__tyhpDefault` |
| `TYHP4021` | assign to `readonly` property |
| `TYHP4025` | member visibility violation |
| `TYHP4028` | `await` / `using await` outside async code |
| `TYHP3031`–`TYHP3036` | overload signatures: no implementation / not adjacent / lone (`3033`, warning) / duplicate / modifier mismatch / constructor |
| `TYHP4030` | Resource of `:=`, `using` or `using await` requires a type implementing `IsDisposable` |
| `TYHP4035` | type argument does not satisfy constraint (a `mixed` argument fails any bound other than `mixed`) |
| `TYHP4118` | overload signature not compatible with the implementation |
| `TYHP4160` | `mixed` must be narrowed before this use |
| `TYHP4181` | erased (`=>`) extension member call cannot be spliced faithfully |
| `TYHP4373` | `Promise<T>` / `extends __Awaited<…>` type argument is not settled (promise, promise-bearing union, or unconstrained `T`) |
| `TYHP4374` | no overload accepts the call, even with implicit `operator convert` conversions (first declared signature accepting it as written wins) |
| `TYHP4375` | ambiguous overload: several signatures accept the call only through implicit conversions to different types — convert an argument explicitly |
| `TYHP4376` | a first-class callable of an overload set cannot convert an argument at one position: the overloads convert that type to different targets — call the overload directly |
| `TYHP4031` | property doesn't exist on type (e.g. in `with`) |
| `TYHP4032` | type-guard function must return `bool` |
| `TYHP4036` | wrong number of generic arguments |
| `TYHP4037` | struct property must be typed / required |
| `TYHP4043` | condition must be `bool` |
| `TYHP4101` | method/property not declared on the receiver's class, interface, or `object` type (also: extension not in scope in this file) |
| `TYHP4121` | empty `catch` swallows exceptions — name `$e` and `(void)$e` to discard on purpose |
| `TYHP4138` | can't infer closure parameter type — annotate it |
| `TYHP4139` | readonly `clone … with` on PHP 8.3–8.4 without `build.experimentalReadonlyCloneWith` |
| `TYHP4140` | readonly `with` on a `final` class, PHP &lt; 8.5 |
| `TYHP4141` | in-place `with` on a readonly property |
| `TYHP4400` | readonly `new … with` or `clone … with` when `output.phpVersion` is below 8.3 |
| `TYHP4401` | `clone … with` on a `readonly class` when `output.phpVersion` is below 8.5 |
| `TYHP4402` | `lazy new` of an internal class other than `\stdClass`, a class that extends one, or a non-class |
| `TYHP4403` | warning: `lazy new` of a class with no stored instance property runs the constructor immediately |
| `TYHP4300`–`TYHP4306` | PHP version gates ([§30](30-php-version-gating.md)) |
| `TYHP4307` | `.tyhp` used an `extern` name (type / function / const) |
| `TYHP4326` | user type listed `\Traversable`; use `\Iterator` or `\IteratorAggregate` |
| `TYHP4327` / `TYHP4329` | `#[\Tyhp\PhpType]` illegal target / invalid PHP hint spelling |
| `TYHP4328` | generator `TSend` intersection of typed `yield` targets is empty |
| `TYHP4330` | `ArrayAccess` `TKey` is a struct or array |
| `TYHP4331`–`TYHP4333` | `ArrayAccessShape` wide key / append / unhandled key |
| `TYHP4334`–`TYHP4336` | Closure bind incompatible / `'static'` stored as `TScope` / non-rebindable |
| `TYHP5018` | runtime `interopContractVersion` ≠ compiler |
| `TYHP8029` | `extern` kind mismatch vs a real type or another `extern` |
| `TYHP8031` | overlay `partial function` target missing (skipped) |
| `TYHP8032` | `partial function` outside an overlay tyhpdef |
| `TYHP8033` | `partial function` member outside an overlay `partial` type |
| `TYHP8034` | `partial class Foo;` header form outside an overlay tyhpdef |
| `TYHP8035` | overlay `partial class Foo;` has no effect (warning) |
| `TYHP7700` | `extra.tyhp.require` constraint disjoint from root pin |
| `TYHP7701` | stale extra missing from root require-dev / not installed |
| `TYHP7702` | `tyhp/compiler` missing from root require-dev (`--strict` fails) |
| `TYHP7703` | root `composer.json` parse/write failed during extras sync |

A call to a method or a read of a property that a fully-known class, interface, or `object` receiver
does not declare is `TYHP4101` (not reported when the class has `__get` / `__call` / `__callStatic`, part of
its ancestry is unresolved, or the name is dynamic; a `\property_exists` / `\method_exists` guard with a
literal name makes the member known). Passing too many arguments is `TYHP4143`.

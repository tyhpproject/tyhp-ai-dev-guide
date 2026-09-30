## 12. Extensions (add methods/operators to types you don't own)

The target is on the block. `$this` is that target and is not a caller parameter. `&$this` (no type) is only the by-reference receiver annotation.

```tyhp
<?tyhp
extension MoneyFormatting extends Money {
    function format(string $currency): string {
        return $currency . ' ' . $this->amount;
    }
    operator + (self $left, self $right): Money { return $left->plus($right); }
    operator + (int $left, self $right): Money { return $right; }
}
use extension MoneyFormatting;      // .tyhp requires explicit activation

extension NumericHelpers {
    extends int { fn abs(): int => \abs($this); }
    extends<T> \App\Box<T> { function id(): T { return $this->id; } }
}
extension BoxOps<T> extends \App\Box<T> {
    function mapTo<R>(callable(T): R $fn): R { return $fn($this->id); }
}
```

- Header `extends Type` covers every member. Nested `extends Type { }` / `extends<T> Type { }` groups are the mixed form (no header target, no loose members beside the groups). One level of groups. Header plus groups is `TYHP4362`; a member outside a group is `TYHP4363`; members and no target is `TYHP4147`; empty `{}` is `TYHP4172`.
- `extension Name<T extends Constraint> extends Target` — the `extends` inside `<…>` is the constraint; the `extends` after the name is the target. Same for `extends<T> Type { }`.
- Header and group type parameters live on that symbol and resolve by scope. They are not copied onto each method. Method type parameters are only the ones written on the method. Unused `T` is `TYHP4365`. A method generic that reuses an in-scope name is `TYHP4366`.
- Names follow PHP method naming (`match`, `and`, `or` are allowed). `abstract`/`final` not allowed. Members accept `#[…]` (e.g. `#[\Tyhp\Optimize\Pure]`). `#[\Tyhp\Optimize\Inline]` is `TYHP4176`.
- **Target.** One class, interface, enum, alias, builtin, or generic application, including `object`, `callable`, and `struct`. `?T` applies to `T` and to `?T`; `$this` is `?T`. A union is legal when every member is a legal target. The member applies when every type the receiver might be is a member of the union; `$this` stays the whole union. `extends string|int` applies to `string`, `int`, and `string|int`, not to `?string` or `string|int|bool`. A union that includes `null` applies to every non-empty combination of its members and not to a receiver that is only `null`. Intersection, `void`, `never`, `mixed`, a type parameter, an `object { … }` shape, `_`, or a union containing one of those is `TYHP4364`. Unresolved target: `TYHP3016`. Operator on `void` / `never` / `null` / `mixed` / `resource` / `true` / `false`: `TYHP3025`.
- **Overlap.** Two groups in one extension that can both match one receiver and declare the same method or operator are `TYHP4367`. Different names on overlapping targets are fine. Separate extensions are `use extension` / `insteadof`, not `TYHP4367`.
- **`$this`.** Not a source parameter and not a named argument. `&$this` only when the body writes `$this` (assignment, `++`/`--`, pass to `&`). Missing annotation: `TYHP4369`. Annotation with no write: `TYHP4370`. Object method calls and `$this->prop =` do not write `$this`. On a scalar, `array`, or struct, `$this->field =` / `$this[$k] =` / `$this[] =` does. PHP emits `$this_`, and `&$this_` only when the member is annotated `&$this` (objects included). A by-ref call needs a referenceable receiver (`TYHP4180`).
- **`self`.** `self` / `new self()` / `self::` are the block target, including scalars (`extends int` emits `int`, not the extension class). Name the extension by its identifier. `static::` and `parent::` are `TYHP4368`. An anonymous class nested in the member keeps its own `self`.
- **Hover.** Header: `extension Name extends Target` (generics on the name). Nested group: `extends Target` or `extends<T> Target`. Method signatures omit `$this`.
- **Form decides PHP:** `fn … => expr;` is Tyhp-only (spliced, no PHP method). `function … { return expr; }` is spliced in Tyhp **and** emitted so PHP can call it on the backer. Multi-statement `{ … }` is a real PHP method Tyhp calls. All-`=>` extensions emit no backer class. A written parameter must be `&`, and only those parameters are (`TYHP4174`). `++`/`--` in either position is a write — a non-mutating increment is `$v + 1`. A `&` argument must be referenceable. A repeated parameter is evaluated once.
- `$money->format('USD')` → splice of `=>` / single-`return`, or `\MoneyFormatting::format($money, 'USD')` for a multi-statement member. Operators emit `OperatorMethodNameGenerator` names (`__add`, …).
- Writing the target on the member (`extends Type $this` or `operator op<Type>`) is `TYHP4361`.
- **`tyhp/core` scalars are already on:** `global use extension \Tyhp\StringExtensions` (and Array / Int / Float / Bool / Closure). No per-file import. Local `use extension \Tyhp\StringExtensions;` with no adaptations → `TYHP4169`. Local `use` only to `hide` / `as` / `insteadof`. `$s->length()` → `\Tyhp\StringHelper::strlen($s)`; `byteLength()` → `\strlen`.
- Tyhpdef class-body members ([§23](23-tyhpdef.md)) are thin `=>` mappings. `$this` / `self` are the enclosing class. They erase (PHP cannot call them) unless `#[\Tyhp\GenericRuntime]` is present — then foreign `$s->jsonDecodeAs<User>()` uses `\Tyhp\Generic::bind`. `.tyhp` needs `use extension` unless a loaded tyhpdef issued `global use extension`. A compiled library that authored in-package `global use extension` copies it into `package.tyhpdef`.
- `use extension E { E::foo as bar; E::old hide; E::operator *<string> hide; F::operator +<T> insteadof E; }` — `as` aliases methods only (not operators, `TYHP4170`). The `<Type>` here is an adaptation qualifier, not a declaration. `hide` of a name drops that member; `operator * hide` hides every `*` on that extension; `operator +<Money> hide` hides every `+` for that target. `insteadof` chooses which extension wins.
- `.tyhpdef` standalone `extension Name { }` uses the same header and groups (`=>` only; no brace bodies; no generated PHP). Multi-statement bodies live in `tyhp_src/` with `class PhpName as Name__tyhpExtensionBacker`. `#[\Tyhp\Php]` is illegal on `extension` — wrap with `declare(php=…)` ([§30](30-php-version-gating.md)).

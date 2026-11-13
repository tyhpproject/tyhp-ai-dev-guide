# Regenerating the Tyhp language guide

Use this when Tyhp changes (new features, cleared gotchas, syntax updates) and the guide needs to be
rebuilt. It contains (1) context on how the guide was produced and (2) a ready-to-use prompt.

Regeneration needs **both** of these sibling clones, plus this repository as the output:

- Compiler: `../tyhp` (grammar, compiler source, tests, `dev-docs/`). [tyhp](https://github.com/tyhpproject/tyhp).
- Runtime package source: `../tyhp-runtime-src` (`packages/`). [tyhp-runtime-src](https://github.com/tyhpproject/tyhp-runtime-src).

Paths below assume that sibling layout (`~/repos/tyhp`, `~/repos/tyhp-runtime-src`, and this clone side by side). Set the paths to those checkouts if they live somewhere else. Compiler sources are under `../tyhp`. Package sources are under `../tyhp-runtime-src/packages`.

Output is **this repository root** (no extra directory prefix):

```
AGENTS.md  CLAUDE.md  QUICK_GUIDE.md  SKILL.md  README.md  REGEN.md
guide/     00-index.md + 01-…-31-*.md   (the dense language delta, one file per section)
handbook/  00-index.md + 01-…-08-*.md   (setup/CLI/interop/testing/examples/runtime API, rhetoric)
```

The content is authored as two logical documents (the **guide**, ~31 sections, and the **handbook**,
~7 sections) but written out **one file per section** so an agent loads only what it needs.
`QUICK_GUIDE.md` / `SKILL.md` are the one-line-per-feature index whose `→` pointers name the exact section files;
`AGENTS.md`/`CLAUDE.md` are the entry points.

---

## How the guides are built (method)

1. **Authoritative sources only** (everything else may be stale):
   - Grammar: `../tyhp/Tyhp/TyhpLang/Grammar/TyhpParser.g4`, `../tyhp/Tyhp/TyhpLang/Grammar/TyhpLexer.g4` (syntax = ground truth).
   - Current compiler code under `../tyhp/Tyhp/TyhpLang/**` and runtime packages under `../tyhp-runtime-src/packages/**`
     (**behavior/emit = ground truth; code wins over stories**).
   - Implementation stories `../tyhp/dev-docs/IMPLEMENTATION_PLAN_TODO_STORY_*.md` (intent/semantics).
   - `../tyhp/dev-docs/ROADMAP.md` (sequencing, what's planned vs done).
   - Concrete fixtures `../tyhp/tests/Tyhp.Tests/TestData/ValidTyhp/**/*.tyhp` and emitter tests
     `../tyhp/tests/Tyhp.Tests/Emitter/*.cs` (real syntax + expected PHP output).
2. **Explore in parallel, then verify.** Extract semantics per area (type system, checker/narrowing,
   emitter/PHP lowering, tyhpdef/runtime/config) — then confirm emit/availability claims against the
   emitter and checker code before writing.
3. **Maturity is author-facing, not compiler-jargon.** For each feature decide: compiles cleanly →
   "use freely"; parses/checks but emit is broken or missing → "use the PHP form instead"; not in the
   toolchain → "not in the language yet." Base this on the **code**, not the stories.

---

## The prompt (paste to an agent that can read the sibling compiler and runtime-src clones and write this repository)

> **Task:** Regenerate the Tyhp AI dev docs in this repository (English). Author the language guide
> (~31 sections) and the handbook (~7 sections), but **write one file per section** into
> `guide/NN-slug.md` and `handbook/NN-slug.md`; also (re)produce
> `QUICK_GUIDE.md`, `AGENTS.md`, `CLAUDE.md`, and a `00-index.md` in each folder. Overwrite existing
> files at the repository root. Produce **English only**.
>
> **Audience:** an AI agent that is already 100% proficient in PHP 8.x and will write/maintain a Tyhp
> *application* (not the compiler). The guide is a dense, token-efficient info dump of only what is
> **new or different** vs PHP, and how each construct maps back to PHP. Anything that behaves like
> PHP is omitted.
>
> **Sources of truth (use only these; other docs may be outdated).** Paths are sibling clones of this repository:
> - `../tyhp/Tyhp/TyhpLang/Grammar/TyhpParser.g4`, `../tyhp/Tyhp/TyhpLang/Grammar/TyhpLexer.g4` — syntax.
> - Current code in `../tyhp/Tyhp/TyhpLang/**` and `../tyhp-runtime-src/packages/**` — **behavior and PHP emission; the
>   code wins over any story when they disagree.**
> - `../tyhp/dev-docs/IMPLEMENTATION_PLAN_TODO_STORY_*.md` — semantics/intent.
> - `../tyhp/dev-docs/ROADMAP.md` — planned vs done.
> - `../tyhp/tests/Tyhp.Tests/TestData/ValidTyhp/**/*.tyhp` and `../tyhp/tests/Tyhp.Tests/Emitter/*.cs` — real syntax
>   and expected PHP output.
> Prefer delegating the per-area extraction to parallel exploration subagents, then verify emit and
> availability claims directly against the emitter/checker code.
>
> **Content — cover every section below, in order. Keep it complete; do not drop features.**
> 1. Mental model (Tyhp is a **typed superset of the PHP language**; compiles to PHP; types erased in
>    hints; source `type` aliases emit a `\Tyhp\Type` factory; property-hook polyfill when
>    `output.phpVersion` < 8.4; `async`/`await` → `\Tyhp\Promise::_async` / `_await`; structs →
>    arrays with no runtime shape check; remaining `$x is T` → `\Tyhp\Type::is`; `\Tyhp\` runtime
>    for a few features; stricter than PHP; locals lead with `$x = …`, `int $x = …` when inference
>    is not enough) + one small orientation example.
> 2. Files & open tags (`.tyhp`/`.tyhpdef`/`.php`, `<?tyhp `/`<?tyhpdef `, tagless mode). In a
>    `.tyhp` file `<?= expr ?>` is a Tyhp echo (parsed and checked as Tyhp, emitted as `echo`, no
>    escaping; `is` / `typeof` / `with` / `nameof` active; non-stringable is TYHP4120; a missing
>    member is TYHP4101 on the `.tyhp` line). `.php` `<?=` is unchanged. HTML emits as written
>    (source indentation kept; one newline after `?>` swallowed; the pretty-printer does not indent
>    HTML). `if (…):` / `endif;` and `foreach (…):` / `endforeach;` lower to braces. `source.tagless`
>    forbids `?>`, so a template is a tagged file (first 6 characters `<?tyhp` in a tagless project).
>    Capture with `\ob_start()` / `\ob_get_clean()`.
> 3. The two strict rules: non-nullable by default (absence is `?T` assigned `null`; `unset` of a local, a typed instance property, or a static property is TYHP4158; array-offset `unset` stays legal; a possibly null receiver must be narrowed before
>    `->` property/method access or struct/object indexing — TYHP4198; `?->` / `??` are the
>    shorthands; a `?->` chain's value is nullable, later links in the same chain skip with it);
>    conditions must be `bool`; narrowing resets on reassignment.
> 4. Type system: known types now enforced (`static` return-only; `resource` not user-writable); new
>    types (`decimal`→`\Tyhp\Decimal`, `struct`→`array`, literal types); unions/intersections/`?T`;
>    `mixed` collapse; `decimal` creation (`\Tyhp\decimal()`, `new Decimal`, `(decimal)` cast from
>    `string`/`int`/`float` only — TYHP4008 otherwise) and operator-first usage (operators on `decimal`
>    return `decimal`/`bool`/`int`; methods only for `divide($x, $scale)`, `round`, `withScale`, `sqrt`,
>    `format`); `(object)` is identity on an already-object operand
>    (concrete class, builtin `object`, all-object union) and converts array/scalar/null/`mixed` to
>    engine `\stdClass` (not builtin `object`; emit stays `(object)`); undeclared property writes
>    (TYHP4134) are legal only on exact `\stdClass` — subclasses declare properties or use
>    `__get`/`__set`; `#[\AllowDynamicProperties]` is not an opt-in.
> 5. Type inference / what must be annotated: params & properties & consts always typed; return types
>    and non-initialized locals always required; **locals lead with `$x = expr` (inferred)**;
>    `T $x = …` is the explicit form when inference is not enough; **array literals are NOT inferred — annotate them**; literal-type inference; a call
>    to a callee with no declared return → `mixed`; closure params inferred from call-site context;
>    `mixed` is one-way. Cite `TYHP4016`/`TYHP4138`.
> 6. Assignability & subtyping: nullable/union/intersection rules; literal→base; `int`→`float`;
>    `iterable` vs `array`/`\Traversable`; struct width subtyping; `mixed` top / `never` bottom;
>    variance (user generics **invariant**, `array`/`iterable` covariant, `callable` params
>    contravariant + return covariant; `\Closure` `TThis`/`TScope` invariant).
> 7. Generics (erased from signatures): declaration, `extends` constraints, `= Type` defaults (note defaults not yet
>    applied — pass all type args), generic args on `extends`/`implements`, erasure rules,
>    compiled libraries stamp `#[\Tyhp\GenericRuntime]` on every emitted generic (erased callables, classes, and type aliases are `layouts: [1]`; a non-erased callable that is not `#[\Tyhp\InferGeneric]` is a layout-2 binder under the declared name, `Json::decode($T)($json)`; `#[\Tyhp\InferGeneric]` keeps the value signature and a `name__tyhpGeneric` companion stamped `layouts: [1]`; explicit type arguments use the companion; `Generic::bind` binds layouts 1 and 2 with `\Tyhp\TypeLike|string`; PHP calls `Box::new(T::int)(42)` and `Box::type(T::int)`, `new()` matches constructor visibility and is not `final`, abstract classes get `type()` only, interfaces and traits get neither; a Tyhp `new` of a foreign class emits `Generic::bind`; a foreign layout-2 call emits `name(types)(args)`; TYHP4404 magic methods, TYHP4405 overriding a non-generic method, TYHP4406 a method named `new` or `type`, TYHP4407 `InferGeneric` on a type parameter no argument can supply, a by-ref parameter is not a source, TYHP4408 warning on a non-generic declaration; `ArrayAccessShape::offsetGet` and `offsetSet` are not generic; overlays that omit the stamp keep erase/inline; `#[\Tyhp\EraseGeneric]` opts properties out of tracking),
>    `#[\Tyhp\PropertyType('…')]` on properties / promoted params whose PHP hint is lossy (type args, structs, source aliases; not when a type parameter appears; written stamps replaced) read by `\Tyhp\Type::fromProperty()` + `elementType()`/`typeArguments()`,
>    `array<T>`/`array<K,V>`, `iterable<…>`, callable shapes `callable(…): R`
>    (`callable(): bool` is 0-arg; `callable(...): TReturn` is an `extends`-only any-arity bound;
>    packs auto-splice inside a callable-shape parameter list; postfix `T...` is a trailing PHP variadic),
>    `\Closure<callable(...): mixed, TThis, TScope>`, `\Fiber<TResume, TCallableShape>`,
>    `\Generator` declared-return shapes, `ArrayAccess<K,V>` / `ArrayAccessShape`, Traversable
>    implements rule, alias return-constraint propagation.
> 8. Typed locals (erased; only params/props/returns keep spelled types). Lead examples with inferred
>    `$x = …`; `int $x = …` when inference is not enough. A typed local may omit the initializer;
>    a read before assignment is TYHP4014. `unset` of a local is TYHP4158; drop the value with
>    `?T` and `= null`. `unset($items[$i])` stays legal.
> 9. Type aliases (hints expand to the underlying PHP type; source `.tyhp` aliases emit a
>    `\Tyhp\Type` factory — `UserId()`, `Optional(typeof(int))` / `typeof(Optional<int>)`;
>    `Optional<int>()` is invalid; `use App\Types\UserId;` stays class-kind, emitted PHP is
>    `use function` when the factory is used; class-level `self::NameType()` /
>    `UserService::NameType()`; tyhpdef aliases inline unless `#[\Tyhp\GenericRuntime(aliasFactory: …)]`).
> 10. Structs (structural value type, properties-only, `extends`; struct vs class table incl. width
>     subtyping; primary construction is the bracket list `new Name[field => value]`, including empty
>     `new Name[]`, `new Box<int>[value => 1]` and `new Box[value => 1]` (type arguments from the target
>     type), and
>     `new struct { … }[field => value]`; state `new Name()` and `new Name() with […]` once as the same
>     forms; keys are property names, bare or quoted, including a keyword in key position (`class =>`),
>     not positions; a field alias is still the property name in the list; an unkeyed entry is an error;
>     a trailing comma is allowed, and a newline may sit between the name or `}` and `[`; spread
>     `new Opts[...$base, name => 'bob']`: the source is a named struct, a generic struct instance, or an
>     anonymous struct (`array`, `array<K, V>`, `mixed`, a nullable struct, and a union are errors), no
>     extra fields (including through `extends`), field types assignable, PHP array keys match so spread
>     cannot rename an alias (`'status_code' as $statusCode`, `0 as $arg1`), a required field is satisfied
>     by a key or by a spread that requires or defaults it, a nullable source field without a default does
>     not, defaults apply first and later entries override, and spread field types are checked against the
>     generic instantiation the same way keyed values are; those rules also apply to a struct `with` list (`new Name() with`,
>     `clone $s with`, `$s with`); emit spread as `\array_replace` (remaining defaults, then each spread
>     and each run of keyed entries), not PHP `[...$src]`, so integer keys survive; `new Name[...] with [...]`
>     is an error — one list; dereference with `(new Point[…])->x`; property access lowers to arrays; PHP
>     sees `array`; no runtime shape check; `unset` of an optional field is legal (array-backed);
>     `unset` of a class property is TYHP4158; anon/`clone` gotchas).
> 11. Operator overloading (class + shorthand + `convert`; arity/`self` rules; overloadable set;
>     left-first resolution; synthetic-method lowering). `operator default(): self` on a class or enum
>     (no parameters, at most one, static body, return `self`): `default(ThatType)`, `default(self)`, and
>     `default(static)` then have that type and call `ThatType::__tyhpDefault()` (`public static`, return
>     `static`). Each call runs the body. The call is not a constant (TYHP4090). Without the operator,
>     including a subclass whose parent declares it, the result is null and is a constant. Not allowed on
>     an interface, abstract class, trait, extension, or as an abstract operator (TYHP4423). `$this` is
>     TYHP4097. Direct `default` of the same type in the body is TYHP4424; omitting a constructor parameter
>     whose default is `default` of that type is TYHP4425. `__tyhpDefault` is reserved in any letter case, declared operator or not (TYHP4426).
>     Hand-written PHP may declare the method; `Type::defaultValue()` calls it only when that exact type
>     declares it. `default(T)` for a generic parameter goes through `Type::defaultValue()`.
> 12. Extensions (block target `extension Name extends Type` or nested `extends Type { }` / `extends<T> Type { }`; `$this` implied, `&$this` only when by-ref; `self` is the block target including scalars; unions legal when every receiver alternative is a union member; header generics stay on the extension symbol; three member forms: `=>` omits the
>     PHP method, single-`return` brace splices and emits, multi-statement is a real method Tyhp
>     calls; written params must be `&`; static-class lowering; `use extension`; `tyhp/core` scalars
>     via `global use extension \Tyhp\StringExtensions` (no per-file import; local `use` only to
>     adapt; `TYHP4169` if redundant); tyhpdef class-body members are thin `=>` mappings that splice
>     unless `#[\Tyhp\GenericRuntime]` is present (foreign sites use `\Tyhp\Generic::bind`); compiled libraries copy in-package
>     `global use extension` into `package.tyhpdef`). The optional PHP module is `ext-tyhp`
>     (`pie install tyhp/ext-tyhp`, loaded name `tyhp`, `phpversion('tyhp') >= 0.1.0`). It is not required.
>     Installing it is recommended. A missing or older module is ignored with no warning. `tyhp/core`,
>     `tyhp/lambda`, and `tyhp/io` suggest it and do not require it.
> 13. async/await (Fiber/Promise; `\Tyhp\Promise::_async/_await`; outer return `\Tyhp\Promise`; **declared
>     async return type is the settled type** (`__Awaited<R>`), `Promise<T extends __Awaited<void|mixed>>`, TYHP4373; no
>     async generators; Promise API surface incl. `then` overloads and `Promise::run` typing; **cancellation** via `CancellationToken(Source)` +
>     `OperationCancelledException`/`TimeoutException`, `run(…, $token)` loop shutdown, unhandled-rejection behavior; `foreach (await
>     …)` gotcha; blocking PHP I/O blocks the `tyhp/async` loop).
> 14. Deterministic disposal `using`/`:=` (`IsDisposable`/`AsyncIsDisposable`; try/finally and
>     `DisposableScope` lowering; **failure semantics**: reverse order, dispose-throw masks the body
>     exception, multi-resource → `AggregateException` of disposal errors only, `:=` `__destruct` only
>     warns).
> 15. `with` expressions (clone/new/in-place forms; readonly rule; structs **and** classes compile;
>     object `with` mutates identity, struct `with` rebinds an array — PHP object-vs-array argument
>     analogy; readonly `new`/`clone … with` below 8.3 is TYHP4400; PHP 8.3–8.4 readonly `clone … with`
>     needs `build.experimentalReadonlyCloneWith` and is TYHP4401 on a `readonly class`).
> 16. Type guards + narrowing + `is` (guard `: $x is T`→`: bool`; `is` emit is tyhpdef-driven:
>     `#[\Tyhp\NativeTypeTest]` on a free function or concrete static method with `$param is T`
>     → `\Fqn($x)` / `\Class::method($x)`; unmarked
>     class/interface/enum/trait → `instanceof`; else `\Tyhp\Type::is`. `$x is null` allowed.
>     Narrowing table incl. loose `!=`/`==` null; if-arm early exit (also with `else`); `\assert($c);`;
>     `switch (true)` / `match (true)` earlier-arm negation; what does not narrow; property reads
>     `$this->p` / `$obj->p` / nested `$a->b->c` / `self::$p` / `Foo::$p` narrow like variables,
>     assignment clears the path and sub-paths, method-call results do not). Closure/Fiber
>     generic shapes: `as`, not `is`.
> 17. Compile-time helpers (`nameof(User::$name)` is `__PropertyName<User>` and a constant
>     expression, so an attribute parameter of that type accepts it and rejects a name that is
>     not a property of `User`; `User::class` is `__ClassName<User>`; `default(T)` (a class or enum
>     `operator default(): self` makes `default(ThatType)`, `default(self)`, and `default(static)` that
>     instance via `__tyhpDefault()`; otherwise null, including a subclass and a constant context; see §11), `typeof` takes a **type** — `typeof(int)`,
>     `typeof(Optional<int>)`, source alias → factory; value path is `\Tyhp\Type::of($value)`;
>     `variable_exists` → `\array_key_exists`).
> 18. String-as-type features (symbol-name types list + existence-check narrowing;
>     `__PropertyName<T>` accepts `nameof(T::$prop)` and a literal property name;
>     `__ClassName<T>` accepts `T::class`; template string
>     types + quantifiers; holes may name aliases (`${ApiMethod}`, composing template aliases such as
>     `"${ApiMethod} ${ApiPath}"`; unknown names are errors; a hole must be a string, `int`, or
>     `float` type, literal, or union — `${array}` / `${bool}` / `${mixed}` / `${$x}` / an inline
>     nested template are errors (TYHP4199)); numeric holes take the text PHP's `(string)` cast
>     produces (`${int}`: `'3'`, not `'03'`); a double-quoted interpolation whose text
>     and hole types fit the pattern is assignable to it (`${string}` holes take any interpolated
>     value; a possibly-null value is `''` when null, so it needs a hole that accepts `''` or other
>     text in the same `${string}` hole); all erase to `string`; include `__SuperTypeName`).
> 19. Utility types (global `__Partial<T>` `__Pick<T,K>` `__Record<K,V>` `__Awaited<T>`
>     `__Nullable<T>` `__NonNullable<T>` `__AsReadOnly<T>` …; callable-keyed `__CallableReturnType` /
>     `__CallableParametersStruct` / `__CallableParametersTuple` / `__CallableParametersRest` /
>     `__CallableParametersSlice`; packs splice in a callable-shape parameter list; `__Nullable`/`__NonNullable` map packs;
>     `__CallableThis` / `__CallableScope` / `__SuperType` / `__SuperTypeName` / `__CurrentScope` /
>     `__IndexKeys` / `__IndexValueType` / `__IndexValueTypes`).
> 20. Property accessors (= PHP 8.4 hooks; `.tyhp` has bodies; polyfill on 8.2–8.3, native on 8.4+;
>     `&get` needs 8.4; `.tyhpdef` describes hooks as bodyless `{ get; set; }` / `{ &get; }`, no
>     `get { }` / `get =>`; `#[\Tyhp\Php]` on the property not the hook).
> 21. Expression trees / parsable lambdas (`Expression<callable(...)>`/`PropertyPath<callable(T): R>`;
>     inline-`fn` only; `TCallableShape`; `->callable`; `tyhp/lambda`; experimental;
>     a generic `R` is inferred from the inline `fn` body for both (also in a union with `callable(T): R`; `?->` gives nullable `R`);
>     body rules: nested arrow `fn` (params typed from the callee incl. a `PropertyPath` param, may read outer params, shadowing by name,
>     no by-ref/variadic/default params), list/keyed array literals (spread rejected), TYHP4322 for a class in a
>     variable (`dynamic class`), a variable member name (`dynamic member`), named/unpacked call args, `?->` method calls with
>     PHP chain short-circuit; an extension method call stays `MethodCallExpression` (the body is not substituted into the tree; `compile()` still runs it);
>     nodes `LambdaExpression` / `ArrayExpression` / `NullSafeMethodCallExpression`;
>     parameter / property node `->type` is the full declared type as a `\Tyhp\Type::parse()` spelling (`App\Table<App\Blog>`, `array<App\Post>`);
>     `ExpressionSerializer::shapeKey` / `shapeParameters` for per-shape translation caches).
>     `tyhp build` emits `backedBy` chains for an inline `fn`.
>     A parameter typed `Expression` or `PropertyPath` is `\Tyhp\Expression|\Closure` or `\Tyhp\PropertyPath|\Closure`
>     with a `fromClosure` prologue, except a promoted, by-ref, variadic, property-hook, inline-closure, or
>     `#[\Tyhp\PhpType]` parameter, or a union with a type other than `null`. PHP packages use `_expr` / `_path` /
>     `Expression::of` / `PropertyPath::on`. A caller who knows `ext-tyhp` >= `0.1.0` is loaded may pass a closure.
>     `fromClosure` uses `closureTree` or tokenizes the source and throws `\InvalidArgumentException` for a closure
>     it cannot read, including a return type PHP checks such as `: bool` and a `use` clause.
>     `PropertyPath::fromClosure` needs one parameter and a property chain.
> 22. Declarations — deltas vs PHP (declaration-site generics on class/interface/trait/enum/function;
>     **constructor return type `: void` (optional; omitted ≡ `: void`) / `: parent(...)`**; `async` modifier; return-type guards;
>     top-level overload signatures; trait property alias `$prop as $x`; `internal` omits from
>     published `package.tyhpdef` (members emit public; cannot combine with public/protected/private — `TYHP4002`);
>     `#[\Tyhp\PhpType]`; Traversable implements rule; enums are `\UnitEnum` (backed also
>     `\BackedEnum`) without a written `implements` — do not list those interfaces or redeclare
>     `cases` / `from` / `tryFrom` (`TYHP4337` / `TYHP4338`); engine attributes: `#[\Deprecated]`
>     use-site TYHP4500 (literal `$message` in `{1}`; still warns when phpVersion < 8.4; same code
>     as tyhpdef `deprecated`); `#[\Override]` methods TYHP4129 if not overriding, properties legal
>     ≥ 8.5 else 4127, `__construct` always TYHP4339; `#[\NoDiscard]` unused-return TYHP4165 from
>     the invoked declaration (interface/abstract callees do not warn; overrides do not inherit
>     unless marked; literal `$message`; `(void)` suppresses; not gated on phpVersion);
>     `#[\DelayedTargetValidation]` ≥ 8.5 skips TARGET (4127) for Core attributes on that
>     declaration, not functional checks; **initialized storage**: `unset` of a local, typed
>     instance property, or static property is TYHP4158 (`?T = null` drops the value; array-offset
>     `unset` and an optional struct field stay legal); every typed static on a class, trait, or
>     anonymous class needs an initializer (TYHP4418), including `?T` and `mixed`; a `.tyhpdef`
>     may document a PHP static with no default; interface properties, enums, and statics of a
>     readonly class are not TYHP4418; after construction every typed instance storage property
>     is initialized (TYHP4419; TYHP4157 is the read); a declaration initializer counts when a
>     subclass replaces `__construct`; a promoted parameter counts only when its constructor runs
>     (an inherited promoted parameter with a default does not, if the child does not run that
>     constructor); `parent::__construct` or `: parent(...)` counts only on every path to the end
>     and every `return` (one `if` arm does not cover the other; `throw` abandons the object; a
>     helper or closure does not count; direct `$this->prop =` does); a class with no constructor
>     runs the nearest ancestor constructor, including a trait constructor PHP would import, and
>     a grandchild that also declares none is the same; a private ancestor property is named on
>     the child constructor when skipping the parent would leave it uninitialized, and a private
>     property with a declaration initializer does not need that call; trait properties are checked
>     on each using class; hooked properties are not TYHP4419; calling a tyhpdef constructor, or
>     inheriting it by declaring none, counts for that stub's uninitialized typed instance
>     properties, and a tyhpdef chain with no constructor is not asked for; an anonymous class is
>     `class@anonymous`; everything else identical to PHP — say so explicitly).
> 23. `.tyhpdef` declaration files (what's declarable, including hooked class/interface/trait
>     properties as bodyless `{ get; set; }` / `{ &get; }`; a static property may omit a default
>     (TYHP4418 and TYHP4419 are `.tyhp` checks; calling or inheriting a tyhpdef constructor counts
>     for that stub's typed instance properties with no initializer; a stub chain with no constructor
>     has nothing to call); class-body `type`; `deprecated`/`obsolete`; `extends` import;
>     inline + standalone `extension { fn … => …; }` as thin mappings only — no brace bodies, no
>     generated PHP; a written parameter must be `&`; compiled libraries copy in-package `global use`; `hide`/`insteadof`; `#[\Tyhp\GenericRuntime]` stamps every emitted generic (`erased`/`layouts`; helpers optional); PHP gates;
>     harvested enums may keep `implements \UnitEnum` / `\BackedEnum`;
>     `extra.tyhp.package` `include` vs `overlay` (stubs then hand, last Tyhp name wins, overlay
>     `partial` header `;` / brace member merge / overlay `partial function` keep-or-`as`-rename / `omit`, `// @overlay-against:`); `extern \Name;` kind-unspecified placeholders
>     (or `extern class` / `interface` / `enum` / `function` / `const` — name-only; specified type
>     kind must match the originating declaration, mismatch is `TYHP8029`); library `tyhp build`
>     emits `extern` for author-only `require-dev` owners into `package.tyhpdef`; `extra.tyhp.require`
>     ambient FQNs (any Composer name; duplicate in require-dev; never put author-only tyhpdefs in extras);
>     `internal` omitted from published tyhpdef; `tyhp generate_tyhpdef`).
> 24. PHP interop & name resolution (compiled output is ordinary PHP in same namespaces; resolution
>     like PHP; **symbol discovery order** built-in → embedded tyhpdef → package → user tyhpdef → user
>     `.tyhp`, first-registered-wins).
> 25. Runtime packages table (`tyhpdef/php` always-present 8.2–8.5 builtins;
>     `tyhpdef/php-ext-*` optional extensions, Composer `php: >=8.2`, no per-minor forks; `tyhp/core`
>     (`PhpType`, `GenericRuntime` layouts 1 and 2 / `InferGeneric` / `PropertyType` runtime-visible, `EraseGeneric` compile-only, `Type` constants `T::int` as `BuiltIn`, `TypeLike` is `toType` and `or` only, `Generic::bind` takes `TypeLike|string`, `Box::new` / `Box::type`, `ArrayAccessShape` with non-generic `offsetGet` / `offsetSet`, scalar methods) / `decimal` / `async` (`Tyhp\async`, `Tyhp\await`, `$promise->wait`; InferGeneric plain calls vs binder-first `reject` / `withResolvers` / `allSettled` / `batch`) / `lambda` (`_expr` / `_path` companions, `Expression::of` / `PropertyPath::on` InferGeneric, `lambda` / `from` / `backedBy` binders, `fromClosure`, which reads the closure through the `tyhp` module at >= `0.1.0` and tokenizes its source otherwise; PHP packages keep the fluent calls), key symbols).
>     `ext-tyhp` is suggested by `tyhp/core`, `tyhp/lambda`, and `tyhp/io` and required by none (`pie install tyhp/ext-tyhp`, loaded name `tyhp`).
>     DNS uses that module at `0.1.0` or newer and `Offload` otherwise. An unresolved host is `IoException`
>     (`Could not resolve host {host}.`) or, when `Offload::usesWorkers()` is true, `ParallelException` whose
>     remote class is `IoException`, on both paths. The `ParallelException` message from `Offload` may also name
>     the worker file and line. A cancelled token is `OperationCancelledException`.
>     `php-ext-decimal` is PECL Decimal, not `tyhp/decimal`. json/hash/libxml stay in `tyhpdef/php` only.
>     Package tests run `../tyhp-runtime-src/packages/test-all-tyhpdef.sh`.
> 26. Build, output layout & CLI: `tyhp.json` keys, including `emit.docblocks` (`lossy-only` default,
>     `all`, `off`; every profile; independent of `output.comments`; `@param`, `@param-out`,
>     `@return`, `@var`, `@template` with `of` / variance / default, `@extends`, `@implements`,
>     `@use`, `@internal`; `@throws` is not generated; author tags win per key; `off` emits no
>     generated tags, including `@template` and lowered-hook `@property`) and `emit.docblockDialect`
>     (`both` default, `phpstan`, `psalm`, `phpdoc`; parsed; does not change output; unprefixed
>     shared syntax; a `@phpstan-` / `@psalm-` prefix only where the dialects diverge — none today);
>     **output layout** (classes → FQN segments under
>     `output.path`; entrypoints mirror source; `_functions.php`); library `package.tyhpdef` keeps
>     attributes, class-level `type`, in-package `global use`, `#[\Tyhp\GenericRuntime]`; libraries
>     additive-merge `extra.tyhp.package` onto publish-directory `composer.json` (applications do not);
>     `composer.json` `require`/autoload only when
>     `build.updateComposer` (path repos under `../tyhp-runtime-src/packages/` + `composer install`); **all-or-nothing error gate** (no
>     partial emit); incremental build state file; CLI commands/flags including `tyhp overlay create`
>     / `stamp`, `tyhp symbol_tree --filter` / `--out`, `generate_tyhpdef --php-targets`, `tyhp composer sync`
>     / `tyhp build --fix` for `extra.tyhp.require` (default explain, `--strict` fails, ordinary build never auto-updates;
>     first `composer require tyhp/core` often misses that transaction), `allow-plugins.tyhp/core`, and the 8.2–8.5 `output.phpVersion` lint/build
>     matrix.
> 27. Diagnostics: `TYHP4xxx` codes, error/warning/info, checker continues after errors (cap
>     100/file), common-codes table (include `TYHP4014` read before assignment, `TYHP4157` uninitialized property read, `TYHP4158` `unset` of a local or property, `TYHP4418` static property without an initializer, `TYHP4419` instance property not initialized after construction, `TYHP4420`–`TYHP4426` `operator default`, `TYHP4121` empty `catch` → name `$e` and `(void)$e`;
>     a comment-only body still warns; `TYHP4500` deprecated use; `TYHP4129` / `TYHP4339` Override;
>     `TYHP4165` NoDiscard unused return); note there is no dedicated unknown-member / arg-count error.
> 28. Availability & gotchas: "use freely" list (property hooks polyfill on 8.2–8.3, native on 8.4+;
>     `&get` needs 8.4; **`tyhp/core` scalar catalog is use-freely** — `$s->length()` with no import);
>     "annotate don't assume" (arrays/params/returns; a typed local may omit its initializer, TYHP4014 on a read); `operator default(): self` on a class or enum is use-freely (`default(ThatType)` is that instance; without the operator, `default` of that class or enum, `default(self)`, and `default(static)` are null — do not list the operator under "not in the language yet"); HTML templates in `.tyhp` (`<?=` checked
>     `echo`) are use-freely; bare `<?` is not a short open tag; a **storage** section: absence is `?T = null`; `unset` of a local, typed instance property, or static property is TYHP4158 (do not list it under "not in the language yet"); array-offset `unset` and optional struct fields stay legal; TYHP4418 on every typed static of a class, trait, or anonymous class, including `?T` and `mixed`; TYHP4419 after construction; a declaration initializer counts when a subclass replaces `__construct`; a promoted parameter counts only when its constructor runs; `parent::__construct` or `: parent(...)` must run on every path to the end and every `return`; a class with no constructor runs the nearest ancestor constructor, including a trait constructor (do not write that a missing constructor skips the parent); "use the PHP form instead"
>     table (include readonly property defaults below 8.6 → `TYHP4378`, enum `__debugInfo()` below 8.6
>     → `TYHP4379`, other enum magic methods → `TYHP4024`); "not in the language yet" list (`internal` **is** in the language — omit from published
>     `package.tyhpdef`; storage `unset` is an error, not a missing feature). **Derive strictly from the current code.**
> 29. PHP-mapping reference table (`<?= expr ?>` in `.tyhp` → `echo expr;`; a lossy signature also
>     emits a PHPStan/Psalm tag under `emit.docblocks`; source `type X = …` → factory `function X(): \Tyhp\Type`;
>     hints still expand; tyhpdef aliases inline unless `#[\Tyhp\GenericRuntime(aliasFactory: …)]`;
>     a layout-2 call emits `name(types)(args)`; `#[\Tyhp\InferGeneric]` bare call stays plain and explicit type arguments emit `name__tyhpGeneric`; PHP `Box::new(T::int)(42)` / `Box::type(T::int)`; a foreign class `new` still emits `Generic::bind`; PHP `await(async(...))` and `$promise->wait()`; `default(Class)` with `operator default` → `Class::__tyhpDefault()`, otherwise `null`).
> 30. PHP version gating: `declare(php="…")` (alone in that declare; file `;` vs `{ }` block);
>     `#[\Tyhp\Php]` (illegal on `struct`/`extension`); Composer constraints vs `output.phpVersion`
>     (unset → `8.2` + TYHP4306); disjoint same-name variants; `output.phpVersion` is the minimum PHP; gates that hold for only some versions at or above it emit `\PHP_VERSION_ID` checks, `declare(ext)` emits `\extension_loaded`.
>     `declare(ext="name")` / `declare(ext="!name")` (alone; loaded tyhpdef `require` `ext-name`).
>     `fallback function` / `fallback const` (autoload.files winner; `tyhpdef/php` and `ext-*`
>     providers beat fallbacks). Nest php and ext. Harvest emits these from top-level
>     `function_exists` / `defined` / `extension_loaded` / `PHP_VERSION_ID` gates.
>     Engine-attribute classes stay gated in tyhpdef (`Deprecated` ≥ 8.4, `NoDiscard` /
>     `DelayedTargetValidation` ≥ 8.5) but Tyhp still runs Deprecated / NoDiscard checks below those
>     minors; Override-on-property and DelayedTargetValidation TARGET skip follow phpVersion ≥ 8.5.
> 31. Lazy objects: `lazy new Class(args)` / `lazy new Class<T>(args)` only (`lazy` is a keyword only
>     before `new`); arguments run at the expression, the constructor runs later on that instance when
>     `output.phpVersion` ≥ 8.4, ordinary `new` below that; a class with no stored instance property
>     stays eager and warns `TYHP4403`; `TYHP4402` for an internal class other than `stdClass`, a
>     class extending one, or a non-class; a tracked generic is lazy and binds its type arguments;
>     `\Tyhp\Lazy` (`ghost` is binder-first `Lazy::ghost(User::class)($initializer)`, `proxy` and `reset` are `#[\Tyhp\InferGeneric]`, `isLazy`, `initialize`, `preset`; native on the 8.4/8.5
>     builds, eager on 8.2/8.3; `ghost` does not bind the class's type arguments); `\Tyhp\Generic::bind($className, T::int)->ghost(...)`
>     (binds type arguments, matches `new` when the object cannot stay lazy); disposal skips an
>     uninitialized lazy object; agents write `lazy new` or `Generic::bind($className, T::int)->ghost(...)`, not
>     `ReflectionClass::newLazyGhost`. **Derive strictly from the current code.**
>
> **Output format (the split):**
> - One file per section: `guide/NN-slug.md` (e.g. `guide/07-generics.md`) and `handbook/NN-slug.md`.
>   Keep the `## N. Title` heading at the top of each file.
> - Rewrite every cross-reference as a link to the **sibling section file**, not a `§`-only pointer or
>   a line number (e.g. `[§16](16-type-guards.md)`; from handbook to guide use `../guide/NN-slug.md`).
>   Never hardcode line numbers.
> - `00-index.md` in each folder: the shared intro/legend + a list of the section files.
> - `QUICK_GUIDE.md` / `SKILL.md`: one line per feature, cross-language analogy, `→` naming the exact file(s).
>   Under "Declarations", include an **Initialized storage** line: `unset` of a local, a typed instance
>   property, or a static property is TYHP4158 (`?T = null` drops the value); a typed static needs an
>   initializer (TYHP4418); every typed instance storage property is initialized after construction
>   (TYHP4419); a class with no constructor runs the nearest ancestor constructor; array-offset
>   `unset` and an optional struct field stay legal; `→ guide/22-declarations.md`,
>   `guide/08-typed-locals.md`, `guide/28-availability-gotchas.md`.
>   The operator-overloading line includes `operator default(): self`. The compile-time helpers line
>   says a class or enum that declares it makes `default(ThatType)` that instance, and one that does
>   not makes null.
>   Frame it as "PHP plus additions" — no compile-target/erasure detail there. Public tagline is
>   **typed superset of the PHP language** (not “strongly typed”). Locals lead with `$x = …`.
> - `AGENTS.md` (+ `CLAUDE.md` pointing to it): entry point; how to load section-by-section, and the
>   handful of PHP habit-breakers.
>
> **Style (token-efficient but complete):**
> - Dense; no filler, hedging, or meta commentary. Prefer tables over prose.
> - Define shorthands up front: `→` = "compiles to"; `⚠️` = doesn't compile cleanly yet, use the PHP
>   form.
> - Use fenced code blocks tagged `tyhp`, `php`, or `json`. Keep examples minimal but real (draw from
>   fixtures/tests).
> - Do **not** reference compiler internals (story numbers, phase names, C# file names) in the guide
>   itself — it's for app authors, not compiler devs.
> - Do **not** apply automated token-pruners (LLMLingua etc.) — they corrupt the verbatim code. Keep
>   density gains lossless (tables over prose, minimal examples, no cross-section repetition).
>
> **Acceptance checks before finishing:**
> - All 31 guide + 8 handbook section files exist, are lint-clean, and each `00-index.md` lists them.
> - Cross-references resolve to real sibling files; no `§`-only or line-number references remain.
> - `QUICK_GUIDE.md` pointers name files that exist; `AGENTS.md`/`CLAUDE.md` present.
> - Every ⚠️/"use the PHP form" claim matches the current emitter/checker behavior.
> - No compiler-internal jargon leaked into the guide/handbook prose.
> - Compiled runtime packages live under `../tyhp-runtime-src/packages/tyhp/`; tyhpdef packages (`php/_tyhpdef`, `php-ext-*`, Composer-lib wrappers) under `../tyhp-runtime-src/packages/tyhpdef/`; `test-all-tyhpdef.sh` is in `../tyhp-runtime-src/packages/`.
> - Measure total token count with `tiktoken` (`o200k_base`) and note it in the summary.
>
> **Handbook content** (same audience, on-demand load), split across `handbook/` section files, covering:
> project setup + `tyhp.json` layout keys; namespace→folder output mapping (from FQN; `psr4` is
> composer-only); `.tyhp`/`.php`/`.tyhpdef` handling; autoloading (`build.updateComposer`,
> `entryPointAutoloader`, `declare(output_file=…)`); the build/CLI dev loop with **honest** command
> status (mark stubs ⚠️: `watch`, `lint --fix`; `init`, `composer`, `generate_tyhpdef`, source maps,
> `lint --format json`, `language_server`, `xdebug_proxy`, `overlay`, `symbol_tree` **are implemented** — document them);
> testing (no `tyhp test` — test emitted PHP; tyhpdef package tests are `../tyhp-runtime-src/packages/test-all-tyhpdef.sh`; compiler emit-and-run at `../tyhp/tests/conformance/emit-and-run/`
> executes PHP with `error_reporting=-1` / `display_errors=1` and is not golden-PHP dumps under `storyNN/`)
> and debugging; a concrete PHP-interop walkthrough
> (`.tyhpdef` for an untyped library; emitted PHPStan/Psalm docblocks on lossy types —
> `emit.docblocks` default `lossy-only`, `emit.docblockDialect` parsed but not changing output,
> author tags win, `@throws` not generated, FQNs, `class-string` only for class/interface/enum
> names, struct `array{key?: ?type}`, callable `callable(int): string` with union parentheses;
> `tyhpdef/php` + `tyhpdef/php-ext-*` and overlays;
> library authors fill `extra.tyhp.require`, never put author-only tyhpdefs in extras;
> first `composer require tyhp/core` miss → `composer update` / `tyhp composer sync`);
> fuller worked examples (extension, operator overload including `operator default(): self`,
> `using`/`:=`, `with`, guards, async+cancellation); and a runtime API signature reference (Promise,
> Deferred, EventLoop, CancellationToken(Source), Contracts, DisposableScope, Decimal, Type,
> Exceptions, Expression). `Type::defaultValue()` on a class, enum, or generic class calls that type's own public static `__tyhpDefault()` when that exact type declares it and returns null when the method is only inherited or absent; hand-written PHP may declare the method. `handbook/08-rhetoric.md` is the `tyhp/rhetoric` package: the Tyhp material (models, contexts, queries, save, mapping, migrations, scaffold, Eloquent queries, `Sql::fn`) and a distinct PHP section a PHP developer can open without the Tyhp language pages. PHP samples call binder-first methods as `name(types)(args)` (`table(Post::class)()`, `model(Post::class)()`, `belongsTo(Blog::class)($path)`, `transaction(T::int)($fn)`, `ContextFactory::new(Context::class)()`, `EloquentQuery::query(Model::class)()`), call `entry($model)` and `\Tyhp\Expression::of` / `\_expr` as ordinary methods, and build trees with `\Tyhp\Expression::of` / `\Tyhp\Expr`. `\Tyhp\Generic::bind($target, T::int)` and `\_gnrc($target, T::int)` are for a class or callable chosen at runtime. Keep that file when regenerating; do not fold it into `07-runtime-api.md`. Verify every tooling claim against `../tyhp/Tyhp/CLI/**`, `../tyhp/Tyhp/Config/**`, the
> emitter, and `../tyhp-runtime-src/packages/**` — **prefer each package's `.tyhpdef` for public signatures**, and
> mark unimplemented behavior ⚠️ rather than describing the story's intent as fact.

---

## After regenerating

- Skim `guide/26-build-cli.md`, `guide/27-diagnostics.md`, and `guide/28-availability-gotchas.md` to
  confirm build/diagnostics/availability reflect the current toolchain.
- Update the "still maturing / not yet in the language" items if features have since landed.
- Projects that symlink this repository as `.cursor/skills/tyhp` pick up the updated files from the clone (see `README.md`).

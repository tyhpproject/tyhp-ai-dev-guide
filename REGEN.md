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
guide/     00-index.md + 01-…-30-*.md   (the dense language delta, one file per section)
handbook/  00-index.md + 01-…-07-*.md   (setup/CLI/interop/testing/examples/runtime API)
```

The content is authored as two logical documents (the **guide**, ~30 sections, and the **handbook**,
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
> (~29 sections) and the handbook (~7 sections), but **write one file per section** into
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
> 2. Files & open tags (`.tyhp`/`.tyhpdef`/`.php`, `<?tyhp `/`<?tyhpdef `, tagless mode).
> 3. The two strict rules: non-nullable by default; conditions must be `bool`; narrowing resets on
>    reassignment.
> 4. Type system: known types now enforced (`static` return-only; `resource` not user-writable); new
>    types (`decimal`→`\Tyhp\Decimal`, `struct`→`array`, literal types); unions/intersections/`?T`;
>    `mixed` collapse; `(decimal)` cast status; `(object)` is identity on an already-object operand
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
>    compiled libraries stamp `#[\Tyhp\GenericRuntime]` on every emitted generic (`erased` + `layouts: [1]`; `factory`/`binder` only when helpers exist; foreign sites use `\Tyhp\Generic::bind`; overlays that omit the stamp keep erase/inline; `#[\Tyhp\EraseGeneric]` opts properties out of tracking),
>    `array<T>`/`array<K,V>`, `iterable<…>`, callable shapes `callable(…): R`
>    (`callable(): bool` is 0-arg; `callable(...): TReturn` is an `extends`-only any-arity bound;
>    packs auto-splice inside a callable-shape parameter list; postfix `T...` is a trailing PHP variadic),
>    `\Closure<callable(...): mixed, TThis, TScope>`, `\Fiber<TResume, TCallableShape>`,
>    `\Generator` declared-return shapes, `ArrayAccess<K,V>` / `ArrayAccessShape`, Traversable
>    implements rule, alias return-constraint propagation.
> 8. Typed locals (erased; only params/props/returns keep spelled types). Lead examples with inferred
>    `$x = …`; `int $x = …` when inference is not enough.
> 9. Type aliases (hints expand to the underlying PHP type; source `.tyhp` aliases emit a
>    `\Tyhp\Type` factory — `UserId()`, `Optional(typeof(int))` / `typeof(Optional<int>)`;
>    `Optional<int>()` is invalid; `use App\Types\UserId;` stays class-kind, emitted PHP is
>    `use function` when the factory is used; class-level `self::NameType()` /
>    `UserService::NameType()`; tyhpdef aliases inline unless `#[\Tyhp\GenericRuntime(aliasFactory: …)]`).
> 10. Structs (structural value type, properties-only, `extends`; struct vs class table incl. width
>     subtyping; `new`/`with`/property-access lowering to arrays; PHP sees `array`; no runtime shape
>     check; anon/`clone` gotchas).
> 11. Operator overloading (class + shorthand + `convert`; arity/`self` rules; overloadable set;
>     left-first resolution; synthetic-method lowering).
> 12. Extensions (block target `extension Name extends Type` or nested `extends Type { }` / `extends<T> Type { }`; `$this` implied, `&$this` only when by-ref; `self` is the block target including scalars; unions legal when every receiver alternative is a union member; header generics stay on the extension symbol; three member forms: `=>` omits the
>     PHP method, single-`return` brace splices and emits, multi-statement is a real method Tyhp
>     calls; written params must be `&`; static-class lowering; `use extension`; `tyhp/core` scalars
>     via `global use extension \Tyhp\StringExtensions` (no per-file import; local `use` only to
>     adapt; `TYHP4169` if redundant); tyhpdef class-body members are thin `=>` mappings that splice
>     unless `#[\Tyhp\GenericRuntime]` is present (foreign sites use `\Tyhp\Generic::bind`); compiled libraries copy in-package
>     `global use extension` into `package.tyhpdef`).
> 13. async/await (Fiber/Promise; `\Tyhp\Promise::_async/_await`; outer return `\Tyhp\Promise`; no
>     async generators; Promise API surface; **cancellation** via `CancellationToken(Source)` +
>     `OperationCancelledException`/`TimeoutException`, unhandled-rejection behavior; `foreach (await
>     …)` gotcha; blocking PHP I/O blocks the `tyhp/async` loop).
> 14. Deterministic disposal `using`/`:=` (`IsDisposable`/`AsyncIsDisposable`; try/finally and
>     `DisposableScope` lowering; **failure semantics**: reverse order, dispose-throw masks the body
>     exception, multi-resource → `AggregateException` of disposal errors only, `:=` `__destruct` only
>     warns).
> 15. `with` expressions (clone/new/in-place forms; readonly rule; structs **and** classes compile;
>     object `with` mutates identity, struct `with` rebinds an array — PHP object-vs-array argument
>     analogy; PHP < 8.5 readonly `clone … with` needs `build.experimentalReadonlyCloneWith`).
> 16. Type guards + narrowing + `is` (guard `: $x is T`→`: bool`; `is` emit is tyhpdef-driven:
>     `#[\Tyhp\NativeTypeTest]` on a free function or concrete static method with `$param is T`
>     → `\Fqn($x)` / `\Class::method($x)`; unmarked
>     class/interface/enum/trait → `instanceof`; else `\Tyhp\Type::is`. `$x is null` allowed.
>     Narrowing table; what does not narrow). Closure/Fiber generic shapes: `as`,
>     not `is`.
> 17. Compile-time helpers (`nameof`, `default(T)`, `typeof` takes a **type** — `typeof(int)`,
>     `typeof(Optional<int>)`, source alias → factory; value path is `\Tyhp\Type::of($value)`;
>     `variable_exists` → `\array_key_exists`).
> 18. String-as-type features (symbol-name types list + existence-check narrowing; template string
>     types + quantifiers; all erase to `string`; include `__SuperTypeName`).
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
>     inline-`fn` only; `TCallableShape`; `->callable`; `tyhp/lambda`; experimental).
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
>     declaration, not functional checks; everything else identical to PHP — say so
>     explicitly).
> 23. `.tyhpdef` declaration files (what's declarable, including hooked class/interface/trait
>     properties as bodyless `{ get; set; }` / `{ &get; }`; class-body `type`; `deprecated`/`obsolete`; `extends` import;
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
>     (`PhpType`, `GenericRuntime` runtime-visible, `EraseGeneric` compile-only, `Generic::bind`, `ArrayAccessShape`, scalar methods) / `decimal` / `async` / `lambda`, key symbols).
>     `php-ext-decimal` is PECL Decimal, not `tyhp/decimal`. json/hash/libxml stay in `tyhpdef/php` only.
>     Package tests run `../tyhp-runtime-src/packages/test-all-tyhpdef.sh`.
> 26. Build, output layout & CLI: `tyhp.json` keys; **output layout** (classes → FQN segments under
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
>     100/file), common-codes table (include `TYHP4121` empty `catch` → name `$e` and `(void)$e`;
>     a comment-only body still warns; `TYHP4500` deprecated use; `TYHP4129` / `TYHP4339` Override;
>     `TYHP4165` NoDiscard unused return); note there is no dedicated unknown-member / arg-count error.
> 28. Availability & gotchas: "use freely" list (property hooks polyfill on 8.2–8.3, native on 8.4+;
>     `&get` needs 8.4; **`tyhp/core` scalar catalog is use-freely** — `$s->length()` with no import);
>     "annotate don't assume" (arrays/params/returns); "use the PHP form instead"
>     table; "not in the language yet" list (`internal` **is** in the language — omit from published
>     `package.tyhpdef`). **Derive strictly from the current code.**
> 29. PHP-mapping reference table (source `type X = …` → factory `function X(): \Tyhp\Type`;
>     hints still expand; tyhpdef aliases inline unless `#[\Tyhp\GenericRuntime(aliasFactory: …)]`).
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
>
> **Output format (the split):**
> - One file per section: `guide/NN-slug.md` (e.g. `guide/07-generics.md`) and `handbook/NN-slug.md`.
>   Keep the `## N. Title` heading at the top of each file.
> - Rewrite every cross-reference as a link to the **sibling section file**, not a `§`-only pointer or
>   a line number (e.g. `[§16](16-type-guards.md)`; from handbook to guide use `../guide/NN-slug.md`).
>   Never hardcode line numbers.
> - `00-index.md` in each folder: the shared intro/legend + a list of the section files.
> - `QUICK_GUIDE.md` / `SKILL.md`: one line per feature, cross-language analogy, `→` naming the exact file(s).
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
> - All 30 guide + 7 handbook section files exist, are lint-clean, and each `00-index.md` lists them.
> - Cross-references resolve to real sibling files; no `§`-only or line-number references remain.
> - `QUICK_GUIDE.md` pointers name files that exist; `AGENTS.md`/`CLAUDE.md` present.
> - Every ⚠️/"use the PHP form" claim matches the current emitter/checker behavior.
> - No compiler-internal jargon leaked into the guide/handbook prose.
> - Package trees, `test-all-tyhpdef.sh`, and `php/_tyhpdef` live under `../tyhp-runtime-src/packages/`.
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
> (`.tyhpdef` for an untyped library; `tyhpdef/php` + `tyhpdef/php-ext-*` and overlays;
> library authors fill `extra.tyhp.require`, never put author-only tyhpdefs in extras;
> first `composer require tyhp/core` miss → `composer update` / `tyhp composer sync`);
> fuller worked examples (extension, operator overload,
> `using`/`:=`, `with`, guards, async+cancellation); and a runtime API signature reference (Promise,
> Deferred, EventLoop, CancellationToken(Source), Contracts, DisposableScope, Decimal, Type,
> Exceptions, Expression). Verify every tooling claim against `../tyhp/Tyhp/CLI/**`, `../tyhp/Tyhp/Config/**`, the
> emitter, and `../tyhp-runtime-src/packages/**` — **prefer each package's `.tyhpdef` for public signatures**, and
> mark unimplemented behavior ⚠️ rather than describing the story's intent as fact.

---

## After regenerating

- Skim `guide/26-build-cli.md`, `guide/27-diagnostics.md`, and `guide/28-availability-gotchas.md` to
  confirm build/diagnostics/availability reflect the current toolchain.
- Update the "still maturing / not yet in the language" items if features have since landed.
- Projects that symlink this repository as `.cursor/skills/tyhp` pick up the updated files from the clone (see `README.md`).

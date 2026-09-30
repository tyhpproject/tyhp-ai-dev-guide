---
name: tyhp
description: >-
  Writes and edits Tyhp (a typed superset of the PHP language) from the AIDevGuide index.
  Use when creating, changing, reviewing, or answering questions about .tyhp /
  .tyhpdef files, Tyhp syntax, types, generics, structs, tyhpdefs, PHP interop,
  PHP version gates, or the tyhp CLI. Open only the referenced guide/ or
  handbook/ section for the feature being written — do not guess Tyhp syntax
  from PHP or from the one-line analogies.
---

# Tyhp (superset of PHP)

**Tyhp is a typed superset of the PHP language.** You already know PHP — below are
only the deltas, with analogies to languages you know.

**How to use this skill:** this file is the index. Each entry ends with `→` pointing to the file
that explains it in full. **Before writing a feature marked `→`, open that file** — don't reconstruct
the syntax from the analogy alone. Open **only** the files the current task needs. For a full listing
of what's available, read `guide/00-index.md` and `handbook/00-index.md` (or list the `guide/` and
`handbook/` folders). `⚠️` = not supported yet; use the alternative noted.

## Core rules
- **Static typing** — like TypeScript over JS: every parameter, property, and return type is required. `→ guide/01-mental-model.md, guide/05-type-inference.md`
- **Files/tags** — `.tyhp` + `<?tyhp ` (needs trailing space); `.tyhpdef` = declarations only. `→ guide/02-files-and-tags.md`
- **Typed / inferred locals** — `$s = "hi";` is the primary form (like C# `var` / TS `let`); `string $s;` / `int $x = …` when inference is not enough. `→ guide/05-type-inference.md, guide/08-typed-locals.md`
- **Non-nullable by default** — like C#/TS strict null: write `?string` / `|null` to allow `null`. `→ guide/03-strict-rules.md`
- **Conditions must be `bool`** — no truthiness; write `if ($x !== null)` not `if ($x)`. `→ guide/03-strict-rules.md`
- **Empty `catch`** — `TYHP4121`; swallow on purpose with `(void)$e` (a comment is not enough). `→ guide/27-diagnostics.md`
- **Inference limits** — literals infer, but annotate arrays (`array<int> $xs = [...]`); an untyped call result is `mixed`. `→ guide/05-type-inference.md`

## Types
- **`decimal`** — arbitrary-precision decimal, like C# `decimal` / Java `BigDecimal`. Write values with `\Tyhp\decimal('19.99')` (⚠️ avoid the `(decimal)` cast). `→ guide/04-type-system.md`
- **Literal types** — `'GET'|'POST'`, `42` used as types, like TS literal/union types. `→ guide/04-type-system.md`
- **Generics** — `class Box<T extends X = Def>`, like C#/TS generics. Pass all type args explicitly. `→ guide/07-generics.md`
- **Generic collection types** — `array<K,V>`, `iterable<T>`, `callable(A, B): R`; `callable(...): R` as an `extends` bound. `\Closure<callable(A, B): R>` (class). `→ guide/07-generics.md`
- **Type aliases** — `type Id = int;` like TS `type`; `Id()` / `typeof(Id)` is a `\Tyhp\Type` factory. `→ guide/09-type-aliases.md`
- **Structs** — value type with typed properties only; **structural** compatibility like TS/Go (vs nominal classes). `→ guide/10-structs.md`
- **Utility types** — `__Partial<T>` `__Pick<T,K>` `__Record<K,V>` `__Awaited<T>` `__Nullable<T>` `__NonNullable<T>` `__AsReadOnly<T>` …, plus callable-keyed `__CallableReturnType<C>` / `__CallableParametersStruct<C>` / `__CallableParametersTuple<C>` / `__CallableParametersRest<C>` / `__CallableParametersSlice<C, Start, Min>`. `__CallableThis` / `__SuperType` / `__IndexKeys`. `→ guide/19-utility-types.md`
- **String-as-type** — symbol-name types (`__ClassName`) and template string types (`"api/${string}"`), like TS template-literal types. `→ guide/18-string-as-type.md`

## Behavior / expressions
- **async / await** — like C#/JS; cancel via `CancellationToken`. `→ guide/13-async-await.md, handbook/07-runtime-api.md`
- **using / disposables / `:=`** — like C# `using` + `IDisposable`; `$x := new R()` = scope-disposed local (C# `using` declaration / Go `defer`-ish). `→ guide/14-disposal.md`
- **`with` expressions** — object `with` mutates identity; struct `with` rebinds an array (same as passing an object vs an array into a PHP function). `new Point() with [x => 1]` / `clone $obj with […]`. (Readonly `clone … with` on PHP &lt; 8.5 needs a config flag.) `→ guide/15-with-expressions.md`
- **Operator overloading** — like C# `operator +`, declared in the class/extension body (incl. `convert`). `→ guide/11-operator-overloading.md`
- **Extension methods** — like C#/Kotlin: `extension E extends Money { function f(...); }` (`$this` implied); activate with `use extension`. `tyhp/core` scalars need no import. `→ guide/12-extensions.md`
- **Type guards + narrowing** — like TS: `function isX(mixed $v): $v is X`, narrows in `if`. `instanceof` / `is` are equivalent. `→ guide/16-type-guards.md`
- **Compile-time helpers** — `nameof()`, `typeof()`, `default(T)`, `variable_exists()` like C#. `→ guide/17-compile-time-helpers.md`
- **Expression trees** — inline `fn` captured as an AST for an `Expression<callable(T): R>` param, like C# `Expression<Func<>>` / LINQ. Experimental. `→ guide/21-expression-trees.md`
- **Property accessors/hooks** — PHP 8.4 hook syntax in `.tyhp` (bodies); `.tyhpdef` uses bodyless `{ get; set; }` / `{ &get; }`. `→ guide/20-property-accessors.md, guide/23-tyhpdef.md`

## Declarations (deltas vs PHP)
- **Constructor return type / chaining** — constructors declare `: void`, or `: parent(<args>)` to call the base ctor (like C#/Java `: base(...)`). `→ guide/22-declarations.md`
- **Method overload signatures** — bodiless signature declarations, like TS/C# overloads. `→ guide/22-declarations.md`
- **Trait property alias** — Tyhp adds `use T { $prop as $renamed; }` (PHP only aliases methods). `→ guide/22-declarations.md`
- **`internal` modifier** — omit from published `package.tyhpdef` (like C#/TS `internal` at the package boundary). `→ guide/22-declarations.md`
- **PHP version gates** — `declare(php=">=8.4")` / `#[\Tyhp\Php(">=8.4")]` (`output.phpVersion` is the minimum PHP; gates true for only some versions emit `\PHP_VERSION_ID` checks). `declare(ext="name")` / `fallback function` / `fallback const` (also in `.tyhp`). `→ guide/30-php-version-gating.md, guide/23-tyhpdef.md`
- **`#[\Tyhp\PhpType]`** — emit a PHP type hint that differs from the checker type. `→ guide/22-declarations.md`

## External code / tooling
- **`.tyhpdef`** — declaration stubs for untyped PHP, exactly like TS `.d.ts`. Generate from PHP with `tyhp generate_tyhpdef` (`--audit-stubs` reports Layer 2 vs stub corpora). Overlays: last-wins `overlay` array, overlay `partial class Foo<T>;` header / brace `partial` / `partial function` / `as`, `omit`, `@overlay-against`. Optional-peer placeholders: `extern \Name;` or `extern class`/`interface`/`enum`/`function`/`const` (specified type kind must match the originating declaration; mismatch is `TYHP8029`). `extra.tyhp.require` = ambient for consumers. `→ guide/23-tyhpdef.md, handbook/05-php-interop.md, handbook/03-build-cli-workflow.md, handbook/01-project-setup.md`
- **Use existing PHP libraries** — describe them in a `.tyhpdef`, then call as usual. `tyhpdef/php` + `tyhpdef/php-ext-*`. `→ guide/24-php-interop.md, handbook/05-php-interop.md`
- **Runtime helpers** — `\Tyhp\` classes back some features (`decimal`, `Promise`, `CancellationToken`, …). `→ guide/25-runtime-packages.md, handbook/07-runtime-api.md`
- **Tooling** — `tyhp lint` to type-check, `tyhp build` to compile, `tyhp composer sync` / `tyhp build --fix` for extras, `tyhp overlay create` / `stamp`, `tyhp symbol_tree` to dump bound names, `tyhp explain TYHP####` for a diagnostic. `→ guide/26-build-cli.md, handbook/03-build-cli-workflow.md`

## Other topics (open only if the task needs them)

Language (`guide/`):
- Assignability / subtyping — `guide/06-assignability.md`
- Diagnostics (TYHPxxxx) — `guide/27-diagnostics.md`
- What does not compile yet — `guide/28-availability-gotchas.md`
- How constructs lower to PHP — `guide/29-php-mapping.md`
- PHP version gates — `guide/30-php-version-gating.md`

Project / toolchain (`handbook/`):
- New project / composer layout — `handbook/01-project-setup.md`
- Autoloading — `handbook/02-autoloading.md`
- Build / CLI workflow — `handbook/03-build-cli-workflow.md`
- Testing and debugging — `handbook/04-testing-debugging.md`
- PHP interop (full) — `handbook/05-php-interop.md`
- Worked examples — `handbook/06-examples.md`
- Runtime API signatures — `handbook/07-runtime-api.md`

> Everything not listed behaves like plain PHP 8.x. When in doubt, open the referenced file; use the
> `handbook/` files for setup, interop, and runtime API.

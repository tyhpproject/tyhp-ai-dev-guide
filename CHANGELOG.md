# Changelog

Notable changes to the Tyhp AI development guide (`SKILL.md`, `QUICK_GUIDE.md`, `guide/`, `handbook/`). The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The guide is not versioned; releases are dated by push to `main`.

## [Unreleased]

### Added

- `guide/31-lazy-objects.md`: `lazy new` and `\Tyhp\Lazy`. Agents write `lazy new` or `Generic::bind(...)->ghost(...)`. `Lazy::ghost` does not bind type arguments, and a hand-written `ReflectionClass::newLazyGhost` skips that binding.
- `handbook/08-rhetoric.md`: `tyhp/rhetoric` for Tyhp callers and a separate PHP section. `handbook/00-index.md`, `QUICK_GUIDE.md`, `guide/25-runtime-packages.md`, and `REGEN.md` point at it.

### Changed

- `guide/03-strict-rules.md`, `guide/08-typed-locals.md`, `guide/10-structs.md`, `guide/16-type-guards.md`, `guide/22-declarations.md`, `guide/23-tyhpdef.md`, `guide/27-diagnostics.md`, `guide/28-availability-gotchas.md`, `QUICK_GUIDE.md`, `SKILL.md`, and `REGEN.md`: agents assign `null` on a `?T` instead of calling `unset` on a local or property (TYHP4158). A typed static property needs an initializer (TYHP4418). After construction every typed instance storage property is initialized (TYHP4419). A class that declares no constructor runs the nearest ancestor constructor. Array-offset `unset` and an optional struct field stay legal.
- `guide/10-structs.md` and `QUICK_GUIDE.md`: agents construct a struct with `new Name[field => value]` (empty `new Name[]`, `new Box<int>[value => 1]` / `new Box[value => 1]`, `new struct { … }[…]`, spread `new Opts[...$base, name => 'bob']`). `new Name()` and `new Name() with […]` are the same forms.
- `guide/21-expression-trees.md`, `guide/12-extensions.md`, `guide/25-runtime-packages.md`, `guide/07-generics.md`, `guide/29-php-mapping.md`, `handbook/05-php-interop.md`, `handbook/07-runtime-api.md`, and `REGEN.md`: agents treat `ext-tyhp` as optional and recommended (`pie install tyhp/ext-tyhp`), keep fluent expression trees in PHP packages and pass a closure only when the process is known to have the module loaded, and expect DNS to use the `tyhp` module when it is loaded and `Offload` otherwise.
- `handbook/07-runtime-api.md` and `handbook/08-rhetoric.md`: `_gnrc` type arguments are `TypeLike|string|null`, the same as `Generic::bind`, including `T::int`.
- `handbook/08-rhetoric.md`: PHP calls `table(Post::class)()`, `model(Post::class)()`, `belongsTo(Blog::class)($path)`, `transaction(T::int)($fn)`, `ContextFactory::new(Context::class)()`, and `EloquentQuery::query(Model::class)()`. `entry($model)` and `Expression::of` are ordinary calls. `QUICK_GUIDE.md` and `guide/25-runtime-packages.md` name those shapes.
- `guide/07-generics.md`, `guide/29-php-mapping.md`, `guide/25-runtime-packages.md`, `handbook/05-php-interop.md`, `handbook/07-runtime-api.md`, `QUICK_GUIDE.md`, and `SKILL.md`: agents write `T::int`, `Box::new(T::int)(42)`, `Box::type(T::int)`, binder-first `name(types)(args)` (`Promise::reject()($e)`, `Promise::withResolvers(T::int)()`), and plain `#[\Tyhp\InferGeneric]` calls (`await(async(...))`, `_expr`). `new()` matches the constructor's visibility and is not `final`. TYHP4404–TYHP4408.
- `guide/12-extensions.md`, `guide/23-tyhpdef.md`, `guide/26-build-cli.md`, and `handbook/03-build-cli-workflow.md`: a foreign layout-2 call emits `name(types)(args)` (`jsonDecodeAs($type)($s)`); only a foreign class `new` emits `Generic::bind`. `handbook/07-runtime-api.md`: `Promise::wait(?int $timeoutMs = null, ?CancellationToken $token = null)`.
- `guide/30-php-version-gating.md`: a class or trait method whose `#[\Tyhp\Php]` gate holds for only some versions at or above the floor is emitted inside `\PHP_VERSION_ID`, and every such sibling stays in `package.tyhpdef`. Functions and types still compile only the declaration that matches the floor.
- `QUICK_GUIDE.md`, `SKILL.md`, `guide/00-index.md`, `guide/14-disposal.md`, `guide/25-runtime-packages.md`, `guide/27-diagnostics.md`, `guide/29-php-mapping.md`, and `handbook/07-runtime-api.md`: the index line, the disposal skip, TYHP4402 and TYHP4403, and the `Lazy` / `Generic::ghost` signatures.
- `guide/17-compile-time-helpers.md` and `guide/18-string-as-type.md`: `nameof(User::$name)` is `__PropertyName<User>` and a constant attribute argument; a class reference is `User::class` or `T::class` (`__ClassName<User>` / `__ClassName<T>`).
- `guide/21-expression-trees.md`: an extension method call stays a method call in the tree (`$title->startsWith($prefix)`); the extension body is not substituted into the tree.

## 2026-11-04

### Changed

- `QUICK_GUIDE.md`, `SKILL.md`, `guide/02-files-and-tags.md`, `guide/26-build-cli.md`, `guide/28-availability-gotchas.md`, `guide/29-php-mapping.md`, `handbook/01-project-setup.md`, and `handbook/05-php-interop.md`: agents write `<?=` in a `.tyhp` file as a checked echo (emitted as `echo`), and expect PHPStan/Psalm tags where the emitted PHP hint drops a type (`emit.docblocks` default `lossy-only`; `emit.docblockDialect` does not change the tags).
- `guide/02-files-and-tags.md`: in tagless mode a `.tyhp` file whose first 6 characters are `<?tyhp` is a tagged template (empty first section allowed, then HTML, `<?=`, and a later `<?tyhp`). The tagless entry has no open tag, so `<?tyhp` anywhere else in the file is not accepted.
- `QUICK_GUIDE.md`, `guide/15-with-expressions.md`, `guide/27-diagnostics.md`, `guide/28-availability-gotchas.md`, and `handbook/01-project-setup.md`: readonly `new … with` and `clone … with` below PHP 8.3 are TYHP4400; `clone … with` on a `readonly class` below 8.5 is TYHP4401. The experimental flag applies on 8.3–8.4 for a class that is not readonly.
- `guide/03-strict-rules.md`: a possibly null receiver must be narrowed before `->` access or indexing (TYHP4198), and a `?->` chain's value is nullable.
- `guide/16-type-guards.md`: `\assert()`, loose `!= null`, `if` arms that exit, `switch (true)` / `match (true)` arms, and nested or static property paths narrow; method-call results do not.
- `guide/18-string-as-type.md`: template holes can name aliases so templates compose, holes without a string form are TYHP4199, numeric holes take PHP's `(string)` text, and a matching interpolation is assignable to a template type. A nullable hole value fits only a hole that allows empty text.
- `guide/21-expression-trees.md`: tree bodies may use nested arrow `fn`, array literals, and `?->` method calls; a class in a variable, a variable method name, and named or unpacked arguments are TYHP4322; `R` is inferred for `PropertyPath` too; nodes carry full declared types; `shapeKey()` supports per-shape caching.
- `guide/07-generics.md` and `guide/25-runtime-packages.md`: the compiler stamps `#[\Tyhp\PropertyType]` on lossy property types, read with `\Tyhp\Type::fromProperty()`.
- `guide/27-diagnostics.md`: TYHP4198 in the common-codes table.
- `guide/28-availability-gotchas.md`: readonly property defaults below PHP 8.6 (TYHP4378) go in the constructor or a promoted parameter; enum `__debugInfo()` below 8.6 (TYHP4379) and the other magic methods PHP rejects on enums (TYHP4024) become regular methods.
- `handbook/07-runtime-api.md`: `Type::parse()`, `Type::fromProperty()`, and the new `Type` accessors; the new `tyhp/lambda` nodes, `Expr::closure()` / `Expr::arrayOf()`, and `ExpressionShape`; `EventLoop::run()` accepts `Promise<void>`.

## Baseline - 2026-10-22

Tracking starts at commit `6a8aba8`. Earlier changes are not listed.

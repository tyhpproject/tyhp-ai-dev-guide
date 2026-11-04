# Changelog

Notable changes to the Tyhp AI development guide (`SKILL.md`, `QUICK_GUIDE.md`, `guide/`, `handbook/`). The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The guide is not versioned; releases are dated by push to `main`.

## [Unreleased]

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

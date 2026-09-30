## 25. Runtime packages (`\Tyhp\`, PHP ≥ 8.2; compiler auto-adds the dependency)

| Package | Needed for | Key symbols |
|---------|-----------|-------------|
| `tyhpdef/php` | Always-present PHP 8.2–8.5 builtins | `\strlen`, `DateTime`, SPL, json/hash/libxml |
| `tyhpdef/php-ext-*` | Optional / PECL extensions (curl, PDO, mbstring, …) | That extension’s functions/classes. `php-ext-decimal` = PECL Decimal, **not** `tyhp/decimal` |
| `tyhp/core` | generics runtime, `typeof`, `:=`/disposal, object `with`, `PhpType`, `NativeTypeTest`, `GenericRuntime`, `EraseGeneric`, `ArrayAccessShape`, scalar methods | `\Tyhp\Type`, `\Tyhp\PhpType`, `\Tyhp\NativeTypeTest` (compile-only `NoEmit`), `\Tyhp\EraseGeneric` (compile-only `NoEmit`), `\Tyhp\GenericRuntime` (runtime-visible), `\Tyhp\Generic::bind`, `Contracts\IsDisposable`, `Contracts\ArrayAccessShape`, `DisposableScope`, `ObjectHelper`; `$s->length()` via `global use extension \Tyhp\StringExtensions` |
| `tyhp/decimal` | `decimal` | `\Tyhp\Decimal`, `\Tyhp\decimal()` |
| `tyhp/async` | `async`/`await` | `\Tyhp\Promise`, `PromiseSettledResult`, `EventLoop`, `CancellationToken(Source)`, `Contracts\AsyncIterator` |
| `tyhp/lambda` | `Expression<>`/`PropertyPath<>` | `\Tyhp\Expression`, `ExpressionVisitor` |

One `tyhpdef/php` (and one `tyhpdef/php-ext-*` per extension) covers 8.2–8.5. Composer `"php": ">=8.2"`. No `tyhpdef/php-8.2` forks. Gates: `declare(php=…)` / `#[\Tyhp\Php]` vs `output.phpVersion`. `declare(ext="name")` is satisfied when a loaded tyhpdef package requires `ext-name`. json/hash/libxml stay in `tyhpdef/php` only.

Libraries declare ambient tyhpdefs in `extra.tyhp.require` (duplicate in `require-dev`; any Composer
name). Author-only wrappers stay require-dev — `tyhp/decimal` extras are `tyhpdef/php` only.
`tyhp/core` extras are `tyhpdef/php` only (`tyhpdef/php-ext-mbstring` stays author-only in core `require-dev`). First `composer require tyhp/core`
often misses that transaction: `composer update` or `tyhp composer sync`
([handbook §1](../handbook/01-project-setup.md)).

**Package tests:** [tyhp-runtime-src](https://github.com/tyhpproject/tyhp-runtime-src) `packages/test-all-tyhpdef.sh` lints each `tyhpdef/*` `tests/` tree, one job per package/version (`--jobs`, default CPU count). `--package` limits the run to one path; the default is every package. A package that runs longer than 90 seconds (`--timeout`) fails. Positives must be clean. Expected errors are `tests/fail/*.tyhp` plus sidecar `tests/fail/<stem>.expect.json`. Skips `core` / `async` / `lambda` / `decimal` / `compiler`.

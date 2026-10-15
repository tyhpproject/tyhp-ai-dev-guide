## 25. Runtime packages (`\Tyhp\`, PHP ≥ 8.2; `tyhp build` reports the packages the code needs)

| Package | Needed for | Key symbols |
|---------|-----------|-------------|
| `tyhpdef/php` | Always-present PHP 8.2–8.5 builtins | `\strlen`, `DateTime`, SPL, json/hash/libxml |
| `tyhpdef/php-ext-*` | Optional / PECL extensions (curl, PDO, mbstring, …) | That extension’s functions/classes. `php-ext-decimal` = PECL Decimal, **not** `tyhp/decimal` |
| `tyhp/core` | generics runtime, `typeof`, `:=`/disposal, object `with`, `PhpType`, `NativeTypeTest`, `GenericRuntime`, `EraseGeneric`, `ArrayAccessShape`, scalar methods | `\Tyhp\Type`, `\Tyhp\PhpType`, `\Tyhp\NativeTypeTest` (compile-only `NoEmit`), `\Tyhp\EraseGeneric` (compile-only `NoEmit`), `\Tyhp\GenericRuntime` (runtime-visible), `\Tyhp\Generic::bind`, `Contracts\IsDisposable`, `Contracts\ArrayAccessShape`, `ObjectHelper`; `$s->length()` via `global use extension \Tyhp\StringExtensions` |
| `tyhp/decimal` | `decimal` | `\Tyhp\Decimal`, `\Tyhp\decimal()` |
| `tyhp/async` | `async`/`await` | `\Tyhp\Promise`, `PromiseSettledResult`, `Deferred`, `EventLoop`, `CancellationToken(Source)`, `DisposableScope` (what `:=` compiles to), `DisposableHelper`, `Contracts\AsyncIterator`/`AsyncIterable`/`AsyncKeyValueIterator`/`AsyncIsDisposable` |
| `tyhp/lambda` | `Expression<>`/`PropertyPath<>` | `\Tyhp\Expression`, `ExpressionVisitor` |
| `tyhp/parallel` | Explicit `composer require`. A CLI program hosts the pool in-process (`ext-pcntl`, `ext-posix`). php-fpm sets `TYHP_PARALLEL_SOCKET` and uses the `tyhp-parallel serve` worker service. Result types are inferred from the task class: `await Parallel::runAsync(new MyTask())`. Not reported by the compiler and not part of the TYHP8027 expected set | `\Tyhp\Parallel`, `\Tyhp\ParallelTask`, `\Tyhp\ParallelRemoteError` |
| `tyhp/compiler` | `require-dev`; written by `tyhp init` (exact version) and `tyhp composer sync`, or `@dev` by the `tyhp/core` plugin. `vendor/bin/tyhp` shim downloads the native compiler via the `post-autoload-dump` hook (`vendor/bin/tyhp --install-binary`); needs `ext-curl` | none (`vendor/bin/tyhp`) |
| `tyhp/io` | Explicit `composer require`. Non-blocking HTTP (`ext-curl`), processes, files, sockets, Redis, PostgreSQL (`ext-pgsql`), MySQL (`ext-mysqli`), and PDO on the `tyhp/async` event loop. PDO, DNS, and `Offload` jobs use `tyhp/parallel` workers when it is installed. Not reported by the compiler and not part of the TYHP8027 expected set | `\Tyhp\Io\Http\HttpClient`, `\Tyhp\Io\Process\Process`, `\Tyhp\Io\File\File`, `\Tyhp\Io\Socket\Socket`, `\Tyhp\Io\Redis\RedisClient`, `\Tyhp\Io\Db\*Connection`, `\Tyhp\Io\Offload` |

One `tyhpdef/php` (and one `tyhpdef/php-ext-*` per extension) covers 8.2–8.5. Composer `"php": ">=8.2"`. No `tyhpdef/php-8.2` forks. Gates: `declare(php=…)` / `#[\Tyhp\Php]` vs `output.phpVersion`. `declare(ext="name")` is satisfied when a loaded tyhpdef package requires `ext-name`. json/hash/libxml stay in `tyhpdef/php` only.

Libraries declare ambient tyhpdefs in `extra.tyhp.require` (duplicate in `require-dev`; any Composer
name). Author-only wrappers stay require-dev — `tyhp/decimal` extras are `tyhpdef/php` only.
`tyhp/core` extras are `tyhpdef/php` only (`tyhpdef/php-ext-mbstring` stays author-only in core `require-dev`). First `composer require tyhp/core`
often misses that transaction: `composer update` or `tyhp composer sync`
([handbook §1](../handbook/01-project-setup.md)).

**Package tests:** [tyhp-runtime-src](https://github.com/tyhpproject/tyhp-runtime-src) `packages/test-all-tyhpdef.sh` lints each `tyhpdef/*` `tests/` tree, one job per package/version (`--jobs`, default CPU count). `--package` limits the run to one path; the default is every package. A package that runs longer than 90 seconds (`--timeout`) fails. Positives must be clean. Expected errors are `tests/fail/*.tyhp` plus sidecar `tests/fail/<stem>.expect.json`. Only `packages/tyhpdef/` is tested; the compiled `tyhp/*` runtime packages under `packages/tyhp/` are not.

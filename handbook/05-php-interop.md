## 5. PHP interop — using existing PHP libraries

Compiled Tyhp is ordinary PHP, so calling a PHP library is about giving Tyhp its **types**:

1. Install the library the normal way (`composer require vendor/lib`).
2. **If it ships Tyhp types** (`extra.tyhp.package` on `composer.json` pointing at `.tyhpdef`/`.tyhp`), they're
   discovered automatically — just `use` the classes.
3. **If it doesn't**, write a `.tyhpdef` describing only the symbols you call, and point config at it
   (`tyhpdefInclude`, or list it in `include`):
   ```tyhpdef
   <?tyhpdef
   namespace Monolog;
   class Logger {
       public function __construct(string $name): void;
       public function info(string $message, array<string, mixed> $context = []): void;
       public function error(string $message, array<string, mixed> $context = []): void;
   }
   ```
4. Use it from `.tyhp`; the emitted PHP references the real class (`\Monolog\Logger`) unchanged:
   ```tyhp
   <?tyhp
   namespace App;
   use Monolog\Logger;
   function makeLogger(): Logger {
       Logger $log = new Logger('app');
       $log->info('ready');
       return $log;
   }
   ```

Notes: always-present PHP stdlib is `tyhpdef/php` (version-gated to `output.phpVersion`); optional
extensions are `tyhpdef/php-ext-*` (`composer require --dev tyhpdef/php-ext-curl`). That package's
`require` of `ext-curl` is what makes curl present for `declare(ext="curl")`. Polyfill signatures
that exist only when the extension package is absent use `declare(ext="!curl")` around
`fallback function` / `fallback const`. Scalar methods
(`$s->length()`) come from `tyhp/core` and need no import. A compiled Tyhp library's `package.tyhpdef`
is the same Tyhp surface as its source: `#[\Tyhp\GenericRuntime]` is stamped on every emitted generic.
A foreign layout-2 call emits `name(types)(args)`. A foreign class `new` emits `\Tyhp\Generic::bind`.
An `#[\Tyhp\InferGeneric]` call with explicit type arguments emits the `name__tyhpGeneric` companion.
`internal` symbols are omitted.
`tyhpdef/php-ext-decimal` is PECL Decimal, not `tyhp/decimal`.
json/hash/libxml stay in `tyhpdef/php`. Anything unknown, declare it in a `.tyhpdef`. To stub a
Composer package or unpublished PHP extension:

```
tyhp generate_tyhpdef --package-path=./vendor/monolog/monolog --output=./tyhpdef/
tyhp generate_tyhpdef --ext-name=curl --php-targets=8.2,8.3,8.4,8.5
tyhp overlay create \Iterator
tyhp overlay stamp
```

Then `include` the generated files (or rely on `extra.tyhp.package` on that package’s `composer.json`). Harvested parameter defaults that are not tyhpdef literals stay optional with `= null` (same dummy as harvested implicit-nullable PHP; the declared type is not widened); const and property `??` values that are not literals are omitted. Unmatched `#[Name(args)]` is `#[Name]`. A harvested type whose short name is a reserved word (`is`, `isset`, …) is written fully qualified (`class \Hamcrest\Core\Is`). PHP `static` in a value position (parameter, property, `@param`, `@var`, magic `@method` parameter, `@property`) is written as `self`; return positions keep `static` (`: static`, magic `@method` returns, including generic arguments such as `Builder<static>`). PHPDoc `@template` bounds qualify nested class names to FQCNs; missing generic args are filled from that type's default, constraint, or `mixed`. Native `extends` / `implements` / trait `use` prefer a same-package declared type over a unique PHP / `\Psr\*` short of the same name ([handbook §3](03-build-cli-workflow.md)). PHP-source harvest omits types whose parent is a catalog `extern` or `@internal` (unless `--include-internal`); descendants and members that name those FQCNs are dropped. A bare PHPDoc name with no import is not a cross-namespace last-segment match. When a later harvested type has the same FQCN as one already emitted and the body is identical (kind, modifiers, extends/implements, members, flags — not doc comments), generate keeps the first, skips the later, and warns. Divergent bodies are both written; include-layer `TYHP8002` still fires. Overlay `partial class Foo<T>;`
replaces written header clauses without restating members. Overlay `partial function X as Y`
renames the live Tyhp name without restating the signature; inspect with `tyhp symbol_tree --filter=<name>`.
Overlays: [guide §23](../guide/23-tyhpdef.md).
Flags: [handbook §3](03-build-cli-workflow.md).

**Libraries:** fill `extra.tyhp.require` with tyhpdefs consumers must install (duplicate in
`require-dev`; any Composer name). Leave author-only wrappers out of extras. First
`composer require tyhp/core` often misses ambient pins — `composer update` or `tyhp composer sync`
([handbook §1](01-project-setup.md)).

Symbol resolution order ([guide §24](../guide/24-php-interop.md)) is built-in → embedded tyhpdef → vendor package → your tyhpdef →
your `.tyhp`, and **the first registration wins** — you can't shadow a library symbol with a
same-named local declaration.

**Emitted docblocks (what PHP consumers can rely on).** Every build profile (`debug`, `balanced`,
`release`) writes PHPStan/Psalm tags where the native declaration drops a type the checker has.
Independent of `output.comments`. The compiler does not run PHPStan or Psalm; run them on the emitted PHP.

| Key | Default | Effect |
|-----|---------|--------|
| `emit.docblocks` | `lossy-only` | Tag only when it beats the native hint. `all` tags every signature type. `off` emits no generated tags (including generated `@template` and lowered-hook `@property`). |
| `emit.docblockDialect` | `both` | `phpstan` \| `psalm` \| `both` \| `phpdoc`. Parsed and validated. Does not change output. Shared syntax is unprefixed; a `@phpstan-` / `@psalm-` prefix is used only where the dialects diverge, and no current tag needs one. |

Invalid value → warning `TYHP6003`; the default stays.

Tags when the checker has the data: `@param` (variadic `...$name`), `@param-out` on a by-ref parameter,
`@return`, `@var`, `@template` / `@template-covariant` / `@template-contravariant` (`of Bound` and
`= Default` when the parameter has them), `@extends` / `@implements` / `@use` (under `lossy-only`,
when the reference has type arguments), `@internal`. `@throws` is not generated.

Merge: summary prose, then `@tyhp` directives and `/@@ … @@/` blocks in original order, then tags.
An author analysis tag wins per key and is copied as written (that key's place, signature order);
the generator fills gaps. Author tags with no generated counterpart, including every `@throws`, follow.
`output.comments: false` drops prose and keeps directives and every author analysis tag.

Generated names are fully qualified; `use … as X` expands to the real name. `self`, `parent`, and
`static` stay relative. `class-string` / `class-string<T>` only for class, interface, and enum name
brands; trait, used-trait, and struct name brands are `string`; a function-name brand is
`callable-string`. A struct parameter is `array{…}`; a field with a default or a nullable type is
`key?:` (`key?: ?type` when nullable). Callables are `callable(int): string`.
Parentheses keep a union unambiguous: `(callable(int): string)|null`, `callable(): (int|string)`,
`callable(int): ?string` for a `string|null` return. Public property hooks lowered below PHP 8.4
also get `@property` / `@property-read` / `@property-write`.

**PHP that calls `tyhp/lambda`.** Build the tree with `_expr` / `Expression::of` and `_path` /
`PropertyPath::on`. Ship that fluent form. A package cannot know whether a later process has `ext-tyhp`
loaded, or which version is loaded. The fluent form is the recommended one for speed and stability
whether or not the module is installed. A caller who knows the process has `tyhp` at `0.1.0` or newer
may pass a closure. That is the easier spelling in that case. `tyhp build` still emits the `backedBy`
chain for an inline `fn`. `ext-tyhp` is not required. Installing it is recommended: `pie install tyhp/ext-tyhp`.
No Tyhp package requires it. `tyhp/core`, `tyhp/lambda`, and `tyhp/io` suggest it.

```tyhp
<?tyhp

type Money = struct {
    int $cents;
    string $currency = "USD";
    ?string $note;
};

function price(Money $money): int {
    return $money->cents;
}
```

```php
/**
 * @param array{cents: int, currency?: string, note?: ?string} $money
 */
function price(array $money): int
```

## 23. Typing external PHP: `.tyhpdef` (signatures only, like `.d.ts`)

```tyhpdef
<?tyhpdef
namespace Acme\Payments;
use Acme\Currency\Formatter;
deprecated class LegacyMoney { public function __toFloat(): float; }   // `deprecated`/`obsolete` keywords
class Money implements \Stringable {
    public readonly string $amount;
    public string $label { get; set; }
    public array $parts { &get; }
    public function plus(Money $other): self;
    extension operator +(self $a, self $b): self => $a->plus($b);      // thin mapping; erased
    use extension Formatter { Formatter::format as formatMoney; };      // attach extension
}
class Instant { operator +(self $left, DateInterval $right): Instant; } // native passthrough
async function fetchRate(string $currency): Promise<float>;
const string DEFAULT_CURRENCY = 'USD';
function strlen as str_len(string $s): int;

extension StringOps extends string {
    fn toUpper(): string => \strtoupper($this);
    operator * (self $left, int $right): string => \str_repeat($left, $right);
}
global use extension StringOps;

#[\Tyhp\Php(">=8.4")]
function array_find(array $array, callable $callback): mixed;
```
- Declarable: functions, classes, interfaces, traits, enums, constants (`const T NAME;`, optional
  `?? default`), globals (`T $var;`), structs, type aliases (file-level and class-body `type` with
  visibility), `namespace`/`use`, standalone
  `extension { }` (header `extends Type` or nested `extends` / `extends<T>` groups; `fn`/`operator` members: `=>` only, `$this` implied, return type required, empty `{}` is an error).
  Class-body `extension fn` / `extension operator` are the same thin `=>` mappings (no brace bodies,
  no generated PHP). A parameter a mapping writes must be declared `&`.
  Class/interface/trait properties may be hooked as a bodyless `{ get; set; }` / `{ &get; }` list
  (no `get { }` / `get =>`; one `$name`; `#[\Tyhp\Php]` on the property, not the hook — `TYHP8016`).
- `deprecated`/`obsolete` are keywords (not docblocks); checker warns on use.
- `extends` on an imported function's first param = extension-method import.
- Inline `.tyhpdef` class-body members are thin `=>` mappings only (no brace bodies, no generated
  PHP; PHP cannot call them). They are auto-active for consumers.
- **Operators:** bodyless `operator …;` = native PHP passthrough (no emit rewrite);
  `extension operator` **requires a thin `=>` expression** and splices call sites. Bodyless
  `extension operator` is an error.
- **`global use` / `global use function` / `global use const` / `global use extension`** apply for
  the whole compilation. A compiled library copies in-package `global use` that it authored into
  `package.tyhpdef`. File-level `use` is not copied. `use extension { … hide; … insteadof; }` as in [§12](12-extensions.md).
- **`#[\Tyhp\GenericRuntime]`** is a real PHP attribute (not `NoEmit`) stamped on every emitted
  generic class/method/function (`erased`, `layouts: [1]`, `factory`/`binder` only when helpers
  exist, optional `compiler:`). Foreign consumer sites emit `\Tyhp\Generic::bind(...)(...)`.
  Overlay tyhpdefs that omit the stamp keep erase / inline. Unknown layouts → `TYHP5023`.
  `#[\Tyhp\EraseGeneric]` (`NoEmit`) opts properties out of Mechanism C tracking.
- **PHP version gates** (`declare(php="…")` / `#[\Tyhp\Php]`) — [§30](30-php-version-gating.md).
  `#[\Tyhp\Php]` is illegal on `struct` / `extension`.
- **`fallback function` / `fallback const`** — the name is filled only when nothing else declared it
  (`if (!function_exists)` / `if (!defined)`). Several packages: first file in
  `vendor/composer/autoload_files.php` wins. `tyhpdef/php` beats every fallback. A loaded package
  whose `require` or `require-dev` contains `ext-X` also beats a fallback of that name. Same-package
  plain `function` / `const` replaces that package's fallback. Later mismatched fallback warns
  (`TYHP8037` / `TYHP8042`) and is ignored. No autoload list and two fallbacks → `TYHP8036` /
  `TYHP8041`. Ordinary declaration loaded after a fallback → `TYHP8038` / `TYHP8043`. A user
  `.tyhp` function or const of the same name is a duplicate. Combines with `deprecated` / `obsolete`
  / `async`. Not with `partial`, `omit`, or `extern`. Not on classes or methods.
- **`declare(ext="name")` / `declare(ext="!name")`** — alone in that `declare` (`TYHP8039`). Value is
  the extension name or `!` plus it (`TYHP8040`), not `ext-intl`. Present means a **loaded tyhpdef
  package** requires `ext-name`. The app `composer.json` `ext-*` line does not count by itself.
  Nest inside `declare(php=…)` when both apply. File-level `;` gates the file, same as `declare(php)`.
  Polyfill shims that exist only when the extension package is absent:

```tyhpdef
declare(php="<8.6") {
    declare(ext="!intl") {
        fallback function grapheme_strrev(string $string): string|false;
        fallback const int GRAPHEME_EXTR_COUNT ?? 0;
    }
}
```

  Harvest (`tyhp generate_tyhpdef` from PHP) emits those forms for top-level
  `if (!function_exists)` / `if (!defined)` around one declaration, and for a top-level
  `if (extension_loaded|function_exists|defined|PHP_VERSION_ID …) return;` (including
  `return require 'other.php'`). `if (!extension_loaded(…)) return;` drops what follows.
  Other `if` bodies are skipped.
- A library exposes types via `extra.tyhp.package` on that package’s `composer.json`:
  `{ "include": ["./_tyhpdef/*.tyhpdef"], "overlay": ["./_tyhpdef/overlays/stubs/*.tyhpdef", "./_tyhpdef/overlays/*.tyhpdef"] }`.
  `include` first (duplicates error). `overlay` after, **array order, last Tyhp name wins** (full
  replace / overlay `partial` member merge / overlay `partial` header `;` / overlay `partial function` / `omit`).
  Stubs first, hand last. Overlay `partial` missing target → warning; include `partial` missing
  target → error. Overlay `partial class Foo<T> implements …;` (also interface/trait/enum) replaces
  written header clauses; omitted clauses and members stay (overlay-only, `TYHP8034`). Brace
  `partial class Foo { }` is member merge only (written headers ignored). `X as Y;` renames unless
  the same overlay file also keeps `X`. Bare `Foo;` with no clauses, attributes, or keep-for-alias
  warns `TYHP8035`. Overlay `partial function` is name-only (no signature): `X;` keep + attributes;
  `X as Y;` replace `X` unless the same overlay file also keeps `X`. Missing target → warning;
  include use → error. Match the **current** Tyhp name. `omit` is overlay-only.
  `// @overlay-against:` stamps Layer 1; mismatch warns (`TYHP8021`). Name-only stamps are valid on
  `partial function`. Layer 2 harvest emits header `;` plus a member `partial` instead of a full
  type rewrite. `tyhp overlay create` / `stamp`. `tyhp symbol_tree --filter=` dumps live names.
  `"type": "library"` builds always write `package.tyhpdef` and additive-merge `extra.tyhp.package` into
  publish-directory `composer.json`. Generate stubs from PHP with `tyhp generate_tyhpdef` ([handbook §3](../handbook/03-build-cli-workflow.md)).
  Harvest writes PHPDoc `callable(...)` / `closure(...)` as `callable` and Psalm/PHPStan `empty`
  in a type as `mixed` (the PHP `empty()` construct is unchanged). PHP `static` in a value
  position (parameter, property, `@param`, `@var`, magic `@method` parameter, `@property`) is
  written as `self`; return positions keep `static` (`: static`, magic `@method` returns,
  including generic arguments such as `Builder<static>`). A harvested `class` /
  `interface` / `trait` / `enum` whose short name is a PHP or tyhpdef reserved word (`is`,
  `isset`, …) is written fully qualified (`class \Hamcrest\Core\Is`) so the name is not
  tokenized as a keyword. Alias headers and `extern function` names in a `namespace {}` use
  the same enclosing-namespace prefix. Unqualified PHPDoc / hint names
  with no `use` become catalog FQCNs when the short is unique among `php` / `ext-*` wrappers and
  unique `\Psr\*` types (not Composer-library class names), or same-package FQCNs when the type
  is declared in the harvest. Native `extends` / `implements` / trait `use` prefer that
  same-package type even when the short is a unique PHP / `\Psr\*` global
  (`implements Exception` → `\Acme\Exception\Exception`); a true global the package does
  not declare (`extends \InvalidArgumentException`) stays global. That rewrite walks
  identifiers inside generics / unions /
  intersections (`@template T of Foo<object>`). Template parameters (`T`) and builtins stay
  unqualified. A generic bound written without type arguments is filled from that type's own
  default, else constraint, else `mixed`. PHP-source harvest omits a type whose immediate
  `extends`/`implements` is a catalog `extern` (`suggest` / not in `require`) or that is
  `@internal` (`--include-internal` off). Descendants that `extends`/`implements` an omitted
  type are omitted; members, trait `use`, functions, and aliases that name an omitted FQCN
  are stripped (`use` imports and already-qualified names). A bare PHPDoc name with no import
  is not a cross-namespace last-segment match. Intra-package unique-short qualify runs after
  omit. `--include-internal` keeps `@internal` types and their references. When a later
  harvested type has the same FQCN as one already emitted and the body is identical (kind,
  modifiers, extends/implements, members, flags — not doc comments), generate keeps the
  first, skips the later, and warns. Divergent bodies are both written; include-layer
  `TYHP8002` still fires. Harvest keeps a
  tyhpdef-safe parameter `=` / const or
  property `??` literal; an unrepresentable parameter default stays optional as `= null` (the same
  dummy as harvested implicit-nullable PHP; the declared type is not widened). Const and property
  values omit an unrepresentable `??` (`const string NAME;`). Unmatched `#[Name(args)]` is
  `#[Name]`; copied PHP-source doc lines replace `*/` with `* /`. Catalog indexing of existing
  tyhpdefs treats `{` / `}` inside `/** */`, `/* */`, `//`, and quoted strings as non-delimiters.
  `--audit-stubs` reports Layer 2 vs stub corpora. `internal` declarations are omitted from
  `package.tyhpdef`.
- **`extern`** — name-only placeholder for a type, function, or const this package mentions but
  does not own (`suggest` / `require-dev` peers, author-only tyhpdefs). `extern \Foo\Bar;` when
  the kind is unknown; `extern class` / `interface` / `enum` / `function` / `const` when known
  (`extern function \bcadd;`, `extern const \FOO;` — no signature or value). A specified type
  kind must match the originating declaration (`interface` → `extern interface`, `class` →
  `extern class`, `enum` → `extern enum`); a mismatch is `TYHP8029`. Legal in tyhpdef
  signatures; `.tyhp` use is `TYHP4307` until the providing wrapper is included (real declaration
  replaces the placeholder). Optional `// @provided-by: tyhpdef/…`. Do not overlay a hollow
  `class \Foreign\Type {}`.
  Library `tyhp build` writes `extern` into `package.tyhpdef` for names owned by `require-dev`
  but not `require` and not `extra.tyhp.require`. Ambient extras and runtime `require` owners
  stay FQNs. `internal` symbols are omitted from that file. Authors fill `extra.tyhp.require`
  with what consumers must install (any Composer name; duplicate in `require-dev`); never put
  author-only tyhpdefs in extras ([handbook §1](../handbook/01-project-setup.md)).

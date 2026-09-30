## 3. Build & CLI workflow (what works today)

**Dev loop:** `tyhp lint` (type-check, no emit) → `tyhp build` (emit) → run the PHP.

| Command | Status | Use |
|---------|--------|-----|
| `build` | ✅ | parse→bind→check→emit→write `output.path`. |
| `lint [paths]` | ✅ | same checks, no emit; `--format=text\|json\|sarif`, `--file`, `--quiet`. `--fix` is a stub. |
| `init [dir]` | ✅ | `basic` template only. |
| `version [--json]` | ✅ | compiler / .NET / ANTLR / OS. |
| `help [--subject=…]` | ✅ | also `tyhp --help`, `tyhp build --help`. |
| `explain TYHP####` | ✅ | long-form diagnostic (`--explain TYHP4008`). |
| `composer <args>` | ✅ | proxies Composer (`php` + `composer` on PATH). After `require`/`install`/`update`, runs `generate_tyhpdef --vendor` unless `--no-tyhpdef`. `tyhp composer sync` writes `extra.tyhp.require` onto root `require-dev`. |
| `generate_tyhpdef` | ✅ | stubs from a PHP extension, Composer package, or `.php` globs (see below). |
| `overlay` | ✅ | `create <FQN>` copies a declaration into a hand-written overlay; `stamp` writes `@overlay-against`. |
| `language_server` | ✅ | LSP over **stdio** (no `--tcp`). |
| `xdebug_proxy` | ✅ | IDE :9003 ↔ Xdebug :9004; needs sourcemaps. |
| `tokenize` / `dump_ast` / `dump-ast` | ✅ | debug JSON (lex / parse). |
| `symbol_tree` / `symbol-tree` | ✅ | parse+bind (no check/emit); JSON of the live environment. `--filter=<substring>` (Tyhp name / FQN / PHP name), `--out=<file.json>`. Status on stderr. |
| `clear_cache` | ✅ | delete AST parse cache. |
| `integrity_check` | ✅ | config / tyhpdef parse / cache / environment. |
| `watch` | ⚠️ | not implemented. |

**`build` flags:** `--clean`, `--dry-run`, `--strict` (lint also honors `--strict`), `--verbose`,
`--no-cache`, `--quiet`, `--suppress-warnings`, `--fix` (write extras + Composer-update; same as
`tyhp composer sync`). Incremental state: `tyhp-build-state.json` under the project cache directory (`cache-dir`).
Library `"type": "library"` always writes `package.tyhpdef` with the same Tyhp surface as source
(attributes, class-level `type`, in-package `global use`, `#[\Tyhp\GenericRuntime]` on every emitted
generic; foreign consumers use `\Tyhp\Generic::bind`) and additive-merges
`extra.tyhp.package` onto publish-directory `composer.json`. `internal`
is omitted. Author-only `require-dev` owners → `extern`; `extra.tyhp.require` / `require` → FQN.

Default `tyhp build`/`lint` **explain** stale extras (`TYHP7701` fails compile; `TYHP7702`
compiler pin is a warning unless `--strict`). They never auto-update Composer. First
`composer require tyhp/core` often misses extras — `composer update` or `tyhp composer sync`.
Allow `tyhp/core` under `config.allow-plugins` (`tyhp init` / `tyhp composer` write it when missing). `--no-dev` does not rewrite `composer.json`.

**Diagnostics:** `file(line,col): error TYHP####: message` ([guide §27](../guide/27-diagnostics.md)).
`lint --format=json|sarif` → stdout document only; progress on stderr.

### `tyhp generate_tyhpdef`

Exactly one of `--ext-name`, `--package-path`, `--source` (unless `--validate` or `--audit-stubs` alone).

```
tyhp generate_tyhpdef --ext-name=curl
tyhp generate_tyhpdef --package-path=./vendor/guzzlehttp/guzzle --output=./tyhpdef/
tyhp generate_tyhpdef --source=./lib/**/*.php
tyhp generate_tyhpdef --validate ./tyhpdef/
tyhp generate_tyhpdef --audit-stubs=../tyhp-runtime-src/packages/php/_tyhpdef --out=stub-audit.md
tyhp generate_tyhpdef --ext-name=curl --php-targets=8.2,8.3,8.4,8.5
```

- **`--ext-name`:** reflect a PHP extension via **Tyhp-managed PHP** (not `PATH` / `PHP_BINARY`).
  `--php=<path>` is a single-target escape hatch; illegal with `--php-targets` (`TYHP7510`).
- **`--package-path`:** Composer dir, C# PHP parser (no PHP runtime). `--include-dev` adds
  `autoload-dev`. No `--composer-package` alias.
- **`--source`:** `.php` globs only (`TYHP7507` otherwise). Don't pass `.tyhp` — library APIs come from
  `tyhp build`.
- Harvested signatures are tyhpdef text: PHPDoc `callable(...)` / `closure(...)` → `callable`;
  Psalm/PHPStan `empty` in a type → `mixed`; unmatched types → `mixed`. PHP `static` in a
  value position (parameter, property, `@param`, `@var`, magic `@method` parameter,
  `@property`) is written as `self`; return positions keep `static` (`: static`, magic
  `@method` returns, including generic arguments such as `Builder<static>`). The PHP `empty()`
  construct is unchanged (`bool`, emitted `empty(...)`). A harvested `class` / `interface` /
  `trait` / `enum` whose short name is a reserved word (`is`, `isset`, …) is written fully
  qualified (`class \Hamcrest\Core\Is`); alias headers and `extern function` names in a
  `namespace {}` use the same enclosing-namespace prefix. Unqualified PHPDoc / hint names with no
  `use` become catalog FQCNs when the short is unique among `php` / `ext-*` wrappers and unique
  `\Psr\*` types (not Composer-library class names), or same-package FQCNs when the type is
  declared in the harvest. Native `extends` / `implements` / trait `use` prefer that
  same-package type even when the short is a unique PHP / `\Psr\*` global
  (`implements Exception` → `\Acme\Exception\Exception`); a true global the package does
  not declare (`extends \InvalidArgumentException`) stays global. That rewrite walks
  identifiers inside generics / unions /
  intersections (`@template T of Foo<object>`). Template parameters (`T`) and builtins stay
  unqualified. A generic bound written without type arguments is filled from that type's own
  default, else constraint, else `mixed`. `{` / `}` inside `/** */`, `/* */`, `//`, and
  quoted strings are not catalog namespace delimiters. Parameter `=` / property and const `??`
  keep a single-line quoted string, number, `true`/`false`/`null`/`[]`, or `Class::CONST`. Nowdocs,
  regex bodies, unquoted or unmatched defaults are not copied. Parameters stay optional with
  `= null` (same dummy as harvested implicit-nullable PHP; the declared type is not widened).
  Consts and properties omit `??` (`const string NAME;`). Unmatched
  `#[Name(args)]` → `#[Name]`. Copied PHP-source doc lines replace `*/` with `* /`. Restore
  specific start values in a hand overlay.
- Default out: `{project}/tyhpdef/`. `--split=file|namespace|type` (default `file`).
- `--php-targets` emits one tree gated with `declare(php=…)` / `#[\Tyhp\Php]`. `--php` is the
  ungated escape hatch for private unpublished extensions (illegal with `--php-targets`).
  Lint/build at `output.phpVersion` 8.2–8.5; contributors do not install a PHP version matrix.
  Layer 2 harvest emits overlay `partial class Foo<T> implements …;` plus a member `partial` instead
  of a full type rewrite ([guide §23](../guide/23-tyhpdef.md)).
  `--audit-stubs=<path>` reports Layer 2 vs Psalm/PHPStan/PhpStorm/Phan consensus without regenerating
  files (`--out` writes markdown). Optional-peer catalog hits emit `extern` with the originating
  kind (`extern class` / `interface` / `enum`); a specified-kind mismatch is `TYHP8029`
  ([guide §23](../guide/23-tyhpdef.md)).
  PHP-source harvest omits a type whose immediate `extends`/`implements` is a catalog `extern`
  or `@internal` (`--include-internal` off); descendants and members that name an omitted FQCN
  are dropped. A bare PHPDoc name with no import is not a cross-namespace last-segment match.
  Intra-package unique-short qualify runs after omit. `--include-internal` keeps `@internal`
  types and their references. When a later harvested type has the same FQCN as one already
  emitted and the body is identical (kind, modifiers, extends/implements, members, flags —
  not doc comments), generate keeps the first, skips the later, and warns. Divergent bodies
  are both written; include-layer `TYHP8002` still fires.

### `tyhp overlay`

```
tyhp overlay create \Iterator
tyhp overlay stamp
```

Overlay last-wins, brace `partial`, header-only `partial class Foo<T>;` (keep / `as` rename), overlay `partial function` (keep / `as` rename), and `omit`: [guide §23](../guide/23-tyhpdef.md).

### `tyhp symbol_tree`

Parse and bind the project (same environment as `lint` / `build`; no type-check or emit), then dump the resolved declarations as JSON.

```
tyhp symbol_tree
tyhp symbol_tree --filter=strtoupper
tyhp symbol_tree --out=symbols.json
```

`--filter` is a case-insensitive substring on the current Tyhp name, FQN, or PHP emit name (not `--name` — that collides with `tyhp.json` `"name"`). JSON goes to stdout or `--out`; status to stderr. Empty project still binds builtins and loaded tyhpdefs. Use this to confirm overlay `partial function` / `partial class` keep / rename.
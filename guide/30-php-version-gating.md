## 30. PHP version gating

Keep or drop declarations by PHP version. `output.phpVersion` (unset → `"8.2"` + warning `TYHP4306`
once per compilation) is the **oldest PHP the build runs on**, with no upper limit. Supported values:
`"8.0"`–`"8.5"`. Invalid explicit value → warning `TYHP6006` and fallback `"8.4"`. The whole emitted
file uses syntax that parses on that version.

This is **not** `function_exists` / declaration-existence gating (§29 `fallback`), and **not** the
managed PHP that `tyhp generate_tyhpdef --php-targets` downloads.

### What `.tyhp` emits

`declare(php=…)` and `#[\Tyhp\Php]` never appear in PHP. Relative to `output.phpVersion`:

| Constraint true for… | Emitted |
|----------------------|---------|
| no version at or above it | nothing |
| every version at or above it | the body, no check |
| only some | the body inside `if (\PHP_VERSION_ID …)` |

```tyhp
declare(php="<8.6") { function legacy(): string { return 'legacy'; } }
```

emits (with `output.phpVersion` `"8.2"`):

```php
if (\PHP_VERSION_ID < 80600) {
    function legacy(): string { return 'legacy'; }
}
```

Applies to functions, classes, interfaces, traits, enums and file-level `const` (as `\define()` inside
the check). Nested gates nest, outermost first. Guarded functions go after the ungated ones in
`_functions.php`; a guarded class gets its own file with its own copy of the check. If the same
function/type name is also declared under another version gate that the minimum does not satisfy,
only the declaration matching the minimum is compiled and it is emitted without a check.
`.tyhpdef` gates are never emitted.

### `declare(php="…")`

Key must be **alone** in that `declare` (`TYHP4301` if mixed with `strict_types` / `output_file` / …).
Value is a Composer constraint string. Bare `"8.2"` / `"=8.2"` = the whole `8.2.*` minor.

```tyhp
<?tyhp
declare(strict_types=1);
declare(php=">=8.4");          // file-level: if unmatched, the file's symbols are silently absent

function always(): void {}

declare(php=">=8.4") {
    function json_validate(string $json, int $depth = 512, int $flags = 0): bool { return true; }
}
```

Colon/`enddeclare` is the same as a brace block. Nested gates **AND**. Sibling disjoint blocks are
alternate variants (not unreachable). Nested constraint that cannot overlap its outer gate →
`TYHP4302`.

### `#[\Tyhp\Php]`

```tyhp
#[\Tyhp\Php(">=8.4")]
function array_find(array $array, callable $callback): mixed { return null; }

#[\Tyhp\Php(version: ">=8.2 <8.4")]
function example(string $v): mixed { return $v; }
#[\Tyhp\Php(version: ">=8.4")]
function example(string $v, bool $strict = false): string { return $v; }
```

Legal on functions, classes, interfaces, traits, enums, and methods of classes/traits/enums.
**Illegal on `struct` and on Tyhp/tyhpdef `extension` (`TYHP4304`)** — wrap those in `declare(php=…)`.
**Illegal in `.tyhp` on a property, class constant, enum case or interface method (`TYHP4371`)** —
PHP cannot declare those conditionally; declare the whole class/interface/enum once per version in
`declare(php=…)` blocks. `.tyhpdef` members may still carry it.
Missing/non-string `version` → `TYHP4305`. Invalid constraint → `TYHP4300`.

Same symbol twice is OK iff effective constraints are pairwise **disjoint**; overlap → `TYHP4303` unless the declarations are distinct function/method overloads (different parameter lists, same static/instance-ness). A static method and an instance method of the same name still conflict under overlapping gates.

### `.tyhpdef`

Same file-level `;` and `{ }` declare forms. Put `#[\Tyhp\Php]` on classes/functions; wrap
`struct` / `extension { }` in `declare(php=…)`. One `tyhpdef/php` stubs package covers 8.2–8.5 this way;
the checker keeps only declarations that match `output.phpVersion`.

### `declare(ext="…")`

Extension presence, not PHP version. Alone in that `declare` (`TYHP8039`). `"intl"` type-checks the
block when a loaded tyhpdef package `require`s or `require-dev`s `ext-intl`. `"!intl"` type-checks it
when none do. Invalid value → `TYHP8040`. Nest with `declare(php=…)` (both must match). File-level `;`
gates the file. In `.tyhp` the matching block is emitted inside `if (\extension_loaded('intl'))` /
`if (!\extension_loaded('intl'))`; in `.tyhpdef` it only selects signatures.

### `fallback` in `.tyhp`

`fallback` before `function` / `class` / `interface` / `trait` / `enum` / `const` emits the
`if (!\function_exists(__NAMESPACE__ . '\name'))` form (`class_exists`, `interface_exists`, … by kind).
`fallback const` emits `if (!\defined(NAME)) { \define(NAME, value); }` (`'NAME'` in the global
namespace, `__NAMESPACE__ . '\NAME'` otherwise). Combines with `deprecated` / `obsolete` / `async`,
not `partial` / `omit` / `extern`. Version and extension checks wrap the existence check:

```tyhp
declare(php="<8.6") {
    declare(ext="!intl") {
        fallback function graphemeStrrev(string $s): string { return $s; }
    }
}
```

```php
if (\PHP_VERSION_ID < 80600) {
    if (!\extension_loaded('intl')) {
        if (!\function_exists(__NAMESPACE__ . '\graphemeStrrev')) {
            function graphemeStrrev(string $s): string { return $s; }
        }
    }
}
```

`fallback function` / `fallback const` in `.tyhpdef` are the name-still-empty signatures
(`function_exists` / `defined`). They compose inside an ext or php block. Winner: first
`autoload.files` entry; `tyhpdef/php` and any loaded package that requires `ext-*` beat a fallback of
the same name. Details and diagnostics: [§23](23-tyhpdef.md).

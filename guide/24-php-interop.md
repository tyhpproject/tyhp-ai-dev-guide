## 24. PHP interop & name resolution

- **Compiled output is ordinary PHP** in the same namespaces/class names, so plain PHP can call it and
  it can call plain PHP. To call PHP that has no Tyhp types, describe it in a `.tyhpdef` ([§23](23-tyhpdef.md)); the
  standard library comes from `tyhpdef/php`; optional extensions from `tyhpdef/php-ext-*`
  (`require` `ext-*` on that package is the `declare(ext="…")` signal). Polyfill-only symbols are
  `declare(ext="!name")` plus `fallback function` / `fallback const` ([§23](23-tyhpdef.md)).
- Namespaces, `use`, FQCN (`\A\B\C`), and `use function`/`use const` **resolve like PHP**. Constant
  names are case-sensitive and matched before the case-insensitive class/function index. PHP allows
  a class and a nested namespace to share an FQN (`class \Ns\Foo` plus types under `\Ns\Foo\…`);
  inherited members still resolve from that parent. In type position, `\Ns\Foo\Bar` is a class-level
  alias `Bar` on `\Ns\Foo` when that alias exists ([§9](09-type-aliases.md)); otherwise it is the
  nested type.
- **Symbol discovery order** (first registered wins; earlier layers shadow later same-name symbols):
  1. built-in types/functions (compiler) → 2. embedded tyhpdefs → 3. vendor `composer.json` with `extra.tyhp.package`
  packages → 4. your `tyhpdef` include paths → 5. your `.tyhp` sources. A duplicate name in the same
  scope is an error; you cannot override a tyhpdef symbol with a same-named user declaration.

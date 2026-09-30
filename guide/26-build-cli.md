## 26. Build, output layout & CLI

`tyhp.json`:
```json
{
  "include": ["src/**/*.tyhp"],
  "exclude": ["vendor/**"],
  "source":  { "tagless": false },
  "output":  { "path": "build/", "phpVersion": "8.4", "strictTypes": true, "namespacePrefix": null },
  "psr4":    { "App\\": "src/" },
  "build":   { "decimalBacking": "bcmath", "decimalScale": 28, "structBacking": "array", "updateComposer": false }
}
```
- **`output.phpVersion`:** `"8.0"`–`"8.5"`. Omitted → `"8.2"` + `TYHP4306` once. Unsupported explicit
  value → `TYHP6006` and `"8.4"`. Drives emit lowering (pipe, `(void)`, `clone(…)`, property hooks)
  and [§30](30-php-version-gating.md) gates. Not the managed PHP used by `generate_tyhpdef`.
- **Output layout** (under `output.path`, default `build/`): classes/extensions →
  `build/<Namespace/Segments>/<Class>.php` (from FQN; `psr4` does **not** remap disks — Composer
  autoload only); entry files mirror source; namespace functions → `_functions.php`.
- **`composer.json`** is written/merged only when `build.updateComposer: true`; it then `require`s
  runtime packages, path-repos a sibling `tyhp-runtime-src/packages/*` (`@dev`), and runs `composer install`.
- **Pipeline:** parse → bind → check → emit. **All-or-nothing:** any error (or `--strict` warning)
  emits nothing. Checker continues after errors (cap 100/file). Incremental state:
  `tyhp-build-state.json` under the project cache directory. `--clean` wipes outputs and that state file.
- **Library `"type": "library"`:** always writes `package.tyhpdef` and additive-merges `extra.tyhp.package` into
  publish-directory `composer.json` (`build.generateTyhpdef` ignored). Applications write `package.tyhpdef` only if
  `build.generateTyhpdef: true` (no `extra.tyhp.package`). Generated `package.tyhpdef` keeps attributes, class-level `type`,
  in-package `global use`, and `#[\Tyhp\GenericRuntime]` on every emitted generic (foreign sites
  use `\Tyhp\Generic::bind`; `factory`/`binder` names are not baked into consumer PHP).
  `internal` is omitted. Author-only `require-dev` owners become name-only `extern`;
  `extra.tyhp.require` and runtime `require` owners stay FQNs.
- **`extra.tyhp.require`:** library map of ambient Composer packages for consumers (any name;
  duplicate in `require-dev`). `tyhp/core` plugin + `allow-plugins.tyhp/core`. First
  `composer require tyhp/core` often misses that transaction — `composer update` or
  `tyhp composer sync`. Default `tyhp build`/`lint` explain (`TYHP7701`); `--strict` fails;
  `--fix` / `tyhp composer sync` writes; ordinary build never auto-updates.
  [handbook §1](../handbook/01-project-setup.md), [handbook §3](../handbook/03-build-cli-workflow.md).
- **`build.generateSourcemap`:** `true` writes Source Map v3 `*.php.map` beside each PHP file
  (`//# sourceMappingURL=…`). Default false. Failures warn (`TYHP5020`–`TYHP5022`) and still emit PHP.
- **CLI working today:** `build`, `lint`, `init`, `version` (`--json` OK), `help`, `explain`,
  `composer`, `generate_tyhpdef`, `overlay` (`create <FQN>` / `stamp`), `language_server` (stdio),
  `xdebug_proxy`, `tokenize`, `dump_ast`, `symbol_tree` (`--filter` / `--out`), `clear_cache`,
  `integrity_check`. ⚠️ `watch` is not implemented. Flags: `--clean --dry-run --strict --quiet/--verbose
  --no-cache --suppress-warnings`. `tyhp build --fix` / `tyhp composer sync` writes extras (not
  ordinary `build`). `lint --format=text|json|sarif`, `--file`. `generate_tyhpdef --php-targets=8.2,8.3,8.4,8.5`
  (managed PHP; `--php` only for private extensions). Lint/build the same tree at `output.phpVersion`
  8.2–8.5. `--audit-stubs` reports Layer 2 vs stub corpora. Details in `../handbook/03-build-cli-workflow.md`.

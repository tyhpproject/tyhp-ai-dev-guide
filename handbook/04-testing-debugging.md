## 4. Testing & debugging

- **tyhpdef packages:** [tyhp-runtime-src](https://github.com/tyhpproject/tyhp-runtime-src) `packages/test-all-tyhpdef.sh` lints each `tyhpdef/*` package `tests/` tree (positives must be clean). Expected-error cases live in `tests/fail/*.tyhp` with a sidecar `tests/fail/<stem>.expect.json` (numeric diagnostic codes; not in-source comments). Runs one job per package/version (`--jobs`, default CPU count; `-j 1` streams a single package). `--package` limits the run to one path (default: every package). A package that runs longer than 90 seconds (`--timeout`), including `composer update`, fails. `--dry-run`, `--fail-fast`. Skips `core` / `async` / `lambda` / `decimal` / `compiler`.
- ⚠️ **No `tyhp test`.** Test **emitted PHP** (PHPUnit, Pest, `php`). Same namespaces/class names.
- **Compiler emit-and-run vs golden PHP:** `tests/conformance/storyNN/` compares emitted PHP to
  committed goldens and does not execute it. `tests/conformance/emit-and-run/` compiles fixtures
  the checker accepts (0 errors; listed warnings only), runs
  `php -d error_reporting=-1 -d display_errors=1`, and fails on non-zero exit or stderr matching
  Warning / Notice / Deprecated / Fatal / Parse error. Checker-error fixtures in that tree must
  not reach PHP. The corpus is curated (beaten path), not randomly generated.
- **Sourcemaps:** `build.generateSourcemap: true` writes `*.php.map` next to each `.php` (Source Map
  v3 + `//# sourceMappingURL=`). Debug `.tyhp` through the proxy; or read `build/` PHP directly
  (`output.comments: true` helps). Map write failures warn and do not fail the build.
- **`tyhp xdebug_proxy`:** IDE listens on `--ide-port` (9003); Xdebug connects to `--xdebug-port`
  (9004). Translates paths/lines via maps; files without maps pass through. Xdebug `client_port`
  must be **9004**. `--ide-key` filters sessions. Missing maps: warning, passthrough.
- **`tyhp language_server`:** stdio JSON-RPC (diagnostics, completion, hover, definition, references,
  rename, highlights, symbols, signature help, folding, code actions, format, semantic tokens,
  workspace symbols). ⚠️ `--tcp` / named pipe are not implemented.
- **`tyhp symbol_tree`:** parse+bind (no check/emit); JSON of the live environment (builtins,
  tyhpdefs, overlays, sources). `--filter=<substring>` on Tyhp name / FQN / PHP emit name;
  `--out=<file.json>`. Status on stderr. Confirm overlay `partial function` / `partial class` keep / rename here.
- **Editors:** `tyhp-lang/vscode/` (VSIX `tyhp-lang.tyhp`) and `tyhp-lang/phpstorm/` (ZIP
  `com.tyhp.lang`). Language id `tyhp` for `.tyhp` and `.tyhpdef`. Settings: `tyhp.path`,
  `tyhp.projectPath`, `tyhp.xdebugProxy.idePort`. Sideload the VSIX / “Install Plugin from Disk”.
  Marketplace publish is out of scope.
- **Verify lowering:** read `build/` PHP. Mapping table: [guide §29](../guide/29-php-mapping.md).

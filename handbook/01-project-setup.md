## 1. Project setup (`tyhp init` works)

`tyhp init [dir]` scaffolds `tyhp.json`, `src/`, `build/`, `tyhpdef/`, and `src/index.tyhp`. Only
template `basic` exists. Refuses if `tyhp.json` is already there. `--yes` skips prompts.
`--namespace=App\\` writes `psr4`. `--php-version=8.3` sets `output.phpVersion`.

A project is `tyhp.json` plus sources. **You must set `include`** — empty/missing include builds
nothing.

```json
{
  "include": ["src/**/*.tyhp"],
  "exclude": ["vendor/**", "build/**"],
  "output":  { "path": "build/", "phpVersion": "8.4", "strictTypes": true, "comments": true }
}
```

**Layout (convention):** `src/` → `build/` compiled PHP → `tyhpdef/` hand-written or generated
declarations.

**Namespace → folder:** emitter writes `{output.path}/<Namespace/Segments>/<Class>.php` from the
FQN (`\App\Models\User` → `build/App/Models/User.php`). ⚠️ `psr4` does **not** remap output folders
— Composer autoload only ([§2](02-autoloading.md)). Namespace functions → `_functions.php`;
non-class files mirror source.

**Mixing file types:** `.tyhp` and `.php` can share a project. A `.php` file under `include` is
parsed and re-emitted. `.tyhpdef` is types only, never emitted.

| Key | Default | Effect |
|-----|---------|--------|
| `include` / `exclude` | *(none)* | Source globs. Set `include`. |
| `suppressWarnings` | `[]` | Warning codes to hide (`"TYHP8027"`). Errors are never hidden. |
| `type` | `"application"` | `"library"` **always** emits `package.tyhpdef` and additive-merges `extra.tyhp.package` on publish-directory `composer.json`. |
| `output.path` | `"build/"` | Emit target. |
| `output.publishPath` | `"."` | Package root for `composer.json` / `package.tyhpdef` (libraries merge `extra.tyhp.package` here). |
| `output.publishClean` | `false` | Wipe `publishPath` before compile (refused for project root). |
| `output.publishContent` | `[]` | Extra files copied into `publishPath`. |
| `output.namespacePrefix` | `null` | Prepended to emitted namespaces **and** path segments. |
| `output.phpVersion` | *(unset → `"8.2"` + `TYHP4306`)* | `"8.0"`–`"8.5"`. Gates + lowering. Unsupported explicit → `TYHP6006` and `"8.4"`. |
| `output.strictTypes` | `true` | `declare(strict_types=1);` |
| `output.comments` | `true` | Generated-by header. |
| `psr4` | `null` | Composer autoload only. |
| `build.updateComposer` | `false` | Write `composer.json` in `output.publishPath` + `composer install` ([§2](02-autoloading.md)). |
| `build.entryPointAutoloader` | `null` | Inject `require_once` into entry files. |
| `build.generateSourcemap` | `false` | Write `*.php.map` beside PHP. |
| `build.generateTyhpdef` | apps: false | Apps only; libraries ignore (always emit). |
| `build.experimentalReadonlyCloneWith` | `false` | PHP 8.2–8.4 readonly `clone … with`. |
| `psr4Includes` | `null` | ⚠️ parsed, no effect. |

`tyhp init` writes `config.allow-plugins.tyhp/core: true`. `tyhp composer` / `sync` / `build --fix` also write it when missing (never overwrites explicit `false`). A raw `composer` still needs the key on disk.

**Libraries — `extra.tyhp.require`:** object of Composer package → constraint (any name, not
`tyhpdef/*` only). Ambient for **consumers**; duplicate those pins in this package’s `require-dev`.
Author-only tyhpdefs stay `require-dev` only (never extras) — `tyhp/decimal` keeps bcmath/gmp/
php-decimal wrappers there. `tyhp build` spells extras as FQNs and emits name-only `extern` for
require-dev-only owners. `internal` is omitted from `package.tyhpdef`.

First `composer require tyhp/core` often misses ambient tyhpdefs in that transaction. Run
`composer update` or `tyhp composer sync`. Default `tyhp build`/`lint` explain (`TYHP7701`);
`--strict` fails; `tyhp build --fix` / `tyhp composer sync` writes; ordinary build never
auto-updates. `--no-dev` does not rewrite `composer.json`. Details: [§3](03-build-cli-workflow.md).

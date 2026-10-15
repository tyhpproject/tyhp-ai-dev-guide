## 14. Deterministic disposal: `using` / `:=`

Resource must be a class implementing `\Tyhp\Contracts\IsDisposable` (`dispose(): void`), or `AsyncIsDisposable`
for `using await`. `:=`, `using` and `using await` reject any other type (TYHP4030); `using await` outside
async code is TYHP4028.
```tyhp
using (FileHandle $h = new FileHandle()) { /* disposed on exit */ }
using ($a = openA(), $b = openB()) { /* disposed reverse order */ }
using await ($c = openAsync()) { /* inside async code; awaits disposeAsync() */ }
$conn := new DbConnection();   // `:=` = disposable local, disposed when its enclosing block ends
```
`:=` is block-scoped: at function level it is disposed when the function returns; inside an `if`/loop block it is
disposed when that block ends (each loop iteration, and on `return`/`break`/exception), before outer resources.
`using` → try/finally:
```php
$h = new FileHandle();
try { /* body */ } finally { if ($h instanceof \Tyhp\Contracts\IsDisposable) { $h->dispose(); } }
```
`using await` → the same try/finally with `_await($c->disposeAsync())` for `AsyncIsDisposable` (else `dispose()`).
Function-level `:=` → `$__scope_N = \Tyhp\DisposableScope::create(); $conn = $__scope_N->using(new DbConnection());`.
A `:=` in a nested block → the same try/finally as `using`, around the rest of that block.
`:=` variables that reference each other (`$a->peer = $b; $b->peer = $a;`) get warning TYHP4212 and compile to try/finally instead.

**Failure semantics (know these):**
- Disposal order is **reverse** (LIFO).
- Calling `dispose()` yourself on a `:=` variable disposes it twice.
- A `dispose()` that throws in a `using` finally **masks the body exception** (PHP `finally`).
- Multiple resources whose disposals throw → thrown as `\Tyhp\Exceptions\AggregateException`
  (`getInnerExceptions()`); it collects **disposal** errors only — the body exception is not merged.
- Function-level `:=` disposal runs via the scope's `__destruct`, which only emits a warning on failure (does **not**
  rethrow) — so prefer `using` when you need dispose errors to propagate.

## 13. async / await (Fiber/Promise-based, not native PHP; pkg `tyhp/async`)

```tyhp
async function fetchCount(): int { int $v = await loadCountAsync(); return $v; }
```
→
```php
function fetchCount(): \Tyhp\Promise {
    return \Tyhp\Promise::_async(function (): int {
        return \Tyhp\Promise::_await(loadCountAsync());
    });
}
```
- `async` makes the outer return type `\Tyhp\Promise`; `await $x` → `\Tyhp\Promise::_await($x)`,
  valid inside `async` or at application entry-point top level (emitter wraps with `Promise::run`).
- **The declared return type of an async function, method, closure, arrow function, interface method, or
  tyhpdef signature is the settled type** (read as `__Awaited<R>`): `async function f(): int` is `int` when
  awaited, a call is `Promise<int>`. `async function f(): Promise<int>` is the same `int` function, not a
  promise of a promise. To pass promises around as values, wrap them (array/class) or use `Deferred`.
- **`__Awaited<T>`** unwraps `\Tyhp\Promise` (repeatedly, member-wise over unions) and never appears in
  emitted PHP. It stays deferred while `T` is open. `Promise<T>` requires `T extends __Awaited<void|mixed>`:
  `Promise<Promise<int>>`, a union containing a promise, and an unconstrained `T` in `Promise<T>` report
  TYHP4373. Write your own generic helpers as `function f<T extends __Awaited<void|mixed>>(Promise<T> $p)`.
- **No async generators** — yield `Promise<T>` from a `Generator` + `Promise::fromGenerator()`.
- `Promise<TReturn extends __Awaited<void|mixed> = __Awaited<mixed>>`. API (`\Tyhp`): `resolve resolved reject rejected withResolvers
  all race any allSettled whenAll whenAny delay timeout batch yield yieldTick fromGenerator run`
  (`run<T>(callable(): T, ?CancellationToken = null): __Awaited<T>` is the blocking sync entry; it returns the
  settled value and rethrows; `Promise::run(async fn(): int => …)` is `int` with no type argument) and
  `then catch finally continueWith`; plus `EventLoop`, `Deferred<T>`, `CancellationToken(Source)`,
  `DisposableScope`/`DisposableHelper`, `Contracts\AsyncIterator`/`AsyncIterable`/`AsyncKeyValueIterator`.
  Pass explicit type arguments to `resolve/resolved/reject/rejected` (`Promise::resolved<int>(1)`).
  `resolve<T>` does not accept a typed promise (use `resolve<mixed>`); `resolved<T>($p)` adopts a promise
  value and returns `Promise<__Awaited<T>>`.
  `then` has overloads: `then()`/`then(null, null)` → `Promise<TReturn>`; `then(null, callable(\Throwable): TReturn)` →
  `Promise<TReturn>`; `then(callable(TReturn): U, ?callable(\Throwable): U)` → `Promise<U>`. `catch(callable(\Throwable): TReturn)`
  keeps `Promise<TReturn>`.
  `Promise::withResolvers<T>()` gives a promise you settle yourself (use it from plain PHP too).
- `Promise::all` keeps keys and returns `array<T>`. An `async` closure has type `callable(…): Promise<T>`, so
  `\array_map(async fn(int $id): User => await $repo->find($id), $ids)`, `Promise::batch`, and a generic
  `callable(TIn): Promise<TOut>` parameter infer `T` / `TOut` from it; `fn(X $x): Promise<R> => …` is the same type.
  `fn(): void => expr` is a valid void callback (`->finally(fn(): void => cleanup())`).
- **Cancellation:** pass a `CancellationToken` to `run/all/race/any/delay/timeout/batch` (not
  `allSettled`). On cancel, the awaited call throws `\Tyhp\Exceptions\OperationCancelledException`;
  `timeout` throws `TimeoutException`. Use `$token->throwIfCancellationRequested()` in loops.
  `CancellationTokenSource`: `getToken()`, `cancel()`, `cancelAfter(ms)`.
  `Promise::run($fn, $token)` and `EventLoop::getInstance()->run($promise, $token)` check the token once per
  loop iteration. If it is cancelled while the root is pending, `run` throws `OperationCancelledException` and
  shuts the whole loop down: microtasks, fiber starts, timers, intervals, and stream watchers are cleared;
  never-started fibers' promises reject; suspended fibers get the exception thrown in (their `finally` blocks
  run). Every still-pending fiber-backed promise is rejected with `OperationCancelledException` — including a bare
  `Deferred`'s promise and `Promise::delay()`; a later `resolve()`/`reject()` on such a `Deferred` does not throw
  and has no effect. Already-settled promises keep their outcome. The loop stays reusable. If the root settled in the same iteration, its own outcome wins. A callback that never yields to the loop is not interrupted.
- A rejection surfaces only where the promise is awaited (or `run()`/`wait()` observes it). A rejected
  promise nobody awaits is silently discarded — there is no global handler — and unawaited work is
  abandoned if the root promise finishes first.
- `foreach (await $iter as $x)` desugars by operand type:
  - `AsyncIterable<T>` → `$__asyncIter_N = $iter->getAsyncIterator(); while (_await(…->next())) { $x = _await(…->current()); … }`
  - `Promise<Iterable<T>>` → `foreach (_await($iter) as $x)`
  - `Promise<AsyncIterable<T>>` → await then async-iterate; key-value uses `currentKey()`/`currentValue()`
    (needs `getAsyncIterator(): AsyncKeyValueIterator<K, V>` and a value statically typed as your class)
  - any class implementing `AsyncIterable<T>` works; iterating one without `await` is TYHP4046
- Application entry points with top-level `await` wrap in `\Tyhp\Promise::run(function() { … });`
  (library projects skip). Async closures/arrows emit as Promise-returning anonymous functions.
- Blocking PHP I/O (`\file_get_contents()`, PDO, curl without async wrappers) blocks the
  `tyhp/async` event loop; other fibers wait until the call returns.

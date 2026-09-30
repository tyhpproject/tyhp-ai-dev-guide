## 7. Runtime API reference (public signatures)

Signatures follow the intended public API (`.tyhpdef` of each package). PHP builtins come from
`tyhpdef/php`; scalar methods from `tyhp/core`; optional extension APIs from `tyhpdef/php-ext-*`.
Compile-time `#[\Tyhp\EraseGeneric]` (`#[\Tyhp\NoEmit]`) opts properties out of Mechanism C tracking.
`#[\Tyhp\GenericRuntime]` is a **runtime-visible** stamp on every emitted generic class, method, and
function (`erased`, `layouts: [1]`; `factory`/`binder` only when helpers exist). Tyhp **foreign**
compiled-library sites and hand-written PHP use `\Tyhp\Generic::bind(...)(...)`; same-compilation
Tyhp still emits factories/binders directly when tracked. `async`-marked
methods return `T` when awaited; the compiled call returns `\Tyhp\Promise<T>` until you `await` it. Callable
shapes are `callable(…): R`. `\Closure` is `\Closure<callable(...): mixed, TThis, TScope>` (class type arguments).

**`\Tyhp\Generic`** — PHP and foreign-library interop. `bind()` does not construct.
```
static bind(string|object|array|callable $target, ?Type ...$types): Generic
// then __invoke(...$args)  ≡  new(...$args)
// erased class → new $class(...); tracked class → named factory(types then ctor args)
// erased callable → Closure::fromCallable (by-ref preserved)
// missing GenericRuntime / unsupported layouts → InvalidTypeException
```

**`\Tyhp\Promise<TReturn extends void|mixed>`** — *`_async`/`_await` are compiler-emitted; never call
them by hand.*
```
static resolve<T>(T $value = null): Promise<T>
static reject<T>(Throwable $reason): Promise<T>
static withResolvers<T>(): Deferred<T>
static run<T>(callable(): T $fn, ?CancellationToken $token = null): T     // blocking sync entry point
async static delay(int $ms, ?CancellationToken $token = null): void
async static all<T>(array<Promise<T>> $ps, ?CancellationToken $token = null): array<T>
async static allSettled<T,K>(array<K, Promise<T>> $ps): array<K, PromiseSettledResult<T>>  // no cancellation; JS {status,value,reason}
async static race<T>(array<Promise<T>> $ps, ?CancellationToken $token = null): T
async static any<T>(array<Promise<T>> $ps, ?CancellationToken $token = null): T
async static timeout<T>(Promise<T> $p, int $ms, ?CancellationToken $token = null): T
async static batch<I,R>(array<I> $items, callable(I): Promise<R> $fn, int $concurrency = 5, ?CancellationToken $token = null): array<R>
async static fromGenerator(Generator $gen): mixed                      // no async generators
static whenAll(Promise ...$ps): static    // alias → all
static whenAny(Promise ...$ps): static    // alias → race
// instance:
async then<R>(?callable(TReturn): R $onOk = null, ?callable(Throwable): R $onErr = null): TReturn|R
async catch<R>(callable(Throwable): R $onErr): TReturn|R
async finally(callable(): void $onFinally): TReturn
wait(int $timeoutMs = -1): mixed          getResult(): mixed          getState(): PromiseState
isCompleted(): bool   isFulfilled(): bool   isFaulted(): bool   getError(): ?Throwable
// enum PromiseState: string { Pending | Fulfilled | Rejected }
```

**`\Tyhp\PromiseSettledResult<T extends void|mixed>`** — JS `Promise.allSettled` outcome: public `string $status` (`'fulfilled'` / `'rejected'`), `?T $value`, `?\Throwable $reason`.

**`\Tyhp\Deferred<T extends void|mixed>`**: `getPromise(): Promise<T>` · `resolve(T $v = null): void`
· `reject(Throwable $r): void`.

**`\Tyhp\CancellationToken`**: `static none(): CancellationToken` · `isCancellationRequested(): bool`
· `throwIfCancellationRequested(): void` · `register(callable $cb): Closure` (call the returned
closure to unregister). *(`cancel()` is internal — go through the source.)*

**`\Tyhp\CancellationTokenSource` (IsDisposable)**: `__construct(?int $cancelAfterMs = null)` ·
`getToken(): CancellationToken` · `cancel(): void` · `cancelAfter(int $ms): void` ·
`isCancellationRequested(): bool` · `dispose(): void`.

**`\Tyhp\DisposableScope` (IsDisposable)**: `static create(): DisposableScope` ·
`using(IsDisposable|AsyncIsDisposable $r): mixed` · `release(IsDisposable|AsyncIsDisposable $r): void`
· `dispose(): void`.

**Contracts**: `\Tyhp\Contracts\IsDisposable { dispose(): void }` ·
`AsyncIsDisposable { disposeAsync(): Promise<void> }` ·
`AsyncIterator<T> { next(): Promise<bool>; current(): Promise<T> }` ·
`AsyncIterable<T> { getAsyncIterator(): AsyncIterator<T> }`.

**`\Tyhp\Decimal`** — readonly `string $value`, `int $scale`, `int $roundingMode`.
```
__construct(float|int|string|DecimalConvertible|null $value = null, ?int $scale = null, int $roundingMode = PHP_ROUND_HALF_UP)
add/subtract/multiply/modulo/power(...)   divide(..., ?int $scale = null)   negate() abs() sqrt(?int $scale = null)
compareTo(...): int   equals/greaterThan/greaterThanOrEqual/lessThan/lessThanOrEqual(...)
isZero() isPositive() isNegative()   __toInt() __toFloat() withScale(int $scale)   round(int $p = 0, int $mode = …) floor() ceil()
format(int $decimals = 2, string $dec = '.', string $thou = ','): string   __toString(): string
static zero(int $scale = 2) one(int $scale = 2) min(...) max(...) sum(...) avg(...)
// helper: \Tyhp\decimal(float|int|string|DecimalConvertible|null $v = null): Decimal
// operators + - * / % ** and == != < <= >= <=> plus `convert` are defined
```

**`\Tyhp\Type` (Stringable)** — factories: `string() int() float() bool() null() void() mixed()
never() array() object() callable() iterable() resource()`, plus `union(self ...$t)`,
`intersection(self ...$t)`, `nullable(self $t)`, `generic(string $class, NamedType ...$p)`,
`struct(string $name, array $fields, array $requiredKeys = [])`,
`fromClassName(string $c)`, `of(mixed $v)`, `is(mixed $v, self $t): bool`,
`isType<T>(mixed $v): bool`, `check(mixed $v, self $t): void`,
`compatible(self $broad, self $narrow): bool`. Instance: `asReadOnly()/asNullable()/asNonNullable()`,
`getKind(): string`, `getName(): ?string`, `isNullable(): bool`, `isReadOnly(): bool`, `__toString()`.

**`\Tyhp\Json`**: `static decode<T>(string $json, int $depth = 512, int $flags = 0): T` — objects as
arrays, always `JSON_THROW_ON_ERROR`; shape-mismatch throws `IncompatibleTypeException`.
`static tryDecode<T>(string $json, T &$out, int $depth = 512, int $flags = 0): bool` — same rules;
writes `$out` and returns `true` on success, otherwise leaves `$out` unchanged and returns `false`.
String extensions `$json->jsonDecodeAs<T>()` / `$json->jsonTryDecodeAs<T>(T &$out): bool` wrap those
helpers. `\json_decode` itself is not generic.

**Exceptions**: `\Tyhp\Exceptions\AggregateException(array<Throwable> $inner, string $msg = …)` →
`getInnerExceptions(): array<Throwable>` · `OperationCancelledException(CancellationToken $t, …)` →
`getToken(): CancellationToken` · `TimeoutException(string $msg = …, int $timeoutMs = 0, …)` →
`getTimeoutMs(): int`.

**`\Tyhp\Expression<TCallableShape>` / `PropertyPath` — experimental** (end-to-end wiring
incomplete): `Expression` carries `->body`, `->parameters`, `->callable`, `->returnType`, is callable
via `__invoke(...)`, and `compile(): \Closure<TCallableShape>`. Write `Expression<callable(T): R>`.
Translate a captured tree with an `ExpressionVisitor`
(`visitBinary`, `visitPropertyAccess`, `visitMethodCall`, `visitConstant`, … per node type);
`ExpressionSerializer::toJson(Expression $e): string`. `PropertyPath` adds
`getPath()/getSegments()/getValue(T $s)`.

**`\Tyhp\EventLoop`** (low-level; you rarely touch it directly): `static getInstance()`,
`run(Promise $root): mixed`, `runUntilSettled(Promise $p, int $timeoutMs = -1)`, `delay(int $ms,
callable $cb): string` (+ `cancelTimer`, `interval`), `defer/queueMicrotask(callable $cb)`, stream
registration, `tick(): bool`, `stop()`, `isRunning(): bool`.

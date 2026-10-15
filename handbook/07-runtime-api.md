## 7. Runtime API reference (public signatures)

Signatures follow the intended public API (`.tyhpdef` of each package). PHP builtins come from
`tyhpdef/php`; scalar methods from `tyhp/core`; optional extension APIs from `tyhpdef/php-ext-*`.
Compile-time `#[\Tyhp\EraseGeneric]` (`#[\Tyhp\NoEmit]`) opts properties out of Mechanism C tracking.
`#[\Tyhp\GenericRuntime]` is a **runtime-visible** stamp on every emitted generic class, method, and
function (`erased`, `layouts: [1]`; `factory`/`binder` only when helpers exist). Tyhp **foreign**
compiled-library sites and hand-written PHP use `\Tyhp\Generic::bind(...)(...)`; same-compilation
Tyhp still emits factories/binders directly when tracked. `async`-marked
functions, methods, closures, and signatures declare the **settled** type `R` (read as `__Awaited<R>`); the compiled call
returns `\Tyhp\Promise<__Awaited<R>>` until you `await` it. Callable
shapes are `callable(…): R`. `\Closure` is `\Closure<callable(...): mixed, TThis, TScope>` (class type arguments).

**`\Tyhp\Generic`** — PHP and foreign-library interop. `bind()` does not construct.
```
static bind(mixed $target, ?Type ...$typeArguments): mixed
// class name → Generic binder; callable → \Closure taking the value arguments
// in Tyhp code the result is `mixed`: prefer `new Box<int>(…)` over `bind(Box::class, …)->new(…)`
// binder: new(mixed ...$args): object  ≡  __invoke(mixed ...$args): object   (named arguments forwarded)
// each binder constructs once: a second new()/call throws \LogicException
// erased class → new $class(...); tracked class → named factory(types then ctor args)
// erased callable → Closure::fromCallable (by-ref preserved)
// missing GenericRuntime / unsupported layouts → InvalidTypeException
static wrapCallable(\Closure $target, \Closure $factory): \Closure   // compiler-emitted
// $factory(fn(\Closure $target): \Closure) builds the converting wrapper; result ≡ $factory($target)
// bind() on the result binds $target and re-applies $factory (conversions survive); non-Closure factory result → \InvalidArgumentException
// bind() on any other anonymous Closure → \InvalidArgumentException
```

**Root functions** — plain-PHP spellings. Each is `fallback`, so an existing global of that name is left in place.
```
gnrc(mixed $target, ?Type ...$typeArguments): mixed   // tyhp/core. Same signature as Generic::bind(); does not construct
async<T extends void|mixed>(callable(): T $fn): Promise<__Awaited<T>>   // tyhp/async. Starts $fn on the event loop
await<T extends __Awaited<void|mixed>>(Promise<T> $promise): __Awaited<T>   // tyhp/async. Suspends until $promise settles; throws the rejection
asyncMain<T extends void|mixed>(callable(): T $fn, ?CancellationToken $token = null): __Awaited<T>   // tyhp/async. Sync entry into async; same signature as Promise::run()
```

**`\Tyhp\Promise<TReturn extends __Awaited<void|mixed> = __Awaited<mixed>>`** — *`_async`/`_await` are
compiler-emitted. Tyhp code uses the `async` / `await` keywords; plain PHP uses the root functions above.* `TReturn` is always a settled type (TYHP4373 for `Promise<Promise<T>>`
or an unconstrained `T`); declare your own generics as `T extends __Awaited<void|mixed>`.
```
static resolve<T extends __Awaited<void|mixed>>(?T $value = null): Promise<T>   // a Promise argument is returned unchanged (typed promise arguments: use resolve<mixed>)
static resolved<T extends void|mixed = void>(?T $value = null): Promise<__Awaited<T>>  // always a new, already-fulfilled promise; a Promise $value is adopted: resolved<Promise<int>>($p) is Promise<int>
static reject<T extends __Awaited<void|mixed> = void>(Throwable $reason): Promise<T>   // alias of rejected()
static rejected<T extends void|mixed = void>(Throwable $error): Promise<__Awaited<T>>
// pass the type argument explicitly: Promise::resolved<int>(1), Promise::rejected<string>($e)
static withResolvers<T extends void|mixed>(): Deferred<__Awaited<T>>   // promise settled from outside (also the way to get one from plain PHP)
static yield(): Promise<void>      static yieldTick(): Promise<void>   // cooperative yield / yield past the timer phase
static run<T extends void|mixed>(callable(): T $fn, ?CancellationToken $token = null): __Awaited<T>   // blocking sync entry point; returns the settled value, rethrows a rejection
// T is inferred: Promise::run(async fn(): int => await f()) is int; a cancelled token ends the whole loop (OperationCancelledException)
async static delay(int $ms, ?CancellationToken $token = null): void
async static all<T extends __Awaited<void|mixed> = mixed>(array<Promise<T>> $ps, ?CancellationToken $token = null): array<T>   // keys preserved; assign to array<T>, not array<int,T>
async static allSettled<T extends __Awaited<void|mixed> = mixed, K extends int|string = int|string>(array<K, Promise<T>> $ps): array<K, PromiseSettledResult<T>>  // no cancellation; JS {status,value,reason}
async static race<T extends __Awaited<void|mixed>>(array<Promise<T>> $ps, ?CancellationToken $token = null): __Awaited<T>
async static any<T extends __Awaited<void|mixed>>(array<Promise<T>> $ps, ?CancellationToken $token = null): __Awaited<T>
static timeout<T extends __Awaited<void|mixed>>(Promise<T> $p, int $ms, ?CancellationToken $token = null): Promise<__Awaited<T>>   // rejects TimeoutException; does not stop the operation
async static batch<I, R extends __Awaited<void|mixed>>(array<I> $items, callable(I): Promise<R> $fn, int $concurrency = 5, ?CancellationToken $token = null): array<int, R>
// batch/array_map callbacks: write fn(I $x): Promise<R> => $svc->load($x), so R is inferred from the declared Promise<R>
async static fromGenerator(Generator $gen): mixed                      // no async generators
static whenAll<T extends __Awaited<void|mixed> = mixed>(Promise<T> ...$ps): Promise<array<int|string, T>>   // variadic form of all()
static whenAny<T extends __Awaited<void|mixed>>(Promise<T> ...$ps): Promise<__Awaited<T>>                // variadic form of race()
// instance (each returns a new Promise); `then` is an overload set, the first matching signature wins:
async then(null $onOk = null, null $onErr = null): TReturn
async then(null $onOk, callable(Throwable): TReturn $onErr): TReturn
async then<R>(callable(TReturn): R $onOk, ?callable(Throwable): R $onErr = null): R
catch(callable(Throwable): TReturn $onErr): Promise<TReturn>
async finally(callable(): void|mixed $onFinally): TReturn               // a void callback may be fn(): void => cleanup()
async continueWith<R>(callable(?TReturn, ?Throwable): R $k): R
wait(int $timeoutMs = -1): mixed          getResult(): TReturn        getState(): PromiseState
isCompleted(): bool   isFulfilled(): bool   isFaulted(): bool   getError(): ?Throwable
// enum PromiseState: string { Pending | Fulfilled | Rejected }
```

**`\Tyhp\PromiseSettledResult<T extends void|mixed>`** — JS `Promise.allSettled` outcome: public `string $status` (`'fulfilled'` / `'rejected'`), `?T $value`, `?\Throwable $reason`.

**`\Tyhp\Deferred<T extends __Awaited<void|mixed> = void>`**: `getPromise(): Promise<T>` · `resolve(?T $v = null): void`
· `reject(Throwable $r): void`.

**`\Tyhp\CancellationToken`**: `static none(): CancellationToken` · `isCancellationRequested(): bool`
· `throwIfCancellationRequested(): void` · `register(callable $cb): Closure` (call the returned
closure to unregister). *(A token cannot be cancelled directly — cancel through its source.)*

**`\Tyhp\CancellationTokenSource` (IsDisposable)**: `__construct(?int $cancelAfterMs = null)` ·
`getToken(): CancellationToken` · `cancel(): void` · `cancelAfter(int $ms): void` ·
`isCancellationRequested(): bool` · `dispose(): void`.

**`\Tyhp\DisposableScope` (IsDisposable)**: `static create(): DisposableScope` ·
`using(IsDisposable|AsyncIsDisposable $r): mixed` · `release(IsDisposable|AsyncIsDisposable $r): void`
· `dispose(): void`. The compiler creates one for `:=` (disposed in reverse order when the function returns; a `:=` in a nested block is disposed when that block ends).

**`\Tyhp\DisposableHelper`**: `async static dispose(IsDisposable|AsyncIsDisposable ...$disposables): void`
(reverse order; several failures throw `AggregateException`).

**Contracts**: `\Tyhp\Contracts\IsDisposable { dispose(): void }` ·
`AsyncIsDisposable { disposeAsync(): Promise<void> }` ·
`AsyncIterator<T extends __Awaited<void|mixed>> { next(): Promise<bool>; current(): Promise<T> }` ·
`AsyncIterable<T extends __Awaited<void|mixed>> { getAsyncIterator(): AsyncIterator<T> }` ·
`AsyncKeyValueIterator<K, V> (both extends __Awaited<void|mixed>) extends AsyncIterator<V> { currentKey(): Promise<K>; currentValue(): Promise<V> }`
(for `foreach (await $src as $k => $v)`: declare `getAsyncIterator(): AsyncKeyValueIterator<K, V>` and iterate a value typed as your class).

**`\Tyhp\Decimal`** (`decimal`; `final`, immutable) — readonly `string $value`, `int $scale`,
`int $roundingMode`. **In Tyhp code use operators; call the methods below only where no operator exists.**
```
__construct(float|int|string|DecimalConvertible|null $value = null, ?int $scale = null, int $roundingMode = PHP_ROUND_HALF_UP)
add/subtract/multiply/modulo/power(...)   divide(..., ?int $scale = null)   negate() abs() sqrt(?int $scale = null)
compareTo(...): int   equals/greaterThan/greaterThanOrEqual/lessThan/lessThanOrEqual(...)
isZero() isPositive() isNegative()   __toInt() __toFloat() __toDecimal() withScale(int $scale)   round(int $p = 0, int $mode = …) floor() ceil()
format(int $decimals = 2, string $dec = '.', string $thou = ','): string   __toString(): string   jsonSerialize(): string
static zero(int $scale = 2) one(int $scale = 2) min(self ...$v) max(self ...$v) sum(self ...$v) avg(self ...$v)
static getBackendName(): string   setBackend(\Tyhp\Backend\DecimalBackend $b): void
// helper: \Tyhp\decimal(float|int|string|DecimalConvertible|null $v = null): Decimal   (new Decimal(null, 2) is 0.00)
// operand params take \Tyhp\DecimalCoercible = float|int|string|DecimalConvertible
// operators (prefer these): + - * / % ** (int exponent), unary - and + (+ gives float),
//   == != < <= > >= <=>, compound forms (+= …); other operand may be decimal/string/int/float on either side;
//   the right operand may also be a float|int|string union, a \Tyhp\DecimalCoercible, or a DecimalConvertible object
//   (a class declaring `operator convert(self): decimal` qualifies) (** needs decimal base); results decimal/bool/int. `(int)`/`(float)`/`(string)` casts work on a decimal;
//   `(decimal) $x` accepts string/int/float only -> \Tyhp\Decimal::__from($x)
// string operands keep their written scale; arithmetic truncates to the larger operand scale
// (divide(..., $scale) overrides); negative scale/precision throws \ValueError; ÷0 throws \DivisionByZeroError
```

**`\Tyhp\Type` (Stringable)** — factories: `string() int() float() bool() null() void() mixed()
never() array() object() callable() iterable() resource()`, plus `union(self ...$t)`,
`intersection(self ...$t)`, `nullable(self $t)`, `generic(string $class, NamedType ...$p)`,
`struct(string $name, array $fields, array $requiredKeys = [])`,
`objectShape(string $name, array $methods = [], array $properties = [])`, `callableShape(string $name)`,
`fromClassName(string $c)`, `fromReflectionType(\ReflectionType $t, ?string $relativeClassName = null)`,
`fromCallableReturn(callable $c): ?self`, `of(mixed $v)`, `is(mixed $v, self $t): bool`,
`isType<T>(mixed $v): $value is T` (type guard), `check(mixed $v, self $t): void`,
`compatible(self $broad, self $narrow): bool`, `awaited(self $t): self` (runtime form of `__Awaited<T>`: unwraps
`Promise` layers, member-wise over unions; a bare `Promise` becomes `mixed`). Instance: `asReadOnly()/asNullable()/asNonNullable()`,
`defaultValue(): mixed` (what `default(T)` produces), `genericParameter(string $name): ?NamedType`,
`getKind(): string` (`scalar|class|union|intersection|generic|struct|objectShape|callableShape|named`),
`getName(): ?string`, `isNullable(): bool`, `isReadOnly(): bool`, `__toString()`.
`union`/`intersection` throw `InvalidTypeException` with fewer than two distinct members.

**`\Tyhp\NamedType` (extends `Type`)** — `__construct(string $parameterName, Type $underlyingType, bool $readOnly = false)` ·
`getParameterName(): string` · `getUnderlyingType(): Type` · `asReadOnly(): self`; matching and
`__toString()` delegate to the underlying type. It is the argument type of `Type::generic()`.

**`\Tyhp\Json`**: `static decode<T>(string $json, int $depth = 512, int $flags = 0): T` — objects as
arrays, always `JSON_THROW_ON_ERROR`; invalid JSON throws `\JsonException`; a value that is not an
object, has a wrongly typed field, or lacks a required field (neither nullable nor defaulted) throws
`IncompatibleTypeException`. Extra keys are kept; defaults are not applied to absent fields.
`static tryDecode<T>(string $json, T &$out, int $depth = 512, int $flags = 0): bool` — same rules;
writes `$out` and returns `true` on success, otherwise leaves `$out` unchanged and returns `false`.
String extensions `$json->jsonDecodeAs<T>()` / `$json->jsonTryDecodeAs<T>(T &$out): bool` wrap those
helpers. `\json_decode` itself is not generic.

**`\Tyhp\ObjectHelper`** (the helper behind `with`): `static with<TProperties extends struct, T extends object&TProperties>(T $object, TProperties $properties): T`
assigns each entry to the same-named property of `$object` and returns the same instance (no clone);
unknown key → `PropertyNotFoundException`; readonly properties cannot be assigned. Compiled `with`
expressions call it for `new X() with [...]` and, below PHP 8.5, `clone $x with [...]`.

**`\Tyhp\StringHelper`** — mbstring-or-byte switch used by the string methods: `hasMbStringExt(): bool`,
`strlen`, `substr`, `strpos`, `strrpos`, `stripos`, `strtolower`, `strtoupper`, `strSplit`, `ord`,
`ucWords`, `convertEncoding`, `detectEncoding`; `strPad` (PHP ≥ 8.3), `ucFirst` / `lcFirst` (PHP ≥ 8.4).

**Conversion contracts** (`\Tyhp\Contracts`): `Convertible` (marker) · `IntConvertible { __toInt(): int }` ·
`FloatConvertible { __toFloat(): float }` · `BoolConvertible { __toBool(): bool }` ·
`StringConvertible` (`\Stringable`). Declaring `operator convert(self $v): int|float|bool|string|decimal`
adds the interface and method and rewrites explicit casts, call arguments, and returns to the call;
an interface implemented by hand gets none of those rewrites. `ArrayAccessShape<TStruct>` is the per-key
`\ArrayAccess` contract for struct schemas.

**Core exceptions** (`\Tyhp\Exceptions`, all `final`): `IncompatibleTypeException(Type $expected, Type $actual, ?string $variableOrParameterName = null, ?\Throwable $previous = null)`
→ `getExpectedType()` · `getActualType()` · `getVariableOrParameterName()` ·
`InvalidTypeException` (`\LogicException`: bad union/intersection, `Generic::bind` on a non-generic) ·
`PropertyNotFoundException(string $propertyName, string $className)` → `getPropertyName()` · `getClassName()` ·
`InvalidParametersForOperatorOverloadException(string $className, string $methodName, mixed ...$operands)` →
`getClassName()` · `getMethodName()`.

**Exceptions**: `\Tyhp\Exceptions\AggregateException(array<Throwable> $inner, string $msg = …)` →
`getInnerExceptions(): array<Throwable>` · `OperationCancelledException(CancellationToken $t, …)` →
`getToken(): CancellationToken` · `TimeoutException(string $msg = …, int $timeoutMs = 0, …)` →
`getTimeoutMs(): int`.

**`\Tyhp\Expression<TCallableShape>` / `PropertyPath<TCallableShape>`** (`tyhp/lambda`): an inline `fn`
passed to a parameter typed `Expression<callable(T): R>` (or `PropertyPath<callable(T): R>` for a plain
`$x->a->b` chain) arrives as a node tree. `Expression` carries `->body` (an `ExpressionNode`),
`->parameters`, `->callable`, `->returnType` (a type string), and `->lastResult`; call it like a function
(`$expr($arg)`, also accepted by PHP `callable` parameters such as `\array_filter`), `compile():
\Closure<TCallableShape>` (a `\Closure` parameter accepts an `Expression` argument directly, via `->callable`),
`equals(Expression $other): bool`.
`Expression::of(User::class, 'a', 'b')` builds paths for PHP callers, with `and()`/`or()` and static
`andAll()`/`orAll()` (arguments typed `Expression<callable(mixed): bool>`). Node classes
(`Tyhp\Expression\*`): `ParameterExpression`, `PropertyAccessExpression`, `NullSafeAccessExpression`,
`MethodCallExpression`, `StaticMethodCallExpression`, `BinaryExpression`, `UnaryExpression`,
`ConstantExpression`, `TernaryExpression`, `CoalesceExpression` (`??`), `ArrayAccessExpression`,
`CastExpression`, `NewExpression`, `InstanceofExpression`. Translate a tree with an `ExpressionVisitor`
(`visit($node): mixed`, one `visitBinary` / `visitPropertyAccess` / `visitConstant` / … per node type;
defaults return `null` and do not recurse); `ExpressionSerializer::toJson(Expression $e): string` and
`::equals($a, $b): bool`. `PropertyPath` adds `getPropertyName(): string`, `getPath(): string`
(`'address.city'`), `getSegments(): array<string>`, and `getValue(mixed $source)`.
An `fn` body must be one expression made of: property/null-safe property access, method and static
calls, binary/unary operators, casts, ternary/`?:`/`??`, array access, `new`, `instanceof`/`is`,
int/float/string literals (no interpolation), `true`/`false`/`null`, `Cls::CONST`, and captured
variables. Anything else (function calls, assignments, `match`, `await`, array literals, first-class
callables, shell commands, string interpolation, …) is **TYHP4322**, which names the construct; non-`fn` arguments are **TYHP4320** (`PropertyPath`) / **TYHP4323**
(`Expression`); an `Expression` of another callable shape is **TYHP4010**.

**`\Tyhp\Parallel` / `ParallelTask<T>`** (`tyhp/parallel`) — process pool, not fibers.
A CLI program calls `start()` before opening database connections and hosts the pool
in that process (`ext-pcntl`, `ext-posix`). A php-fpm app runs
`vendor/bin/tyhp-parallel serve --socket=<path> --workers=4` as the php-fpm user
and sets `TYHP_PARALLEL_SOCKET` to that path. The socket is mode 0660 unless
`--mode=<octal>` says otherwise. `--socket` defaults to `TYHP_PARALLEL_SOCKET`, and
the optional `--bootstrap=<file>` is loaded after the autoloader.
`runAsync()`, `forAsync()`, `forEachAsync()`, and `invokeAsync()` are the same calls in both
modes, and need no type arguments: `T` is inferred from the task class
(`int $n = await Parallel::runAsync(new CountTask())`); `forEachAsync<string, int>(…)` may
name some and infer the rest. At a synchronous entry point wrap the call in `Promise::run()`. The
service accepts many php-fpm workers at once and shares one pool. A task is a serializable
class the service can autoload; its `run()` returns a serializable value. The task is
serialized when it is queued, so later changes to the object do not reach the worker. The
result must match `T` or the Promise rejects. `forAsync` treats `$toExclusive` as exclusive.
`forEachAsync` and `invokeAsync` keep the input keys. `shutdown()` on a client closes that
process's connection and leaves the service running.

Every call takes an optional `CancellationToken`. Cancelling removes a queued task, or
kills the worker running it and starts a replacement; the Promise rejects with
`OperationCancelledException`. `forAsync`, `forEachAsync`, and `invokeAsync` fail fast: the
first failure cancels the other tasks and is thrown. A php-fpm request that ends with tasks
still running cancels them on the service.
```
static start(int $workers = 4): void
static serve(string $socketPath, int $workers = 4, int $socketMode = 0660): void
static shutdown(): void
static isAvailable(): bool   // TYHP_PARALLEL_SOCKET set, or CLI with pcntl + posix; false inside a worker
static runAsync<T extends __Awaited<void|mixed>>(ParallelTask<T> $task, ?CancellationToken $token = null): Promise<T>
async static forAsync<T extends __Awaited<void|mixed>>(int $fromInclusive, int $toExclusive, callable(int): ParallelTask<T> $factory, ?CancellationToken $token = null): array<int, T>
async static forEachAsync<TItem, TResult extends __Awaited<void|mixed>, TKey extends int|string = int|string>(array<TKey, TItem> $items, callable(TItem): ParallelTask<TResult> $factory, ?CancellationToken $token = null): array<TKey, TResult>
async static invokeAsync<T extends __Awaited<void|mixed>, TKey extends int|string = int|string>(array<TKey, ParallelTask<T>> $tasks, ?CancellationToken $token = null): array<TKey, T>
interface ParallelTask<T extends void|mixed = mixed> { run(): T }
type ParallelRemoteError = struct { string $class; string $message; int $code; string $file; int $line; string $trace; }
```
`ParallelException::getRemoteError(): ?ParallelRemoteError` describes the exception thrown
in the worker (null when the worker exited or the connection failed instead);
`getRemoteClass()` is its class name. `ParallelUnavailableException` means this process
cannot submit a task (also thrown by every `Parallel` method inside a worker). A serialized
task and a serialized result are each limited to 16 MiB; closures, anonymous classes, and
resources (which arrive as `0`) cannot be task properties.

**`tyhp/compiler`** — `vendor/bin/tyhp` is a PHP shim that forwards every argument, the
working directory, the environment, the streams, and the exit code to the native compiler.
The `post-autoload-dump` entry `vendor/bin/tyhp --install-binary` (written by `tyhp init`)
downloads the compiler for the release tag `v<package version>` from GitHub, verifies its
SHA-256, and stores it in `vendor/tyhp/compiler/native/`. Ordinary commands never download
or compare versions; if the binary is missing the shim tells you to run `--install-binary`.
`GITHUB_TOKEN` / `GH_TOKEN` raise the GitHub API rate limit. Use `composer install --no-dev
--no-scripts` for production, because the dump hook calls the dev-only `vendor/bin/tyhp`.

**`\Tyhp\Io\*`** (`tyhp/io`) — non-blocking I/O on the `tyhp/async` event loop. Every method
whose name ends in `Async` returns a `Promise` (or is `async`), so `await` it inside `async`
code, or combine several with `Promise::all`. The same naming applies to `tyhp/parallel`. Every call takes an optional trailing `?CancellationToken`; cancelling aborts
the work (kills a process, detaches an HTTP transfer, sends `pg_cancel_query` / `KILL QUERY`)
and rejects with `OperationCancelledException`. Timeouts reject with `TimeoutException`.
Errors use `\Tyhp\Io\Exceptions\*`: `IoException` (base), `IoUnavailableException` (missing
extension), `HttpException`, `HttpStatusException` (`getResponse()`), `ProcessFailedException`
(`getResult()`), `DbException` (`getSqlState()`), `RedisException`.
```
// Http — ext-curl. Only http:// and https:// (redirects included). Every status resolves.
new HttpClient(array<string, string> $defaultHeaders = []) · static shared(): HttpClient
sendAsync(HttpRequest $r): Promise<HttpResponse> · getAsync(string $url, array<string, string> $headers = []): Promise<HttpResponse>
postAsync(string $url, string $body, array<string, string> $headers = []) · postJsonAsync(string $url, mixed $payload, array<string, string> $headers = [])
async getJsonAsync<T extends struct>(string $url, array<string, string> $headers = []): T   // requires 2xx; adds Accept: application/json unless $headers has an Accept (any case)
type HttpRequest = struct { string $url; string $method = 'GET'; array<string, string> $headers; string $body;
    int $timeoutMs = 30000; int $connectTimeoutMs = 10000; bool $followRedirects = true; int $maxRedirects = 5; bool $verifyTls = true; }
HttpResponse: readonly int $status, array<string, array<int, string>> $headers (lower-case names), string $body, string $url
    isSuccess(): bool · header(string $name): ?string · headerValues(string $name): array<int, string> · json<T extends struct>(): T · ensureSuccess(): HttpResponse

// Process — argument list, no shell. Resolves whatever the exit code; mustRunAsync throws on non-zero.
static Process::runAsync(array<int, string> $command, ?ProcessOptions $o = null): Promise<ProcessResult> · async static mustRunAsync(...): ProcessResult
type ProcessOptions = struct { ?string $cwd; ?array<string, string> $env; string $input; int $timeoutMs = 0; }
type ProcessResult = struct { array<int, string> $command; int $exitCode; int $signal; string $stdout; string $stderr; }   // signalled: exitCode = 128 + signal

// File — chunked, yields to other fibers between chunks.
async static File::readAsync(string $path): string · readJsonAsync<T extends struct>(string $path): T · writeAsync/appendAsync(string $path, string $contents): void
async static File::copyAsync(string $from, string $to): void · hashAsync(string $path, string $algorithm = 'sha256'): string
static File::linesAsync(string $path): AsyncIterable<string>   // foreach (await File::linesAsync($path) as $line); line endings removed

// Socket — tcp://host:port, tls://host:port (verifies host), unix:///path/to/socket
async static Socket::connectAsync(string $address, int $timeoutMs = 10000, ?CancellationToken $token = null): SocketConnection
SocketConnection: async readAsync(int $maxBytes = 65536): string ('' at EOF) · async readLineAsync(int $maxBytes = 1048576): ?string (null at EOF)
    async readExactlyAsync(int $length): string · writeAsync(string $data): Promise<void> (writes stay ordered) · close() · isClosed()

// Redis — RESP2, pipelined: commands sent before earlier replies arrive.
async static RedisClient::connectAsync(?RedisOptions $o = null): RedisClient
type RedisOptions = struct { string $host = '127.0.0.1'; int $port = 6379; bool $tls; ?string $username; ?string $password; int $database; int $connectTimeoutMs = 10000; }
commandAsync(string $name, string|int|float ...$arguments): Promise<mixed>   // error replies reject with RedisException
pingAsync · getAsync · setAsync(string $key, string $value, int $ttlMs = 0) · delAsync · existsAsync · incrAsync(string $key, int $by = 1) · expireAsync
    hSetAsync(string $key, array<string, string>) · hGetAsync · hGetAllAsync · lPushAsync · rPushAsync · lRangeAsync · publishAsync · close() · isClosed()

// Db — one statement at a time per connection (calls queue). Placeholders: $1, $2, … for Postgres; ? for MySQL and PDO.
interface DbConnection { queryAsync(string $sql, array<int, DbValue> $params = []): Promise<DbResult>; transactionAsync(array<int, DbStatement> $s): Promise<array<int, DbResult>>; close(): void; }
type DbValue = int|float|string|bool|null · type DbStatement = struct { string $sql; array<int, DbValue> $params; }
async static PostgresConnection::connectAsync(string $connectionString): PostgresConnection   // ext-pgsql, async libpq
async static MySqlConnection::connectAsync(?MySqlOptions $o = null): MySqlConnection          // ext-mysqli, MYSQLI_ASYNC
type MySqlOptions = struct { string $host = '127.0.0.1'; int $port = 3306; string $user = 'root'; string $password; string $database; string $charset = 'utf8mb4'; ?string $socket; int $connectTimeoutSeconds = 10; }   // socket is used only when host is 'localhost'
new PdoConnection(PdoOptions $o)   // any PDO driver, via Offload
type PdoOptions = struct { string $dsn; ?string $username; ?string $password; array<int, int|string|bool> $attributes; }
DbResult: readonly array<int, array<string, mixed>> $rows, int $affectedRows, ?string $lastInsertId
    count() · first(): ?array<string, mixed> · scalar(): mixed · rowsAs<T extends struct>(): array<int, T> · firstAs<T extends struct>(): ?T

// Offload / Dns — blocking work in a tyhp/parallel worker when available, otherwise inline.
interface OffloadJob<T extends void|mixed = mixed> { run(): T }
async static Offload::runAsync<T>(OffloadJob<T> $job, ?CancellationToken $token = null): T   // T inferred: await Offload::runAsync(new MyJob())
static Offload::usesWorkers(): bool
static Dns::resolveAsync(string $host, ?CancellationToken $token = null): Promise<array<int, string>>   // IPv4 and IPv6 addresses
```
PostgreSQL `int2`/`int4`/`int8`/`oid` columns arrive as `int`, `float4`/`float8` as `float`, `bool`
as `bool`; MySQL integer and floating-point columns arrive as `int` or `float` (`BOOLEAN` is
`TINYINT`, so `int`); PDO rows are what the driver returns. Other columns (including `numeric` and
`DECIMAL`) are strings and SQL NULL is `null`. A `transactionAsync()`
rolls back and rethrows when any statement fails. An `OffloadJob` follows the `ParallelTask` rules
when workers are used: it must be serializable and autoloadable by the worker, and its result
must be serializable. Without workers (no `tyhp/parallel`, or `Parallel::isAvailable()` is
false), `Offload`, `PdoConnection`, and `Dns` block the event loop while they run. With workers, a
failure inside the job (including a `Dns` lookup that resolves nothing) rejects with
`ParallelException`; without workers `Dns` rejects with `IoException` and `Offload` rethrows what the
job threw.

**`\Tyhp\EventLoop`** (low-level; you rarely touch it directly): `static getInstance()`, `static resetInstance()`,
`run<T extends __Awaited<void|mixed> = mixed>(Promise<T> $root, ?CancellationToken $token = null): T` (cancelling the
token while the root is pending shuts the loop down and throws `OperationCancelledException`), `runUntilSettled(Promise $p, int $timeoutMs = -1)`, `delay(int $ms,
callable $cb): string` (+ `cancelTimer`, `interval`), `defer/queueMicrotask(callable $cb)`, stream
registration, `tick(): bool`, `stop()`, `isRunning(): bool`.

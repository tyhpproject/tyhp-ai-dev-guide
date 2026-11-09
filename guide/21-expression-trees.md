## 21. Expression trees (parsable lambdas) — marquee feature

Pass an **inline `fn`** to a param typed `Expression<callable(...)>` / `PropertyPath<callable(T): R>`
and the compiler captures a runtime AST of the lambda (not just a closure). Libraries translate the
tree, e.g. → SQL (like C# `Expression<Func<…>>` / LINQ-to-SQL).
```tyhp
class QueryBuilder<T> {
    public function where(Expression<callable(T): bool> $predicate): static { return $this; }
    public function select(Expression<callable(T): string> $selector): static { return $this; }
}
$q = new QueryBuilder<User>()
    ->where(fn ($u) => $u->age > $minAge)
    ->select(fn ($u) => $u->firstName);
```
- One type argument: the callable shape. `Expression<callable(User): string>` takes `User`, returns
  `string`. Zero-arg: `Expression<callable(): R>`. `\Closure` on `->callable` / `compile()` is `\Closure<TCallableShape>`.
- Only an **inline `fn`** (or an existing `Expression` of the same shape) converts — not a stored
  closure variable or `function(){}` (**TYHP4320** `PropertyPath` / **TYHP4323** `Expression`); the param
  type decides tree (`Expression<>`) vs plain closure (`callable`/`\Closure`). The `fn` body is a single
  expression: function calls, assignments, `await`/`yield`/`match`/`throw`, `function () {}` closures,
  array spread (`...$xs`), first-class callables, shell commands and string interpolation are rejected with
  **TYHP4322** (the message names the construct, e.g. `array spread`). So are a class held in a variable
  (`new $cls()`, `$cls::make()`, `$x::class` → `dynamic class`), a variable member name (`$x->$m()` →
  `dynamic member`), and named or unpacked call arguments (`named argument`, `argument unpacking`); name
  the class (`Cls::make()`, `new Cls()`) and pass arguments positionally. String literals, `true`/`false`/`null`,
  `Cls::CONST`, captured variables, `instanceof`/`is`, `new`, casts, `?:`/`??`, list and keyed array
  literals (`['id' => $u->id, 'tags' => [$u->a, $u->b]]`), and `?->` on properties and method calls are fine.
  A `?->` that meets `null` skips the rest of that chain, as in PHP.
- An extension method call stays a method call in the tree (`$title->startsWith($prefix)` is a
  `MethodCallExpression` whose method is `startsWith`). The extension body is not substituted into
  the tree. `compile()` still runs that body.
- A nested arrow `fn` is part of the tree: `fn ($b) => $b->posts->any(fn ($p) => $p->rating > $min)`.
  Its parameter types come from the callee parameter (`callable(Post): bool`, `PropertyPath<callable(Tag): string>`,
  or the callable member of `Expression<callable(Post): bool>|(callable(Post): bool)`). Its body follows the same rules and may read the
  outer parameters and captures; a nested parameter shadows an outer one of the same name. Nested
  parameters cannot be by-reference, variadic or have defaults (**TYHP4322**). Nodes: `LambdaExpression`
  (evaluates to a `\Closure`), `ArrayExpression` of `ArrayItem`s, `NullSafeMethodCallExpression`.
- Every `Expression` also carries `->callable` (and `__invoke`, so `$expr($arg)` runs it). It can be passed
  to a PHP `callable` parameter or a Tyhp `callable(T): R` parameter of the same shape, and any `Expression`
  argument to a `\Closure` parameter is converted with `->callable` for you. `compile()` returns the closure.
  For an `Expression<callable(T): R>` or `PropertyPath<callable(T): R>` parameter (alone or in a union with
  `callable(T): R`), an inline `fn` infers a generic `R` from its body; a `?->` chain gives a nullable `R`.
  `PropertyPath<callable(T): R>` is the property-chain subset (`$x->a->b`, including `?->`). Pkg `tyhp/lambda`.
- `tyhp build` emits a fluent chain for an inline `fn`: `\Tyhp\Expr::param(...)->…->backedBy(fn)` or
  `\Tyhp\PropertyPath::on(...)->…->backedBy(fn)`. The file is the same whether or not the `tyhp` module is
  loaded. A closure variable in `.tyhp` is still **TYHP4323** / **TYHP4320**.
- A parameter typed `Expression` or `PropertyPath` is emitted as `\Tyhp\Expression|\Closure` or
  `\Tyhp\PropertyPath|\Closure`. A nullable parameter keeps `|null`. The value body calls
  `Expression::fromClosure` or `PropertyPath::fromClosure` when the argument is a `\Closure`. A promoted
  constructor parameter, a by-reference or variadic parameter, a property-hook parameter, an inline closure's
  own parameter, a union with a type other than `null`, and `#[\Tyhp\PhpType]` keep the spelled type.
- PHP in a package, and any PHP whose later process you do not control, builds the tree with `_expr` /
  `_path` / `Expression::of` / `PropertyPath::on`. That fluent form is the recommended one for speed and
  stability whether or not `ext-tyhp` is installed. A caller who knows the process has the module loaded may
  pass a closure. That is the easier PHP spelling in that case. The gate is `extension_loaded('tyhp')` once
  per process, then `phpversion('tyhp') >= 0.1.0`. A missing or older module is ignored, with no warning, and
  `fromClosure` tokenizes the closure source. The module path calls `\Tyhp\Ext::closureTree`. A closure the
  reader cannot read throws `\InvalidArgumentException` on both paths, including a closure with a return type
  PHP checks (`: bool`) or a `use` clause. Write an arrow function, or a `function` whose body is one `return`,
  with no declared return type. `PropertyPath::fromClosure` needs one parameter and a property chain.
- Parameter and property nodes keep the full declared type in `->type`, as a `\Tyhp\Type::parse()`
  spelling: `fn (Table<Blog> $t) => …` has parameter type `App\Table<App\Blog>`, and `$b->posts` on an
  `array<Post>` property has type `array<App\Post>` (read through `\Tyhp\Type::fromProperty()`).
- Captured variables, literals and `Cls::CONST` are `ConstantExpression` nodes. `ExpressionSerializer::shapeKey($e)`
  keys the tree without their values (cache a translation per shape) and `shapeParameters($e)` lists the
  values in written order → `handbook/07-runtime-api.md`.

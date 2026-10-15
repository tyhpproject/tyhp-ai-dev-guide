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
  expression: function calls, assignments, nested `fn`, `await`/`yield`/`match`/`throw`, array literals,
  first-class callables, shell commands and string interpolation are rejected with **TYHP4322** (the message
  names the construct, e.g. `array literal`). String literals, `true`/`false`/`null`, `Cls::CONST`,
  captured variables, `instanceof`/`is`, `new`, casts, `?:`/`??` are fine.
- Every `Expression` also carries `->callable` (and `__invoke`, so `$expr($arg)` runs it). It can be passed
  to a PHP `callable` parameter or a Tyhp `callable(T): R` parameter of the same shape, and any `Expression`
  argument to a `\Closure` parameter is converted with `->callable` for you. `compile()` returns the closure.
  For an `Expression<callable(T): R>` parameter, an inline `fn` infers a generic `R` from its body.
  `PropertyPath<callable(T): R>` is the property-chain subset (`$x->a->b`, including `?->`). Pkg `tyhp/lambda`.

## 21. Expression trees (parsable lambdas) — marquee feature

Pass an **inline `fn`** to a param typed `Expression<callable(...)>` / `PropertyPath<callable(T): R>`
and the compiler captures a runtime AST of the lambda (not just a closure). Libraries translate the
tree, e.g. → SQL (like C# `Expression<Func<…>>` / LINQ-to-SQL).
```tyhp
class QueryBuilder<T> {
    public function where(Expression<callable(T): bool> $predicate): static { return $this; }
    public function select<R>(Expression<callable(T): R> $selector): static { return $this; }
}
$q = new QueryBuilder<User>()
    ->where(fn ($u) => $u->age > $minAge)
    ->select(fn ($u) => $u->firstName);
```
- One type argument: the callable shape. `Expression<callable(User): string>` takes `User`, returns
  `string`. Zero-arg: `Expression<callable(): R>`. `\Closure` on `->callable` / `compile()` is `\Closure<TCallableShape>`.
- Only **inline `fn`** converts (not a stored var or `function(){}`); the param type decides tree
  (`Expression<>`) vs plain closure (`callable`/`\Closure`). Nested `fn`, statement-bodies,
  `await`/`yield`/`match`/`throw` in the tree are rejected.
- Every `Expression` also carries `->callable` (and `__invoke`), so it stays executable.
  `PropertyPath<callable(T): R>` is the property-chain subset (`$x->a->b`, including `?->`). Pkg `tyhp/lambda`.
  Still being wired up — treat end-to-end use as experimental.

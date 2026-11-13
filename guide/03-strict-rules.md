## 3. Two rules that break PHP habits

**Non-nullable by default** — add `?`/`|null` explicitly. Absence is `?T` assigned `null`.
`unset` of a local, a typed instance property, or a static property is TYHP4158.
Array-offset `unset` stays legal. Initialization rules are in [§8](08-typed-locals.md) and
[§22](22-declarations.md).
```tyhp
string $n = null;    // ERROR
?string $n = null;   // OK
```
A possibly null value must be narrowed before `->` property access, method calls, or struct/object
indexing (TYHP4198):
```tyhp
?User $user = findUser($id);
$user->name;                           // ERROR (TYHP4198)
$user?->name ?? '';                    // OK
if ($user !== null) { $user->name; }   // OK
```
A `?->` chain's value is nullable when its receiver may be null (`$user?->address` is
`?Address`); a `?->` that meets `null` skips the rest of that chain, so later links
(`$user?->address->city`) do not need another `?->`.
**Conditions must be `bool`** (no truthiness):
```tyhp
if ($user) {}          // ERROR (TYHP4043)
if ($user !== null) {} // OK  (also: $count !== 0, $list !== [])
```
Narrowing ([§16](16-type-guards.md)) resets on reassignment.

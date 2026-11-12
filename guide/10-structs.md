## 10. Structs

Structural value type = "typed associative array": **typed properties only, no methods**, → PHP
`array`. PHP callers see `array`; there is no runtime shape check.

```tyhp
type Point = struct { int $x = 0; int $y = 0; ?string $label; };   // nullable prop may omit default
type ColoredPoint = struct extends Point { string $color = 'red'; };
```

| | `struct` | `class` |
|-|----------|---------|
| identity | value (array) | object ref |
| compatibility | **structural** — target's props ⊆ source's, types assignable | **nominal** (inheritance) |
| members | typed properties only | full OO |
| rule | non-nullable props need a default | — |

Construct a named, generic, or anonymous struct with a bracket list. Keys are property names, bare or
quoted (`x =>`, `'x' =>`), including a keyword (`class =>`), not positions. A field with an alias is
still named by its property (`strVal =>` for `'String Value' as $strVal`). An entry with no key is an
error (TYHP4413). A trailing comma is allowed, and a newline may sit between the name (or the anonymous
struct's `}`) and `[`.

```tyhp
Point $p = new Point[x => 1, y => 2];   // → ['x'=>1,'y'=>2]
$p->x                                    // → $p['x']
new Point[]                              // → ['x'=>0,'y'=>0]
(new Point[x => 1, y => 2])->x
```

`new Point[]` is `new Point()`. `new Point[x => 1, y => 2]` is `new Point() with [x => 1, y => 2]`.
`new Point[x => 1] with [y => 2]` is an error (TYHP4417); put every entry in the one list. Dereference a
bracket list with parentheses: `(new Point[x => 1, y => 2])->x`.

```tyhp
type Box<T> = struct { T $value; };
new Box<int>[value => 1]
Box<int> $b = new Box[value => 1];       // type argument comes from the target type
new struct { string $name; int $age; }[name => 'Alice', age => 30]
```

Spread copies one struct into another. Later entries override:

```tyhp
type Opts = struct { string $name; int $retries = 3; };
new Opts[...$base, name => 'bob']        // → \array_replace($base, ['name' => 'bob'])
```

The source is a named struct, a generic struct instance, or an anonymous struct. `array` (including
`array<K, V>`), `mixed`, a nullable struct, or a union is TYHP4414. Every source field must exist on
the target, including through `extends` (TYHP4415), and its type must be assignable to the target
field. The PHP array keys must match, so spread cannot rename an alias (`'status_code' as $statusCode`,
`0 as $arg1`) (TYHP4416). A required field is satisfied by a keyed entry or by a spread whose source
requires or defaults that field. A nullable source field with no default does not (TYHP4159). Defaults
apply first; entries then apply left to right, and a later entry or spread wins. A spread's field types
are checked against the generic struct's type arguments the same way a keyed value is, so
`Box<int> $b = new Box[...$stringBox]` is a type error. A bare `new Box[…]` takes its type arguments from
the target type. A struct
`with` list (`new Name() with`, `clone $s with`, `$s with`) follows these same rules.

Emit a spread as `\array_replace`: remaining defaults, then each spread source and each run of keyed
entries, in source order. That is not PHP `[...$src]`, which renumbers integer keys.

Structs are always plain PHP associative arrays at runtime. Nested struct defaults
(`Inner $inner = new Inner[];`) fold into the outer array literal.

Anonymous `struct { … }` types (and `new struct {…}[…]`) support the same operations as named structs:
property access and writes, `with`, `clone … with`, `isset`, `??`, and `as` key aliases. `clone` is a
no-op for a struct; `clone … with` uses `\array_replace`. A statement `$p with […];` (variable or
property target) emits `$p = \array_replace($p, …);` — a spread is its own argument.

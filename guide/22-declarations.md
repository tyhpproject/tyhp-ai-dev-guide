## 22. Declarations — deltas vs PHP

Classes, interfaces, traits, enums, visibility, and members are **the same as PHP** except:

- **Declaration-site generics** on `class`/`interface`/`trait`/`enum`/`function`/method names:
  `class Box<T> {}`, `interface Query<T> {}`, `trait Timestamped<T> {}`, `function map<T,U>(…)`.
  Generic args allowed on `extends`/`implements` (`extends Base<int>`).
- **Constructor return type is optional:** `public function __construct(…): void {}` (or omit
  `: void` — both mean no `parent::__construct(...)` insertion). Use the base-call form
  `: parent(<args>)` to insert `parent::__construct(<args>)` at the start of the body.
- **`async`** is a function/method modifier ([§13](13-async-await.md)).
- **Return type guards** `: $x is T` ([§16](16-type-guards.md)).
- **Overload signatures** (declaration only, no body) exist at top level:
  `function area(int $r): float; function area(int $w, int $h): float;`.
- **Trait property alias** adds a Tyhp form renaming a *property*: `use T { $prop as $renamed; }`
  (PHP only aliases methods).
- Typed properties, constructor promotion, `abstract`/`final`/`readonly`/`static`,
  `&`-return, enum backing types/cases, trait `insteadof`/`as` (methods): **identical to PHP**.
- **Class-level `const`:** a type is required on the original declaration (PHP 8.3 typed class
  constants). A child redeclaration may omit the type and inherits it invariantly from that
  ancestor; changing the type is an error.
- **File-level `const`:** Tyhp source is untyped (`const X = 1;`), matching PHP. `.tyhpdef`
  file-level constants are typed (`const int X;`) because they describe PHP constants that may
  have no exact value.
- **`internal`** — omit from published `package.tyhpdef` (members emit `public`; top-level unprefixed). Cannot combine with `public`/`protected`/`private` (`TYHP4002`).
- **Short `fn` methods:** `public fn add(int $a, int $b): int => $a + $b;` (expression body). Named
  `fn` at file scope is the same idea. PHP `fn($x) => …` closures are unchanged.
- **`global use` / `global use function` / `global use const` / `global use extension`** — compilation-wide
  imports (C# `global using`). A local `use` of the same symbol is redundant (`TYHP4169`).
- **PHP version gates** — `declare(php="…")` and `#[\Tyhp\Php("…")]`; never written to PHP, but a gate true for only some versions at or above `output.phpVersion` emits a `\PHP_VERSION_ID` check
  ([§30](30-php-version-gating.md)).
- **`#[\Tyhp\PhpType('mixed')]`** — replaces the emitted PHP type of a parameter, return, property,
  or typed constant (stripped on emit; checker types unchanged). Illegal on class / enum case /
  catch / local / property hook (`TYHP4327`). Not `#[\Tyhp\Php]`.
- **No nested named functions/methods:** declaring a named `function` inside another function or
  method's body is a compile error (`TYHP4802`) — use a private method or a closure instead.
  Closures/arrow functions are unaffected.
- **Traversable:** user classes/enums/interfaces/traits list `\Iterator` or `\IteratorAggregate`,
  not `\Traversable` (`TYHP4326`).
- **Enums:** every enum is `\UnitEnum` (backed enums also `\BackedEnum`) without a written
  `implements`; `cases()` / `from()` / `tryFrom()` resolve automatically. User classes,
  interfaces, traits, and enums must not list `\UnitEnum` / `\BackedEnum` (`TYHP4337`), and enums
  must not redeclare `cases`, `from`, or `tryFrom` (`TYHP4338`).

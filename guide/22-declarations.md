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
- **Overload signatures** (declaration only, no body) for functions and for class/interface/abstract
  methods (incl. `async`; not constructors/destructors, `TYHP3036`):
  `function area(int $r): float; function area(int $w, int $h): float;` followed **directly** by one
  implementation (`function`, or `fn … => expr;`; no other declaration in between, `TYHP3032`; none at all
  is `TYHP3031` except interface/abstract sets). Rules:
  - Callers see only the declared signatures; the implementation's own signature is not callable.
  - Selection is two-pass. Pass 1: the **first** declared signature whose arguments are assignable
    **as written** wins (declaration order); a conversion-only signature declared
    earlier never beats it. Pass 2, only if pass 1 found nothing: signatures are retried allowing the
    implicit `operator convert` an ordinary call applies (class-owned and extension converts,
    convert-to-scalar such as `takesInt($celsius)` → `__toInt()`; `operator convert hide` respected).
    If several accept only through conversion: a signature converting a strict subset of another's
    arguments (to the same types) dominates it; among the undominated, equal conversion profiles → first
    declared; different profiles → `TYHP4375` (ambiguous; an explicit conversion/cast picks one). No
    signature accepts even with conversions → `TYHP4374` listing the candidates. The emitted call applies
    the conversion (`takeMoney(5)` → `takeMoney(\App\Money::__from(5))`); PHP keeps one function.
    Put narrow signatures first.
  - Every signature must be compatible with the implementation (`TYHP4118`): its parameters assignable
    to the implementation's, return types assignable in either direction. An override of an `abstract`
    set is checked against every signature.
  - All signatures share modifiers (`static`, visibility, `async`, `abstract`, `final`; `TYHP3035`);
    identical signatures are `TYHP3034`; a single signature is warning `TYHP3033`.
  - `async` applies per signature: each declared `R` is read as `__Awaited<R>`.
  - A reference such as `area(...)`, `$o->m(...)`, or an overloaded `__invoke` is a callable with one
    facet per declared signature (`TYHP4374` if none accepts). The value is widened like any first-class
    callable (`$fn = takeMoney(...); $fn(5)` converts through `operator convert`), decided per parameter
    position: a type some signature accepts as written is never converted; another type converts to its
    own signature's target; one type converting to different targets is `TYHP4376` at the `(...)`.
    Assigning to `callable(...)` needs one satisfying facet, and a variable declared `callable(Money): string`
    keeps exact types (`$f(5)` is `TYHP4010`). `$o->__invoke(5)` / `$o(5)` are method calls and follow
    the two-pass rule.
  - **First-class callables and conversions:** `f(...)`, `$o->m(...)`, `C::m(...)`, `$o(...)` (the
    object's `__invoke`) and extension-method callables whose callee has a convertible parameter become a
    wrapper closure accepting the widened types (declared type + convertible sources) and converting
    before the call; `\array_map($fn, [5, 6])` works. No convertible parameter: plain native callable.
    By-reference, variadic and generic-typed parameters are not widened; closures (`fn`/`function`) and
    `callable(...)`-typed variables keep exact types. The wrapper keeps the callee's return type (except
    `static`/`self`/`parent`), adds one frame and reflects as an anonymous closure. A value matching no
    convertible source is passed on and the callee raises `\TypeError`. A generic callee is wrapped with
    `\Tyhp\Generic::wrapCallable`, so `Generic::bind` on it keeps the conversions.
  - Only the implementation is emitted (interface/abstract: the last signature). Published tyhpdef
    lists the declared signatures. Editor hover/signature help show the declared signatures.
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

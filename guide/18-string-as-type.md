## 18. String-as-type features (advanced; all erase to `string`)

**Symbol-name types** — typed strings naming real symbols: `__ClassName __EnumName __InterfaceName
__TraitName __FunctionName __StructName __ConstName __VarName __TypedVarName<T> __PropertyName<T>
__MethodName<T> __ObjectConstName<T> __EnumCaseName<T> __CompatibleTypeName<T> __SuperTypeName<T>`.
They narrow through existence checks and verify literal assignment at compile time:
```tyhp
if (\class_exists($n)) { /* $n: __ClassName */ }
if (\method_exists($o, $n)) { /* $n: __MethodName<typeof($o)> */ }
__ClassName $c = 'App\\User';   // errors if class unknown
__PropertyName<User> $prop = nameof(User::$name);   // also a matching literal `'name'`
__ClassName<User> $user = User::class;
function id<T>(): __ClassName<T> { return T::class; }
```
**Template string types** — types describing sets of strings, written as a double-quoted string in
type position; `${T}` = hole (any string-valued type, including a named alias — imported,
namespace-qualified, `self\Alias`, or generic `Alias<...>`); quantifier right after `}` (`?`=0-1,
`+`=1+, `*`=0+, `{n} {n,} {,m} {n,m}`); canonical `${T}+` not `${T+}`. A hole naming a template
alias includes its pattern, so templates compose; an unknown hole name is an error, and so is a hole
of a type with no string form (`array`, `bool`, `mixed`, a class or enum) or one written as an
expression (`${$x}`, a call, an inline nested template — give it an alias instead):
```tyhp
type ApiMethod = 'GET'|'POST';
type ApiPath   = "api/${string}/items";
type Endpoint  = "${ApiMethod} ${ApiPath}";
```
A matching literal (`'GET api/widgets/items'`) or a double-quoted interpolation whose text and holes
fit the pattern is assignable. A `${string}` hole accepts any interpolated value (an `int` too) and
may span several of them; a narrower hole (`${ApiMethod}`) needs a value of that type. A value that may
be `null` interpolates as `''` when null, so it fits a narrower hole only if that hole accepts `''`
(`${''|int}`), and a `${string}` hole only together with other text:
```tyhp
function itemsPath(string $segment): ApiPath { return "api/{$segment}/items"; }
function itemsEndpoint(ApiMethod $m, string $segment): Endpoint { return "{$m} api/{$segment}/items"; }
```
Numeric holes accept the text PHP's `(string)` cast produces: `${int}` takes `'3'`, `'-12'`, `'0'`
(not `'03'`, `'+3'`, `'-0'`, `'3.0'`); `${float}` takes `'1.5'`, `'3'`, `'1.0E+25'` (not `'1.50'`,
`'.5'`, `'1.5e3'`); `${1|2}` takes `'1'` or `'2'`. An interpolated `int` / `float` value fits too:
```tyhp
type ItemCount = "${int} items";
function itemsOf(int $n): ItemCount { return "{$n} items"; }
```

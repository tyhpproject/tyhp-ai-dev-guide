## 9. Type aliases

```tyhp
type UserId = int;
type Optional<T = mixed> = T|null;
class UserService {
    public type NameType = string;
}

function f(UserId $id): void {}                 // hint → int
\Tyhp\Type $t = typeof(UserId);                 // UserId()
\Tyhp\Type $o = typeof(Optional<int>);          // Optional(\Tyhp\Type::int())
\Tyhp\Type $o2 = Optional(typeof(int));         // same
UserService::NameType();
// Optional<int>()                              // error — factory is not a generic function
```

Hints expand to the underlying PHP type. Source `.tyhp` aliases also emit a `\Tyhp\Type` factory
(`function UserId(): \Tyhp\Type` / `public static function NameType()`). Call the factory with
`\Tyhp\Type` values (`UserId()`, `Optional()`, `Optional(typeof(int))`); `<>` is type-position only.
`use App\Types\UserId;` stays class-kind in Tyhp; emitted PHP is `use function` when the factory is
used by short name (hints-only still pruned). Tyhpdef aliases without
`#[\Tyhp\GenericRuntime(aliasFactory: …)]` stay check-only (`typeof` inlines the body). A compiled
library stamps `aliasFactory` when it emitted the factory; consumers call `DecimalCoercible()` /
`UserService::NameType()`. Class/method stamps are independent (`erased: true` is still written).
Transparent: the checker expands; `UserId()` returns the body's Type, not a distinct identity.
File-level aliases occupy the function namespace (`type Foo` + `function Foo()` → `TYHP3028`).
Type-position `self\Alias` / `static\Alias` / `parent\Alias` / `ClassName\Alias` (and the same
names on `is` / `instanceof`) bind to that class-level alias when the prefix names a class that
declares or inherits it. Otherwise the name is the nested type. PHP may have both a class and a
namespace at the same FQN (`class \Ns\Foo` plus `\Ns\Foo\Bar`); inherited members still resolve
from that parent.

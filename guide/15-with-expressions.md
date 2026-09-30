## 15. `with` expressions (record-style update)

```tyhp
Config $copy = clone $base with [enabled => false, name => 'copy'];   // new object
Point  $p    = new Point() with [x => 0, y => 0];                     // construct + override
$user with [name => 'Eve'];                                            // in-place mutate
```

Every key must be a real property; each value must fit its type. In-place `$obj with […]` is
rejected on `readonly` (`TYHP4141`) — use `clone … with` or `new … with`.

**Structs** → array replace (`\array_replace` / a new array literal) — rebinds the array, same as
passing an array into a PHP function.

**Classes** compile. In-place `$obj with […]` mutates identity, same as passing an object into a
PHP function. Target `output.phpVersion` ≥ `8.5` → native `clone($obj, […])`. On 8.2–8.4,
non-readonly uses a helper around `clone`; **readonly** `clone … with` needs
`build.experimentalReadonlyCloneWith: true` (`TYHP4139` if off). `new … with` on readonly does not
need that flag. `final` + readonly `clone … with` on PHP < 8.5 is `TYHP4140`.

## 20. Property accessors (= PHP 8.4 property hooks)

```tyhp
class Temperature {
    private float $celsius = 0.0;
    public float $fahrenheit = 32.0 {
        get => ($this->celsius * 9 / 5) + 32;
        set(float $value) { $this->celsius = ($value - 32) * 5 / 9; }
    }
}
```

Write them in `.tyhp` exactly like PHP 8.4+ property hooks (`get` / `set` / `&get`, `final`, hook
visibility). On `output.phpVersion` ≥ 8.4 the compiler emits native hooks. On 8.2–8.3 it lowers the
same source through a polyfill. `&get` requires 8.4 (`TYHP4167`).

### `.tyhpdef`

Describe a hooked PHP property with a **bodyless** list. Bodies stay in `.tyhp`.

```tyhpdef
<?tyhpdef
class Holder {
    public string $hooked { get; set; }
    public int $hookedCount { get; }
    public array $refItems { &get; set; }
    public string $name { get; private set; }
}
```

One `$name` per hooked declaration. `#[\Tyhp\Php]` goes on the property or `declare(php=…)`, not on
`get`/`set` (`TYHP8016`).

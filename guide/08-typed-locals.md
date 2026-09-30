## 8. Typed locals (type erased)

```tyhp
$sum = 0;                 // inferred — primary form → $sum = 0;
int $total = $sum;        // explicit when inference is not enough
?string $label;
array<int> $nums = [1,2,3];
for ($i = 0; $i < $n; $i++) {}
```
Only params, properties, and return types keep a spelled PHP type. Locals lead with `$var = …`;
`Type $var = …` stays legal when there is no initializer or you need a wider type.

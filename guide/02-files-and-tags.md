## 2. Files & open tags

| File | Contents | Tag | Result |
|------|----------|-----|--------|
| `.tyhp` | Tyhp code | `<?tyhp ` | → `.php` |
| `.tyhpdef` | type declarations for external PHP (like `.d.ts`) | `<?tyhpdef ` | consumed, not emitted |
| `.php` | plain PHP | `<?php` | used as-is |

Open tag needs trailing whitespace. `?>` closes a Tyhp section; HTML between sections is emitted
as written, including indentation in the source before `<?tyhp`. The pretty-printer does not reindent
that HTML. One newline directly after `?>` is swallowed, same as PHP.

**`<?= expr ?>` in a `.tyhp` file** is a Tyhp echo: parsed and checked as Tyhp, emitted as `echo`,
no escaping. `is`, `typeof`, `with`, and `nameof` work inside it. A comma list is one `echo`.
Non-stringable → `TYHP4120`; a missing member → `TYHP4101`; the diagnostic is on that `.tyhp` line.
`.php` `<?=` is an unchanged PHP echo.

Loops and conditionals are `<?tyhp` blocks. `if (…):` / `endif;` and `foreach (…):` / `endforeach;`
parse and check; emit lowers them to braces. A template is a function or method you call
(`include` / `require` are `TYHP4801`). Capture with `\ob_start()` / `\ob_get_clean()`.

```tyhp
<?tyhp
function userCard(Person $user): void { ?>
    <h1><?= $user->name ?></h1>
<?tyhp }
```

**Tagless mode** (`source.tagless: true`): the open tag may be omitted (the extension picks the
language). A tagless `.tyhp` file is code with no open tag, and `?>` is an error, so a template
must be tagged. A `.tyhp` file whose first 6 characters are exactly `<?tyhp` is parsed as a normal
tagged file instead (a template: the first section may be empty, `<?tyhp ?>` or `<?tyhp?>`, then
HTML, `<?=`, and a later `<?tyhp`). A blank line or other text before `<?tyhp` keeps the tagless
entry, which does not accept `<?tyhp`. Default off.

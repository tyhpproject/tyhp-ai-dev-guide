## 2. Files & open tags

| File | Contents | Tag | Result |
|------|----------|-----|--------|
| `.tyhp` | Tyhp code | `<?tyhp ` | → `.php` |
| `.tyhpdef` | type declarations for external PHP (like `.d.ts`) | `<?tyhpdef ` | consumed, not emitted |
| `.php` | plain PHP | `<?php` | used as-is |

Open tag needs trailing whitespace. `?>`/inline HTML echo like PHP. **Tagless mode**
(`source.tagless: true`): the open tag may be omitted (the extension picks the language). A tagless
`.tyhp` file is code with no open tag, and `?>` is an error. A `.tyhp` file whose first 6 characters
are exactly `<?tyhp` is parsed as a normal tagged file instead (a template: the first section may be
empty, `<?tyhp ?>` or `<?tyhp?>`, then HTML, `<?=`, and a later `<?tyhp`). A blank line or other text
before `<?tyhp` keeps the tagless entry, which does not accept `<?tyhp`. Default off.

# Contributing to the Tyhp AI development guide

Issues and pull requests are welcome. Read this before opening either.

## Legal

You will need to complete a Contributor License Agreement (CLA). Briefly, this agreement testifies that you are granting us permission to use the submitted change according to the terms of the project's license, and that the work being submitted is under appropriate copyright. Upon submitting a pull request, you will automatically be given instructions on how to sign the CLA.

This guide is under the [Apache License 2.0](LICENSE.txt).

## What this repository is

Clone [tyhp-ai-dev-guide](https://github.com/tyhpproject/tyhp-ai-dev-guide). The files at the repository root (`SKILL.md`, `guide/`, `handbook/`) are the guide. There is no build step and no test suite.

The language is implemented in [tyhp](https://github.com/tyhpproject/tyhp). Runtime package sources are in [tyhp-runtime-src](https://github.com/tyhpproject/tyhp-runtime-src). The documentation site is [tyhplang.com](https://tyhplang.com).

## Editing the guide

Change the section file an agent would read, and the index lines that point at it (`QUICK_GUIDE.md`, `SKILL.md`, and the folder `00-index.md`). A correction that lives only in a section file is lost the next time the guide is regenerated, so update the matching item in `REGEN.md` as well.

`REGEN.md` is for maintainers. Regenerating the guide needs a compiler checkout and a tyhp-runtime-src checkout, as sibling clones of this repository. End users do not run it.

## Pull requests

- Target `main`.
- Keep changes focused. A section edit should not include an unrelated handbook rewrite.
- Do not translate or reformat Tyhp/PHP code, keywords, type names, or identifiers in examples. They are literal syntax.

## Code of conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

# Tyhp AI development guide

This repository teaches an AI coding agent to write and maintain **Tyhp** application code. Tyhp is a typed superset of the PHP language. The agent already knows PHP; these files cover only what is different.

The compiler, CLI, and language server live in [tyhp](https://github.com/tyhpproject/tyhp). Runtime package sources live in [tyhp-runtime-src](https://github.com/tyhpproject/tyhp-runtime-src). Human documentation is at [tyhplang.com](https://tyhplang.com) (source: [tyhp-docs-src](https://github.com/tyhpproject/tyhp-docs-src)).

Licensed under the [Apache License 2.0](LICENSE.txt).

## Clone

Clone this repository on its own:

```bash
git clone https://github.com/tyhpproject/tyhp-ai-dev-guide.git
```

## Layout

| Path | What it is |
|------|-----------|
| `AGENTS.md` | Entry point for agents. Explains the docs and the few PHP habit-breakers. |
| `CLAUDE.md` | Same entry point for Claude-based tools (points to `AGENTS.md`). |
| `SKILL.md` | Cursor Agent Skill entry point. Same compact index as `QUICK_GUIDE.md`, plus YAML frontmatter and a cherry-pick list of every `guide/` / `handbook/` topic. The repository root is the skill root. |
| `QUICK_GUIDE.md` | One-line-per-feature **index** with cross-language analogies (C#/TS/…) and `→` pointers to the exact section file. ~1.3k tokens — cheap enough to keep always-on. |
| `guide/` | The core language guide, **one file per section** (`01-mental-model.md` … `30-php-version-gating.md`), plus `00-index.md`. The PHP→Tyhp delta. |
| `handbook/` | Setup / autoloading / CLI / interop / testing / examples / runtime-API, **one file per section** (`01`…`07`), plus `00-index.md`. |
| `README.md` | This file. |
| `REGEN.md` | Maintainer-only: prompt used to regenerate this repository when the language changes. End users don't run it. |

**Reading model:** an agent starts from `SKILL.md`, `QUICK_GUIDE.md`, or `AGENTS.md`, then opens
**only** the one section file it needs via the `→` pointer. The `00-index.md` in each folder (or a
plain folder listing) shows everything available, so the agent never has to grep or load an entire
document.

## Install it in your Tyhp project

### Option A — Cursor Agent Skill (recommended)

Symlink this clone to `.cursor/skills/tyhp/` in one project, or to `~/.cursor/skills/tyhp/` for every project. Cursor loads `SKILL.md` when the task involves Tyhp; the agent then opens individual `guide/` / `handbook/` files as needed. Don't attach the whole `guide/` — that defeats the split.

```bash
git clone https://github.com/tyhpproject/tyhp-ai-dev-guide.git
mkdir -p .cursor/skills
ln -s /path/to/tyhp-ai-dev-guide .cursor/skills/tyhp
```

When this clone is a sibling of the project (`../tyhp-ai-dev-guide`), the link from the project root is:

```bash
mkdir -p .cursor/skills
ln -s ../../../tyhp-ai-dev-guide .cursor/skills/tyhp
```

That relative path is from `.cursor/skills/`, not from the project root.

### Option B — Cursor rule (always-on index, deeper files on demand)

Install the skill link from Option A, then create `.cursor/rules/tyhp.mdc`:

```mdc
---
description: How to write Tyhp (a typed superset of the PHP language). Applies when editing Tyhp files.
globs: ["**/*.tyhp", "**/*.tyhpdef"]
alwaysApply: false
---
@.cursor/skills/tyhp/QUICK_GUIDE.md
```

The `@`-reference pulls the ~1.3k-token index in only when a `.tyhp`/`.tyhpdef` file is in play; the
agent then opens individual `guide/`/`handbook/` files as the task requires. Don't attach the whole
`guide/` — that defeats the split.

### Option C — `AGENTS.md`

This repository's `AGENTS.md` is an agent entry point. Add a pointer from your project-root `AGENTS.md` to the clone:

```md
This is a **Tyhp** project. Before writing/editing `.tyhp` / `.tyhpdef`, read the Tyhp guide's `AGENTS.md` (the `tyhp-ai-dev-guide` clone).
```

### Option D — any other agent/tool

Paste `QUICK_GUIDE.md` into the system prompt / project instructions, and make the `guide/` and
`handbook/` files reachable so the agent can open the referenced section.

## Token-efficiency tips

- **Load only what's needed (highest yield).** Keep `SKILL.md` / `QUICK_GUIDE.md` as the always-on
  index and let the agent open one section file at a time — far cheaper than loading a monolith.
- **Lean on prompt caching.** If your tool/provider caches static prompt prefixes (Anthropic/OpenAI
  do), the stable index is nearly free after the first call.
- **Don't run automated token-pruners** (LLMLingua, etc.) — they strip "low-information" tokens and
  will corrupt the verbatim Tyhp/PHP code. Only prose is safe to compress.
- Don't hardcode line numbers in pointers — filenames are the stable anchor; sections move on edit.

## Keep it accurate

- `guide/28-availability-gotchas.md` reflects what the current Tyhp toolchain actually compiles;
  the "use the PHP form instead" table keeps the agent from emitting code that won't build. The
  `handbook/` marks ⚠️ for unimplemented tooling (`tyhp watch`, `tyhp lint --fix`, a test runner,
  `psr4` folder remapping, etc.).
- **Match your Tyhp version.** Update this clone when you upgrade Tyhp. Maintainers regenerate the guide from a compiler checkout and a [tyhp-runtime-src](https://github.com/tyhpproject/tyhp-runtime-src) checkout (`REGEN.md`). Don't regenerate it yourself.
- **Never translate or reformat the code, keywords, type names, or identifiers** — they are literal
  Tyhp/PHP syntax.

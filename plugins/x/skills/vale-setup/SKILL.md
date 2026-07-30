---
name: vale-setup
description: Use when the user wants to install or bootstrap the Vale prose linter with Claude Code hooks, so Claude's markdown and MCP writes get linted against the CustomBans rules. Triggers include "set up vale", "install vale", "add the vale hooks", "lint Claude's writing", "stop Claude using em dashes", or setting this up on a new machine.
---

# Bootstrap Vale + Claude Code hooks

Three parts: the `vale` binary, a config plus the CustomBans style, and two hooks in `settings.json`. Do them in that order and verify each one.

## 1. Install vale and jq

```bash
command -v vale jq
```

Install whatever is missing. macOS: `brew install vale jq`. Linux: `apt install jq` and the Vale release tarball from https://github.com/errata-ai/vale/releases. Vale 3.x is assumed.

## 2. Place the config

Ask where the config goes only if the user has not said. Otherwise use the default.

**Default (no env var needed).** `vale ls-dirs` prints the platform's global config location; on macOS that is `~/Library/Application Support/vale/`. Copy `assets/.vale.ini` and `assets/styles/` there.

**`~/.config/vale` (Devon's machines).** Vale does not look there on its own. Copy the same files to `~/.config/vale/` and export the env var in the shell profile:

```bash
export VALE_CONFIG_PATH="$HOME/.config/vale/.vale.ini"
```

Note that hooks inherit the environment of the Claude Code process, so a profile export only applies to sessions started afterwards. If the current session needs it, also set it in `~/.claude/settings.json` under `env`.

Either way the layout is:

```
<config dir>/.vale.ini
<config dir>/styles/CustomBans/*.yml
```

`.vale.ini` uses `StylesPath = styles`, relative to the ini, so the pair travels together.

Do not clobber an existing config. If `.vale.ini` is already there, merge: add the `CustomBans` rule files, and add `CustomBans` to `BasedOnStyles` for markdown rather than rewriting the section.

Verify:

```bash
vale ls-config | head -20
echo "This is genuinely a load-bearing sentence — it has an em dash." | vale --ext=.md
```

Expect several errors. Silence means the style is not loading, usually the wrong config dir or a bad `StylesPath`.

## 3. Install the hooks

`assets/hooks.json` holds both hooks, ready to merge into `~/.claude/settings.json` (user scope, so they apply everywhere).

- **PostToolUse** on `Edit|Write|MultiEdit` runs `vale` on any `.md` or `.txt` file Claude just wrote. Exit 2 feeds the errors back so Claude fixes them itself.
- **PreToolUse** on the MCP write tools lints the outgoing text *before* it lands in Notion, Google Docs, or Linear. Exit 2 blocks the call. This one needs `jq`.

Both hooks no-op when `vale` is absent, so they are safe on a machine that has not finished step 1.

Merge by hand or with the `update-config` skill: append to the existing `PreToolUse` / `PostToolUse` arrays, never overwrite them. Extend the MCP matcher regex to cover whichever MCP servers this user actually writes with.

Verify by having Claude write an offending sentence to a scratch `.md` file. The hook should reject it. Delete the scratch file after.

## The rules themselves

`assets/styles/CustomBans/` ships eleven starter bans: the em dash, unicode arrows, emoji, a handful of LLM intensifiers, and several rhetorical patterns. Read the filenames for the list. They encode one person's taste, not a house style, so suggest the user prune or extend after living with them.

To add rules later, use the `x:vale-rule` skill.

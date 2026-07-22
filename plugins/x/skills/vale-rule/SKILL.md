---
name: vale-rule
description: Use when the user wants to add, change, or debug a personal Vale lint rule. Triggers include "ban this phrase in Vale", "add a Vale rule / ban / lint", "make Vale flag X", "add a CustomBans rule". Covers the CustomBans style at ~/.config/vale/styles/CustomBans/.
---

# Personal Vale rules (CustomBans)

Devon keeps a personal Vale style at `~/.config/vale/styles/CustomBans/`. Each rule is one `<RuleName>.yml` file. The global config at `~/.config/vale/.vale.ini` already loads the whole style for markdown:

```ini
[*.{md,MD}]
BasedOnStyles = CustomBans
```

A new rule goes live the moment its file lands in that directory. No `.vale.ini` edit is needed. The filename in PascalCase (without the extension) becomes the rule ID, surfaced to the user as `CustomBans.<RuleName>`.

Most requests are "ban a word or phrase." Start from this template:

```yaml
extends: existence
message: "Avoid using '%s' ..."   # %s interpolates the matched text
level: error
ignorecase: true
tokens:
  - phrase to ban
```

## tokens vs raw

| Banning | Field | Why |
| --- | --- | --- |
| A word or phrase | `tokens` | Entries are regex fragments, joined into one non-capturing group and auto-wrapped in `\b` word boundaries. |
| A symbol or punctuation character | `raw` | No word boundaries, no wrapping. A `\b` next to a symbol like an arrow glyph matches in the wrong places. |
| A multi-word rhetorical pattern | `tokens` with regex | Boundaries still fit; write the pattern as a regex (see the existing rhetorical rules). |

`tokens` are NOT escaped, so regex works inside them, e.g. `it['’]?s not [^.!?;]+, it['’]?s`. `raw` is concatenated verbatim.

## Gotchas

- **YAML quoting**: any regex containing a backslash (`\s`, `\d`, `\b`) must go in **single** quotes or a block scalar. Double quotes make YAML try to read `\s` as a string escape, and the whole style then fails to load. This is a real failure that has happened here.
- **Regex engine**: a superset of Go's RE2. Lookahead and lookbehind both work (`(?=)`, `(?!)`, `(?<=)`, `(?<!)`). Backreferences do NOT, so a repeated-word rule cannot use `\1`.
- **Severity**: the config sets `MinAlertLevel = warning`, so a `level: suggestion` rule stays hidden. Use `error` (the house default) or `warning`.
- **Message voice**: existing rules open with "Avoid ..." and quote the match via `%s`. Multi-line guidance uses a `|` block scalar; see `Emdash.yml` for the format.

## Workflow

1. Write `~/.config/vale/styles/CustomBans/<RuleName>.yml`.
2. Verify it loads and matches. Pipe a sentence containing the banned text through Vale:
   ```bash
   echo "a sentence with the banned phrase" | vale --ext=.md
   ```
   One flagged line means it works. A "config" or parse error means the YAML is malformed, usually the quoting gotcha above.
3. Also confirm it does NOT fire on acceptable text, so the pattern is not too broad.
4. Show the user the rule and the test result.

## Beyond existence bans

For substitution suggestions (ban X, propose Y), use `extends: substitution`. Other check types (capitalization, sequence, conditional, occurrence) are documented at https://docs.vale.sh/checks/. Read the relevant check page and adapt the same one-file-per-rule pattern.

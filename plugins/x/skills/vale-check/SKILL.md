---
name: vale-check
description: Use when the user asks to Vale-check, lint, or proofread some prose against the personal CustomBans rules. Triggers include "vale check that", "run vale on this", "lint this doc", "check that message before I send it", or a request to check text Claude just drafted in chat. Covers files, directories, pasted text, chat drafts, and remote docs fetched over MCP.
---

# Run Vale over target material

The target is whatever the user just gestured at. Resolve it first, then lint.

| Target | How |
| --- | --- |
| A file on disk | `vale path/to/file.md` |
| Several files or a directory | `vale docs/` or `vale --glob='*.md' .` |
| Text in the conversation (a draft, a pasted message, something you just wrote in chat) | pipe it: `printf '%s\n' "$text" \| vale --ext=.md` |
| A file with a non-markdown extension | `vale --ext=.md path/to/file` |
| A Notion page, Google Doc, or Linear issue | fetch it with the MCP read tool, write the body to a scratch `.md`, lint that, delete the scratch file |
| A marimo notebook | use the `x:marimo-lint` skill instead, it exports the `mo.md` cells first |

"That" with no antecedent usually means the last thing you wrote or edited. Say which target you picked before reporting results.

The config only binds `CustomBans` to `[*.{md,MD}]`, so anything arriving over stdin or with another extension needs `--ext=.md` or it lints clean for the wrong reason.

## Piping text safely

Heredocs beat `echo` for multi-line drafts, and avoid the shell mangling backslashes or leading dashes:

```bash
vale --ext=.md <<'EOF'
The draft text goes here.
Multiple lines are fine.
EOF
```

For long text, write it to the session scratchpad and lint the file. That keeps line numbers stable across a fix-and-recheck loop.

## Reading the output

Each hit is `line:col  error  <message>  CustomBans.<RuleName>`. The line numbers refer to the linted artifact, so on piped text they count from the start of the heredoc, not the source document.

Exit status is nonzero when anything fires, which is what the hooks key on. Under `set -e` or a `&&` chain that aborts the rest of the command, so lint last or use `|| true` when you only want the report.

Nothing reported means the prose passed the bans. It does not mean the prose is good. Vale checks a list of banned strings, not clarity or structure. Say so rather than declaring the text clean, and for a real writing pass use `x:writing`.

## After a check

Fix and recheck the same target rather than reporting the list and stopping, unless the material is the user's own writing. Do not paraphrase around a rule while keeping the shape it exists to prevent, e.g. swapping an em dash for a spaced hyphen. Rewrite the sentence.

If a rule fires on text that is correct, e.g. a quotation, a proper noun, or a code identifier, say so and leave the text alone. Markdown comments can suppress a rule for a span:

```markdown
<!-- vale CustomBans.Honestly = NO -->
Quoted text that trips the rule.
<!-- vale CustomBans.Honestly = YES -->
```

Use that sparingly, and only in a file the user owns. Reaching for it on every hit defeats the point. A rule that keeps misfiring wants narrowing via `x:vale-rule` instead.

If Vale reports a config error instead of alerts, the style is not loading. See `x:vale-setup`.

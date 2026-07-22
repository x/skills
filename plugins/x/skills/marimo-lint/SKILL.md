---
name: marimo-lint
description: Use to check the prose of a marimo notebook (a .py file with `import marimo` and `marimo.App`) against the personal CustomBans Vale rules. Run after editing a notebook's `mo.md(...)` cells, or when the user asks to lint / Vale-check a marimo notebook. Exports the notebook markdown to a temp file and runs Vale on it.
---

# Lint marimo notebook prose with Vale

A marimo notebook keeps its prose inside `mo.md(r"""...""")` cells, which are plain string arguments. Vale only reads comments and docstrings in a `.py` file, so a direct `vale notebook.py` never sees that prose. Export the notebook to markdown first, then lint the export.

## Command

```bash
d=$(mktemp -d)
uvx marimo export md "$NOTEBOOK" -o "$d/out.md" && vale "$d/out.md"
rm -rf "$d"
```

Set `NOTEBOOK` to the notebook path, or inline it. The temp directory guarantees an `.md` name (Vale needs the extension to apply the markdown rules) and `rm -rf` always cleans up, whatever Vale's exit code.

`marimo export md` turns each `mo.md(...)` cell into real markdown and each code cell into a fenced ` ```python ` block. Vale lints the prose and skips fenced code, so only the rendered markdown is checked; code, `#` comments, and docstrings in code cells are left alone.

## Mapping findings back

Vale reports line numbers for the temp export, NOT the `.py` source. To fix a finding:

1. Read the flagged text in Vale's output.
2. Find that same text in the notebook's `mo.md(...)` cells.
3. Fix it there, in the `.py`. Never edit the export; it is discarded.

Re-run the command to confirm the finding is gone.

## Notes

- Run this after editing a notebook's markdown cells. The Write/Edit Vale hook fires on `.md` and `.txt`, not `.py`, so notebook prose is otherwise unchecked.
- `uvx` fetches marimo on first use and caches it, so later runs are fast. Pin the format with `uvx marimo@<version>` if a notebook needs a specific one.
- The bans live in the `CustomBans` Vale style. To add or change one, use the `vale-rule` skill.

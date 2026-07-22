---
name: uv-script
description: Use when the user asks for a self-contained / single-file Python script, a script with third-party dependencies but no project or venv, or references uv run, uvx, or PEP 723 inline script metadata. Produces a script that declares its own dependencies inline and runs directly via uv.
---

# Self-contained uv scripts (PEP 723)

Write standalone Python scripts that declare their own dependencies inline, so the user runs one file with no venv, `pip install`, or `requirements.txt`. [uv](https://docs.astral.sh/uv/) reads the [PEP 723](https://peps.python.org/pep-0723/) metadata block, resolves the deps into a cache, and runs the script in an ephemeral environment.

Reach for this for one-off tools, glue scripts, and anything the user wants to `chmod +x` and run. For anything with multiple modules, tests, or a package layout, make a real `uv init` project instead.

## Anatomy

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# dependencies = [
#     "playwright",
#     "httpx>=0.27",
# ]
# ///

import httpx
...
```

- **Shebang** `#!/usr/bin/env -S uv run --script`: `-S` splits the string so `env` runs `uv run --script`. With `chmod +x script.py` the file runs directly (`./script.py`); uv auto-installs the right Python and deps on first run.
- **Metadata block**: must be the first comment block, opened by `# /// script` and closed by `# ///`, every line prefixed with `# `. It is TOML.
- `requires-python`: uv downloads a matching interpreter if the system one doesn't satisfy it. Set the real floor the code needs.
- `dependencies`: PEP 508 strings. Leave it `[]` (or omit) for a stdlib-only script that still wants the shebang.

## Running

```bash
./script.py               # via shebang, once chmod +x
uv run script.py          # explicit; works without the +x bit
uv run --with rich script.py   # add an ad-hoc dep not in the block
```

## Editing dependencies

Prefer uv's commands over hand-editing the block; they resolve and write it for you:

```bash
uv add --script script.py 'httpx>=0.27'
uv remove --script script.py httpx
```

## Reproducibility

For a script that must resolve the same way over time, pin the resolution date:

```python
# /// script
# dependencies = ["httpx"]
#
# [tool.uv]
# exclude-newer = "2026-01-01T00:00:00Z"
# ///
```

Or generate a lockfile beside the script: `uv lock --script script.py` writes `script.py.lock`, which `uv run` then honors.

## Notes

- Keep it to one file. If you're adding a second module, it's a project now: stop and `uv init`.
- Don't emit a `requirements.txt`, `pip install` line, or venv activation step alongside the script; the metadata block replaces all of that.
- `uvx` runs a published tool from PyPI; `uv run --script` runs a local file. This skill is about the latter, so don't tell the user to `uvx` a file path.

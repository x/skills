---
name: codex
description: Use when the user asks to run Codex CLI (codex exec, codex resume) or references OpenAI Codex for code analysis, refactoring, or automated editing
---

# Codex CLI

Run Codex non-interactively with `codex exec`:

```bash
codex exec --skip-git-repo-check --sandbox <mode> [flags] "prompt" </dev/null 2>/dev/null
```

Always include:

- `--skip-git-repo-check`
- `</dev/null` — `codex exec` reads stdin even when the prompt is a positional argument, and hangs forever if stdin is open but silent. The symptom is a process with no output and no CPU time.
- `2>/dev/null` — stderr carries thinking tokens. Show them only if the user asks or you are debugging.

Use Codex's default model and reasoning effort. When the user names one, pass `-m <model>` and/or `--config model_reasoning_effort="<low|medium|high|xhigh>"`. Use `-C <dir>` to run in another directory.

Pick the sandbox by what the task needs:

| Task | Flags |
| --- | --- |
| Review or analysis (default) | `--sandbox read-only` |
| Edit files | `--sandbox workspace-write --full-auto` |
| Network or files outside the workspace | `--sandbox danger-full-access --full-auto` — get the user's OK first |

## Output and timeouts

Codex prints nothing until it finishes. A process killed early leaves empty output and no error, so prefer running synchronously. If you background it, set the timeout by reasoning effort: 150s for low, 300s for medium, 600s for high, 1200s for xhigh.

## Resuming

Pipe a follow-up prompt to resume the last session. It inherits the model, effort, and sandbox from the original run:

```bash
echo "follow-up prompt" | codex exec --skip-git-repo-check resume --last 2>/dev/null
```

Flags go between `exec` and `resume`; don't add model or effort flags unless the user asks to change them. After a run, let the user know the session is resumable.

## Reporting results

Summarize the outcome rather than dumping raw output. Treat Codex as a peer, not an authority: verify claims that look wrong against your own knowledge or the docs, and tell the user when you disagree. If `codex` exits non-zero, report the failure and ask before retrying.

---
name: review-python-pr
description: Use when the user asks to review a Python pull request, "review this PR", "review PR #123", or hands you a PR link/branch for Python code. Pulls the PR into a git worktree, confirms the code imports and runs, then walks the fixed review checklist.
---

# Review a Python PR

Review a Python PR against the checklist below. Do the setup first. A review that never ran the code is incomplete.

## Setup

1. Resolve the target (a PR number, URL, or branch) and read it:

   ```bash
   gh pr view <number-or-url> --json headRefName,title,url,body,files
   ```

2. Pull it into a worktree. Never review in the main checkout.

   ```bash
   git fetch origin
   git worktree add ~/work/<repo>-pr-<number> <headRefName>
   ```

   Work from that directory for the rest of the review.

3. Install and confirm the code runs. Use `uv sync` if the project has it, otherwise the project's own setup. Then actually import the changed modules and run a snippet: call a changed function, instantiate a changed class, or run the tests. If it does not import or run, stop and report that.

4. Scope the review to the diff:

   ```bash
   git diff --stat origin/<base>...HEAD
   ```

## Checklist

Answer each question for the changed code. Report grouped by question, each finding with a `file:line`. Say "not applicable" where it fits.

### README is descriptive, not boilerplate
Descriptive of the project, or leftover template from `init`? Flag a placeholder README.

Sample comment: "Let's burn the majority of the readme, it's just boilerplate."

### Configured linter passes
Is a linter configured (ruff, pylint, flake8, in `pyproject.toml`, `setup.cfg`, `.ruff.toml`, `.pylintrc`)? If so, run it yourself and report pass/fail with the output. A configured linter that fails is a finding.

Sample comment: "ruff's failing for me locally, can you run it and clean up before merge?"

### No commented-out code
Any commented-out code is an issue unless there is a clear written justification for keeping it.

Sample comment: "nit, drop this. we've got git history if we need it back."

### No lazy imports
Imports inside functions instead of at module top are an issue, unless it is a Temporal workflow (where scoped imports are deliberate). Flag each with its reason or exception.

Sample comment: "i have a strong distaste for lazy imports. And I don't think this one is necessary."

### Type-hints are consistent
If the rest of the project is type-hinted, the new code should be too, even when the linter does not enforce it. Selective hinting is fine; focus on function arguments and return types. Flag new functions missing hints when the surrounding code has them.

Sample comment: "The rest of the project uses hinting on args and returns, let's stay consistent and hint this too."

### Serialized data uses pydantic
Any plain dicts or plain classes that get serialized (to JSON, an API, a queue, storage) that should be pydantic models? Flag them.

Sample comment: "This dict gets serialized out to the API, let's make it a pydantic model so we get validation for free."

### Timezone bugs
- Are time objects all stored with timezones?
- Are we assuming local or utc time anywhere without explicitly stating it?
- When serializing, are we using ISO8601 strings or a similar robust format?
- WHen storing a dataetime in a pydantic mode, does it use AwareDatetime?

Sample comment: "I think these should be timezone aware."


### Daylight Savings Bugs
- Does any time-logic code in this code work and make snese if start and end overlap with daylight savings?
- Are there any implicit assumptions that a day only has 24 hours (when it could have 23 or 25 on DST)?

Sample comment: "I'm imagining a scenaior around daylight savings where ... Could we add a test for that and ensure this still works?"

### Range endpoints are consistent
- Does every range pick inclusive-start/exclusive-end or exclusive-start/inclusive-end?
- Are both ends the same (both inclusive or both exclusive)? That's the footgun.
- Is the chosen convention stated somewhere the caller sees it?

Sample comment: "Let's user inclusive start and exclusive end, or exclusive start and inclusive end. Both ends being inclusive or exclusive is a footgun."

### No dead or throwaway files
- Any unused files (dev notebooks, scratch scripts) in the diff?
- Any unused helpers left behind?

Sample comment: "nit, drop this whole file unless you're using it."

## Report

Give a per-question summary with `file:line` references and a verdict (approve /
approve-with-comments / request-changes). State plainly whether the code
imported and ran, and whether the linter passed. Note the worktree path so the
user can clean it up (`git worktree remove <path>`).

## Adding a question

Add a `###` sub-header under Checklist, in the form "assertion of what good
looks like". One question per header. State the rule, then the exception if any,
then what to flag. Use bullets when the header asks several specific things.
End each header with a `Sample comment:` line in my PR voice (lowercase, terse,
"nit," for small stuff, direct about the fix). Keep it to a few lines.

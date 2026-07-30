---
name: writing
description: Apply the writing ruleset in WRITING.md when drafting or revising long-form prose, e.g. docs, specs, reports, articles, criticism, retrospectives, PR descriptions, long-form posts. Prefers concrete specificity over polished generality, and resists common LLM cadence tells (parallel triads, signpost openers, ceremonial wrap-ups). Skip for chat, comments, replies, code, or pure data.
---

# Writing

You are a technical writer. The audience is well educated and informed, and wants it short, terse, and clear.

Read `WRITING.md` in this skill directory. It is the operative ruleset: the core rules and the required checks. Follow it.

Above all, omit needless words. Never use this skill to make something short longer.

## Workflow

1. Identify the genre, audience, and what the reader needs. That decides the shape before you write a word.
2. Draft to the file the piece belongs in, or to the session scratchpad if there is no destination yet.
3. Run the required checks in `WRITING.md`. Revise against them, and do not report the audit.
4. Cut. Ask what sentences or paragraphs can go, and what specificity is decorative rather than earned.
5. Lint. Writing to a `.md` or `.txt` file already triggers the hook. For anything else, e.g. an MCP write to Notion or Google Docs, use `x:vale-check`. Fix what it flags by rewriting the sentence, not by paraphrasing around the rule.

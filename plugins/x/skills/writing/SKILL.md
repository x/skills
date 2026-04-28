---
name: writing
description: Apply the writing ruleset in WRITING.md when drafting or revising prose for human readers — chat messages, emails, docs, articles, PR descriptions, comments, replies, long-form posts. Fits format to the medium, prefers concrete specificity over polished generality, and resists common LLM cadence tells (parallel triads, signpost openers, ceremonial wrap-ups, reflexive em-dashes). Skip for code, type signatures, or pure data.
---

Read `WRITING.md` in this skill's directory and apply it to the writing task at hand. The file is the source of truth — re-read it on each invocation rather than working from a remembered summary, since it may have been updated.

Workflow:

1. Identify the medium, audience, and reader need (per the "Core workflow" and "Medium routing" sections).
2. Draft to fit that context.
3. Run the "Required checks" appropriate to the length and stakes of the piece. Do not output the audit unless the user asks for it.

Precedence order from the ruleset still applies: truth/safety/accessibility first, then explicit user instructions, then genre and medium norms, then the rules themselves.

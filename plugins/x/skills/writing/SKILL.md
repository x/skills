---
name: writing
description: Apply the writing ruleset in WRITING.md when drafting or revising prose for human readers, e.g. chat messages, emails, docs, articles, PR descriptions, comments, replies, long-form posts. Fits format to the medium, prefers concrete specificity over polished generality, and resists common LLM cadence tells (parallel triads, signpost openers, ceremonial wrap-ups, reflexive em-dashes). Skip for code, type signatures, or pure data.
---

# Writing

You are a technical writer. The audience is well educated and informed, and wants it short, terse, and clear.

Read `WRITING.md` in this skill directory. It is the operative ruleset: medium routing, the core rules, and the required checks. Follow it.

Above all, omit needless words. Never use this skill to make something short longer.

## Workflow

1. Identify the medium and audience. `WRITING.md` routes chat, email, docs, and long-form differently, and the routing decides format before you write a word.
2. Draft. For a chat reply or anything under a few paragraphs, draft in your response. For a document, draft to the file it belongs in, or to the session scratchpad if there is no destination yet.
3. Run the required checks in `WRITING.md`. Short pieces get checks 1-5, 7, and 10; longer pieces get all of them. Revise, do not report the audit.
4. Cut. Ask what sentences or paragraphs can go, and what specificity is decorative rather than earned.
5. Vale-check. Writing to a `.md` or `.txt` file already triggers the lint hook. For anything else, including chat replies and MCP writes, use `x:vale-check`. Fix what it flags by rewriting the sentence, not by paraphrasing around the rule.

## Reference

`ELEMENTS_OF_STYLE.md` in this directory is an excerpt of Strunk and White, sections II and V only. Consult it for depth on a specific principle, e.g. rule 17 on needless words, rule 14 on active voice, rule 16 on concrete language. Do not read it end to end for every task, and do not imitate its period prose. `WRITING.md` wins on any conflict, including the em dash.

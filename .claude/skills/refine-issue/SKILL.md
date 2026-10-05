---
name: refine-issue
description: Interview the user about a rough task idea until it becomes a well-scoped Linear issue in the house format, then post it. Use when the user wants to write, refine or file an issue, for a person or for Alti (Homebase), or turn a vague idea into an agent-ready task.
---

Turn a rough idea into an issue that a person can understand in 30 seconds and Alti (Homebase) can build without guessing. The user's rough idea: $ARGUMENTS

## Ground rules

- Interview one question at a time. Wait for the answer before the next question. Multiple questions at once is bewildering.
- For every question, give your recommended answer so the user can just say "yes".
- If a question can be answered by exploring the codebase, its docs, or a linked Figma frame: explore instead of asking. Only ask what genuinely needs a human decision.
- Read the repo's `CLAUDE.md` first when you are in one. Its conventions decide what "done" means for the acceptance criteria.
- Stop interviewing when the parts below are sharp, not when you run out of questions. Usually 3 to 6 questions.
- One task per issue. If the idea has an "and also", it is two issues: say so and offer to file both.

## The house format

The team's standard ("How we write Linear issues", Notion, owner Thor). The test: could someone outside the team read it once and say what needs to happen and why within 30 seconds?

- English. Plain words first, the short form in parentheses after it ("our server (API)", "the consent screens in the onboarding (X9)"). Brand names in full: Altid Hjem, Altid Mad, Altid Energi. No user stories.
- Title: a verb plus a concrete result. No codes.
- Top lines, only when real: `**Decision needed:**` what, from whom. `**Blocked:**` by what.
- `## Context`: what is wrong or missing, who it affects, why it matters. A few whole sentences. The task, not the solution. No history: link it.
- `## Goal`: one sentence that is true when the issue is done, answerable yes or no. Never "better", "improve" or "at least".
- `## Out of scope`: what this issue does not cover. Push the user on this; everyone under-fills it.
- `## Acceptance criteria`: 3 to 6 checkboxes that together prove the Goal without repeating it. Each a result someone else can check, not a method ("the user can try again", not "add a retry button"). Include the boring ones (translations, tests updated) when relevant.
- `## Links`: real links on descriptive words, never "see Slack" or a bare number. Slack: the exact message. Notion: the exact page or section. Figma: the exact frame ("select the frame, right-click, Copy link to selection"), one per screen or section touched.
- Folded at the bottom: `+++ AI Details` on its own line, then the technical part, then `+++` on its own line. Paths and files, repro steps, constraints, what was already tried, exact technical names. Any language, as technical as needed. When the task touches a shared foundation (theme, routing, translations, auth, anything other modules build on), name the doc section that governs it here, not just the file: Alti will not plan foundation work from neighbouring code alone.

## Suitability check

If the user wants Alti to build it, judge whether this is agent work at all before drafting. Not agent-suitable: architecture decisions, work blocked on another team's backend changes, exploratory product work, anything spanning repos. If unsuitable, say so, recommend how to handle it instead, and still offer to file it for a person.

## Finish

1. Show the complete draft issue and get an explicit OK.
2. Create it in Linear, team Altid Hjem (`ALT-`), in Backlog with no owner and no sprint: a person sets points, owner and sprint. Add a type label (`Bug`, `Feature` or `Improvement`). Alti picks the repo from the issue, so no area label is needed. Use the Linear MCP; if it is not connected, stop and tell the user to connect it rather than posting another way.
3. Ask whether to delegate to Alti now. Delegating = set Alti as the issue's delegate in Linear and move it from Backlog to Todo (Homebase skips Backlog); Homebase picks it up on its next tick. Never do this without an explicit yes.

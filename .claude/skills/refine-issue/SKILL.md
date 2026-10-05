---
name: refine-issue
description: Interview the user about a rough task idea until it becomes a well-scoped issue for Alti, then post it to Linear. Use when the user wants to write, refine or file an issue for Alti (Homebase), or turn a vague idea into an agent-ready task.
---

Turn a rough idea into an issue Alti (Homebase) can implement without guessing. The user's rough idea: $ARGUMENTS

## Ground rules

- Interview one question at a time. Wait for the answer before the next question. Multiple questions at once is bewildering.
- For every question, give your recommended answer so the user can just say "yes".
- If a question can be answered by exploring the codebase, its docs, or a linked Figma frame: explore instead of asking. Only ask what genuinely needs a human decision.
- Read the repo's `CLAUDE.md` first when you are in one. Its conventions decide what "done" means for the acceptance criteria.
- Stop interviewing when the four sections below are sharp, not when you run out of questions. Usually 3 to 6 questions.

## What you are filling in

Four sections:

1. **Context**: the problem and the why. Name the repo it belongs in. Link the spec or design doc if one exists. When the task touches a shared foundation (theme, routing, translations, auth, anything other modules build on), name the doc section that governs it, not just the file: Alti will not plan foundation work from neighbouring code alone.
2. **Acceptance criteria**: a checklist, each item independently checkable. "Price card shows the monthly total incl. VAT", not "price works". Include the boring ones (translations, tests updated) when relevant.
3. **Design**: Figma node links, one per screen or section touched. Ask the user to copy them if UI is involved ("select the frame, right-click, Copy link to selection"). "none" for non-UI work.
4. **Out of scope**: what must not be touched. Push the user on this; it is the section everyone under-fills.

## Suitability check

Before drafting, judge whether this is agent work at all. Not agent-suitable: architecture decisions, work blocked on another team's backend changes, exploratory product work, anything spanning repos. If unsuitable, say so, recommend how to handle it instead, and stop.

## Finish

1. Show the complete draft issue and get an explicit OK.
2. Create it in Linear, team Altid Hjem (`ALT-`), with the four sections as markdown headings. Alti picks the repo from the Context, so no area label is needed. Use the Linear MCP; if it is not connected, stop and tell the user to connect it rather than posting another way.
3. Ask whether to delegate to Alti now. Delegating = set Alti as the issue's delegate in Linear; Homebase picks it up on its next tick. Never do this without an explicit yes.

---
name: standup
description: Short standup summary for Lars across the Altid Hjem repos (app, api, homebase) and Linear. What got done since last standup in very broad terms, blockers from other people, and what today is about based on the current sprint. Keeps a running journal so yesterday's open threads carry over. Use when Lars asks for a standup, daily summary, or "what did I do yesterday / what's today".
---

# Standup

Produce a standup Lars can say out loud in under a minute. Broad strokes only: "finished the auth session rework", not "fixed null check in session_state.dart".

## Your role

You are a reporter, not a planner. Understand the vision and where the sprints are heading so you can summarise sensibly, but do not invent work, reprioritise, suggest new tasks, or "improve" the plan. "Today" comes from what is already in the sprint and what was left open yesterday. If the sprint is unclear, say so instead of filling the gap.

Read-only: no Linear changes, no Slack posts, no git writes. The only file you write is the journal.

## Journal

`~/.claude/standup/journal.md` (create it, and the directory, if missing). One entry per standup, newest first:

```markdown
## 2026-10-01
Done: <1-3 broad lines>
Today: <1-3 lines, ALT-ids where they exist>
Blockers: <who, what> or none
Carry over: <open threads the next session needs to keep working: half-done branches, PRs waiting on review, decisions pending, things promised to someone>
```

Read it first. The last entry sets the window ("since" = its date; no entry = last working day, Monday reaches back to Friday) and its Carry over is the first thing to check: resolved, still open, or gone stale. Keep the last 10 entries; delete older ones. If today already has an entry, replace it.

## Gather (in parallel)

1. Vision, cheap and once: the Overview in `/data/development/altid_hjem/app/CLAUDE.md`, the current cycle's description, and the newest project status updates (`get_status_updates`, type project). Enough to know what the sprint is for. Don't read specs.
2. Linear, team Altid Hjem:
   - `list_cycles` type current (teamId must be the UUID `aaff3d60-51d8-492a-90a7-7342996e5de3`; the name fails), then `list_issues` with that cycle. This is the priority source for Today.
   - Issues assigned to me (`assignee: "me"`) updated since the window.
   - Issues labelled `blocked`, and issues where Alti or someone else asked Lars a question (recent comments mentioning him).
3. Git, for each of `/data/development/altid_hjem/{app,api,homebase}` (skip the `homebase-wt-*` worktrees, they are the same repo): `git fetch --quiet`, then `git log --all --since=<window> --format='%h %an %s'`. Count `altid-alti[bot]` work in app and api separately; it is the pipeline, not Lars. Homebase is the exception: everything merged there, by Lars or by the bot, is Lars's orchestration and goes under his Yesterday.
4. GitHub per repo (`Altid-Hjem-Aps/altid-hjem-app`, `altid-hjem-api`, `homebase`): `gh pr list --state merged --search "merged:>=<date>"` and `gh pr list --state open`. Open PRs waiting on someone else's review or action are blocker candidates.
5. Slack (altid-slack MCP, Altid Hjem workspace): skim #engineering and #core-team since the window, only for things blocking Lars or waiting on him. Skip if the server is down and say so in one line.

If a source fails (Linear auth, gh, Slack), say which one in the output. Don't silently report less.

## Blockers

Only things where someone other than Lars has to act: a review, an answer, access, a vendor, a decision. Name the person (see the altid-crew memory for who's who). Things Lars himself has to do are Today, not blockers.

## Output

Write in the language Lars asked in. Three short sections as bullet lists, one short bullet per item, no more than 10 bullets total:

```
Igår:
- <broad item>
I dag:
- <from sprint + carry over, described in plain words>
Blockers:
- <person: what> (or "- ingen")
```

(English: Yesterday / Today / Blockers.) It is Lars's standup: Yesterday and Today hold only what he did or will work on himself (his commits, reviews and fixes, his Homebase orchestration, issues assigned to him). Other people's work, like Thor's readout or Alex's flows, is left out unless Lars did part of it, or it blocks him, and then it goes under Blockers. Say what each item is in plain words ("the Altid Mad-only login token"), never as an issue id: Lars runs an AI-driven team and does not remember what most ALT numbers are. No ALT ids, PR numbers, tables, bold or file paths in the output. The journal keeps the ids, since the next session needs them to look things up. Pipeline merges get one line at most ("Alti merged 4 small fixes"). After the output, write the journal entry, then stop.

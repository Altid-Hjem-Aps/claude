---
name: standup
description: Short personal standup across the Altid Hjem repos, Linear and Slack. What the user got done since last standup in very broad terms, blockers from other people, and what today is about based on the current sprint. Keeps a running journal so yesterday's open threads carry over. Use when the user asks for a standup, daily summary, or "what did I do yesterday / what's today".
---

# Standup

Produce a standup the user can say out loud in under a minute. Broad strokes only: "finished the auth session rework", not "fixed null check in session_state.dart".

## Your role

You are a reporter, not a planner. Understand the vision and where the sprints are heading so you can summarise sensibly, but do not invent work, reprioritise, suggest new tasks, or "improve" the plan. "Today" comes from what is already in the sprint and what was left open yesterday. If the sprint is unclear, say so instead of filling the gap.

Read-only: no Linear changes, no Slack posts, no git writes. The only files you write are the config and the journal.

## Who and where: `~/.claude/standup/config.md`

Read it first. If it is missing, build it once and show it to the user to confirm before the first standup:

- Identity: `git config user.name` and `user.email`, `gh api user -q .login`, Linear's `me`, and the user's Slack name (the Slack MCP's whoami or profile).
- Repos: local clones whose `origin` is in the `Altid-Hjem-Aps` GitHub org. Find them with `find ~ /data -maxdepth 4 -name .git -type d 2>/dev/null` and check each remote. Skip extra worktrees of a repo already listed (same remote).
- Orchestrated repos: repos where bot merges count as the user's own work (for example Homebase for whoever runs it). Ask; default none.
- Slack: which workspace and channels to skim. Default the Altid Hjem workspace, #engineering and #core-team.

```markdown
Name: <name>  Git: <name/email>  GitHub: <login>  Slack: <name>
Repos: <path> (<owner/repo>), ...
Orchestrated: <repo> or none
Slack: <workspace>: #engineering, #core-team
```

## Journal: `~/.claude/standup/journal.md`

One entry per standup, newest first:

```markdown
## 2026-10-01
Done: <1-3 broad lines>
Today: <1-3 lines, ALT-ids where they exist>
Blockers: <who, what> or none
Carry over: <open threads the next session needs to keep working: half-done branches, PRs waiting on review, decisions pending, things promised to someone>
```

The last entry sets the window ("since" = its date; no entry = last working day, Monday reaches back to Friday; no standups on weekends). Its Carry over is the first thing to check: resolved, still open, or gone stale. Keep the last 10 entries; delete older ones. If today already has an entry, replace it.

## Gather (in parallel)

1. Vision, cheap and once: the Overview in the app repo's `CLAUDE.md`, the current cycle's description, and the newest project status updates (`get_status_updates`, type project). Enough to know what the sprint is for. Don't read specs.
2. Linear, team Altid Hjem:
   - `list_cycles` type current (teamId must be the UUID `aaff3d60-51d8-492a-90a7-7342996e5de3`; the name fails), then `list_issues` with that cycle. This is the priority source for Today.
   - Issues assigned to the user (`assignee: "me"`) updated since the window.
   - Issues labelled `blocked`, and issues where Alti or someone else asked the user a question (recent comments mentioning them).
3. Git, for each configured repo: `git fetch --quiet`, then `git log --all --since=<window> --format='%h %an %ae %s'`. The user's commits match their git name or email. `altid-alti[bot]` is the pipeline, not the user, except in orchestrated repos, where everything merged counts as the user's.
4. GitHub, per repo: `gh pr list --state merged --search "merged:>=<date>"` and `gh pr list --state open`. PRs the user authored or reviewed count as their work; their open PRs waiting on someone else's review or action are blocker candidates.
5. Slack (whichever Slack MCP is connected), configured channels since the window, only for things blocking the user or waiting on them. Skip if it is down and say so in one line.

If a source fails (Linear auth, gh, Slack), say which one in the output. Don't silently report less.

## Blockers

Only things where someone other than the user has to act: a review, an answer, access, a vendor, a decision. Name the person. Things the user has to do themselves are Today, not blockers.

## Output

Write in the language the user asked in. Three short sections as bullet lists, one short bullet per item, no more than 10 bullets total:

```
Igår:
- <broad item>
I dag:
- <from sprint + carry over, described in plain words>
Blockers:
- <person: what> (or "- ingen")
```

(English: Yesterday / Today / Blockers.) It is the user's own standup: Yesterday and Today hold only what they did or will work on themselves (their commits, reviews and fixes, their orchestrated repos, issues assigned to them). Other people's work is left out unless the user did part of it, or it blocks them, and then it goes under Blockers. Say what each item is in plain words ("the Altid Mad-only login token"), never as an issue id: the team is AI-driven and nobody remembers what most ALT numbers are. No ALT ids, PR numbers, tables, bold or file paths in the output. The journal keeps the ids, since the next session needs them to look things up. Pipeline merges get one line at most ("Alti merged 4 small fixes"). After the output, write the journal entry, then stop.

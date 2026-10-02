---
name: standup
description: Short personal standup across the Altid Hjem repos, Linear and Slack. What the user got done since last standup in very broad terms, blockers from other people, and what today is about based on the current sprint. Keeps a running journal so yesterday's open threads carry over. Use when the user asks for a standup, daily summary, or "what did I do yesterday / what's today".
---

# Standup

Produce a standup the user can say out loud in under a minute. Broad strokes only: "finished the auth session rework", not "fixed null check in session_state.dart".

## Your role

You are a reporter, not a planner. Understand the vision and where the sprints are heading so you can summarise sensibly, but do not invent work, reprioritise, suggest new tasks, or "improve" the plan. "Today" lists the user's open sprint work and what was left open yesterday, ordered by urgency; the user decides what to pick. If the sprint is unclear, say so instead of filling the gap.

Read-only: no Linear changes, no Slack posts, no git writes. The only files you write are the config and the journal.

## Who and where: `~/.claude/standup/config.md`

Read it first. If it is missing, build it once and show it to the user to confirm before the first standup:

- Identity: `git config user.name` and `user.email`, `gh api user -q .login`, Linear's `me`, and the user's Slack name (the Slack MCP's whoami or profile).
- Repos: local clones whose `origin` is in the `Altid-Hjem-Aps` GitHub org. Find them with `find ~ /data -maxdepth 4 -name .git -type d 2>/dev/null` and check each remote. Skip extra worktrees of a repo already listed (same remote).
- Orchestrated repos: repos where bot merges count as the user's own work (for example Homebase for whoever runs it). Ask; default none.
- Slack: which channels to skim per workspace. Default #engineering and #core-team in Altid Hjem, #engineering in the other workspaces the user is in.

```markdown
Name: <name>  Git: <name/email>  GitHub: <login>  Slack: <name>
Repos: <path> (<owner/repo>), ...
Orchestrated: <repo> or none
Slack: <workspace>: <channels>; <workspace>: <channels>
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
5. Slack (whichever Slack MCP is connected), every workspace the user is in (Altid Hjem, Altid Mad, Altid Forsikring), since the window: the configured channels, plus messages mentioning the user and their DMs (search `to:me` and their @name). Look for things blocking them, waiting on them, or asks they sent (those go in Carry over). Skip a workspace that is down and say so in one line.
6. Claude Code sessions: the user's main session logs since the window, skipping subagent logs: `find ~/.claude/projects -maxdepth 2 -name '*.jsonl' -newermt <window>`. From each, read only the user's own prompts with `jq -r 'select(.type=="user" and (.message.content|type)=="string") | .timestamp + " " + .cwd + " " + (.message.content|.[0:200])'`, dropping lines that start with `<` (tool and system noise). This catches work that never became a commit and work in repos not in the config. Summarise per topic; never quote prompts in the output.
7. Unfinished local work, per configured repo: `git status --short`, branches with commits not on any remote (`git log --branches --not --remotes --oneline`), and `git worktree list`. These go in Carry over, and in Today only if they belong to the sprint.
8. Linear activity that is not code: issues the user created, closed or commented on since the window (`list_issues` with `updatedAt`, then check whether the user made the change). Closing an issue or settling a decision is Yesterday work even with no commit.
9. Calendar, optional: if a calendar integration is connected and authenticated, today's meetings go under Today (one bullet, e.g. "Demo 10:00, roadmap meeting 14:00"). If none is connected, skip it without comment.

If a source fails (Linear auth, gh, Slack, session logs), say which one in the output. Don't silently report less.

## Blockers

Only things where someone other than the user has to act: a review, an answer, access, a vendor, a decision. Name the person. Things the user has to do themselves are Today, not blockers.

## Output

Write in the language the user asked in. Three short sections as bullet lists, one short bullet per item, no more than 12 bullets total:

```
Igår:
- <broad item>
I dag:
- <most urgent open item, in plain words, next step> (ALT-123: <link>)
- ... (max 5)
- +N more in the sprint
Blockers:
- <person: what> (or "- ingen")
```

(English: Yesterday / Today / Blockers.) It is the user's own standup: Yesterday and Today hold only what they did or will work on themselves (their commits, reviews and fixes, their orchestrated repos, issues assigned to them). Other people's work is left out unless the user did part of it, or it blocks them, and then it goes under Blockers. Each bullet must make sense to someone who has forgotten the issue: what it is, why it matters (one clause), and the concrete step today or what it waits on. "Fix build on 8/10" fails; "Decide at Thor's Mad-testen readout which fixes go into the build we submit to Apple" passes. To get there, read the issue description, not just its title (titles carry dates and jargon, descriptions carry the why and the plan). Today is not your pick of what the user should do; it is their open work in order of urgency, at most 5 bullets, then one line "+N more in the sprint" if there are more. Candidates: open issues assigned to them in the current cycle, plus Carry over. Blocked ones go under Blockers instead. Order by: a deadline or launch gate it feeds (nearest first, read from the issue: due date, dates in the title or description, "must land before X"), then Linear priority, then whether someone else waits on it, then started before unstarted. Each bullet ends with what the next step is, or "not started". Say what each item is in plain words ("the Altid Mad-only login token"): the team is AI-driven and nobody remembers what most ALT numbers are, so a bare id is never enough. When an item has an issue, put it after the words in parentheses with its link: "The Altid Mad-only login token (ALT-379: https://linear.app/altid/issue/ALT-379)". One issue per bullet: never combine two issues in one line; split them. No PR numbers, tables, bold or file paths in the output. The journal keeps the ids, since the next session needs them to look things up. Pipeline merges get one line at most ("Alti merged 4 small fixes"). After the output, write the journal entry, then stop.

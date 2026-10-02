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
## 2026-10-01 09:15
Done: <1-3 broad lines>
Today: <1-3 lines, ALT-ids where they exist>
Blockers: <who, what> or none
Carry over: <open threads the next session needs to keep working: half-done branches, PRs waiting on review, decisions pending, things promised to someone>
```

The last entry sets the window: "since" = its date and time (the heading carries both; write the current local time when you write an entry). A second run on the same day therefore only looks at what happened since the first. If an old entry has no time, use its date. No entry at all: the last working day (Monday reaches back to Friday; no standups on weekends). Its Carry over is the first thing to check: resolved, still open, or gone stale. Keep the last 10 entries; delete older ones. A second run on the same day adds a new entry; it does not replace the earlier one.

## Gather (in parallel)

1. Vision, cheap and once: the Overview in the app repo's `CLAUDE.md`, the current cycle's description, and the newest project status updates (`get_status_updates`, type project). Enough to know what the sprint is for. Don't read specs.
2. Linear, team Altid Hjem:
   - `list_cycles` type current (teamId must be the UUID `aaff3d60-51d8-492a-90a7-7342996e5de3`; the name fails), then `list_issues` with that cycle and `assignee: "me"`. This is the source for Today.
   - The whole cycle, once, for context and blockers.
   - Issues labelled `blocked`, and issues where Alti or someone else asked the user a question (recent comments mentioning them).
3. Git, for each configured repo: `git fetch --quiet`, then `git log --all --since=<window> --format='%h %an %ae %s'`. The user's commits match their git name or email. `altid-alti[bot]` is the pipeline, not the user, except in orchestrated repos, where everything merged counts as the user's.
4. GitHub, per repo: `gh pr list --state merged --search "merged:>=<date>"` and `gh pr list --state open`. PRs the user authored or reviewed count as their work; their open PRs waiting on someone else's review or action are blocker candidates.
5. Slack (whichever Slack MCP is connected), every workspace the user is in (Altid Hjem, Altid Mad, Altid Forsikring), since the window: the configured channels, plus messages mentioning the user and their DMs (search `to:me` and their @name; DM channels can't be opened with read_channel, so work from the search snippets). Look for things blocking them, waiting on them, or asks they sent (those go in Carry over). Skip a workspace that is down and say so in one line.
6. Claude Code sessions: the user's main session logs since the window, skipping subagent logs: `find ~/.claude/projects -maxdepth 2 -name '*.jsonl' -newermt <window>`. From each, read only the user's own prompts with `jq -r 'select(.type=="user" and .isMeta != true and .isCompactSummary != true and (.message.content|type)=="string") | .timestamp + " " + .cwd + " " + (.message.content|.[0:200])'`, dropping lines that start with `<` (tool and command output) or with "This session is being continued" (compaction summaries). This catches work that never became a commit and work in repos not in the config. Summarise per topic; never quote prompts in the output.
7. Unfinished local work, per configured repo: `git status --short`, branches with commits not on any remote (`git log --branches --not --remotes --oneline`), and `git worktree list`. Housekeeping (stale worktrees, stray files) goes in the journal's Carry over only, never in the output. A half-done branch for a sprint issue just marks that issue as started.
8. Linear activity that is not code: issues the user created, closed or commented on since the window (`list_issues` with `updatedAt`, then check whether the user made the change). Closing an issue or settling a decision is Yesterday work even with no commit.
9. Calendar, optional: if a calendar integration is connected and authenticated, today's meetings go under Today (one bullet, e.g. "Demo 10:00, roadmap meeting 14:00"). If none is connected, skip it without comment.

If a source fails (Linear auth, gh, Slack, session logs), say which one in the output. Don't silently report less.

## Blockers

Only things that are stuck right now because someone other than the user has to act: a review, an answer, access, a vendor, a decision. Name the person. Something due later (a readout due today, a review requested an hour ago) is not stuck yet. Things the user has to do themselves are Today, not blockers.

## Output

Write in Danish, with æ, ø and å, whatever language the user asked in: the standup is held in Danish. Only the Linear priority prefixes stay in English.

```
Igår:
- <outcome>

I dag:
- <promise to a person, no issue>: <what> (lovet <person>, <where>)
- URGENT <what, next step> (ALT-123: <link>)
- HIGH <...>
- +N mere i sprinten (<what they are>)

Blockers:
- <person: what> (or "- ingen")
```

### What earns a place

A standup tells the team what moved, what is next and where you are stuck. Before writing a bullet, ask: would a teammate care, or does it change what someone does? Outcomes do (a fix shipped, a decision made, an issue closed, an ask sent to another team). Housekeeping does not (worktree cleanup, uncommitted notes, renamed flags, internal tooling tweaks, building this skill), unless it changes how the team works. When in doubt, leave it out; the journal keeps everything.

### Keep it short

- One line per bullet, about 15 words before the link. Cut every word that doesn't change the meaning.
- Igår: at most 3 bullets. Group related work into one outcome ("Homebase deployer nu sig selv og laver kortere PR'er"), not a changelog.
- I dag: every URGENT and HIGH issue gets a bullet; MEDIUM, LOW and NONE fold into "+N mere i sprinten (<few words>)". Each bullet: what it is in plain words, then the next step or "ikke startet". Add the why only when the what doesn't carry it.
- Blockers: one line each, person first.
- One blank line between the three sections.

### Rules

- It is the user's own standup. Igår and I dag hold only what they did or will work on (their commits, reviews and fixes, their orchestrated repos, issues assigned to them). Other people's work appears only under Blockers.
- Only state what you read. Every claim about who did what, or why, must come from something you saw this run (a commit, a comment, a message). If you can't point to it, leave it out.
- Read the issue description, not just the title: titles carry dates and jargon, descriptions carry the why and the plan.
- I dag lists the user's open issues in the current cycle plus Carry over, sorted by Linear priority (prefix in English capitals: URGENT, HIGH, MEDIUM, LOW, NONE), then nearest deadline or launch gate, then whether someone waits on it, then started before unstarted. You don't pick; the user does. Blocked issues go under Blockers instead. Promises the user made to a person with no Linear issue ("jeg laver lige..." in Slack) go first, without a priority prefix, naming who waits: someone is waiting on them.
- Plain words, never a bare id: the team is AI-driven and nobody remembers ALT numbers. One issue per bullet, with "(ALT-123: https://linear.app/altid/issue/ALT-123)" after the words. No PR numbers, tables, bold or file paths.
- Pipeline merges outside orchestrated repos get one line at most ("Alti mergede 4 små rettelser").

After the output, write the journal entry (it keeps the ids and the housekeeping), then stop.

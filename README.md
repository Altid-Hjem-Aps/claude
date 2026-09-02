# claude

Personal Claude Code configuration. `CLAUDE.md` holds the workflow guidelines
Claude follows in every session: communication rules, hard rules, working style,
and coding conventions.

## Who this is for

Software engineers and architects who use Claude Code and want to steer its
behavior with explicit, version-controlled conventions — rather than re-explaining
preferences each session. Use it as a starting point and adapt the guidelines to
your own standards.

## Install

Run the script for your platform from the repo root — it copies `CLAUDE.md` and
the skills into `~/.claude` (backing up any existing `CLAUDE.md` first):

- **Windows** — `powershell -ExecutionPolicy Bypass -File install.ps1`
- **macOS / Linux** — `./install.sh`

### Where it goes

Everything lands in the Claude Code config folder in your home directory so it
applies to every session:

- **Windows** — `C:\Users\<you>\.claude\`
- **macOS** — `/Users/<you>/.claude/`
- **Linux** — `/home/<you>/.claude/`

## License

[MIT](LICENSE) © Lars Støttrup Nielsen

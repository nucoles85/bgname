# Using bgname with Claude Code

Add this to `~/.claude/CLAUDE.md` (applies to every project) or a project's `CLAUDE.md`:

```markdown
## macOS background jobs (launchd)

Whenever you create or modify a launchd LaunchAgent/LaunchDaemon, the job must show up in
System Settings > Login Items & Extensions > Background App Activity under a name that says what
it does — never "bash", "python3", "node", etc.

- Create new jobs with `bgname create --label <reverse-dns label> --name "<Descriptive Name>" [--interval N | --at HH:MM] [--keep-alive] [--log FILE] -- <command...>`.
- If a job was created another way, immediately run `bgname rename <label> "<Descriptive Name>"`.
- Names: short Title Case describing the purpose, prefixed with the project when helpful
  (e.g. "Nightly Backup", "Router Autoheal").
- System-level jobs (/Library/...) need sudo — write the exact `sudo bgname ...` command for the user to run.
- Verify with `bgname list` and report the name the job now appears under.
```

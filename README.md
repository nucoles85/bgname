# bgname

**Give macOS background items meaningful names.**

System Settings → General → Login Items & Extensions → **Background App Activity** labels each
launchd job with the file name of the program it starts. Any job that runs a script through an
interpreter shows up with a useless name:

| Before | After `bgname` |
|---|---|
| bash | Cloudflare Tunnel Monitor |
| bash | Router Autoheal |
| python3 | Menu Bar Widget |
| python3 | Dashboard Server |

`bgname` points the job at a tiny launcher named after what the job does. The launcher just
`exec`s the original command, so the job's behavior, schedule, logs, and environment are unchanged.

```bash
#!/bin/bash
# Launcher created by bgname so macOS Background App Activity shows a meaningful name.
# bgname-label: com.example.dashboard
# Undo with: bgname restore com.example.dashboard
exec /usr/bin/python3 /Users/me/dashboard/server.py "$@"
```

## Install

```bash
git clone https://github.com/nucoles85/bgname.git
cd bgname
install -m 755 bgname /usr/local/bin/bgname   # or symlink it into any folder on your PATH
```

No dependencies — just `bash`, `plutil` and `launchctl`, which ship with macOS.

## Usage

```bash
bgname list                                    # jobs showing as bash/python3/node/... (and ones already renamed)
bgname list --all                              # every launchd job
bgname rename com.example.dashboard "Dashboard Server" --dry-run
bgname rename com.example.dashboard "Dashboard Server"
bgname rename com.example.dashboard "Dashboard"  # rename again
bgname restore com.example.dashboard           # undo: original plist back, launcher removed
```

`rename` and `restore` accept a job label or a path to its `.plist`.

### Creating new jobs with a good name from the start

```bash
bgname create --label com.example.backup --name "Nightly Backup" \
  --at 02:00 --log ~/Library/Logs/backup.log -- /bin/bash ~/scripts/backup.sh

bgname create --label com.example.api --name "Local API Server" \
  --keep-alive --workdir ~/api -- python3 server.py
```

| Option | launchd key |
|---|---|
| `--interval <seconds>` | `StartInterval` |
| `--at HH:MM` (repeatable) | `StartCalendarInterval` |
| `--keep-alive` | `KeepAlive` |
| `--no-run-at-load` | omits `RunAtLoad` (default is to run at load) |
| `--workdir <dir>` | `WorkingDirectory` |
| `--log <file>` | `StandardOutPath` + `StandardErrorPath` |
| `--system` | create a LaunchDaemon in `/Library/LaunchDaemons` (runs as root; needs `sudo`) |

## Where things go

| Job location | Launcher folder | Needs sudo |
|---|---|---|
| `~/Library/LaunchAgents` | `~/Library/Scripts/Background Jobs/` | no |
| `/Library/LaunchAgents`, `/Library/LaunchDaemons` | `/usr/local/libexec/Background Jobs/` (root-owned) | yes |

Before the first rename, the original plist is copied to a `.backups/` folder inside the
launcher folder. Later renames never overwrite that backup, so `restore` always goes back to
the true original. Jobs that were loaded are reloaded; unloaded jobs stay unloaded.

## Caveats

- **Don't move or delete the launcher folder** — the jobs point at those files.
- **Third-party jobs:** you *can* rename them, but app updates (and `brew services`) may rewrite
  their plists, and some privileged helpers verify their own code signature. `bgname` is aimed at
  jobs you own; for vendor jobs, it's usually better to leave them alone.
- macOS may keep showing old entries until you reopen System Settings, and may post a
  "Background Items Added" notification for each renamed job. Both are normal.
- The name must not contain `/`, quotes, or backslashes, or start with a dot.

## Using with Claude Code (or other AI coding agents)

Agents that set up background jobs for you tend to produce a pile of `bash` and `python3`
entries. See [docs/claude-code.md](docs/claude-code.md) for a snippet you can drop into
`~/.claude/CLAUDE.md` so every job an agent creates gets a meaningful name.

## License

[MIT](LICENSE)

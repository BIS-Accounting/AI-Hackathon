# Installing switchboard

`swb` is a single-file Node CLI plus three Claude Code hooks. It keeps a team
in sync on a Linear board straight from the terminal — claim a ticket, work it,
mark it done — and quietly surfaces what teammates are doing while you type.

**Requirements:** Node 18 or newer. Nothing else — zero npm dependencies.

Check your Node:

```sh
node --version   # must print v18.x or higher
```

---

## One-line install

### macOS / Linux

```sh
git clone https://github.com/BIS-Accounting/AI-Hackathon.git && cd AI-Hackathon && node install.js
```

### Windows (PowerShell)

```powershell
git clone https://github.com/BIS-Accounting/AI-Hackathon.git; cd AI-Hackathon; node install.js
```

### Before you start — two things you need

1. **Accept the Linear invite.** Check your email for an invite to the **BIS Agents**
   Linear workspace (from Turni), then join your team — Team 1 – Accounting
   (key `TEA`) or Team 2 – Accounting (key `TEA2`). Accept it before anything
   else — your API key is scoped to your membership, so no invite = no working key.
2. **Mint your personal API key.** In Linear: click your workspace name →
   **Settings → Security & access → Personal API keys → New API key**. Name it
   `switchboard`, copy the `lin_api_…` value somewhere safe — the installer asks
   for it. **Every person needs their OWN key** — a shared key breaks claim-race
   protection, @you routing, and attribution (the board would show one person
   doing everything).

The installer walks eight steps and is **safe to run twice** (it never duplicates
hooks or overwrites your settings):

1. verifies Node ≥ 18
2. creates the `~/.switchboard/` state tree
3. asks for (or accepts) your `LINEAR_API_KEY` and writes `~/.switchboard/env` (mode `600` on macOS/Linux)
4. registers the three Claude Code hooks by **merging** into `~/.claude/settings.json` (your existing hooks are backed up to `settings.json.swb-bak` and left untouched)
5. registers the three **Business Central** MCP servers (`mcp/business-central.json`: BIS Inc., 2136254 Alberta Ltd., BIS Safety Software USA Inc. — Test-Backup environment) user-wide by **merging** into `~/.claude.json` (backed up to `.claude.json.swb-bak`; a server name already present is never overwritten)
6. drops a `swb` shim on your `PATH`
7. installs the `/swb-tour` command into `~/.claude/commands/`
8. runs `swb doctor` and prints the result

**First thing it does** (before step 1): opens `FLOOR-TOUR.html` in your browser — the click-through of the whole system. Read it while the installer works (skip with `--no-open`). The reference `PLAYBOOK.html` path is printed at the end.

### After installing: sign in to Business Central (once)

The installer registers the servers; only you can sign in. Open a **new** Claude
Code session, type `/mcp`, pick `business-central-bis-inc` → **Authenticate**, and
sign in with your BIS Microsoft account in the browser tab that opens. Repeat for
`business-central-alberta` and `business-central-usa`. Prove it: ask Claude
*"list the Business Central companies you can see"* — a real answer means you're in.
Tokens stay on your machine. If the installer could not touch `~/.claude.json` it
prints three `claude mcp add-json … -s user` lines — run those first.

### After installing: take the tour

Open a Claude Code session and type `/swb-tour`. Your own Claude walks you through
the real loop on a disposable practice ticket — create → promote → claim → digest →
done — in about 10 minutes. Then grab a teammate who has also installed and run
**Part 2** together: that's where the multiplayer shows up (their claim arriving in
your digest, the `@you` Q&A round-trip, the claim race). Solo, the digest is
deliberately quiet — it only ever shows what OTHER people do.

### Providing the key without a prompt

```sh
node install.js --key lin_api_XXXXXXXX          # pass it directly (always saved)
LINEAR_API_KEY=lin_api_XXXXXXXX node install.js  # or via env (seeds only; see below)
node install.js --no-prompt                      # skip the key for now, add it later
```

Get a key from Linear → **Settings → API → Personal API keys**. It looks like
`lin_api_...`.

**Key-overwrite rule (safe by default):** re-running the installer never
silently replaces a key you already saved. A `LINEAR_API_KEY` exported in your
shell only **seeds** the saved key when none exists yet — on its own it will
**not** clobber an existing one. To deliberately change a saved key, pass a new
value with `--key` (always replaces), or pass `--force` to let an exported
`LINEAR_API_KEY` replace it.

### Finishing the PATH setup

The installer creates the shim but can't edit your shell profile for you. If it
warns that `~/.local/bin` (or `%USERPROFILE%\.local\bin`) is not on your `PATH`:

**macOS / Linux** — add to `~/.zshrc` or `~/.bashrc`, then open a new terminal:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

**Windows (PowerShell)** — run once, then open a new PowerShell window:

```powershell
[Environment]::SetEnvironmentVariable("Path", "$env:Path;$env:USERPROFILE\.local\bin", "User")
```

Confirm it worked:

```sh
swb doctor
```

A green `swb doctor` means the key is valid, your team resolves, and all five
Linear workflow states exist. If a state is missing, run `swb doctor --fix`.

---

## 60-second quickstart: claim → work → done

Run these from inside a git repo that has a `.swb.json` (see below) — for the
hackathon that is your team's shared repo. Every command is real — nothing here
is a placeholder. You never type these yourself: your agent runs them and you
approve.

**1. See what's happening on the board (5s)**

```sh
swb sync
```

Prints a short digest of new comments, claims, and state changes since you last
looked. Empty board? It prints nothing.

**2. Claim a ticket and the files you'll touch (10s)**

```sh
swb claim TEA-42 --files "eft/**,reconciliation/moneris/**"
```

This assigns the ticket to you, moves it to **In Progress**, creates a side
folder for that ticket — git calls it a worktree — at `../switchboard-wt/TEA-42`
so two tickets never step on each other, and posts a comment listing your files.
If someone already holds it, `swb` backs off and tells you.

**3. Do the work.** Open a Claude Code session in the ticket's side folder. The
hooks now feed your agent the board digest as you go and warn it before editing
a file another teammate has claimed — no action needed on your part.

Found something worth sharing mid-task?

```sh
swb discover "Moneris settlement report lags the bank by one business day"
```

**4. Mark it done (10s)**

```sh
swb done TEA-42 --summary "EFT file builder, tested on the May batch"
swb done TEA-42 --link https://…/sharepoint/eft-checklist.xlsx --summary "…"   # --link only if the deliverable lives outside the repo
```

`swb` checks that the ticket's work is committed and pushed to GitHub first, and
**refuses** if anything is uncommitted or unpushed (your agent commits and
pushes for you before running it). Once pushed, it moves the ticket to
**In Review** and posts a summary comment; a reviewer moves it to Done after
looking at it. Done is a gate, not a status: your agent cannot mark a ticket
done until the work is actually saved to GitHub.

That's the loop: `sync` → `claim` → work → `done`.

### The repo config: `.swb.json`

Each repo that uses `swb` needs a `.swb.json` at its root:

```json
{ "teamKey": "TEA", "defaultBranch": "main" }
```

- `teamKey` — the Linear team every query and mutation is scoped to (`TEA` for
  Team 1 – Accounting, `TEA2` for Team 2 – Accounting).
- `defaultBranch` — used when creating the ticket's side folder.

Without a resolvable team (`.swb.json` `teamKey` or the `SWB_TEAM_KEY` env var),
`swb` refuses to run.

### Relocating the state directory: `SWITCHBOARD_HOME`

By default all switchboard state and config lives under `~/.switchboard/`. To put
it somewhere else — a different disk, a per-project sandbox, CI — set
`SWITCHBOARD_HOME` to the directory you want. `swb`, all three hooks, and the
installer's `swb doctor` step all honor it. The installer **never sets this for
you**; export it yourself before running commands:

```sh
export SWITCHBOARD_HOME="$HOME/.config/switchboard"
```

---

## Uninstall

Uninstalling is fully manual and reversible — the installer only ever *added*
things, and backed up whatever it touched.

**1. Restore your Claude Code settings.** The installer saved your original file
before merging:

```sh
# macOS / Linux
mv ~/.claude/settings.json.swb-bak ~/.claude/settings.json
```

```powershell
# Windows (PowerShell)
Move-Item "$env:USERPROFILE\.claude\settings.json.swb-bak" "$env:USERPROFILE\.claude\settings.json" -Force
```

If there is **no** `.swb-bak` file (you had no settings before installing, or
you've since made other edits you want to keep), just delete the three
switchboard hook groups by hand — search `~/.claude/settings.json` for
`hooks/userpromptsubmit.js`, `hooks/posttooluse.js`, and `hooks/pretooluse.js`
and remove those group objects.

**2. Remove the shim.**

```sh
# macOS / Linux
rm ~/.local/bin/swb
```

```powershell
# Windows (PowerShell)
Remove-Item "$env:USERPROFILE\.local\bin\swb.cmd"
```

**3. Remove the Business Central MCP servers** (they live in `~/.claude.json`,
user scope — Claude Code's own CLI removes them cleanly; sign-in tokens go with them):

```sh
claude mcp remove business-central-bis-inc -s user
claude mcp remove business-central-alberta -s user
claude mcp remove business-central-usa -s user
```

**4. Remove the state and config.** This deletes your saved key, cache, and
event log — do it last.

```sh
# macOS / Linux
rm -rf ~/.switchboard
```

```powershell
# Windows (PowerShell)
Remove-Item -Recurse -Force "$env:USERPROFILE\.switchboard"
```

**5. (Optional) Remove the clone and any leftover ticket side folders (worktrees).**

```sh
rm -rf /path/to/switchboard ../switchboard-wt
```

That's a complete removal — no daemons, no registry keys, no global npm
packages to hunt down.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `swb: command not found` | `~/.local/bin` isn't on `PATH` — see [Finishing the PATH setup](#finishing-the-path-setup), or run via `node /path/to/switchboard/swb.js`. |
| `swb doctor` says a workflow state is missing | Run `swb doctor --fix` to create the missing Linear states. |
| `LINEAR_API_KEY missing` | Add `LINEAR_API_KEY=lin_api_...` to `~/.switchboard/env`, or re-run `node install.js --key ...`. |
| `no team resolved` | Add a `.swb.json` with `"teamKey"` to your repo, or set `SWB_TEAM_KEY` in `~/.switchboard/env`. |
| Installer says `settings.json` is not valid JSON | It left your file untouched and saved a copy at `settings.json.swb-unparseable`. Fix the JSON, then re-run. |

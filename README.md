# Switchboard — Accounting

## START HERE — in this order, nothing else first

1. **Join your Linear team.** Accept the workspace invite from your email, then open
   your team in Linear (Team 1 – Accounting or Team 2 – Accounting) and confirm
   you're a MEMBER of it (invite accepted ≠ team joined — the kit half-works until
   you join).
2. **Run one line** (your Desktop is fine):
   ```sh
   git clone https://github.com/BIS-Safety-Software-Inc/switchboard-accounting.git && cd switchboard-accounting && node install.js
   ```
   The Floor Tour opens in your browser — read it while the installer works. It will
   ask for your personal Linear API key and shows you exactly where to get one, and
   it wires up **Business Central** for your Claude (you sign in once, with your BIS
   Microsoft account, when the installer tells you to type `/mcp`).
   *Prefer being guided? After cloning, just open Claude Code inside the folder and
   say "set me up" — it reads this repo's CLAUDE.md and walks you through everything.*
3. **Open Claude Code and type `/swb-tour`.** Your own Claude walks you through the
   real thing on a practice ticket (~10 min), then tells you to grab a teammate for
   Part 2 — **setup isn't done until you've run Part 2 with a buddy.**
4. **Then, and only then, read [PLANNING-DAY.html](./PLANNING-DAY.html)** — the
   questions your team answers on planning day. Read individually; decide as a team
   ON planning day, not before.

Don't read around this repo first — steps 2 and 3 open everything you need, in order.

---

**A coordination layer that turns Linear into shared memory for a whole
accounting team's Claude Code sessions.** The terminal is the interface, Linear is
the hub, humans are the routing layer.

A team of accountants each driving an agent in their own terminal are islands:
what one person learned about the Moneris report dies in their chat window, claims
go unseen, questions sit unread, and two agents quietly build the same
reconciliation two different ways. Switchboard is the patch panel between the
islands — a single-file CLI (`swb`) plus three Claude Code hooks that hand your
agent "what changed since your last look" on every message.

You speak English; your agent speaks `swb`; a digest keeps everyone current
without anyone opening Linear.

## Install

Node ≥ 18. Zero npm dependencies. Cross-platform (macOS / Linux / Windows).

```sh
git clone https://github.com/BIS-Safety-Software-Inc/switchboard-accounting.git && cd switchboard-accounting && node install.js
```

The installer verifies Node, creates `~/.switchboard/`, saves your
`LINEAR_API_KEY`, merges the three hooks into `~/.claude/settings.json` (backing
your original up first), registers the three Business Central MCP servers
user-wide (from `mcp/business-central.json`, into `~/.claude.json`), drops a `swb`
shim on your `PATH`, and runs `swb doctor`. Business Central needs a one-time
sign-in afterwards: type `/mcp` in Claude Code and authenticate each server with
your BIS Microsoft account. It is safe to run twice. Full details, PowerShell commands, and
uninstall steps are in **[INSTALL.md](./INSTALL.md)**.

## Architecture (in 10 lines)

1. **Three planes:** read (the digest), write (the protocol), guard (collision safety).
2. **Read — daemonless.** A hook checks `cache.json` age; only if stale (>45s) it fetches the team's board inline (time-boxed 5s), then injects a delta digest.
3. **Cache + per-session cursor** live under `~/.switchboard/`; each session only sees what changed since *its* last look.
4. **Write — one CLI, one file.** All Linear mutations go through idempotent `swb` verbs (`claim`, `ask`, `done`, `discover`, `new`, `release`) — never raw API calls.
5. **Every write is signed** `🤖 Claude — via <human> · swb v1.0.0`; the human is always the assignee.
6. **Gates are structural:** `swb new` lands in Backlog (a human promotes to Todo); `swb done` refuses until the work is committed and pushed to GitHub (a reviewer moves it to Done).
7. **Guard — two layers:** a side folder for each claimed ticket — git calls it a worktree — so two tickets never step on each other, plus a `PreToolUse` hook that *warns* (never blocks) when you edit a file another ticket has claimed.
8. **Three hooks:** `UserPromptSubmit` (digest every prompt), `PostToolUse` (mid-run injection, throttled ≥5 min), `PreToolUse` (ownership guard on Edit/Write/MultiEdit).
9. **Fails open:** any verb that errors prints a numbered `MANUAL RECIPE:` a human can follow by hand — nobody is ever blocked on the tool.
10. **Everything is one readable file** (`swb.js`) anyone can patch live, plus `events.jsonl` logging every verb and hook for the post-hoc review.

```
swb.js                  # the entire CLI — one file, zero deps
hooks/                  # userpromptsubmit · posttooluse · pretooluse
install.js  INSTALL.md  # installer + install/uninstall docs
mcp/                    # business-central.json — the 3 Business Central MCP servers the installer registers
PLAYBOOK.html           # the participant playbook (rules, roles, walkthrough)
AGENTS-template.md      # drop-in AGENTS.md protocol for team repos
CONTRACTS.md            # the build contract — interfaces are law
test/                   # node --test unit + live round-trip suites
```

## Read this before the hackathon

- **[PLAYBOOK.html](./PLAYBOOK.html)** — the one document participants read: how
  it works in 60 seconds, MUSTs vs DEFAULTs, role relationships, the Q&A loop,
  goal-loop rules, AI-captain duties, the 15-minute planning-day walkthrough, and
  troubleshooting. One screenful per role.
- **[AGENTS-template.md](./AGENTS-template.md)** — paste into your team repo so
  your agents follow the protocol (and it survives context compaction).
- **[INSTALL.md](./INSTALL.md)** — install, PATH setup, `SWITCHBOARD_HOME`,
  uninstall.
- **[CONTRACTS.md](./CONTRACTS.md)** — the locked build contract (for builders).

## Quick loop

```sh
swb sync                                                        # what changed since your last look
swb claim TEA-42 --files "eft/**"                               # assign → In Progress → side folder → declare files
swb ask TEA-23 @sam "does the EFT file need the bank's 80-character header row?"   # signed question, lands in Sam's next digest
swb done TEA-42 --summary "EFT file builder, tested on May batch"   # refuses until the work is pushed to GitHub → In Review
```

You never type these — your agent does, as ordinary Bash calls you see in the
permission flow. Done is a gate, not a status: your agent cannot mark a ticket
done until the work is actually saved to GitHub.

---

Private project — managed via the `bis-new` dual-repo setup.
Canonical: `BIS-Safety-Software-Inc/switchboard-accounting` · mirror: `TurniSaha/switchboard-accounting` (auto-synced on push).

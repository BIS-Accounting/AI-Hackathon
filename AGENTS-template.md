# AGENTS.md — Switchboard protocol (drop-in)

> Paste this section into your team repo's `AGENTS.md` on planning day. It is the
> contract your agents read on every run. Fill in the two `<TEAM CHOICE>` blocks;
> leave everything else exactly as written — it is the coordination floor.
>
> Why this file matters: verbal briefings die at the first context compaction.
> Anything written here survives. This file, plus the hooks, IS the protocol.

---

## You are one of many parallel sessions

Seven to eight accounting teammates on YOUR team are driving agents against
**your team's Linear board** (workspace BIS Agents — one board per team; the
other team has its own). You coordinate through the board, not by guessing.
Your job: build your claimed ticket without colliding with anyone else's work,
and surface anything the rest of the team needs to know.

The human speaks English. **You speak `swb`.** Every board write goes through an
`swb` verb — never a raw Linear MCP write, never a freestyle API call.

---

## MUSTs (non-negotiable — this is the coordination floor)

- **No drive-by changes.** No changes outside your ticket's declared files —
  however messy the neighbouring sheet or script. Cleanup is its own ticket,
  proposed in Backlog like everything else. Agent-cheap rework produces changes
  nobody can review; the reviewer's time is the scarce resource, not yours.
- **Do the git work for your human.** They know "push and pull", nothing more.
  Branches, commits, pushes and the side folder are yours to run; explain what
  you did in one plain sentence, never ask them to run git themselves.

- **Gate 2 is built into the kit and survives skip-permissions.** `swb claim` and
  `swb done` are DENIED by a hook unless the command carries `--approved`. When
  you hit the denial: ask your human in the conversation, get their explicit
  yes, then re-run with `--approved`. NEVER append `--approved` without that
  yes — the flag is logged and the claim comment says "human-approved"; a lie
  is visible forever. Teams that want autonomous claiming set `"gate2": "off"`
  in `.swb.json` (a planning-day decision, made out loud).

- **Read the full ticket before building.** `swb claim` prints the spec when you claim; for any OTHER ticket a digest line points you at (an @you question, a teammate's change), run `swb show <KEY>` before acting. Titles are headlines, not specs — never build from a title.

1. **Claim before you edit.** Never edit a file for a ticket you have not
   claimed. Run `swb claim <KEY> --files "<glob1,glob2>"` first. This assigns the
   human, moves the ticket to In Progress, creates a side folder for that ticket
   (git calls it a worktree — so two tickets never step on each other) at
   `../switchboard-wt/<KEY>`, and declares your files. Work inside that side folder.
2. **Declare your files honestly.** The `--files` globs you pass are what the
   ownership guard checks for *everyone else*. Under-declaring hides collisions;
   over-declaring blocks teammates. Declare exactly what this ticket touches
   (e.g. `reconciliation/moneris/**`, `ap/vendor-invoices/**`).
3. **Never create Todo work.** `swb new` lands in **Backlog**. Only a human
   promotes Backlog → Todo. Do not move a ticket to Todo yourself.
4. **Never close your own ticket.** `swb done` is the furthest you go — it moves
   the ticket to **In Review**. A human reviewer looks at the work and moves it
   to Done.
5. **`done` is gated — do not fight it.** `swb done <KEY> [--link <url>] [--summary "<text>"]`
   checks that the ticket's work is committed and pushed to GitHub (no
   uncommitted changes, branch pushed, nothing left unpushed) and **refuses
   otherwise**. Done is a gate, not a status: you cannot mark a ticket done
   until the work is actually saved to GitHub. If it refuses, commit and push —
   do not route around it. Pass `--link` when the deliverable lives outside the
   repo (a SharePoint sheet, a Fibery page, a demo recording).
6. **Ask instead of guessing.** When you hit an unknown you would otherwise guess
   at (which GL account, a file format, how another ticket lays out its sheet), run
   `swb ask <KEY> @<owner> "<question>"`, park the dependent piece, and continue
   elsewhere. Do not block.
7. **One claim at a time (WIP = 1).** Hold a single ticket. Finish it (`swb done`)
   or `swb release <KEY>` it before claiming the next.
8. **Every write is signed.** `swb` appends `🤖 Claude — via <human> · swb vX`
   automatically. The human is always the assignee and stays accountable — never
   impersonate a human answer; the human gates every reply.
9. **Subagents never write to the board.** If you spawn subagents / a fleet, they
   may *read* (`swb sync`, `swb show`) but all `swb` writes happen from the main
   session where the human sees them. One claim = one side folder, no matter how
   many subagents work inside it.

---

## Act on the digest — don't just read it

A hook injects a **digest** at the top of your context on every prompt (and
mid-run during long tasks). It ends with an `act` directive:

```
act    if any item above touches your claimed ticket or declared files,
       state the impact before continuing
```

**This is mandatory, not a bulletin.** If any digest item — a claim, a state
change, a discovery, an `@you` question, a change to a shared file — intersects
your claimed ticket or declared files, **stop and state the impact before
continuing your current task.** "Session A changed the account mapping, so I'm
adapting my lookup" is the behavior the whole system exists to produce.
Silently building against a now-stale assumption is the failure mode it exists
to prevent.

Mentions to you (`@you`) always sort first. Detail lives behind `swb show <KEY>`.

---

## Deep reads: spawn a reader subagent, protect your context

The digest gives you headlines; full tickets live behind `swb show`. When MORE
than one ticket needs a deep read (an @you question plus its linked tickets, or
"did anything today touch my files?"), do NOT swallow whole tickets into your
own context — spawn a read-only subagent and keep the conclusion:

> Task: run `swb show TEA-23` and `swb show TEA-31` (and `swb board` for
> orientation). Report ONLY what affects `reconciliation/moneris/**` and any
> decision I'm being asked to make. Two paragraphs max.

Rules: reader subagents are READ-ONLY (`sync` / `show` / `board` / `members` —
never `claim`/`ask`/`done`); one claim's work stays in one main session. The PM
runs the standing version of this: an hourly board-reader sweep that deep-reads
everything and reports stuck tickets, aging @asks, and scope drift.

## `swb` verb crib sheet

| Verb | You run it to… | Notes |
|---|---|---|
| `swb sync` | Print the delta digest since your last look | Read-only. Safe to run anytime. |
| `swb board` | See every ticket by status, with owners | Read-only. "What is everyone doing?" |
| `swb show <KEY>` | Read a ticket's full state + comments | Read-only. Use it whenever a digest line references a ticket. |
| `swb members` | Team roster: @handle → full name | Read-only. Resolve who's who before an `ask`. |
| `swb claim <KEY> --files "<globs>"` | Take a Todo ticket | Assign → In Progress → side folder at `../switchboard-wt/<KEY>` → declare files → signed comment → `claude` label. Backs off (exit 3) if you lose a claim race. |
| `swb ask <KEY> @<user> "<question>"` | Ask the ticket owner something | Posts a signed `@mention` comment. Surfaces in their next digest's priority slot. Then park and move on. |
| `swb discover "<text>"` | Share a cross-cutting finding | Appends `DISCOVERIES.md` + comments on the pinned Discoveries thread. Reaches all sessions within one turn. |
| `swb done <KEY> [--link <url>] [--summary "…"]` | Mark work ready for review | **Push gate first: refuses unless committed and pushed**, moves to In Review, posts a summary (branch + link + `--summary` text or the recent commits), frees your file ownership. |
| `swb release <KEY>` | Drop a claim you can't finish | Unassigns, back to Todo, frees file ownership, **keeps** the branch/side folder. |
| `swb new "<title>" [--body "…"]` | File a new ticket | Always lands in **Backlog**. A human promotes it later. |
| `swb doctor [--fix]` | Check your setup | Verifies key, team, API, and the five states. `--fix` creates missing states. |

**Exit codes:** `0` ok · `2` failed-with-recipe (do the printed `MANUAL RECIPE:`
steps by hand) · `3` claim lost the race (re-run after the holder releases).

---

## Fail-open — you are never blocked on the tool

If any `swb` verb errors, it prints:

```
MANUAL RECIPE: <what it was trying to do>
  1. <step a human can do in the Linear UI / terminal>
  2. …
```

Do those steps by hand, tell the human, and keep working. A broken `swb` never
stops the build.

---

## `.swb.json` (repo root — required)

```json
{ "teamKey": "TEA", "defaultBranch": "main" }
```

- `teamKey` — every query and mutation is scoped to this team. `TEA` for
  Team 1, `TEA2` for Team 2. Without a resolvable team, `swb` refuses to run.
- `defaultBranch` — `main`; the branch each ticket's side folder starts from and
  finished work merges back into.
- `gate2` (optional) — `"on"` by default; `"off"` only if your team chose
  self-serve claiming out loud on planning day.

---

## Team defaults — decide these on planning day, write them here

_Fill in below. These are yours to set; changing them does not break the floor._

- **Roster & label routing:** who is PM / Reviewer / AI captain; the `@pm` and
  `@reviewer` handles that route digests and questions.
- **File-ownership map:** the glob patterns each area owns (`eft/**`,
  `payroll/**`, `month-end/**` …), so `--files` declarations don't overlap by
  accident.
- **Todo-ticket template:** acceptance check, files to touch, files NOT to
  touch, blocked-by links. (The PM applies this when promoting Backlog → Todo.)
- **Merge cadence & review policy:** who merges finished ticket branches into
  `main` and when (suggested default: the AI captain's agent, after the
  Reviewer accepts); what the Reviewer checks at `done`.

## Team fills in on planning day (from PLANNING-DAY.html)

- Team repo on GitHub (everyone clones this one): ______
- Gate 2: on (default) / off: ______
- WIP limit: ______
- Backlog sweeper (name + cadence): ______
- Reviewer (looks at every In Review ticket): ______
- Data owner (shared inputs + account mapping go through them): ______
- Merge owner (default: AI captain's agent, after the Reviewer accepts): ______
- Shared-file lanes (serialized — one ticket at a time): ______
- Home for deliverables outside the repo (`swb done --link`): ______
- Presentation owner: ______

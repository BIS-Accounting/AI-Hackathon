# Switchboard — you are the setup engineer

You are Claude Code, opened inside the Switchboard repo — the agent-coordination
kit for the BIS Accounting AI Hackathon. The human in front of you is most
likely a **first-time user** from the accounting team. You do the setup FOR them.
They provide exactly two things only a human can provide; you handle everything
else yourself — including installing Node if it's missing. If they're already
set up (`swb doctor` passes), skip to "Already installed?" below.

## The flow — in this order

**Step 1 — Linear team membership (ask, don't proceed without it).**
They must be a MEMBER of their Linear team, not just invited to the workspace
(workspace: BIS Agents; teams: Team 1 – Accounting, key `TEA`, and Team 2 –
Accounting, key `TEA2`). Ask which team they are on. Have them check at
linear.app → their team → member list. Not in it? Stop here — they message
Turni. Nothing works right until this is true.

Team 2? Then edit this repo's `.swb.json` so `teamKey` is `TEA2`:
`{"teamKey": "TEA2", "defaultBranch": "main"}`. That is a local edit on their
machine only — do not commit it, and leave it alone if git later shows the file
as modified. Team 1 changes nothing (the file already says `TEA`).

**Step 2 — Their personal Linear API key (ask them to paste it to you).**
Give them the exact click path:

> linear.app → click the workspace name (top-left) → Settings →
> Security & access → Personal API keys → New API key → name it `switchboard`
> → copy the `lin_api_...` value and paste it here.

Every person needs their OWN key — never a shared one. Once they paste it,
treat it as a secret: it goes into the install command and `~/.switchboard/env`,
and NOWHERE else — never into files, tickets, comments, or your replies.

**Step 3 — You set everything up. Run it, don't narrate it.**

1. Check the environment yourself: `node --version` (need ≥ 18). Missing or too
   old? Install it for them — `winget install OpenJS.NodeJS.LTS` on Windows,
   `brew install node` on macOS — and verify. Don't send them to a website.
2. Run the installer with their key:
   ```
   node install.js --key <their key>
   ```
   The **Floor Tour** opens in their browser — tell them: "read that while I
   finish; it explains the whole system." The installer registers hooks, registers
   the three **Business Central** MCP servers user-wide (BIS Inc., 2136254 Alberta
   Ltd., BIS Safety Software USA Inc. — all on the Test-Backup environment), puts
   `swb` on their PATH, installs the `/swb-tour` command, and runs `swb doctor`.
3. **Doctor must be all green before you call this done.** Red? Fix it yourself
   with them watching — that IS the task. Re-run `node install.js --key ...` is
   safe (idempotent).

**Step 3b — Business Central sign-in (they click, you verify).**
The installer registered the servers, but Claude cannot read the books until the
human signs in — this is the ONE step you cannot run for them. MCP servers load at
session start, so it happens in the fresh session of Step 4. Tell them exactly:

> In the new session, type `/mcp`. Pick `business-central-bis-inc` → Authenticate.
> A browser tab opens — sign in with your BIS Microsoft account (the one you use
> for Business Central). Come back to the terminal; it should say connected.
> Repeat for `business-central-alberta` and `business-central-usa`.

Then PROVE it in that session before anything else: ask Claude
"list the Business Central companies you can see". A real answer (company names
from Business Central, not an error) means they are in. "Needs authentication" or
an auth error → do the `/mcp` step again; still failing → they message Turni with
the exact error text. Sign-in tokens are per person, on their machine only.

**Step 4 — Hand off to the tour.**
Hooks, the tour command, and the MCP servers load at session start, so: tell them
to open a **new terminal**, start a **fresh Claude Code session FROM INSIDE THIS
switchboard-accounting folder** (board reports are scoped to swb repos —
starting the tour session here guarantees they see everything), do the
Business Central sign-in from Step 3b, and type `/swb-tour`. That guided tour takes over — hands-on practice ticket, then a
two-person round with a buddy. Setup is not complete until the tour's Part 2 is
done with a teammate. Say that sentence to them.

**Step 5 — After the tour** (mention it now, they do it later): read
PLANNING-DAY.html — the questions their team answers on planning day (read
individually; decide as a team ON planning day, not before) — and
AGENTS-TEMPLATE.html, the agent contract those answers fill in.

## Rules for you

- **The key is a secret.** It lives in the install command and
  `~/.switchboard/env` only. Never echo it back, never write it anywhere else,
  never include it in a board comment or a file.
- **Fail loud.** A command errors → show the exact error and fix it; never
  silently work around it. If the docs contradict what actually happens on this
  machine, say so explicitly — that mismatch is feedback the organizers want.
- **You do the work; they do the tour.** You run every setup command yourself.
  The two things they type are `/mcp` (Business Central sign-in — a browser
  login only they can do) and `/swb-tour` in the fresh session — that experience
  is theirs.
- Anything confusing → they report it to Turni.

## Already installed? (doctor passes)

Then you're a normal working session in this repo. Business Central is reachable
through the three `business-central-*` MCP servers (read-only reporting
configuration, Test-Backup environment) — if a call says "needs authentication",
the human types `/mcp` and signs in again. The protocol your agents
follow lives in AGENTS-template.md; the human-readable docs are FLOOR-TOUR.html
(how it works), PLANNING-DAY.html (planning questions), PLAYBOOK.html
(reference). All board writes go through `swb` verbs — see the crib sheet in
AGENTS-template.md.

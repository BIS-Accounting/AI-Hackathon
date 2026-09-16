---
name: acct-<case-slug>            # e.g. acct-contract-final-review
description: <one line: what this skill does and when to use it>
owner: <accountant>
validator: <accountant>
case: <# from the use-case sheet>
---

## When this runs
<The trigger, in the owner's words. "A signed contract PDF lands in the AR folder." "It is the 1st and the August billing run is done.">

## Inputs — exact locations
- <Dynamics record type / Power BI report / folder path / mailbox — one per line>

## Procedure
<Numbered steps from the walkthrough, in the order the owner actually does them. Each step says what to read, what to decide, what to write.>
1.
2.
3.

## Rules
<Every "it depends" from the walkthrough, written as: IF <condition> THEN <action>. Number them so the validator can cite a rule by number when reporting a miss.>
R1.
R2.
R3.

## Exceptions — stop and ask the owner
<The three to five things that usually go wrong. For each: how to recognise it, and that the answer is to stop and ask, not guess.>
-

## Output
<What Claude produces, in what format, saved where. Draft first. Nothing is posted or sent by this skill unless the "Posting" line below says so.>

## Posting
<"Never" or "Only after the validator has compared the draft to the real answer and said yes in the Linear ticket.">

## Never
- Post to Dynamics or send anything client-facing without the validator's yes.
- Invent a GL code, vendor, customer, or amount. If it is not in the inputs or the rules, ask.
- <owner adds>

## Got-it-right test
Real answer: <exact location of the cycle the owner performed by hand on Day 1>
Compare: <field by field / line by line / match rate>
A miss is: <owner's definition>

## Pilot
Monday owner: <name>. 30-day parallel run alongside the manual process. Retire the manual step only when the validator signs off.

---
DRAFTING PROMPT (paste into Claude Code at minute 25 of the walkthrough, with the transcript and the step table attached):

"Using SKILL-TEMPLATE.md as the structure, write SKILL.md for this process from the attached transcript and step table. Put every step in Procedure in the order performed. Turn every 'it depends' into a numbered IF/THEN rule. List every 'this usually goes wrong' as an exception. Where you had to guess at a step, a rule, or a location, mark the line with [GUESS] so the owner can correct it. Do not invent codes, names, or amounts. Keep it under two pages."

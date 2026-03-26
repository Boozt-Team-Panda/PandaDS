---
name: pre-session-planner
description: >
  Builds a structured, milestone-aware session agenda for Panda DS syncs.
  Use this skill 1–2 days before any design system meeting, or when the user
  says "prepare the next session", "build the agenda", "what should we cover
  next time", "plan the sync", "what's the agenda for tomorrow", or anything
  about scheduling or preparing for the next DS meeting. Always loads
  ds-roadmap first to enforce milestone gates — if Foundation is not complete,
  the agenda contains only Foundation items, no matter what else is in the
  backlog. Output is a structured agenda ready to paste into a meeting invite
  or Confluence page.
---
 
# pre-session-planner — Milestone-Aware Agenda Builder
 
This skill transforms the registry backlog into a focused, time-boxed session
agenda. It is opinionated: the milestone gate is enforced, sessions do not
sprawl, and every item has a clear purpose.
 
---
 
## Before Building
 
Always load first:
1. **`ds-roadmap`** — current milestone, priority algorithm, Foundation blockers
2. **Registry data** — open threads, conflicts, assumptions, deferred items
   from `panda-registry.json` or the current conversation context
 
---
 
## Step 1 — Check the Milestone Gate
 
Ask: **Is Milestone 1 (Foundation) complete?**
 
Foundation is complete when all of these are true:
- All primitive token categories have at least `draft` status on Web
- Semantic layer convention ratified by all platform reps (OPN-2-002 resolved)
- Export script has a named owner (OPN-2-004 resolved)
- Minimum sizing values agreed (OPN-3-001 resolved)
- Icon strategy decided (DEF-1-001 resolved)
 
**If Foundation is NOT complete:** The agenda contains only Foundation items.
Component, pattern, and template topics are blocked. Flag this explicitly
at the top of the agenda.
 
**If Foundation IS complete:** Move to Milestone 2 items. Update `ds-roadmap`
to reflect the milestone change.
 
---
 
## Step 2 — Collect Backlog Items
 
Pull from registry (or conversation context):
- All active `CFX-` items (unresolved conflicts)
- All active `OPN-` items with their session origin and priority
- All active `ASM-` items flagged as at-risk
- All `DEF-` items with a revisit condition that may now apply
- All `PRO-` items with `status: "proposed"` and `readiness: "ready"` —
  these are design-session proposals ready for dev ratification
 
**PRO- filtering rules:**
- Include only `readiness: "ready"` items — explorations and `needs-context`
  proposals stay out of the dev session agenda
- Exclude any PRO- with an active CFX- against it — it must be resolved
  in a design session before it reaches devs
- If Foundation is not complete, include only PRO- items whose layer is
  within Foundation scope (Tokens). Block all component/pattern PRO- items.
- If no PRO- items qualify after filtering, omit the PRO- section entirely
  from the agenda — do not add a placeholder or "no proposals" note unless
  the user explicitly asked about proposal status.
 
---
 
## Step 3 — Apply Priority Algorithm
 
Sort items using the `ds-roadmap` priority algorithm:
 
1. **Unresolved CFX-** — always first, they block downstream work
2. **Foundation blockers** — OPN- or DEF- that prevent Milestone 1 closing
3. **PRO- items ready for ratification** — design proposals cleared for dev
   review. Slot after blockers, before stale OPN-. These arrive prepared
   and focused — they should move fast in session.
4. **OPN- older than 2 sessions** — age signals a stuck decision
5. **OPN- from most recent session** — fresh but unresolved
6. **At-risk ASM-** — could invalidate completed work if wrong
7. **Low-priority OPN-** and **DEF-** — if time permits
 
---
 
## Step 4 — Time-Box the Agenda
 
Default session length: **60 minutes**
 
Time allocation per item type:
 
| Type | Time |
|------|------|
| CFX- resolution | 15–20 min |
| OPN- that needs a decision | 10–15 min |
| OPN- that needs input/proposals | 5–10 min |
| ASM- validation | 5–10 min |
| DEF- pickup | 5–10 min |
 
**If the backlog exceeds 60 minutes:** Split into two sessions. Put all CFX-
and Foundation blockers in Session A. Everything else in Session B.
Flag the split in the output.
 
---
 
## Output Format
 
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PANDA DS SYNC — AGENDA
[Session N] · [Proposed date or TBD] · ~60 min
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 
[If Foundation incomplete:]
⚠️ MILESTONE GATE ACTIVE
Milestone 1 (Foundation) is not complete. This session
covers Foundation items only. Remaining blockers:
· [List unclosed Foundation items]
 
──────────────────────────────────────────────────
AGENDA
──────────────────────────────────────────────────
 
[N min]  [ID]  [Topic]
         Type: [CONFLICT | OPEN | PROPOSAL | ASSUMPTION | DEFERRED]
         Goal: [What needs to happen for this item to close or advance]
         Prep: [What attendees should review or bring]
 
[For PRO- items, add:]
         Origin: Design session [SN] · [YYYY-MM-DD]
         Needs from devs: [needsFromDevs field from PRO- entry]
 
[Repeat per item]
 
──────────────────────────────────────────────────
IF TIME
──────────────────────────────────────────────────
 
[Lower-priority items if session runs under time]
 
──────────────────────────────────────────────────
NOT THIS SESSION
──────────────────────────────────────────────────
 
[Items explicitly excluded with reason — milestone gate, deprioritised,
or PRO- items blocked by active CFX-]
 
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ATTENDEES NEEDED
[List who must be present based on agenda items]
— e.g. Android rep needed for OPN-1-001 naming proposals
— For each PRO- on the agenda, check the `needsFromDevs` field and surface
  any platform-specific requirement here (e.g. "iOS rep needed for PRO-5-001
  — spacing scale requires native safe area confirmation")
 
PREP READING
[What to review before the session]
— e.g. Review DS-2-002 before the semantic layer ratification discussion
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
 
---
 
## Additional Outputs
 
After the agenda, optionally produce:
 
**Confluence-ready version** (if user asks): Same content, formatted as
a flat bulleted list for easy paste into a Confluence page.
 
**Slack announcement** (if user asks): A short 3–4 line message to post
in the DS channel announcing the session and linking to the agenda.
 
---
 
## Agenda Health Checks
 
Before finalising, verify:
- [ ] No component/pattern/template items if Foundation is incomplete
- [ ] All CFX- items appear before OPN- items
- [ ] PRO- items appear after CFX- and Foundation blockers, before stale OPN-
- [ ] No PRO- with an active CFX- appears in the agenda (move to NOT THIS SESSION)
- [ ] PRO- items include `Origin` and `Needs from devs` fields
- [ ] Time allocations sum to ≤ 60 min (or session is split)
- [ ] Each item has a clear Goal — not just a topic label
- [ ] Required attendees are identified for items that need specific reps
- [ ] At least one Foundation blocker appears if Milestone 1 is open

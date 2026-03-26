---
name: registry-analyst
description: >
  Analytical reasoning layer for the Panda DS registry. Use this skill
  whenever the user asks questions that require reasoning OVER the registry
  rather than just looking things up in it. Trigger on: "what's blocking us",
  "are we on track", "what's the critical path", "what changed since last
  session", "which assumptions are most at risk", "is the token layer ready
  to hand off", "what should we focus on", "give me a health check",
  "what's slowing us down", "are there circular dependencies", or any
  question where the answer requires synthesising multiple registry entries.
  Always loads ds-roadmap to evaluate findings against milestone targets.
  This is not a search tool — it is a reasoning tool. It answers questions
  the registry cannot answer on its own.
---
 
# registry-analyst — DS Registry Reasoning Layer
 
The registry stores what was decided. This skill reasons over what was decided
to surface things nobody would notice by scrolling through a table.
 
It answers: **what does the current state of Panda actually mean?**
 
---
 
## Before Analysing
 
Always load:
1. **`ds-roadmap`** — milestone targets, Foundation definition, priority algorithm
2. **Registry data** — from `panda-registry.json` or current conversation context
 
---
 
## Analysis Modes
 
The skill operates in one of five modes depending on what the user asks.
Claude selects the mode automatically — the user does not need to name it.
 
---
 
### Mode 1 — Health Check
 
Triggered by: "health check", "how are we doing", "give me a status",
"are we on track"
 
Produces a concise status report:
 
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PANDA DS — HEALTH CHECK
[Date] · [Session N context]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 
Milestone status:    [Milestone N — % complete estimate]
Blocking items:      [N CFX- unresolved · N Foundation OPN-]
Stale decisions:     [N DS- items in draft for >2 sessions]
At-risk assumptions: [N ASM- that could invalidate completed work]
Velocity (last 2 sessions): [Decisions made · OPN- resolved · OPN- added]
  Note: Count cross-functional sessions only for decision velocity.
  Design sessions contribute to proposal velocity (PRO- added / promoted),
  which is tracked separately in the PROPOSAL PIPELINE block.
 
PROPOSAL PIPELINE
→ [N] proposals in staging · [N] ready for dev review · [N] blocked by CFX-
→ Last design session: [SN · date] | Last cross-functional: [SN · date]
 
CRITICAL PATH
→ [The single most important thing to unblock right now, and why]
 
RISK FLAGS
· [Specific item] — [Why it's a risk]
· [Specific item] — [Why it's a risk]
 
WHAT'S GOING WELL
· [Item that's progressing cleanly]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
 
---
 
### Mode 2 — Critical Path
 
Triggered by: "critical path", "what's blocking us", "what do we need
to unblock", "what's the dependency chain"
 
Maps the blocking chain from current state to milestone completion:
 
1. Identify all items with `status: "unresolved"` (CFX-) or `status: "open"` (OPN-)
2. For each, check `blocks` field — what does it block?
3. Build the dependency chain: A blocks B, B blocks C → A is the root blocker
4. Surface root blockers first. These are the highest-leverage items.
5. Check `proposals` array for PRO- items with `status: "proposed"` that
   have not been ratified for 2+ sessions — these are stale proposals.
   A stale PRO- is not the same as a stale OPN- (different resolution path)
   but it signals a breakdown between design and dev handoff. Flag it
   separately with: "Proposal [PRO-N-NNN] has been in staging for [N] sessions
   without reaching a dev agenda. Check pre-session-planner or CFX- blockers."
 
Output format:
```
Root blocker → What it blocks → What that blocks → Milestone impact
[CFX-N-NNN] → [DS-N-NNN] → [Component layer] → Milestone 2 cannot start
[OPN-N-NNN] → [Token finalization] → [All component work] → Milestone 1 gap
 
STALE PROPOSALS
[PRO-N-NNN] — [N] sessions in staging · [Reason it hasn't reached devs or —]
```
 
---
 
### Mode 3 — Delta since last session
 
Triggered by: "what changed", "what's new since last time", "diff",
"what happened in the last session"
 
Compares the most recent session's entries against the session before it.
If the most recent session is a design session (`type: "design"`), compare
against the last cross-functional session AND the prior design session
separately — they serve different purposes.
 
- New DS- entries added
- OPN- items that were opened
- OPN- items that were resolved (status changed)
- CFX- items opened or resolved
- ASM- items that changed status
- DEF- items picked up or added
- PRO- items added to staging (design sessions)
- PRO- items promoted to DS- (ratified in cross-functional sessions)
- PRO- items blocked by new CFX-
 
Output format:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DELTA — [SN] ([type]) vs [SN-1] ([type])
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
New decisions:        +[N]  [IDs]
New open threads:     +[N]  [IDs]
Resolved:             +[N]  [IDs]
New conflicts:        +[N]  [IDs]
New proposals:        +[N]  [IDs]  (design sessions only)
Proposals promoted:   +[N]  [IDs → DS- IDs]
Net open threads:     [+N / -N]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
 
---
 
### Mode 4 — Assumption Risk Scan
 
Triggered by: "which assumptions are at risk", "could any assumption
invalidate our work", "ASM- review"
 
For each active ASM-:
1. Check if any subsequent decisions depend on it being true
2. Check if any OPN- or CFX- indirectly tests it
3. Rate risk: HIGH if it could invalidate a stable DS- entry, MED if it
   blocks future work, LOW if the downside is minor
 
Output: Ranked list with DS- entries at risk per assumption.
 
---
 
### Mode 5 — Readiness Assessment
 
Triggered by: "is [layer] ready", "can we hand off tokens", "are we ready
for components", "is Foundation done"
 
For a named layer or milestone:
1. List all required conditions from `ds-roadmap`
2. Check each against registry state
3. Output a checklist with current status per condition
4. Give a clear YES / NO / ALMOST with the specific gap if not ready
 
```
FOUNDATION READINESS
──────────────────────────────────────────
[ ] Primitive tokens — all categories     WEB: draft · iOS: tbd · Android: tbd
[ ] Semantic convention ratified          BLOCKED: OPN-2-002 open
[ ] Export script — named owner           BLOCKED: OPN-2-004 open
[ ] Minimum sizing agreed                 BLOCKED: OPN-3-001 open
[ ] Icon strategy decided                 BLOCKED: DEF-1-001 open
 
PROPOSAL PIPELINE
  [N] proposals ready for dev ratification
  [N] proposals blocked by active CFX-
  [N] explorations still in design phase
 
Verdict: NOT READY — 3 open blockers, 2 tbd platform coverages
Next action: Resolve OPN-2-002 (semantic ratification) — it unlocks the most
──────────────────────────────────────────
```
 
---
 
## Lookup Mode
 
For direct factual questions about past decisions:
 
| User asks | Claude does |
|-----------|-------------|
| "What did we decide about [topic]?" | Lists DS- entries tagged to that topic |
| "Is [topic] still open?" | Checks OPN- entries, reports status |
| "Did we discuss [topic]?" | Full search, cites session and ID |
| "What's unresolved?" | All active OPN- and CFX- |
| "Catch me up" | 1 paragraph per session, last 3 sessions |
 
Always cite session ID and date for every item returned in Lookup Mode.
 
---
 
## Output Principles
 
- Lead with the most important finding, not with the most recent
- Every finding cites the specific ID and session it comes from
- Never interpret a finding as a recommendation — surface it, state implications, let the team decide
- Flag when data is incomplete (e.g. S2 entries are unverified)
- Keep the output scannable — use the structured formats above, not prose paragraphs

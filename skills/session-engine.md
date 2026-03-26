---
name: session-engine
description: >
  Full-pipeline session processor for Panda DS. Use this skill the moment a
  meeting transcript is uploaded or pasted — PDF, TXT, DOCX, or raw text.
  Also trigger when the user says "process this session", "analyse the
  transcript", "extract decisions", "what did we decide", or pastes a block
  of meeting notes. This skill replaces both panda-ds-sessions and
  ds-sessions-destilation in a single pass: it filters noise, classifies
  every signal, detects contradictions against the registry, produces a
  human-readable session record for the team to verify, AND outputs a
  JSON patch ready to commit to GitHub — no second trigger needed.
  Always load ds-roadmap before running to ensure milestone-aware
  classification and correct priority assignment.
  Supports two modes: default (cross-functional dev sessions) and --design
  (designer-only working sessions between Facu and Justin). Use --design
  when the session has no developers present or when the user says
  "design session", "Justin and I", or "design-only".
---
 
# session-engine — Full-Pipeline Session Processor
 
One trigger. Complete output. Nothing left to do manually.
 
This skill processes a raw transcript end-to-end: noise filtering →
signal classification → contradiction detection → human record →
JSON patch. Both outputs are produced in a single pass.
 
Two modes are available. The correct mode is selected automatically or
can be forced with a flag.
 
---
 
## Mode Detection
 
Before doing anything else, determine which mode to run:
 
| Mode | When to use |
|------|-------------|
| **default** | Dev-inclusive sessions — developers, iOS, Android, Web present |
| **--design** | Designer-only sessions — Facu and/or Justin, no developers |
 
**Auto-detect signals for `--design`:**
- Attendees list contains only designers (Facu, Justin)
- User says "design session", "Justin and I", "our session", "design-only"
- No platform implementation discussed, only proposals and exploration
 
**Force with flag:** User can say `--design` explicitly at any point.
 
If mode is ambiguous, ask before proceeding:
> "Was this a design-only session or were developers present?"
 
Both modes share Steps 1, 2, 4, and 5 unchanged.
Step 3 (classification) and all outputs differ per mode — see below.
 
---
 
## Before Processing
 
Always load these before starting:
 
1. **`ds-roadmap`** — to know the current milestone, priority algorithm,
   and brand token strategy. Required for correct `audience` and `status`
   classification.
2. **Prior session context** — check the conversation for any existing
   session records or registry data to enable cross-session contradiction
   detection.
 
If neither is available, proceed but add this notice to the output:
> 📎 *ds-roadmap not loaded. Priority classification is best-effort.
> Cross-session contradiction detection limited to current session only.*
 
---
 
## Step 1 — Session Identification
 
Extract from transcript:
- **Date** — from transcript header or metadata. If missing, ask before proceeding.
- **Session number** — infer from prior sessions in context, or ask.
  Design sessions use the same counter as dev sessions — they are part of
  the same sequence. Never restart numbering.
- **Attendees** — named speakers only. Anonymised or unnamed voices → note as "unknown participant".
- **Topic** — infer from dominant discussion theme if not stated.
- **Mode** — label the record clearly as `DESIGN SESSION` or `CROSS-FUNCTIONAL SESSION`.
 
---
 
## Step 2 — Aggressive Noise Filtering
 
Discard without logging:
- Small talk, greetings, scheduling
- Technical issues (audio, screen share, tools)
- Off-topic discussions (unrelated products, personal anecdotes)
- Repeated content (same point made multiple times → keep only the clearest instance)
- Vague agreement ("yeah", "makes sense", "sounds good") without substance
 
**Keep only** content that affects design system decisions, structure,
naming, governance, process, or platform implementation.
 
---
 
## Step 3 (default mode) — Signal Classification
 
Every extracted item gets exactly one label:
 
| Label | ID prefix | Meaning |
|-------|-----------|---------|
| DECISION | DS- | Agreed by participants. Actionable. Permanent. |
| OPEN | OPN- | Discussed but unresolved. Needs follow-up. |
| CONFLICT | CFX- | Contradicts a prior decision. Blocks layer. |
| ASSUMPTION | ASM- | Treated as true but not validated. |
| DEFERRED | DEF- | Consciously postponed with a reason. |
 
**Audience tagging** (required on every DECISION):
 
| Value | When to use |
|-------|-------------|
| `design` | Token naming, layer architecture, Figma structure, visual decisions |
| `dev` | Export scripts, platform implementation, tooling, code pipelines |
| `both` | Governance, process changes, workflow, cross-team agreements |
 
When in doubt → `both`.
 
**Status tagging** (required on every DECISION):
 
| Value | When to use |
|-------|-------------|
| `stable` | Agreed by all relevant parties, not expected to change |
| `draft` | Working direction, may still shift |
| `tbd` | Topic raised, no direction yet |
| `unverified` | Reconstructed, missing sign-off, or inferred |
 
---
 
## Step 3 (--design mode) — Signal Classification
 
Design sessions do not produce ratified DS- decisions.
They produce **proposals** and **explorations** — candidates that may
become DS- decisions once a dev session ratifies them.
 
Every extracted item gets exactly one label:
 
| Label | ID prefix | Meaning |
|-------|-----------|---------|
| PROPOSAL | PRO- | Design direction fully formed. Ready to bring to a dev session. |
| EXPLORATION | EXP- | Still being worked out. Not ready to surface to devs yet. |
| OPEN | OPN- | Discussed but unresolved. Needs more design work or dev input. |
| CONFLICT | CFX- | Contradicts a prior DS- decision. Must be flagged before proceeding. |
| DEFERRED | DEF- | Consciously postponed with a reason. |
 
**Readiness classification (required on every PRO-):**
 
| Value | Meaning |
|-------|---------|
| `ready` | Fully formed. Can be placed on next dev session agenda as-is. |
| `needs-context` | Solid direction but requires background explanation before dev review. |
 
**Audience tagging (required on every PRO-):**
Same values as default mode: `design` / `dev` / `both`.
Note: most PRO- items will be `both` — they are proposals waiting for
dev sign-off. Use `design` only if the proposal is purely structural
(Figma organisation, naming convention internal to design).
 
**No DS- IDs are assigned in design mode.**
PRO- and EXP- items are staging entries. They become DS- only when
ratified in a subsequent cross-functional session.
 
---
 
## Step 4 — Contradiction Detection
 
Before finalising, scan all DECISION / PRO- items against:
- Prior DS- entries in the current conversation
- Any registry data available
 
A contradiction exists when:
- A new decision changes a value, name, or rule from a prior decision
- A new decision removes or replaces a prior one without saying so
- Two decisions in the same session are logically incompatible
 
In **--design mode**, also flag when a PRO- contradicts an existing
DS- decision — this must be surfaced before the proposal reaches devs,
not during the dev session.
 
False positives are cheaper than missed conflicts — when in doubt, flag as CFX-.
 
---
 
## Step 5 — ID Assignment
 
IDs use the session-prefixed convention:
 
```
DS-{N}-{NNN}    e.g. DS-4-001      (default mode only)
PRO-{N}-{NNN}   e.g. PRO-5-001     (design mode only)
EXP-{N}-{NNN}   e.g. EXP-5-001     (design mode only)
OPN-{N}-{NNN}   e.g. OPN-4-001
CFX-{N}-{NNN}   e.g. CFX-4-001
ASM-{N}-{NNN}   e.g. ASM-4-001
DEF-{N}-{NNN}   e.g. DEF-4-001
```
 
`{N}` = session number (integer). `{NNN}` starts at 001 per session.
IDs are permanent. Never reuse or reassign.
 
When a PRO- is ratified in a dev session, it receives a new DS- ID.
The PRO- entry is updated with `"promotedTo": "DS-N-NNN"` and
`"status": "ratified"`. The original PRO- ID is never reused.
 
---
 
## Output A — Human Record (default mode)
 
Produce this formatted block first. This is what the team reads to verify.
 
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SESSION RECORD — CROSS-FUNCTIONAL
[Session N] · [YYYY-MM-DD] · [Attendees]
Topic: [inferred or stated topic]
Milestone context: [Milestone N — current status]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 
## DECISIONS  ✅
[DS-N-NNN] [Layer] [audience: design|dev|both]
→ [One sentence, present tense, active voice]
   Context: [1–2 sentences of why, only if not obvious]
 
## OPEN THREADS  🔵
[OPN-N-NNN]
→ [What was discussed but not resolved]
   Last position: [Where it ended]
   Needs: [What resolves this]
   Priority: [high|med|low per ds-roadmap algorithm]
 
## CONFLICTS DETECTED  🔴
[CFX-N-NNN]
→ Topic: [What this is about]
   Previous [DS-N-NNN · date]: "[Prior stance]"
   Current [Session N · date]: "[New stance]"
   If keep old: [implication]
   If go new: [implication]
   ⚠️ Action required before [layer] work continues.
 
## ASSUMPTIONS  🟡
[ASM-N-NNN]
→ [Statement treated as true but not validated]
   Risk if wrong: [What breaks]
 
## DEFERRED  ⏸️
[DEF-N-NNN]
→ [What was postponed]
   Reason: [Why]
   Revisit: [When / under what condition]
 
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SESSION SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[3–4 sentences. What moved forward, what's stuck,
what's at risk. Written for someone not in the room.]
 
Milestone progress: [Did this session advance Milestone N?
What remains before it closes?]
```
 
---
 
## Output A — Human Record (--design mode)
 
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SESSION RECORD — DESIGN
[Session N] · [YYYY-MM-DD] · [Attendees]
Topic: [inferred or stated topic]
Milestone context: [Milestone N — current status]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 
## PROPOSALS — READY FOR DEV REVIEW  🟢
[PRO-N-NNN] [Layer] [audience: design|dev|both] [readiness: ready|needs-context]
→ [One sentence, present tense, active voice]
   Rationale: [Why this direction. 1–2 sentences max.]
   Needs from devs: [What the team needs to confirm or implement]
 
## PROPOSALS — NEEDS MORE DESIGN WORK  🟡
[PRO-N-NNN] [Layer] [readiness: needs-context]
→ [What the proposal is]
   Blocker: [What's still unresolved on the design side]
 
## EXPLORATIONS  🔵
[EXP-N-NNN] [Layer]
→ [What's being explored]
   Current thinking: [Where the conversation landed]
   Not ready because: [What's missing before this can become a proposal]
 
## CONFLICTS WITH EXISTING DECISIONS  🔴
[CFX-N-NNN]
→ Topic: [What this is about]
   Existing [DS-N-NNN · date]: "[Ratified stance]"
   Proposed direction: "[What the design session is considering]"
   ⚠️ Resolve before bringing to dev session.
 
## DEFERRED  ⏸️
[DEF-N-NNN]
→ [What was postponed]
   Reason: [Why]
   Revisit: [When / under what condition]
 
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ELEVATION LIST — Next dev session candidates
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Ordered by priority. Only PRO- items with readiness: ready qualify.
PRO- items with an active CFX- appear here but are marked as blocked.
 
1. [PRO-N-NNN] — [Short label]
   Why now: [Milestone dependency or blocking reason]
 
2. [PRO-N-NNN] — [Short label] ⚠️ BLOCKED
   Blocked by: [CFX-N-NNN — one sentence on what must be resolved first]
 
[If no items qualify: "No proposals are ready for dev review yet.
Address explorations before the next session."]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 
SESSION SUMMARY
[3–4 sentences. What design directions were formed, what needs
more work, what conflicts exist that must be resolved before devs
can review. Written for a developer who will read this cold.]
```
 
---
 
## Output B — JSON Patch (default mode)
 
Immediately after the human record, output the JSON patch.
Label it clearly so the committer knows what to do.
 
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
JSON PATCH — commit to registry/panda-registry.json
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
 
```json
{
  "_meta": {
    "session": "SN",
    "date": "YYYY-MM-DD",
    "patchVersion": "2.0.0",
    "instructions": "Append each array to the matching key in panda-registry.json. Update meta.lastSession, meta.lastUpdated, meta.totalSessions."
  },
  "sessions": [
    {
      "id": "SN",
      "date": "YYYY-MM-DD",
      "type": "cross-functional",
      "topic": "Short topic label",
      "attendees": ["Name1", "Name2"],
      "decisions": 0,
      "openThreads": 0,
      "conflicts": 0,
      "assumptions": 0,
      "deferred": 0
    }
  ],
  "decisions": [],
  "openThreads": [],
  "conflicts": [],
  "assumptions": [],
  "deferred": []
}
```
 
---
 
## Output B — JSON Patch (--design mode)
 
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
JSON PATCH — commit to registry/panda-registry.json
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
 
```json
{
  "_meta": {
    "session": "SN",
    "date": "YYYY-MM-DD",
    "patchVersion": "2.0.0",
    "instructions": "Append sessions array. Append proposals array to panda-registry.json proposals key (create if absent). Update meta.lastSession, meta.lastUpdated, meta.totalSessions."
  },
  "sessions": [
    {
      "id": "SN",
      "date": "YYYY-MM-DD",
      "type": "design",
      "topic": "Short topic label",
      "attendees": ["Name1", "Name2"],
      "proposals": 0,
      "explorations": 0,
      "conflicts": 0,
      "deferred": 0
    }
  ],
  "proposals": [],
  "conflicts": [],
  "deferred": []
}
```
 
### JSON field schemas (default mode)
 
**Decision:**
```json
{
  "id": "DS-N-NNN",
  "session": "SN",
  "date": "YYYY-MM-DD",
  "layer": "Tokens — Spacing | Tokens — Naming | Tokens — Architecture | Tokens — Process | Components | Patterns | Templates | Process / Governance | Process / Tooling | Process / Adoption",
  "audience": "design | dev | both",
  "statement": "One sentence present tense no trailing period",
  "context": "Why this was decided or empty string",
  "status": "stable | draft | tbd | unverified"
}
```
 
**Open thread:**
```json
{
  "id": "OPN-N-NNN",
  "session": "SN",
  "topic": "Short label",
  "lastPosition": "Where the conversation ended",
  "needs": "What would resolve this",
  "layer": "Layer or process area",
  "priority": "high | med | low",
  "blocks": "DS-N-NNN or description or —",
  "status": "open"
}
```
 
**Conflict:**
```json
{
  "id": "CFX-N-NNN",
  "session": "SN",
  "topic": "Short label",
  "previousDecision": "Prior stance",
  "currentDirection": "New stance",
  "implKeepOld": "Consequence of keeping old",
  "implGoNew": "Consequence of adopting new",
  "resolutionStatus": "unresolved",
  "notes": "Context needed to resolve",
  "sessionResolved": null
}
```
 
**Assumption:**
```json
{
  "id": "ASM-N-NNN",
  "session": "SN",
  "statement": "What was treated as true",
  "risk": "What breaks if wrong",
  "status": "open"
}
```
 
**Deferred:**
```json
{
  "id": "DEF-N-NNN",
  "session": "SN",
  "topic": "Short label",
  "reason": "Why deferred",
  "revisit": "When or under what condition"
}
```
 
### JSON field schemas (--design mode)
 
**Proposal:**
```json
{
  "id": "PRO-N-NNN",
  "session": "SN",
  "date": "YYYY-MM-DD",
  "layer": "Tokens — Spacing | Tokens — Naming | Tokens — Architecture | Components | Patterns | Templates | Process / Governance",
  "audience": "design | dev | both",
  "statement": "One sentence present tense no trailing period",
  "rationale": "Why this direction",
  "needsFromDevs": "What the team needs to confirm or implement",
  "readiness": "ready | needs-context",
  "status": "proposed",
  "promotedTo": null
}
```
 
*When a PRO- is ratified in a dev session, update:*
```json
{
  "status": "ratified",
  "promotedTo": "DS-N-NNN"
}
```
 
**Exploration:**
```json
{
  "id": "EXP-N-NNN",
  "session": "SN",
  "layer": "Layer or process area",
  "description": "What is being explored",
  "currentThinking": "Where the conversation landed",
  "blockedBy": "What is missing before this can become a PRO-",
  "status": "open"
}
```
 
---
 
## Validation Checklist
 
**Default mode:**
- [ ] Every decision has `audience` and `status`
- [ ] All IDs follow `TYPE-N-NNN` format
- [ ] No `statement` ends with a period
- [ ] Session counts match actual array lengths
- [ ] `blocks` references are real IDs or `—`
- [ ] `_meta` block present with merge instructions
- [ ] Session `type` is `"cross-functional"`
 
**--design mode:**
- [ ] No DS- IDs assigned
- [ ] Every PRO- has `readiness` and `audience`
- [ ] No `statement` ends with a period
- [ ] Elevation list contains only `readiness: ready` items
- [ ] CFX- items flagged for any PRO- conflicting with existing DS-
- [ ] Session `type` is `"design"`
- [ ] `_meta` instructions reference `proposals` key
 
---
 
## Closing Block
 
**Default mode:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SESSION COMPLETE
SN · [X] decisions · [Y] open · [Z] conflicts
Human record ↑ — share with team for verification
JSON patch ↑ — commit to registry/panda-registry.json
Commit message: feat(registry): add SN — [topic]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
 
**--design mode:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DESIGN SESSION COMPLETE
SN · [X] proposals ([Y] ready) · [Z] explorations · [W] conflicts
Elevation list ↑ — [Y] items ready for next dev session agenda
JSON patch ↑ — commit to registry/panda-registry.json
Commit message: feat(registry): add SN — design — [topic]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---
name: ds-roadmap
description: >
  The north star skill for Panda DS. Holds Boozt's product vision for the
  design system — what the company actually needs to ship, what coverage gaps
  exist against real product screens, and what the milestone sequence is.
  Load this skill whenever any other Panda skill needs strategic direction:
  when planning a session agenda, when deciding what to spec next, when
  evaluating whether an open thread is high or low priority, or when the
  user asks "what should we work on next", "are we building the right things",
  "what's missing from the DS", or "what's the priority". This skill is the
  answer to "why are we doing this session?" — always load it before
  pre-session-planner, registry-analyst, or spec-writer runs.
---
 
# ds-roadmap — Panda DS North Star
 
This skill answers the question every other skill assumes someone has already
answered: **what does Boozt actually need from this design system, and are we
building toward it?**
 
Without this, sessions are driven by backlog age. With it, every decision,
every open thread resolution, every spec gets evaluated against whether it
moves Panda closer to covering Boozt's real product surface.
 
---
 
## Boozt's Product Surface
 
Panda serves three platforms across two brands:
 
| Brand | Platforms | Primary surfaces |
|-------|-----------|-----------------|
| Boozt | Web (desktop + mobile), iOS, Android | Product listing, product detail, checkout, account, search, editorial |
| Boozt Outlet | Web (desktop + mobile), iOS, Android | Same surface, different visual identity |
 
**Key constraint:** Outlet Nation requires brand-level token differentiation
from Boozt — this is the primary driver for the semantic layer = brands model
(DS-2-002). Every token decision must be evaluated against whether it supports
this split cleanly.
 
---
 
## DS Coverage Model
 
Panda is complete when it covers all four layers for all surfaces on all
platforms. Coverage is tracked in `panda-registry.json` under `designSystem`.
 
### Coverage tiers
 
| Tier | Meaning |
|------|---------|
| **Foundation** | Tokens stable, primitive layer locked, semantic layer agreed |
| **Core components** | Button, Input, Card, Navigation, Toast, Modal, Badge |
| **Commerce patterns** | ProductCard, PriceBlock, AddToCart, FilterBar, ImageGallery |
| **Page templates** | PLP, PDP, Cart, Checkout, Account, Search results |
 
No component work begins until Foundation is complete.
No pattern work begins until Core components are stable.
No template work begins until Commerce patterns are stable.
 
---
 
## Current Milestone
 
**Milestone 1 — Foundation** (in progress)
 
Target: All spacing, sizing, color, typography, radius, and motion primitive
tokens defined and reviewed. Semantic layer convention agreed across platforms.
Export pipeline established with a named owner.
 
Status as of S3: Spacing primitives drafted (Web only). Semantic layer model
proposed but pending ratification (OPN-2-002). Export script has no owner
(OPN-2-004). Minimum sizing undefined (OPN-3-001). Icons deferred (DEF-1-001).
 
**Foundation is not complete until:**
- [ ] All primitive token categories have at least `draft` status on Web
- [ ] Semantic layer convention ratified by all platform reps
- [ ] Export script has an owner and a defined pipeline
- [ ] Minimum sizing values agreed (OPN-3-001 resolved)
- [ ] Icon strategy decided (DEF-1-001 resolved)
 
**Milestone 2 — Core components** (not started)
Depends entirely on Milestone 1 being complete.
 
---
 
## Priority Algorithm
 
When evaluating what to work on next, apply this order:
 
1. **Unresolved CFX-** — conflicts block everything downstream. Always first.
2. **Foundation blockers** — any OPN- or DEF- that prevents Milestone 1 from
   closing. Treated as HIGH regardless of age.
3. **OPN- older than 2 sessions** — age signals a stuck decision. Needs forcing.
4. **OPN- from current session** — fresh but unresolved.
5. **At-risk ASM-** — assumptions that could invalidate completed work if wrong.
6. **Everything else** — DEF-, low-priority OPN-, nice-to-haves.
 
**The milestone gate is hard.** Even if a component idea is exciting,
no component work enters the agenda until Foundation is complete. The
`pre-session-planner` must enforce this — if Milestone 1 is open, the agenda
contains only Foundation items.
 
---
 
## Brand Token Strategy
 
Semantic tokens are the brand differentiation layer. The model from DS-2-002:
 
```
primitive (reference)  →  raw values, never changes
semantic (system)      →  brand-level aliases, one set per brand
component (exception)  →  platform-level exceptions only
```
 
This means:
- `color.action.primary` resolves differently for Boozt vs Outlet Nation
- The same component library works for both brands
- Platform teams consume semantic tokens, never primitives directly
 
**Any decision that touches the semantic layer must be evaluated against
whether it supports clean brand differentiation.** If it doesn't, it belongs
in the primitive layer instead.
 
---
 
## What Other Skills Should Consult Here
 
| Skill | What to ask ds-roadmap |
|-------|----------------------|
| `pre-session-planner` | What milestone are we on? What are the Foundation blockers? |
| `registry-analyst` | Are the open items actually blocking the current milestone? |
| `spec-writer` | Does this spec belong to the current milestone? What platform coverage is required? |
| `session-engine` | When classifying audience and priority, does this decision unblock the milestone? |
 
---
 
## Updating This Skill
 
When a milestone is completed, update:
1. The "Current Milestone" section — mark complete, activate next milestone
2. The coverage tiers — mark Foundation → Core → Patterns → Templates as each closes
3. The priority algorithm — once Foundation is done, Foundation blockers drop out
 
This skill is the only place milestone state lives. Claude Code commits the
update to `registry/panda-roadmap.md` at the same time as the session patch.

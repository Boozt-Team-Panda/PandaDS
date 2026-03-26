# PANDA_DS_PROJECT.md
# Panda Design System — Project Overview

---

## HOW TO USE THIS FILE

This is a stable reference document. It describes what Panda is, how it is
structured, who is involved, and how the team works. It does not track
current state, decisions, or open threads — those live in panda-registry.json.

Read this file to stay oriented to the plan. Use the registry for current state.

---

## 1. PROJECT IDENTITY

**Name:** Panda
**Owner:** Boozt Fashion AB
**Leads:** Facundo Mau + Justin Daneman (design)
**Purpose:** Single source of truth for design decisions across Web, iOS,
and Android. Serves two brands: Boozt and Boozt Outlet (Outlet Nation).

**North star goal:** Team transparency and synchronization across design
and development. If a decision is undocumented, it doesn't exist.

**Repository:** github.com/Boozt-Team-Panda/PandaDS
**Registry file:** registry/panda-registry.json
**Registry viewer:** GitHub Pages (hosted from repo)

**Team:**
- Design (PD): Facundo Mau, Justin Daneman
- Web: Danas Paulikas
- iOS: Patrick Ahrentløv (senior / team lead — pulled out), Daniel Wennberg (active)
- Android: Lasse Dencker Sørensen (senior / team lead — pulled out), Rasmus Sander Larsen (active)

---

## 2. SYSTEM ARCHITECTURE

### Four layers (bottom to top)

```
Tokens → Components → Patterns → Templates
```

Every decision traces to a token. Never skip layers.
When a decision belongs between two layers, assign it to the lower one.

### Token architecture (three tiers)

```
Primitive  →  raw values, never aliased directly by consumers
Semantic   →  brand-level aliases (one set per brand — drives Boozt vs Outlet split)
Component  →  platform-level exceptions only
```

The semantic layer is the brand differentiation layer. Outlet Nation requires
a distinct set of semantic aliases from Boozt. Every token decision must
support this split cleanly. Platform teams consume semantic tokens — never
primitives directly.

### Platform targets

| Platform | Paradigm | Key constraint |
|----------|----------|----------------|
| Web desktop | CSS Grid / Flexbox | Fluid + breakpoints |
| Web mobile | CSS Flexbox | Touch targets, thumb zones |
| iOS | UIKit / SwiftUI | Safe areas, native nav, HIG compliance |
| Android | Jetpack Compose | Material You baseline, adaptive layout |

Panda is Figma-native and platform-agnostic. The source of truth is always
a Figma file. Token names and component semantics are designed to map
cleanly to any platform. Output must never default to web-only.

---

## 3. MILESTONE PLAN

Four milestones, each gated on the previous. No milestone work begins
until the one before it is complete — this is a hard rule.

### M1 — Foundation
Tokens stable. Primitive layer locked. Semantic layer convention agreed
across platforms. Export pipeline established with a named owner.

Token categories in scope: Spacing, Sizing, Color, Typography, Radius,
Shadow, Motion, Breakpoints.

### M2 — Core components
Discrete UI elements built entirely from Foundation tokens.
Scope: Button, Input, Card, Navigation, Toast, Modal, Badge.

### M3 — Commerce patterns
Recurring compositions of Core components solving specific UX problems.
Scope: ProductCard, PriceBlock, AddToCart, FilterBar, ImageGallery.

### M4 — Page templates
Full-page compositions built from Commerce patterns.
Scope: PLP, PDP, Cart, Checkout, Account, Search results.

---

## 4. SESSION WORKFLOW

Two session types exist. Both feed the same registry.

### Design sessions
**Who:** Facu + Justin only
**Purpose:** Proposal factory — forming directions before they reach devs
**Output:** PRO- (proposals) and EXP- (explorations)
**Rule:** No DS- decisions are made in design sessions

### Cross-functional sessions
**Who:** Facu + Justin + platform reps (Web, iOS, Android)
**Purpose:** Ratification, conflict resolution, platform decisions
**Output:** DS- decisions, OPN- threads, CFX- conflicts

### Proposal pipeline

```
Design session
    ↓  session-engine --design
PRO- / EXP- items → proposals staging (status: proposed)
    ↓  pre-session-planner pulls readiness: ready items
Dev session agenda
    ↓  session-engine default
DS- decision created → PRO- promoted (status: ratified)
```

**A PRO- reaches the dev agenda only when:**
1. Readiness is `ready` (not `needs-context`)
2. No active CFX- exists against it
3. Its layer is within the current milestone scope

---

## 5. ID CONVENTIONS

```
DS-{N}-{NNN}    Ratified decision          e.g. DS-4-001
PRO-{N}-{NNN}   Design proposal (staging)  e.g. PRO-5-001
EXP-{N}-{NNN}   Design exploration         e.g. EXP-5-001
OPN-{N}-{NNN}   Open thread                e.g. OPN-2-002
CFX-{N}-{NNN}   Conflict                   e.g. CFX-3-001
ASM-{N}-{NNN}   Assumption                 e.g. ASM-2-001
DEF-{N}-{NNN}   Deferred item              e.g. DEF-1-001
TOK-{NNN}       Token spec                 e.g. TOK-001
CMP-{NNN}       Component spec             e.g. CMP-001
PAT-{NNN}       Pattern spec               e.g. PAT-001
TPL-{NNN}       Template spec              e.g. TPL-001
```

{N} = session number. {NNN} starts at 001 per session.
IDs are permanent. Never reuse or reassign.
Design and cross-functional sessions share the same counter.

---

## 6. PRIORITY ALGORITHM

When evaluating what to work on, apply in this order:

1. Unresolved CFX- — conflicts block everything downstream
2. Foundation blockers — OPN- or DEF- preventing M1 from closing
3. PRO- items ready for ratification — cleared design proposals
4. OPN- older than 2 sessions — age signals a stuck decision
5. OPN- from most recent session — fresh but unresolved
6. At-risk ASM- — could invalidate completed work if wrong
7. DEF- and low-priority OPN- — if time permits

The milestone gate is hard. No component work enters the agenda until
Foundation is complete, regardless of how ready a proposal looks.

---

## 7. SKILL INVENTORY

All skills are packaged as .skill files and installed in the Claude project.
All skill modifications must go through skill-creator (draft → test → package).

| Skill | Purpose |
|-------|---------|
| `ds-roadmap` | North star — milestones, priority algorithm, brand token strategy |
| `session-engine` | Processes transcripts — default (cross-functional) and --design modes |
| `pre-session-planner` | Builds dev session agendas from registry backlog + PRO- staging |
| `registry-analyst` | Reasons over registry — health check, critical path, delta, readiness |
| `spec-writer` | Converts DS- decisions into human + JSON specs per layer |
| `panda-ds` | System vocabulary, layer definitions, platform philosophy |

**Load order rule:** Always load `ds-roadmap` before running
`pre-session-planner`, `registry-analyst`, `spec-writer`, or `session-engine`.

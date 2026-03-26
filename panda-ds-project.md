# Panda Design System — Project Overview

This is the stable human reference for Panda. It describes what the system is,
who is involved, and how the team works.

It does not track current state — that lives in `registry/panda-registry.json`.
It does not describe tooling or Claude workflows — that lives in `CLAUDE.md`.

---

## Project identity

**Name:** Panda
**Owner:** Boozt Fashion AB
**Leads:** Facundo Mau + Justin Daneman
**Purpose:** Single source of truth for design decisions across Web, iOS, and Android.
Serves two brands: Boozt and Boozt Outlet (Outlet Nation).

**North star:** Team transparency and synchronization. If a decision is undocumented, it doesn't exist.

**Registry viewer:** https://boozt-team-panda.github.io/PandaDS

---

## Team

| Role | Person | Status |
|------|--------|--------|
| Design lead | Facundo Mau | Active |
| Design lead | Justin Daneman | Active |
| Web | Danas Paulikas | Active |
| iOS | Daniel Wennberg | Active |
| iOS | Patrick Ahrentløv | Stepped back |
| Android | Rasmus Sander Larsen | Active |
| Android | Lasse Dencker Sørensen | Stepped back |

Design holds veto on design system decisions. Sync meetings are bi-weekly, owned by Facundo.

---

## System architecture

Four layers, bottom to top. Never skip layers. When a decision belongs between two layers, assign it to the lower one.

```
Tokens → Components → Patterns → Templates
```

### Token tiers

```
Primitive   raw values. Never consumed directly by platform teams.
Semantic    brand-level aliases. One set per brand. This is where Boozt and Outlet Nation diverge.
Component   platform-level exceptions only. Not a general override mechanism.
```

Platform teams consume **semantic tokens**, never primitives directly.

### Platform targets

| Platform | Paradigm | Key constraint |
|----------|----------|----------------|
| Web desktop | CSS Grid / Flexbox | Fluid + breakpoints |
| Web mobile | CSS Flexbox | Touch targets, thumb zones |
| iOS | UIKit / SwiftUI | Safe areas, native nav, HIG compliance |
| Android | Jetpack Compose | Material You baseline, adaptive layout |

Panda is Figma-native and platform-agnostic. Source of truth is always a Figma file. Output must never default to web-only.

---

## Milestone plan

Each milestone is hard-gated on the previous. No work begins on a milestone until the prior one is complete.

| Milestone | Scope |
|-----------|-------|
| **M1 — Foundation** | All token categories defined. Primitive layer locked. Semantic convention ratified. Export pipeline with named owner. |
| **M2 — Core components** | Button, Input, Card, Navigation, Toast, Modal, Badge. |
| **M3 — Commerce patterns** | ProductCard, PriceBlock, AddToCart, FilterBar, ImageGallery. |
| **M4 — Page templates** | PLP, PDP, Cart, Checkout, Account, Search results. |

---

## Session types

### Design sessions
**Who:** Facu + Justin only
**Produces:** PRO- proposals and EXP- explorations — no DS- decisions
**Purpose:** Form directions before they reach developers

### Cross-functional sessions
**Who:** Facu + Justin + platform reps (Web, iOS, Android)
**Produces:** DS- decisions, OPN- threads, CFX- conflicts
**Purpose:** Ratification, conflict resolution, platform sign-off

### Proposal pipeline

```
Design session
    ↓  PRO- and EXP- items enter proposals staging
    ↓  pre-session-planner picks up readiness: ready items
Cross-functional session agenda
    ↓  team ratifies
DS- decision created · PRO- promoted to ratified
```

A PRO- reaches the dev agenda only when:
1. `readiness: "ready"` — not `needs-context`
2. No active CFX- against it
3. Its layer is within the current milestone scope

---

## ID conventions

IDs are permanent. Never reuse or reassign. Design and cross-functional sessions share the same counter.

```
DS-{N}-{NNN}    Ratified decision          e.g. DS-4-001
PRO-{N}-{NNN}   Design proposal            e.g. PRO-5-001
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

---

## Priority algorithm

When deciding what to work on, apply in this order:

1. Unresolved CFX- — conflicts block everything downstream
2. Foundation blockers — OPN- or DEF- preventing M1 from closing
3. PRO- items ready for ratification
4. OPN- older than 2 sessions
5. OPN- from the most recent session
6. At-risk ASM- items
7. DEF- and low-priority OPN- if time permits

# CLAUDE.md — Panda Design System
# Entry point for Claude Code. Read this before doing anything in this repo.

---

## What this repo is

This is the source of truth for **Panda**, Boozt's internal design system.
It stores every decision, open thread, conflict, assumption, and token spec
produced in team sessions. It is read by designers, developers, iOS, Android,
and web — all of them, without needing to ask anyone for the current state.

**If a decision is undocumented here, it doesn't exist.**

**Repository:** https://github.com/Boozt-Team-Panda/PandaDS
**Registry viewer:** GitHub Pages (auto-served from `/registry/index.html`)
**Registry file:** `registry/panda-registry.json`

---

## Team

| Role | Person |
|------|--------|
| Design leads (PD) | Facundo Mau + Justin Daneman |
| Web | Danas Paulikas |
| iOS | Daniel Wennberg (active) · Patrick Ahrentløv (stepped back) |
| Android | Rasmus Sander Larsen (active) · Lasse Dencker Sørensen (stepped back) |

Design holds veto on design system decisions (DS-3-002).
Sync meetings are bi-weekly, owned by Facundo (DS-3-009).

---

## Repo structure

```
PandaDS/
├── CLAUDE.md                        ← you are here
├── PANDA_DS_PROJECT.md              ← stable project reference
├── registry/
│   ├── panda-registry.json          ← the brain. all decisions live here.
│   └── index.html                   ← GitHub Pages viewer
└── skills/
    ├── ds-roadmap.md                ← north star: milestones + priority algorithm
    ├── panda-ds.md                  ← system vocabulary + layer definitions
    ├── session-engine.md            ← processes transcripts into decisions
    ├── pre-session-planner.md       ← builds session agendas from the backlog
    ├── registry-analyst.md          ← reasons over the registry (health, critical path)
    └── spec-writer.md               ← converts decisions into token/component specs
```

---

## The system architecture

Four layers, bottom to top. Never skip layers. When in doubt, go lower.

```
Tokens → Components → Patterns → Templates
```

### Token tiers

```
Primitive   →  raw values. Never consumed directly by platform teams.
Semantic    →  brand aliases. One set per brand (Boozt vs Outlet Nation).
                This is the brand differentiation layer.
Component   →  platform-level exceptions only. Not a general override.
```

Platform teams consume **semantic tokens**, never primitives.
The semantic layer is what makes Boozt and Outlet Nation visually different
while sharing the same component library.

### Platform targets

| Platform | Paradigm | Key constraint |
|----------|----------|----------------|
| Web desktop | CSS Grid / Flexbox | Fluid + breakpoints |
| Web mobile | CSS Flexbox | Touch targets, thumb zones |
| iOS | UIKit / SwiftUI | Safe areas, native nav, HIG compliance |
| Android | Jetpack Compose | Material You baseline, adaptive layout |

Panda is Figma-native and platform-agnostic. Source of truth is always Figma.
Never default to web-only output.

---

## Milestone plan

Milestones are hard-gated. No milestone begins until the prior one is complete.

| Milestone | Scope | Status |
|-----------|-------|--------|
| **M1 — Foundation** | All token categories defined. Primitive locked. Semantic ratified. Export pipeline with named owner. | 🟡 In progress |
| **M2 — Core components** | Button, Input, Card, Navigation, Toast, Modal, Badge | 🔒 Blocked on M1 |
| **M3 — Commerce patterns** | ProductCard, PriceBlock, AddToCart, FilterBar, ImageGallery | 🔒 Blocked on M2 |
| **M4 — Page templates** | PLP, PDP, Cart, Checkout, Account, Search results | 🔒 Blocked on M3 |

**Foundation is not complete until:**
- All primitive token categories have at least `draft` status on Web
- Semantic layer convention ratified by all platform reps (OPN-2-002)
- Export script has a named owner (OPN-2-004)
- Minimum sizing values agreed (OPN-3-001)
- Icon strategy decided (DEF-1-001)

---

## Registry structure

`registry/panda-registry.json` is the single source of truth.
Schema version: `2.0.0`

Top-level keys:

| Key | What it holds |
|-----|---------------|
| `meta` | Schema version, last session, last updated |
| `sessions` | One entry per session (cross-functional and design) |
| `decisions` | All DS- ratified decisions |
| `openThreads` | All OPN- items, open and resolved |
| `conflicts` | All CFX- items |
| `assumptions` | All ASM- items |
| `deferred` | All DEF- items |
| `designSystem` | Token specs (TOK-), component specs (CMP-), patterns (PAT-), templates (TPL-) |

Proposals (PRO-) and explorations (EXP-) from design sessions live under a
`proposals` key (added once the first design session is processed).

---

## ID conventions

IDs are **permanent**. Never reuse or reassign.

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

`{N}` = session number. `{NNN}` starts at 001 per session.
Design and cross-functional sessions share the same counter.

---

## Session types

### Cross-functional sessions
**Who:** Facu + Justin + platform reps (Web, iOS, Android)
**Produces:** DS- decisions · OPN- threads · CFX- conflicts
**Skill:** `session-engine` (default mode)

### Design sessions
**Who:** Facu + Justin only
**Produces:** PRO- proposals · EXP- explorations (no DS- decisions)
**Skill:** `session-engine --design`

### Proposal pipeline

```
Design session → PRO- (status: proposed)
    ↓  pre-session-planner checks readiness: ready
Dev session agenda
    ↓  session-engine ratifies
DS- decision created · PRO- status → ratified · PRO-.promotedTo = DS-N-NNN
```

A PRO- reaches the dev agenda only when:
1. `readiness: "ready"` (not `needs-context`)
2. No active CFX- exists against it
3. Its layer is within the current milestone scope

---

## Priority algorithm

Apply in this order when deciding what to work on:

1. Unresolved CFX- — block everything downstream
2. Foundation blockers — OPN- or DEF- preventing M1 from closing
3. PRO- items ready for ratification
4. OPN- older than 2 sessions
5. OPN- from most recent session
6. At-risk ASM-
7. DEF- and low-priority OPN-

---

## Skills

Skills are the operating instructions for each workflow. They live in `skills/`.
**Always load `ds-roadmap` before running any other skill.**

| Skill file | When to use |
|-----------|-------------|
| `skills/ds-roadmap.md` | North star — milestones, priority, brand token strategy. Load first. |
| `skills/panda-ds.md` | System vocabulary, layer definitions, platform philosophy |
| `skills/session-engine.md` | Processing a meeting transcript → decisions + JSON patch |
| `skills/pre-session-planner.md` | Building a session agenda from the registry backlog |
| `skills/registry-analyst.md` | Health check, critical path, delta between sessions, readiness |
| `skills/spec-writer.md` | Converting a DS- decision into a token/component spec |

---

## Commit conventions

```
feat(registry): add S4 — spacing semantic layer ratification
feat(registry): add S5 — design — color exploration
fix(registry): correct DS-3-006 status to stable
feat(specs): add TOK-015 spacing.semantic.inset
feat(viewer): update index.html navigation
```

---

## How to update the registry

After processing a session:

1. Claude produces a JSON patch (Output B from `session-engine`)
2. Merge the patch arrays into `registry/panda-registry.json`
3. Update `meta.lastSession`, `meta.lastUpdated`, `meta.totalSessions`
4. Commit with the convention above
5. Push — GitHub Pages auto-updates the viewer

Never edit `panda-registry.json` manually except to apply a Claude-produced patch.

---

## Rules that never change

- Never invent a token name. Use `[token.name.TBD]` and flag it.
- Never skip layers. A component is not a pattern.
- Never mark a decision `stable` if it hasn't been agreed by all relevant parties.
- Never resolve a CFX- unilaterally. Surface it, let the team decide.
- If it's undocumented, it doesn't exist.

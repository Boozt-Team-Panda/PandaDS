# Panda DS

Boozt's internal design system — Figma-native, platform-agnostic output for web, iOS, and Android.

| | |
|--|--|
| **Figma** | [Panda DS by Claude](https://www.figma.com/design/fwG5dH7sWi3IAsMXZP1pCV/Panda-DS-by-Claude) |
| **Docs** | [GitHub Pages](https://boozt-team-panda.github.io/PandaDS) |
| **Maintained by** | Platform Team — Facu + Justin (design), devs (sync) |

---

## What this repo is

This is the source of truth for all Panda DS decisions, documentation, and tooling.

It does **not** contain production code. It contains:
- Every design system decision ever made, in order, with rationale
- The session history that produced those decisions
- Auto-generated documentation synced from Figma
- The Claude skills that maintain everything

---

## How it works

### 1. Sessions produce decisions

Design sessions between Facu and Justin, and sync sessions with developers, are transcribed and processed by `panda-session`. Every decision gets a permanent ID (DS-001, DS-002…) and a status:

- `draft` — came from a design session, can be acted on, may be refined
- `stable` — ratified in a sync session with devs present
- `superseded` — replaced by a newer decision (never deleted, history preserved)

### 2. Decisions live in `registry/`

`registry/decisions.json` is the ledger. Every decision is appended. Nothing is ever deleted.

### 3. Docs are generated from decisions + Figma

`panda-spec` reads a decision, reads the corresponding Figma node, writes the spec, and updates `docs/[layer].md`. GitHub Pages publishes the result automatically on every push to `main`.

### 4. Figma and GitHub stay in sync

Token variable values are the source of truth in Figma. Decisions and rationale are the source of truth in GitHub. `panda-spec` checks for drift between the two and flags it before writing.

---

## Repository structure

```
PandaDS/
│
├── registry/
│   ├── decisions.json   ← DS-001 → DS-NNN, permanent ledger
│   ├── sessions.json    ← Session log
│   ├── state.json       ← Current DS snapshot
│   └── changelog.md     ← Human-readable history
│
├── docs/                ← GitHub Pages source
│   ├── _config.yml
│   ├── index.md
│   ├── tokens.md
│   ├── components.md
│   ├── patterns.md
│   ├── templates.md
│   └── conventions.md
│
├── skills/
│   ├── panda-session.md ← Processes transcripts, pushes registry
│   ├── panda-spec.md    ← Specs decisions, syncs Figma + docs
│   └── figma-rules.md   ← Generated once via create_design_system_rules
│
├── .github/
│   └── workflows/
│       ├── publish-docs.yml     ← Auto-publishes Pages on push
│       └── validate-registry.yml ← Validates JSON on PRs
│
├── panda-core.md        ← Shared DS context (loaded by both skills)
└── README.md
```

---

## For designers (Facu, Justin)

- All design decisions are processed through Claude using `panda-session`
- You review the session record before anything is pushed
- Say "approved" → Claude pushes to the registry
- Figma variables are the source of truth for token values — keep them clean
- Run `panda-spec` to sync a decision into the docs and write back to Figma

## For developers

- Read `docs/` or the GitHub Pages site for specs
- Every decision has a permanent DS ID you can reference in PRs
- `registry/changelog.md` gives you the human history of what changed and why
- JSON schema for decisions is in `registry/decisions.json` — predictable, no surprises

## For Claude Code

Claude Code can push to this repo when given explicit approval. The push mechanism:
1. `panda-session` produces a JSON patch after processing a transcript
2. User says "approved"
3. Claude Code appends to `registry/decisions.json` and `registry/sessions.json`,
   prepends to `registry/changelog.md`, and commits with the standard message format:
   `feat(registry): session-[N] — [topic]`

Claude Code must be authenticated with push access to `main` or a deploy key.

---

## Branch rules

- `main` is the single branch — no feature branches for registry changes
- Registry pushes go directly to `main` (they are append-only, not reversible anyway)
- Docs changes that need review should go via PR — the validate-registry Action will run
- Force-push to `main` is disabled

---

## Decision ID sequence

IDs are global and sequential: DS-001, DS-002, DS-003…

They are assigned by `panda-session` at push time. Never manually assign or reorder IDs.
If a decision is wrong, it gets `superseded` by a new one — not edited or deleted.

# CLAUDE.md — Instructions for Claude Code

Read `PANDA_DS_PROJECT.md` first. It has the full project context: architecture,
team, milestones, session types, ID conventions, and priority algorithm.
This file only covers what Claude needs to operate in this repo.

---

## Repo map

```
CLAUDE.md                        ← you are here
PANDA_DS_PROJECT.md              ← project reference (read first)
registry/
  panda-registry.json            ← the source of truth. all decisions live here.
  index.html                     ← GitHub Pages viewer
skills/
  ds-roadmap.md                  ← load before any other skill
  session-engine.md              ← process transcripts
  pre-session-planner.md         ← build session agendas
  registry-analyst.md            ← reason over the registry
  spec-writer.md                 ← write token and component specs
  panda-ds.md                    ← system vocabulary and layer definitions
```

---

## Which skill to load when

Always load `skills/ds-roadmap.md` first. Then:

| Task | Skill |
|------|-------|
| Processing a meeting transcript | `session-engine.md` |
| Processing a design-only session (Facu + Justin) | `session-engine.md` with `--design` flag |
| Building the agenda for the next session | `pre-session-planner.md` |
| Health check, critical path, what's blocking us | `registry-analyst.md` |
| Writing a token or component spec from a DS- decision | `spec-writer.md` |
| Any question about layers, tokens, or platform rules | `panda-ds.md` |

---

## How to update the registry

1. Process the transcript → `session-engine` produces a JSON patch
2. Merge the patch arrays into `registry/panda-registry.json`
3. Update `meta.lastSession`, `meta.lastUpdated`, `meta.totalSessions`
4. Commit and push — GitHub Pages updates automatically

Never edit `panda-registry.json` manually except to apply a Claude-produced patch.

---

## Commit conventions

```
feat(registry): add S4 — spacing semantic layer ratification
feat(registry): add S5 — design — color token exploration
fix(registry): correct DS-3-006 status to stable
feat(specs): add TOK-015 spacing.semantic.inset
feat(viewer): update index.html navigation
```

---

## Hard rules

- Never invent a token name. Use `[token.name.TBD]` and flag it.
- Never skip layers. A component is not a pattern.
- Never mark a decision `stable` unless all relevant parties have agreed.
- Never resolve a CFX- unilaterally. Surface it, let the team decide.
- If it's undocumented, it doesn't exist.

---
title: Panda DS
nav_order: 1
---

# Panda DS

Boozt's internal design system — Figma-native, platform-agnostic.

| | |
|--|--|
| **Figma** | [Panda DS by Claude](https://www.figma.com/design/fwG5dH7sWi3IAsMXZP1pCV/Panda-DS-by-Claude) |
| **GitHub** | [Boozt-Team-Panda/PandaDS](https://github.com/Boozt-Team-Panda/PandaDS) |
| **Last updated** | — |
| **Active milestone** | — |

---

## Layer architecture

```
Tokens → Components → Patterns → Templates
```

Every decision in Panda traces back to a token.
Every component is built from tokens.
Every pattern composes components.
Every template composes patterns.

## Platforms

Panda targets web (desktop + mobile), iOS, and Android from the same token layer.

## Documentation

- [Tokens](tokens) — Color, spacing, typography, radius, shadow, motion
- [Components](components) — UI elements built from tokens
- [Patterns](patterns) — Recurring compositions of components
- [Templates](templates) — Full-screen structures
- [Conventions](conventions) — Naming rules, process, governance

## Session history

See [registry/changelog.md](https://github.com/Boozt-Team-Panda/PandaDS/blob/main/registry/changelog.md) for the full decision history.

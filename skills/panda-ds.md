---
name: panda-ds
description: >
  Full context skill for Panda, the Boozt design system. Use this skill whenever
  the user mentions Panda, asks about design tokens, components, patterns, or
  templates in the context of Boozt's design work. Also trigger when the user
  asks to audit, document, extend, or implement anything related to the Panda DS —
  even if they don't say "Panda" explicitly but the context is clearly a Boozt
  design system task. This skill speaks both designer and developer language and
  ensures all outputs are consistent with Panda's layered structure and
  multi-platform philosophy (Figma-native, platform-agnostic output).
---
 
# Panda Design System — Skill
 
Panda is Boozt's internal design system. It lives in Figma and is the single
source of truth for design decisions across all platforms. The skill's job is to
help Claude reason about, document, extend, and communicate Panda in a way
that's useful to both designers and developers.
 
---
 
## System Architecture
 
Panda is organized in four layers, bottom-up:
 
```
Tokens
  └── Components
        └── Patterns
              └── Templates
```
 
Every decision in Panda traces back to a token. Every component is built from
tokens. Every pattern composes components. Every template composes patterns.
When in doubt, go one layer down.
 
### Layer 1 — Tokens
The atomic values of the system. No UI logic, just raw design decisions.
 
Categories:
- **Color** — semantic (e.g. `color.action.primary`) and primitive (`color.blue.500`)
- **Typography** — typefaces, sizes, weights, line-heights, letter-spacing
- **Spacing** — scale-based (e.g. `space.4` = 16px) used for padding, margin, gap
- **Radius** — corner radius values
- **Shadow** — elevation levels
- **Motion** — duration and easing tokens
- **Breakpoints** — platform breakpoint definitions (mobile, tablet, desktop)
 
> 📂 When token documentation is available, load `references/tokens.md` for
> the full token list and naming conventions.
 
### Layer 2 — Components
Discrete UI elements built entirely from tokens. Each component has:
- **Anatomy** — labeled parts (e.g. Button: container, label, icon)
- **Variants** — visual alternatives (e.g. primary, secondary, ghost)
- **States** — default, hover, active, disabled, focus, loading, error
- **Sizes** — defined using spacing + typography tokens (S, M, L or equivalent)
- **Platform notes** — any behavioral differences between iOS, Android, Web
 
> 📂 When component documentation is available, load `references/components.md`.
 
### Layer 3 — Patterns
Recurring compositions of components that solve a specific UX problem.
Examples: form layout, card grid, empty state, confirmation dialog.
 
Patterns include:
- **Intent** — what problem this solves
- **Component recipe** — which components are used and how
- **Do / Don't** — usage guidance with rationale
- **Platform adaptations** — how layout or interaction shifts per platform
 
> 📂 When pattern documentation is available, load `references/patterns.md`.
 
### Layer 4 — Templates
Full-page or full-screen compositions built from patterns. These are the closest
thing to actual product screens — they define structure, not content.
 
Templates always specify:
- Target platform (iOS / Android / Web mobile / Web desktop)
- Layout grid used
- Responsive behavior or native adaptation
 
> 📂 When template documentation is available, load `references/templates.md`.
 
---
 
## Platform Philosophy
 
Panda is **Figma-native, platform-agnostic**. This means:
 
- The source of truth is always a Figma file
- Token names and component semantics are designed to map cleanly to any platform
- Output must never be web-only by default — always consider all three targets
 
| Platform     | Layout paradigm   | Key constraint                         |
|--------------|-------------------|----------------------------------------|
| Web desktop  | CSS Grid/Flexbox  | Fluid + breakpoints                    |
| Web mobile   | CSS Flexbox       | Touch targets, thumb zones             |
| iOS          | UIKit / SwiftUI   | Safe areas, native nav, HIG compliance |
| Android      | Jetpack Compose   | Material You baseline, adaptive layout |
 
When generating specs or documentation, always flag platform-specific
considerations rather than defaulting to web assumptions.
 
---
 
## Output Standards
 
When Claude produces output using this skill, it should default to:
 
### For designers
- Use Figma-friendly language (frames, auto layout, constraints, variables)
- Reference token names as they would appear in a Figma variable collection
- Describe component anatomy using part names, not CSS selectors
- Include Do/Don't guidance when documenting patterns
- Output should be scannable: short descriptions, tables, labeled sections
 
### For developers
- Map token names to likely implementation equivalents (CSS custom properties,
  Swift constants, Compose tokens) — but note these are inferred until a
  reference file confirms them
- Describe component states in terms of props and conditions, not just visuals
- Flag which behaviors are platform-native vs. custom-built
 
### General rules
- Never invent a token name — if no reference file exists, use descriptive
  placeholders like `[color.token.TBD]` and flag it clearly
- Never collapse layers — a pattern is not a component, a template is not a page
- When documenting anything new, follow the layer it belongs to and note which
  reference file it should eventually be added to
- If asked about something Panda doesn't seem to cover yet, say so and offer
  to draft a proposal following the correct layer structure
 
---
 
## Adding Reference Files Later
 
When Panda documentation becomes available, drop files into `references/` and
update this table:
 
| File                        | Status      | Contains                            |
|-----------------------------|-------------|-------------------------------------|
| `references/tokens.md`      | 🔜 Pending  | Full token list + naming convention |
| `references/components.md`  | 🔜 Pending  | Component library + anatomy         |
| `references/patterns.md`    | 🔜 Pending  | Pattern catalog + usage guidance    |
| `references/templates.md`   | 🔜 Pending  | Template library + platform specs   |
| `references/figma-api.md`   | 🔜 Pending  | Figma variable structure + API keys |
 
Until these exist, Claude operates from this SKILL.md alone and clearly marks
any inferred information.
 
---
 
## Skill Behavior Summary
 
| User asks about…            | Claude does…                                           |
|-----------------------------|--------------------------------------------------------|
| A token                     | Explains it, maps it to all platforms, flags if TBD   |
| A component                 | Documents anatomy, variants, states, platform notes   |
| A pattern                   | Describes intent, recipe, do/don't, adaptations       |
| A template                  | Defines structure, grid, platform targets             |
| Something cross-layer        | Traces from template → pattern → component → token    |
| Something not in Panda yet  | Proposes addition using correct layer and format      |
| Design ↔ dev handoff        | Speaks both languages in the same output              |
 

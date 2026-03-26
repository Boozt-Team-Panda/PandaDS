---
name: spec-writer
description: >
  Converts confirmed Panda DS decisions into complete, usable design system
  specifications. Use this skill after session-engine has produced decisions,
  or whenever the user says "write the spec for [DS-ID]", "document this
  token", "create the component spec", "write up [token/component name]",
  "document this decision", "add this to the design system docs", or
  "generate the spec entry". This skill produces two outputs simultaneously:
  a human-readable specification (for designers in Figma, developers
  implementing it, and the team reviewing it) AND a JSON entry ready to
  merge into the designSystem section of panda-registry.json. Only runs on
  decisions with status stable or draft — never on tbd or unverified.
  Always loads ds-roadmap and panda-ds before running.
---
 
# spec-writer — DS Specification Generator
 
This skill is what closes the gap between "we decided it" and "it's documented."
 
Every DS- decision that touches a token, component, pattern, or template
needs a spec entry before it can be implemented. This skill writes that entry —
in both human-readable and machine-readable format — from the decision alone.
 
---
 
## Before Writing
 
Always load:
1. **`ds-roadmap`** — current milestone, brand token strategy, platform requirements
2. **`panda-ds`** — layer definitions, naming conventions, platform philosophy,
   output standards for designers and developers
 
Only run on decisions with `status: "stable"` or `status: "draft"`.
Never run on `tbd` or `unverified` — those must be resolved in a session first.
 
---
 
## Input
 
The user provides one of:
- A decision ID (e.g. `DS-1-003`)
- A token/component name (e.g. `spacing.primitive.4`)
- A description of what needs to be specced
 
Claude resolves the input to a specific DS- entry from the registry or
current conversation context before proceeding.
 
---
 
## Step 1 — Layer Classification
 
Determine which layer this spec belongs to:
 
| If the decision is about… | Layer |
|---------------------------|-------|
| Raw values (spacing, color, typography, radius, motion, sizing) | Token |
| A reusable UI element (button, input, card, modal) | Component |
| A recurring composition of components | Pattern |
| A full-page or full-screen layout | Template |
 
If classification is ambiguous, apply the rule from `panda-ds`:
**when in doubt, assign to the lower layer.**
 
---
 
## Step 2 — Generate the Spec
 
### Token Spec
 
Produce for every token decision:
 
**Human-readable:**
```
TOKEN SPEC
──────────────────────────────────────────
Name:         [full token name]
Layer:        primitive | semantic | component
Category:     Spacing | Color | Typography | Sizing | Radius | Motion
Value:        [resolved value or TBD]
Alias:        [primitive it maps to, if semantic] or —
Description:  [What this token represents. One sentence.]
Usage:        [When to use this. One sentence.]
Don't use for: [Common misuse to avoid. One sentence.]
 
Platform mapping:
  Web:     [CSS custom property name or TBD]
  iOS:     [Swift/SwiftUI constant name or TBD]
  Android: [Compose token name or TBD]
 
Adoption status:
  Web:     stable | draft | tbd
  iOS:     stable | draft | tbd
  Android: stable | draft | tbd
 
Decision source: [DS-N-NNN · Session N · YYYY-MM-DD]
Open threads:    [Any OPN- that affects this token, or —]
──────────────────────────────────────────
```
 
**JSON entry** (for `designSystem.tokens` array in panda-registry.json):
```json
{
  "id": "TOK-NNN",
  "name": "token.name",
  "category": "Spacing | Color | Typography | Sizing | Radius | Motion",
  "layer": "primitive | semantic | component",
  "value": "value or TBD",
  "alias": "aliased.token.name or null",
  "description": "One sentence.",
  "status": "stable | draft | tbd",
  "platforms": {
    "web": "stable | draft | tbd",
    "ios": "stable | draft | tbd",
    "android": "stable | draft | tbd"
  },
  "session": "SN"
}
```
 
---
 
### Component Spec
 
Produce for every component decision:
 
**Human-readable:**
```
COMPONENT SPEC
──────────────────────────────────────────
Name:      [ComponentName]
Layer:     Component (Layer 2)
Intent:    [What UX problem this solves. One sentence.]
 
Anatomy:
  · [Part name] — [What it does]
  · [Part name] — [What it does]
 
Variants:    [primary | secondary | ghost | danger | ...]
States:      [default | hover | active | disabled | focus | loading | error]
Sizes:       [S | M | L or custom scale]
 
Token dependencies:
  · [token.name] — [which part uses it]
  · [token.name] — [which part uses it]
 
Platform notes:
  Web:     [Any CSS or layout-specific behaviour]
  iOS:     [Any UIKit/SwiftUI-specific behaviour or HIG compliance note]
  Android: [Any Compose-specific behaviour or Material You note]
 
Do:
  · [Correct usage rule]
Don't:
  · [Anti-pattern to avoid]
 
Decision source: [DS-N-NNN · Session N · YYYY-MM-DD]
Open threads:    [Any OPN- that affects this component, or —]
──────────────────────────────────────────
```
 
**JSON entry** (for `designSystem.components` array):
```json
{
  "id": "CMP-NNN",
  "name": "ComponentName",
  "intent": "One sentence.",
  "anatomy": ["part1", "part2"],
  "variants": ["primary", "secondary"],
  "states": ["default", "hover", "active", "disabled", "focus"],
  "sizes": ["S", "M", "L"],
  "tokenDeps": ["token.name.one", "token.name.two"],
  "platformNotes": {
    "web": "Note or empty string",
    "ios": "Note or empty string",
    "android": "Note or empty string"
  },
  "dos": ["Correct usage rule"],
  "donts": ["Anti-pattern to avoid"],
  "status": "stable | draft | tbd",
  "platforms": {
    "web": "stable | draft | tbd",
    "ios": "stable | draft | tbd",
    "android": "stable | draft | tbd"
  },
  "session": "SN"
}
```
 
---
 
### Pattern Spec
 
**Human-readable:**
```
PATTERN SPEC
──────────────────────────────────────────
Name:      [PatternName]
Layer:     Pattern (Layer 3)
Intent:    [What UX problem this solves. One sentence.]
 
Component recipe:
  · [ComponentName] — [role in this pattern]
  · [ComponentName] — [role in this pattern]
 
Do:
  · [Correct usage rule]
Don't:
  · [Anti-pattern to avoid]
 
Platform adaptations:
  Web:     [How layout or interaction shifts]
  iOS:     [How layout or interaction shifts]
  Android: [How layout or interaction shifts]
 
Decision source: [DS-N-NNN]
──────────────────────────────────────────
```
 
**JSON entry** (for `designSystem.patterns` array):
```json
{
  "id": "PAT-NNN",
  "name": "PatternName",
  "intent": "One sentence.",
  "componentRecipe": ["ComponentA", "ComponentB"],
  "dos": ["Rule"],
  "donts": ["Anti-pattern"],
  "platformAdaptations": {
    "web": "Note",
    "ios": "Note",
    "android": "Note"
  },
  "status": "stable | draft | tbd",
  "session": "SN"
}
```
 
---
 
### Template Spec
 
**Human-readable:**
```
TEMPLATE SPEC
──────────────────────────────────────────
Name:         [TemplateName]
Layer:        Template (Layer 4)
Platform:     Web desktop | Web mobile | iOS | Android
Intent:       [What screen this represents. One sentence.]
Layout grid:  [12-column CSS Grid | Native stack | etc.]
Patterns used:
  · [PatternName] — [where/how used]
Responsive:   [How it adapts across breakpoints or orientations]
Decision source: [DS-N-NNN]
──────────────────────────────────────────
```
 
**JSON entry** (for `designSystem.templates` array):
```json
{
  "id": "TPL-NNN",
  "name": "TemplateName",
  "platform": "Web desktop | Web mobile | iOS | Android",
  "intent": "One sentence.",
  "layoutGrid": "12-column CSS Grid",
  "patternsUsed": ["PatternA", "PatternB"],
  "responsiveBehavior": "Note",
  "status": "stable | draft | tbd",
  "session": "SN"
}
```
 
---
 
## Closing Block
 
After every spec output:
 
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPEC READY
[TOK/CMP/PAT/TPL]-NNN · [name] · [layer]
Human spec ↑ — share with team / paste into Figma/Confluence
JSON entry ↑ — merge into designSystem.[tokens|components|patterns|templates]
  in registry/panda-registry.json
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
 
---
 
## Quality Rules
 
- Never invent a token name — use `[token.name.TBD]` and flag it
- Never assign `stable` status unless the decision source is `stable`
- Platform mapping fields default to `tbd` — never infer a platform name
  without a decision source
- If a token depends on an open OPN-, its platform status stays `tbd`
  until that OPN- is resolved
- One spec per decision. Don't batch multiple decisions into one spec.
 

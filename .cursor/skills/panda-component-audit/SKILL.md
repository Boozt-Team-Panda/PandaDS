---
name: panda-component-audit
description: Audits a single component page in the Panda DS Figma file against the AI-ready documentation standard — checks for the main component, header (title + purpose), Semantic-only token bindings, dual Human/AI Agent descriptions, variant docs with use examples, plus the Examples, Accessibility, and Documentation sections (do's/don'ts, binding chart, nested-components tree, anatomy wireframe, special info). Returns a gap report plus a fill plan, then asks whether to execute the plan and write the missing content into Figma. Use this skill whenever a team member mentions auditing a Panda DS component page, "checking if a component is ready", reviewing a Figma component for completeness, validating a component spec, or pastes a Figma link to a Panda DS page. Trigger even when phrased casually — "audit this", "is the Button page done?", "check the new Input component" — the user wants this skill.
---

# Panda Component Audit

Audits one component page in the Panda DS Figma file against the documentation standard for
AI-ready components. Returns a structured gap report and a fill plan, then offers to write the
missing content into Figma after explicit user approval.

This is a **page-level** audit — one component at a time. The skill is self-contained: it carries
its own vocabulary, layer rules, and Panda-specific quirks below. Don't pull from other skills
for context.

---

## 1. Inputs

- A Figma URL pointing to a page in the Panda DS file.
- If no URL was pasted in the conversation, ask exactly once:
  > Drop the Figma page link.
  Don't proceed without it. Don't guess the page from context.
- If the URL points to a node that isn't a top-level page (e.g., a nested frame), surface that
  and confirm the correct page node before continuing.

The file key is read from the URL — don't hardcode it. The Panda DS file may move or branch.

---

## 2. Tools and reading order

Use the Figma MCP. Sequence matters:

1. `Figma:get_metadata` on the page node — enumerate top-level frames and sections.
2. `Figma:get_design_context` with `excludeScreenshot=False` — authoritative source for variant
   property names, option ordering, and structural truth. Never reconstruct variant data from
   metadata alone — naming drift here breaks downstream agent compatibility.
3. `Figma:get_variable_defs` — verify every binding resolves to the Semantic collection.
4. `Figma:get_screenshot` — visual confirmation when reporting visual gaps (anatomy, do's/don'ts).
5. `Figma:use_figma` — only after explicit "yes" to execute the plan. Never before.

If `Figma:get_metadata` on `0:1` returns only first-page content, use `use_figma` with
`figma.root.children` to enumerate pages reliably.

---

## 3. The audit checklist

Group findings by section. Don't dump a flat list — the team reads this for sync, structure helps.

### 3.1 — Main component (in the "Component" section)

**Component frame**
- A frame or section labelled "Component" exists on the page.
- Inside it, exactly one main component is present (the design source of truth — not an instance).
- If multiple candidates exist or the main component is ambiguous, **stop and ask**. Don't guess.

**Header (above or adjacent to the Component frame)**
- Contains a Title — the component name, matching the Figma component's name.
- Contains a short description — one sentence, the component's purpose.
- If the header is missing or its structure differs from the standard, ask before flagging — it
  may be a layout variation rather than a gap.

**Token bindings**
- Every styled property (fill, stroke, typography, spacing, radius, etc.) of every variant binds
  to a Semantic-collection variable.
- No bindings to `_Primitives`. No raw hex / px / font values.
- Allowed exception: shadow tints — the 5 unaliased semantic shadow tokens are intentional, not
  errors.
- For each unbound or wrongly-bound property, capture: variant name, layer, property, current
  value, suggested Semantic token (if obvious from naming).

**Component description — dual format**
The description must contain BOTH lines, in this exact format:

  Human description: {context in which the component should be used, in human-friendly syntax}
  AI Agent: {same purpose, optimized for AI models — concrete, structured, machine-parseable}

If either line is missing, malformed, or the two have been merged into one paragraph — flag it.
Both lines are required.

**Variants**
For every variant of the main component:
- Has a description specific to that variant (not a copy of the generic component description).
- Has at least one written best-use example.
- A variant with only a name and no description/example → flag.

### 3.2 — Examples section
- A frame or section named "Examples" exists.
- Contains at least one realistic usage example showing the component in real context (a card,
  a form, a toolbar — not just the bare component on a blank canvas).
- Empty or missing → flag.

### 3.3 — Accessibility section
- A frame or section named "Accessibility" exists.
- Covers at minimum: keyboard interaction, focus state behavior, contrast notes, and any
  platform-specific a11y guidance.
- Empty section is treated as missing — undocumented = doesn't exist.

### 3.4 — Documentation section

Must include all of:

- **Do's / Don'ts** — at least one Do and one Don't, each tied to a concrete scenario.
- **Binding chart per variant** — table or visual mapping every variant to the Semantic tokens
  it consumes. Should match the actual bindings found in the Component frame; mismatches between
  the chart and the live bindings count as a documentation drift and must be flagged.
- **Tree of nested components** — if the component composes others, a tree showing the hierarchy.
  If standalone, the section can be omitted but should still be acknowledged (e.g., "Standalone
  — no nested components") so a reader doesn't assume it was forgotten.
- **Component anatomy** — labelled wireframe with spec annotations (sizes, paddings, gaps).
  Annotations should reference token names, not raw px values.
- **Special information** — only if the component has unusual constraints, browser quirks,
  platform divergence, or migration notes. Optional, but if it should be there and isn't, flag.

---

## 4. Output format

After running the audit, return ONE response with this exact structure:

```
# Audit: [Component name]

Page: [Figma URL]
Run: [date]

## ✅ Present
One short line per section that's complete. No bullet expansion.

## ❌ Missing or incomplete
Grouped by section (Component / Examples / Accessibility / Documentation).
For each gap:
- **What**: short label.
- **Where**: section / frame name.
- **Severity**: blocker · important · nice-to-have.
- **Why it matters**: one sentence.

## 📋 Fill plan
Numbered steps. Each step states exactly what would be written, where it lands, and which Figma
tool would do it. Drafts of text content (Human/AI descriptions, do's/don'ts, variant text) are
written here in full so the user can read and approve before any Figma write happens.

## ⚠️ Open questions for the team
Anything that needs a human decision and would block the plan. Examples:
- "What should TransparentDense's Human description say?"
- "Is the Examples section meant to live on this page or in the patterns library?"
```

After the report, ask exactly:

> Want me to execute this plan? **Yes** = I write it into Figma. **No** = leave it for the team
> (common reasons: component isn't ready, team is unsure how to execute, conflicts with Devs).

---

## 5. Executing the plan

Only after explicit "yes". Never auto-execute.

1. Re-read each step. If anything in the plan was a placeholder ("draft Human description goes
   here"), STOP and ask the user to provide the real content. Don't ship guesses into the source
   of truth.
2. Use `Figma:use_figma` to write changes. Group writes by frame to minimize calls.
3. After each batch, call `Figma:get_screenshot` on the affected frame and visually confirm the
   change landed correctly. Surface anything that looks wrong instead of moving on.
4. Re-run the relevant audit checks on the modified frame to confirm the gap is closed.
5. Produce a short close-out: what was filled, what was deferred to humans, any new gaps that
   surfaced during execution.

---

## 6. Things this skill does NOT do

- Does not create the main component itself. If the Component frame is empty, that's a blocker —
  flag it and stop.
- Does not change token values or fix wrong bindings. Flags them in the plan; resolution is the
  user's call.
- Does not audit multiple pages in one run. One page per invocation.
- Does not write production code. Figma-side audit only.

---

## 7. Panda-specific quirks to remember

- **TransparentDense button** only supports `hasIcon=Right`. Don't flag missing variants.
- **Success and Destructive button hover/pressed** use swapped token names in Figma — visually
  correct, naming confusing. Don't flag this as a binding error.
- **Shadow tints** are the only unaliased semantic tokens (5 total). Don't flag them as missing
  aliases.
- **Naming mismatch**: semantic uses `brand-accent/clubboozt`, primitives use `brand/club`.
  Acknowledged drift, not an error.
- **Undocumented = doesn't exist**. Empty descriptions count as missing, not partial.

---

## 8. When to ask vs. when to proceed

**Ask the user when:**
- The Figma link is missing.
- The Component frame contains multiple candidates and the main component is ambiguous.
- The Header is missing or its structure doesn't match the standard.
- A required text block (Human/AI description, variant description, do's/don'ts) needs drafting
  and the user hasn't authorized placeholder content.

**Proceed without asking when:**
- A whole section is clearly missing (no frame matching the expected name).
- A token binding is clearly wrong (raw hex, primitive ref, no binding at all).
- A variant has no description content at all.

---

## 9. Style notes

- Communication style: direct, practical. Skip preamble. Flag process gaps rather than silently
  working around them. The audience is design and development teammates who want sync, not prose.
- Precision over approximation. Match exact Figma structure (names, ordering, property values).
  Non-negotiable for agent compatibility — small naming drift breaks downstream automation.
- Visual confirmation before acceptance, especially for any change involving color or layout.

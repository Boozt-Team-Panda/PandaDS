# Panda DS — Token usage rules

This document defines how to use Panda's design tokens when building components. It covers the logic behind each token group, accessibility requirements, and the rules that keep the system consistent across Boozt and Outlet Nation on Web, iOS, and Android.

If a decision is undocumented, it doesn't exist. When in doubt about which token to use, check this file first — then ask.

---

## Architecture overview

Panda uses two token layers:

**Primitives** — raw values with no meaning. Color ramps, spacing numbers, type scales. Hidden from publishing. Designers never apply these directly. Think of them as the paint cans in a warehouse — you don't hand a can to a customer, you use it to make something.

**Semantic** — purpose-driven names that alias to primitives. These are what designers and developers consume. `color/primary` means "the main action color" regardless of whether that resolves to Boozt's dark plum or ON's vivid green. Semantic tokens are published to the Figma library and have two modes: Boozt and Outlet Nation.

The rule is simple: **components only consume semantic tokens. Never reference a primitive directly.** If you need a value that doesn't have a semantic token, that's a gap in the system — flag it, don't hardcode.

---

## Color

### Token groups and when to use each

**Primary group** — the brand's signature interactive color.

- `color/primary` — button backgrounds, link text, active tab indicators, toggle fills, primary CTAs. This is the single most important brand differentiator: dark plum for Boozt, vivid green for ON.
- `color/primary-foreground` — text and icons sitting on top of primary fills. Always pair with `primary`. Never use this on a non-primary background.
- `color/primary-dim` — hover state on primary elements. Slightly muted version of the brand color. Use for `:hover` on buttons, link hover, pressed breadcrumb.
- `color/primary-container` — pressed/active state. Deeper than dim. Use for `:active` on buttons, selected chip background, active toggle track.
- `color/primary-container-foreground` — text on pressed/active primary surfaces.

**Secondary group** — the neutral interactive surface.

- `color/secondary` — secondary button fills, toggle off-state backgrounds, less prominent interactive surfaces. In Boozt this is white (outlined buttons), in ON it's a filled surface.
- `color/secondary-foreground` — text on secondary surfaces. In Boozt this is black (dark text on white), in ON this adapts to the dark surface.

**Background / foreground** — the page canvas.

- `color/background` — the root page color behind everything. White for Boozt, near-black for ON.
- `color/foreground` — default highest-contrast text color on the page background. Headlines, primary body text, any text that needs maximum readability.

**Surface group** — elevated containers sitting on the background.

- `color/surface` — card backgrounds, table fills, any container that lifts off the page. Must be visually distinct from `background` — if they're the same color, the elevation is invisible.
- `color/surface-foreground` — text inside surface containers. Body text in cards, table cell text, modal content.
- `color/surface-container` — brand-tinted surface for sections that need warmth (Boozt's cream) or personality. Use for hero blocks, featured sections, promotional containers. Not for generic UI.
- `color/muted` — de-emphasized fills. Disabled input backgrounds, skeleton loading, chip fills, striped table rows, selected row highlight. The "I'm here but not important" surface.
- `color/muted-foreground` — text on muted surfaces. Helper text, placeholder text, disabled labels, metadata, timestamps. Must be readable but clearly secondary.

**Edge colors** — borders, outlines, focus.

- `color/border` — default structural borders. Card outlines, input resting borders, table dividers, separator lines. Subtle — should not compete with content.
- `color/outline` — active/emphasized borders. Input focus borders, selected card outlines, form field active state. One step more prominent than `border`.
- `color/ring` — keyboard focus indicator. Only appears on `:focus-visible`. In Boozt this is a neutral mid-gray; in ON it's the brand green. This token exists specifically for accessibility — it must always be visually distinct from `outline`.

**Utility group** — feedback and status colors. Each status (info, success, error, warning) follows a 6-token pattern:

- `*/[status]` — the default status color. Icon fills, badge backgrounds, status indicator dots, inline status text.
- `*/-foreground` — text on the status color fill. White for info/success/error (dark backgrounds need light text), black for warning (yellow needs dark text for contrast).
- `*/-hover` — interactive hover state on status-colored elements.
- `*/-pressed` — interactive pressed/active state.
- `*/-container` — light tinted background for banners, toasts, inline alerts. These are pastels — they provide context without screaming.
- `*/-container-foreground` — text inside status containers. Always dark (neutral/800) for readability on light tinted backgrounds.

**Brand accents** — sub-brand colors for Club Boozt and BooztPay.

- `color/brand-accent/clubboozt` + `-foreground` — loyalty badges, rewards UI, member-exclusive sections.
- `color/brand-accent/booztpay` + `-foreground` — payment method badges, checkout trust marks, wallet UI.

**Shadow tints** — the color component of drop shadow effects.

- `color/shadow/xs` through `xl` — same hue (#121212) at increasing alpha (3% → 15%). Applied as the shadow color property in effect styles, not as fills. The shadow geometry (x, y, blur, spread) is defined when building the actual shadow effects on components.

### Color accessibility rules

**Contrast ratios are non-negotiable.** These are WCAG 2.1 AA minimums:

- Normal text (under 18px, or under 14px bold): **4.5:1** against its background.
- Large text (18px+ regular, or 14px+ bold): **3:1** against its background.
- UI components and graphical objects (icons, borders, focus rings): **3:1** against adjacent colors.

**How this maps to Panda tokens:**

Every `-foreground` token must pass contrast against its paired surface. `primary-foreground` must pass against `primary`. `surface-foreground` must pass against `surface`. There are no exceptions.

The `muted-foreground` token is the most at-risk — it's intentionally low-emphasis, but it still must hit 4.5:1 against `muted` and against `background` (since muted text can appear on either). If it doesn't, the value is wrong — raise it, don't ignore it.

**The foreground pairing rule:** Never use a foreground token without its matching surface. `primary-foreground` on `surface` is a bug, even if it happens to be readable — the contract is between paired tokens, and when the brand mode switches, unpaired combinations will break.

**Focus indicators:** `color/ring` must have 3:1 contrast against the background it appears on AND against the element's own border. This is why `ring` is a separate token from `outline` — they need to serve different contrast contexts. For ON, the green ring on dark backgrounds is high-contrast by nature. For Boozt, the neutral/500 ring on white backgrounds needs careful checking.

**Utility containers on dark backgrounds:** The pastel container tokens (info/50, success/50, etc.) were designed for light backgrounds. On ON's dark canvas, these light pastels will appear as bright floating rectangles. When ON mode is fully built, evaluate whether these containers need mode-specific values (e.g., using the 500/600 stops as dark-mode containers with light text).

**Color alone must never be the only indicator.** A red border on an error input is not sufficient — add an error icon or error text alongside it. A green checkmark on a success state must have a label. This isn't a token rule, it's a component rule — but it starts with the tokens: utility colors are *supplementary*, not *sufficient*.

### Color logic: why the modes differ the way they do

Boozt's surface system works light-to-dark: white background → gray surfaces → dark text. The brand's warmth comes from the cream tones in `surface-container` (boozt/100).

ON's surface system inverts: dark background → slightly lighter dark surfaces → light text. The brand's energy comes from the green accent that appears in `primary`, `ring`, and interactive states.

The neutral ramp is shared — both brands use the same grays. What changes is which end of the ramp each brand draws from. Boozt lives in the 50–200 range for surfaces and 600–800 for text. ON lives in the 600–800 range for surfaces and 50–300 for text.

Utility colors (info, success, error, warning) are intentionally shared across brands. Status feedback must feel universal — a user shouldn't have to re-learn what red means when switching between Boozt and ON.

---

## Spacing

### Token groups

Panda semantic spacing is organized by *intent* — why you're spacing, not just how much. Four groups:

**flow/** — vertical distance between stacked elements. Use when elements stack top-to-bottom: paragraphs, form fields, card rows, sections.


| Token      | px  | When to use                                       |
| ---------- | --- | ------------------------------------------------- |
| `flow/xs`  | 4   | Label to helper text, error message below input   |
| `flow/sm`  | 8   | Between form fields, list items, radio options    |
| `flow/md`  | 16  | Between paragraphs, heading to content, card rows |
| `flow/lg`  | 24  | Between content groups, form section to section   |
| `flow/xl`  | 32  | Between page sections                             |
| `flow/2xl` | 48  | Major section break, hero to content              |


**inline/** — horizontal distance between side-by-side elements. Use when elements sit in a row: button groups, tags, nav items.


| Token       | px  | When to use                                |
| ----------- | --- | ------------------------------------------ |
| `inline/xs` | 4   | Icon to label, dot to text                 |
| `inline/sm` | 8   | Between buttons, tags, breadcrumb segments |
| `inline/md` | 12  | Nav item gap, toolbar groups               |
| `inline/lg` | 16  | Card column gaps, filter chips             |
| `inline/xl` | 24  | Wide column gaps, sidebar to content       |


**padding/** — inset inside containers. Use for the space between a container's edge and its content.


| Token         | px  | When to use                               |
| ------------- | --- | ----------------------------------------- |
| `padding/xs`  | 4   | Pills, small badges, tiny tags            |
| `padding/sm`  | 8   | Tags, compact buttons, tooltips           |
| `padding/md`  | 12  | Input fields, small cards, dropdown items |
| `padding/lg`  | 16  | Standard cards, modals, dropdowns         |
| `padding/xl`  | 24  | Large cards, desktop sections, dialogs    |
| `padding/2xl` | 32  | Page content area, hero block insets      |


**boundary/** — outer clearance between distinct regions. Use for the space that says "these are separate concerns."


| Token          | px  | When to use                                      |
| -------------- | --- | ------------------------------------------------ |
| `boundary/sm`  | 24  | Between related card groups, filter to results   |
| `boundary/md`  | 32  | Between content sections, main to aside          |
| `boundary/lg`  | 48  | Page-level section breaks, checkout step to step |
| `boundary/xl`  | 64  | Header to content, content to footer             |
| `boundary/2xl` | 96  | Hero-level clearance, landing page sections      |


### Spacing rules

**Always use the intent token, even when the values match.** `flow/md`, `inline/lg`, and `padding/lg` all resolve to 16px today. Use the one that describes what you're doing. When iOS needs card padding at 20px but vertical flow at 16px, only the intent-based name lets us adjust one without breaking the other.

**The 4px grid is sacred.** Every spacing value is a multiple of 4. If you ever feel the urge to use 5px, 7px, 10px, or 14px — you're off-grid. The closest grid value is the right one. This matters because sub-pixel rendering differs across browsers and platforms; 4px multiples render crisply everywhere.

**Don't stack spacing tokens.** If you need 24px between a heading and a paragraph, use `flow/lg` (24px) — don't use `flow/sm` (8px) as margin-bottom on the heading and `flow/md` (16px) as margin-top on the paragraph. Stacking creates invisible math that breaks when someone changes one side. One token, one gap.

**Padding is symmetric unless you have a reason.** Use the same `padding/` token on all four sides of a container. If a card needs 16px on all sides, that's `padding/lg` everywhere. If a card needs 24px vertical and 16px horizontal (common on mobile), use `padding/xl` top/bottom and `padding/lg` left/right — document the exception.

### Spacing and touch targets

Mobile touch targets must be at least 48×48dp (Android) or 44×44pt (iOS). Spacing tokens interact with this:

- A 24px icon (`icon-size/sm`) in a button needs at least 12px vertical padding (`padding/md`) to reach 48px total height.
- A list item with `flow/sm` (8px) between items means the tap target of each item must be internally tall enough — the gap between items does not count as part of the touch target.
- Inline elements spaced with `inline/xs` (4px) will have adjacent touch targets that nearly overlap. On mobile, increase to `inline/sm` (8px) minimum for tappable elements.

---

## Typography

### Text styles and their use

Every text style is decomposed into three tokens: `font-size`, `line-height`, and `font-weight`. The `font-family` is a single mode-switched token — Suisse Int'l for Boozt, ABC Favorit for ON.

**Display** — for marketing and hero content. Not for UI.


| Style        | Size | Line height | Weight       | Use                                              |
| ------------ | ---- | ----------- | ------------ | ------------------------------------------------ |
| `display-lg` | 34px | 40px        | medium (500) | Hero headlines, landing page titles, splash text |
| `display-md` | 28px | 32px        | medium (500) | Page titles, featured product names              |
| `display-sm` | 22px | 28px        | medium (500) | Sub-section titles, promotional headings         |


**Headline** — the bridge between marketing and UI.


| Style         | Size | Line height | Weight       | Use                                                   |
| ------------- | ---- | ----------- | ------------ | ----------------------------------------------------- |
| `headline-md` | 20px | 24px        | medium (500) | PDP section names, modal titles, filter group headers |


**Title** — for UI element labels and navigation.


| Style      | Size | Line height | Weight       | Use                                              |
| ---------- | ---- | ----------- | ------------ | ------------------------------------------------ |
| `title-lg` | 17px | 24px        | medium (500) | Product names, card headers, list item titles    |
| `title-md` | 15px | 20px        | medium (500) | Form group labels, table headers, tab labels     |
| `title-sm` | 13px | 16px        | medium (500) | Compact headers, badge labels, sidebar nav items |


**Body** — for readable content.


| Style     | Size | Line height | Weight        | Use                                                            |
| --------- | ---- | ----------- | ------------- | -------------------------------------------------------------- |
| `body-lg` | 17px | 24px        | regular (400) | Product descriptions, article body, long-form content          |
| `body-md` | 15px | 20px        | regular (400) | Default UI text: inputs, buttons, list content, dropdown items |
| `body-sm` | 13px | 16px        | regular (400) | Helper text, timestamps, metadata, price breakdowns            |


**Label** — for the smallest readable text.


| Style      | Size | Line height | Weight        | Use                                                      |
| ---------- | ---- | ----------- | ------------- | -------------------------------------------------------- |
| `label-sm` | 11px | 16px        | regular (400) | Input labels, footnotes, legal micro-copy, overline text |


### Typography rules

**11px is the floor.** Nothing in the system goes below `label-sm` (11px). At smaller sizes, text becomes unreadable on low-density Android screens and fails accessibility audits. If you think you need 10px or 9px, the content needs to be redesigned — not shrunk.

**Title and body share sizes, not roles.** `title-lg` and `body-lg` are both 17px, but title is medium weight (500) and body is regular (400). Don't mix them — a product name is always title weight, a product description is always body weight. The weight difference is what creates hierarchy at the same size.

**Line height ratios tighten as size increases.** Small text (11–15px) has ~1.3–1.45× line-height for readability. Large text (22–34px) has ~1.17–1.27× because display type needs tighter leading to feel cohesive. These ratios are baked into the paired line-height primitives — don't override them.

**Font-family switches automatically.** The `typography/font-family` token resolves to Suisse Int'l in Boozt mode and ABC Favorit in ON mode. You never need conditional font logic in components — the mode handles it. Both typefaces were chosen because they share similar x-heights and metric proportions, so layouts don't break when switching.

### Typography accessibility

**WCAG requires resizable text.** Users must be able to scale text to 200% without loss of content or functionality. This means:

- Never use fixed-height containers for text. A card's height must grow with its content when text is scaled.
- Never truncate critical text (product names, prices, error messages) based on a fixed line count. Truncation is acceptable for preview/teaser content (product descriptions in grids) if a full-text view exists.
- Line heights must accommodate accented characters and descenders at all scales. The Panda line-height values have 4–8px of breathing room above the font-size for this reason.

**Minimum tap target for text links:** A standalone text link must have at least 44×44pt / 48×48dp tap area. If the text itself is smaller than that, the interactive area must extend via padding. Use `padding/sm` (8px) minimum around text links.

---

## Sizing

### Icon sizing


| Token                             | Size       | Stroke                                               | When to use |
| --------------------------------- | ---------- | ---------------------------------------------------- | ----------- |
| `icon-size/xs` + `icon-stroke/xs` | 16px / 1.0 | Inline with body text, status dots, tiny affordances |             |
| `icon-size/sm` + `icon-stroke/sm` | 24px / 1.3 | Default UI icons: nav, actions, form fields          |             |
| `icon-size/md` + `icon-stroke/md` | 32px / 1.6 | Emphasized icons: empty states, feature cards        |             |
| `icon-size/lg` + `icon-stroke/lg` | 48px / 2.0 | Large icons: category thumbnails, avatars            |             |
| `icon-size/xl` + `icon-stroke/xl` | 96px / 3.0 | Display icons: hero illustrations, splash brands     |             |


**Always pair size with stroke.** A 48px icon with a 1.0 stroke looks broken — the stroke weight is designed to scale proportionally with the icon container. If you use `icon-size/lg`, use `icon-stroke/lg`.

**Icon accessibility:** Icons must have a text alternative unless they're purely decorative. For interactive icons (icon-only buttons), provide an `aria-label` (web), `accessibilityLabel` (iOS), or `contentDescription` (Android). The icon color must have 3:1 contrast against its background.

### Radius


| Token         | px   | When to use                                               |
| ------------- | ---- | --------------------------------------------------------- |
| `radius/none` | 0    | Full-bleed images, tables, hard-edged dividers            |
| `radius/sm`   | 8    | Inputs, buttons, tags, badges, small interactive elements |
| `radius/md`   | 16   | Cards, dropdowns, modals, toast notifications             |
| `radius/lg`   | 24   | Feature cards, hero images, promotional blocks            |
| `radius/xl`   | 32   | Floating action areas, bottom sheets, large panels        |
| `radius/full` | 9999 | Avatars, pill buttons, status dots, toggle thumbs         |


**Radius rule: nested elements use one step smaller.** A card with `radius/md` (16px) should have its inner image container at `radius/sm` (8px). This prevents the inner element's corners from clipping the outer element's corner rounding. The exception is `radius/full` — a circular avatar inside any card stays circular.

**Don't round single-sided borders.** If an element uses `border-left` or `border-top` as an accent (like a colored left bar on a notification), set radius to `none`. Rounded corners on single-sided borders create visual artifacts.

---

## Border width


| Token                   | px  | When to use                                                                                             |
| ----------------------- | --- | ------------------------------------------------------------------------------------------------------- |
| `border-width/hairline` | 0.5 | Subtle card outlines, decorative separators, light dividers. Retina-only — renders as 1px on 1x screens |
| `border-width/thin`     | 1   | Default borders: input fields, card strokes, table dividers, button outlines                            |
| `border-width/medium`   | 1.5 | Emphasized borders: active input focus, selected state outlines                                         |
| `border-width/thick`    | 2   | Heavy emphasis: active tab indicators, selected card accent, focus-visible rings                        |


**Focus-visible rings use `thick` (2px).** This ensures the focus indicator meets the WCAG 3:1 contrast requirement even on busy backgrounds. Combined with `color/ring`, this creates a visible, accessible focus state that works for keyboard navigation.

---

## Opacity


| Token         | Value | When to use                                           |
| ------------- | ----- | ----------------------------------------------------- |
| `opacity/0`   | 0%    | Fade-out animations, hidden states                    |
| `opacity/5`   | 5%    | Ghost hover tints, subtle pressed ripple              |
| `opacity/10`  | 10%   | Skeleton loading, light scrims, disabled overlays     |
| `opacity/20`  | 20%   | Active state backgrounds, selected row highlights     |
| `opacity/50`  | 50%   | Modal backdrops, image overlay text protection        |
| `opacity/80`  | 80%   | Heavy scrims, dark image overlays for text legibility |
| `opacity/100` | 100%  | Default state for all solid elements                  |


**Modal backdrops always use `opacity/50` as a starting point.** Below 50%, content behind the modal is too visible and distracting. Above 80%, the modal feels disconnected from the page.

**Disabled elements use `opacity/10` as an overlay on the element's own color.** Don't change the element's color for disabled states — overlay it. This keeps the element recognizable while clearly indicating it's not interactive.

---

## Brand mode behavior

### What changes between Boozt and Outlet Nation

18 of 122 semantic tokens change when switching brand mode. Every component that uses only semantic tokens will adapt automatically. Here's the mental model:

**Surfaces invert.** Boozt: light backgrounds, dark text. ON: dark backgrounds, light text. The neutral ramp flips — Boozt uses 50–200 for surfaces, ON uses 600–800.

**Primary changes identity.** Boozt's primary is a dark plum (the brand's understated elegance). ON's primary is a vivid green (the brand's energy). Both serve as button fills, but the character is completely different.

**Typography switches typeface.** Suisse Int'l (Boozt) → ABC Favorit (ON). Sizes, weights, and line-heights are shared — only the font family changes.

**Everything else stays.** Utility colors, spacing, sizing, radius, border-width, opacity, shadows — all shared. Status feedback is universal.

### Rules for mode-safe components

1. **Never hardcode a hex value.** Every color in a component must come from a semantic token. A hardcoded `#241D26` will be invisible on ON's dark background.
2. **Always use foreground pairs.** If you use `color/primary` as a fill, the text on top must be `color/primary-foreground`. Not `color/foreground`, not `color/white`, not anything else. The pair is designed to maintain contrast across both modes.
3. **Don't assume light or dark.** A component that adds a white shadow assuming a light background will look wrong on ON. A component that uses `background` for text color (because "it's dark") will break on Boozt. Use the semantic names and trust the system.
4. **Test both modes.** In Figma, switch the frame's variable mode between Boozt and ON. If anything looks wrong — unreadable text, invisible borders, wrong emphasis — the component has a hardcoded assumption. Fix it before shipping.

---

## Platform considerations

### Web (Danas)

Semantic tokens map directly to CSS custom properties. `color/primary` becomes `--color-primary`. The spacing intent tokens (`flow/`, `inline/`, `padding/`, `boundary/`) map to `gap`, `padding`, and `margin` properties — the token name tells you which CSS property it belongs to.

### iOS (Daniel)

UIKit/SwiftUI: semantic tokens become named colors in an asset catalog and spacing constants in a shared file. The `font-family` token switches between Suisse Int'l and ABC Favorit based on the brand configuration. Line heights must be set explicitly — iOS's default leading differs from the Panda values.

### Android (Rasmus)

Jetpack Compose: semantic tokens become `MaterialTheme` color/typography overrides. The 48dp minimum touch target is already covered by the spacing system — `padding/md` (12px) on a 24px icon gives exactly 48dp. Brand switching maps to Compose theme providers.

### Cross-platform rules

- Touch targets: 48×48dp minimum (Android), 44×44pt minimum (iOS), no minimum on web desktop but 44px recommended.
- Font rendering: Suisse Int'l and ABC Favorit must be tested on all three platforms. Hinting, anti-aliasing, and weight rendering differ — a weight-500 heading on Android may look heavier than on iOS.
- Spacing: The 4px grid works natively on all three platforms (4dp on Android, 4pt on iOS, 4px on web). No conversion needed.

---

## Decision log

This section tracks the reasoning behind structural choices that might otherwise look arbitrary.

**Why book weight (450) was considered then dropped for body text.** The initial build used 450 (book) for body text to give it more presence than regular (400). Testing showed that Suisse Int'l's 450 weight renders inconsistently across browsers — Chrome rounds it to 400, Safari honors it, and Firefox behavior varies. Regular (400) is universal. If body text needs more weight, use `title-`* styles instead.

**Why the spacing scale skips 2px and 6px.** The original proposal included 2px (hairline gaps) and 6px (compact pill padding). Both were cut because they violate the 4px grid — 2px is half a grid unit, 6px is 1.5 grid units. The system uses 4px as the minimum intentional gap. For sub-4px needs (icon nudges, optical alignment), use manual offsets outside the token system and document them as exceptions.

**Why utility colors are shared across brands.** Red means error in both Boozt and ON. Green means success. These are universal conventions that override brand identity. A mode-specific error color would confuse users who switch between the two brands — they'd have to re-learn what "danger" looks like. Utility colors are the shared language.

**Why `secondary` and `surface` are both neutral/50 in Boozt.** In Boozt's visual language, secondary buttons are white with outlines — they don't have a filled surface distinct from cards. The visual distinction comes from the border and the button's interactive states (hover/pressed), not from the resting fill. This means a secondary button on a surface card is visually merged at rest, which is intentional — the outline provides the boundary.

**Why shadow tints don't have primitive aliases.** Shadows are visual effects, not surface colors. They don't need to participate in the primitive → semantic alias chain because they'll never be swapped between brand modes (both brands use the same dark shadow tint). They're raw values in the semantic layer — the only 5 unaliased tokens in the system.

**Why `ring` differs from `outline`.** Both serve as "emphasized border," but for different audiences. `outline` is for the visual design (active input borders, selected states). `ring` is for assistive technology users (keyboard focus). They must be independently adjustable because a design that uses a colored `outline` for selection might need a different-colored `ring` to make focus visible against that selection state.
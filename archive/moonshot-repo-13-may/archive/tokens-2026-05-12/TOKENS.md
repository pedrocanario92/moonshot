# Infios design tokens — 2026-05-12 delivery

Transcribed from 15 PNG screenshots in this folder. Organized by category. Source filename noted next to each group.

---

## ⚠️ Brand-yellow safety flag

The delivery introduces `--infios-color-icon-brand: #D0FF25` (labeled "Brand"). This is **not** the same shade as the existing `--accent: #CADF35` that the moonshot deck has used as its load-bearing brand accent.

- **Existing accent (preserved):** `--accent: #CADF35`, `--wa-nav-accent: #CADF35`
- **New token from delivery (added separately, name does not collide):** `--infios-color-icon-brand: #D0FF25`

Per the `design-tokens` skill safety check: **`--accent` and `--wa-nav-accent` are left unchanged.** The new `--infios-color-icon-brand` token is added under its own name. If the user wants to consolidate the two brand-yellow shades, that's a follow-up decision.

---

## Fonts (`Screenshot 2026-05-12 114140.png`, `114146.png`)

### Family
- `--infios-font-family-default`: `'Noto Sans Variable', sans-serif`
- `--infios-font-family-code`: `'Noto Sans Mono Variable', monospace`

### Weight
- `--infios-font-weight-light`: 300
- `--infios-font-weight-regular`: 400
- `--infios-font-weight-bold`: 700
- `--infios-font-weight-black`: 900

### Size (font-size / line-height pairs)
- `--infios-text-font-size-sm`: 12px / 16px
- `--infios-text-base-font-size`: 14px / 20px
- `--infios-text-font-size-md`: 16px / 24px
- `--infios-text-font-size-lg`: 18px / 24px
- `--infios-text-font-size-xl`: 20px / 24px
- `--infios-text-font-size-2xl`: 24px / 32px
- `--infios-text-font-size-3xl`: 28px / 32px
- `--infios-text-font-size-4xl`: 32px / 40px

### Line height
- `--infios-text-line-height-sm`: 16px
- `--infios-text-base-line-height`: 20px
- `--infios-text-line-height-md`: 24px
- `--infios-text-line-height-lg`: 24px
- `--infios-text-line-height-xl`: 32px
- `--infios-text-line-height-2xl`: 32px
- `--infios-text-line-height-3xl`: 40px
- `--infios-text-line-height-4xl`: 40px

### Letter spacing
- `--infios-letter-spacing-normal`: normal

---

## Spacing (`Screenshot 2026-05-12 114125.png`)

- `--infios-spacing-3xs`: 4px
- `--infios-spacing-2xs`: 8px
- `--infios-spacing-xs`: 12px
- `--infios-spacing-sm`: 16px
- `--infios-spacing-md`: 20px
- `--infios-spacing-lg`: 24px
- `--infios-spacing-xl`: 28px
- `--infios-spacing-2xl`: 32px
- `--infios-spacing-3xl`: 40px
- `--infios-spacing-4xl`: 48px
- `--infios-spacing-5xl`: 64px

> Existing prototype has legacy `--infios-spacing-s` (16px), `-m` (20px), `-l` (24px). Those are still referenced and kept under "Legacy tokens still referenced."

---

## Component heights (`Screenshot 2026-05-12 114119.png`)

- `--infios-height-size-compact`: 32px
- `--infios-height-size-standard`: 40px
- `--infios-height-size-comfortable`: 48px
- `--infios-height-size-comfortable-xl`: 56px

### Clickable sizes
- `--infios-clickable-size-standard`: 24px (desktop)
- `--infios-clickable-size-comfortable`: 48px (mobile)

---

## Colors (`Screenshot 2026-05-12 114001.png`, `114009.png`, `114018.png`, `114028.png`, `114039.png`, `114048.png`, `114056.png`, `114101.png`)

### Background colors
- `--infios-color-bg`: `#FFFFFF` (default page bg)
- `--infios-color-bg-alt`: `#F4F4F4` (high contrast page bg)
- `--infios-color-bg-surface`: `#FFFFFF`
- `--infios-color-bg-surface-secondary`: `#F4F4F4`
- `--infios-color-bg-surface-secondary-hover`: `#E6E6E6`
- `--infios-color-bg-surface-tertiary`: `#F4F4F4`
- `--infios-color-bg-overlay`: `#373737`
- `--infios-color-bg-fill`: `#0060FF`
- `--infios-color-bg-fill-hover`: `#4A8FFF`
- `--infios-color-bg-fill-pressed`: `#00358C`
- `--infios-color-bg-fill-secondary`: `#FFFFFF`
- `--infios-color-bg-fill-secondary-hover`: `#E6E6E6`
- `--infios-color-bg-fill-secondary-pressed`: `#999999`
- `--infios-color-bg-fill-tertiary`: `#F4F4F4`
- `--infios-color-bg-fill-tertiary-hover`: `#F5F5F5`
- `--infios-color-bg-fill-quaternary`: `#E6E6E6`
- `--infios-color-bg-fill-disabled`: `#E6E6E6`
- `--infios-color-bg-fill-disabled-on-bg-fill`: `#373737`
- `--infios-color-bg-fill-brand`: `#232625`
- `--infios-color-bg-fill-brand-hover`: `#6F6F6F`
- `--infios-color-bg-fill-brand-pressed`: `#373737`
- `--infios-color-bg-fill-warning`: `#FFC177`
- `--infios-color-bg-fill-warning-secondary`: `#FFCC73`
- `--infios-color-bg-fill-error`: `#CD3B00`
- `--infios-color-bg-fill-error-hover`: `#BD5757`
- `--infios-color-bg-fill-error-pressed`: `#801919`
- `--infios-color-bg-fill-success`: `#84B98B`
- `--infios-color-bg-fill-special`: `#7B6AA9`
- `--infios-color-bg-fill-info`: `#2D78C1`
- `--infios-color-bg-fill-alt`: `#373737`
- `--infios-color-bg-fill-field`: `#FFFFFF`

### Subtle background (table rows)
- `--infios-color-bg-warning-subtle`: `#FFE8C6`
- `--infios-color-bg-error-subtle`: `#FFC8C8`
- `--infios-color-bg-success-subtle`: `#C7E9C5`
- `--infios-color-bg-info-subtle`: `#C5DAFF`

### Data viz support
- `--infios-color-bg-fill-support-1`: `#0E8385` (green water)
- `--infios-color-bg-fill-support-2`: `#734B6D` (violet)
- `--infios-color-bg-fill-support-3`: `#4E7599` (gray blue)
- `--infios-color-bg-fill-support-4`: `#4421AF` (dark purple)
- `--infios-color-bg-fill-support-5`: `#546B6B` (dark green)
- `--infios-color-bg-fill-support-6`: `#8C564B` (dark brown)
- `--infios-color-bg-fill-support-7`: `#C6263A` (ruby)
- `--infios-color-bg-fill-support-8`: `#C94081` (pink)
- `--infios-color-bg-fill-support-9`: `#946F18` (gold)
- `--infios-color-bg-fill-support-10`: `#AD603A` (light brown)

### Text colors
- `--infios-color-text`: `#232625`
- `--infios-color-text-bg-fill`: `#FFFFFF`
- `--infios-color-text-primary`: `#0060FF`
- `--infios-color-text-primary-hover`: `#4A8FFF`
- `--infios-color-text-secondary`: `#373737`
- `--infios-color-text-tertiary`: `#6F6F6F`
- `--infios-color-text-disabled`: `#999999`
- `--infios-color-text-error`: `#CD3B00`
- `--infios-color-text-error-hover`: `#BD5757`
- `--infios-color-text-error-pressed`: `#801919`
- `--infios-color-text-link`: `#4A8FFF`
- `--infios-color-text-link-hover`: `#4A8FFF`
- `--infios-color-text-link-pressed`: `#00358C`
- `--infios-color-text-success`: `#008049`
- `--infios-color-text-warning`: `#FFC177`
- `--infios-color-text-info`: `#2D78C1`
- `--infios-color-text-on-bg-fill`: `#FFFFFF`
- `--infios-color-text-brand`: `#D0FF25` ⚠️ see brand-yellow flag above

### Border colors
- `--infios-color-border`: `#E6E6E6`
- `--infios-color-border-hover`: `#4A8FFF`
- `--infios-color-border-disabled`: `#E6E6E6`
- `--infios-color-border-tertiary`: `#6F6F6F`
- `--infios-color-border-focus`: `#0060FF`
- `--infios-color-border-error`: `#CD3B00`
- `--infios-color-border-error-hover`: `#BD5757`
- `--infios-color-border-success`: `#84B98B`
- `--infios-color-border-info`: `#2D78C1`
- `--infios-color-border-warning`: `#FFD377`
- `--infios-color-border-pressed`: `#00358C`
- `--infios-color-border-quinary`: `#F5F5F5`
- `--infios-color-border-primary`: `#232625`

### Icon colors
- `--infios-color-icon`: `#232625`
- `--infios-color-icon-hover`: `#4A8FFF`
- `--infios-color-icon-pressed`: `#00358C`
- `--infios-color-icon-primary`: `#0060FF`
- `--infios-color-icon-secondary`: `#373737`
- `--infios-color-icon-tertiary`: `#6F6F6F`
- `--infios-color-icon-on-bg-fill`: `#FFFFFF`
- `--infios-color-icon-disabled`: `#999999`
- `--infios-color-icon-error`: `#CE3800`
- `--infios-color-icon-warning`: `#FFD377`
- `--infios-color-icon-success`: `#84B98B`
- `--infios-color-icon-info`: `#2D78C1`
- `--infios-color-icon-error-hover`: `#BD5757`
- `--infios-color-icon-error-pressed`: `#801919`
- `--infios-color-icon-link-visited`: `#734B6D`
- `--infios-color-icon-link-disabled`: `#999999`
- `--infios-color-icon-hover-secondary`: `#E6E6E6`
- `--infios-color-icon-brand`: `#D0FF25` ⚠️ see brand-yellow flag above

---

## Shadows / elevation (`Screenshot 2026-05-12 114109.png`)

- `--infios-shadow-header`: `0 1px 2px rgba(35, 38, 37, 0.1)`
- `--infios-shadow-sidebar-left`: `1px 0 2px rgba(35, 38, 37, 0.1)`
- `--infios-shadow-sidebar-right`: `-1px 0 2px rgba(35, 38, 37, 0.1)`
- `--infios-shadow-accordion-nested-edge`: `0 0 1px rgba(0, 96, 255, 1)`
- `--infios-shadow-container`: `0px 2px 8px 0px rgba(35, 38, 37, 0.15)`

---

## Border radius / width / offset (`Screenshot 2026-05-12 114152.png`)

### Radius
- `--infios-border-radius-sm`: 2px
- `--infios-border-radius-md`: 4px
- `--infios-border-radius-lg`: 8px
- `--infios-border-radius-xl`: 16px
- `--infios-border-radius-pill`: 9999px

### Width
- `--infios-border-width-sm`: 1px
- `--infios-border-width-md`: 2px
- `--infios-shape-border-divider`: 1px

### Offset
- `--infios-border-offset`: 2px

---

## Transitions / easing (`Screenshot 2026-05-12 114130.png`)

- `--infios-transition-x-slow`: 1000ms
- `--infios-transition-slow`: 500ms
- `--infios-transition-medium`: 250ms
- `--infios-transition-fast`: 150ms
- `--infios-transition-x-fast`: 50ms

The delivery did **not** include named easing curves (cubic-bezier). The existing `--ease-out` and `--ease-spring` in the presentation are preserved as legacy.

---

## Legacy tokens still referenced

Tokens that exist in current files, are not in the new delivery, but are still referenced in CSS/JS bodies. Per skill rule "never rename existing tokens," these are preserved verbatim in both `:root` blocks.

From presentation:
- `--surface-base`, `--surface-canvas`, `--surface-tinted`, `--surface-elevated`
- `--border-hairline`, `--border-subtle`, `--border-strong`
- `--text-primary`, `--text-secondary`, `--text-tertiary`, `--text-quaternary`
- `--brand-fill`, `--accent`, `--accent-deep`, `--accent-darker`
- `--link`, `--link-soft`, `--warn`, `--warn-soft`, `--info`, `--info-soft`, `--success`, `--error`
- `--bg-success`, `--bg-warning`, `--bg-info`, `--bg-error`, `--bg-accent`
- `--shadow-xs`, `--shadow-sm`, `--shadow-md`, `--shadow-lg`, `--shadow-xl`
- `--radius-sm`, `--radius-md`, `--radius-lg`, `--radius-xl`, `--radius-pill`
- `--font-display`, `--font-text`, `--font-mono`
- `--ease-out`, `--ease-spring`

From prototype:
- `--wa-header-bg`, `--wa-nav-bg`, `--wa-nav-accent`, `--wa-nav-hover`, `--wa-nav-text`
- `--infios-font-family` (legacy alias for `--infios-font-family-default`)
- `--infios-spacing-s` (16px), `--infios-spacing-m` (20px), `--infios-spacing-l` (24px)
- `--infios-radius-desktop` (8px), `--infios-radius-pill` (9999px)
- `--infios-color-bg-fill-overlay`, `--infios-color-bg-fill-success-subtle`, `--infios-color-bg-fill-warning-subtle`, `--infios-color-bg-fill-error-subtle`, `--infios-color-bg-fill-info-subtle`
- `--infios-color-text-warning`, `--infios-color-text-success`, `--infios-color-text-error`, `--infios-color-text-info` (preserved at existing values which differ slightly from the spec's text-color group — kept to avoid breaking existing references)
- `--infios-shadow-card`, `--infios-shadow-modal`, `--infios-shadow-tile`

---

## Intentional off-grid values

The following hardcoded values appear in the codebase but don't map cleanly to any delivered token and are left as-is by intent:

- `rgba(255,255,255,.72)` and similar translucent whites for glass/overlay effects in the deck.
- Magic numbers in chart/bar SVG coordinates (`viewBox`, path coordinates).
- Step-by-step animation timing values that don't match the transition scale.
- `rgba(202,223,53,0.X)` accent-yellow tint variants in the deck — these depend on `--accent` (`#CADF35`) and stay tied to the legacy accent shade by design.

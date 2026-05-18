---
name: design-tokens
description: Use this skill when the user delivers a design-token spec (zip of screenshots, Figma export, JSON file, or similar) OR asks to integrate/reconcile/sweep design tokens in this moonshot deck. The skill handles extracting the delivery, transcribing tokens into a versioned reference markdown, reconciling new tokens with the existing :root custom-property blocks in moonshot presentation/moonshot.html and moonshot prototype/moonshot-prototype.html, and sweeping both files for hardcoded values (hex colors, rgba stacks, raw box-shadows, hardcoded radii, magic-number spacing) that should reference tokens. Auto-trigger keywords include "design tokens", "token zip", "integrate tokens", "tokens.zip", "reconcile tokens", "sweep for hardcoded", "Figma tokens", and any path matching tokens-*.zip or *tokens*.md inside this project. The brand-yellow accent #CADF35 is load-bearing — if a new token spec proposes changing it, the skill stops and flags the discrepancy to the user instead of overwriting.
---

# design-tokens workflow

This skill runs every time a new design-token spec lands in the moonshot deck repo. The same loop repeats: extract → transcribe → reconcile → mirror → sweep → verify, gated by a brand-yellow safety check.

The deck consists of two single-file vanilla-HTML artifacts:

- `moonshot presentation/moonshot.html` — polished, ship-ready deck. `:root` block at lines ~11–72.
- `moonshot prototype/moonshot-prototype.html` — live demo loaded inside an iframe on the "The Prototype" tab. `:root` block at lines ~10–38.

No build step, no external CSS. Token names are CSS custom properties under `:root { ... }` at the top of each file.

## Naming-scheme reality (read before reconciling)

The two `:root` blocks today use **different naming conventions**:

- Presentation: short names — `--surface-*`, `--text-*`, `--accent-*`, `--shadow-*`, `--radius-*`, `--font-*`, `--ease-*`.
- Prototype: Infios-prefixed namespace — `--infios-color-*`, `--infios-spacing-*`, `--infios-shadow-*`, `--infios-radius-*`, plus warehouse-specific `--wa-*`.

The prototype's `--infios-*` and `--wa-*` names are referenced throughout its 4,000+ lines. Renaming them cascades and breaks the file. The constraint is firm: **never rename existing custom properties**. When mirroring, the reconciled block is therefore a **union** — both naming schemes coexist, with new tokens from the delivery added once per file. Both files end up with the same superset `:root`, which satisfies byte-identical mirroring without breaking any reference.

## The 8-step workflow

### 1. Locate the delivery

Default location: `C:\Users\apcan\Pictures\Screenshots\tokens.zip`.

If the user supplies a different path or hands over a folder / JSON / Figma export directly, use that instead. If the delivery is already an unzipped folder, skip step 2.

### 2. Extract

Extract to `archive/tokens-<YYYY-MM-DD>/` inside the repo, where `<YYYY-MM-DD>` is today's date.

PowerShell:
```
Expand-Archive -Path '<zip>' -DestinationPath 'archive/tokens-<YYYY-MM-DD>' -Force
```

Or Bash via `tar -xf <zip> -C archive/tokens-<YYYY-MM-DD>`.

### 3. Transcribe

Transcribe every PNG / image / JSON / file in the extracted folder into a markdown reference at `archive/tokens-<YYYY-MM-DD>/TOKENS.md`.

Capture every:
- Font family + weight + size scale.
- Spacing value.
- Named color + its hex.
- Named shadow + its full `box-shadow` value.
- Radius / easing / border token shown.

Organize by category: **Fonts / Spacing / Colors / Shadows / Radii / Easing / Borders**. Note the source filename next to each transcribed group so the lineage is traceable.

For PNG deliveries, use the Read tool on each PNG file — it accepts images visually.

### 4. Reconcile with the presentation `:root`

Open `moonshot presentation/moonshot.html` and locate the `:root { ... }` block near the top.

For each existing custom property:

- **Direct equivalent in new spec** → update the value to match the spec.
- **New token with no equivalent** → add it, naming consistently with the existing scheme (`--surface-*`, `--text-*`, `--accent-*`, `--shadow-*`, `--radius-*`, `--font-*`, `--ease-*`).
- **Existing token dropped by new spec but still referenced elsewhere in the file** → keep it and document under a "Legacy tokens still referenced" section in TOKENS.md.
- **Never rename** existing custom properties — renaming cascades.

### 5. Mirror into the prototype

Open `moonshot prototype/moonshot-prototype.html` and write the same union of tokens into its `:root` block, so the two `:root` blocks are byte-identical after this step.

Because the prototype has its own `--infios-*` and `--wa-*` token names referenced throughout, **the reconciled `:root` is the union of (presentation existing tokens, prototype existing tokens, new delivery tokens)**. Both schemes coexist. Both files end up with the identical superset. No existing reference breaks.

### 6. Sweep for hardcoded values

Across both HTML files, find and replace hardcoded values that should now reference tokens:

- Hex colors `#xxxxxx` matching a token value.
- `rgba(...)` calls matching a token value.
- Raw `box-shadow` declarations using rgba stacks that match a `--shadow-*` token.
- Hardcoded `font-family` strings matching a `--font-*` token.
- Fixed pixel radii matching a `--radius-*` token.
- Spacing values (`padding`, `margin`, `gap`) matching `--space-*` / `--infios-spacing-*` if the delivery introduces a spacing scale.

Replace each clean match with `var(--token-name)`.

**Don't force a token where the original value was intentional and off-grid** — flag those in TOKENS.md under "Intentional off-grid values" instead.

### 7. Verify

Use the `preview_*` MCP tools (never Bash or browser MCP for this):

1. `preview_start` on `moonshot presentation/moonshot.html`.
2. `preview_screenshot` the Home view, the Agents view (both Timeline and System modes via the Architecture toggle), and the Prototype tab (which loads the prototype HTML inside an iframe).
3. The visuals must be identical to before — same colors, shadows, spacing — just driven by tokens instead of hardcoded values.
4. `preview_console_logs` clean.
5. Both Guardian audit verdicts (Heuristics, Accessibility) still SHIP.

### 8. Brand-yellow safety check

`--accent: #CADF35` is load-bearing for Infios visual identity.

If any token in the delivery proposes a value different from `#CADF35` for the brand-yellow accent, **stop and flag the discrepancy to the user** before changing the accent. Never silently overwrite.

## Constraints that always apply

- Single-file architecture stays. No external CSS files. No build step.
- Don't add tokens that aren't in the delivery. The delivery is authoritative.
- Don't break existing class names. Token names can evolve; class names can't (other CSS rules and JS reference them).
- Don't silently overwrite brand-yellow. If the delivery disagrees, stop and ask.

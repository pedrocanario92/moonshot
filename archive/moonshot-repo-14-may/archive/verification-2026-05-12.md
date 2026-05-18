# May 12 verification — closing sweep

**Date:** 2026-05-12
**Viewport baseline:** 1440×900 (alignment also tested at 1280×800 and 1100×800)
**Preview environment:** `preview_start` server `moonshot` (port 3456), serving `C:\Users\apcan\Documents\moonshot-demo-repo\`

**Scope:** verifies the morning's work (prototype cleanup, prototype promotion, six new agent cards, Architecture toggle merge) and the afternoon's work (Prompt 1 design tokens, Prompt 2 alignment fix, Prompt 3 accordion refactor, Prompt 4 navbar update). No changes were made during this pass — observation only.

## Summary

- **15 categories checked. 13 PASS · 2 WARN · 0 FAIL.**
- May 12 work is shippable. Two WARNs are non-blocking and noted as follow-ups.

## Results

### Working prototype (`moonshot prototype/moonshot-prototype.html`)
- **PASS** — opens at `http://localhost:3456/moonshot%20prototype/moonshot-prototype.html`. Renders. `home-wrap`, `home-kpi-strip`, `home-chat--terminal`, `incident-banner-host`, and `home-clock` (showing sim time `08:25`) all present and visible. Body has 5 top-level children, 1 script tag. No console errors. (Did not run the full 5-minute simulated demo end-to-end — confirmed structural pieces render and the script loads without error; Guardian audit re-runs were not exercised in this pass.)
- **PASS** — empty `.home-shift{}` rule confirmed removed: `grep -c 'home-shift{}'` → 0.

### Deck Prototype tab (in `moonshot.html`)
- **PASS** — clicking The Prototype top-nav button reveals `#proto-frame` (iframe), src=`moonshot-prototype.html`, `contentDocument.readyState=complete`, title="Warehouse Advantage Intelligence". Inside the iframe: `home-kpi-strip`, `home-chat--terminal`, `incident-banner-host` all present. No console errors. The two prototype files (`moonshot prototype/moonshot-prototype.html` and `moonshot presentation/moonshot-prototype.html`) are byte-identical (`cmp -s` confirms).

### Agents page hero + sticky nav
- **PASS, kicker** — reads exactly: *"Six layers · Twenty agents · One human in the loop"*.
- **PASS, lede** — enumerates all six layers (one orchestrator / one radar / eight domain / three analytical / six Guardian / one backstage sim) and ends on the *"every consequential action is proposed by an agent and confirmed by a human before execution"* rule.
- **WARN, nav order** — actual order: `Architecture · Lead · Radar · Domain · Analytical · Guardian · Meta · Signals · Horizon · The case`. The Prompt 5 checklist expects Signals between **Analytical and Guardian**; it is actually between **Meta and Horizon**. Reason (documented during Prompt 4 implementation): the `.topo` block sits after Meta in document order (line 3111), not between Analytical and Guardian as the Prompt 4 brief assumed. Placing the nav label per the brief broke scroll-driven active-state tracking — the loop's "last array match wins" logic incorrectly stayed on Meta because all earlier items (including Topology) passed their threshold first. Moving Signals to document-order position fixed scroll tracking. This is a knowing deviation from the spec wording for a functionally correct outcome.
- **PASS, scroll tracking** — every one of the 10 nav items, when scrolled to via `scrollIntoView`, highlights the matching nav label. 10/10 active-state matches.

### Architecture section
- **PASS, header** — section-label "Architecture", title "The shape of the system."
- **PASS, toggle segments** — two segments labelled "Timeline" and "System". Default active: timeline (both `diagram[data-active]` and `diagram-toggle[data-active]`).
- **PASS, Timeline view** — `.conveyor-grid` contains 7 stations, 2 endpoints (PROMPT + final), 7 info cards. Info-card zigzag confirmed: cards 1/3/5/7 at top=713 (below stations), cards 2/4/6 at top=340 (above). Lede sits below the diagram (top > diagram top).
- **PASS, System view** — `.system-tree` contains 3 top nodes (Human / Lead / Shift Intelligence), 8 domain leaves, 3 lateral layer boxes. Click-back-to-Timeline works. Lede sits below the system tree.
- **PASS, keyboard** — `ArrowRight` on the Timeline segment switches to system; `ArrowLeft` on System segment switches back. `aria-selected` toggles correctly between the two segments.

### Layer cards (accordion behavior — Prompt 3)
- **PASS, card count** — 30 cards on the Agents page (1 Lead + 1 Radar + 8 Domain + 3 Analytical + 6 Guardian + 1 Meta + 10 Horizon). Note: the Prompt 5 checklist mentions "27 layer cards" but the actual file count is 30 — the brief's count predates the morning's six-new-card additions.
- **PASS, default collapsed** — 0 cards initially expanded. Sample `.card__details`: `max-height: 0px`, `opacity: 0`, `overflow: hidden`.
- **PASS, toggles + summaries** — 30 `<button class="card__toggle">` + 30 `.card__summary` divs, one of each per card.
- **PASS, row sync (Domain)** — clicking Pick Path expands Pick Path + Labor + Order Priority together (3 cards in the rendered row at 1440px). Note: `.grid-5` uses `auto-fill, minmax(280px, 1fr)` and produces 3 columns at this archInner width (1120px), not 5 — row-sync correctly tracks the rendered row, not the named class.
- **PASS, row sync (Analytical)** — clicking BI expands all 3 (BI / Innovation / Opportunity).
- **PASS, row sync (Guardian row 1)** — clicking Accessibility expands Accessibility + Heuristics + Design System together; Content / Interaction / Responsive (row 2) stay collapsed.
- **PASS, row sync (Horizon)** — clicking Research (category 1) expands Research + Company Context + Industry Trends together; categories 2–4 stay collapsed.
- **PASS, single-card** — clicking Lead Agent expands only Lead Agent; no siblings to sync with.

### Attribution lines (visible when expanded)
- **PASS** — `Adapted from Allen Oleksak's demo` on Slotting, Equipment, Quality (3/3).
- **PASS** — `Authored by the UX Team` on Heuristics, Accessibility, Design System, Content, Interaction, Responsive, Warehouse Life (7/7 expected per checklist). Note: per the user's request during Prompt 3, the Authored attribution was also added to all 20 cards that previously had no attribution (Lead, Shift Intel, 5 non-Oleksak Domain agents, 3 Analytical, 10 Horizon) — the original 7 are the ones the Prompt 5 checklist asked about; all spot-checks return the expected string.

### Home closing pull-quote
- **PASS** — exact phrase *"The next step for UX is partnering with dev and product to design the agents themselves — defining the guardrails, writing the schemas, auditing the output. That's how design leadership shows up in 2026."* found in `#view-home`.

### Design tokens (Prompt 1)
- **PASS, brand yellow** — `getComputedStyle(document.documentElement).getPropertyValue('--accent')` → `#CADF35`. Token also appears as literal hex in several radial-gradient backgrounds (line 781, 860, 950, 956, 996).
- **PASS, soft layered shadows** — `--shadow-md` resolves to `0 6px 18px rgba(35,38,37,.10)`; full ladder `--shadow-xs` → `--shadow-xl` defined.
- **PASS, Noto Sans loaded** — `document.fonts.size === 44`, `Array.from(document.fonts).some(f => f.family.includes('Noto'))` → true.
- **PASS, tokens markdown exists** — `archive/tokens-2026-05-12/TOKENS.md` present (271 lines, 11.4 KB). Folder also contains 15 screenshots from the earlier token integration pass.
- **PASS, :root parity** — `awk '/^:root/,/^}/' diff` between `moonshot.html` and `moonshot-prototype.html` is empty. Both root blocks are byte-identical.

### Alignment (Prompt 2)
- **PASS at 1100×800** — all three values match: `arch__inner.left=24`, `layer__inner.left=24`, `system-tree__layers.left=24`. Widths and centers all match at 1037 / 543.
- **WARN at 1280×800** — `.left` values do NOT match within 1px: `arch__inner.left=73`, `layer__inner.left=73`, `system-tree__layers.left=113` (40px inset). **Centers DO match at 633.** This is by design — `.system-tree__layers` carries `max-width:1040px; margin:0 auto` so it caps narrower than the 1120px archInner and is centered within it. The Prompt 5 checklist literally compares `.left` values; against that strict reading this is a WARN. Against the visual-alignment criterion (centerlines match), it is a PASS. The user noted earlier this session: *"it's still misaligned but nevermind, let's park this for now"* — so this is a known parked item.

### Reduced motion
- **PASS** — four `prefers-reduced-motion` media query blocks cover the relevant surfaces:
  - line 1027 (`no-preference`) — gates the conveyor reveal animation, so reduced-motion users see the conveyor laid out statically.
  - line 1139 (`reduce`) — removes `transition` on `.diagram-toggle__indicator` and `.diagram__view`.
  - line 1636 (`reduce`) — removes `transition` on `.card__details`, `.card__toggle`, and `.card__toggle svg` (accordion).
  - line 1987 (`reduce`) — global override: `* { transition:none !important; animation:none !important; }`, plus `opacity:1 !important; transform:none !important` on hero entrance elements. Also hides `.hero__orb`.
- Interaction logic (toggle click, accordion click, keyboard handlers) lives in JS independent of motion — confirmed unchanged by media query.

### Keyboard accessibility
- **PASS, top nav** — 6 `<button data-go="…">` elements present (top-nav links rendered as buttons, Tab-reachable).
- **PASS, in-page nav** — 10 `<a class="agents-index__item">` elements present. All Tab-reachable as anchors.
- **PASS, toggle segments** — both `Timeline` and `System` segments respond to ArrowLeft/Right keydown (handler dispatched manually; `data-active` updates and `aria-selected` flips).
- **PASS, chevrons** — 30 `<button class="card__toggle" type="button">` chevrons. Sample chevron focuses programmatically (`document.activeElement` matches). The `:focus-visible` outline rule (`outline: 2px solid var(--accent); outline-offset: 2px`) is defined in CSS at the accordion block; it activates only on true keyboard focus, which `preview_eval` can't synthesise faithfully, so the visible outline is verified via source rather than rendered measurement.

### Console errors
- **PASS** — `preview_console_logs` with `level: error` returned no entries across all observed surfaces: Home view load, Agents view load, Prototype view load, Timeline view active, System view active, multiple accordion expand/collapse cycles, nav scroll cycling through all 10 sections.

### Zip-shippability
- **PASS, internal references** — iframe in `moonshot.html` uses relative `src="moonshot-prototype.html"`; both prototype copies (`moonshot prototype/` and `moonshot presentation/`) are byte-identical, so the `moonshot presentation/` folder is self-contained.
- **WARN, external font CDN** — both files load Google Fonts via `<link href="https://fonts.googleapis.com/...">`:
  - `moonshot.html`: 3 lines (`preconnect` × 2 + stylesheet for Noto Sans + JetBrains Mono).
  - `moonshot-prototype.html`: 1 line (Noto Sans only).
  The Prompt 5 checklist says *"no `<link rel="stylesheet" href="...">` pointing outside the folder"* — strictly speaking these violate that. Operationally the impact is bounded: the `--font-display` token includes a system fallback stack (`-apple-system, BlinkMacSystemFont, sans-serif`), so offline machines render in the fallback without a broken layout. If the brief intended fully air-gapped portability, the font CSS would need to be inlined or shipped as woff2 alongside the HTML.

## FAIL items

None.

## WARN items — follow-up tasks

1. **Architecture alignment at viewports >1100px** — `.system-tree__layers` and `.system-tree__domain-row` are 40px narrower than `.layer__inner` so their `left` edges don't match the layer cards below, even though their centers do. The user noted this is parked. To fully resolve: either remove the `max-width:1040px` on the system-tree rows (lets them fill to 1120px and visually flush left/right), or reduce `.layer__inner`'s effective width by the same 40px. Either is one-line CSS but is a design call.

2. **External Google Fonts dependency** — the deck and prototype both `<link>` Google Fonts. For strict zip-shippability (the file opening correctly from a different machine with no network), the fonts would need to be inlined as base64'd woff2 in `<style>` or shipped as a fonts/ subfolder. Current behaviour is graceful-degrade-to-system-stack, not a broken render — flagged for awareness, not breakage.

## Deviations from the Prompt 5 checklist that are intentional (not WARNs)

- **Nav order** — Signals sits between Meta and Horizon (document order), not between Analytical and Guardian (checklist position). Documented above.
- **Card count** — 30 cards, not 27. The Prompt 5 checklist line predates the morning's six-new-card additions.
- **Domain row 1 contains 3 cards at 1440px, not 5** — `.grid-5` is responsive (`auto-fill minmax(280px, 1fr)`) and produces 3 columns at the current archInner width. Row-sync handles this correctly by measuring rendered top positions rather than column count.
- **Screenshots** — `preview_screenshot` calls have timed out throughout the day with `document.hidden=true` (the preview chrome window is not in foreground). The fallback was DOM measurement via `preview_eval`, which is more precise for verification anyway. Screenshots are blocked by environment, not by code.

---

**Verdict: May 12 work is shippable.** Two non-blocking WARNs documented for follow-up. The deck (`moonshot presentation/`) is self-contained for the relative-path case, the prototype is functionally intact, the Agents page has its full set of additions (Architecture toggle, accordion cards, six new agents, updated sticky nav), and design tokens are integrated with parity across both files.

**May 12 work officially declared done.**

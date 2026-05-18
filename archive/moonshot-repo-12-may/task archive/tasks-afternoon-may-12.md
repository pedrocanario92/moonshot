# Afternoon of May 12 2026 — task prompts

Six prompts, in execution order. The first four are afternoon build work; Prompts 5 and 6 pick up leftover tasks from `MAY-12-PLAN.md` (the full verification pass and the one-shot prompt experiment). Each prompt is self-contained — copy the relevant prompt body into a fresh chat with no prior context and it should be enough to act on.

**Execution order rationale**

1. **Design tokens skill first.** Tokens are foundational. Every downstream task uses them, so they should land before the alignment fix, the accordion refactor, or the nav update — otherwise we'd re-touch the same CSS twice. This prompt installs an auto-triggered skill, then runs it for the May 12 token zip.
2. **Architecture-section alignment fix second.** Small surgical CSS adjustment. Best done after tokens are in place so the fix can reference the new spacing/radius/shadow tokens directly rather than raw values.
3. **Agent tile accordion refactor third.** Largest change. Touches every agent card on the Agents page. Better done with tokens in place so all the new accordion CSS uses tokens, not magic numbers. Also better done after alignment is correct, so accordion behavior isn't masking pre-existing layout drift.
4. **Sticky-navbar update fourth.** Small structural fix. Done after the accordion work so the page structure is stable before the nav is updated to match it.
5. **Full verification pass fifth.** Closing sweep across both the morning's work (cleanup, swap, agent cards, toggle-merge) and the afternoon's work (tokens, alignment, accordion, navbar). Catches anything that drifted; produces a PASS / WARN / FAIL report.
6. **One-shot agent-driven prototype build last — in a separate fresh chat.** MVP of "do the agent specs actually drive good interfaces?" Must run in an uncontaminated session; cannot be done alongside Prompts 1–5.

Each prompt is written assuming a fresh Claude Code session opened in `C:\Users\apcan\Documents\moonshot-demo-repo\` (except Prompt 6, which has its own context note).

---

## Prompt 1 — Install the design-tokens skill, then run it for the May 12 token delivery

You are working in a single-page presentation deck at `moonshot presentation/moonshot.html` plus its embedded working prototype at `moonshot prototype/moonshot-prototype.html`. The deck is the polished, ship-ready artifact; the prototype is the live demo loaded inside an iframe on the deck's "The Prototype" tab. Both files are vanilla HTML with inline CSS and inline JS — no build step, no framework, no external assets. The deck uses Infios brand language with brand-yellow `#CADF35` as the accent.

There is currently a token system at the top of each file under `:root { ... }` that defines CSS custom properties for surfaces, borders, text colors, brand colors, status colors, accent backgrounds, shadows, radii, fonts, and easing curves. Look at lines ~11–72 of `moonshot presentation/moonshot.html` for the existing setup. The naming pattern is `--surface-*`, `--border-*`, `--text-*`, `--accent-*`, `--shadow-*`, `--radius-*`, `--font-*`, `--ease-*`.

A canonical token spec has been delivered as a zip at `C:\Users\apcan\Pictures\Screenshots\tokens.zip`. The zip contains 15 PNG screenshots covering font tokens, spacing tokens, color tokens, and shadow tokens. The zip is the source of truth for the design tokens going forward.

The shape of this work is procedural and **repeating** — every time a new token spec lands, the same dance happens (extract → transcribe → reconcile → sweep → verify). So we capture it as an auto-triggered Claude skill once, and reuse it from there.

**Your task — two steps:**

### Step 1: Create the design-tokens skill

Write a new file at **`.claude/skills/design-tokens.md`** at the repo root. This makes the skill project-scoped — it only fires inside this project, not globally. The skill has YAML frontmatter that defines an auto-trigger description, plus a body that holds the procedural workflow.

**Frontmatter** (use this verbatim — the description is what triggers auto-loading in future sessions, and the exact phrasing matters):

```yaml
---
name: design-tokens
description: Use this skill when the user delivers a design-token spec (zip of screenshots, Figma export, JSON file, or similar) OR asks to integrate/reconcile/sweep design tokens in this moonshot deck. The skill handles extracting the delivery, transcribing tokens into a versioned reference markdown, reconciling new tokens with the existing :root custom-property blocks in moonshot presentation/moonshot.html and moonshot prototype/moonshot-prototype.html, and sweeping both files for hardcoded values (hex colors, rgba stacks, raw box-shadows, hardcoded radii, magic-number spacing) that should reference tokens. Auto-trigger keywords include "design tokens", "token zip", "integrate tokens", "tokens.zip", "reconcile tokens", "sweep for hardcoded", "Figma tokens", and any path matching tokens-*.zip or *tokens*.md inside this project. The brand-yellow accent #CADF35 is load-bearing — if a new token spec proposes changing it, the skill stops and flags the discrepancy to the user instead of overwriting.
---
```

**Body of the skill** (the procedural workflow — write this as the skill's own instructions to itself):

1. **Locate the delivery.** Default location: `C:\Users\apcan\Pictures\Screenshots\tokens.zip`. If the user supplies a different path or hands over a folder/JSON/Figma export directly, use that instead. If the delivery is already an unzipped folder, skip step 2.
2. **Extract** the zip to `archive/tokens-<YYYY-MM-DD>/` inside the repo, where `<YYYY-MM-DD>` is today's date. Use PowerShell `Expand-Archive` or `tar -xf` via Bash.
3. **Transcribe** every PNG / image / JSON / file in the extracted folder into a markdown reference at `archive/tokens-<YYYY-MM-DD>/TOKENS.md`. Capture: every font family + weight + size scale, every spacing value, every named color + its hex, every named shadow + its full `box-shadow` value, and any radius/easing/border tokens shown. Organize by category (Fonts / Spacing / Colors / Shadows / Radii / Easing / Borders). Note the source filename next to each transcribed group so the lineage is traceable.
4. **Reconcile** the new tokens with the existing `:root` block in `moonshot presentation/moonshot.html` (search for `:root {` near the top of the file). For each existing custom property: if the new spec has a direct equivalent, update the value; if the new spec introduces tokens that don't exist yet, add them with consistent naming aligned to the existing scheme (`--surface-*`, `--text-*`, `--accent-*`, `--shadow-*`, `--radius-*`, `--font-*`, `--ease-*`); if the new spec drops a token that's still in use elsewhere in the file, **keep** the legacy one and flag it in TOKENS.md under a "Legacy tokens still referenced" section. **Do not rename** existing custom properties — renaming triggers a cascade of edits elsewhere in the codebase.
5. **Mirror** the reconciled `:root` block into `moonshot prototype/moonshot-prototype.html`. After this step, the two `:root` blocks should be byte-identical.
6. **Sweep** both files for hardcoded values that should now reference tokens. Specifically: hex colors (`#xxxxxx`), `rgba()` calls that match a token value, raw `box-shadow` declarations using rgba stacks, hardcoded `font-family` strings, fixed pixel radii that match a `--radius-*` token, and spacing values (`padding`, `margin`, `gap`) that should use a `--space-*` token if the new spec introduces one. Replace each clean match with `var(--token-name)`. Don't force a token where the original value was intentional and off-grid — flag those in TOKENS.md under "Intentional off-grid values" instead.
7. **Verify** by spot-checking via `preview_start` + `preview_screenshot`:
   - The Home view, the Agents view (including the Architecture toggle in both Timeline and System modes), and the Prototype tab should render visually identical to before — same colors, shadows, spacing — just driven by tokens instead of hardcoded values.
   - The prototype loaded inside the iframe (`moonshot presentation/moonshot-prototype.html`) should also render identically.
   - No console errors. Both Guardian audit verdicts (Heuristics, Accessibility) still SHIP.
8. **Brand-yellow safety check.** If any new token in the delivery proposes a value different from `--accent: #CADF35`, **stop and flag the discrepancy to the user** before changing the accent. Brand-yellow is load-bearing for the visual identity; it must not be silently overwritten.

### Step 2: Run the skill for the May 12 token delivery

After the skill file is created, execute the workflow against `C:\Users\apcan\Pictures\Screenshots\tokens.zip`. Follow the skill body's steps in order. The output:

- Extracted token reference at `archive/tokens-2026-05-12/TOKENS.md`.
- Updated `:root` block in both `moonshot presentation/moonshot.html` and `moonshot prototype/moonshot-prototype.html` — byte-identical between the two files.
- Hardcoded values across both files swept and replaced with `var(--token-name)` where matches were clean.
- Verification screenshots showing Home / Agents / Prototype render unchanged.
- Summary of what changed at the end of your final response.

**Constraints (apply to both Step 1 and Step 2):**

- Single-file architecture stays. No external CSS files, no build step.
- Don't add tokens that aren't in the delivery. The delivery is authoritative.
- Don't break existing class names. Token names can evolve; class names can't (other CSS rules and JS reference them).
- Don't silently overwrite brand-yellow. If the delivery disagrees, stop and ask.

**Future sessions:** with the skill installed, any future token delivery — or any user prompt mentioning the trigger keywords — will auto-load this same workflow without needing this brief re-pasted.

---

## Prompt 2 — Fix the horizontal alignment of the Architecture section on The Agents page

You are working in `moonshot presentation/moonshot.html`. The Agents page (`<section class="view view--agents" id="view-agents">`) has an Architecture section near the top that contains a Timeline · System toggle (Apple-style segmented control) and two diagrams that swap based on the toggle: a horizontal "Timeline" conveyor (CSS Grid, 9 columns) and a "System" hierarchy tree.

**The bug:**

The Architecture section's content (especially the System tree's three lateral layer boxes for Analytical / Guardian / Meta) appears horizontally skewed to the **left** relative to the toggle above it and the layer cards below it (Orchestration / Radar / Domain / Analytical / Guardian / Meta / Horizon). The skew is more evident at narrower viewport widths (e.g., 1280px, 1100px) than at 1440px+. At ~1280px the System tree's lateral layer boxes hug the left edge of the viewport while the toggle remains centered on the page; at 1100px the misalignment is even more visible.

**Where to look in the CSS:**

Search the file for these selectors and read them first:

- `.arch` — the outer architecture section
- `.arch__inner` — the constrained inner container (`max-width:1120px; margin:0 auto`)
- `.layer` — the outer wrapper for each agent-layer section below
- `.layer__inner` — the constrained inner container (should match `.arch__inner` exactly)
- `.diagram-toggle-row` — the row that centers the segmented control
- `.diagram-toggle` — the toggle itself
- `.diagram` — the CSS Grid wrapper that stacks the two views via `grid-template-areas: "stack"`
- `.diagram__view`, `.diagram__view--timeline`, `.diagram__view--system` — the two views
- `.howagents-conveyor`, `.howagents-conveyor__inner` — the legacy wrapper inside each `.diagram__view` (yes, the class name is now misleading; rename is out of scope here)
- `.conveyor-grid` — the timeline view's grid layout
- `.system-tree` — the system view's outermost container
- `.system-tree__domain-row` — the row holding the 8 domain agent leaves (has `max-width: 1040px`)
- `.system-tree__layers` — the row holding the three lateral layer boxes (also has `max-width: 1040px`)

**Likely root causes (verify before fixing):**

- `.system-tree__domain-row` and `.system-tree__layers` both set `width: 100%` plus `max-width: 1040px` but **may not be horizontally centered**. They might lack `margin: 0 auto` — so when the parent flex/grid container is wider than 1040px, they stick to the left edge instead of centering.
- The `.system-tree` itself uses `display: flex; flex-direction: column; align-items: center` which should center its direct children. Confirm the domain-row and layers div are direct children of `.system-tree` (not nested in another wrapper that loses the centering).
- Compare against the `.conveyor-grid` (timeline view), which uses an explicit `grid-template-columns: 96px repeat(7, 1fr) 140px` and naturally fills the parent — so the alignment looks correct for that view. The bug is specific to the system view.

**Your task:**

1. **Verify the bug visually first.** Start the preview server (`preview_start` against the moonshot presentation folder), navigate to The Agents page, click the System segment, and capture a screenshot at viewport widths 1280×800 and 1100×800. Note where the lateral layer boxes sit relative to the toggle above and the first layer card ("Lead Agent" / Orchestration) below.
2. **Use DOM measurements to confirm the misalignment.** Use `preview_eval` to log `getBoundingClientRect().left` for the toggle, the `.system-tree__layers`, and the `.layer__inner` of the Orchestration section. They should all sit at the same left edge if alignment is correct.
3. **Fix the CSS.** The most likely fix is adding `margin: 0 auto` (or `margin-left: auto; margin-right: auto`) to `.system-tree__domain-row` and `.system-tree__layers` so they horizontally center within `.system-tree`. If that doesn't fully resolve it, check whether `.system-tree` itself needs a width constraint, or whether `.howagents-conveyor__inner` (the wrapper around the tree) is doing something unexpected.
4. **Verify again at both widths.** Take fresh screenshots at 1280×800 and 1100×800. The DOM measurements from step 2 should now show matching `left` values across toggle, system tree, and Orchestration layer card.
5. **Also confirm the Timeline view didn't regress.** Click back to Timeline, check the prompt card, the 7 stations, the info cards, and the centered lede all still render correctly.

**Constraints:**

- Don't change anything outside the `.system-tree*` family of selectors unless the root cause is provably elsewhere. The Timeline view is currently correctly aligned and shouldn't be touched.
- Don't introduce a new wrapper element. The fix should be one or two CSS property additions, not a markup restructure.
- Reduced-motion users must still see the system view render correctly (toggle motion is wrapped in `@media (prefers-reduced-motion: no-preference)` — your fix shouldn't introduce motion).

**Deliverables:**

- CSS edits to align the System view horizontally with the rest of the Architecture section.
- Before/after screenshots at both 1280×800 and 1100×800 showing the fix.
- DOM measurement comparison (left edges of toggle, system tree, Orchestration layer card) confirming alignment.

---

## Prompt 3 — Convert agent cards into row-synchronised accordions

You are working in `moonshot presentation/moonshot.html` on The Agents page (`<section class="view view--agents" id="view-agents">`). The page has multiple layer sections — Orchestration (1 card, full-width), Radar (1 card, full-width), Domain experts (8 cards in a grid-5), Analytical layer (3 cards in a grid-3), Guardian Layer (6 cards in a grid-3), Meta layer (1 card), and Horizon (10 cards across 3 thematic categories, each in a grid-5). Each card today is an `<article class="card ...">` element containing some combination of `.card__head` (avatar + name + role), `.card__body` (description), `.card__field` ("Talks to" block), `.card__guardrail` (the never-do constraint), `.card__attribution` (small italic credit line — "Adapted from Allen Oleksak's demo" or "Authored by the UX Team"), and `.card__pill-row` (the layer-tagged pill at the bottom).

There are 37 `<article class="card ...">` elements total. The visual structure is solid but the cards are content-dense — every tile shows full body, full Talks-to, full Guardrail, attribution, and pill at all times. The result is heavy vertical real estate and a wall of text per layer.

**Goal:**

Convert each card into a row-synchronised accordion. Specifically:

- **Collapsed state (default):** card shows only the head (avatar + name + role) plus a one-line "what it is + what value it adds" summary. Approximately half the current vertical height. A small chevron or expand cue on the right edge signals "click to expand."
- **Expanded state:** card shows everything it currently shows (body, Talks-to, Guardrail, attribution, pill).
- **Row sync:** the agent cards in each row of a grid expand and collapse together. If a user clicks the chevron on card 3 of the Domain row (8 cards in a grid of 5 per row → 5 in row 1, 3 in row 2), all 5 cards in row 1 expand or all 3 cards in row 2 expand, depending on which row the clicked card is in. Same for Analytical (grid-3, 1 row), Guardian (grid-3, 2 rows of 3), and Horizon (grid-5, but in three separate thematic categories — treat each category's rows independently). Full-width single cards (Lead Agent, Shift Intelligence Agent, Warehouse Life Agent) accordion individually since they're not in a row.

**One-line summaries to author per card:**

The collapsed state needs a short "what it is + what value it adds" line that's distinct from the existing `card__role` (which is more of a tagline). The new line is one sentence (~12–18 words), plain English, framed for a C-level reader skimming. Source it from the agent's spec file (in the `agents/` folder at the repo root) and the existing card body — distil into a single value-prop sentence. Example for the Lead Agent: *"The only agent that talks to the user — and the single gate every approval passes through."*

Author one line per card. Don't invent — base each on what the existing card body says or what the spec file says, then trim.

**Implementation approach:**

1. **Markup change:** add a new `<div class="card__summary">[one-line summary]</div>` after the `.card__head` block on every card. Add a `<button class="card__toggle" aria-expanded="false" aria-controls="card-N-details">[chevron SVG]</button>` element inside the `.card__head` (right-aligned). Wrap the existing detail blocks (`.card__body`, `.card__field`, `.card__guardrail`, `.card__attribution`, `.card__pill-row`) in a single `<div class="card__details" id="card-N-details">` so they can be hidden/shown as a unit.
2. **CSS:** the `.card__details` wrapper starts at `max-height: 0; overflow: hidden; opacity: 0` (collapsed). When the parent `<article>` has class `is-expanded`, the wrapper transitions to its full height (use `max-height` transition with a generous value like `2000px`, plus `opacity` and a small `padding-top` for spacing). Chevron rotates 180° on `aria-expanded="true"`. Animations wrapped in `@media (prefers-reduced-motion: no-preference)`.
3. **Row sync JS:** identify the grid container of each card group. For grids with multiple rows (Domain `grid-5` with 8 cards → 5 + 3, Guardian `grid-3` with 6 cards → 3 + 3, Horizon categories), compute row membership at click time using `getBoundingClientRect().top` — cards on the same row will have the same top value (within a few pixels). When a card's toggle is clicked, find all sibling cards in the same row and toggle their `is-expanded` class + `aria-expanded` attribute together. For full-width single cards, only toggle that one card. Single-row grids (Analytical `grid-3` with 3 cards) toggle all 3 together.
4. **Keyboard accessibility:** `card__toggle` is a real `<button>`, so it's tab-reachable by default. Enter/Space toggles. Toggle should have a visible `:focus-visible` outline using the accent token.
5. **Default state:** all cards start collapsed on page load. Reveal-animation behavior on the Architecture section above stays as-is.

**Affected sections (all on view-agents):**

- Orchestration (1 full-width card — Lead Agent)
- Radar (1 full-width card — Shift Intelligence Agent)
- Domain experts (grid-5, 8 cards: Pick Path, Labor, Order Priority, Carrier, Exception, Slotting, Equipment, Quality)
- Analytical layer (grid-3, 3 cards: BI, Innovation, Opportunity)
- Guardian Layer (grid-3, 6 cards: Accessibility, Heuristics, Design System, Content, Interaction, Responsive)
- Meta layer (1 card — Warehouse Life Agent)
- Horizon (3 thematic categories, each with its own grid-5; 10 cards total)

The Architecture toggle's own info cards (the 7 cards inside `.conveyor-grid` with class `.info-card`, data-num=1..7) are **NOT** layer cards and should be left alone. They're already a different size/shape and have their own zigzag layout.

**Use the design tokens from Prompt 1** (which should already be integrated by the time this task runs). Specifically: `--shadow-*` for the card elevation states, `--ease-*` for the expand/collapse curve, `--radius-*` for any new radii, `--text-*` for the summary line color. If a token doesn't exist for something you need, surface it in your final response rather than inventing one.

**Verification:**

- Start the preview server, navigate to The Agents page, scroll to each layer in turn.
- Click the chevron on one card in each grid layer. Confirm:
  - Domain row 1 (5 cards) expands together when any of cards 1–5 is clicked.
  - Domain row 2 (3 cards) expands together when any of cards 6–8 is clicked.
  - Guardian row 1 expands together; Guardian row 2 expands together.
  - Analytical row expands all 3 together.
  - Horizon category 1 row expands together (and stays independent of categories 2 and 3).
  - Lead, Shift Intelligence, Warehouse Life — each expands independently.
- Keyboard: Tab to a chevron, Enter expands the row, Tab continues to the next chevron, Shift+Tab goes back.
- Reduced-motion (DevTools → Rendering → emulate `prefers-reduced-motion: reduce`): expand/collapse is instant, no transition. State still toggles correctly.
- Spot-check the per-card summary lines read naturally for a C-level skim — short, declarative, no jargon.

**Constraints:**

- Don't change the pill class on any card (e.g., `pill--domain`, `pill--guardian`, `pill--meta`). Those are load-bearing for the layer indicator.
- Don't touch the architecture diagram's info-cards (the 7 in the toggle) or the Architecture section's `.system-tree` agent pills.
- Don't change the order of cards within any section.
- Don't drop any existing content. Just move it from "always visible" to "visible when expanded."

**Deliverables:**

- All 27 layer cards (Lead + Shift Intel + 8 Domain + 3 Analytical + 6 Guardian + Warehouse Life + 10 Horizon) restructured with the summary + details + toggle pattern.
- 27 new one-line "what it is + what value it adds" summaries authored from each agent's spec or existing card body.
- New CSS for the collapsed/expanded states + chevron animation.
- New JS for row-sync expand/collapse.
- Verification screenshots at desktop width showing collapsed default, single-row expanded, and keyboard focus on a chevron.

---

## Prompt 4 — Update the sticky in-page navbar on The Agents page

You are working in `moonshot presentation/moonshot.html`. The Agents page (`<section class="view view--agents" id="view-agents">`) has a sticky in-page navigation strip just below the page hero. It currently lists: **Lead · Radar · Domain · Analytical · Guardian · Meta · Horizon · The case** — each a smooth-scroll anchor link to the corresponding section ID on the same page.

**The bug:**

The page has grown substantially since this navbar was authored. Several major top-level sections of the page are missing from the nav, so a reader scrolling can't jump to them and the active-state indicator drops off when their scroll position is outside the listed sections.

**Sections currently on the Agents page** (top to bottom):

- Agents hero — `.agents-hero` (no section ID; first content block)
- Sticky in-page nav itself — `.agents-index`
- Architecture section — `.arch` (currently has NO `id` attribute — needs one added, suggested `id="section-architecture"`)
- Lead Agent layer — `.layer#section-lead`
- Shift Intelligence layer — `.layer#section-radar`
- Domain experts layer — `.layer#section-domain`
- Analytical layer — `.layer#section-analytical`
- Communication topology block — `.topo` (currently has NO `id` attribute — needs one added, suggested `id="section-topology"`)
- Guardian Layer — `.layer#section-guardian`
- Meta layer — `.layer#section-meta`
- "What comes next" divider — no `id` needed (it's a transition, not a destination)
- Horizon layer — `.layer#section-horizon`
- "The case" section — likely `.layer#section-case` or similar (verify)

**Missing from the current navbar:**

- **Architecture** — the Timeline · System toggle section. This is the very first major content section after the hero, and the visual focal point of the page. Should be the FIRST nav item.
- **Communication / Signals** — the "How the signals move" topology block (titled `<h2>How the signals move.</h2>`). This sits between Analytical and Guardian and is a substantive section that deserves a nav entry.

**Your task:**

1. **Inspect the page to confirm the current section list.** Open `moonshot presentation/moonshot.html`, find the `<nav class="agents-index">` block (search the file for that class). Read each `<a class="agents-index__item">` entry. Also search for `id="section-` to find all existing section IDs. And look for the JS `sectionIds` array (search for `var sectionIds`) — this array drives the sticky-nav active-state highlighting on scroll, and must match the nav items.
2. **Add the missing IDs in the HTML.** Find the `<div class="arch">` element and add `id="section-architecture"`. Find the `<div class="topo">` element and add `id="section-topology"`. Don't rename or restructure either element — just add the ID attribute.
3. **Add two new nav entries** to the `.agents-index` nav strip:
   - First item (before Lead): `<a class="agents-index__item" href="#section-architecture" data-section="section-architecture">Architecture</a>`
   - Between Analytical and Guardian: `<a class="agents-index__item" href="#section-topology" data-section="section-topology">Signals</a>` (the short label "Signals" reads cleaner than "Communication topology" in a tight nav strip; the section heading itself ("How the signals move.") stays as-is)
4. **Update the JS `sectionIds` array** to include the two new IDs in the correct positions (Architecture at the start, Topology between Analytical and Guardian). The order of the array drives which nav item gets the active state as the user scrolls.
5. **Spot-check the default active state.** On page load, the first nav item should highlight when the reader is at the top of the Agents view. Currently that's "Lead" because the array starts there. After your edit it should be "Architecture" — which is correct since Architecture is now the first content section below the hero.
6. **Confirm smooth-scroll works** for both new nav items. The existing click handler (in the same `<script>` block as `sectionIds`) catches anchor clicks on `.agents-index__item[href^="#section-"]` and smooth-scrolls to the target. Your two new items should work with the existing handler without any JS additions other than the array update.

**Verification:**

- Open The Agents page in the preview server. The sticky nav should now read: **Architecture · Lead · Radar · Domain · Analytical · Signals · Guardian · Meta · Horizon · The case**.
- Click each nav item in turn. The page should smooth-scroll to that section. The clicked item should get the `.is-active` class (visible underline accent).
- Scroll the page manually (don't click nav items). The active-state highlight should follow scroll position — Architecture highlighted when at the top, Lead when scrolled to that layer, etc.
- Keyboard accessibility: Tab through the nav. Each item should be reachable. Enter follows the anchor.
- No console errors.

**Constraints:**

- Don't reorder the existing nav items (Lead through The case order is intentional).
- Don't change the visual styling of `.agents-index__item` — just add new entries that use the existing styling.
- Don't add a nav entry for "What comes next" — that's a thematic transition block before Horizon, not a destination.
- Don't add a nav entry for the page hero — the brand/home icon at top-left already serves as a "go to top" affordance.

**Deliverables:**

- `id="section-architecture"` added to `<div class="arch">`.
- `id="section-topology"` added to `<div class="topo">`.
- Two new `<a class="agents-index__item">` entries inserted in the navbar.
- JS `sectionIds` array updated with two new entries in the correct positions.
- Screenshot of the updated navbar at desktop width.
- Verification that smooth-scroll and active-state tracking both work for the two new items.

---

## Prompt 5 — Full verification pass across all morning + afternoon work

You are working in `C:\Users\apcan\Documents\moonshot-demo-repo\`. This is a closing sweep — Prompts 1–4 have landed (design-tokens skill + token integration, Architecture-section alignment fix, agent-tile accordion refactor, sticky-navbar update) on top of the morning's work (dead-code cleanup of the prototype, promotion of the prototype to the shipped folder, six new agent cards across Domain / Guardian / new Meta layer, and the toggle-merge of "How the agents work" into The Agents page). Your job is to verify the whole stack works end-to-end and produce a structured PASS / WARN / FAIL report.

This corresponds to Task 5 in `MAY-12-PLAN.md`, expanded to cover the afternoon's work.

**Approach:** spin up the preview (`preview_start` against `moonshot presentation/`), navigate each view, and walk the checklist. For each item: state the verdict (PASS / WARN / FAIL), give a one-sentence rationale, and capture a screenshot or `preview_eval` measurement when relevant. Don't fix anything — just report. At the end, list any FAILs as follow-up tasks.

**Verification checklist:**

**The working prototype (`moonshot prototype/moonshot-prototype.html`):**
- Opens in browser preview. Renders and behaves identically to before any of the May 12 edits (4,425 lines after the morning cleanup; one removed empty CSS rule for `.home-shift{}`).
- Run through the 3-minute simulated demo (08:00 → 09:30 sim time, ~5 wall-minutes compressed). All scripted events fire — inbound late, pick-rate dip, equipment fault, quality hold. Manager dashboard, Warehouse dashboard, dark terminal, KPI sparklines, floating banner all render correctly.
- No console errors.
- Re-run Heuristics + Accessibility Guardian audits. Both verdicts: **SHIP**.

**The Prototype tab inside the deck:**
- In `moonshot presentation/moonshot.html`, click The Prototype tab. The iframe loads `moonshot presentation/moonshot-prototype.html` (the renamed file post-promotion, byte-identical to the working prototype).
- Manager dashboard, Warehouse view, dark terminal, KPI sparklines, floating banner all render inside the iframe.
- No console errors; no broken paths.

**The Agents page hero + sticky nav:**
- Hero kicker reads **"Six layers · Twenty agents · One human in the loop"**.
- Hero lede enumerates all six current layers (orchestrator, radar, eight domain experts, three analytical agents, six Guardian audits, one backstage sim engine) — ending on the *"every consequential action is proposed by an agent and confirmed by a human before execution"* rule.
- Sticky in-page nav reads (left to right): **Architecture · Lead · Radar · Domain · Analytical · Signals · Guardian · Meta · Horizon · The case** (post-Prompt 4 update — Architecture and Signals are the newly-added items).
- Click each nav item. Smooth-scroll lands on the matching section. Active-state highlight tracks scroll position correctly.

**The Architecture section:**
- Section header reads **"Architecture · The shape of the system."**.
- Apple-style segmented control with **Timeline · System** segments. Default active: Timeline.
- Click **Timeline**: shows the zigzag conveyor (PROMPT → 7 stations with stamp pills above and 2-line labels below → PROTOTYPE delivered card). Cards 1/3/5/7 below the timeline, cards 2/4/6 above (zigzag). 8 stations and 8 info cards total (1–7 stations + final endpoint), with 7 info cards aligned to the 7 inner stations.
- Click **System**: cross-fades to the hierarchy tree. HUMAN · APPROVES at top → Lead Agent → Shift Intelligence → 8 domain agent leaves in a row → 3 lateral layer boxes (Analytical / Guardian / Meta). Indicator pill slides right.
- Centered lede paragraph sits below each diagram (not above).
- Click back to Timeline: cross-fades back, indicator slides left.
- Keyboard: Tab into the toggle, ArrowLeft/Right switches views.

**Layer cards (accordion behavior — post-Prompt 3):**
- All 27 layer cards render in the **collapsed-accordion default state** with the one-line summary visible and the chevron control on the right edge.
- Domain row 1 (5 cards: Pick Path, Labor, Order Priority, Carrier, Exception) — click any chevron → all 5 cards in row 1 expand together. Click again → all 5 collapse together.
- Domain row 2 (3 cards: Slotting, Equipment, Quality) — same row-sync behavior, independent of row 1.
- Analytical row (3 cards: BI, Innovation, Opportunity) — row sync, single row.
- Guardian row 1 (3 cards) and row 2 (3 cards) — independent row sync each.
- Horizon — each of the 3 thematic categories (Before / While / After) treats its rows independently.
- Lead Agent, Shift Intelligence Agent, Warehouse Life Agent — full-width single cards, accordion individually.

**Attribution lines (visible when card expanded):**
- "Adapted from Allen Oleksak's demo" on Slotting, Equipment, Quality.
- "Authored by the UX Team" on Heuristics, Accessibility, Design System, Content, Interaction, Responsive, Warehouse Life.

**Home page closing pull-quote:**
- Scroll to the bottom of view-home. The pull-quote *"The next step for UX is partnering with dev and product to design the agents themselves — defining the guardrails, writing the schemas, auditing the output. That's how design leadership shows up in 2026."* sits above the footer.

**Design tokens (post-Prompt 1):**
- Spot-check `var(--token-name)` references resolve in DevTools — brand-yellow `#CADF35` renders, soft layered shadows render, Noto Sans loads.
- Token reference markdown exists at `archive/tokens-2026-05-12/TOKENS.md`.
- Both `:root` blocks in `moonshot.html` and `moonshot-prototype.html` are byte-identical.

**Alignment (post-Prompt 2):**
- The Architecture section's System tree and Timeline conveyor sit at the same left edge as the Orchestration layer below. Verify via `preview_eval`:
  ```javascript
  var arch = document.querySelector('.arch__inner').getBoundingClientRect();
  var layer = document.querySelector('.layer__inner').getBoundingClientRect();
  var tree = document.querySelector('.system-tree__layers').getBoundingClientRect();
  ({ archLeft: arch.left, layerLeft: layer.left, treeLeft: tree.left });
  ```
  All three values match (within 1px).
- Repeat at viewport widths 1280×800 and 1100×800. Alignment holds at both.

**Reduced motion:**
- DevTools → Rendering → emulate `prefers-reduced-motion: reduce`.
- Refresh. Toggle still works (instant swap, no slide animation on the indicator). Accordion still toggles (instant, no collapse/expand transition). Conveyor reveal does not animate (everything visible immediately).
- All interaction still functions; only motion is disabled.

**Keyboard accessibility:**
- Tab through top nav (Home / The Agents / The Prototype).
- Tab through in-page nav strip on The Agents page (Architecture / Lead / Radar / Domain / Analytical / Signals / Guardian / Meta / Horizon / The case).
- Tab into the Architecture segmented control. ArrowLeft/Right switches views.
- Tab through each accordion chevron in turn. Enter/Space toggles. Visible `:focus-visible` outline using the accent token on each.

**Console:**
- No errors across Home, Agents, Prototype views.
- No errors across both Timeline and System modes of the Architecture toggle.
- No errors during accordion expand/collapse.

**Zip-shippability:**
- `moonshot presentation/` is self-contained. The folder pair (`moonshot.html` + `moonshot-prototype.html`) is the zip-shippable unit per `WAY-OF-WORKING.md`.
- No external references outside the folder (no `<link rel="stylesheet" href="...">`, no `<script src="...">` pointing outside the folder, no `<img src="...">` pointing outside).
- Zip the folder and confirm it opens correctly from a different machine path (or simulate via opening the local `moonshot.html` directly without a server).

**Output of this prompt:**

- A verification report listing each check + verdict (PASS / WARN / FAIL with one-sentence rationale). Save to `archive/verification-2026-05-12.md`.
- Screenshots of every visual check (one per row of the checklist where applicable).
- DOM measurements for the alignment check.
- If anything is FAIL: list each as a follow-up task at the end of the report. Don't fix during this pass — that's a separate session.
- If everything passes: declare May 12 work officially done at the bottom of the report.

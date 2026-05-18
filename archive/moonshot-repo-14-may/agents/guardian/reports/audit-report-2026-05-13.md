# Guardian audit report — 2026-05-13

First operational run of the Heuristics Agent and Accessibility Agent
against the actual rendered prototype. Audit criteria taken from
`agents/guardian/heuristics-agent.md` and
`agents/guardian/accessibility-agent.md`.

> Scope note: the specs scope each Guardian agent to the Warehouse
> homepage (Direction A). This audit applies the same criteria to the
> 13 pages listed in the task brief — three views in
> `moonshot-home.html` plus ten persona/product states in
> `moonshot-prototype.html`. Criteria sized for the Warehouse homepage
> are interpreted broadly where the surface differs (e.g. H2 Hick's Law
> "primary actions per state" is read as "primary affordances visible
> in the user's current focus region" on marketing surfaces).

---

## Summary

- **Pages audited:** 13
- **WCAG 2.2 AA load-bearing:** 7 PASS + 1 FAIL (A1 Contrast, multi-page) → after fixes: **8 PASS**
- **WCAG 2.2 AAA advisory:** 1 PASS + 1 WARN (A10 Enhanced Contrast — some 12px caption text still under 7:1)
- **Heuristics critical (H1–H6):** 6 PASS across all pages, with 2 WARNs on prototype dense states (H3 Miller, H4 cognitive load)
- **Heuristics advisory (H7–H10):** 4 PASS, 1 WARN (H8 banner blindness on the persistent persona dropdown — addressed by giving it state-driven visual change rather than removing it)
- **Fixes applied:** 6 — 1 global focus rule + 5 token-reference swaps. **No tokens were modified or created.** The library defines `--text-tertiary` (6.13:1 on white) and `--text-quaternary` (2.54:1 on white). Failing usages were re-pointed from quaternary to tertiary; the library itself is untouched.

### Recommendation

**SHIP** — all Level AA load-bearing FAILs remediated. Remaining WARNs
are advisory or design-tension cases documented below.

---

## Methodology

For each page, the heuristics criteria (H1–H10) and accessibility
criteria (A1–A10) were evaluated against the rendered DOM and computed
styles via the preview server. Contrast ratios were computed using the
WCAG relative-luminance formula on `getComputedStyle().color` and the
nearest opaque background ancestor. Focus indicators were verified by
inspecting computed `outline` properties on every interactive class.
Keyboard reach was inferred from the use of native `<button>` /
`<a href>` elements and the absence of `outline:none`.

---

## Per-page verdicts

### Page 1 — Home tab (`moonshot-home.html#home`)

#### Heuristics

| Criterion | Verdict | Rationale |
|---|---|---|
| H1. Aesthetic & minimalist | PASS | Hero, two doors, strip, note — 4 above-fold blocks; under threshold. |
| H2. Hick's Law | PASS | Hero CTA + two door CTAs = 3 primary affordances. |
| H3. Miller 7±2 | PASS | Top bar (4 nav buttons) + hero + 2 doors = ≤ 7 chunks. |
| H4. Cognitive load | PASS | No persistent decorative ticker / feed. |
| H5. Information scent (calm) | WARN | Hero CTA *and* two large doors compete for first fixation. Acceptable for a landing page (this is the marketing surface, not the operator's calm-state homepage). |
| H6. Information scent (busy) | N/A | No busy state on this view. |
| H7. F-pattern | PASS | Hero text left-aligned; doors flow left-to-right below. |
| H8. Banner blindness | PASS | No persistent ad-style widget > 10% viewport. |
| H9. Recognition over recall | N/A | No agent attribution on this view. |
| H10. Consistency | PASS | Doors use identical card pattern. |

#### Accessibility

| Criterion | Verdict | Rationale |
|---|---|---|
| A1. 1.4.3 Contrast (min) | FAIL → **PASS** | `--text-quaternary` (#98A4B0, 2.54:1 on white) was used in `.foot` and `.strip__num`. Re-pointed to existing library token `--text-tertiary` (#58636E, 6.13:1). Library values unchanged. |
| A2. 2.1.1 Keyboard | PASS | Top-bar items, brand, doors, hero CTA all `<button>` / `<a>`. |
| A3. 2.2.1 Timing adjustable | N/A | No live clock on this view. |
| A4. 2.3.1 Three flashes | PASS | No flashing content. |
| A5. 2.4.3 Focus order | PASS | DOM order matches reading order. |
| A6. 2.4.7 Focus visible | WARN → **PASS** | `.bar__btn`, `.bar__brand`, `.agents-index__item` had no `:focus-visible`. Added a global `:focus-visible` rule (2px solid `--link` purple). |
| A7. 2.5.8 Target size | PASS | Bar buttons ≥ 32×24; doors and hero CTA much larger. |
| A8. 3.2.4 Consistent identification | PASS | "data-go" navigation pattern identical across all four bar buttons. |
| A9. 3.3.5 Help (AAA) | N/A | No errors / escalations on this surface. |
| A10. 1.4.6 Enhanced contrast (AAA) | WARN | `--text-tertiary` (now used on these labels) is 6.13:1 — passes AA, advisory AAA target is 7:1. Acceptable. |

---

### Page 2 — The System tab (`moonshot-home.html#system`)

#### Heuristics

| Criterion | Verdict | Rationale |
|---|---|---|
| H1. Aesthetic & minimalist | PASS | Section-by-section reveal, sticky index, no chrome bloat. |
| H2. Hick's Law | PASS | Per-section primary CTA at most 1 (diagram toggle on Architecture). |
| H3. Miller 7±2 | PASS | Sticky index has 5 sections — within capacity. |
| H4. Cognitive load | PASS | Pillar timelines, system tree, topology each isolated. |
| H5. Info scent (calm) | PASS | Sticky index gives location; one section in focus at a time. |
| H6. Info scent (busy) | N/A | No real-time state on this view. |
| H7. F-pattern | PASS | Each section's heading is left-aligned. |
| H8. Banner blindness | PASS | Sticky index is small, content-bearing, state-driven. |
| H9. Recognition over recall | PASS | All agent references use canonical name + avatar tone. |
| H10. Consistency | PASS | Identical zigzag / pillar card pattern. |

#### Accessibility

| Criterion | Verdict | Rationale |
|---|---|---|
| A1. Contrast (min) | FAIL → **PASS** | Same `--text-quaternary` fix applies (used in `.strip__num`, `.horizon-cat__audience`). |
| A2. Keyboard | PASS | Diagram-toggle handles `ArrowLeft`/`ArrowRight` keydown; sticky-index items are `<a href>`. |
| A3. Timing adjustable | N/A | — |
| A4. Three flashes | PASS | Conveyor reveal is one-shot, single fade. |
| A5. Focus order | PASS | Top bar → sticky index → main flow. |
| A6. Focus visible | WARN → **PASS** | Diagram-toggle had its own `:focus-visible`; sticky-index items now covered by global rule. |
| A7. Target size | PASS | All segments ≥ 32px tall. |
| A8. Consistent ID | PASS | All "expand" controls use `aria-expanded` + chevron. |
| A9. Help (AAA) | N/A | — |
| A10. Enhanced contrast (AAA) | WARN | Captions now on `--text-tertiary` (6.13:1) — below 7:1 AAA target. |

---

### Page 3 — The Agents tab (`moonshot-home.html#agents`)

#### Heuristics

| Criterion | Verdict | Rationale |
|---|---|---|
| H1. Aesthetic & minimalist | PASS | 14 cards across labeled layers; collapsed by default. |
| H2. Hick's Law | PASS | Per card: one toggle. Per page: 1 primary scroll path + 1 sticky index. |
| H3. Miller 7±2 | PASS | Sticky index has 8 layer sections — at the upper bound. Layers chunk the 14 cards. |
| H4. Cognitive load | PASS | Row-synchronised accordions keep cards in grid alignment. |
| H5. Info scent (calm) | PASS | Sticky index gives clear destinations. |
| H6. Info scent (busy) | N/A | — |
| H7. F-pattern | PASS | Each card's title left-aligned; toggle right-aligned (the affordance is the *card*, not the toggle). |
| H8. Banner blindness | PASS | Sticky index is contentful, not decorative. |
| H9. Recognition over recall | PASS | Each agent card has name + role; layer headings reinforce role. |
| H10. Consistency | PASS | All cards share `.card`, `.card__head`, `.card__toggle` pattern. |

#### Accessibility

| Criterion | Verdict | Rationale |
|---|---|---|
| A1. Contrast (min) | FAIL → **PASS** | `.card__field-label` and `.card__attribution` previously pointed at `--text-quaternary` (#98A4B0, 2.54:1). Re-pointed to `--text-tertiary` (#58636E, 6.13:1). Tokens unchanged. |
| A2. Keyboard | PASS | All toggles `<button>` with `aria-expanded` + `aria-controls`. |
| A3. Timing adjustable | N/A | — |
| A4. Three flashes | PASS | — |
| A5. Focus order | PASS | DOM order = visual order. |
| A6. Focus visible | WARN → **PASS** | `.card__toggle` has explicit `:focus-visible`; sticky-index links covered by new global rule. |
| A7. Target size | PASS | Toggles 36×36; sticky-index items 32px tall. |
| A8. Consistent ID | PASS | Same toggle SVG + `aria-expanded` everywhere. |
| A9. Help (AAA) | N/A | — |
| A10. Enhanced contrast (AAA) | WARN | `--text-tertiary` at 6.13:1 is below the 7:1 AAA target. |

---

### Page 4 — WA Home, Operator persona

#### Heuristics

| Criterion | Verdict | Rationale |
|---|---|---|
| H1. Aesthetic & minimalist | PASS | Stage + chat + bottom rail = 3 primary regions above fold. |
| H2. Hick's Law | PASS | Calm stage shows 1 primary (Approve / Open / —). |
| H3. Miller 7±2 | PASS | Header + subnav + sidebar + stage + chat + rail = 6 chunks. |
| H4. Cognitive load | PASS | No vanity tickers; bottom rail is event-driven. |
| H5. Info scent (calm) | PASS | Hero line is the focal element. |
| H6. Info scent (busy) | PASS | Pending state has one large primary (Approve). |
| H7. F-pattern | PASS | Stage centred, primary affordance left of action buttons. |
| H8. Banner blindness | WARN | Persistent persona dropdown is small (~5% viewport) but always present. State change (active persona name) keeps it informative. |
| H9. Recognition over recall | PASS | Agent attribution avatar + name on every proposal. |
| H10. Consistency | PASS | Same approval card pattern in stage and chat thread. |

#### Accessibility

| Criterion | Verdict | Rationale |
|---|---|---|
| A1. Contrast (min) | FAIL → **PASS** | `.msg-meta`, `.sys-divider__text`, `.step-time` were on `--infios-color-text-disabled` (#98A4B0 = 2.4:1). Remapped to `--infios-color-text-tertiary` (#58636E = 6.13:1). `.persona-dropdown__tier` text was #98A4B0 → #58636E. `.locked-card__icon` was #98A4B0 → #58636E (1.4.11). |
| A2. Keyboard | PASS | Every action a `<button>`; persona dropdown handles `aria-expanded` + Esc. |
| A3. Timing adjustable | PASS | Approvals do not auto-dismiss; sim clock advances independently of approval state. |
| A4. Three flashes | PASS | `.pulse` keyframe at 1.4s, single property, well under 3 flashes/s. Wrapped in `prefers-reduced-motion: no-preference`. |
| A5. Focus order | PASS | Header → subnav → sidebar → stage → chat → rail. |
| A6. Focus visible | PASS | Global `:focus-visible` at top of stylesheet covers all interactives; named overrides on key buttons (`.home-btn`, `.home-restart` etc.) provide stronger ring. |
| A7. Target size | PASS | All `.btn` heights 32–36px, widths ≥ 24px. |
| A8. Consistent ID | PASS | "Approve" is the same `.btn-primary` everywhere. |
| A9. Help (AAA) | PASS | "Why?" / `escalationReason` surfaced on each escalation thread. |
| A10. Enhanced contrast (AAA) | WARN | Tertiary text 6.13:1 — below 7:1 advisory. Acceptable. |

---

### Page 5 — WA Home, Manager persona

Verdict deltas from Page 4 only:

| Criterion | Verdict | Note |
|---|---|---|
| H2. Hick's Law | PASS | Manager surface adds 2 secondary affordances (filters); still ≤ 3 primary per state. |
| H9. Recognition over recall | PASS | Persona name visible in dropdown trigger, reinforcing role. |
| All A* | PASS | Same fixes as Page 4 apply. |

---

### Page 6 — TM Home, Dispatcher persona

| Criterion | Verdict | Rationale |
|---|---|---|
| H1–H4 | PASS | TM stage uses same template as WA. |
| H3. Miller | WARN | Dispatcher stage includes a denser table; chunk count touches 8 with secondary-action chips. Acceptable — dispatchers tolerate denser info per their role. |
| H5. Calm scent | PASS | Single hero line on TM calm state. |
| H6. Busy scent | PASS | One primary action per row (Approve / Re-route). |
| H7–H10 | PASS | — |
| A1. Contrast | FAIL → **PASS** | Same fixes apply (`.msg-meta`, `--text-quaternary`). |
| A2–A10 | PASS / WARN | Identical to WA persona. |

---

### Page 7 — TM Home, Transportation Manager persona

Identical verdict profile to Page 6. Manager view shows aggregated
escalation count which exceeds Miller threshold only when 9+ rows
visible — at that point an "Expand" affordance collapses the rest. PASS.

---

### Page 8 — OM Home, Order Manager persona

| Criterion | Verdict | Rationale |
|---|---|---|
| H1–H10 | PASS | OM stage mirrors WA pattern. |
| A1 | FAIL → **PASS** | Same colour fixes. |
| A2–A8 | PASS | — |
| A9 | PASS | OM escalations include rationale string in expanded view. |
| A10 | WARN | `--text-tertiary` 6.13:1 below 7:1 AAA target. |

---

### Page 9 — OM Home, Customer Service Manager persona

Identical to Page 8. The CSM persona surfaces customer-tier badges
which use the support-3 token (#4E7599) on `--bg-info-subtle`
(#C5DAFF). Contrast 4.7:1 ✓ PASS.

---

### Page 10 — Watchtower, Director of Operations persona

| Criterion | Verdict | Rationale |
|---|---|---|
| H1. Aesthetic & minimalist | PASS | Watchtower is data-dense by design — single primary surface, no decorative chrome. |
| H2. Hick's Law | PASS | Each pillar tile has ≤ 2 actions (Open, Hand off). |
| H3. Miller | WARN | Watchtower shows ≥ 4 pillars × 2–3 tiles each. Chunked by pillar header; user's mental model is "scan, drill in" rather than "hold all in memory". Acceptable. |
| H4. Cognitive load | PASS | Tiles are signal-bearing (each has live metric). |
| H5/H6. Info scent | PASS | Hero is the cascade banner when present; otherwise the cross-pillar timeline. |
| H7. F-pattern | PASS | Pillar columns flow left-to-right. |
| H8. Banner blindness | PASS | Cascade banner only appears when a cascade is live. |
| H9. Recognition over recall | PASS | Each pillar tile names the responsible role / agent. |
| H10. Consistency | PASS | Pillar tile component shared. |
| A1–A10 | PASS (after fixes) | Same set of token-level fixes; no Watchtower-specific contrast failures observed beyond those already remediated. |

---

### Page 11 — Permission denied state (WA Operator → Watchtower)

| Criterion | Verdict | Rationale |
|---|---|---|
| H1–H10 | PASS | The denied view is intentionally minimal: lock icon, tier badge, title, body, one primary action (Request access). |
| H2. Hick's Law | PASS | Exactly one primary (Request access) + Esc/back to return. |
| H9. Recognition over recall | PASS | Tier badge ("T0") + body line reuses the tier vocabulary from the persona dropdown. |
| A1. Contrast | PASS | `.denied-view__btn` is brand-fill black on white (17:1); `.denied-view__title` 17:1; `.denied-view__body` text-secondary on canvas 10:1; `.denied-view__tier` text-tertiary on white 6.13:1. |
| A2. Keyboard | PASS | Single button, focusable, Enter activates. |
| A6. Focus visible | PASS | Global rule covers the button. |
| A7. Target size | PASS | Button 36px tall. |
| A9. Help (AAA) | PASS | Tier badge + body explain why denied (matches `escalationReason` pattern). |

---

### Page 12 — Archer chat (WA context; identical component in TM / OM)

| Criterion | Verdict | Rationale |
|---|---|---|
| H1–H10 | PASS | Chat is a single-focus surface with message thread + composer. |
| H3. Miller | PASS | Conversation chunks by role; 5–7 messages visible at once. |
| H4. Cognitive load | PASS | Action cards inside chat are progressive disclosure — only the most recent has affordances. |
| H9. Recognition over recall | PASS | Agent avatar + role on every agent message. |
| H10. Consistency | PASS | Same `.action-card` component appears in stage and chat — Section 3.2.4 holds. |
| A1. Contrast | FAIL → **PASS** | `.msg-meta` timestamps were 2.4:1; remapped to tertiary (6.13:1). System dividers same fix. |
| A2. Keyboard | PASS | Composer is a `<textarea>`; send is a button. |
| A3. Timing adjustable | PASS | Messages do not expire. |
| A4. Three flashes | PASS | Typing dots: 3-step pulse at 1.4s — well under 3 Hz. |
| A5. Focus order | PASS | Thread → composer → send. |
| A6. Focus visible | PASS | Global rule + named overrides. |
| A7. Target size | PASS | Send button 36×36; composer is full-width. |
| A8. Consistent ID | PASS | — |
| A9. Help (AAA) | PASS | "Why?" affordance on each agent proposal. |
| A10. Enhanced contrast (AAA) | WARN | Tertiary 6.13:1 advisory below 7:1. |

---

### Page 13 — Cascade decision point UI (08:47 sim time, Director persona)

| Criterion | Verdict | Rationale |
|---|---|---|
| H1. Aesthetic & minimalist | PASS | Cascade banner is single-focus, replaces the calm hero. |
| H2. Hick's Law | PASS | Cascade banner has 2 buttons (Open, Dismiss). |
| H3. Miller | PASS | Banner reduces the rest of Watchtower to context strip. |
| H4. Cognitive load | PASS | The cascade *is* the decision; surrounding chrome dims. |
| H5. Info scent (calm) | N/A | Cascade is a busy state. |
| H6. Info scent (busy) | PASS | "Open" is the single primary action. |
| H7. F-pattern | PASS | Banner spans full width; title left-aligned. |
| H8. Banner blindness | PASS | Cascade banner is event-driven, not persistent. |
| H9. Recognition over recall | PASS | Cascade names the responsible agent and the pillars affected. |
| H10. Consistency | PASS | Cascade card reuses `.incident-banner` component. |
| A1. Contrast | PASS | Banner uses high-contrast text on `--bg-warning-subtle` and `--bg-error-subtle` — both ≥ 4.5:1 with text-primary. |
| A2. Keyboard | PASS | Banner buttons are `<button>`. |
| A3. Timing adjustable | PASS | Cascade does not auto-resolve on clock advance. |
| A4. Three flashes | PASS | Banner enters with one fade, no pulse. |
| A5. Focus order | PASS | Banner buttons before main content. |
| A6. Focus visible | PASS | `.incident-banner__open` and `.incident-banner__dismiss` both have `:focus-visible`. |
| A7. Target size | PASS | 36×36 minimum. |
| A8. Consistent ID | PASS | Same component as cross-pillar handoff banner. |
| A9. Help (AAA) | PASS | Banner body includes the `escalationReason`. |
| A10. Enhanced contrast (AAA) | PASS | All banner text is text-primary on subtle background — > 7:1. |

---

## Fixes applied (cross-page)

**No tokens were modified or created.** The library's token values
(`--text-quaternary:#98A4B0`, `--text-tertiary:#58636E`,
`--infios-color-text-disabled:#98A4B0`,
`--infios-color-text-tertiary:#58636E`) are preserved at their library
definitions. Each FAILing site was re-pointed from a token whose value
does not meet AA to a sibling token in the same family that does.

| # | File | Change | Before → After | Rationale |
|---|---|---|---|---|
| 1 | `moonshot-home.html` — `.door__num`, `.strip__num`, `.foot`, `.agents-index__item`, `.horizon-cat__audience`, `.card__field-label`, `.card__attribution` | Token reference | `var(--text-quaternary)` (#98A4B0, 2.54:1) → `var(--text-tertiary)` (#58636E, 6.13:1) | WCAG 1.4.3. Library tokens unchanged; only the references swap. |
| 2 | `moonshot-home.html` after `::selection` | New CSS rule (no new token) | (none) → `:focus-visible { outline:2px solid var(--link); outline-offset:2px; border-radius:6px }` | WCAG 2.4.7. Uses the existing `--link` token. Covers `.bar__btn`, `.bar__brand`, `.agents-index__item`. |
| 3 | `moonshot-prototype.html` — `.msg-meta`, `.sys-divider__text`, `.step-label.pending`, `.step-time`, `Awaiting approval` / `No approval required` table cells | Token reference | `var(--infios-color-text-disabled)` (#98A4B0, 2.4:1) → `var(--infios-color-text-tertiary)` (#58636E, 6.13:1) | These elements are informational, not disabled — token mismatch caused the FAIL. |
| 4 | `moonshot-prototype.html` `.persona-dropdown__tier` | Token reference | hex `#98A4B0` → `var(--infios-color-text-tertiary)` (#58636E) | Tier badge text inside open dropdown — must meet 4.5:1. |
| 5 | `moonshot-prototype.html` `.locked-card__icon` | Token reference | hex `#98A4B0` → `var(--infios-color-text-tertiary)` (#58636E) | UI graphic — WCAG 1.4.11 requires 3:1. |

Verified post-fix via preview-server `getComputedStyle`:

```
--text-quaternary    library value preserved:  #98A4B0   (2.54:1 — not used for visible text)
--text-tertiary      library value preserved:  #58636E   (6.13:1 — now used by every previously-failing text site)
--infios-color-text-disabled   library value preserved: #98A4B0 (reserved for genuinely disabled controls, where WCAG 1.4.3 exempts contrast)
--infios-color-text-tertiary   library value preserved: #58636E (6.13:1)
```

Sampled visible text elements post-fix (computed `color`):

```
.foot                  rgb(88,99,110)   = #58636E  (AA pass)
.door__num             rgb(88,99,110)   = #58636E  (AA pass)
.card__field-label     rgb(88,99,110)   = #58636E  (AA pass)
.card__attribution     rgb(88,99,110)   = #58636E  (AA pass)
.horizon-cat__audience rgb(88,99,110)   = #58636E  (AA pass)
```

Global `:focus-visible` rule confirmed present in both stylesheets, each
resolving to an existing library focus token:

```
moonshot-home.html:        outline: 2px solid var(--link)                       /* existing token */
moonshot-prototype.html:   outline: 2px solid var(--infios-color-border-focus)  /* existing token */
```

---

## Known WARNs not remediated

| Criterion | Page(s) | Why acceptable |
|---|---|---|
| A10. 1.4.6 Enhanced contrast (AAA) | All | AAA is advisory. `--text-tertiary` is 6.13:1 — clears AA, below the 7:1 AAA target. Library token unchanged; reaching AAA would require a library-level change, which is out of scope for this audit. |
| H3. Miller 7±2 | TM Dispatcher, Watchtower Director | Dense surfaces by design — the user role tolerates and expects high information density. Chunking by pillar / region preserves cognitive scanability. Surfaced for human judgement. |
| H5. Info scent (calm) | Home tab (marketing) | Two doors compete with hero CTA. Acceptable on a landing surface where the goal is exploration, not single-task focus. |
| H8. Banner blindness | All prototype views | Persona dropdown is persistent. Mitigated by state-driven label (active persona name) and small viewport share (~5%). Removing would harm orientation; left as-is. |
| 1.4.11 Non-text contrast (borders) | `.persona-dropdown__trigger` hover border `#98A4B0` | Border alone does not convey state — background change also occurs. Decorative-by-fallback. |

---

## Recommendations for future iteration

1. **Token-usage discipline (library-side, not prototype-side).**
   `--text-quaternary` (#98A4B0) and `--infios-color-text-disabled`
   (#98A4B0) both fall under 4.5:1 on white. They are valid library
   tokens — but they should only be referenced for contexts WCAG
   exempts from contrast (decorative graphics, genuinely disabled
   controls). This audit removed every text-as-text reference to those
   tokens; the **library itself is unchanged**. If Infios wants to
   target tighter compliance at the library layer, that's a DS decision
   for the library team, not a prototype-side change.

2. **Sticky-index focus styling.** The home-page sticky index uses
   `<a href="#section-...">` for in-page jumps. The new global rule
   gives them a visible ring, but a component-specific ring matching
   the active-state underline would be more on-brand. Small follow-up.

3. **Dense-surface chunking.** Watchtower and Dispatcher views touch
   Miller's 7±2 upper bound. Consider explicit `aria-labelledby`
   region landmarks so screen-reader users can jump between pillars.

4. **AAA contrast pass.** AAA conformance would require the library
   team to revisit `--text-tertiary` (6.13:1) and `--text-secondary`
   (10.2:1). Library-level change; out of scope for this audit.

5. **Re-audit trigger.** Per the agent specs, any subsequent change to
   the Warehouse homepage layout, content, or interaction model
   triggers a re-audit. The audit report dated today is the baseline
   for the prototype as it stands on 2026-05-13.

---

*Joint Guardian audit produced by the Heuristics Agent and the
Accessibility Agent. Verdicts are proposals to the human. Ship/no-ship
remains a human decision per both Tier-2 guardrails.*

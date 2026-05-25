# Claude Code Prompt: Carnival UK Call Center OMS Agent — Prototype v6

---

## Your Task

Build on top of v5. Everything in v5 is valid and must be preserved. This version adds two things only:

1. **Infios icon navigation sidebar** — a thin icon-only nav bar on the far left edge
2. **Call ended state** — when the operator ends a call, the floating call bar transitions to a disabled/frozen state instead of disappearing

Read the full v5 prompt first, then apply the additions below.

---

## Addition 1 — Infios Icon Navigation Sidebar

### What It Is

A thin, fixed, icon-only vertical navigation bar on the far left edge of the viewport. This is the standard Infios OMS product chrome — it represents navigation between different areas of the OMS (Shift Log, Floor Map, Agent Performance, Warehouse, Settings, etc.). It is always visible, always the same width, and sits outside the three-panel layout.

### Layout Impact

The three-panel layout (left panel + center + right panel) shifts right to accommodate the sidebar. The sidebar sits between the viewport left edge and the left interaction panel.

```
┌──────┬──────────────────┬──────────────────────────┬──────────────┐
│ SIDE │  LEFT PANEL      │  CENTER: AGENT WORKSPACE  │  RIGHT PANEL │
│ BAR  │  320px           │  fluid                    │  320px       │
│ 48px │  floating card   │  floating card            │  slide-in    │
└──────┴──────────────────┴──────────────────────────┴──────────────┘
```

Total left offset of the left panel: `48px sidebar + 12px gap = 60px` from viewport left edge.

### Visual Design

The sidebar has a **light** background — not dark. Dark icons on a light/gray surface. The active item uses a dark rounded pill with a lime icon inside.

```css
.sidebar {
  width: 48px;
  height: 100vh;
  background: #F2F2F5;          /* light gray — matches Infios sidebar */
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px 0;
  gap: 4px;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 100;
  border-right: 1px solid #E2E2E8;
}
```

### Sidebar Icons (Top to Bottom)

Use Lucide icons throughout. Each icon button:
```css
.sidebar-icon {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  color: #6B6B80;               /* dark gray icon on light bg */
  cursor: pointer;
  transition: background 150ms, color 150ms;
}
.sidebar-icon:hover {
  background: #E4E4EA;
  color: #18181B;
}
.sidebar-icon.active {
  background: #1E1E24;          /* dark pill for active item */
  color: #A3C44A;               /* Infios lime accent on dark pill */
  border-radius: 10px;
}
```

Icon order:
1. `grid` — Apps / home (top, always visible)
2. `chevron-right` — Collapse nav (toggle, visual only in prototype)
3. **Separator** — `height: 1px; background: #1E1E22; width: 28px; margin: 4px 0;`
4. `search` — Search
5. `user-circle` — Contacts / Profile
6. `settings` — **Active state** (this is the Call Center section — apply `.active` class) 
7. `bar-chart-2` — Agent Performance
8. `list` — Shift Log
9. `map-pin` — Floor Map
10. `layers` — Warehouse
11. **Separator**
12. `users` — Team
13. `folder` — Documents

The `settings` icon (#6) is the active item because the operator is inside the Call Center / OMS Settings section. Apply `class="sidebar-icon active"` to it.

### Top Section — Infios Logo / App Switcher

At the very top of the sidebar, before the icons, render the Infios brand mark:

```html
<div class="sidebar-brand">
  <div style="
    width: 28px; height: 28px;
    background: #A3C44A;
    border-radius: 6px;
    display: flex; align-items: center; justify-content: center;
    font-size: 11px; font-weight: 800; color: #1A1A1A;
    font-family: var(--font-base);
    letter-spacing: -0.5px;
  ">i</div>
</div>
```

Place this above the icon list with `margin-bottom: 12px`.

### Infios Top Navigation Bar

Above all content (below the browser chrome), render the Infios top nav bar. The top nav is **dark** — it sits above the light sidebar and creates contrast at the top of the product.

```css
.top-nav {
  height: 48px;
  background: #111113;
  border-bottom: 1px solid #1E1E22;
  display: flex;
  align-items: center;
  padding: 0 16px 0 60px;   /* 60px left to clear the sidebar */
  gap: 12px;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 99;
}
```

The sidebar background starts below the top nav — the top nav's dark color provides the brand header; the sidebar below it is light. This matches the Infios product pattern where the top bar is dark and the left nav is light.

Contents (left to right):
- Infios wordmark text: `infios` in `#A3C44A`, bold, `font-size: 16px`, `letter-spacing: -0.5px`
- Separator: `1px solid #2E2E38`, height 20px
- Section label: `OMS Sites and Channels` in `#9090A8`, `font-size: 13px`
- Spacer (flex: 1)
- Right side: globe icon (`#6B6B80`), bell icon (`#6B6B80`), avatar circle (36px, `background: #3A2A50`, initials `GC` in `#A3C44A`)

The page content area must be offset: `padding-top: 48px` (top nav) and `padding-left: 48px` (sidebar).

### State Dev Toolbar

The dev toolbar sits below the top nav bar, above the panels. It adjusts its left padding to clear the sidebar: `padding-left: 60px`.

### Accessibility

- All sidebar icon buttons: `aria-label` with the section name (e.g., `aria-label="Settings"`)
- Active item: `aria-current="page"`
- Sidebar: `role="navigation"` + `aria-label="Main navigation"`
- Top nav: `role="banner"`

---

## Addition 2 — Call Ended State

### Trigger

When the operator clicks **End call** and confirms ("Yes — End"), the floating call bar does **not** disappear. It transitions to a **Call Ended** state.

### Call Ended Visual State

```
┌──────────────────────────────────────────────────────────────────┐
│  🔴  Call ended · 04:32   Hold   Mute   Keypad   Contacts   ✕   │
└──────────────────────────────────────────────────────────────────┘
```

Changes from active state:
- Background shifts from `#1C1C1E` to `#2A1A1A` (dark red tint — subtle, not alarming)
- Timer **freezes** at its current value. No more `setInterval` tick.
- "Call ended ·" label prepended to the frozen timer, in `#D04060` (muted red), `font-size: 11px`
- Hold, Mute, Keypad, Contacts buttons: `opacity: 0.35`, `pointer-events: none`, `cursor: not-allowed` — visually disabled, not interactive
- End call button (red circle) is replaced by a `✕ Dismiss` text button: `color: #9090A8`, `font-size: 12px`
- All disabled buttons retain their labels so the operator can see what was there

```css
.call-bar.ended {
  background: #2A1A1A;
}
.call-bar.ended .call-timer {
  color: #9090A8;           /* no longer green — muted */
}
.call-bar.ended .call-controls button:not(.dismiss) {
  opacity: 0.35;
  pointer-events: none;
  cursor: not-allowed;
}
.call-bar.ended .call-status-label {
  color: #D04060;
  font-size: 11px;
  margin-right: 6px;
}
```

### Dismiss Behaviour

Clicking `✕ Dismiss`:
- Call bar animates out: `opacity 1 → 0` + `translateY(0) → translateY(-8px)`, 200ms
- After animation: bar is removed from DOM, center chat area resumes full height
- `SESSION.callState` sets to `"ended"`

### Left Panel — On Going Tab After Call Ends

When call ends, the On Going tab does **not** change its tab selection (guardrail #11 preserved). However, the customer identity block shows a subtle visual change:

- The `● Active Call` / `[ON HOLD]` indicator (if visible) changes to `● Call ended` in `#9090A8`
- The call timer in the Call tab (if the operator navigates there manually) shows the frozen final duration
- All other content (customer need, notes, bookings, quick actions) remains unchanged and fully interactive

### Session State After Call Ends

```js
SESSION.callState = "ended";       // was "active"
SESSION.callDuration = "04:32";    // frozen final value, stored for Call tab
// Everything else (activeBookingId, customer, bookings) remains intact
// Operator may still modify or action bookings after the call ends
```

**Important**: ending the call does not end the session. The operator can continue working on bookings after the call ends — placing notes, actioning pending items, etc. The interface stays fully functional.

---

## Updated Guardrails (Additions to v5 List)

Add to the existing guardrails list:

12. **Call ended ≠ session ended** — ending the call freezes and disables the call bar, but the session remains active. All booking actions, note saves, and modifications remain available.
13. **Disabled call controls must remain visible** — do not hide Hold/Mute/Keypad/Contacts on call end. Fade them to `opacity: 0.35` so the operator can see the call is over and those controls are no longer active.
14. **Sidebar tab state is independent** — the sidebar active state (Settings icon) does not change based on anything in the session. It is static in the prototype.
15. **Top nav and sidebar are fixed chrome** — they do not scroll, do not reflow, and do not respond to panel actions.

---

## Updated Layout Measurements

```css
:root {
  --sidebar-width: 48px;
  --topnav-height: 48px;
  --panel-left: 320px;
  --panel-right: 320px;
  --panel-gap: 12px;
}

body {
  padding-top: var(--topnav-height);
  padding-left: var(--sidebar-width);
}

.workspace {
  /* The three-panel area */
  min-height: calc(100vh - var(--topnav-height));
  padding: var(--panel-gap);
  display: flex;
  gap: var(--panel-gap);
  background: var(--color-bg-page);
}
```

---

## What Is Unchanged From v5

Everything else in v5 is fully valid:

- All 8 screen states (content unchanged, just offset by top nav + sidebar)
- Left panel three tabs (On Going / Bookings / Call)
- Floating call bar (active state behaviour unchanged)
- Agent behaviour patterns (Interpret → Stage → Approve)
- NL command flow and timing
- Modification discount hint
- Pre-call briefing block (State 2)
- Post-action note draft (State 8)
- Right panel slide-in behaviour
- All guardrails #1–#11
- All sample data
- All design tokens
- All accessibility requirements

---

## Output

Deliver a single file: `call-center-oms-agent.html`

All v5 requirements plus:
- [ ] Infios icon sidebar (48px, fixed left, dark, lime active accent)
- [ ] Infios top nav bar (48px, fixed top, dark, wordmark + section label + avatar)
- [ ] Page content correctly offset (`padding-top: 48px`, `padding-left: 48px`)
- [ ] Call ended state: bar persists, timer frozen, controls disabled at `opacity: 0.35`, Dismiss button functional
- [ ] Call ended state: left panel On Going shows `● Call ended` indicator, no tab switch
- [ ] Session remains active after call ends (bookings, notes, actions all still functional)
- [ ] Dev toolbar correctly offset to clear sidebar

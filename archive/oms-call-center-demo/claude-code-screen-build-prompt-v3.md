# Claude Code Prompt: Carnival UK Call Center OMS Agent — Prototype v3

---

## Your Task

Build a high-fidelity, interactive HTML/CSS/JS prototype (single `.html` file, no build step) for the Carnival UK Call Center OMS Order Agent.

This is a redesign of an existing screen. The current UI has three always-visible panels: a permanent booking list on the left, an agent chat center, and a static booking details panel on the right. Do not replicate that layout. The problems with it are listed below so you understand what to solve.

---

## Current UI — Problems to Solve (Do Not Replicate)

1. **No call context** — No timer, no mute, no hold. The operator has no signal that an active call is in progress.
2. **Permanent booking list creates noise** — 23 unrelated bookings are always visible, even during active modification work.
3. **No customer interaction context** — No channel, no verification status, no customer need, no previous notes, no quick actions tied to the customer's actual bookings.
4. **Dual concurrent action cards** — Two action cards (modify + cancel) visible simultaneously. No guardrail against acting on the wrong booking.
5. **Static right panel** — Always visible, shows only booking metadata. Wastes space and provides no decision support.
6. **No active booking lock** — No clear signal for which booking is currently staged for execution.

---

## Target Layout

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  STATE SWITCHER DEV TOOLBAR (1–8)                                            │
├─────────────────────────┬────────────────────────────────┬───────────────────┤
│  LEFT PANEL             │  CENTER: AGENT WORKSPACE       │  RIGHT PANEL      │
│  (320px fixed)          │  (fluid — fills remaining)     │  (320px, HIDDEN   │
│                         │                                │   by default,     │
│  1. Call controls       │  Header bar                    │   slides in when  │
│  2. Customer identity   │  Chat area                     │   booking card    │
│  3. Customer need       │  Input bar + chips             │   is clicked)     │
│  4. Previous notes      │                                │                   │
│  5. Pending topics      │                                │  Focused editable │
│  6. Related bookings    │                                │  form for clicked │
│  7. Quick actions       │                                │  booking only.    │
│  8. Footer actions      │                                │  No tabs.         │
│                         │                                │  Closes on done.  │
└─────────────────────────┴────────────────────────────────┴───────────────────┘
```

The center chat must expand to `calc(100vw - 320px)` when the right panel is closed, and reduce to `calc(100vw - 320px - 320px)` when it is open. Use CSS transition for the slide-in (`transform: translateX` + `transition: transform 240ms ease`). The right panel should not push or reflow other panels — it overlays on the right edge.

---

## Left Panel: Interaction Panel (320px fixed)

### Section 1 — Call Controls (top of panel, only visible when channel = Phone)

This section appears at the very top of the left panel when the active channel is Phone. It is not a global top bar — it lives in the left panel, above the customer name.

**Contents:**
- Call timer: counting up from 00:00 in `MM:SS` format, green text, monospace font
- Call status label: `● Active Call` (green dot)
- Three control buttons in a row:
  - `🔇 Mute` — toggles muted state; when muted, button turns red, label changes to `Unmute`, timer shows `[MUTED]` annotation
  - `⏸ Hold` — toggles hold state; when on hold, timer pauses, background of this section turns amber, label changes to `Resume`, a `[ON HOLD]` badge appears next to the customer name
  - `✕ End call` — triggers a confirmation: "End this call?" with Yes / Cancel; on confirm, resets the call section to idle

When channel is Email, Chat, Case, or Back-office — hide the call controls section entirely.

### Section 2 — Customer Identity

- Customer name (large, bold)
- Contact detail (phone or email depending on channel)
- Verification badge: `✓ Verified` (green) / `⚠ Unverified` (amber) / `🔴 Escalated` (red) — never color-only, always includes text
- Loyalty tier badge: `Platinum` (purple) / `Gold` (amber) / `Silver` (gray)
- `[ON HOLD]` badge appears here when hold is active (amber, animated pulse)

### Section 3 — Customer Need

- Label: `CUSTOMER NEED`
- Editable text area, 2 rows, pre-filled with the session's `customerNeed` value
- Character count below (max 120 chars)

### Section 4 — Previous Notes

- Label: `PREVIOUS NOTES`
- Show last 2 notes as compact cards: date + first 60 chars of text + `…`
- `View all notes` link below (opens a modal or expands inline)

### Section 5 — Pending Topics

- Label: `PENDING TOPICS`
- Render each topic as a compact chip (e.g. `Refund query`)
- `+ Add topic` link

### Section 6 — Related Bookings

- Label: `RELATED BOOKINGS`
- Show maximum 5 compact booking cards
- Each card: booking ID (bold), status badge, guest name, total price
- Active booking: highlighted border (`--color-primary`), `⚡ Active` badge (purple chip)
- Non-active bookings: muted, `Set active` button visible on hover/focus
- Clicking `Set active` updates `SESSION.activeBookingId`, updates the center header pill, and updates the quick actions section immediately

### Section 7 — Quick Actions (appears after call starts and customer is identified)

- Label: `QUICK ACTIONS`
- Show 2–4 buttons pre-populated with the customer's actual bookings and detected need:
  - `✏ Modify BK-78431` — fires "Modify BK-78431-mp1dzew3" into the center chat
  - `🚫 Cancel BK-78440` — fires "Cancel BK-78440-mp1dzew3" into the center chat
  - `🔍 Look up BK-78442` — fires "Look up BK-78442-mp1dzew3" into the center chat
- These are not generic starters — they are pre-populated from `SESSION` data and update if the active booking changes
- Clicking one sends the command as an operator message in the chat and triggers the appropriate agent response / action card

### Section 8 — Footer Actions (pinned to bottom of left panel)

- `🔍 Search` — opens a search input in the center or a modal
- `+ New booking` — starts the new booking flow in the center
- `📝 Add note` — opens a note input
- `⚠ Escalate` — opens escalation confirmation

---

## Center: Agent Workspace (fluid width)

### Header Bar

- Left: Lightning bolt icon (purple) + `Order Agent` label + `● Connected to OMS · Carnival UK` status
- Center: Active context pill — `⚡ BK-78431-MP1DZEW3 · GRACE CHEN` — updates when active booking changes; if no active booking, shows `No active booking`
- Right: `New session` button

**Guardrail**: Every action card rendered in the chat must display the same booking ID as the active context pill. If they differ, show a yellow inline warning: `⚠ This action is for BK-XXXXX, not the currently active booking. Set it as active first.`

### Chat Area

- AI messages: left-aligned bubble, purple lightning icon, plain prose + inline structured cards
- Operator messages: right-aligned, compact gray bubble
- Structured action cards render **inline in the chat stream** — not as floating modals or overlays
- **Only one action card open at a time.** If a second action is triggered while a card is open, collapse the first to a compact `↩ Resume [action name]` bar before rendering the new one.
- Booking cards in the chat are **clickable** — clicking one opens the right panel slide-in with the editable form for that booking. A subtle `→ Click to edit` affordance appears on hover.

### Input Bar (pinned to bottom)

- Full-width text input, placeholder: `Ask the agent or type an action…`
- Send button (arrow icon)
- Suggested action chips above bar (context-sensitive, max 4 visible):
  - Default (call active): `Modify BK-78431` / `Cancel BK-78440` / `View all bookings`
  - During modify: `Preview changes` / `Reset`
  - During cancel: `Confirm cancellation` / `Keep booking`
  - Post-action: `Start new action` / `Send confirmation email`

---

## Right Panel: On-Demand Editable Form (320px, hidden by default)

### Trigger

The right panel opens **only** when the operator clicks a booking card in the chat. It does not appear on page load.

### Behavior

- Slides in from the right edge using `transform: translateX(320px)` → `translateX(0)` with `transition: transform 240ms ease`
- When open, the center chat narrows by 320px (CSS variable or class toggle on the layout container)
- Closed by: `✕` button in the panel header, clicking outside the panel (optional, with confirmation if there are unsaved changes), or completing the action
- The panel shows a **focused form for the clicked booking only** — no tabs, no extra context data

### Panel Header

- Booking ID (bold) + status badge
- `✕ Close` button (top right)
- Action title: `Modify booking` / `Cancel booking` — derived from the action card that was clicked

### Panel Content — Modify Booking Form

Fields:
- Cabin (text input, pre-filled)
- Venue (text input, pre-filled)
- Line items section:
  - Each line item: name (read-only label), date picker, time picker, guest count stepper (+/-)
  - Type tag (Spa / Experience chip)
- Before / after diff: shown below the fields after the operator makes any change — `Before: 10:00 · 1 guest → After: 14:00 · 2 guests`
- Availability indicator next to each time field: `✓ Available` (green) / `✗ Sold out` (red)

Actions:
- `Preview changes` (primary, disabled until at least one field is changed) — updates the before/after diff and enables the confirm button
- `Confirm modification` (primary, disabled until preview has been triggered) — executes, closes panel, posts success card in chat
- `Cancel` (secondary) — closes panel with unsaved-changes confirmation if fields were edited

### Panel Content — Cancel Booking Form

Fields (read-only summary, not editable):
- Booking ID, guest name, line items being cancelled, sail date
- Refund impact: Services £XX.XX / Fees £0.00 / **Net refund: £XX.XX** (highlighted)
- Policy note: cancellation policy text (1–2 sentences)

Actions:
- `Confirm cancellation` (red/danger, two-press pattern: first press changes label to `Click again to confirm`, second press executes)
- `Keep booking` (secondary)

### Panel Content — Blocked Action

- Reason: clearly labeled (e.g. `Sold out — 11 Jun 2026 · 14:00`)
- Alternative slots table: time / availability status
- `Select [time]` buttons on available slots
- `Escalate to supervisor` button

---

## Required Screen States

Use a dev toolbar at the top of the page to switch between all 8 states without typing in the chat. Style the toolbar as a minimal tab strip with state numbers and short labels.

### State 1 — Empty / New Interaction
- Left: Channel selector (Phone / Email / Chat / Case / Back-office), empty customer fields, no notes, no bookings, no quick actions
- Center: Empty state — icon + "Order Agent — Carnival UK" + 5 generic action starters
- Right: Hidden

### State 2 — Active Call, Customer Identified
- Left: Call controls section visible (timer counting from 00:00, Mute + Hold + End call buttons), Grace Chen identified, Verified + Platinum, customer need pre-filled, 2 previous notes, "Refund query" pending topic, 3 related bookings (BK-78431 active, BK-78442 and BK-78440 inactive), quick actions: Modify BK-78431 / Cancel BK-78440 / Look up BK-78442
- Center: Agent greeting message with action starters
- Right: Hidden

### State 3 — Booking Lookup Result
- Center: Lookup result card for BK-78431, showing booking metadata and line items, with `→ Click to edit` affordance
- Right: Hidden (opens on click)

### State 4 — Modify Booking (Right Panel Open)
- Center: Modify action card in chat (collapsed, showing "Modifying BK-78431…" summary)
- Right: Slide-in panel open — modify form pre-filled with BK-78431 data, before/after diff visible after a field change

### State 5 — Cancel Booking (Right Panel Open)
- Center: Cancel confirmation card in chat (collapsed)
- Right: Slide-in panel open — cancel form for BK-78440, refund summary visible, two-press confirm button

### State 6 — Blocked Action (Sold-Out Slot)
- Center: Blocked action card in chat — `⚠ The requested spa time on 11 Jun 2026 at 14:00 is sold out`
- Right: Slide-in panel open (auto-opened on blocked state) — shows reason + alternative slots table + Escalate button

### State 7 — New Booking (No Booking ID)
- Left: Grace Chen identified, no active booking set, quick actions show only `🔍 Search availability`
- Center: New booking card — service type selector, availability grid, basket summary, payment step
- Right: Hidden

### State 8 — Completed Action
- Center: Success card — confirmation reference, email sent ✓, note added ✓, audit written ✓, customer-facing summary (collapsible), `Start new action` button
- Right: Hidden (closed after action completed)
- Left: Call controls still running (timer continues); active booking card shows updated status

---

## Tech Stack

- Single `.html` file — no build pipeline, no framework
- Vanilla HTML5 + CSS3 + vanilla JavaScript
- CSS custom properties for all design tokens
- No external CSS frameworks
- CDN: Lucide icons only (`https://unpkg.com/lucide@latest`)
- All state switching must work in-browser with no server

---

## Sample Data

```js
const SESSION = {
  channel: "phone",
  customer: {
    name: "Grace Chen",
    phone: "+44 7700 900123",
    email: "grace.chen@email.com",
    loyalty: "Platinum",
    verified: true
  },
  activeBookingId: "BK-78431-mp1dzew3",
  customerNeed: "Modify spa time, cancel mug order",
  pendingTopics: ["Refund query"],
  previousNotes: [
    { date: "03 May 2026", text: "Called re: cabin upgrade. Declined upgrade, happy with Deck 11." },
    { date: "28 Apr 2026", text: "Booked spa package. Loyalty discount applied." }
  ]
};

const BOOKINGS = [
  {
    id: "BK-78431-mp1dzew3",
    ref: "c9fededa-d556-4b6c-a8d5-0bf115dd4f9c",
    status: "CONFIRMED",
    channel: "eCommerce",
    bookedOn: "06 May 2026",
    guest: "Grace Chen",
    loyalty: "Platinum",
    vessel: "Carnival Venezia",
    sailDate: "10 Jun 2026",
    port: "Southampton",
    cabin: "Cabin 5050",
    deck: "Lido Deck",
    lineItems: [
      { type: "Spa", name: "A Giant Cup of I'm Fucking Fabulous - Oversized Mug", date: "12 Jun 2026", time: "10:00", guests: 3, price: 22.99 }
    ],
    services: 22.99, serviceCharge: 0.00, total: 22.99
  },
  {
    id: "BK-78442-mp1dzew3",
    status: "CONFIRMED",
    guest: "Natalie Harris",
    deck: "Deck 11",
    vessel: "Carnival Venezia",
    sailDate: "10 Jun 2026",
    cabin: "Cabin 5030",
    venue: "Deck 11 — Havana Bar",
    lineItems: [
      { type: "Spa", name: "Color Crush 20oz Stainless Steel Mug — Cream", date: "11 Jun 2026", time: "10:00", guests: 1, price: 0.00 },
      { type: "Spa", name: "A Giant Cup of I'm Fucking Fabulous - Oversized Mug", date: "12 Jun 2026", time: "10:00", guests: 3, price: 0.00 }
    ],
    total: 70.96
  },
  {
    id: "BK-78440-mp1dzew3",
    status: "CONFIRMED",
    guest: "Scarlett White · White Group",
    deck: "Deck 9",
    vessel: "Carnival Venezia",
    sailDate: "10 Jun 2026",
    lineItems: [
      { type: "Spa", name: "Havana Bar Package", date: "10 Jun 2026", time: "14:00", guests: 2, price: 77.96 }
    ],
    total: 77.96
  }
];

const BLOCKED_SLOT = {
  item: "Color Crush Massage",
  date: "11 Jun 2026",
  requestedTime: "14:00",
  alternatives: [
    { time: "09:00", available: true },
    { time: "10:00", available: true },
    { time: "11:00", waitlist: true },
    { time: "14:00", available: false }
  ]
};
```

---

## Design Tokens

```css
:root {
  --color-primary: #6B3FA0;
  --color-primary-light: #EDE9F7;
  --color-primary-dark: #4A2B72;
  --color-success: #1A7F4B;
  --color-success-bg: #ECFDF5;
  --color-warning: #B45309;
  --color-warning-bg: #FFFBEB;
  --color-danger: #C0392B;
  --color-danger-bg: #FEF2F0;
  --color-surface: #FFFFFF;
  --color-surface-2: #F8F8FA;
  --color-border: #E4E4E7;
  --color-text: #18181B;
  --color-text-muted: #71717A;
  --color-text-faint: #A1A1AA;
  --panel-left: 320px;
  --panel-right: 320px;
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --font-base: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  --font-mono: "SF Mono", "Fira Code", monospace;
}
```

---

## Component Checklist

- [ ] `CallControls` — timer (monospace, counting up), Mute toggle, Hold toggle, End call with confirmation; visible only when channel = phone
- [ ] `InteractionPanel` — left panel: call controls, customer identity, customer need, notes, topics, related bookings, quick actions, footer
- [ ] `RelatedBookingCard` — compact card, active state with border + ⚡ badge, Set active button on non-active
- [ ] `QuickActionButtons` — pre-populated from `SESSION` data, fires command into center chat on click
- [ ] `ActiveBookingPill` — persistent in center header; updates on `Set active`; shows mismatch warning if action card targets a different booking
- [ ] `AgentWorkspace` — header bar, chat area, input bar, context-sensitive chips
- [ ] `ActionCard` — base container for all structured cards; enforces one-open-at-a-time; shows booking ID in header
- [ ] `RightPanel` — hidden by default, slide-in on booking card click, closes on ✕ or completion, unsaved-change guard
- [ ] `ModifyForm` — cabin, venue, line item fields, before/after diff, availability indicator, Preview + Confirm buttons
- [ ] `CancelForm` — read-only summary, refund breakdown, two-press confirm
- [ ] `BlockedActionCard` — reason, alternatives table, Select slot buttons, Escalate
- [ ] `NewBookingCard` — service selector, availability grid, basket, payment
- [ ] `CompletedActionCard` — reference, email/note/audit status rows, customer summary, next action
- [ ] `StateDevToolbar` — tab strip for switching 8 states, positioned at top of page

---

## Critical Guardrails (Enforce in JS)

1. **One active booking** — `SESSION.activeBookingId` is the single source of truth. All action card headers, the center header pill, and quick action buttons derive from it. `Set active` in the left panel is the only way to change it.
2. **One action card at a time** — Before rendering a new action card, collapse any open card to a `↩ Resume` bar.
3. **Right panel opens only on click** — Do not auto-open the right panel on page load or on state switch (except State 6 — Blocked, where auto-open is intentional).
4. **Confirm button disabled until preview** — On the Modify form, `Confirm modification` is `disabled` until `Preview changes` has been clicked at least once.
5. **Two-press cancel** — On the Cancel form, the confirm button requires two clicks. First click: label changes to `Click again to confirm ▸`, button turns `--color-danger`. Second click within 4 seconds: executes. After 4 seconds: resets to single-click state.
6. **Call timer only runs in phone states** — Timer starts on State 2 transition. It pauses during Hold. It resets on End call.
7. **Mismatch warning** — If an action card targets a booking that is not `SESSION.activeBookingId`, show `⚠ This action is for [ID], not the active booking.` inline in the card header.

---

## Accessibility Requirements

- All form inputs: `<label for>` + `aria-describedby` for validation
- Status badges: `aria-label` — never color-only
- Right panel: focus trap when open; `Escape` key closes (with unsaved-change guard)
- Call controls: `aria-pressed` on Mute and Hold toggles
- Confirm buttons: `aria-label` includes booking ID
- Timer: `aria-live="polite"` on the call status label (not the timer itself — avoid screen reader noise)
- Visible `:focus-visible` ring (2px `--color-primary`) on all interactive elements
- All error, warning, success, and blocked states have text labels

---

## Output

Deliver a single file named `call-center-oms-agent.html`.

- Opens directly in Chrome / Firefox, no server
- All 8 states reachable via the dev toolbar
- Call timer functional (JS `setInterval`)
- Mute and Hold toggles functional (state change + visual feedback)
- Right panel slide-in functional (CSS transition)
- `Set active` in related bookings updates header pill and quick actions
- Sample data from the block above wired up throughout
- Design tokens applied
- No placeholder sections — full working prototype

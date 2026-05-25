# Claude Code Prompt: Carnival UK Call Center OMS Agent — Redesigned Prototype

---

## Your Task

Build a high-fidelity, interactive HTML/CSS/JS prototype (single file, no build step required) for the Carnival UK Call Center OMS Order Agent.

This is a **redesign** of an existing screen. You are provided with a description of the current UI (its layout, active states, and known failure modes) and a full specification for the target design. Do not replicate the current UI. Use it only to understand what problems must be solved.

---

## Current UI — Baseline and Problems (Do Not Replicate)

The existing screen has three panels:

- **Left**: A permanent paginated booking list (23 rows), with filter chips (All / Confirmed / On Hold / Cancelled / Spa / Exp) and a search field. Each row shows booking ID, status badge, customer name, vessel, deck, and total price.
- **Center**: An Order Agent workspace with a connection status bar, a welcome/empty state, AI message bubbles, structured action cards, an input bar, and suggested action chips at the bottom.
- **Right**: A static booking details panel showing Reference #, Channel, Booked On, Guest, Loyalty, Vessel, Sail Date, Port, Cabin, and a price breakdown.

### Specific failure modes visible in the current screenshots

1. **Dual concurrent action cards** — both a "Modify BK-78442-mp1dzew3" card and a "Cancel BK-78440-mp1dzew3" card are visible at the same time in the center. The operator could confirm the wrong action.
2. **Context mismatch risk** — the left panel has BK-78431 selected (Grace Chen), but the modify action card targets BK-78442 and the cancel card targets BK-78440. No visual guardrail prevents execution on the wrong booking.
3. **Permanent booking list noise** — 23 unrelated bookings are always visible even during active modification or cancellation work.
4. **No customer interaction context** — the left panel shows no channel (call/email/chat), no customer need, no verification status, no previous notes, and no pending topics.
5. **Static right panel** — the right panel shows only booking and voyage data. It provides no availability check, no refund/fee impact, no rule or policy context, and no audit consequence.
6. **No active booking lock** — there is no visual treatment that identifies "this is the one booking currently staged for execution."

---

## Target Design Specification

### Information Architecture

Replace:
```
Permanent Booking List | Agent Workspace | Static Booking Details
```

With:
```
Interaction Panel | Agent Workspace | Dynamic Context / Impact / Rules Panel
```

The interaction — not the booking — is the primary anchor. A session represents one customer contact. Multiple related bookings may be referenced within a session, but only one booking is active for execution at any time.

---

### Left Panel: Interaction Panel (240px fixed width)

**Purpose**: Anchor the operator in the customer interaction context before they touch any booking.

**Contents — always visible at the top:**
- Interaction source chip: `📞 Phone Call` / `✉ Email` / `💬 Chat` / `📋 Case` / `🏢 Back-office`
- Customer name (prominent)
- Contact detail: phone number or email
- Verification status badge: `Verified ✓` (green) / `Unverified` (amber) / `Escalated` (red)
- Loyalty tier badge: Platinum / Gold / Silver / None

**Interaction summary section:**
- "Customer need" text field (editable inline, e.g. "Modify spa time, cancel mug order")
- Previous notes (collapsed summary, expandable — last 2 notes visible, "View all" link)
- Pending topics list (compact chips, e.g. "Refund query", "Seat upgrade")

**Related bookings section (compact cards, not a full list):**
- Show maximum 4–5 related bookings for this customer/session
- Each card: booking ID, status badge, key line item (e.g. "Spa · Deck 11"), total, "Set active" button
- Active booking: highlighted with a distinct border or background and an `⚡ Active` badge
- Non-active bookings: muted, still clickable to set as active

**Panel footer actions:**
- `🔍 Search bookings / customers`
- `+ New booking`
- `📝 Add note`
- `⚠ Escalate`

**What this panel must NOT contain:**
- A paginated list of all 23 bookings
- Any execution controls (those belong in the center)
- Static booking metadata (voyage details belong in the right panel)

---

### Center: Agent Workspace (fluid, takes remaining width)

**Header bar:**
- Left: Agent name `Order Agent` with a purple lightning bolt icon
- Connection status dot + label: `● Connected to OMS · Carnival UK`
- Active context pill: `⚡ BK-78431-mp1dzew3 · GRACE CHEN` — this updates when active booking changes
- Right: `New session` button

**Guardrail rule**: The active context pill in the header must always match the booking referenced in any visible action card. If they diverge, show a mismatch warning.

**Chat area:**
- AI messages: left-aligned bubble, purple icon, plain text + structured cards inline
- User/operator actions: right-aligned bubble (compact, gray)
- Structured action cards appear inline in the chat stream — not as floating overlays
- Action cards must display their target booking ID prominently at the top

**One active action card at a time**: Do not render two action cards simultaneously. If a second action is initiated while one card is open, the first card collapses to a compact "pending" state with a resume button.

**Structured action card types (see Screen States below for detail):**
- Lookup result card
- Modify booking card (with before/after fields)
- Cancel booking confirmation card
- Cancel line item card
- New booking card
- Blocked action card
- Completed action card

**Input bar (bottom):**
- Full-width text input: placeholder "Look up, modify, or cancel a booking…"
- Send button (icon)
- Suggested action chips above the bar: `Look up BK-78431-mp1dzew3` / `Modify BK-78442-mp1dzew3` / `Cancel BK-78440-mp1dzew3` / `🧖 Cancel spa` / `✨ Cancel experience` / `On hold`

**Empty / new session state:**
- Large icon + title: "Order Agent — Carnival UK"
- Subtitle: "Look up, modify, or cancel spa and experience bookings."
- 5 suggested action starter cards (icon + label + example command):
  - 🔍 Look up a booking — "Look up BK-78442-mp1dzew3"
  - ✏️ Modify a booking — "Change guest count, time slot, or cabin"
  - 🚫 Cancel a single booking — "Cancel BK-78440-mp1dzew3"
  - 🧖 Cancel by spa treatment — "Search by name — cancels only that line item"
  - ✨ Cancel by experience — "Search by name — cancels only that line item"

---

### Right Panel: Dynamic Context / Impact / Rules (280px fixed width)

This panel **changes entirely** based on the active workflow state. It never shows static booking details only.

**Tab system at the top:**
- `Context` / `Impact` / `Rules` — active tab highlights

**Per workflow, default tab content:**

| Workflow | Context tab | Impact tab | Rules tab |
|---|---|---|---|
| Lookup | Customer profile, loyalty, verification, related records | — | — |
| Modify | Current state vs. requested change, availability status | Price diff, service charge change | Policy checks (eligibility, cutoff date) |
| Cancel | Cancellation scope, what is being removed | Refund amount, fee breakdown, net payment | Cancellation policy, fee schedule |
| New booking | Availability slots, basket summary | Pricing, payment status | Required fields, eligibility |
| Blocked | Why it is blocked (rule text) | Alternatives with availability | Escalation route |
| Completed | Confirmation reference, email sent status | Final financials | Audit event summary, note status |

**Booking info section (collapsed by default, expandable):**
- Reference #, Channel, Booked On, Guest, Loyalty
- Voyage: Vessel, Sail Date, Port, Cabin
- Total: Services, Service Charge, Booking Total

This section should NOT be the dominant content of the panel.

---

## Required Screen States

Build all 8 states. Use a state switcher (a small dev toolbar or tab row) so the prototype can demonstrate each state without requiring chat input.

### State 1 — Empty / New Interaction
- Left panel: Interaction source selector (phone/email/chat/case/back-office), empty customer name field, no related bookings, no notes
- Center: Empty state with 5 action starter cards
- Right panel: Empty — shows "No active booking" placeholder

### State 2 — Active Call, Customer Identified
- Left panel: `📞 Phone Call` chip, Grace Chen, +44 7700 900123, `Verified ✓`, Platinum badge, customer need: "Modify spa time, cancel mug order", 2 previous notes (collapsed), pending topic: "Refund query", 3 related bookings (BK-78431 as Active ⚡, BK-78440, BK-78442), footer actions visible
- Center: Agent greeting message: "I found **Grace Chen** on Carnival Venezia sailing 10 Jun 2026. She has 3 related bookings. BK-78431-mp1dzew3 is set as active. What would you like to do?" — then the 5 action starters
- Right panel: Context tab — customer profile card (Grace Chen, Platinum, eCommerce, Booked 06 May 2026), loyalty benefits summary, previous interactions count

### State 3 — Booking Lookup Result
- Center: Agent message "I found **BK-78431-mp1dzew3**. Here are the details:" followed by a lookup result card showing booking metadata, line items (with service type tags), status, cabin, venue, total, and two action buttons: `Modify this booking` / `Cancel this booking`
- Right panel: Context tab — full booking and voyage details, line item breakdown

### State 4 — Modify Booking Draft (Before / After Preview)
- Center: Agent message "What would you like to change on **BK-78431-mp1dzew3**?" followed by the Modify action card:
  - Header: `✏️ MODIFY BOOKING BK-78431-MP1DZEW3` with booking clearly labeled
  - Cabin field (editable)
  - Venue field (editable)
  - Line items section: each item shows date picker, time picker, guest count stepper
  - `Preview changes` button (primary) / `Cancel` button (secondary)
  - Before/after diff visible in the card or triggered by Preview
- Right panel: Impact tab — "Before" vs. "After" state side by side, availability check result (`✓ Available` / `✗ Sold out`), price difference line, service charge recalculation

### State 5 — Cancel Booking Confirmation
- Center: Agent message "I found **BK-78440-mp1dzew3**. Please confirm cancellation:" followed by the Cancel action card in red-tinted container:
  - Header: `⚠ CONFIRM CANCELLATION` with booking ID
  - Table: Booking, Guest, Line items being cancelled, Sail Date
  - Refund summary visible (or "No refund applicable" if applicable)
  - Two buttons: `Confirm cancellation` (destructive red) / `Cancel` (secondary)
- Right panel: Impact tab — refund breakdown (services: £77.96, fee: £0.00, net refund: £77.96), Rules tab — cancellation policy text, cutoff date check

### State 6 — Blocked Action (Sold-Out Slot)
- Center: Agent message "⚠ The requested spa time slot on 11 Jun 2026 at 14:00 is **sold out**." followed by a Blocked action card:
  - Header: `🚫 ACTION BLOCKED — AVAILABILITY`
  - Reason: "Color Crush Massage · 11 Jun 2026 · 14:00 · No availability"
  - Alternative slots table: 09:00 (Available), 10:00 (Available), 11:00 (Wait list), 14:00 (Sold out ✗)
  - Action buttons: `Select 09:00` / `Select 10:00` / `Escalate to supervisor`
- Right panel: Rules tab — availability policy, wait list policy, escalation criteria

### State 7 — New Booking Flow (No Booking ID)
- Left panel: Customer identified (Grace Chen), but no related booking is active — active badge shows "No active booking"
- Center: Agent message "Starting a new booking for Grace Chen. Let's check availability." followed by the New Booking card:
  - Vessel/voyage pre-filled from customer profile
  - Service type selector (Spa / Experience / Retail)
  - Category and item search
  - Date and time availability grid
  - Guest count stepper
  - `Add to basket` button
  - Basket summary section
  - `Proceed to payment` button
- Right panel: Context tab — availability calendar grid, Impact tab — basket total, Rules tab — eligibility check (loyalty tier access, sail date requirement)

### State 8 — Completed Action (Post-Execution)
- Center: Agent success message "✓ **BK-78440-mp1dzew3 has been cancelled.** Here is the confirmation:" followed by Completed action card:
  - Confirmation reference number
  - Action summary: what was cancelled, for whom, when executed
  - Follow-up row: `✉ Confirmation email sent to grace.chen@email.com` (green check)
  - Note row: `📝 Note added to booking record` (green check)
  - Audit row: `🗂 Audit event written · 21 May 2026 14:32` (green check)
  - `Customer-facing summary` (collapsed, expandable): ready-to-read text the operator can read to the customer
  - `Start new action` button
- Right panel: Impact tab — final refund/charge summary, Rules tab — audit event record

---

## Tech Stack

- Single `.html` file — no build pipeline, no framework dependencies
- Vanilla HTML5 + CSS3 + vanilla JavaScript
- Use CSS custom properties for the design token layer (colors, spacing, radius)
- No external CSS frameworks
- Minimal external dependencies: use a CDN-loaded icon set only (e.g. Lucide icons via unpkg CDN)
- All state switching must work in the browser with no server

---

## Sample Data (Use Exactly)

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
    services: 22.99,
    serviceCharge: 0.00,
    total: 22.99,
    active: true
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
      { type: "Spa", name: "Color Crush 20oz Stainless Steel Logo Handled Mug with Rubber Bottom - Cream", date: "11 Jun 2026", time: "10:00", guests: 1, price: 0.00 },
      { type: "Spa", name: "A Giant Cup of I'm Fucking Fabulous - Oversized Mug", date: "12 Jun 2026", time: "10:00", guests: 3, price: 0.00 }
    ],
    total: 70.96,
    active: false
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
    total: 77.96,
    active: false
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

## Design Token Reference

```css
:root {
  /* Brand */
  --color-primary: #6B3FA0;         /* Carnival purple */
  --color-primary-light: #EDE9F7;
  --color-primary-dark: #4A2B72;

  /* Semantic */
  --color-success: #1A7F4B;
  --color-success-bg: #ECFDF5;
  --color-warning: #B45309;
  --color-warning-bg: #FFFBEB;
  --color-danger: #C0392B;
  --color-danger-bg: #FEF2F0;
  --color-blocked: #7C3AED;
  --color-blocked-bg: #F5F3FF;

  /* Neutral */
  --color-surface: #FFFFFF;
  --color-surface-2: #F8F8FA;
  --color-border: #E4E4E7;
  --color-text: #18181B;
  --color-text-muted: #71717A;
  --color-text-faint: #A1A1AA;

  /* Status badges */
  --badge-confirmed: #1A7F4B;
  --badge-confirmed-bg: #ECFDF5;
  --badge-cancelled: #C0392B;
  --badge-cancelled-bg: #FEF2F0;
  --badge-on-hold: #B45309;
  --badge-on-hold-bg: #FFFBEB;

  /* Layout */
  --panel-left-width: 240px;
  --panel-right-width: 280px;
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Typography */
  --font-base: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  --text-xs: 11px;
  --text-sm: 12px;
  --text-base: 13px;
  --text-md: 14px;
  --text-lg: 16px;
}
```

---

## Component Checklist

Build and wire up at minimum:

- [ ] `InteractionPanel` — left panel with customer context, verification, notes, related bookings, footer actions
- [ ] `RelatedBookingCard` — compact card with Set Active / Active badge
- [ ] `ActiveBookingPill` — persistent pill in center header showing active booking + customer
- [ ] `AgentWorkspace` — center panel with header, chat area, input bar, chips
- [ ] `ActionCard` — container for any structured in-chat action (lookup / modify / cancel / blocked / completed)
- [ ] `ModifyBookingCard` — cabin, venue, line item fields with before/after diff
- [ ] `CancelConfirmCard` — red-tinted, shows scope, refund, confirm/cancel buttons
- [ ] `BlockedActionCard` — reason, alternative slots table, escalate button
- [ ] `NewBookingCard` — service selector, availability grid, basket, payment step
- [ ] `CompletedActionCard` — confirmation ref, email/note/audit status row, customer summary
- [ ] `DynamicContextPanel` — right panel with tab switcher (Context / Impact / Rules) and per-workflow content
- [ ] `StateDevToolbar` — small top bar to switch between the 8 screen states (dev use only, can be styled as a tab strip)

---

## Critical Guardrails (Enforce in Code)

1. **One active booking at a time** — setting a booking as active in the Interaction Panel updates `SESSION.activeBookingId` and immediately updates the center header pill and all action card headers. No execution card may reference a booking other than the active one.
2. **No concurrent action cards** — if a new action is triggered while an existing card is open, collapse the old card before rendering the new one.
3. **Financial impact must be visible before the confirm button is enabled** — on Cancel and Modify cards, the confirm button is disabled until the operator has scrolled to or clicked "Preview changes."
4. **Blocked state must show alternatives** — the system must never show only "cannot proceed" without at least one alternative or escalation path.
5. **Confirmation button on cancel must be visually distinct** — use `--color-danger` background, and require a keyboard-accessible confirmation step (e.g., button text changes to "Click again to confirm" on first press).

---

## Accessibility Requirements

- All form inputs: `<label for>` + `aria-describedby` for validation messages
- Status badges: `role="status"` or `aria-label` — never color-only
- Action card confirm buttons: `aria-label` includes booking ID (e.g. "Confirm cancellation of BK-78440-mp1dzew3")
- Focus trap inside confirmation modal/card when open
- Keyboard: Tab order follows visual order left → center → right
- Visible `:focus-visible` ring on all interactive elements (2px `--color-primary` outline)
- Error, warning, success, blocked states all have text labels in addition to color/icon
- No critical text truncated — use `text-overflow: ellipsis` only on secondary IDs, never on prices, names, or policy text

---

## Output

Deliver a single `.html` file named `call-center-oms-agent.html`.

The file should:
- Open directly in Chrome/Firefox with no server
- Show all 8 states via the state switcher toolbar
- Use the sample data above
- Apply the design tokens from the CSS reference
- Pass a basic WCAG AA color contrast check on all text
- Work at 1440px viewport width minimum

Do not scaffold, do not leave placeholder sections. Deliver the full working prototype.

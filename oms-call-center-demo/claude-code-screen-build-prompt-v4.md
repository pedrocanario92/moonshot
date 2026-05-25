# Claude Code Prompt: Carnival UK Call Center OMS Agent — Prototype v4

---

## Your Task

Build a high-fidelity, interactive, single-file HTML/CSS/JS prototype for the Carnival UK Call Center OMS Order Agent.

This is a redesign. Do not replicate the existing layout (a permanent booking list + agent chat + static booking details). The problems with the existing design are documented below; use them to understand what to solve, not what to build.

---

## Existing UI Problems (Context Only — Do Not Replicate)

1. No active call indicator, timer, or call controls.
2. Permanent booking list visible at all times — creates noise during active work.
3. No customer interaction context — no channel, verification, customer need, notes, or pending topics.
4. Two action cards visible simultaneously — no guardrail against acting on the wrong booking.
5. Static right panel — always visible, always the same content.
6. No active booking lock — no clear execution anchor.

---

## Visual Style Direction

All panels are **floating cards** with rounded corners and drop shadows. They do not fill the entire viewport edge-to-edge. There is a visible gap between panels and the screen edges.

```css
/* Apply to every panel */
border-radius: 12px;
box-shadow: 0 2px 8px rgba(0,0,0,0.08), 0 0 1px rgba(0,0,0,0.1);
background: var(--color-surface);
overflow: hidden;
```

The overall page background is `--color-bg-page: #F0EFF4` (a soft off-white/lavender-tinted gray). Panels float on top of this background with visible gaps — approximately 12px gap from the viewport edges and 12px between panels.

Do **not** use full-viewport-height flush panels. Do **not** use a white page background. Panels should look like elevated cards.

---

## Layout Structure

```
┌────────────────────────────────────────────────────────────────────────────────┐
│  Infios top navigation bar (full width, dark — per brand)                      │
├────────────────────────────────────────────────────────────────────────────────┤
│  STATE SWITCHER DEV TOOLBAR                                                     │
├──────────────────────────┬─────────────────────────────────────────────────────┤
│  LEFT PANEL (card)       │  CENTER AREA                                        │
│  320px, floating card    │                                                     │
│                          │  ┌─────────────────────────────────────────────┐   │
│  Tab strip at top:       │  │  FLOATING CALL BAR (dark pill)              │   │
│  On Going | Bookings |   │  │  Animates in/out. Only shown when active    │   │
│  Call                    │  │  phone call. Anchored above the chat panel. │   │
│                          │  └─────────────────────────────────────────────┘   │
│  Tab content changes     │  ┌─────────────────────────────────────────────┐   │
│  per active tab          │  │  AGENT WORKSPACE (floating card)            │   │
│                          │  │  Header + chat area + input bar             │   │
│                          │  │  Fluid width: fills remaining space         │   │
│                          │  └─────────────────────────────────────────────┘   │
│                          │                                                     │
│                          │               RIGHT PANEL (floating card, 320px)   │
│                          │               Hidden by default. Slides in from    │
│                          │               right edge on booking card click.    │
│                          │               Overlaps the chat panel.             │
└──────────────────────────┴─────────────────────────────────────────────────────┘
```

Measurements:
- Left panel: `320px` fixed width
- Right panel: `320px`, `position: fixed`, right edge, hidden by default (`transform: translateX(320px)`), slides in on trigger
- Center area: `calc(100vw - 320px - 24px - 48px)` (accounts for left panel width + gaps + page nav)
- Gap between all panels: `12px`

---

## Left Panel: Three-Tab Interaction Panel

The left panel always shows a tab strip at the top with three tabs.

### Tab 1 — On Going

This tab is the active default when a call is in progress or a customer is identified. It contains the operator's current interaction context.

**Contents (top to bottom):**

**Customer Identity block:**
- Customer name (large, bold)
- Contact detail (phone number or email)
- Verification badge: `✓ Verified` (green) / `⚠ Unverified` (amber) / `🔴 Escalated` (red) — text + color
- Loyalty tier badge: `Platinum` (purple) / `Gold` (amber) / `Silver` (gray)
- If call is on hold: `[ON HOLD]` amber pulsing badge next to name

**Customer Need:**
- Label: `CUSTOMER NEED`
- Editable text area, 2 rows, max 120 chars, character count shown

**Previous Notes:**
- Label: `PREVIOUS NOTES`
- Last 2 notes as compact cards: date + 60 chars + `…`
- `View all` link

**Pending Topics:**
- Label: `PENDING TOPICS`
- Topics as chips (e.g. `Refund query`)
- `+ Add topic`

**Related Bookings:**
- Label: `RELATED BOOKINGS`
- Max 5 compact booking cards: booking ID, status badge, guest name, total
- Active booking: highlighted border (`--color-primary`), `⚡ Active` badge
- Non-active: muted, `Set active` button on hover/focus
- `Set active` updates `SESSION.activeBookingId`, center header pill, and quick actions

**Quick Actions:**
- Label: `QUICK ACTIONS`
- 2–4 buttons pre-populated from `SESSION` data:
  - `✏ Modify BK-78431` → fires command into center chat
  - `🚫 Cancel BK-78440` → fires command into center chat
  - `🔍 Look up BK-78442` → fires command into center chat
- Buttons update when active booking changes

**Footer actions (pinned to bottom of panel):**
- `🔍 Search` / `+ New booking` / `📝 Add note` / `⚠ Escalate`

---

### Tab 2 — Bookings

This tab shows the **full operator booking queue** — not scoped to the current customer. This is the equivalent of the original 23-booking list, available when the operator needs to browse, search, or switch context between calls.

**Contents:**

**Filter chips row:**
- `All` / `Confirmed` / `On Hold` / `Cancelled` / `Spa` / `Exp`

**Search input:**
- Placeholder: `Search guest, booking ID, ship…`

**Booking list:**
- Each row: booking ID (bold), status badge, customer name, vessel + deck, total price
- Active booking highlighted with a left border in `--color-primary`
- Clicking a row: sets it as the active booking (prompts confirmation if a different booking is currently active and an action is in progress)
- Count: `1–20 of 23` with prev/next pagination controls

**Usage note for the prototype:**
- Populate with all 3 sample bookings plus 5–7 additional synthetic rows to demonstrate the list style. Use realistic Carnival UK names and amounts.

---

### Tab 3 — Call

This tab shows **call history** and, when a call is active, a live summary of the current call.

**Contents when a call is active:**

**Current call block (top):**
- Status: `● Active Call` (green)
- Customer: Grace Chen
- Duration: mirrors the floating call bar timer
- Channel: Phone
- Summary field: auto-populated by agent with key points as the call progresses (editable by operator)
  - Example: "Customer requesting spa time change and mug cancellation. Refund query pending."
- `Add call note` button

**Call history list:**
- Each entry: date, customer name, duration, outcome tag (`Resolved` / `Escalated` / `Callback`)
- Last 5 calls shown, `View all` link

**Contents when no call is active:**
- `No active call` placeholder at top
- Call history list below

---

## Floating Call Bar (Center Area, Above Chat)

This is a **separate floating component**, not part of the left panel. It appears above the agent workspace card in the center area.

### Trigger

- Shown only when `SESSION.channel === "phone"` AND `STATE >= 2` (call is active)
- Hidden entirely (no space reserved, no placeholder) when no call is active
- Animates in: `opacity 0 → 1` + `translateY(-8px) → translateY(0)` over 200ms when call starts
- Animates out: reverse, when call ends

### Visual Design

Based on image reference: a dark, rounded pill-shaped bar. Full width of the center area it sits above.

```css
background: #1C1C1E;           /* near-black */
border-radius: 12px;
padding: 10px 16px;
display: flex;
align-items: center;
gap: 16px;
color: #FFFFFF;
box-shadow: 0 4px 16px rgba(0,0,0,0.24);
margin-bottom: 8px;            /* gap between bar and chat card */
```

### Contents (left to right)

1. **Timer**: `00:40` format, monospace font, green (`#34C759`), counting up from 00:00. Pauses when on hold.
2. **Hold button**: icon + label `Hold`. On active: amber background, label `Resume`. Pauses timer.
3. **Mute button**: icon + label `Mute`. On active: red/danger tint, label `Unmute`. Visual indicator only in prototype.
4. **Keypad button**: icon + label `Keypad`. Click shows a 3×4 DTMF keypad modal (functional number buttons, no audio needed).
5. **Contacts button**: icon + label `Contacts`. Click shows a small dropdown with 2–3 example contacts.
6. **End call button**: red circle button, phone icon (✕). Click shows confirmation: "End this call? / Yes — End / Cancel". On confirm: bar animates out, Call tab shows call summary, left panel On Going tab shows idle state, timer resets.

### Accessibility
- `aria-label` on all buttons
- `aria-pressed` on Hold and Mute toggles
- Timer: `aria-live="off"` (avoid screen reader noise on every second tick); call status label uses `aria-live="polite"`

---

## Center: Agent Workspace (Floating Card)

### Header Bar

- Left: Lightning bolt icon (purple) + `Order Agent` + `● Connected to OMS · Carnival UK`
- Center: Active context pill — `⚡ BK-78431-MP1DZEW3 · GRACE CHEN` — updates on `Set active`; shows `No active booking` if none
- Right: `New session` button

**Guardrail**: If an action card targets a booking other than `SESSION.activeBookingId`, show inline warning: `⚠ This action targets BK-XXXXX — not the active booking. Set it active first.`

### Chat Area

- AI messages: left-aligned, purple lightning icon
- Operator messages: right-aligned, gray bubble
- Structured action cards render **inline in the chat stream**
- **One action card open at a time** — second action collapses the first to a `↩ Resume [action]` bar
- Booking cards in chat have a `→ Click to edit` hover affordance; clicking opens the right panel slide-in

### Input Bar (pinned to card bottom)

- Full-width input: placeholder `Ask the agent or type an action…`
- Send button
- Context-sensitive chips above bar (max 4):
  - Default (on going call): `Modify BK-78431` / `Cancel BK-78440` / `View all bookings`
  - During modify: `Preview changes` / `Reset`
  - During cancel: `Confirm` / `Keep booking`
  - Post-action: `Start new action` / `Send email`

---

## Right Panel: On-Demand Slide-In (Unchanged From v3)

Hidden by default. Slides in (`translateX(320px) → translateX(0)`, 240ms ease) when a booking card is clicked in the chat.

**Header:** Booking ID + status badge + `✕ Close`

**Modify form:** Cabin, venue, line items (date picker, time picker, guest stepper), before/after diff, availability indicator, `Preview changes` (disabled until field changed) + `Confirm` (disabled until preview triggered).

**Cancel form:** Read-only summary, refund breakdown, two-press confirm button (first press: label → `Click again to confirm ▸`, second press within 4 seconds: executes).

**Blocked form:** Reason, alternatives table with `Select [time]` buttons, `Escalate` button.

Apply the same floating card style (border-radius 12px, box-shadow) to the right panel.

---

## Required Screen States

Dev toolbar at the top cycles through all 8 states.

| # | Label | Call bar | Left panel tab | Right panel |
|---|---|---|---|---|
| 1 | Empty | Hidden | On Going (empty) | Hidden |
| 2 | Call Active | Visible, timer running | On Going (Grace Chen loaded) | Hidden |
| 3 | Lookup | Visible | On Going | Hidden |
| 4 | Modify | Visible | On Going | Open — Modify form |
| 5 | Cancel | Visible | On Going | Open — Cancel form |
| 6 | Blocked | Visible | On Going | Open — Blocked form (auto-opens) |
| 7 | New Booking | Hidden (email channel) | On Going (no booking active) | Hidden |
| 8 | Completed | Visible (call still active) | Call tab (summary populated) | Hidden |

### State 1 — Empty / New Interaction
- Call bar: hidden
- Left On Going: channel selector chips (Phone / Email / Chat / Case / Back-office), empty customer fields, no notes, no bookings, no quick actions
- Center: empty state — icon + "Order Agent — Carnival UK" + 5 generic action starters
- Right: hidden

### State 2 — Active Call, Customer Identified
- Call bar: visible, timer counting up from 00:00, all controls active
- Left On Going: Grace Chen, +44 7700 900123, Verified ✓, Platinum; customer need "Modify spa time, cancel mug order"; 2 previous notes; "Refund query" pending topic; 3 related bookings (BK-78431 active ⚡, BK-78442 and BK-78440 inactive); quick actions: Modify BK-78431 / Cancel BK-78440 / Look up BK-78442
- Center: agent greeting: "I found **Grace Chen** on Carnival Venezia sailing 10 Jun 2026. She has 3 related bookings. **BK-78431-mp1dzew3** is active. What would you like to do?" + 5 action starters
- Right: hidden

### State 3 — Booking Lookup Result
- Center: agent message + lookup result card for BK-78431 with `→ Click to edit` affordance
- Right: hidden

### State 4 — Modify Booking
- Center: agent message + collapsed "Modifying BK-78431…" summary bar in chat
- Right: slide-in open — modify form for BK-78431, pre-filled, before/after diff visible

### State 5 — Cancel Booking
- Center: agent message + collapsed cancel summary bar in chat
- Right: slide-in open — cancel form for BK-78440, refund shown, two-press confirm

### State 6 — Blocked Action
- Center: agent message: "⚠ The requested spa time on 11 Jun 2026 at 14:00 is **sold out**."
- Right: auto-opens — blocked form: reason + alternatives table + Escalate button

### State 7 — New Booking (Email Channel)
- Call bar: hidden (channel = email)
- Left On Going: Grace Chen, no active booking
- Center: new booking card — service type selector, availability grid, basket, payment step
- Right: hidden

### State 8 — Completed Action
- Call bar: still visible (call ongoing)
- Left: Call tab auto-selected; current call summary populated by agent
- Center: success card — confirmation reference, email sent ✓, note added ✓, audit ✓, customer-facing summary (collapsible), `Start new action` button
- Right: hidden (closed after completion)

---

## Tech Stack

- Single `.html` file, no build step, no framework
- Vanilla HTML5 + CSS3 + vanilla JS
- CSS custom properties for all tokens
- Lucide icons via CDN: `https://unpkg.com/lucide@latest`
- No external CSS frameworks

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
    id: "BK-78431-mp1dzew3", ref: "c9fededa-d556-4b6c-a8d5-0bf115dd4f9c",
    status: "CONFIRMED", channel: "eCommerce", bookedOn: "06 May 2026",
    guest: "Grace Chen", loyalty: "Platinum", vessel: "Carnival Venezia",
    sailDate: "10 Jun 2026", port: "Southampton", cabin: "Cabin 5050", deck: "Lido Deck",
    lineItems: [{ type: "Spa", name: "Oversized Mug Package", date: "12 Jun 2026", time: "10:00", guests: 3, price: 22.99 }],
    services: 22.99, serviceCharge: 0.00, total: 22.99
  },
  {
    id: "BK-78442-mp1dzew3", status: "CONFIRMED", guest: "Natalie Harris",
    deck: "Deck 11", vessel: "Carnival Venezia", sailDate: "10 Jun 2026",
    cabin: "Cabin 5030", venue: "Deck 11 — Havana Bar",
    lineItems: [
      { type: "Spa", name: "Color Crush 20oz Mug — Cream", date: "11 Jun 2026", time: "10:00", guests: 1, price: 0.00 },
      { type: "Spa", name: "Oversized Mug Package", date: "12 Jun 2026", time: "10:00", guests: 3, price: 0.00 }
    ], total: 70.96
  },
  {
    id: "BK-78440-mp1dzew3", status: "CONFIRMED", guest: "Scarlett White · White Group",
    deck: "Deck 9", vessel: "Carnival Venezia", sailDate: "10 Jun 2026",
    lineItems: [{ type: "Spa", name: "Havana Bar Package", date: "10 Jun 2026", time: "14:00", guests: 2, price: 77.96 }],
    total: 77.96
  }
];

const CALL_HISTORY = [
  { date: "03 May 2026", customer: "Grace Chen", duration: "4:22", outcome: "Resolved" },
  { date: "14 Apr 2026", customer: "Natalie Harris", duration: "7:01", outcome: "Escalated" },
  { date: "02 Apr 2026", customer: "Priya Patel", duration: "2:45", outcome: "Resolved" },
  { date: "28 Mar 2026", customer: "Oliver Clarke", duration: "9:15", outcome: "Callback" },
  { date: "15 Mar 2026", customer: "Sarah Thompson", duration: "3:30", outcome: "Resolved" }
];

const BLOCKED_SLOT = {
  item: "Color Crush Massage", date: "11 Jun 2026", requestedTime: "14:00",
  alternatives: [
    { time: "09:00", available: true }, { time: "10:00", available: true },
    { time: "11:00", waitlist: true }, { time: "14:00", available: false }
  ]
};
```

---

## Design Tokens

```css
:root {
  /* Page */
  --color-bg-page: #F0EFF4;

  /* Brand */
  --color-primary: #6B3FA0;
  --color-primary-light: #EDE9F7;
  --color-primary-dark: #4A2B72;

  /* Semantic */
  --color-success: #1A7F4B;
  --color-success-bg: #ECFDF5;
  --color-warning: #B45309;
  --color-warning-bg: #FFFBEB;
  --color-danger: #C0392B;
  --color-danger-bg: #FEF2F0;

  /* Surfaces */
  --color-surface: #FFFFFF;
  --color-surface-2: #F8F8FA;
  --color-border: #E4E4E7;

  /* Text */
  --color-text: #18181B;
  --color-text-muted: #71717A;
  --color-text-faint: #A1A1AA;

  /* Call bar */
  --color-call-bar-bg: #1C1C1E;
  --color-call-timer: #34C759;

  /* Panels */
  --panel-left: 320px;
  --panel-right: 320px;
  --panel-radius: 12px;
  --panel-shadow: 0 2px 8px rgba(0,0,0,0.08), 0 0 1px rgba(0,0,0,0.10);
  --panel-gap: 12px;

  /* Type */
  --font-base: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  --font-mono: "SF Mono", "Fira Code", monospace;
}
```

---

## Component Checklist

- [ ] `PageLayout` — `--color-bg-page` background, flex row, `--panel-gap` gaps, panels as floating cards
- [ ] `LeftPanel` — 320px card, tab strip (On Going / Bookings / Call), tab content switcher
- [ ] `OnGoingTab` — customer identity, verification, loyalty, need, notes, topics, related bookings, quick actions, footer
- [ ] `BookingsTab` — filter chips, search input, full booking list with pagination
- [ ] `CallTab` — active call summary block (mirrors timer), call history list
- [ ] `RelatedBookingCard` — compact, active state + Set active
- [ ] `QuickActionButtons` — pre-populated from SESSION, fire into chat
- [ ] `FloatingCallBar` — dark pill, timer (setInterval), Hold toggle, Mute toggle, Keypad (DTMF modal), Contacts dropdown, End call confirmation; animates in/out with call state
- [ ] `AgentWorkspace` — floating card, header with active context pill, chat area, input bar, chips
- [ ] `ActiveBookingPill` — updates on Set active; mismatch warning if action card targets different booking
- [ ] `ActionCard` — base container, one-at-a-time rule, booking ID in header
- [ ] `RightPanel` — fixed position, slide-in on booking card click, floating card style, ✕ close
- [ ] `ModifyForm` — fields, before/after diff, availability, staged confirm
- [ ] `CancelForm` — read-only summary, refund, two-press confirm
- [ ] `BlockedForm` — reason, alternatives table, escalate
- [ ] `NewBookingCard` — service selector, availability, basket
- [ ] `CompletedCard` — reference, status rows, customer summary
- [ ] `StateDevToolbar` — tab strip for 8 states

---

## Critical Guardrails (Enforce in JS)

1. **One active booking** — `SESSION.activeBookingId` is the single source. All action cards, the header pill, and quick action buttons derive from it.
2. **One action card at a time** — collapse current card before opening a new one.
3. **Call bar only on phone + active call** — do not render it for email, chat, case, or back-office states.
4. **Call timer** — starts on State 2, pauses on Hold, resets on End call. The Call tab's summary duration mirrors this timer.
5. **Confirm disabled until preview** — Modify form confirm button is `disabled` until Preview has been triggered.
6. **Two-press cancel** — first press: label → `Click again to confirm ▸` (danger style); second press within 4s: executes; after 4s: resets.
7. **Mismatch warning** — action card targeting a non-active booking shows inline `⚠ Set as active first` warning in the card header.
8. **Bookings tab switching** — if operator switches to Bookings tab and clicks a booking while an action is in progress, show confirmation: `"Switch active booking to [ID]? This will collapse the current action."` before executing.

---

## Accessibility Requirements

- All form inputs: `<label for>` + `aria-describedby`
- Status badges and chips: `aria-label`, never color-only
- Hold / Mute: `aria-pressed`
- Right panel: focus trap when open, `Escape` closes (with unsaved-change guard)
- Active context pill: `aria-live="polite"` — announced when booking changes
- Call bar timer: `aria-live="off"` (avoid per-second announcement)
- `:focus-visible` ring (2px `--color-primary`) on all interactive elements
- Error / warning / success / blocked states: text labels required, not icon/color alone

---

## Output

Deliver a single file: `call-center-oms-agent.html`

Requirements:
- Opens in Chrome/Firefox, no server
- All 8 states reachable via the dev toolbar
- Floating call bar visible + timer functional in States 2–8 (phone channel); hidden in State 7 (email)
- Hold, Mute, End call controls functional with correct visual state changes
- Left panel tab switching functional (On Going / Bookings / Call)
- Bookings tab shows list with filter chips
- Call tab shows history + live summary when call is active
- Right panel slide-in functional on booking card click
- Set active in related bookings updates header pill + quick actions
- Floating card visual style applied to all panels
- Design tokens applied throughout
- No placeholder sections — full working prototype

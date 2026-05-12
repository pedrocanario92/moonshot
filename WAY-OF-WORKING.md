# Way of Working — operating rules and ship discipline

Operational reference. For project intent, the agent roster, and the file tree see `README.md`. For the deep architecture see `AGENT-ARCHITECTURE.md`. For session-by-session history see `ITERATION-LOG.md`.

---

## Files

### Active

| File | Purpose |
|---|---|
| `README.md` | Single landing page. Start here. File tree, agent roster summary, working style, briefing block for new Claude sessions. |
| `AGENT-ARCHITECTURE.md` | Deep architecture document — source of truth for the agent system. Design principles, agent layer decisions, approval and escalation logic, product surfaces, key metrics, risks, design system tokens. |
| `WAY-OF-WORKING.md` | This file. Operating rules and ship discipline. |
| `ITERATION-LOG.md` | Session-by-session history. Append a new entry after each work block. |
| `tasks-*.md` *(at root)* | Current task list(s). Move to `task archive/` once verified complete. |
| `moonshot presentation/moonshot.html` | Primary deliverable. Strategy document + agent showcase + embedded prototype viewer. Open this to see everything. |
| `moonshot presentation/moonshot-prototype.html` | The shipped WMS prototype. Loads inside moonshot.html's Prototype tab via iframe `src`. Must be in the same folder as moonshot.html — that's why they live together in `moonshot presentation/`. |
| `moonshot prototype/moonshot-prototype.html` | Working copy of the prototype. This is where ongoing prototype work happens. Edit this one; leave the file inside `moonshot presentation/` untouched until a change is ready to ship. |
| `admin-panel.html` | Secondary prototype — document library and knowledge base admin panel. Standalone. |

### Archived

| File / folder | Why archived |
|---|---|
| `archive/WMS-AGENT-CLI-PROTOTYPE.md` | Build spec for the earlier prototype. Useful if extending — not needed to understand moonshot.html. |
| `archive/CONTEXT.md` | Project context for the chatbot UI + admin panel (Warehouse Advantage / K.One). Separate project. |
| `archive/legacy html/` | Earlier HTML iterations preserved for lineage. |
| `archive/agents/` | Pre-May-12 agent spec snapshots, kept for diff against current `/agents/`. |
| `moonshot presentation/archive/` | Previously-shipped artifacts (e.g., `wms-unified-figma-sidebar.PRE-MAY-12.html`). Preserved for rollback. |
| `task archive/` | Completed task lists (e.g., `MAY-12-PLAN.md`). Move tasks here when they're done. |

### Subdirectories

| Directory | Contents |
|---|---|
| `moonshot presentation/` | The finished, ship-ready pair: `moonshot.html` + `moonshot-prototype.html`. Both must stay co-located here so the iframe inside `moonshot.html` resolves. |
| `moonshot prototype/` | Working copy of the prototype as `moonshot-prototype.html`. Runs by double-clicking, no dependencies on parent folders. |
| `agents/` | 14 markdown files — one per agent. Each defines what the agent does, what it never does, what it can do autonomously, and what requires human approval. Sub-folders: `agents/meta/` (Warehouse Life), `agents/guardian/` (Heuristics, Accessibility). |
| `inspiration/` | Reference screenshots and raw concept images used during design. |
| `original references/` | Allen Oleksak's WMS demo (lineage) and the initial Claude prompt. |
| `prompt prototype/` | The one-shot prompt experiment — Form A (control) and Form B (real MVP). Runs in a separate fresh chat. |
| `task archive/` | Completed task lists. |
| `archive/` | All earlier HTML iterations, planning files, and token deliveries. Not needed to run the demo. |
| `.claude/skills/` | Project-scoped Claude Code skills (e.g., `design-tokens.md` — auto-triggered token integration). |

### To share

Zip the contents of `moonshot presentation/` (both `moonshot.html` and `moonshot-prototype.html`). Recipient unzips and opens `moonshot.html`. Requires internet for Google Fonts — no other external dependencies. Never share the in-progress copy from `moonshot prototype/`.

> Never share HTML files as bare attachments in Teams or Outlook. Microsoft Defender flags standalone HTML files with embedded scripts. Always send as a zip.

---

## How moonshot.html is built

Single-file HTML. No framework, no build step. Vanilla HTML, CSS custom properties (Infios design tokens, governed by the `design-tokens` skill), and vanilla JavaScript for view switching.

Three tabs in the top nav: **Home · The Agents · The Prototype**.

### Agents page structure

The Agents page has a sticky in-page section index that highlights the active section as you scroll. Sections in order:

1. **Architecture** — Apple-style Timeline · System toggle. Timeline view renders a zigzag conveyor of seven UX stations from prompt to delivered prototype. System view renders a CSS Grid hierarchy tree of the operational chain plus lateral Analytical, Guardian, and Meta layers.
2. **Lead** (Orchestration)
3. **Radar** (Shift Intelligence)
4. **Domain experts** (8 Domain agent cards)
5. **Analytical** (3 Analytical cards)
6. **Signals** (`How the signals move.` topology block)
7. **Guardian** (6 Guardian cards)
8. **Meta** (Warehouse Life)
9. **Horizon** (10 cards across 3 thematic categories)
10. **The case** — "UX as infrastructure, not overhead" (closing value prop)

The Prototype tab uses a direct iframe `src` on `moonshot-prototype.html` (co-located in `moonshot presentation/`). This is intentional — an earlier version embedded the prototype as a Base64 blob to create a single self-contained file, but that pattern is flagged as HTML smuggling by Microsoft Defender. The two-file approach is the correct one.

---

## Extending the system

### To add a new WMS agent

1. Create a markdown spec in `/agents/` following the existing 8-section structure (Identity & Purpose · Responsibility Scope · Trigger Conditions · Actions · Inter-Agent Communication · Human Guardrails · Escalation Behaviour · Performance Metrics). Use any existing spec (e.g., `agents/pick-path-agent.md`) as the template.
2. If the agent is adapted from Allen Oleksak's demo, include an "Adapted from Allen's demo" provenance section at the top.
3. Add a card to the appropriate layer on `moonshot.html`'s Agents page using the existing `.card` + `.pill--*` pattern. Match the existing tone and length — source language from the spec; don't invent new copy.
4. If the agent isn't WMS-domain (e.g., a Guardian process gate, or a Meta backstage agent), use the appropriate sub-folder (`agents/guardian/` or `agents/meta/`) and the matching pill style (`pill--guardian` or `pill--meta`).
5. Update the topology section ("How the signals move.") to add the new agent's communication edges if applicable.
6. Update the `README.md` agent roster table and the `AGENT-ARCHITECTURE.md` architecture diagram.

### To add a new Horizon agent

Add a card to the appropriate Horizon category on `moonshot.html`'s Agents page. Each category uses `.horizon-cat` → `.grid-5` → `.card.card--horizon` with `.pill--horizon`. Horizon agents currently don't have full specs in `/agents/` — they're described at the card level only.

### To add a new Horizon category

Use `.horizon-cat` with `.horizon-cat__label`, `.horizon-cat__desc`, and `.horizon-cat__audience` before the card grid.

### To update the prototype

Edit `moonshot prototype/moonshot-prototype.html` directly. The prototype is a single-file vanilla JS state machine — no build required. When the change is ready to ship, promote it to `moonshot presentation/moonshot-prototype.html` (overwrite the file there; archive the prior version under `moonshot presentation/archive/`).

### To integrate new design tokens

The `design-tokens` skill at `.claude/skills/design-tokens.md` is auto-triggered when the user mentions a token delivery or asks to integrate / reconcile / sweep tokens. The skill handles: extracting the delivery, transcribing tokens into a versioned reference markdown at `archive/tokens-<YYYY-MM-DD>/TOKENS.md`, reconciling with the existing `:root` block in both `moonshot.html` and `moonshot-prototype.html`, and sweeping hardcoded values for token replacement.

---

## Ship discipline

- **Verify in the browser via the preview tool**, not by assertion. Run `preview_start` against `moonshot presentation/` and `preview_screenshot` every visual claim.
- **Never declare a screenshot fine if the screenshot tool timed out.** Say "couldn't capture" and use DOM / geometry checks via `preview_eval` instead.
- **Never overwrite `moonshot presentation/` files without explicit go-ahead.** That folder is the shipped artifact; treat it with the same caution as a production deploy. Archive the old file first.
- **Always run both Guardian audits** (Heuristics + Accessibility) before declaring a homepage change shippable. Their specs in `agents/guardian/` define the criteria. Any load-bearing FAIL blocks ship.
- **Always zip** before sharing the deliverable in Teams, Outlook, or any environment where Microsoft Defender or similar will flag standalone HTML with embedded scripts.
- **Never share the in-progress copy** from `moonshot prototype/` — always share from `moonshot presentation/`.

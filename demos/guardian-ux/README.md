# Guardian UX

A distributable **Claude Code plugin**: six read-only "guardian" agents that audit any UI you build and hand you a single PASS / WARN / FAIL report **before you ship**. The guardians never modify your code — they audit and recommend; **you** decide ship.

It auto-activates: when you prompt Claude Code about *reviewing, auditing, QA-ing, or shipping* a screen, the review skill kicks in and runs the suite against your UI.

## The six guardians

| Guardian | Audits | A few load-bearing checks |
|---|---|---|
| **accessibility-guardian** | WCAG 2.2 (AA load-bearing, AAA advisory) | contrast, keyboard, focus visible/order, target size, semantics, timing |
| **heuristics-guardian** | Nielsen 10 + Hick + Miller + cognitive load | above-fold clutter, choices per state, info scent, system status, error recovery |
| **design-system-guardian** | design-system / token adherence | color/spacing/type/radius tokens, component reuse, brand-locked values |
| **content-guardian** | UX writing / microcopy | actionable labels, error messages, empty states, terminology consistency |
| **interaction-guardian** | behaviour / interaction model | no dead controls, state coverage, feedback, no traps, destructive-action safety |
| **responsive-guardian** | layout across breakpoints | no overflow, reflow at 320px, touch targets, no content loss, readable at 200% zoom |

Each emits the same verdict shape (`PASS | WARN | FAIL | N/A` + one-line rationale per criterion, then a recommendation). A single **joint report** is written to `reports/guardian-audit-<target>-<date>.md` in your project.

## Install

From any Claude Code session:

```
/plugin marketplace add pedrocanario92/moonshot
/plugin install guardian-ux@infios-guardian-marketplace
```

> The marketplace manifest lives at the repo root (`.claude-plugin/marketplace.json`) and points at the `demos/guardian-ux/` subdirectory. If you split `guardian-ux/` into its own repo, update the `source` in that manifest (or add the new repo directly).

Local development / trying it before publishing:

```
/plugin marketplace add /absolute/path/to/moonshot
/plugin install guardian-ux@infios-guardian-marketplace
```

## Use

**Automatic** — just ask, once you've built or changed a screen:

> "Review this screen before I ship it."
> "Audit http://localhost:3000/checkout"
> "Is this component ready to go?"

The `guardian-review` skill fans out all six guardians and returns the joint report.

**Explicit** — run the command directly:

```
/guardian-ux:guardian-audit http://localhost:3000/dashboard
/guardian-ux:guardian-audit src/components/Checkout.tsx
```

**Single dimension** — ask for one guardian and only it runs:

> "Just check the accessibility of this page."

## How it works

- **Skill (`guardian-review`)** — the auto-triggering entry point. Its `description` + `when_to_use` match UI review/ship intent, so Claude loads it and orchestrates the audit.
- **Command (`/guardian-ux:guardian-audit`)** — the explicit manual trigger; same orchestration.
- **Agents (`*-guardian`)** — six subagents Claude can also delegate to individually. Each is `disallowedTools: Write, Edit, NotebookEdit`, so it is structurally **read-only**.
- **Evidence** — if a browser-preview tool is available, guardians inspect the rendered DOM, computed styles, and behaviour (highest fidelity). Otherwise they audit the source statically and label the report accordingly.

## Design principles (inherited from the Guardian Layer concept)

- **Read-only.** Guardians audit and emit verdicts; they never modify the product.
- **No self-certification.** The report is a proposal to the human. Only the human approves ship.
- **No silent downgrades.** A load-bearing FAIL is reported as a FAIL.
- **No evidence → N/A.** Unmeasured criteria are marked `N/A` with a reason, never guessed.
- **Surface tensions, don't resolve them.** When two criteria conflict, both findings go to the human.

## Repository layout

```
guardian-ux/
├─ .claude-plugin/plugin.json        # plugin manifest
├─ agents/                           # 6 read-only guardian subagents
├─ commands/guardian-audit.md        # /guardian-ux:guardian-audit
├─ skills/guardian-review/SKILL.md   # auto-triggering orchestrator
└─ README.md
.claude-plugin/marketplace.json      # (repo root) marketplace entry → ./demos/guardian-ux
```

## License

MIT

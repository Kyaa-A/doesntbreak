# doesntbreak

A Claude Agent Skill that makes Claude write web UI which holds up on small screens by default — mobile-first, no horizontal scroll, fluid layouts — working from 320px up, without you ever having to ask for "responsive."

## The problem

Vibe-coded UI looks great on a 1440px desktop and falls apart on a 375px phone. The usual symptoms:

- A horizontal scrollbar that shouldn't be there.
- Elements overlapping or squishing into each other.
- Text overflowing its container or getting clipped.
- Tap targets too small to reliably hit with a thumb.

The cause is almost always desktop-first habits: fixed pixel widths, hand-written breakpoints that fight each other, and layouts that were only ever checked at one width.

## What it does

The skill bakes responsive discipline into Claude's default output so you don't have to remember to ask for it.

- **Mobile-first by default.** Base styles target the smallest screen; complexity is layered upward with `min-width` queries — never the reverse.
- **Non-negotiables, applied automatically:**
  - No fixed pixel widths on layout containers — cap with `min(100%, Npx)`.
  - Let flex and grid reflow (`flex-wrap`, `auto-fit` grids) instead of hand-written breakpoints.
  - `min-height`, not fixed `height`, so containers grow with content.
  - Media capped with `max-width: 100%`.
  - Touch targets at least 44x44px.
  - At least 16px font on inputs, so iOS Safari doesn't zoom on focus.
  - Long unbroken strings broken with `overflow-wrap`, not left to overflow.
- **Tests at the real failure points.** 320px, 375px, the awkward 768px tablet middle, long content, and a hard check for no horizontal scroll at any width.
- **Review mode.** Point it at existing layout code and it scans for the usual offenders — fixed widths, missing `min-width: 0`, desktop-first `max-width` queries, fixed heights, unconstrained images — and fixes them.

## File structure

```
doesntbreak/
├── SKILL.md            # the skill instructions Claude loads
├── README.md           # this file
├── LICENSE             # MIT
└── references/
    └── patterns.md     # detailed CSS mechanics + annotated example
```

`SKILL.md` carries the rules and triggers. `references/patterns.md` holds the deeper mechanics — the flexbox `min-width: 0` trap, `clamp()` typography, auto-fit grids, container queries, safe-area insets, and a fully annotated example — which Claude reads when it needs the details.

## Install

Drop the `doesntbreak/` folder into your Claude skills directory:

- **Claude Code:** `~/.claude/skills/doesntbreak/`
- **Plugin:** your plugin's `skills/` directory.

Claude discovers it automatically via the `description` field in `SKILL.md`. No configuration needed.

## Usage

You don't invoke it manually. Just ask Claude to build or fix any web UI and it applies the rules on its own:

- "Build a pricing page."
- "This page breaks on my phone — fix it."

Claude triggers the skill whenever it writes or reviews layout code, even if you never say the word "responsive."

## License

MIT. Copyright (c) 2026 Asnari (Kyaa-A). See [LICENSE](LICENSE).

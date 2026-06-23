# doesntbreak

A Claude Agent Skill that makes Claude write web UI which holds up on small screens by default (mobile-first, no horizontal scroll, fluid layouts) working from 320px up, without you ever having to ask for "responsive."

## The problem

Vibe-coded UI looks great on a 1440px desktop and falls apart on a 375px phone. The usual symptoms:

- A horizontal scrollbar that shouldn't be there.
- Elements overlapping or squishing into each other.
- Text overflowing its container or getting clipped.
- Tap targets too small to reliably hit with a thumb.

The cause is almost always desktop-first habits: fixed pixel widths, hand-written breakpoints that fight each other, and layouts that were only ever checked at one width.

## What it does

The skill bakes responsive discipline into Claude's default output so you don't have to remember to ask for it.

- **Mobile-first by default.** Base styles target the smallest screen; complexity is layered upward with `min-width` queries. Never the reverse.
- **Non-negotiables, applied automatically:**
  - No fixed pixel widths on layout containers: cap with `min(100%, Npx)`.
  - Let flex and grid reflow (`flex-wrap`, `auto-fit` grids) instead of hand-written breakpoints.
  - `min-height`, not fixed `height`, so containers grow with content.
  - Media capped with `max-width: 100%`.
  - Touch targets at least 44x44px.
  - At least 16px font on inputs, so iOS Safari doesn't zoom on focus.
  - Long unbroken strings broken with `overflow-wrap`, not left to overflow.
  - `dvh`/`svh`/`lvh` for full-height sections, so a `100vh` hero isn't clipped by the mobile address bar.
  - Wide content (tables, `<pre>`, code blocks) scrolls in its own `overflow-x: auto` container, never the page.
- **Tests at the real failure points.** 320px, 375px, the awkward 768px tablet middle, long content, and a hard check for no horizontal scroll at any width.
- **Review mode.** Point it at existing layout code and it scans for the usual offenders (fixed widths, missing `min-width: 0`, desktop-first `max-width` queries, fixed heights, unconstrained images) and fixes them.
- **Tailwind-aware.** Maps each rule to the right utilities and flags the common footguns: `w-[1200px]`, missing `min-w-0`, `w-screen`, and `min-h-screen` for heroes.

## File structure

```
doesntbreak/
├── .claude-plugin/
│   ├── plugin.json       # plugin manifest (name, version, metadata)
│   └── marketplace.json  # one-plugin marketplace, so it installs from this repo
├── skills/
│   └── doesntbreak/
│       ├── SKILL.md      # the skill instructions Claude loads
│       └── references/
│           └── patterns.md  # detailed CSS mechanics + annotated example
├── examples/
│   ├── broken.html       # demo page seeded with all 9 rule violations
│   └── fixed.html        # same content, each rule corrected
├── README.md             # this file
├── LICENSE               # MIT
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── SECURITY.md
└── CITATION.cff
```

`skills/doesntbreak/SKILL.md` carries the rules and triggers. `skills/doesntbreak/references/patterns.md` holds the deeper mechanics (the flexbox `min-width: 0` trap, `clamp()` typography, auto-fit grids, container queries, mobile viewport-height units (`dvh`/`svh`/`lvh`), safe-area insets, contained-scroll patterns for wide content (tables, `<pre>`, code), a Tailwind cheat sheet, and a fully annotated example), which Claude reads when it needs the details.

## Install

Two ways to install, depending on whether you want a managed plugin or a plain skill.

### As a plugin (from this repo's marketplace)

In Claude Code, add the marketplace and install the plugin:

```
/plugin marketplace add Kyaa-A/doesntbreak
/plugin install doesntbreak@doesntbreak
```

You get version tracking and one-command updates. Plugin skills are namespaced by the plugin name, so the command is `/doesntbreak:doesntbreak`.

### As a plain skill (for a bare `/doesntbreak`)

Copy the skill folder into your personal skills directory:

```bash
git clone https://github.com/Kyaa-A/doesntbreak.git
cp -r doesntbreak/skills/doesntbreak ~/.claude/skills/doesntbreak
```

Installed this way the command is simply `/doesntbreak`. Skill directories under `~/.claude/skills/` are not namespaced.

Either way, Claude discovers the skill automatically via the `description` field in `SKILL.md`. No configuration needed.

## Usage

You don't have to invoke it manually. Just ask Claude to build or fix any web UI and it applies the rules on its own:

- "Build a pricing page."
- "This page breaks on my phone. Fix it."

Claude triggers the skill whenever it writes or reviews layout code, even if you never say the word "responsive." To run it deliberately, type the command from your install method above (`/doesntbreak` for a plain skill, `/doesntbreak:doesntbreak` for the plugin) and you can pass context after it, e.g. `/doesntbreak audit my dashboard layout`.

## Examples

`examples/` holds a deliberately broken page and its corrected twin, so you can
watch the audit fire and compare before/after.

Audit the broken page: expect Critical findings (fixed 1200px width, `100vh`
hero, unwrapped flex row, uncontained wide table, sub-44px tap target), plus the
unconstrained image and sub-16px input:

```
/doesntbreak:audit examples/broken.html
```

Audit the corrected page: expect a clean sweep:

```
/doesntbreak:audit examples/fixed.html
```

Both pages render the same content (hero, feature cards, data table). Open them
in your browser's device toolbar at 320 / 375 / 768px: `broken.html` scrolls
sideways while `fixed.html` reflows.

## License

MIT. Copyright (c) 2026 Asnari (Kyaa-A). See [LICENSE](LICENSE).

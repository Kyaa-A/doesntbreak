---
description: Report-only responsive UI audit. Scans HTML/CSS/React/Tailwind layout code for the 9 doesntbreak rules and reports findings with file:line, severity, and exact fixes. Never edits files.
argument-hint: [path, glob, or component to audit — defaults to the project's UI files]
allowed-tools: Read, Grep, Glob
---

You are running a **report-only** responsive UI audit. Your job is to find layout
that breaks on small screens and report it. You MUST NOT edit, format, or commit
anything. Output a report only.

## Rules source

Apply the `doesntbreak` rules as your rubric. Read `skills/doesntbreak/SKILL.md`
for the rules and triggers, and read `skills/doesntbreak/references/patterns.md`
when you need the underlying mechanics or the exact corrected snippet for a
finding — rather than reasoning from memory.

## Scope

Audit target: `$ARGUMENTS`

- If an argument is given, scope to that path / glob / component.
- If no argument is given, discover the project's UI layout files: `.html`,
  `.css`, `.scss`, `.jsx`, `.tsx`, `.vue`, `.svelte`, and Tailwind config /
  class-bearing markup.
- **If no UI/layout files are found, say so plainly and stop.** Do not invent
  findings or audit non-UI code. Example: "No HTML/CSS/React/Tailwind layout
  files found in `<scope>` — nothing to audit."

## What to check (the 9 rules)

For each, scan in this order and record concrete offenders:

1. **Fixed layout widths** — fixed px `width:` on containers; Tailwind `w-[...px]`,
   `w-screen`. Fix: `min(100%, Npx)` / `max-w-*` + `w-full`.
2. **Flex/grid reflow** — flex rows without `flex-wrap`; hand-written breakpoints
   where `repeat(auto-fit, minmax(280px, 1fr))` would do. Fix: let it reflow.
3. **Fixed heights** — fixed `height:` on content containers. Fix: `min-height`.
4. **Unconstrained media** — `img`/`video`/`svg` without `max-width: 100%`.
   Fix: cap media globally.
5. **Tap targets** — interactive elements below 44×44px (icon buttons, tight
   links). Fix: padding to reach 44px.
6. **Input font size** — inputs below 16px (iOS Safari zooms on focus).
   Fix: ≥16px on inputs.
7. **Text overflow / `min-width: 0`** — flex/grid children missing `min-width: 0`
   (`min-w-0`); long unbroken strings without `overflow-wrap: break-word`;
   grid `1fr` that should be `minmax(0, 1fr)`. Fix per patterns.md §1.
8. **Mobile viewport height units** — `100vh` / `min-h-screen` / `h-screen` on
   full-height sections. Fix: `100dvh`/`svh` (`min-h-[100dvh]`).
9. **Contained scroll for wide content** — tables, `<pre>`, code blocks without an
   `overflow-x: auto` wrapper (page-level horizontal scroll risk). Fix: wrap each.

## Severity

Tag every finding:

- **Critical** — guaranteed to break mobile: causes horizontal scroll, clips the
  hero, or makes a control unusable (rules 1, 7, 8, 9; sub-44px primary actions).
- **Risk** — likely to break under real content or edge widths but not certain
  (rules 2, 3, 4; borderline tap targets).
- **Footgun** — Tailwind-specific class that invites the above (`w-[1200px]`,
  missing `min-w-0`, `w-screen`, `h-screen`/`min-h-screen` on heroes).

## Output format

Report in exactly these sections. Omit a section only if it has no entries
(except "Clean Areas / No Findings", which always appears).

### Critical Mobile Breakers
For each: `path:line` — what's wrong — **Fix:** exact suggested change.

### Likely Responsive Risks
Same shape.

### Tailwind Footguns
Same shape, keyed to the offending utility.

### Verification Suggestions
Concrete next checks: the breakpoints to test (320 / 375 / 768 / 1440px) and the
`* { outline: 1px solid red; }` overflow hunt. A dedicated `/doesntbreak:screenshot-check`
command (load each width and assert no horizontal scroll) is planned but not yet
available — until it exists, verify manually in the browser's device toolbar.

### Clean Areas / No Findings
Note rules that passed and files that looked solid, so the report reads as a full
sweep rather than a list of complaints.

## Constraints

- Report only. No edits, no reformatting, no commits — your tools are read-only by
  design.
- Every finding needs a real `path:line` you actually read. Do not guess locations.
- Suggested fixes are text in the report, not applied changes.
- Be precise over exhaustive: a wrong finding costs more than a missed nitpick.

# Contributing to doesntbreak

Thanks for your interest in improving doesntbreak. This is a Claude Agent Skill — a small set of Markdown files that teach Claude to write mobile-first web UI by default. Contributions are mostly content: clearer rules, better examples, and new responsive patterns that prevent real-world breakage.

## What lives where

- `SKILL.md` — the rules Claude loads, plus the `description` that triggers the skill. Keep it tight; this is always in context.
- `references/patterns.md` — the deep mechanics (annotated CSS, the Tailwind cheat sheet). Claude reads this on demand, so detail belongs here, not in `SKILL.md`.
- `README.md` — the human-facing overview. Keep it in sync when you change the rules.

## Principles

- **Mobile-first.** Base styles target the smallest screen; layer up with `min-width`. Never document a desktop-first `max-width` pattern as the default.
- **Every snippet must be correct and copy-pasteable.** Test the CSS. A wrong example in a skill teaches Claude the wrong thing.
- **From 320px up.** Any new pattern should hold at 320px with no horizontal scroll.
- **No filler, no emojis.** Terse, expert prose. The identifiers and examples carry the weight.

## Proposing a change

1. Open an issue describing the breakage the change prevents, ideally with a minimal repro (a CodePen/JSFiddle link or a short HTML snippet).
2. Fork the repo, branch, and make the change.
3. If you add or change a rule in `SKILL.md`, update `references/patterns.md` and `README.md` to match.
4. Open a PR. Use [Conventional Commits](https://www.conventionalcommits.org/) for the title (e.g. `feat: add container-query stacking pattern`).

## Testing a change locally

This skill has no build step. To try it:

1. Copy the `doesntbreak/` folder into your Claude skills directory (for Claude Code, `~/.claude/skills/`).
2. Ask Claude to build or fix a layout (e.g. "build a pricing page", "this breaks on mobile, fix it").
3. Confirm the output applies the rules — no fixed widths, fluid containers, and no horizontal scroll at 320 / 375 / 768px.

## Scope

doesntbreak is deliberately focused on layout that survives small screens. Accessibility, animation, and design-system concerns are adjacent but out of scope unless they directly prevent a small-screen break. When in doubt, open an issue first.
